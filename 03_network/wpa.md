---
title: WPA2 Attacks
parent: Network Security
nav_order: 5
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Wireless Local Networks: WPA2 Attacks

## Cheat sheet

- **Layer**: Link (Layer 2)
- **Purpose**: Provide confidentiality, integrity, and authentication for wireless traffic on a Wi-Fi network
- **Main Vulnerability**: The 4-way handshake sends nonces and MICs in plaintext. An on-path attacker who captures the handshake can derive a client’s encryption keys if they know (or can crack) the Wi-Fi password
- **Attack Result**: The attacker can decrypt unicast traffic for targeted clients, decrypt broadcast traffic, and inject or modify packets (effectively becoming a man-in-the-middle)
- **Core Weakness**: WPA2-PSK uses a single shared password for the entire network. Anyone who knows or cracks this password can derive other users’ encryption keys
- **Defenses**:
  - WPA2-Enterprise: Gives each user a unique key (via RADIUS authentication)
  - Strong, high-entropy Wi-Fi passwords (to slow down offline brute-force attacks)
  - WPA3 (newer standard): Provides stronger protection against offline dictionary attacks and improves handshake security
  - Higher-layer encryption (TLS) as a safety net

---

## Networking Background: WiFi and the Need for Link-Layer Security

WiFi (IEEE 802.11) is another implementation of the Link Layer (Layer 2). It behaves similarly to wired Ethernet in many ways: devices are identified by MAC addresses and protocols such as ARP are still used to resolve IP addresses to MAC addresses. However, there is one fundamental and security-critical difference: WiFi is wireless. When a device connects to a WiFi network, it communicates with an Access Point (AP) using radio waves. Unlike a wired switch, which can direct traffic only to the intended port, a WiFi access point broadcasts frames into the air. Any device within radio range that is listening on the same channel can potentially receive those frames.
This broadcast nature of wireless communication significantly expands the attack surface. An attacker does not need physical access to a cable, nor do they even need to be connected to the network. Anyone nearby with a wireless card can potentially observe or interfere with traffic.

If a WiFi network has no password (commonly called an “open” network), the risks are severe. Any device can connect immediately without authentication, and all data is transmitted in plaintext with no encryption or integrity protection. Anyone within radio range can passively eavesdrop on traffic, actively inject or modify packets, and perform attacks such as ARP spoofing or DHCP spoofing against other devices on the network.
Connecting to an open WiFi network is roughly equivalent to plugging your laptop into a hub in a room full of strangers. There is no confidentiality, no integrity protection, and no reliable way to know who else might be listening or interfering.

Because wireless networks broadcast traffic into the air with no physical boundaries, they are far more exposed than wired networks. To address these risks, WPA2 (Wi-Fi Protected Access 2) was developed. WPA2 aims to provide three core security properties for wireless traffic:

- Confidentiality: Only the intended recipients should be able to read the data.
- Integrity: Receivers should be able to detect whether data has been modified in transit.
- Authentication: Devices should be able to verify they are communicating with the legitimate access point (and vice versa).

WPA2 is available in two main forms:

- **WPA2-PSK** (Pre-Shared Key): Uses a single shared Wi-Fi password. This is the most common form used in homes, small offices, and many public hotspots.
- **WPA2-Enterprise**: Uses individual credentials for each user, typically backed by a central authentication server. This form is more common in universities and large organizations.

## WPA2-PSK and the 4-Way Handshake

**WPA2-PSK** (Wi-Fi Protected Access 2 – Pre-Shared Key) is the most widely used form of WPA2. It uses a single shared password (the Wi-Fi password) to secure the network. When a network administrator configures WPA2-PSK:

1. They set a Wi-Fi password.
2. Both the Access Point and every client derive a **PSK (Pre-Shared Key)** by running a password-based key derivation function (PBKDF2-SHA1) on the combination of the SSID (network name) and the password. Including the SSID in the derivation ensures that two different networks using the exact same password will still produce different PSKs.

At this point, the client and the Access Point share the same PSK. However, we cannot simply use this PSK to encrypt all future communication. If we did, every device on the network would be able to decrypt every other device’s traffic, since they all know the same key. We need a way to generate unique encryption keys for each client.

### The 4-Way Handshake

To solve this problem, WPA2-PSK uses a **4-way handshake** after a client connects. The goals of this handshake are to:

