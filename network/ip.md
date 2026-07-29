---
title: IP Address Spoofing and Network Scanning
parent: Network Security
nav_order: 6
layout: page
header-includes:
  - \pagenumbering{gobble}
---

## Cheat sheet

- **Layer**: Network (Layer 3)
- **Purpose**: Provide global addressing and routing of packets across the Internet
- **Main Problems**:
  - **IP Address Spoofing**: Any host can forge the source IP address in a packet. Routers generally do not validate whether the source IP is legitimate.
  - **Network Scanning**: Attackers can systematically discover live hosts, open ports, and running services on a network.
- **Core Weakness**: The source IP address in a packet is not authenticated. It is set by the sender and can be easily forged. There is also no built-in mechanism to prevent reconnaissance scanning.
- **Attack Result**:
  - Spoofing enables attacks such as blind spoofing, reflection/amplification DDoS, and bypassing IP-based access controls.
  - Scanning allows attackers to map networks and identify targets before launching further attacks.
- **Defenses**:
  - Ingress filtering (BCP 38) at network edges to reduce spoofing from customer networks.
  - Detection systems, rate limiting, and darknets/honeypots to detect scanning activity.
  - Stronger authentication at higher layers (e.g., TLS) instead of relying on IP addresses.

---

## IP Packets and Best-Effort Delivery

The **Internet Protocol (IP)** operates at the Network Layer (Layer 3) and is responsible for routing packets from one host to another across the global Internet. Every device on the Internet is identified by an IP address, which serves two main purposes:

- The _destination IP address_ tells routers where to forward the packet.
- The _source IP address_ indicates where the packet came from (so the recipient can reply).

When a router receives a packet, it makes its forwarding decision based on the destination IP address. In general, routers do not validate whether the source IP address is legitimate.

Note that the only router that has a reasonable chance of performing source IP validation is the first-hop router (typically your ISP’s edge router). This is because your ISP knows which IP addresses it has assigned to you. It can check whether the source IP in the packet matches the IP addresses it expects from your connection. This technique is known as ingress filtering.
However, in practice, many ISPs do not perform strict source IP validation on every packet. Even when they detect spoofing, they often respond by sending a warning notification rather than immediately dropping the traffic. This is partly due to the risk of accidentally blocking legitimate traffic and partly due to operational costs. As a result, spoofed packets frequently pass through the network without being blocked at the source.

However, once a packet leaves your ISP’s network and enters the core of the Internet, other routers have no reliable way to determine whether the source IP address is authentic. They do not know which network originally originated the packet, and routing on the Internet is often asymmetric. As a result, forged packets are usually forwarded without issue once they pass the first hop.

### IP Provides “Best-Effort” Delivery

IP was deliberately designed as a _best-effort_ protocol. This means it makes no guarantees about:

- Whether a packet will be delivered
- Whether packets will arrive in the correct order
- Whether packets will arrive without corruption

If a packet is lost, corrupted, or arrives out of order, IP does not attempt to recover it. These responsibilities are left to higher layers (such as TCP).

From a security perspective, this design has an important consequence: **IP provides no authentication or integrity protection**. There is nothing in the protocol that prevents a sender from putting a false source IP address in a packet, and there is no built-in mechanism for routers to verify whether the claimed source address is legitimate.

## IP Address Spoofing

**IP address spoofing** is the act of forging the source IP address in a packet. In other words, a sender can put any IP address they want in the source field of an IP packet, rather than their actual IP address.

When a normal application sends a packet, the operating system automatically fills in the correct source IP address. However, if an attacker uses a _raw socket_, they can bypass the operating system’s normal packet creation process and manually craft the entire IP header themselves. This allows them to set the source IP address to any value they choose.

Raw sockets are a privileged operation on most systems and typically require root or administrator access. Once an attacker has this level of access, forging the source IP address becomes trivial.

When a router receives a packet, it generally looks only at the _destination IP address_ to decide where to forward it. Routers do not, by default, validate whether the source IP address is legitimate or whether it matches the actual origin of the packet. As a result, spoofed packets are often forwarded without any issue.

There are two main categories of IP spoofing attacks, depending on the attacker’s position:

- _Blind Spoofing_: The attacker cannot see the replies to the spoofed packets. This type of spoofing is useful when the attacker does not need a response (for example, in certain Denial-of-Service attacks or when injecting packets into a connection).
- _On-Path Spoofing_: The attacker is positioned on the path between the sender and receiver (or can observe traffic). In this case, the attacker can see the replies and therefore has much more power. They can, for example, hijack connections or perform more sophisticated attacks.

Because the source IP address can be easily forged, it cannot be trusted for security purposes. Many systems and firewalls in the past attempted to use source IP addresses for access control (for example, “only allow connections from this specific IP address”). IP spoofing shows why such approaches are fundamentally unreliable.

## Network Scanning and Reconnaissance

Before attackers can exploit a target, they usually need to discover what hosts and services exist on a network. This process is called **network scanning** (or reconnaissance scanning). It is one of the most common first steps in network attacks.

Network scanning refers to the systematic probing of a network to discover:

- Which IP addresses are currently in use (live hosts)
- Which ports are open on those hosts
- What services and operating systems are running
- Potential vulnerabilities

Attackers use this information to identify promising targets and choose appropriate follow-up attacks.

### The Role of `nmap`

The most widely used tool for network scanning is **nmap** (Network Mapper). It is a powerful, free, and open-source tool that can perform a wide variety of scans, including:

