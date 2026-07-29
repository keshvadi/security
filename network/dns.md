---
title: DNS
parent: Network Security
nav_order: 10
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# DNS: The Domain Name System and Its Vulnerabilities

## Cheat sheet

- **Layer**: Application (Layer 7), but critical to almost every other protocol
- **Purpose**: Translate human-readable domain names (e.g., `www.example.com`) into IP addresses (and other records) that computers and routers can use
- **Core Design**: Distributed, hierarchical database with caching; relies on UDP for speed and TCP for large responses
- **Vulnerability**: No authentication or integrity protection in classic DNS. Responses can be forged by on-path or (with effort) off-path attackers.
- **Classic Attack**: DNS cache poisoning (including the 2008 Kaminsky attack) lets an attacker inject malicious records into a recursive resolver's cache, redirecting all users of that resolver to attacker-controlled IPs.
- **Impact**: Complete compromise of confidentiality, integrity, and availability for any service that uses DNS (virtually everything). Can bypass or undermine TLS, BGP defenses, etc.
- **Modern Mitigations**:
  - UDP source-port randomization + DNS transaction ID
  - DNSSEC (see next chapter)
  - DNS over HTTPS (DoH) / DNS over TLS (DoT) for confidentiality and some integrity on the last hop
  - Response Rate Limiting (RRL), bailiwick checking, and aggressive caching policies

---

## DNS and Network Security

Every time you type a URL, send an email, or open an app, your device performs a DNS lookup. The Domain Name System is the "phone book of the Internet": it turns names humans like (`mail.google.com`) into addresses routers like (`142.250.72.46`). Without it, the Internet would be unusable for people.

Because almost every protocol depends on DNS, an attack on DNS is an attack on _everything_. If an attacker can make your computer believe that `mail.google.com` lives at an IP address the attacker controls, then:

- Your browser will connect to the attacker's server instead of Google.
- Even if the real site uses TLS, the attacker can present a certificate for the real domain if they also compromised a CA or used a BGP/DNS combination attack.
- The attacker becomes a perfect man-in-the-middle for all traffic to that name.

DNS security is therefore foundational. Weaknesses in DNS can nullify the protections provided by TLS, firewalls, and higher-layer protocols. This topic explains how DNS works, why the original design is so fragile against forgery, and how the most famous DNS attack (Kaminsky 2008) exploited that fragility. The next topic (DNSSEC) shows the cryptographic fix that was eventually standardized.

## The DNS Hierarchy

It would be impossible for any single server to store every domain name on the planet and answer queries from billions of clients. DNS solves this with a **hierarchical, distributed** design.

### Zones and Delegation

The namespace is divided into **zones**. A zone is a contiguous portion of the name tree managed by a single administrative authority. For example, the `tru.ca` zone contains `www.tru.ca`, `cs.tru.ca`, `mail.tru.ca`, etc. The `ca` zone contains `tru.ca`, `ucalgary.ca`, and thousands of other second-level domains.

Each zone has one or more **authoritative name servers** that hold the definitive data for that zone. When a parent zone wants to delegate a child (e.g., `.ca` delegates `tru.ca`), it does not store the child's individual host records. Instead, it stores **NS (name server) records** that point to the child's authoritative servers, plus **glue A/AAAA records** (where **A** records map domains to IPv4 addresses, and **AAAA** records map to IPv6 addresses) that give the IP addresses of those name servers (so the resolver can actually reach them).

This delegation creates a tree:

- The **root** (`.`) is hard-coded into every resolver. There are currently 13 root server identities (operated by 12 organizations) that answer for the root zone (for example, `a.root-servers.net` at IP address `198.41.0.4`).
- **Top-level domains (TLDs)**: `.com`, `.org`, `.ca`, `.edu`, country codes, etc.
- **Second-level and deeper**: `google.com`, `tru.ca`, `cs.tru.ca`, etc.

A resolver that wants the address of `www.cs.tru.ca` starts at the root, gets redirected to a `.ca` server, gets redirected to a `tru.ca` server, and finally gets redirected (or answered) by the `cs.tru.ca` authoritative server.

