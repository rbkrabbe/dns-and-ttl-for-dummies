# Appendix B: Glossary of DNS Terms (and Insults)

> *"The DNS glossary is the only glossary that needs its own glossary."*

---

Every field has its jargon. DNS has more than its fair share, accumulated over four decades of evolution, standardization, and creative problem-solving. This glossary covers the terms you'll encounter in the wild, along with the author's editorials in *italics*.

---

### A

**A Record**
A DNS record mapping a hostname to an IPv4 address. The original, the classic, the MVP of DNS. Without A records, nothing works.

**AAAA Record (Quad-A)**
Like an A record but for IPv6 addresses. Four times as many bits, hence four A's. *The fact that we've needed this for 30 years and still rely primarily on IPv4 is a testament to humanity's ability to procrastinate on infrastructure upgrades.*

**Anycast**
A networking technique where multiple servers share the same IP address and routing directs traffic to the nearest one. Used by root nameservers and major DNS providers. *The reason "13 root nameservers" isn't as scary as it sounds.*

**Authoritative Nameserver**
The DNS server that holds the actual, official DNS records for a domain. When you update your DNS, you update the authoritative nameserver. This is the ground truth.

---

### B

**BCP38**
Best Current Practice document recommending that ISPs filter packets with spoofed source IPs. Widely recommended, incompletely deployed, directly responsible for the continued viability of DNS amplification attacks. *We've known how to fix DNS amplification at the network level since 2000. We just haven't.*

**Bailiwick**
The scope of authority a nameserver has. A nameserver is only authoritative for records within its bailiwick. DNS resolvers use this to prevent off-topic glue records from poisoning caches. *Sounds like a term from a Terry Pratchett novel, but it's real.*

---

### C

**CAA Record (Certification Authority Authorization)**
Specifies which Certificate Authorities are allowed to issue certificates for your domain. *The DNS record that stops rogue certificate issuance. Add it to your zones. Today.*

**Cache**
A temporary storage of DNS answers so they don't need to be looked up fresh every time. The reason DNS is fast. Also the reason DNS changes take time to propagate. The simultaneous best and worst thing about DNS.

**Cache Poisoning**
An attack where malicious DNS records are injected into a resolver's cache, directing users to attacker-controlled servers. Fixed by DNSSEC. Not everyone has deployed DNSSEC. *This is fine.*

**CNAME (Canonical Name)**
An alias record pointing one hostname to another. Cannot exist at the zone apex. Cannot coexist with other records for the same name. These constraints cause 40% of DNS support tickets. *The record that looks simple but isn't.*

**CoreDNS**
The default DNS server for Kubernetes clusters since version 1.13. Written in Go, plugin-based, highly configurable. The Corefile is its configuration. *Better than its predecessor, kube-dns, by most measures.*

**cluster.local**
The default internal DNS domain for Kubernetes clusters. All service names resolve to something.cluster.local. Technically violates the .local mDNS reservation but works because Kubernetes controls its own DNS.

---

### D

**dig**
*Domain Information Groper.* The command-line DNS query tool. Your best friend when debugging DNS. Should be the first tool you reach for when anything DNS-related seems wrong. *The DNS Swiss Army knife. Learn it.*

**DKIM (DomainKeys Identified Mail)**
An email authentication mechanism that uses cryptographic signatures stored in DNS TXT records. Allows recipients to verify that email claiming to be from your domain was actually sent by you. *Without DKIM, your email domain is a welcome sign for phishers.*

**DMARC (Domain-based Message Authentication, Reporting and Conformance)**
A DNS-based policy that tells mail receivers what to do when SPF and DKIM fail. Also provides reporting so you know who is sending email that claims to be from your domain. *Set it to `p=reject` and actually mean it.*

**DNS (Domain Name System)**
The hierarchical, distributed database that maps human-readable domain names to IP addresses and other information. The invisible foundation of the internet. *If DNS breaks, nothing else works. Act accordingly.*

**DNS Amplification**
A DDoS technique that abuses open DNS resolvers to flood victims with traffic. The attacker sends small queries spoofing the victim's IP; the resolvers send large responses to the victim. *The reason "run an open resolver" is terrible advice.*

**DNS Cache Snooping**
A technique to determine which domains a recursive resolver has cached by querying it non-recursively. Can reveal information about recent DNS activity. *Not a major threat, but interesting for reconnaissance.*

