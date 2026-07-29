---
title: Denial-of-Service (DoS)
parent: Network Security
nav_order: 12
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Denial-of-Service (DoS) Attacks

## Cheat sheet

- **Goal**: Make a service or network unavailable to legitimate users by exhausting resources or triggering bugs.
- **Two Fundamental Approaches**:
  1. Resource exhaustion ("while(1);"): consume bandwidth, CPU, memory, disk, connections, or file descriptors.
  2. Triggering a bug ("\*NULL"): send input that crashes the service or causes it to enter a bad state.
- **Key Amplification Attacks**: Smurf (ICMP), DNS amplification, NTP amplification, Memcached amplification — small request produces much larger response sent to the victim.
- **Distributed DoS (DDoS)**: Use thousands or millions of compromised machines (botnet) to multiply the attack volume.
- **Defenses**:
- Perfect prevention is impossible. The goal is usually to survive the attack long enough for the attacker to lose interest or for upstream mitigation to kick in.
  - Rate limiting, connection limits, SYN cookies, anycast scrubbing centers
  - BGP Flowspec and remote-triggered blackholing (RTBH)
  - Over-provisioning and anycast
  - Application-level quotas, proof-of-work (CAPTCHA), isolation
  - DDoS mitigation services (Cloudflare, Akamai, AWS Shield, etc.)

---

## What Is a Denial-of-Service Attack?

Since the resources in any computer or network are finite, every service has a maximum load it can handle. A web server has a limited number of simultaneous connections it can maintain, a limited amount of bandwidth on its uplink, a limited number of file descriptors, and a limited amount of CPU and memory. When any of these bottlenecks is exhausted, additional legitimate requests are dropped or delayed.

A **denial-of-service (DoS) attack** is any deliberate attempt to make a machine or service unavailable to its intended users by driving one or more of these resources past their limit. Because different parts of a system have different resource limits, an attacker only needs to exhaust the part of the system with the least resources (i.e., the bottleneck). The attacker does not need to break cryptography, steal passwords, or plant malware on the target. They only need to make the service _too busy_ (or _crashed_) to answer real users.

DoS is fundamentally an attack on **availability**.

## Why DoS Is So Hard to Defend Against

Three facts make DoS uniquely difficult compared with most other network attacks:

1. Huge attack surface. Anything that can receive a packet can potentially be DoS'ed. The attacker does not need a vulnerability in your application code; the mere fact that you are reachable on the Internet is often enough.
2. Asymmetric economics. A small amount of attacker effort can consume a large amount of victim resources. A 1 Gbps attacker can sometimes take down a 10 Gbps server if the protocol or implementation is naive.
3. No skill required for the basics. The simplest attacks are literally "send as many packets as you can." Script kiddies, botnets-for-hire, and nation-states can all play in the same game.

The defender's job is therefore not to make the attack impossible (usually impossible), but to make it _expensive enough_ or _short-lived enough_ that the attacker gives up or the business impact stays within acceptable bounds.

## Two Basic Approaches: Bugs vs. Resource Exhaustion

### Triggering a Bug ("\*NULL")

The attacker sends input that the target software was never designed to handle correctly:

- A ping packet larger than 64 KB (the famous "Ping of Death" that crashed old Windows and some network devices).
- A specially crafted TLS ClientHello or HTTP request that triggers an integer overflow or null-pointer dereference.
- A request that causes the server to fork an unbounded number of processes or allocate unbounded memory.

The defense here is the same as for any bug: keep software patched, use memory-safe languages or sanitizers, do input validation and fuzzing, and run with the least privilege possible so that a crash does not give the attacker a shell.

### Resource Exhaustion ("while(1);")

The attacker simply consumes the scarcest resource the target has. Classic examples:

- **Bandwidth**: Flood the victim's uplink so that legitimate packets are dropped before they even reach the server.
- **TCP connections / state**: Send millions of SYN packets (SYN flood). Each SYN causes the server to allocate a connection state entry that waits for the never-coming ACK.
- **CPU**: Send requests that trigger expensive cryptographic operations or database queries (e.g., exhausting processing threads with continuous calls to `fork`).
- **Memory / file descriptors**: Open as many connections or files as the OS allows (e.g., exhausting RAM with continuous calls to `malloc`).
- **Disk**: Write huge logs or upload huge files if the application permits it (e.g., exhausting filesystem space with continuous calls to `write`).

Because the attacker does not care about responses, they can spoof source IP addresses, making it hard to filter or trace the attack.

## Classic DoS Attacks

### SYN Flood

Recall the TCP three-way handshake: client sends SYN, server allocates state and sends SYN-ACK, client sends ACK. If the server never receives the final ACK, it must keep the half-open connection state for a timeout (often 30–120 seconds) before giving up.

A SYN flood sends millions of SYNs (usually with spoofed source IPs) and never sends the ACKs. The server's connection table fills up and it stops accepting new legitimate connections.