- Ping sweeps — to find which hosts are alive
- TCP SYN scans (also called half-open scans) — fast and relatively stealthy
- TCP Connect scans — more reliable but easier to detect
- UDP scans — to discover UDP-based services
- Service and OS fingerprinting — to identify specific software versions

Because `nmap` is so effective and easy to use, it has become the standard tool for both legitimate security professionals and attackers.

Network scanning is problematic for several reasons:

- It allows attackers to _map_ a network from the outside without needing any prior access.
- It reveals information that organizations often prefer to keep private (such as which services are exposed to the Internet).
- Many scanning techniques can be performed _passively or with spoofed source IPs_, making the scanner harder to trace.
- Scanning is often the _first step_ before more targeted attacks such as exploitation, password guessing, or denial-of-service.

Even if a network has strong protections at higher layers, simply being discoverable can increase its risk of being targeted.

A well-known real-world example of large-scale scanning is the Mirai botnet (2016). Mirai systematically scanned large portions of the IPv4 address space looking for vulnerable Internet of Things (IoT) devices with weak or default credentials. Once it compromised these devices, it turned them into bots that could be used for massive distributed denial-of-service attacks. This case demonstrated how automated, high-speed scanning across the entire Internet can quickly build extremely large botnets.

### Privacy and Legal Considerations

Running network scans raises important ethical, privacy, and legal issues:

- Scanning a network without permission can be considered a violation of privacy expectations.
- In many jurisdictions, unauthorized scanning may be illegal under computer crime laws, even if no further damage is caused.
- Some organizations and internet service providers actively monitor for scanning activity and may block or report scanners.
- Security professionals are generally expected to obtain explicit written permission (such as a Rules of Engagement document) before performing any scanning activity on a network they do not own.

For these reasons, network scanning should always be approached with caution and responsibility.

## Why IP-Based Defenses Are Weak

A common intuition in network security is that we can protect systems by controlling access based on IP addresses — for example, allowing connections only from specific trusted IP addresses or blocking traffic from suspicious ones. Unfortunately, both _IP spoofing_ and _network scanning_ make this approach fundamentally unreliable.

Because anyone can forge the source IP address in a packet, the claimed origin of a packet is not reliable. An attacker can:

- Send packets that appear to come from a trusted IP address.
- Bypass simple IP-based access control lists (ACLs).
- Launch attacks while hiding their true origin (especially in blind spoofing scenarios).

This means that any security mechanism that makes decisions solely based on the source IP address in a packet can be circumvented. Firewalls, servers, and applications that trust incoming connections simply because they “come from the right IP” are vulnerable to spoofing attacks.

### Network Scanning Makes Hiding Difficult

Even if an organization tries to limit exposure by not advertising certain services or by using non-standard ports, network scanning makes discovery relatively easy. Tools like `nmap` can quickly identify:

- Which hosts are alive on a network
- Which ports are open
- What services are running

This reconnaissance can be performed from the public Internet, often without the target organization’s knowledge. As a result, simply relying on “security through obscurity” or assuming that attackers will not find your systems is not a strong defense.

### The Combined Problem

When we consider spoofing and scanning together, the limitations of IP-based security become even clearer:

- An attacker can first use scanning to discover targets.
- They can then use spoofing to send malicious packets that appear to come from legitimate or trusted sources.
- Defenses that depend on knowing “who is sending the traffic” based on IP addresses become ineffective.

This is why security professionals generally do not rely on IP addresses alone for authentication or access control. Instead, stronger mechanisms — such as cryptographic authentication at higher layers — are needed.

## Limited Defenses and Mitigations

Defending against IP spoofing and network scanning at the Network Layer is challenging. The Internet was designed for reachability and scalability, not for preventing reconnaissance or validating packet origins. Nevertheless, some partial defenses exist.

### Defenses Against IP Spoofing

The most effective defense against IP spoofing is _ingress filtering_, also known as _BCP 38_.

Ingress filtering works by configuring routers at the edge of a network (typically at an ISP) to reject packets that claim to come from IP addresses outside the range assigned to that customer. For example, if a customer is assigned the prefix `203.0.113.0/24`, the ISP’s router should drop any packet leaving that customer’s network that has a source IP address outside this range.

When properly implemented, ingress filtering makes it much harder for attackers inside a customer network to spoof arbitrary IP addresses. However, enforcing ingress filtering between Autonomous Systems is more difficult. There is no universal mandate, and some networks still do not implement it. As a result, spoofing remains possible, especially from networks that do not perform filtering.

### Defenses Against Network Scanning

Completely preventing network scanning is very difficult because the Internet is designed to allow hosts to communicate with each other. However, organizations can use several techniques to make scanning less effective or easier to detect:

- **Rate limiting and throttling** on responses to probes.
- **Intrusion Detection Systems (IDS)** and **Intrusion Prevention Systems (IPS)** that monitor for scanning patterns.
- **Darknets and honeypots** — unused IP address space that is monitored for unexpected traffic. Any connection attempt to these addresses is likely to be malicious or scanning activity.
- **Firewall rules** that limit exposure of unnecessary services to the public Internet.

While these measures can help, determined attackers can still perform reconnaissance using slow, distributed, or spoofed scans.

### The Fundamental Limitation

Both spoofing and scanning highlight a core limitation of the Network Layer: it was not designed to provide strong authentication or to hide the existence of hosts and services. Any defense that relies solely on IP addresses or on the assumption that attackers cannot discover systems will eventually fail.

This is why security ultimately depends on **higher-layer protections**. Even if an attacker successfully spoofs an IP address or discovers a host through scanning, properly implemented cryptographic protocols (such as TLS) can still protect the confidentiality and integrity of the actual communication.
