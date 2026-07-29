---
title: Physical Layer
parent: Network Security
nav_order: 2
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Physical Layer

## Cheat sheet

- **Layer**: Physical (Layer 1)
- **Purpose**: Transmit raw bits over a physical medium using copper cables, fiber optic cables, or radio waves
- **Key Vulnerability**: The Physical Layer has no built-in security. Anyone who can access the physical medium can eavesdrop on or tamper with the raw signals.
- **Main Threat**: Physical eavesdropping (tapping)
  - On wireless: trivial (anyone in range can listen)
  - On wired (copper/fiber): possible with physical access or specialized equipment
- **Attack Result**: Attacker can passively capture all traffic on the link without detection in many cases
- **Defenses**:
  - Encryption at higher layers (TLS, IPsec, WPA)
  - Physical security and access control
  - Detection systems for fiber tapping (limited effectiveness)

---

## The Physical Layer

The Physical Layer (Layer 1) is the lowest layer in the Internet protocol stack. Its job is to transmit raw bits by encoding them into signals that can travel across a physical medium. A physical medium can be wired or wireless.
In wired technologies, this involves voltage levels on copper cables (such as Ethernet over twisted pair) or light pulses in fiber optic cables.
In wireless technologies, such as Wi-Fi, cellular networks, or satellite systems like Starlink, data is transmitted using radio frequency (RF) signals that travel through the air or space.

The Physical Layer operates on individual bits or small groups of bits and has no concept of packets, addresses, or messages as these are added by higher layers. It provides no authentication, confidentiality, or integrity protection. Its only concern is getting bits from one point to another as reliably as possible.
Because it deals with raw physical signals, the Physical Layer is inherently exposed. Anyone who can access or observe the physical medium can potentially see or interfere with the signals being transmitted.

When discussing network security, it is important to distinguish between two very different types of entities that can observe traffic.

_Legitimate on-path routers_ are part of the normal forwarding path. By design, they must inspect the IP header of every packet to determine the next hop and actively modify packets (for example, by decrementing the TTL field). Because traffic is supposed to flow through them, they inherently have the technical ability to inspect, log, mirror, or drop packets. While operators are expected to follow privacy policies, the technical capability exists.

_Physical Layer eavesdroppers_, by contrast, are not part of the legitimate routing path. These attackers obtain a copy of the data by exploiting the physical medium itself either by intercepting wireless signals in the air or by physically tapping into copper or fiber optic cables. In this case, the attacker is an unauthorized third party who should not have access to the communication at all.

## Physical Layer Eavesdropping

One of the most fundamental threats at the Physical Layer is **eavesdropping**, also known as **sniffing**.

To understand how sniffing tools actually capture data, we have to look at how a Network Interface Card (NIC) operates by default. Normally, your network card acts as a strict gatekeeper. When a frame passes by on the wire or through the air, the NIC checks the destination MAC address. If the MAC address does not match your device’s hardware address (or a universal broadcast address), the NIC quietly drops the frame. It never alerts your operating system or your applications that the data existed.

To perform sniffing, tools must put the network card into **promiscuous mode** (or a specialized **monitor mode** when capturing unassociated wireless traffic). In this mode, the NIC disables its address filter and passes _every_ frame it physically receives up to the operating system. This allows tools like Wireshark and `tcpdump` to capture traffic that was never intended for that device.

**Note:** Wireshark enables promiscuous mode by default (when it has the necessary permissions). This is why simply opening Wireshark and starting a capture on an interface often allows you to see traffic not destined for your machine.
Regardless of what you can see, doing this in public carries **massive legal risks**. In many jurisdictions, capturing and analyzing data packets belonging to other people without their explicit authorization is a serious federal crime.

### Wireless Eavesdropping

Wireless networks are especially vulnerable to eavesdropping because data is broadcast indiscriminately into the air as radio waves. Any device with a compatible wireless adapter within range can intercept these transmissions. Tools such as `tcpdump` and Wireshark make it easy to capture and analyze the traffic. This vulnerability applies not only to open, unencrypted networks but also to shared password-protected networks.

In a public environment that uses a single shared password (such as a typical WPA2-PSK network in a café), an attacker who knows the password can decrypt and read the traffic of other connected clients. However, this attack requires precise timing and synchronization, which we will explore in the upcoming topic on wireless network security.

### Wireless Jamming

In addition to eavesdropping, wireless networks are also vulnerable to **jamming** attacks at the Physical Layer.

Jamming is a form of denial-of-service attack in which an attacker transmits interference or noise on the same radio frequency used by the wireless network. This can disrupt or completely block legitimate communication between devices and the access point.

Because wireless communication relies on shared radio spectrum, jamming is relatively easy and inexpensive to perform using simple devices. Unlike eavesdropping, jamming does not require the attacker to know any passwords or encryption keys. The attacker only needs to transmit powerful enough noise on the target frequency.