- Prove that both sides know the PSK without ever sending it over the air.
- Generate a fresh **PTK (Pairwise Transient Key)**, a unique encryption key used only between this specific client and the Access Point.
- Securely deliver the **GTK (Group Temporal Key)**, which is used to encrypt broadcast and multicast traffic for everyone on the network.

Here is how the 4-way handshake works:
<img src="{{ site.baseurl }}/assets/images/wpa2-real.png" alt="Diagram of the optimized WPA2 4-way handshake" width="55%">

_Message 1 (Access Point → Client)_: The Access Point sends a random value called the _ANonce_ (Authenticator Nonce). This value is sent in plaintext.

_Message 2 (Client → Access Point)_: The client now has enough information to derive the _PTK_. It calculates the PTK using:

- The ANonce received from the Access Point
- A random value it generates itself called the _SNonce_ (Supplicant Nonce)
- The PSK
- The MAC addresses of both the Access Point and the client

Once the client derives the PTK, it uses a portion of the PTK to calculate a MIC (Message Integrity Code) over Message 2. It then sends the SNonce and this MIC to the Access Point. The MIC proves that the client knows the PSK and that Message 2 was not tampered with.

_Message 3 (Access Point → Client)_: The Access Point can now also derive the same PTK. It sends the _GTK_ (encrypted using the PTK) along with its own MIC to prove that it also knows the PSK.

_Message 4 (Client → Access Point)_: The client acknowledges that it has successfully received and installed the GTK.

Once the handshake completes successfully, the client and Access Point share: a unique _PTK_ used to encrypt all unicast traffic between them, and the _GTK_ used to encrypt broadcast and multicast traffic sent to everyone on the network.

Sanity check: Why can’t we simply use the PSK to encrypt all traffic?  
Answer: Because every device on the network would know the same key and could decrypt everyone else’s traffic.

## The Attack: Capturing the Handshake and Offline Cracking

The WPA2-PSK 4-way handshake has a critical design weakness: most of the messages are sent in plaintext. An on-path attacker who can observe the wireless traffic can capture the ANonce, SNonce, and the MICs. Combined with the SSID, this information is sufficient to mount two powerful attacks.

### Attack 1: Attacker Already Knows the Wi-Fi Password

If the attacker knows the Wi-Fi password (and has therefore derived the PSK), they can perform the following attack:

1. Capture a complete 4-way handshake between a victim client and the Access Point.
2. Use the captured nonces, MAC addresses, and the known PSK to independently derive the exact same _PTK_ that the victim negotiated.
3. Decrypt all future unicast traffic between that client and the Access Point.
4. Encrypt and inject packets as if they were the legitimate client or Access Point.

### Attack 2: Offline Password Brute-Forcing

Even if the attacker does not know the Wi-Fi password, they can still attempt to recover it through offline brute-forcing:

1. Capture a complete 4-way handshake.
2. For each password guess:
   - Derive the PSK from the guessed password + SSID.
   - Use the captured nonces and MAC addresses to derive a candidate PTK.
   - Check whether the candidate PTK produces the correct MICs seen in the handshake.
3. If the MICs match, the guessed password was correct.

This attack is performed offline, meaning the attacker does not need to interact with the network after capturing the handshake. They can test password guesses at very high speed using a powerful computer or GPU. Weak or low-entropy passwords (short passwords, dictionary words, common phrases, predictable patterns, etc.) can often be cracked in seconds or minutes.

Sanity check: Why can an attacker verify password guesses without ever connecting to the network?  
Answer: The MICs in the handshake act as a cryptographic fingerprint of the correct PTK. If a guessed password produces matching MICs, it must be the correct password.

### Impact and Real-World Consequences

Once an attacker has derived a client’s _PTK_ (either by knowing the Wi-Fi password or by successfully brute-forcing it), they gain significant power over that client’s wireless communication. With the PTK, an attacker can:

- Decrypt all unicast traffic between the victim client and the Access Point. This includes web browsing, email, file transfers, application data, and internal network traffic.
- Decrypt broadcast and multicast traffic using the _GTK_, which is shared across all devices on the network.
- Actively inject or modify packets. The attacker can forge frames that appear to come from either the client or the Access Point.
- Perform real-time man-in-the-middle attacks, such as redirecting traffic, injecting malicious content, stealing session cookies, or intercepting credentials.