**Modern defenses**:

- **SYN cookies**: The server does not allocate state on the SYN. Instead, it encodes the necessary state in the _sequence number_ of the SYN-ACK using a cryptographic hash paired with a server-side secret. The client is forced to remember this sequence number and return it in the corresponding ACK packet. Only when the real ACK arrives does the server check the cookie against the secret and reconstruct the state. Because the client must receive the SYN-ACK to echo the correct sequence number, an attacker cannot spoof their source IP (a spoofed IP means they never receive the cookie and cannot complete the handshake), effectively eliminating resource consumption for spoofed SYNs.
- SYN rate limiting per source IP or per destination port.
- Hardware offload and connection table scaling on modern network cards.

### ICMP and Smurf Amplification

ICMP echo (ping) is a simple request/reply. In the 1990s the "Smurf" attack sent ICMP echo requests to a broadcast address with the victim's IP as the source. Every host on the LAN replied to the victim, multiplying the traffic by the number of hosts.

Modern networks disable directed broadcast and rate-limit ICMP, but the pattern, i.e., \*amplification\*\*, remains the most powerful DoS technique.

### DNS, NTP, Memcached, and Other Amplification Attacks

The general recipe for a devastating amplification attack:

1. Find a UDP-based protocol that produces a large response to a small request.
2. Spoof the source IP of the request to be the victim's address.
3. Send the tiny request to an open server (or, better, thousands of them).
4. The server(s) send the huge response to the victim.

Famous amplifiers:

- **DNS**: A ~60-byte query for a TXT record with a long response can yield 50–100× amplification. Open recursive resolvers were the original vector; today attackers also use authoritative servers that allow ANY queries or large responses.
- **NTP**: The "monlist" command (now disabled on most servers) could return a list of the last 600 clients — huge amplification.
- **Memcached**: UDP mode with no authentication allowed attackers to get 50,000× amplification in 2018 incidents that reached hundreds of Gbps.

The defense for amplification is two-fold: operators must close or rate-limit open amplifiers (and monitor for abuse), and victims must rely on upstream scrubbing because the attack volume often exceeds what any single site can handle.

Sanity check: Why is source-IP spoofing essential for amplification attacks to be practical at scale?  
Answer: Without spoofing, the huge responses would be sent back to the attacker's own machines, turning the attack into a self-inflicted DoS. Spoofing lets the attacker "launder" the traffic through third-party amplifiers and direct the flood at the real victim.

### Application-Level DoS

Not every attack needs to be a packet flood. A few carefully crafted requests can be worse than a million garbage packets:

- Upload a 10 GB file to a service that tries to read the whole thing into memory.
- Submit a query that causes a full table scan or an expensive regular-expression match on every request.
- Repeatedly call an endpoint that forks a new worker or allocates a large object without rate limits per user.

**Defenses** at the application layer:

- **Identification + Isolation + Quotas**: Know who is making each request (even approximately), make sure one user's actions cannot starve others, and enforce hard limits (requests per second, bytes uploaded, CPU time, etc.). This can also include assigning specific roles, ensuring that only trusted or authenticated users can execute computationally expensive requests.
- **Proof-of-work or CAPTCHA** for expensive operations.
- **Asynchronous processing and back-pressure** so that slow work does not block the main request path.
- **Least-privilege and sandboxing** so that a runaway request cannot take down the whole machine.

## Distributed Denial of Service (DDoS)

The real world long ago moved from single-source DoS to **DDoS**. A botnet of 10,000 compromised IoT cameras, home routers, or Windows PCs can generate attack traffic from tens of thousands of different source IPs and networks. This defeats simple IP-based filtering and can easily exceed 1 Tbps when large botnets are rented on the criminal market.

Modern DDoS is a commodity service. For a few hundred dollars an hour you can rent enough bots to take most unprotected sites offline. The attackers do not even need to write code; they point-and-click at a web panel.

## Defenses

There is no silver bullet. Good defenses are layered and assume that some attack traffic will always get through.

### At the Network Edge (What You Can Control)

- Stateless or low-state designs: SYN cookies, stateless firewalls, anycast.
- Rate limiting and connection limits at the load balancer or reverse proxy.
- BGP Flowspec or RTBH (remote-triggered blackholing) to ask your upstream to drop traffic to the attacked prefix before it reaches your link.
- Anycast and global load balancing so that attack traffic is absorbed by multiple data centers around the world.

### Upstream Scrubbing and DDoS Mitigation Services

For attacks larger than your own uplink, you need help from your ISP or a specialized DDoS mitigation provider (Cloudflare, Akamai, AWS Shield, Imperva, etc.). These providers use massive anycast networks, proprietary heuristics, and machine-learning classifiers to separate "attack" from "legitimate" traffic and forward only the good stuff to you. The good news is that this service is now cheap or even free for small sites; the bad news is that you are now dependent on the mitigation provider's capacity and willingness to keep you online.