Jamming attacks highlight another important weakness of the Physical Layer in wireless environments: the medium itself (radio waves) can be deliberately interfered with, making availability a concern in addition to confidentiality.

### Wired Eavesdropping and Cable Tapping

In the early days of Ethernet, networks commonly used hubs. Hubs are simple devices that broadcast every incoming frame to _all_ connected ports. This made eavesdropping trivial as any device on the same hub could see all traffic by simply enabling promiscuous mode.

On modern switched Ethernet networks, traffic is normally directed only to the intended port, making casual sniffing from a normal switch port difficult. However, this protection disappears if an attacker gains direct access to the physical cable itself.

**Cable tapping** is a distinct and more powerful threat. It does not require the attacker to connect to the network logically. Instead, the attacker targets the physical cable while it is in transit — for example, in walls, utility conduits, drop ceilings, or undersea cables. If an attacker can physically access the cable at any point, they can intercept the raw signals. The technique used depends on the medium.

Copper cables transmit data using electrical signals. An attacker can tap them by directly connecting to the copper conductors or by using inductive probes that detect electromagnetic emissions without making physical contact. Fiber optic cables are harder to tap because they use light. However, they can still be compromised using a technique called **micro-bending**.

In the next section, we will examine two real-world examples of these types of physical eavesdropping attacks.

## Operation Ivy Bells: Tapping Soviet Undersea Cables

Physical layer tapping is not just a theoretical threat. One of the most famous real-world examples occurred during the Cold War.

In the 1970s, the U.S. Navy, with support from the NSA, carried out Operation Ivy Bells. The operation involved locating and tapping Soviet undersea communication cables in the Sea of Okhotsk. Divers installed a large, sophisticated recording device on the cable that clamped onto it without cutting or piercing the outer casing. The device recorded all traffic passing through the cable. If the cable ever needed repair and was lifted, the device was designed to detach and fall to the ocean floor, leaving no trace.

Every month, divers would retrieve the recorded tapes. Because the Soviets believed their undersea cables were physically secure, they transmitted a large amount of sensitive information **without encryption**.

<img src="{{ site.baseurl }}/assets/images/ivy-bells.jpg" alt="Operation Ivy Bells – Historical undersea cable tapping" />

> **Figure**: Conceptual illustration of undersea cable tapping during the Cold War (Operation Ivy Bells).

## Tapping Modern Fiber Optic Cables

Today, most long-distance internet traffic travels over fiber optic cables. While fiber is more secure than copper, it is not immune to physical tapping.

Attackers can tap fiber optic cables using a technique called _micro-bending_. The attacker carefully exposes the glass fiber and places it into a precision clamping device. This device applies a very small, controlled bend to the fiber. When light pulses hit this bend, a tiny fraction of the light leaks out through the side of the cladding (due to evanescent waves). A sensitive optical sensor captures this leaked light and converts it back into usable data.

Because only a small amount of light is diverted, the main signal continues to its destination with minimal disruption. In many cases, the communicating parties remain completely unaware that their physical link has been compromised.

<img src="{{ site.baseurl }}/assets/images/fiber-tap.jpeg" alt="Fiber optic cable tapping using micro-bending technique" />

> **Figure**: Simplified diagram of fiber optic tapping using a micro-bend clamping device.

## What Physical Layer Tapping Shows

The inherent vulnerabilities of physical layer eavesdropping and cable tapping reinforce a core architectural principle of network security: the physical transmission medium must always be treated as untrusted.

Because the lowest layers of the network stack are designed strictly for data delivery rather than data security, the physical signals carrying our data remain exposed to anyone who can gain physical proximity to the airwaves or the cables. As we will observe throughout this course, vulnerabilities persist across other lower layers of the internet stack as well. This systemic exposure is precisely why modern network architecture relies heavily on higher-layer, end-to-end encryption protocols such as TLS, which will be explored in depth in a later unit.

However, relying entirely on higher-layer encryption creates a false sense of absolute security. While TLS completely hides the content of a communication session (such as the actual messages, login credentials, or specific URLs), it cannot hide the physical and structural properties of the transmission itself. An attacker performing a Physical Layer attack can still extract valuable metadata from the unencrypted parts of the network headers.

For example, the destination IP address remains visible in the IP header. In addition, the Server Name Indication (SNI) field sent during the TLS handshake often reveals the domain name the client is trying to reach (for example, `youtube.com`). By analyzing the size, volume, and timing of the encrypted packets (a technique known as _traffic fingerprinting_) an attacker can often infer what kind of activity is taking place, such as video streaming, web browsing, or file downloads.

This shows that while encryption effectively protects the _content_ of communication, it does not hide all metadata. A determined eavesdropper at the Physical Layer can still learn a surprising amount about a user’s activity even when TLS is properly enabled. Because data must travel across vast geographic distances through physical infrastructure, protecting that infrastructure remains essential. This is also why many countries have laws that restrict or prohibit sending certain sensitive data (such as medical records) outside their borders.
