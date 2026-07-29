---
title: DHCP
parent: Network Security
nav_order: 5
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# DHCP

## Cheat sheet

- **Layer**: Application (7) (over UDP). It uses Layer 2 broadcasts during initial configuration (when the client has no IP address). Its main job is to provide Layer 3 network configuration (IP address, subnet mask, default gateway, and DNS servers). So its primary security impact occurs at Layers 2 and 3.
- **Purpose**: Automatically provide Layer 3 network configuration (IP address, subnet mask, default gateway, and DNS servers) to devices joining the network
- **Vulnerability**: Broadcast protocol with no authentication. On-path attackers can see DHCP requests and win the race to send malicious responses
- **Result**: Attacker becomes a man-in-the-middle by supplying a rogue gateway and/or rogue DNS server
- **Defenses**: Higher-layer encryption (TLS), DHCP snooping on switches, port security, and static IP configuration for critical devices

---

## Background: DHCP and Network Configuration

Before a device can communicate on a network, it needs several pieces of information. When you connect a laptop or phone to new Wi-Fi, or when a server boots up on a wired network, it does not magically know how to send and receive traffic. It needs to be _configured_. At minimum, a device needs three critical pieces of information:

- _An IP address_, so other devices on the network (and on the Internet) can send packets to it.
- _The IP address of the gateway (router)_, so the device knows where to send packets that are destined for other networks or the Internet.
- _The IP address of a DNS server_, so the device can translate human-readable names (such as `www.google.com`) into IP addresses.

Without these three pieces of information, a device is effectively isolated. It cannot be reached by others, and it cannot reach the rest of the network or the Internet. In small or static environments, an administrator can manually configure these values on every device. However, this approach does not scale. In a university, a company, a coffee shop, or a home with many devices, manually assigning and tracking IP addresses quickly becomes impractical. This is why **Dynamic Host Configuration Protocol (DHCP)** was created.

DHCP allows a device to automatically obtain the configuration it needs when it first joins a network. The device simply broadcasts a request for help, and a local DHCP server responds with the necessary settings. Depending on your environment, this server lives and runs in different places. At home, the DHCP server is a built-in software service running directly on your wireless router, pre-configured by the manufacturer or your ISP to work straight out of the box. In small public settings such as coffee shops or restaurants, the DHCP server is often still running on a consumer or prosumer-grade router, though some locations use simple commercial access points or cloud-managed gateways. In large enterprise or campus environments, such as a university, the DHCP server typically runs as dedicated software on centralized servers managed by the IT department, because consumer hardware cannot reliably handle thousands of users.

The configuration a device receives determines where its traffic goes. If an attacker can influence this configuration, they can redirect traffic, intercept it, or manipulate name resolution. This lack of trust during the initial configuration phase is exactly what attackers exploit.

## How DHCP Works

DHCP follows a four-step process commonly known as _DORA_ (Discover, Offer, Request, Acknowledge). This handshake allows a client to obtain an IP address and other network configuration from a DHCP server.

### The DORA Handshake

1. **Discover**  
   When a device first connects to a network and lacks an IP address, it broadcasts a DHCP _Discover_ message to locate available servers. Because the device does not yet have a network identity, this message is sent to the broadcast address at both Layer 3 (using the destination IP `255.255.255.255`) and Layer 2 (using the destination MAC address `FF:FF:FF:FF:FF:FF`). Any DHCP server on the local network can hear this request.

2. **Offer**  
   One or more DHCP servers respond with a DHCP _Offer_ message. This message contains a proposed IP address (called a _lease_), the subnet mask, the gateway (router) address, DNS server addresses, and other configuration options. The offer also includes a lease duration (the amount of time the client is allowed to use the configuration).

3. **Request**  
   The client chooses one of the offers it received and broadcasts a DHCP _Request_ message. This broadcast informs all DHCP servers which offer the client accepted. Servers whose offers were not chosen can then withdraw their offers and make those addresses available to other clients.

4. **Acknowledge**  
   The chosen DHCP server sends a final DHCP _Acknowledge_ message confirming the configuration. At this point, the client can begin using the IP address, gateway, and DNS settings it received.

After the handshake completes, the client is fully configured and can communicate on the network.

Both the _Discover_ and _Request_ messages are sent as broadcasts. This is necessary because the client does not yet know the IP address of any DHCP server. Broadcasting allows the client to reach any available DHCP server on the local network without prior configuration. This design makes DHCP very convenient, but it also creates a significant security weakness, which we will examine in this topic.