**DNS Hijacking**
Taking over DNS records for a domain you don't own. Can happen through compromised registrar accounts, social engineering, or attacking DNS infrastructure. *Enable MFA on your DNS provider. Please.*

**DNS Propagation**
The process by which DNS changes spread across the internet as caches expire. Not actually a "push" — it's cache expiry. *"DNS can take 24-48 hours to propagate" means "someone set a 24-48 hour TTL." Lower your TTL.*

**DNS Rebinding**
An attack where a malicious website changes its DNS TTL to 0 and then starts responding with internal IP addresses, potentially allowing a browser to make requests to internal services. *One of those attacks that sounds impossible until you understand how browsers work.*

**DNSSEC (DNS Security Extensions)**
An extension to DNS that adds cryptographic signing to records, allowing resolvers to verify that answers haven't been tampered with. The correct solution to cache poisoning. *Technically brilliant, operationally terrifying, but you should still deploy it.*

**DoH (DNS over HTTPS)**
DNS queries sent as HTTPS requests over port 443. Encrypted and indistinguishable from regular web traffic. *Loved by privacy advocates. Despised by network administrators. Both groups have valid points.*

**DoQ (DNS over QUIC)**
DNS over the QUIC transport protocol. Encrypted, low-latency. The newest DNS transport. *The future, in about 5 years.*

**DoT (DNS over TLS)**
DNS queries encrypted with TLS over TCP port 853. Encrypted but recognizable as DNS traffic. *Better than plaintext DNS. Slightly less controversial than DoH.*

**DS Record (Delegation Signer)**
A DNSSEC record in a parent zone that contains a hash of the child zone's KSK. Forms part of the chain of trust in DNSSEC. *The glue that holds the DNSSEC trust chain together.*

---

### E

**EDNS0 (Extension Mechanisms for DNS, version 0)**
An extension to the DNS protocol that enables larger UDP message sizes and additional fields for DNSSEC and other features. Supported by all modern DNS software. *Some firewalls still break it. Those firewalls are wrong.*

**ECS (EDNS Client Subnet)**
An EDNS extension that includes a truncated client IP address in DNS queries, enabling geographically accurate responses. *The privacy trade-off: more accurate CDN routing in exchange for revealing your approximate location.*

---

### F

**Forwarder**
A DNS server configured to forward queries to another DNS server rather than performing full recursive resolution. Common in corporate environments. *The reason your office can sometimes resolve internal names but not external ones, or vice versa.*

**FQDN (Fully Qualified Domain Name)**
A domain name that specifies its complete location in the DNS hierarchy, ending with a trailing dot: `www.example.com.` *The trailing dot matters in zone files. Forgetting it is mistake #2 in Chapter 7.*

---

### G

**GeoDNS**
A DNS service that returns different answers based on the geographic location of the querying resolver. Used for directing users to the nearest server. *Works well with ECS. Works less well when your users are behind a resolver far from them.*

**Glue Record**
An A or AAAA record in a parent zone for a nameserver that is "in-zone" (within the domain it serves). Needed to break the chicken-and-egg problem of looking up a nameserver's address when the nameserver is within the domain it serves. *Forgetting to update glue records is a classic DNS migration mistake.*

---

### H

**Headless Service (Kubernetes)**
A Kubernetes Service with `clusterIP: None`. Instead of a single virtual IP, DNS returns the individual Pod IPs. Essential for stateful applications. *The right tool for databases, caches, and anything else where individual pod identity matters.*

**HOSTS file**
A local file (`/etc/hosts` on Linux/Mac, `C:\Windows\System32\drivers\etc\hosts` on Windows) that overrides DNS for specific names. The ancestor of DNS. *The ghost of DNS past. Checked before DNS. Responsible for many "why isn't DNS working?" incidents when you forgot you edited it.*

---

### I

**ICANN (Internet Corporation for Assigned Names and Numbers)**
The non-profit organization responsible for coordinating internet naming and numbering systems, including top-level domains and IP address allocation. *The gatekeeper of the internet's address book. Their Key Signing Ceremonies are theatrical masterpieces.*

**in-addr.arpa**
The special domain used for reverse DNS lookups. IPv4 addresses are reversed and appended: `34.216.184.93.in-addr.arpa` for IP `93.184.216.34`. *Because things that are already confusing should be extra confusing.*

