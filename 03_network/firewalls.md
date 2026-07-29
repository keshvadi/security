---
title: Firewalls
parent: Network Security
nav_order: 13
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Firewalls

## Cheat sheet

- **Core Concept**: A firewall is a network choke point that enforces an _access control policy_ between a trusted internal network and the untrusted external Internet (or between sensitive internal segments).
- **Default-Deny (Whitelist) vs. Default-Allow (Blacklist)**: Default-deny starts by blocking everything and explicitly permits only the services you need. It _fails closed_ (mistakes cause loss of functionality, not breaches) and is strongly preferred. Default-allow _fails open_.
- **Packet Filters**: The basic building block.
  - **Stateless**: Examines each packet's headers (source/dest IP, ports, protocol) independently against an ordered list of rules. First match wins (ALLOW or DROP).
  - **Stateful**: Tracks active connections. Automatically permits return traffic for allowed outbound connections and can handle complex protocols (e.g., FTP data channels).
- **Major Strengths**: Central administration (one place to update policy for thousands of machines), easy deployment (transparent to most internal hosts), orthogonal protection for legacy systems.
- **Major Weaknesses**: The "crunchy outer coating with a soft, chewy center" (once an attacker is inside the perimeter, the firewall provides no help); loss of functionality; tunneling over allowed ports; inability to inspect encrypted traffic or stop application-layer attacks; high risk of subtle misconfiguration.
- **Circumvention**: Run anything over allowed ports (e.g., BitTorrent over UDP/53 DNS), use external relays, or tunnel inside HTTPS. Encryption hides intent.

---

## Motivation: Controlling Network Access at Scale

Suppose you are given a single machine and asked to harden it against external attacks. A good starting point is to examine every network service it offers to the outside world. Every piece of code that listens on the network is a potential entry point. Bugs are inevitable; bugs in network-facing code are especially dangerous because anyone on the Internet can reach them. The principle is simple: _code you do not run cannot be exploited against you_. Therefore, the first and most effective step for a single machine is to disable every unnecessary network service and to secure carefully the ones that remain.

This approach works reasonably well when you have only a handful of machines. But now imagine you are responsible for the entire TRU University network—thousands of computers, dozens of departments, users with wildly different requirements, multiple operating systems, and machines that you may not even know exist. Auditing and hardening every single host individually is impractical. You will inevitably miss machines, and any missed machine becomes a beachhead from which an attacker can pivot to the rest of the campus.

The observation that still holds at this scale is that _the more network services are reachable from the outside, the larger the attack surface_. If we cannot perfectly harden every host, perhaps we can reduce the number of services that outsiders can even _attempt_ to reach. This is the fundamental idea behind a **firewall**: a device (or set of devices) placed at a strategic point in the network that blocks unwanted traffic before it ever reaches the internal hosts.

A firewall does not eliminate the need to secure the services you _do_ expose. A vulnerable web server behind a firewall is still vulnerable. What the firewall buys you is leverage: one place where you can control access for thousands of machines at once, and a single point where you can react quickly when a new threat appears.

## Security Policies: Deciding Who May Talk to What

Before you can build or configure a firewall, you must have a **security policy** that answers two questions:

1. Which network services should be visible to the outside world, and which should be blocked?
2. How do we distinguish "insiders" from "outsiders"?

### Inbound versus Outbound Connections