You can examine real DHCP traffic yourself using Wireshark. A sample DHCP capture is available on the Wireshark wiki:
[https://wiki.wireshark.org/DHCP](https://wiki.wireshark.org/DHCP)

## The Attack: DHCP Spoofing

Just like ARP, DHCP was designed with no authentication. Any device on the local network can send DHCP messages, and clients have no reliable way to verify that a DHCP server is legitimate. This makes DHCP vulnerable to spoofing attacks.

An attacker on the same local network can run a malicious DHCP server (often called a _rogue DHCP server_). The attack works as follows:

When a new device connects to the network and broadcasts a DHCP _Discover_ message, the attacker sees it. The attacker then sends a forged DHCP _Offer_ containing malicious configuration settings. If the attacker’s forged offer arrives before the legitimate DHCP server’s offer, the client will usually accept it.

This is a _race condition_, exactly like the one we saw in ARP spoofing. The attacker does not need to block the legitimate server. They only need to reply faster.

### How the Attacker Becomes a Man-in-the-Middle

The attacker can manipulate two especially powerful settings in the DHCP offer:

- Gateway (Router) Address: The attacker can set their own IP address as the default gateway. All traffic from the victim that is destined for the Internet will be sent to the attacker first. The attacker can then forward the traffic to the real gateway, becoming a classic man-in-the-middle.
- DNS Server Address: The attacker can provide their own DNS server. This allows them to return incorrect IP addresses for domain names. For example, when the victim tries to visit their bank’s website, the attacker’s DNS server can direct them to a phishing site instead.

Before the attack, the adversary is an _on-path attacker_. They can observe DHCP traffic on the local network. After successfully spoofing the DHCP offer, they become an _in-path attacker_ (a full man-in-the-middle).

The attack is particularly dangerous on open or weakly protected networks such as public Wi-Fi, university networks, or conference networks, where anyone can join the local network and run their own DHCP server.

Once the victim accepts the malicious configuration, the attacker is in a powerful position. They can:

- **Credential Theft**: The attacker can intercept usernames, passwords, and session cookies sent over unencrypted protocols (such as plain HTTP, FTP, or Telnet). Even some older or misconfigured applications may still send sensitive information in cleartext.

- **Phishing and DNS Redirection**: By controlling the DNS server address, the attacker can redirect victims to malicious websites. For example, when a user types their bank’s website, they may be sent to a fake login page that looks legitimate. This type of attack is difficult for users to detect because the address bar may show the correct domain name (depending on how the attack is executed).

- **Session Hijacking**: The attacker can observe and potentially take over active sessions, especially for protocols that do not properly implement encryption or authentication.

- **Malware Distribution**: The attacker can inject malicious content into web pages or serve malware when the victim downloads files.

- **Denial of Service**: The attacker can simply drop traffic or provide invalid configuration, preventing the victim from using the network properly.

## Defenses Against DHCP Attacks

Defending against DHCP attacks is difficult because the protocol was designed for convenience and automatic configuration, not security. DHCP has no built-in authentication, so a client cannot easily verify whether a DHCP server is legitimate. While we cannot fully prevent these attacks at the protocol level, several practical defenses can significantly reduce their impact.

### Higher-Layer Protections (The Most Important Defense)

The most effective defense is to use secure protocols at higher layers. When applications use TLS (which we will cover in detail later), SSH, or VPNs, traffic is encrypted end-to-end between your device and the destination server. Even if an attacker becomes a man-in-the-middle through a rogue DHCP server, they cannot read or modify your data. However, this protection only works if cryptographic validation is properly enforced. Users should never ignore browser certificate warnings or unexpected SSH host key alerts.

Note that SSH uses a "Trust on First Use" model. It can only protect the session if the very first connection was not intercepted by an attacker.

### Network-Level Defenses

Network administrators can implement several controls to make DHCP spoofing more difficult:

- **DHCP Snooping**: Many managed switches support DHCP snooping. This feature allows the switch to distinguish between trusted and untrusted ports. DHCP messages from servers are only accepted on trusted ports (typically the uplink port connected to the legitimate DHCP server). Messages from untrusted ports are dropped. This is one of the most effective defenses against rogue DHCP servers. However, DHCP snooping is an enterprise feature found almost exclusively on managed switches. It is rarely available on consumer home routers or basic access points.

- **Port Security and MAC Address Limiting**: Switches can be configured to limit the number of MAC addresses allowed on a port or to only permit specific MAC addresses. While not a complete solution, this makes it harder for an attacker to operate freely on the network.

- **802.1X Network Access Control**: 802.1X is a network authentication standard that requires devices to prove their identity (for example, with a username and password or a digital certificate) before they are allowed to connect to the network. This makes it much harder for an attacker to introduce a rogue DHCP server or other unauthorized devices.

- **VLAN Segmentation**: Separating different types of devices or users into different VLANs can limit the scope of an attack. For example, an attacker connected to a guest VLAN would not be able to affect devices on staff or server VLANs.

Most of the network-level defenses described above require managed switches and proper configuration. They are often unavailable on simple home routers, public Wi-Fi networks, or low-cost network equipment. In many real-world environments, especially open or guest networks, these protections are not present.

### Static Configuration

For critical systems (such as servers or important workstations), administrators can use static IP configuration instead of DHCP. This removes the device’s reliance on DHCP and eliminates the risk of receiving malicious configuration. However, this approach does not scale well for large numbers of client devices.

## Note: DHCP Spoofing vs. Evil Twin Attacks

It is important to distinguish between a _rogue DHCP server_ attack and an _Evil Twin_ attack, as both are common on wireless networks but operate at different layers. _DHCP spoofing_ occurs _after_ a device has already associated with a Wi-Fi network (or connected to a wired network). The attacker runs a malicious DHCP server on the same local network and attempts to respond faster than the legitimate DHCP server. This allows them to supply a rogue gateway or rogue DNS server.

In contrast, an _Evil Twin_ attack happens at the wireless association layer (Layer 2). The attacker sets up a fraudulent Wi-Fi access point that broadcasts the same network name (SSID) as a legitimate network (for example, “CoffeeShop_Free_WiFi”). Because many devices automatically connect to known or stronger-looking networks, a victim may associate with the attacker’s access point instead of the real one. Once associated, the attacker is immediately in a man-in-the-middle position and can also run a rogue DHCP server on their fake access point. While the two attacks are technically different, a successful Evil Twin attack often removes the need for DHCP spoofing against the legitimate network. Because the victim connects directly to the attacker’s fake access point, the attacker can simply run their own DHCP server on that access point. Attackers often combine Evil Twin attacks with deauthentication attacks. By sending fake deauthentication frames, they can forcefully disconnect devices from the legitimate network, increasing the chance that victims will connect to the attacker’s fake access point instead.