### Authoritative vs. Recursive Name Servers

- **Authoritative name server**: Holds the master data for one or more zones and answers queries for those zones with "I am the authority."
- **Recursive resolver** (or "recursive name server"): The workhorse that clients actually talk to. When you configure `8.8.8.8` or your ISP's DNS server on your laptop, you are configuring a recursive resolver. It performs the full tree walk on your behalf, caches answers, and returns the final result to you.

Most clients are "stub resolvers", i.e., they simply send a query to their configured recursive resolver and wait for the answer. The recursive resolver does all the iteration and caching.

## How a DNS Query Actually Works (Iterative vs. Recursive)

Because every website lookup must start with a DNS query, DNS is designed to be extremely lightweight and fast. It relies primarily on UDP (best-effort packets) to avoid the overhead of a TCP handshake, falling back to TCP only for very large responses.

Formally, a DNS resource record is a key-value pair. The key is a 3-tuple `<Name, Class, Type>` (where `Class` is almost always `IN` for the Internet), and the value is `<TTL, Value>`. The DNS packet itself consists of a header—containing a 16-bit transaction ID and status flags like `NOERROR` (success) or `NXDOMAIN` (non-existent domain)—followed by four main sections: _Question_, _Answer_, _Authority_, and _Additional_.

There are two query styles:

- **Iterative query**: The server answers with the best information _it_ has. If it doesn't know, it gives a referral (NS + glue) to a closer server. The client is responsible for following the referrals.
- **Recursive query**: The server takes full responsibility. It will follow referrals itself until it obtains the final answer (or an error) and then return it to the client. Almost all queries from end hosts to their ISP or Google DNS are recursive.

You can observe these packet sections and unravel the iterative process yourself using the `dig +norecurse <domain>` command-line utility. A typical recursive resolution for `www.cs.tru.ca` from a stub resolver looks like:

1. Stub → Recursive (your ISP): "What is the A record for www.cs.tru.ca?"
2. Recursive (no cache) → Root: iterative query or referral request.
3. Root → Recursive: "I don't have it, but here are the NS records and glue for the `.ca` zone."
4. Recursive → a `.ca` server: "What is www.cs.tru.ca?"
5. `.ca` server → Recursive: referral to `tru.ca` servers + glue.
6. Recursive → tru.ca authoritative: query.
7. tru.ca → Recursive: referral to cs.tru.ca or the final A record.
8. Recursive → cs.tru.ca authoritative (or the parent if it has the record): final answer.
9. Recursive caches the answer with its TTL and returns it to the stub.

Every response that is not the final answer includes **authority** and **additional** sections that help the resolver continue the search.

**Caching** is essential for performance. Once a recursive resolver learns that `tru.ca` is served by certain NS records, it caches that information (and the glue) for the TTL of those records. Subsequent queries for anything under `tru.ca` can skip the root and `.ca` steps. The final A record for `www.cs.tru.ca` is also cached for its own (usually short) TTL.

## The Fatal Flaw: DNS Has No Authentication or Integrity

Classic DNS (RFC 1035 and friends) sends every query and every response in plaintext over UDP (port 53) or TCP. There is:

- No encryption (anyone on the path sees the question and answer).
- No cryptographic signature on responses.
- Only a 16-bit transaction ID (`ID` field) and (originally) the source port as "authentication."

The ID is supposed to be random and unpredictable so that an off-path attacker cannot guess which ID a resolver is expecting. In early implementations it was often sequential or predictable.

Because there is no real authentication, any party that can send a UDP packet to the resolver with the correct ID (and later also the correct source port) can inject a fake response.

### DNS and On-Path vs. Off-Path Attackers

- **On-path (or "in-path") attacker**: Can see the real query and therefore knows the exact ID, source port, and timing. They can easily forge a response that races the legitimate one. They win if their fake packet arrives first.
- **Off-path attacker**: Cannot see the query. Must guess the 16-bit ID (and later the 16-bit source port). Before Kaminsky, this was considered hard because the attacker got only a few guesses per TTL window.