---

### K

**KSK (Key Signing Key)**
The DNSSEC key used to sign the Zone Signing Key (ZSK). Kept highly secure and rotated infrequently. *The key that signs the key that signs the keys. Key all the way down.*

**kube-dns**
The predecessor to CoreDNS in Kubernetes. Based on dnsmasq and a custom Go binary with a sidecar container. Replaced by CoreDNS in Kubernetes 1.13. *Not missed by many.*

---

### L

**Lame Delegation**
When a domain's NS records point to nameservers that are not configured to be authoritative for that domain. Causes SERVFAIL errors. *It's in the name. If your delegation is lame, your DNS is broken.*

---

### M

**mDNS (Multicast DNS)**
DNS-like name resolution without a central server, using multicast packets. Used for `.local` names on local networks. *The protocol behind printer discovery. The reason `.local` is reserved and you shouldn't use it for internal domains.*

**MX Record (Mail Exchange)**
DNS record specifying mail servers for a domain. Must point to hostnames, not CNAMEs. Lower priority number = higher priority. *The record that controls your email. The one nobody updates until email breaks.*

---

### N

**Nameserver**
A server that stores and serves DNS records. Can be authoritative (holds the actual records), recursive (looks up records on behalf of clients), or both.

**ndots**
A resolver option specifying how many dots a hostname must have to be treated as fully qualified without appending search domains. Default in Kubernetes is 5. *The number responsible for 4x DNS query overhead for external names in Kubernetes. Set it to 2 and tell your team.*

**Negative Caching**
Caching of "this name doesn't exist" (NXDOMAIN) responses. Duration controlled by the SOA minimum TTL field. *The reason newly created DNS records can take time to resolve even after propagation: someone already cached the negative answer.*

**NodeLocal DNSCache**
A Kubernetes add-on that runs a DNS cache on every node as a DaemonSet, reducing latency and CoreDNS load. *The correct solution to DNS performance problems at scale in Kubernetes.*

**NS Record**
Specifies the authoritative nameservers for a domain. Set at both the registrar and in the zone itself. *Changing these is the slowest DNS change. Also the most consequential.*

**NXDOMAIN**
DNS response code meaning "this domain doesn't exist." The DNS equivalent of "we don't have that here." *Usually a typo. Sometimes a deleted record. Occasionally the authoritative server being broken.*

---

### O

**Open Resolver**
A recursive DNS resolver that answers queries from any IP address on the internet. Great for the public (see: 8.8.8.8). A liability if it's your server because it can be used for amplification attacks.

---

### P

**PTR Record (Pointer)**
Reverse DNS record mapping an IP address to a hostname. Lives in `in-addr.arpa` (IPv4) or `ip6.arpa` (IPv6) zones. Controlled by the IP block owner, not the domain owner. *Essential for email deliverability. Forgotten until email ends up in spam.*

---

### R

**RCODE (Response Code)**
4-bit code in a DNS response indicating success or failure type. NOERROR, NXDOMAIN, SERVFAIL, REFUSED, etc. *The first thing to check when DNS is broken.*

**Recursive Resolver**
A DNS server that performs the full DNS resolution process on behalf of clients, walking from root to authoritative. Also called a recursive nameserver or caching resolver.

**Registrar**
A company accredited by ICANN to register domain names. They charge you money, point your domain to your nameservers, and hopefully have good security. *Enable MFA. Always.*

**Response Rate Limiting (RRL)**
A feature on authoritative nameservers that limits how often they respond to similar queries from the same source, mitigating their use in amplification attacks. *Simple, effective, should be on by default.*

**Reverse DNS**
Looking up a hostname from an IP address, using PTR records in the `in-addr.arpa` zone. The opposite of normal DNS. *Like going to a phone book knowing someone's number but wanting their name. Useful for email, logging, and forensics.*

**Root Hints**
The hardcoded list of root nameserver addresses that every recursive resolver uses to start its queries. *Has not needed to change since 1997. This is an engineering success story.*

**Root Nameserver**
The 13 DNS servers (by address) at the top of the DNS hierarchy that can answer queries about TLD nameservers. ~1600 physical instances worldwide via anycast. *13 addresses. 1600 servers. Billions of queries per day. Works fine.*

**RR (Resource Record)**
The technical term for a DNS record. "Resource Record" is the formal name; everyone just says "record."

