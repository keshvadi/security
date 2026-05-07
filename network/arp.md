---
title: ARP Spoofing
parent: Network Security
nav_order: 1
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Wired Local Networks: ARP Spoofing

## Cheat sheet

- **Layer**: Link (2)
- **Purpose**: Translate IP addresses into MAC addresses on a local network
- **Vulnerability**: On-path attackers can see ARP requests and send spoofed replies faster than the legitimate response (race condition)
- **Result**: Attacker becomes a man-in-the-middle and can read, modify, or inject traffic
- **Defenses**: Switches (instead of hubs), port security, arpwatch, VLANs, static ARP entries, and higher-layer encryption (TLS)

---

## 1. Networking Background: Ethernet, MAC Addresses, and Broadcast Networks

Before we can understand ARP attacks, we need to revisit a few concepts from Topic 1 with a security lens.

On a **local area network (LAN)**, computers are connected so they can communicate directly with each other. The most common wired technology for LANs is **Ethernet**. In Ethernet, every device has a unique **MAC address** (a 6-byte identifier usually written as six pairs of hex digits, e.g., `ca:fe:f0:0d:be:ef`).

**Critical security fact**: MAC addresses are **self-reported**. Every Ethernet frame contains the sender’s MAC address in the header, but the network has no way to verify that the claimed MAC address is correct. A device can simply lie. This is why you should **never** use MAC addresses for authentication or access control.

Early Ethernet networks used **hubs**, simple devices that broadcast every frame to *everyone* on the LAN. Even modern Wi-Fi networks behave like this by default. Any device on the network can put its network card into **promiscuous mode** and receive *every* frame, not just the ones addressed to it. This is called **packet sniffing**.

**Sanity check**: If you are on the same LAN as a victim (for example, the same coffee-shop Wi-Fi or university dorm network), what kind of adversary are you?  
**Answer**: You are an **on-path adversary**. you can see all traffic and you can send any traffic you want with a spoofed MAC address.

This broadcast nature of local networks is exactly why ARP is so easy to attack.

## 2. The ARP Protocol

**ARP** (Address Resolution Protocol) solves a simple but essential problem: given an IP address, what is the corresponding MAC address?

When Alice wants to send a packet to Bob on the same LAN and she only knows Bob’s IP address (`10.0.0.12`), she uses ARP:

1. Alice **broadcasts** to everyone on the LAN:  
   “Who has IP address 10.0.0.12? Tell ca:fe:f0:0d:be:ef (my MAC).”

2. Bob (the owner of 10.0.0.12) replies **directly to Alice**:  
   “10.0.0.12 is at ca:fe:f0:0d:be:ef.”

3. Alice **caches** this mapping (IP → MAC) so she doesn’t have to ask again for a while.

If Bob is outside the LAN, the router replies instead with *its own* MAC address.

**Important detail**: ARP is completely trust-based. Any ARP reply is accepted and cached, even if it was never requested. There is no authentication, no encryption, and no way to verify the sender.

## 3. The Attack: ARP Spoofing (Race Condition)

Because ARP trusts every reply, an attacker can easily poison the cache.

**Attack scenario**:
- Alice wants to talk to Bob (IP 10.0.0.12).
- Mallory (the attacker) is on the same LAN and can see the ARP request.
- Mallory immediately sends a **spoofed ARP reply** claiming:  
  “10.0.0.12 is at my MAC address (de:ad:be:ef:00:01).”

If Mallory’s spoofed reply arrives **before** Bob’s legitimate reply, Alice updates her ARP cache with the malicious mapping. 
Even if Bob’s legitimate reply arrives later, most systems will **overwrite** the existing cache entry with Bob’s updated MAC address (ARP is stateless and updates on any new reply). However, because Alice now has a cached entry, she will **not** send another ARP request right away. To maintain the attack, Mallory must keep sending spoofed ARP replies (often called *gratuitous ARPs*) at regular intervals to re-poison Alice’s cache. From that moment on, every packet Alice sends to Bob is actually delivered to Mallory’s MAC address.

Mallory is now a **man-in-the-middle**. She can:
- Read all traffic between Alice and Bob
- Modify packets before forwarding them
- Inject new packets
- Drop packets (denial of service)

**This is a classic race condition**. The attacker wins simply by being faster. On-path attackers cannot block the legitimate reply, so they must race to reply first. This pattern appears again in DHCP attacks (next section).

**Sanity check**: After a successful ARP spoofing attack, what type of adversary has Mallory become?  
**Answer**: A full **in-path (man-in-the-middle)** adversary.

## 4. Impact and Real-World Consequences

Once an attacker is a man-in-the-middle via ARP spoofing, the damage can be severe:

- Steal login credentials (HTTP, FTP, Telnet, etc.)
- Perform session hijacking
- Inject malware or malicious JavaScript
- Redirect traffic to phishing sites
- Capture sensitive files or emails

ARP spoofing is especially dangerous on open or weakly secured networks (coffee shops, hotels, university networks, conference Wi-Fi) where anyone can join the LAN.

## 5. Defenses Against ARP Spoofing

There is no perfect defense at Layer 2, but several practical mitigations exist:

**Switches instead of hubs**  
Modern switches learn which MAC address is on which port and only forward frames to the correct port. This dramatically reduces the effectiveness of promiscuous-mode sniffing.

**Port security / MAC address filtering**  
Many enterprise switches can be configured to allow only one (or a specific) MAC address per port. This makes MAC spoofing much harder.

**Monitoring tools**  
- `arpwatch` logs all ARP activity and alerts on suspicious changes.
- Similar tools exist for Windows and macOS.

**VLANs (Virtual LANs)**  
Higher-end switches can segment the network into separate virtual networks. Even if an attacker joins one VLAN, they cannot easily attack devices on another VLAN.

**Static ARP entries** (for critical systems)  
On important servers or routers, you can manually configure permanent IP → MAC mappings. This defeats ARP spoofing for those specific devices.

**Higher-layer protections (the ultimate safety net)**  
Even if an attacker becomes a man-in-the-middle at Layer 2, properly implemented **TLS** (with certificate validation) prevents them from reading or modifying application data. This is why HTTPS, SSH, and VPNs remain effective even on compromised networks.

ARP spoofing is the first concrete example in this unit of how an **on-path attacker** can become a full man-in-the-middle. It perfectly illustrates two recurring themes in network security:

- Many protocols were designed with **no authentication** at the lower layers.
- **Race conditions** are a powerful attack technique when the attacker cannot block legitimate messages.

In the next section we will see almost the exact same attack pattern used against **DHCP**, another critical protocol that runs when a device first joins a network.

