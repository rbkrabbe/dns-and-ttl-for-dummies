# Appendix A: DNS Cheat Sheet

> *"Everything you need to remember, on one page — because DNS is complicated enough without also requiring you to memorize it."*

---

## DNS Record Types Quick Reference

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | IPv6 address | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Alias to another name | `www → example.com` |
| **MX** | Mail server (must point to A/AAAA, not CNAME) | `example.com → 10 mail.example.com` |
| **TXT** | Arbitrary text (SPF, DKIM, verification) | `"v=spf1 include:..."` |
| **NS** | Authoritative nameservers | `example.com → ns1.provider.com` |
| **SOA** | Zone authority record | Primary NS, admin email, serial, timers |
| **PTR** | Reverse DNS (IP → name) | `34.216.184.93.in-addr.arpa → example.com` |
| **SRV** | Service location with port | `_http._tcp.example.com → 80 www.example.com` |
| **CAA** | Authorized certificate authorities | `0 issue "letsencrypt.org"` |

---

## TTL Reference

| Situation | Recommended TTL |
|-----------|----------------|
| Active migration / incident | 60s |
| Load-balanced / dynamic services | 300s |
| Standard web service | 300–3600s |
| Very stable infrastructure | 3600s |
| NS records | 86400–172800s |
| **Default recommendation** | **300s (5 minutes)** |

**The Golden Rule:** Lower TTL **before** you need to make a change. Wait for the old TTL to expire. Then make the change.

---

## dig Command Reference

```bash
# Basic lookup
dig example.com                          # A record
dig example.com MX                       # Mail servers
dig example.com TXT                      # Text records
dig example.com NS                       # Nameservers
dig example.com SOA                      # Zone authority

# Query specific server
dig @8.8.8.8 example.com                 # Google DNS
dig @1.1.1.1 example.com                 # Cloudflare DNS
dig @ns1.example.com example.com         # Authoritative (bypass cache)

# Output control
dig +short example.com                   # Just the answer
dig +noall +answer example.com           # Only answer section
dig +trace example.com                   # Full resolution trace

# Special queries
dig -x 93.184.216.34                     # Reverse lookup
dig +dnssec example.com                  # Include DNSSEC records
dig example.com +cd                      # Disable DNSSEC validation

# Check TTL remaining in cache
dig @8.8.8.8 example.com | grep -A1 "ANSWER SECTION"
# The number after the name is the TTL remaining
```

---

## Kubernetes DNS Quick Reference

### Service DNS Names

```
<service>.<namespace>.svc.cluster.local     # Full FQDN
<service>.<namespace>.svc                   # Works within cluster
<service>.<namespace>                       # Works within cluster
<service>                                   # Works within same namespace only
```

### StatefulSet Pod DNS Names

```
<pod-name>.<service>.<namespace>.svc.cluster.local
cassandra-0.cassandra.default.svc.cluster.local
```

### /etc/resolv.conf in a Pod

```
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

### Debug Commands in Kubernetes

```bash
# Deploy debug pod
kubectl run debug --image=infoblox/dnstools -it --rm -- sh

# Inside the pod:
nslookup kubernetes.default                        # Test cluster DNS
nslookup my-svc.my-ns.svc.cluster.local           # Test service DNS
dig @10.96.0.10 my-svc.my-ns.svc.cluster.local    # Query CoreDNS directly
cat /etc/resolv.conf                               # Check DNS config

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl get configmap -n kube-system coredns -o yaml
```

---

## DNS Migration Procedure

```
1. CHECK current TTL:
   dig example.com | grep -A1 "ANSWER SECTION"

2. LOWER TTL to 300:
   Update record: TTL = 300

3. WAIT for old TTL to expire:
   If old TTL was 86400, wait 24 hours
   If old TTL was 3600, wait 1 hour
   (You MUST wait this long)

4. MAKE the DNS change

5. VERIFY propagation:
   dig @8.8.8.8 example.com     ← Google
   dig @1.1.1.1 example.com     ← Cloudflare
   dig @ns1.example.com example.com ← Authoritative

6. WAIT for new TTL (300s):
   5 minutes

7. RAISE TTL back to 3600 (optional)
```

---

## DNS Response Codes

| Code | Name | Meaning |
|------|------|---------|
| 0 | NOERROR | Query succeeded |
| 1 | FORMERR | Malformed query |
| 2 | SERVFAIL | Server error (check DNSSEC, NS reachability) |
| 3 | NXDOMAIN | Domain doesn't exist |
| 4 | NOTIMP | Query type not supported |
| 5 | REFUSED | Server won't answer (check ACLs) |

---

## Email DNS Checklist

```
☐ MX record exists and points to correct hostname (NOT CNAME)
☐ MX hostname has A record with correct IP
☐ PTR record set at hosting provider (IP → mail hostname)
☐ SPF TXT record includes all sending sources
☐ DKIM key published in DNS (selector._domainkey.example.com)
☐ DMARC policy published (_dmarc.example.com)
☐ DMARC reporting email is monitored
☐ No MX records pointing to CNAMEs (RFC violation)
```

---

## Common TTL Mistakes and Fixes

| Mistake | Fix |
|---------|-----|
| High TTL during migration | Lower TTL 24-48h before, wait for expiry, then change |
| TTL = 0 in production | Set to at least 60s |
| TTL = 86400 by default | Evaluate and lower to 300-3600 |
| Java app ignoring TTL | Set `networkaddress.cache.ttl=30` in security.properties |
| Not waiting for TTL expiry after lowering | Actually wait the full old TTL duration |

---

## The DNS Debugging Decision Tree

```
Problem: "DNS isn't working"

1. Can you resolve anything?
   No → Check network connectivity to DNS server
   Yes → Continue

2. Can you resolve the specific name from the authoritative server?
   dig @ns1.example.com www.example.com
   No → Record doesn't exist or wrong zone → Create/fix record
   Yes → Continue

3. Is it a caching issue?
   dig @8.8.8.8 example.com (check TTL remaining)
   High TTL remaining → Wait for it to expire
   TTL expired but wrong answer → Something else

4. Is it a propagation issue?
   Different answers from different resolvers → Wait, propagation in progress
   Same wrong answer everywhere → Problem at authoritative server

5. Is it DNSSEC?
   dig example.com +cd (bypass DNSSEC validation)
   Works with +cd but not without → DNSSEC misconfiguration
   dig example.com DNSKEY → Check keys

6. Is it application-level caching?
   Java? → Set networkaddress.cache.ttl
   Node.js? → Check DNS caching in your HTTP client
```

---

*[← Back to Chapter 12](12-advanced-dns.md) | [Appendix B: Glossary →](appendix-b-glossary.md)*