Bailiwick checking helps a little: a name server is only allowed to supply records that are inside the zone it is authoritative for. A `tru.ca` server cannot tell you the address of `google.com`. But as we will see, clever attackers can stay inside the bailiwick while still poisoning useful records.

## DNS Cache Poisoning and the Kaminsky Attack (2008)

The classic goal of a DNS attacker is _cache poisoning_: get a malicious record into a recursive resolver's cache so that _every_ client using that resolver is redirected for the lifetime of the TTL (which the attacker can set very high).

Before 2008, an off-path attacker would send thousands of forged responses for a popular domain (e.g., `www.bank.com`), each with a different guessed ID. If one matched, the malicious A record would be cached. Success probability per attempt was $1/2^{16}$, and caching limited the attacker to one "race" per TTL (often minutes to hours). This was considered "theoretical" for most sites.

Dan Kaminsky realized that the attacker does not need to poison a _popular_ name. (His 2008 discovery of this flaw was so severe that he was even awarded his own Wikipedia article.) The attacker can force the victim resolver to issue queries for _non-existent_ names that the attacker completely controls in the attack.

1. The attacker tricks or forces the victim resolver (or a client behind it) to issue a query for a name the attacker just made up, e.g., `random123456.tru.ca`.
2. The legitimate response will be `NXDOMAIN` (or `NOERROR` with no answer). Nothing useful is cached for that specific name.
3. The attacker therefore gets _unlimited races_—as soon as one attempt fails (wrong ID or legitimate answer arrives first), the attacker can immediately trigger another query for a new random name under the same parent (`tru.ca`).
4. In the forged response for the random name, the attacker includes not only an answer for the random name but also **additional records** for the _parent_ zone the attacker actually wants to poison:

   ```
   ;; QUESTION: random123456.tru.ca. A
   ;; ADDITIONAL: tru.ca. 999999 IN A 6.6.6.6
   ```

   Because the response is "from" the `tru.ca` authoritative server (the one that was asked about the random child), bailiwick checking permits the additional record for `tru.ca` itself.

5. If the forged response wins the race, the resolver caches the malicious A record for `tru.ca` (or its NS/glue). All future queries for anything under `tru.ca` are redirected to the attacker's IP for the long TTL.

To force the resolver to issue the many random queries, the attacker simply serves a web page (or any content) containing hundreds of `<img src="http://randomXXXX.tru.ca/1.jpg">` tags, or uses other side channels that cause DNS lookups.

The attack was devastating because it worked against off-path attackers with only modest bandwidth, and the poisoned record could have a TTL of days or weeks.

## Partial Mitigations Before DNSSEC

After Kaminsky, the community deployed two important (but incomplete) defenses:

1. **Source-port randomization** (and later DNS cookies). The resolver now picks a random 16-bit UDP source port for each query in addition to the random 16-bit transaction ID. An off-path attacker must now guess _both_ correctly, reducing the probability to roughly \(1/2^{32}\) per packet. On-path attackers are unaffected (they see the port).

2. **Response Rate Limiting (RRL)** and other anomaly detection on authoritative servers to slow down the flood of queries that Kaminsky-style attacks generate.

These raised the bar but did not eliminate the fundamental problem: DNS responses are still unauthenticated. A sufficiently powerful attacker (or one who is on-path) can still win races or simply spoof on a local network.

## What DNS Does Not Protect (and Why It Matters)

DNS leaks almost everything:

- The full domain name you are looking up (no encryption in classic DNS).
- The IP address of the recursive resolver you trust.
- The timing and frequency of your lookups (useful for surveillance and fingerprinting).

Even if the subsequent connection is protected by TLS, the initial DNS query reveals the destination. This is why encrypted DNS transports (DoH, DoT) and DNSSEC were developed.

DNS is also a critical dependency for higher-layer security. A successful DNS poisoning attack can:

- Redirect you to a phishing site even if you type the correct name.
- Defeat HSTS preload (if the attacker controls the name long enough to serve their own HSTS header or simply never lets you reach the real site).
- Combine with BGP hijacking for even more powerful interception.
