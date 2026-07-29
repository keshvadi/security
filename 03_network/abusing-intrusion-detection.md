---
title: Abusing Intrusion Detection
parent: Network Security
nav_order: 15
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Abusing Intrusion Detection

The previous topic described how organizations try to notice attacks. Attackers, of course, know these techniques exist and actively try to make detection fail. This topic examines the other side of the coin: how detectors can be _evaded_, _abused_, or _scaled far beyond_ defensive scenarios.

## Technical Evasion of Network Intrusion Detection

A NIDS sitting at a network choke point faces a fundamentally difficult task: it must observe a passing stream of packets and perfectly predict how the destination server will interpret them. To answer a seemingly simple question like, "Is this HTTP request malicious?", the NIDS must reassemble the TCP stream and parse the application data exactly as the target web server would.

Attackers exploit the subtle discrepancies between the NIDS's "model of the world" and the target server's actual behavior. By carefully crafting anomalous packets, an attacker can intentionally desynchronize the NIDS's reconstructed stream from the server's reality.

### TCP and IP Layer Tricks

At the network and transport layers, attackers exploit ambiguities in protocol specifications and the differing ways operating systems implement their TCP/IP stacks.

- **Time-To-Live (TTL) Games**: Every IP packet has a TTL field that decrements by one at each router. If it reaches zero, the packet is dropped. An attacker can map the network to find exactly how many hops away the NIDS and the target server are (using `traceroute` for example). By sending malicious packets with a normal TTL, interleaved with harmless "garbage" packets possessing a short TTL that expires _after_ the NIDS but _before_ the server, the attacker forces the NIDS to read a jumbled, harmless string. The server, however, only receives the malicious payload.
- **Fragmentation and Overlapping Reassembly**: When packets are fragmented, the receiving host must reassemble them. If an attacker sends overlapping fragments with conflicting data, which fragment wins? Some operating systems favor the original fragment; others favor the most recent. If the NIDS and the target server resolve overlaps differently, the NIDS will "see" benign text while the server reconstructs the malicious payload.
- **Checksum and Corruption Tricks**: A compliant server silently drops packets with invalid mathematical checksums. However, to save CPU cycles, a poorly configured NIDS might skip verifying checksums. An attacker can send benign data with an invalid checksum. The NIDS accepts it, corrupting the attack signature it was looking for, while the target server strictly checks the math and drops the benign packet, processing only the underlying attack.

### Application-Layer Encoding

Even if the NIDS reconstructs the TCP stream perfectly, it still has to parse the application layer. Attackers bypass simple signature-matching by disguising malicious strings (like `/etc/passwd`) using encoding tricks.

- **URL Encoding**: Replacing characters with their hexadecimal equivalents (e.g., `%2e%2e%2f` instead of `../`).
- **Double Encoding**: An attacker encodes the payload twice (e.g., `%252e`, where `%25` is the code for the `%` symbol). If the NIDS decodes the string only once, it sees `%2e` and lets it pass. The target web server, however, might decode it once at the proxy level and again at the application level, executing the true payload.

To catch these, a NIDS must perform _exactly_ the same sequence of normalizations the target application performs, for every possible application on the network. This is computationally and logically exhausting.

## Encryption as the Ultimate Content Evasion

The single most effective way to defeat any network-based content inspection is to encrypt the traffic. Modern HTTPS, SSH, and corporate VPNs do this by default. A NIDS that used to look for SQL injection strings in cleartext HTTP now sees only a TLS handshake, encrypted application data, and an opaque stream of bytes.

Faced with ubiquitous encryption, network defenders have two unsatisfying choices:

1. _Terminate TLS early_ using a reverse proxy to inspect the cleartext before re-encrypting it. This creates a massive single point of failure; whoever compromises the proxy has access to all organizational data.
2. _Give up on content inspection_ and rely instead on metadata, traffic fingerprinting, and endpoint detection.

The latter has become the dominant reality. Widespread encryption has forced the detection community to shift its focus from _"what did they say?"_ to _"who is talking to whom, when, how much, and using what software?"_

## Evasion and Abuse at the Endpoint

If the network is blind, defenders must rely on Host-based Intrusion Detection Systems (HIDS) and logging. Naturally, attackers have adapted to evade these as well.

- _Log Tampering_: Upon achieving a foothold, an attacker's first priority is often to edit or delete system logs, clear shell histories, or install rootkits that hide malicious processes from the operating system itself.
- _Living Off the Land (LOL)_: Instead of dropping custom, easily detectable malware (like `evil_keylogger.exe`), attackers use administrative tools that are already installed on the system (like PowerShell, `curl`, or Windows Management Instrumentation). Because system administrators use these exact same tools daily, signature-based HIDS cannot simply block them.
- _Mimicry and Time Bombs_: Attackers script their automated behaviors to match the rhythm of legitimate users, or they plant "time bombs" that only execute malicious actions on weekends or after standard log-retention periods have rotated out the evidence of their initial entry.

## Large-Scale Surveillance: Detection at Planet Scale

The same conceptual building blocks used in a corporate NIDS (i.e., packet capture, stream reassembly, metadata extraction, and long-term storage) scale to nation-state wiretap programs when the choke points are international fiber-optic cables and the storage is measured in exabytes.

One well-documented example is the NSA's **Xkeyscore** system. In intelligence community parlance, this system relies on three concepts:

- A **selector**: Information that identifies a target (an email address, IP, or tracking cookie).
- A **fingerprint**: A detector rule that recognizes a specific application or protocol (similar to a corporate NIDS signature).
- An **implant**: Malware placed on a target machine for persistent access.

Xkeyscore acts as a massively scalable NIDS. Built on big-data technologies, it provides analysts with an interface to run pre-canned scripts, allowing them to sift through global traffic. But because it is a network monitor, it faces the same encryption problem as corporate defenders: it cannot read the content inside a TLS tunnel.

To bypass the encryption wall, modern intelligence relies on metadata and the "Collect It All" strategy. By storing exabytes of metadata indefinitely, analysts can retrospectively search for patterns.

Consider an analyst trying to identify two anonymous users chatting on an encrypted IRC channel. The content is unreadable. However, the analyst knows both users clicked a link to a specific, obscure news article at a specific time. By correlating web-traffic metadata across global collection points, the analyst can identify the IP addresses that visited that article at that exact timestamp.

This is known as building a **"pattern of life."** Once the identities are unmasked via metadata, the analyst pivots from passive surveillance to active **Computer Network Exploitation (CNE)**, deploying an implant directly onto the targets' endpoints to bypass the network encryption entirely.

_Note: Programs like these operate under complex legal frameworks, such as the UKUSA agreement (underpinning the "Five Eyes" intelligence alliance) and FISA Section 702 in the United States, which dictate how and when data can be collected or shared across borders._