Importantly, the attacker does not need to remain connected to the network after capturing the handshake. They can record the handshake, leave the area, and decrypt the captured traffic later at their convenience. This attack is particularly harmful on networks that use a single shared password for all users, such as many coffee shops, hotels, airports, conference venues, university guest networks, and corporate guest networks. In these environments, the Wi-Fi password is often known by a large number of people or is easily obtainable, making it much more likely that an attacker can either know the password or successfully crack it offline.

Users often connect to these networks assuming that “WPA2” provides strong protection. In reality, anyone else on the same network who knows the password (or can crack it) can decrypt their traffic. This creates a false sense of security. People often feel safer on a “password-protected” Wi-Fi network than on an open network, but if the password is shared or weak, the protection is much weaker than it appears.

## Defenses and Improvements

The root cause of the attacks we have discussed is that WPA2-PSK uses a single shared secret (the Wi-Fi password) for everyone on the network. Anyone who knows this password, or can brute-force it, can derive the encryption keys of other users.
Two main approaches exist to address this problem: **WPA2-Enterprise** and the newer **WPA3** standard.

### WPA2-Enterprise

_WPA2-Enterprise_ was designed to solve the fundamental weakness of the pre-shared key model by giving each user their own unique credentials.
Instead of using one shared password for the entire network, WPA2-Enterprise requires each authorized user to have their own credentials (typically a username and password).

In WPA2-Enterprise, before the handshake occurs, the client connects to a secure authentication server and proves its identity to that server by providing a username and password. (The connection to the authentication server is secured with TLS, which is covered in a later section.)
If the username and password are correct, the authentication server presents both the client and the access point with a random _PMK_ (Pairwise Master Key) to use instead of the PSK. The handshake proceeds as in the previous section, but it uses the PMK (unique for each user) in place of the PSK (same for all users) to derive the PTK.
Because each user receives a different, randomly generated PMK, knowing one user’s credentials does not allow an attacker to derive the encryption keys of other users.

WPA2-Enterprise is significantly stronger than PSK, but it is not perfect. It still relies on the 4-way handshake, and if an attacker is already authenticated on the network, they can still perform ARP or DHCP spoofing to become a man-in-the-middle for other users. It is also more complex to set up and manage than PSK.

### WPA3

WPA3 is the successor to WPA2 and was designed to address several of its security weaknesses. One of the most important improvements is better protection against offline dictionary attacks. Even if an attacker captures the handshake, they can no longer efficiently brute-force the password offline as they could with WPA2. WPA3 also provides forward secrecy, meaning that compromising the Wi-Fi password in the future does not allow an attacker to decrypt previously captured traffic. In addition, WPA3 introduces Opportunistic Wireless Encryption (OWE), which improves security on open networks by encrypting traffic even when no password is used. It also uses stronger encryption algorithms and provides better protection during the handshake process.

Although WPA3 is gradually being adopted, many networks and devices still rely on WPA2.

Regardless of whether a network uses WPA2-PSK, WPA2-Enterprise, or WPA3, higher-layer encryption such as TLS remains a critical safety net. Even if an attacker becomes a man-in-the-middle at the Link Layer, properly implemented TLS can still protect the confidentiality and integrity of application data.

## What ARP, DHCP, and WPA2 Show

By examining ARP, DHCP, and WiFi, we can identify several important lessons that apply broadly to network security at the link layer.

- **Broadcast protocols are inherently risky.** Both ARP and DHCP rely on broadcast messages, allowing any device on the local network to see requests and respond to them. WiFi takes this even further by broadcasting frames into the air. This design makes it easy for on-path attackers to inject malicious responses, whether through ARP spoofing, rogue DHCP servers, or packet injection on wireless networks.

- **Initialization is a particularly vulnerable moment.** When a device first joins a network or boots up, it often has no prior trust relationship or configuration. It must accept information from whoever responds first. This is why both ARP and DHCP are especially dangerous during the initial connection phase, and why connecting to an open WiFi network carries significant risk.

- **There is a fundamental trade-off between convenience and security.** Many link layer protocols were designed to make networking automatic and user-friendly. ARP and DHCP allow devices to operate with minimal configuration, and open WiFi networks allow easy access. However, this convenience comes at a cost: the lack of authentication makes these protocols easy to attack. Adding security often requires sacrificing some of that simplicity.

- **Man-in-the-middle attacks can be invisible.** A successful ARP spoofing or DHCP spoofing attack can turn an on-path attacker into a full man-in-the-middle without any obvious signs to the user. The same risk exists on WiFi, especially on open or weakly protected networks. Victims may continue using the network normally while their traffic is being intercepted or modified.
