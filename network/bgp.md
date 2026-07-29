---
title: BGP
parent: Network Security
nav_order: 9
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# BGP Route Hijacking

## Cheat sheet

- **Layer**: Network / Inter-domain routing (Layer 3)
- **Purpose**: Enable routing of traffic between thousands of Autonomous Systems (ASes) across the global Internet
- **Vulnerability**: BGP is based on trust. There is no built-in authentication or authorization. Any AS can announce routes for IP prefixes it does not own or control.
- **Attack Result**: An attacker can _hijack_ traffic destined for a prefix. This can be used to:
  - Blackhole traffic (drop it) → Denial of Service
  - Intercept traffic as a man-in-the-middle → eavesdrop, collect metadata, or modify data
- **Core Weakness**: BGP has no mechanism to verify that an AS is authorized to announce a particular IP prefix
- **Defenses**:
  - Very limited at the BGP layer itself
  - _RPKI_ (Resource Public Key Infrastructure) provides cryptographic proof of authorization via Route Origin Authorizations (ROAs); adoption is growing but incomplete
  - Operational practices: prefix filtering, max-prefix limits, monitoring
  - Strong reliance on higher-layer protocols:
    - TCP (for reliability and connection integrity)
    - TLS (for confidentiality, integrity, and endpoint authentication)
  - Even with TLS, metadata and "harvest now, decrypt later" remain concerns

---

## Border Gateway Protocol (BGP)

Imagine you are a student at TRU University. You open your laptop in the dorm, connect to the campus Wi-Fi, and type `www.example.com` into your browser. Within a second or two the page appears. Behind that simple click lies an extraordinarily complex piece of global infrastructure: thousands of independently run networks have to agree, in real time, on exactly which path your packets should take to reach the server and which path the server's replies should take back to you.

The protocol responsible for that agreement is the **Border Gateway Protocol (BGP)**. BGP's job is deceptively simple: it lets each network (called an Autonomous System) tell its neighbors, "Here are the IP prefixes I can reach, and here is the sequence of other networks you would traverse to get there." When everything works, the system is remarkably robust. When it fails, by accident or by deliberate attack, entire countries can suddenly lose access to YouTube, a cryptocurrency exchange can be robbed in minutes, or an intelligence agency can silently copy traffic that was never intended to cross its borders.

The root security problem is not a flaw in any particular router's software. It is a design decision made when the Internet was small and cooperative: BGP assumes that every network tells the truth about which addresses it owns. In today's adversarial environment that assumption is routinely violated. This chapter explains exactly how the protocol works, why the trust model is so fragile, how attackers exploit it, what real incidents have looked like, and why the available defenses, while improving, are still fundamentally limited.

## Background: From Your Laptop to the Global Internet

Before we can discuss attacks, we need a clear mental model of how packets actually travel.

### IP Addresses and Subnets

Every device that wants to communicate on the Internet needs an IP address. But the Internet does not keep a separate routing entry for every one of the billions of individual addresses. Instead, addresses are grouped into _subnets_ (also called prefixes) that share a common prefix of bits.

A subnet is written in CIDR notation: `base-address/prefix-length`. For example:

- `192.0.2.0/24` means "the first 24 bits are fixed; the remaining 8 bits can vary."
- This subnet contains \(2^{8} = 256\) addresses (from 192.0.2.0 to 192.0.2.255).

Sanity check: How many addresses are in the subnet `10.0.0.0/8`?  
Answer: \(2^{24} = 16{,}777{,}216\) addresses.

When a host wants to send a packet, it first checks whether the destination IP is inside one of its own subnets (using its netmask). If yes, it can deliver the packet locally (usually via ARP on Ethernet). If not, it forwards the packet to its _default gateway_, the router that knows how to reach the rest of the world.

### Autonomous Systems (ASes)

Your laptop and the Wi-Fi router in your dorm are not "the Internet." They are a tiny stub network inside a much larger network run by a single organization—an **Autonomous System (AS)**. An AS is a collection of IP networks under the control of one organization (an ISP, a university, a cloud provider, a government, etc.) that presents a single, consistent routing policy to the rest of the Internet.

Each AS is identified by a unique Autonomous System Number (ASN). Examples you might recognize:

- Large ISPs: Comcast (AS 7922), Deutsche Telekom, Telus
- Content and cloud giants: Google (AS 15169), Amazon, Cloudflare, Microsoft
- Universities and research networks: many have their own ASNs
- Governments and critical infrastructure operators