An **inbound connection** is initiated by an external party and attempts to reach a service running on an internal machine (for example, someone on the Internet trying to load TRU's public website).

An **outbound connection** is initiated by an internal user or machine and reaches out to a service on the external Internet (a student loading a page from Wikipedia).

Many threat models treat inbound connections as riskier: internal users have usually authenticated themselves (by logging into a machine or by physical presence on campus), while external users could be anyone. A simple but often too-restrictive policy is therefore "outbound only": permit all outbound connections, deny all inbound. This makes no internal services visible to the world, but it also prevents TRU from running any public web server, mail server, or other externally useful service.

A more realistic policy for an organization is: _internal users may connect to any service they like; external users may connect only to a carefully chosen list of public-facing services_.

### Default-Deny versus Default-Allow

Once you have identified the services that should be externally reachable, you still need a rule for everything else. There are two fundamental philosophies:

- **Default-allow (blacklist)**: Everything is permitted unless it has been explicitly forbidden. You start with a wide-open policy and add "deny" rules as you discover problems.
- **Default-deny (whitelist)**: Everything is forbidden unless it has been explicitly permitted. You start with a "deny everything" rule and add narrow "allow" exceptions only for services that have been reviewed and approved.

**Default-deny is almost always the safer choice.** It _fails closed_: if you forget to allow a safe service, users complain and you fix it. The error produces a loss of functionality, not a security breach. Default-allow _fails open_: if you forget to block a dangerous service, the breach may go unnoticed for a long time because attackers have no incentive to tell you they succeeded.

At the scale of a university or company, errors of omission are inevitable. Default-deny turns those errors into visible complaints rather than silent compromises. It also forces a more deliberate risk-assessment process: every service that appears on the "allowed" list must justify its presence.

Sanity check: Suppose TRU accidentally leaves an old, unpatched FTP server reachable from the Internet under a default-allow policy. What happens? Under a default-deny policy? Which situation is easier for the security team to discover and remediate?

Answer: Under default-allow the server is reachable; an attacker can exploit it quietly and the team may never notice until data is stolen or the machine is used to attack others. Under default-deny the service is blocked; legitimate users who need it will complain loudly and quickly. The complaint is the signal that lets the team investigate whether the service is truly needed and whether it can be made safe.

Services are identified by the triplet (machine IP address, protocol TCP or UDP, port number). A typical allowed entry might be "anyone may open a TCP connection to port 443 on the public web server at 1.2.3.4."

## Packet Filters: The Basic Enforcement Tool

The power of a firewall comes from its position at a _choke point_. Just as airport security is far more effective when every passenger must pass through a small number of checkpoints, network security is far more effective when all traffic between the internal network and the outside world must pass through one (or a few) devices that can inspect and filter it.

A **packet filter** is a router that has been augmented with an access-control list. For every packet that arrives, the filter compares the packet against its rules in order. The first rule that matches determines the fate of the packet: forward it toward its destination, or drop it silently.

### Stateless Packet Filters

A stateless packet filter looks only at the current packet's headers:

- Source IP address and port
- Destination IP address and port
- Protocol (TCP, UDP, ICMP, etc.)
- Sometimes TCP flags (SYN, ACK, etc.)

Rules use wildcards (`*`) for "any value." Classic examples (in a simplified syntax):

```bash
allow tcp 1.2.3.4:* -> 10.0.0.1:80
allow tcp : -> 10.0.0.1:443
deny  tcp : -> :
```

The last rule is a catch-all "default deny." Rules are evaluated top to bottom; order matters.

**For readers new to networking**: Think of an IP address as a building address and a port as an apartment number inside that building. Well-known ports (0–1023) are conventionally used by standard services (80/443 for web, 25 for mail, 22 for SSH, 53 for DNS). Client programs usually pick a random high-numbered "ephemeral" port (>1023) for their side of the conversation.

Stateless filters are simple and fast, and they remain the foundation of most real firewalls (Linux `iptables`/`nftables`, Cisco ACLs, etc.). They have, however, important limitations that motivate stateful filtering.

### Limitations of Stateless Filtering

Servers listen on well-known ports. Clients use random high ports for their side of the conversation. If you want to allow a TRU student to visit an external website, you must allow outbound connections to port 80 or 443. The replies will come back to the student's random high port. A purely stateless filter has no memory that this particular high-port conversation was started by an internal host, so the only safe way to permit the reply is to open _all_ high ports to the world. That is obviously dangerous—any program on an internal machine could listen on a high port and receive unsolicited traffic.

Many protocols are even more awkward. The classic example is active-mode FTP:

1. The client connects from a random port (say 2352) to the server's port 21 (control channel) and says "please send the file data to me on port 3573."
2. The server acknowledges.
3. The server then makes a new connection _from its port 20_ to the client's port 3573 (data channel).

A stateless filter that only saw the first packet would have no idea that a later inbound connection from port 20 to 3573 should be allowed. Opening all ports >1023 is the naive (and insecure) workaround.

### Stateful Packet Filters

A **stateful** packet filter maintains a table of currently active connections (or at least enough information to recognize legitimate return traffic). When an internal host is allowed to open an outbound TCP connection, the firewall records the 5-tuple (source IP, source port, dest IP, dest port, protocol) and automatically permits the matching return packets until the connection closes or times out.

For the FTP example above, a stateful firewall that understands the FTP protocol can watch the control channel, see the `PORT` command, and _dynamically_ create a temporary rule that allows the expected data connection from the server. When the transfer finishes, the temporary rule is removed. This is far safer than leaving high ports open all the time.

Stateful inspection also enables richer policies: "do not allow FTP logins with the username root," "rate-limit connections from any single external IP," etc. The cost is memory and CPU; the firewall must track state for every active connection without exhausting its resources. Careful engineering is required.

Modern packet filters (iptables with conntrack, pf, etc.) are stateful by default for most traffic.

Sanity check: Why can a stateful firewall safely allow "any outbound connection" while still protecting internal servers?  
Answer: Because it only permits return traffic that exactly matches a connection that an internal host previously initiated. Unsolicited inbound packets that do not correspond to any tracked connection are dropped.

## Application-Layer Firewalls and Proxies

While stateless and stateful packet filters operate primarily at the network and transport layers (looking at IP addresses and TCP/UDP ports), **application-layer firewalls** restrict traffic based on the actual content of the application data fields.

Rather than simply inspecting packets as they fly by, we can build firewalls that actively participate in application-layer exchanges using an **application proxy** (or gateway proxy). In this design, local systems are configured to connect _to the proxy_ rather than connecting directly to remote servers. The proxy receives the request, analyzes it, and then makes the actual remote request to the server on the user's behalf.

This provides a major security benefit: the proxy has access to the full application-layer context (e.g., the specific URL requested, HTTP headers, or FTP commands) and can make fine-grained allow/deny decisions, or even safely transform data on the fly.

However, application proxies have two significant drawbacks:

- **Implementation Complexity**: The application proxy must deeply understand all of the intricate details of every application protocol that it mediates.
- **Performance Bottlenecks**: Because the proxy acts as an active middleman that terminates and recreates connections, bottlenecking an entire site's outbound traffic through a few proxy systems can easily overwhelm them under heavy load.

## Firewall Architectures and Topologies

Packet-filtering logic (and more advanced controls) can be deployed in several classic arrangements. The progression from simplest to true defense-in-depth is worth understanding in detail; each step adds protection at the cost of complexity. Understanding the trade-offs is essential for designing a real network.

### Bastion Hosts

A **bastion host** is a machine that is directly exposed to the Internet and provides a public service (web, mail, VPN, DNS, etc.). It is the "lobby" of your building: visitors are expected to interact with it, but it should be extremely difficult for them to reach the offices upstairs.

Because it is the most exposed machine, it must be treated with special care:

- Keep the software load minimal. Every daemon is a potential vulnerability.
- Assume it _will_ be compromised. Design the rest of the network on that assumption.
- Disable ordinary user accounts. Normal users should never log into a bastion interactively.
- Learn its normal behavior (CPU, memory, processes, traffic patterns) so anomalies stand out.
- Watch for unexpected reboots or crashes—signs of possible compromise.
- Maintain secure, versioned backups so that after a compromise you can rebuild cleanly and investigate what happened.

### Screening Router

The simplest topology: a single router with packet-filtering rules sits between the Internet and the internal LAN. This is what most home users have (the "firewall" built into their ISP-supplied router or their own Wi-Fi router). It is cheap and simple, but provides no defense in depth. If the router itself is compromised, the entire internal network is exposed.

### Dual-Homed Host

A bastion host is given two network interfaces: one connected to the Internet, one to the internal network. IP forwarding is disabled or heavily mediated by the host's own software. No packet can travel from the Internet to an internal machine without passing through the bastion's application logic. This allows for deep inspection (like the application proxies discussed earlier) but creates a single point of failure and a potential performance bottleneck.

### Screened Host

A screening router (packet filter) is placed in front of the internal network, but the bastion host(s) live on the _internal_ side. The router is configured to forward traffic to the bastion(s) only. Internal users reach external services by going through the bastion (for example, internal mail clients retrieve mail from a bastion mail server that talks to the outside world). This works well when the bastions do not need to handle enormous numbers of simultaneous connections.

### Screened Subnet (DMZ) — The Gold Standard

This is the architecture you will see in almost every serious organization.

- An _exterior (access) router_ sits between the Internet and a special "perimeter" network (the DMZ).
- One or more _bastion hosts_ live on the DMZ.
- An _interior (choke) router_ sits between the DMZ and the true internal LAN.

The exterior router performs relatively light filtering (it mostly protects the perimeter itself). The interior router is stricter: it protects the internal network both from the Internet _and_ from any bastion that has been compromised. Different rule sets apply on the two routers.

Advantages:

- Defense in depth. Compromising a bastion (expected) does not automatically give the attacker free rein inside.
- Limits the damage from a compromised bastion: it can only snoop on or attack traffic that actually crosses the perimeter network.
- Allows different policies for "services we must expose" versus "services internal users may use."

Important rules of thumb (drawn from operational best practices):

- It is fine to use multiple bastion hosts (separation of duties, load balancing, redundancy).
- It is fine to use multiple exterior routers if you have multiple Internet uplinks.
- It is fine, in some cases, to combine the exterior router with a bastion if the filtering role is minimal.
- Do _not_ merge the interior router with a bastion host. Their security roles are different and not complementary.
- Do _not_ deploy multiple interior routers. Internal traffic could be routed through the perimeter (where a compromised bastion could observe it) for performance or other reasons, defeating the purpose of the DMZ.
- Do _not_ create "exceptions" that allow direct inbound connections from the Internet to internal hosts while claiming you have a screened subnet. Such exceptions are a common source of catastrophic breaches.

Sanity check: Why is it especially dangerous to run a second interior router "for performance" even if it has the "right" rules?  
Answer: Because a compromised bastion on the perimeter network can then observe (or tamper with) internal-to-internal traffic that would otherwise never have left the trusted LAN. The whole point of the DMZ is to confine the damage of a perimeter breach; leaking traffic back through the perimeter defeats that isolation.

A properly designed screened subnet with bastions in the DMZ is almost always the appropriate choice for any organization larger than a very small office.

### Personal Firewalls

In addition to perimeter devices, modern operating systems include host-based ("personal") firewalls. These run on the endpoint itself and can enforce rules that are aware of which application is generating the traffic. They are a useful second layer, especially for laptops that travel outside the corporate perimeter, but they do not replace a well-designed network firewall architecture.

## Advantages of Firewalls

- Central control. When a new vulnerability is announced in some widely used service, you can often block access to it (or to the vulnerable versions) at the firewall in minutes, protecting every internal machine without touching them individually.
- Leverage and ease of deployment. One device (or small set of devices) protects thousands of hosts. Adding the firewall is usually transparent to internal applications and users, so adoption can be incremental.
- Orthogonal security. Firewalls protect existing systems without requiring changes to those systems' code. They are one of the few security mechanisms that work well against legacy software.

## Disadvantages and Limitations

- Loss of functionality. Connectivity creates risk; firewalls reduce connectivity. Some applications (especially peer-to-peer, gaming, and certain video-conferencing tools) become painful or impossible when both endpoints are behind restrictive firewalls.
- The malicious-insider and "laptop problem". Firewalls draw a sharp line between "inside" (trusted) and "outside" (untrusted). Once an attacker crosses that line—by compromising a laptop that later returns to the office, by social-engineering an insider, or by exploiting a contractor's machine—the firewall can no longer help. Bill Cheswick famously described firewalled networks as having "a crunchy outer coating with a soft, chewy center."
- Adversarial applications and tunneling. Application developers who want their software to work through firewalls often tunnel their traffic inside HTTP (port 80) or HTTPS (port 443)—ports that almost every firewall must allow. Once the traffic is encrypted, the firewall loses visibility into what is actually being done.
- \*\*No protection against application-layer attacks. If your public web server has a SQL-injection vulnerability, the firewall will happily forward the malicious HTTP requests; it has no idea what the bytes mean at the application level.
- Lack of authentication / IP spoofing. Packet filters decide based on claimed source IP addresses, which can be forged. A determined attacker on the Internet can send packets that appear to come from an internal address (unless ingress filtering is deployed on the upstream links to drop obviously forged packets). This is one reason stateful filters and higher-layer controls matter.
- Misconfiguration risk. Large rule sets are programs. They contain bugs. They require the same engineering discipline (testing, review, version control) that we apply to any other critical software.

## Circumventing Firewalls

Because firewalls ultimately rely on port numbers and IP addresses as proxies for "what application is this and who is allowed to use it," they are inherently circumventable when those proxies are abused.

- Port abuse: Port numbers are conventions, not technical enforcement mechanisms. Nothing prevents a BitTorrent client and server from agreeing to communicate on UDP port 53 (normally DNS) or on TCP port 80 (normally HTTP). If the firewall permits DNS or web traffic, it will permit the disguised traffic as well.
- Relays and proxies: An internal user who can reach some unfiltered host on the Internet can run a small relay program there. The user sends innocuous-looking requests to the relay ("please forward this data to IP:port X"), and the relay forwards the real traffic. Detecting such relays is difficult.
- Encryption and VPNs: End-to-end encryption hides both the content and, to a large extent, the true purpose of the communication. A user who establishes an outbound HTTPS connection to a relay outside can tunnel arbitrary traffic inside it.
- Insider action: The ultimate circumvention is simply to carry a compromised laptop across the physical perimeter.

These realities are why firewalls are only one layer in a defense-in-depth strategy. They must be complemented by host hardening, intrusion detection, logging and monitoring, endpoint protection, and, most importantly, good operational practices and a realistic understanding of the insider threat.