**RRSIG**
The DNSSEC record that contains a cryptographic signature for a set of DNS records. Used to verify that records were signed by the zone's ZSK. *Without RRSIGs, there's no DNSSEC.*

---

### S

**Search Domain**
A domain suffix configured in a resolver that is appended to short hostnames before looking them up. Allows `ssh server` instead of `ssh server.corp.example.com`. *Convenient until it resolves the wrong thing.*

**SERVFAIL**
DNS response code for "server failure." The nameserver couldn't complete the query. Caused by DNSSEC failures, unreachable nameservers, or resolver bugs. *The vaguest DNS error. Could be anything.*

**SOA Record (Start of Authority)**
The authoritative record for a DNS zone, containing administrative info including the primary nameserver, admin contact, serial number, and various TTL values. *Required. Every zone has one. The serial number must be incremented when records change.*

**SPF (Sender Policy Framework)**
A DNS TXT record specifying which servers are authorized to send email for your domain. First line of defense against email spoofing. *~all is soft fail. -all is hard fail. Use -all.*

**Split-Brain DNS (Split-Horizon DNS)**
Serving different DNS answers to different clients (typically internal vs external). Useful when intentional. A debugging nightmare when accidental.

**SRV Record (Service Record)**
A DNS record specifying service location including hostname, port, priority, and weight. Used by XMPP, SIP, and Kubernetes internal DNS. *The record type that would solve many problems if people used it. People don't use it much.*

**Stub Resolver**
The minimal DNS client built into your operating system that forwards queries to a recursive resolver. Not the same as a recursive resolver.

---

### T

**TLD (Top-Level Domain)**
The last part of a domain name: `.com`, `.org`, `.net`, `.io`, etc. Managed by registries delegated by ICANN. *`.com` has over 150 million registered domains. The most valuable two-letter namespace in human history.*

**TTL (Time To Live)**
The number of seconds a DNS record should be cached by resolvers. The most important setting in DNS that most people set once and forget. *Chapter 4 has entered the chat.*

**TXT Record**
A DNS record storing arbitrary text. Used for SPF, DKIM, domain verification, ACME challenges, and random notes from 2014 that nobody will ever delete. *The junk drawer of DNS.*

---

### U

**UDP**
The default transport for DNS queries. Fast, connectionless. Falls back to TCP for large responses. Port 53. *Why DNS is fast. Also why DNS can be spoofed (UDP has no connection state).*

---

### W

**Wildcard DNS**
A DNS record using `*` to match any subdomain without a more specific record. Convenient for multi-tenant applications. Potentially dangerous if not managed carefully. *`*.example.com` matches anything. Including typos.*

---

### Z

**Zone**
A portion of the DNS namespace managed by a specific set of nameservers. Every domain is a zone. Subdomains can be delegated into separate zones.

**Zone File**
A text file containing DNS records in BIND zone file format. The original way to manage DNS records. Still used by many DNS servers. Mind your trailing dots.

**ZSK (Zone Signing Key)**
The DNSSEC key used to sign the actual DNS records in a zone. Changed more frequently than the KSK. *Signs the things. Needs to be rotated. Not as scary as KSK rotation.*

---

## DNS Meme Glossary

Because no DNS book would be complete without it:

| Phrase | Meaning |
|--------|---------|
| "It's always DNS" | 90% of the time, it's DNS |
| "Have you tried flushing your DNS cache?" | The DNS equivalent of "have you tried turning it off and on again?" |
| "DNS can take 24-48 hours to propagate" | Usually means TTL was set too high, not that DNS is slow |
| "Works on my machine" | You're on a different resolver than the broken user |
| "I already checked DNS" | The record you checked is not the record that's broken |
| "Just add a CNAME" | Famous last words before an RFC violation |
| "The TTL is fine" | Famous last words before a 24-hour outage window |
| "I'll fix the TTL after the migration" | *sobs in propagation delay* |

---

*[← Back to Appendix A](appendix-a-cheatsheet.md) | [Back to Table of Contents](../README.md)*

---

*This book was written during many cups of coffee, several DNS incidents, and at least one 3am pager alert that was, in fact, DNS. May your TTLs be low and your caches always warm.*

*— The Author*

*P.S. If you found a DNS error in this DNS book, please lower your TTL before submitting a correction. Thank you.*