The global Internet is the result of these ASes interconnecting and exchanging reachability information. When a packet leaves the TRU campus network (let's say TRU operates AS 12345), it enters this much larger system of ASes, where the real inter-domain routing decisions happen. The router at the edge of TRU's network does not know every individual host on the planet; it only needs to know which of its neighbors can eventually deliver packets to the destination prefix. That knowledge comes from BGP.

## How BGP Works: Announcements, Paths, and Decisions

BGP is the protocol that ASes use to tell each other, "I can reach these IP prefixes, and here is the sequence of ASes you would have to traverse to get there."

### Route Advertisements

When an AS decides to advertise a prefix, it sends a BGP UPDATE message to its neighbors. A minimal advertisement contains:

- The IP prefix (e.g., `203.0.113.0/24`)
- The **AS Path**—the list of ASes the traffic would cross
- Various attributes (origin, next-hop, communities, etc.)

A simple example of propagation:

1. AS 1234 owns `203.0.113.0/24` and advertises it to its neighbor AS 5678 with the path `[1234]`.
2. AS 5678 accepts the route, installs it in its routing table, and re-advertises it to _its_ neighbors with the path `[5678, 1234]`.
3. Those neighbors add themselves and continue advertising.

Within minutes, a new prefix can be known by routers on the other side of the planet.

### The AS Path and Loop Prevention

The AS Path is not just informational—it is BGP's primary loop-prevention mechanism. When a router receives an advertisement that already contains its own ASN in the path, it rejects the route. This simple rule prevents packets from circulating forever in a loop.

### Longest-Prefix Match

When a router has multiple routes that could match a destination IP, it always prefers the **longest (most specific) prefix**. This is a fundamental rule of IP forwarding, not a BGP invention.

Example: if a router has both `208.65.152.0/22` (YouTube's old block) and `208.65.153.0/24` (a sub-block), traffic for 208.65.153.1 will follow the /24 route even if the /22 route looks "better" by other metrics. This rule is crucial for understanding many hijacking attacks.

Sanity check: Suppose your router has learned two routes for the same destination: a /16 via a fast but expensive provider and a /24 via a slow but cheap link. Which route will it actually use for addresses inside that /24?  
Answer: The /24, because it is the longest (most specific) match. The /16 will only be used for addresses that do not fall inside any more-specific route.

### Path Selection and Policy

When multiple paths exist to the same prefix, BGP does not simply pick the shortest AS Path (although that is the default tie-breaker). Each AS applies its own **routing policy**, which can consider:

- Business relationships (customer > peer > provider)
- Local preference values
- AS Path length
- Origin type
- IGP metric to the border router
- etc.

The result is that "shortest path" in BGP is often not the shortest in terms of latency or geography—it is the path that best satisfies the local operator's business and policy goals.

Why does any AS accept routes from its neighbors in the first place? The answer lies in the economics of Internet connectivity. When TRU buys **transit service** from a large provider (a "provider" relationship), the provider is contractually obligated to deliver packets _to_ TRU's prefixes _and_ to carry TRU's outbound traffic to the rest of the Internet. In return, TRU pays money. When two large ISPs connect at a **peering** point, they typically agree to exchange traffic for free as long as the volumes are roughly balanced. When a smaller network (a "customer") connects to TRU, TRU may provide transit _for_ that customer. In all cases, the business relationship dictates which routes are accepted, which are exported, and which are preferred. BGP policy is the technical mechanism that implements these business contracts. Without those contracts, the global Internet would not function.

### Modern Default Behavior

Early BGP implementations accepted every route they received by default ("permit-by-default"). This led to several large-scale outages from simple misconfigurations. The modern specification (RFC 8212) requires that eBGP sessions default to "deny-by-default": an operator must explicitly configure an import or export policy before any routes are accepted or advertised. This change protects against accidental leaks and misconfigurations, but it does _not_ solve the authorization problem. Once a policy is configured to accept routes from a neighbor (which is required for connectivity), the router still has no way to know whether the neighbor is authorized to announce the prefixes it is advertising.

## The Root Cause: BGP Has No Notion of Authorization

The single most important security fact about BGP is this:

> **BGP has no built-in mechanism to verify that an AS is actually authorized to originate or transit a particular IP prefix.**

There is no cryptographic signature on route announcements in the base protocol, no central database that is consulted in real time, and no strong authentication that proves "this AS really owns this address block". When an AS announces a prefix, its neighbors generally believe it, because the protocol gives them no reason not to.

This design made sense in the early, cooperative days of the Internet. It is catastrophic in an adversarial environment.

## The Attack: BGP Route Hijacking

A **BGP route hijack** (also called prefix hijacking) occurs when an AS announces reachability to an IP prefix that it does not actually own or have a legitimate path to. Other ASes that accept the announcement will begin forwarding traffic for that prefix toward the attacker.

There are two common flavors of hijack:

- **Origin hijack**: The attacker simply claims "I own this prefix" (or announces a more-specific sub-prefix). This is the easiest to execute and the most common in real incidents. The 2008 YouTube outage was an origin hijack.
- **Path hijack** (or "AS-path poisoning" / man-in-the-middle at the routing layer): The attacker inserts itself into the middle of an existing AS path, claiming "I have a better or equally good path through me." These are harder to pull off at scale but can be used for targeted interception when the attacker already has a presence near the target.

### Two Primary Attack Goals

**1. Blackholing (Denial of Service)**
The attacker announces the prefix but does not forward the traffic anywhere useful. Packets arrive at the attacker's routers and are dropped (or sent into a "black hole"). The legitimate destination becomes unreachable from anywhere that accepted the bogus route. This is a pure availability attack.

**2. Traffic Interception (Man-in-the-Middle)**
The attacker announces the prefix _and_ forwards the received traffic onward to the real destination (usually over a different path). Because the traffic still reaches its intended recipient, the attack can be extremely stealthy. The attacker can now:

- Passively observe all traffic (even if encrypted, metadata remains visible)
- Actively modify packets in transit
- Inject new packets
- Perform targeted attacks on protocols that lack strong endpoint authentication

Because the attacker is on the forwarding path chosen by BGP, the victims have no reason to suspect anything is wrong at the IP layer.

A subtle but important detail: BGP is _destination-based_. A hijack of the victim's prefix only controls traffic _heading toward_ the victim. Return traffic (from the victim back to clients) follows whatever routes the client's networks have installed for the clients' own prefixes. The result is often _asymmetric routing_, i.e., the attacker's network sees only one direction of the flow. This is still extremely useful for passive surveillance and metadata collection, and it can be sufficient for certain active attacks, but it makes full bidirectional man-in-the-middle manipulation more difficult than a traditional local-network MITM.

You might hope that IP header checksums, TTL values, or sequence numbers would detect a redirect. They do not. Every legitimate router that forwards a packet is _supposed_ to decrement the TTL and update the header checksum. A malicious router can do exactly the same thing while also changing the next-hop or even the payload (and recomputing the checksum). From the perspective of every other router and the destination, the packet looks perfectly valid.

## Real-World Consequences

Because BGP controls the actual paths that packets take, a successful hijack can affect millions of users in minutes.

### The 2008 YouTube Incident

In February 2008, the government of Pakistan ordered Pakistan Telecom (AS 17557) to block access to YouTube inside the country. Engineers attempted to achieve this by announcing a more-specific prefix (`208.65.153.0/24`) for YouTube's address space into their local network, intending to blackhole the traffic domestically.

Because of a configuration error, the more-specific route was leaked to Pakistan Telecom's upstream provider, which then propagated it to the global Internet. Due to the longest-prefix-match rule, routers worldwide preferred the fraudulent /24 over YouTube's legitimate /22 advertisement. Within minutes, a large fraction of the world's YouTube traffic was being sent into Pakistan Telecom's network, where it was blackholed. The outage lasted several hours and was visible on every continent.

This was an accident, not malice, but the exact same mechanism can be (and has been) used deliberately.

The same technique has been used deliberately for more sinister purposes. A criminal group can redirect traffic destined for a cryptocurrency exchange to a server they control, present a fake login page, and steal credentials and two-factor codes in real time. A nation-state can divert traffic from a foreign university or government agency through its own territory, gaining both surveillance and the ability to inject malicious content. Ad-fraud operations have used hijacks to manufacture fake "views" on video platforms by routing traffic through bots they control. Even a hijack that lasts only a few minutes can be devastating for financial transactions, live video, or emergency services that depend on low-latency reachability.

The key lesson for a defender is that the damage is not limited to the moment the route is active. Once an attacker has demonstrated the ability to blackhole or intercept a prefix, they can repeat the attack whenever it suits them, and the global Internet has no built-in way to remember "this AS lied last time."

## Defenses and Their Limitations

Securing BGP is fundamentally a problem of _authorization at Internet scale_. There is no central authority that can instantly bless every route announcement. Each AS is sovereign over its own routing decisions.

### RPKI: Cryptographic Authorization

The most promising technical defense is the **Resource Public Key Infrastructure (RPKI)**.

- Regional Internet Registries (ARIN, RIPE, APNIC, etc.) issue X.509 certificates to organizations that prove they hold certain IP address blocks and ASNs.
- The organization can then create a **Route Origin Authorization (ROA)**, a signed statement that says "AS X is authorized to originate prefix Y."
- Routers that perform **Route Origin Validation (ROV)** can check incoming BGP announcements against the published ROAs. An announcement that is not covered by a valid ROA can be marked "invalid" and filtered or given lower preference.

Current status (mid-2020s): A majority of announced IPv4 and IPv6 prefixes now have at least one ROA published. However, the fraction of networks that actually _enforce_ Route Origin Validation (ROV)—dropping or de-preferencing invalid announcements—is much smaller. When a few large transit providers and content networks enforce ROV, they protect a substantial portion of global traffic, but many networks still do not validate. Full, uniform protection across the Internet is still years away. RPKI is therefore a valuable and steadily improving defense, but it is not yet a complete solution.

RPKI is a partial defense even when widely deployed: it primarily protects _origin_ authorization. It does not (by itself) prevent sophisticated path manipulation attacks, nor does it solve the problem of an AS that legitimately holds a ROA but then leaks or hijacks routes further downstream.

### Operational Defenses

Network operators use a variety of pragmatic techniques:

- Prefix filtering: Explicitly configure which prefixes you will accept from each neighbor (customer cone, peer, etc.).
- Max-prefix limits: Automatically tear down a BGP session if a neighbor suddenly advertises far more prefixes than expected (a common sign of misconfiguration or attack).
- Internet Routing Registries (IRR): Historical databases of routing policy; many operators still filter based on IRR objects, though accuracy varies.
- Monitoring and alerting: Tools such as BGPStream, BGPmon, and commercial services detect anomalous announcements in near real time and notify operators so they can manually intervene.

These techniques raise the bar but cannot eliminate the problem.

When an operator detects (or is alerted to) a suspicious announcement, the typical response is manual and social: they contact the upstream provider that is leaking the route, ask them to filter it, or apply a temporary filter themselves using prefix lists or BGP communities. In serious cases they may "shut down" the BGP session entirely until the problem is resolved. There is no global "undo" button; every affected network must be contacted or must act on its own. This is why even well-intentioned misconfigurations can take hours to fully resolve.

### Defense in Depth: Higher-Layer Protections

Because the routing layer cannot fully protect itself, security engineers rely heavily on protocols above IP:

- TCP provides reliable, ordered delivery. A simple blackhole attack will cause TCP connections to time out and retransmit, giving applications a clear signal that something is wrong (even if they cannot diagnose the root cause).
- TLS (and other cryptographic protocols) provide confidentiality, integrity, and strong server authentication. A passive interceptor learns nothing about the content of a properly protected TLS session. Active modification is detected.

However, higher layers are not a complete solution:

- Metadata leakage: Even with perfect encryption, an observer on the path sees source and destination IP addresses, port numbers, packet sizes, and timing. This is often enough to identify which websites are being visited (website fingerprinting), which organizations are communicating, or when sensitive activities are occurring.
- Harvest-Now-Decrypt-Later (HNDL): A sophisticated adversary can record encrypted traffic today in the hope that future cryptanalysis, quantum computers, or stolen private keys will allow decryption years later. A BGP hijack that diverts traffic through a nation-state's infrastructure makes this attack easier.
- Data sovereignty and legal risk: Many jurisdictions require that certain categories of data (health records, financial data, government secrets, citizen communications) never leave the country's physical borders. A BGP hijack that silently routes such traffic through another country can create immediate legal and regulatory violations, even if the data remains encrypted.

## Outlook

BGP route hijacking is one of the clearest illustrations of a recurring theme in network security: protocols designed under assumptions of honesty and cooperation become dangerous when those assumptions no longer hold. The global routing system has no single owner, no central policy enforcement point, and strong economic and operational incentives to keep the network running even when some participants misbehave.

Progress is real. RPKI deployment, better monitoring tools, and increased operator awareness have all improved the situation since the 2008 YouTube incident. Yet the fundamental mismatch between the protocol's trust model and today's threat environment remains. As long as the Internet carries high-value traffic, attackers will have incentives to exploit the routing layer.
