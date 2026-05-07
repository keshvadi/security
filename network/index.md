---
title: Network Security
nav_order: 5
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Network Security

Welcome to Unit 2: Network Security. 

In Unit 1 you built a strong foundation in cryptography. You learned how symmetric-key and public-key encryption, cryptographic hashes, message authentication codes, digital signatures, key-exchange protocols such as Diffie-Hellman, and certificates work together to provide confidentiality, integrity, and authentication for individual messages or files. These cryptographic tools are powerful, but they are only part of the picture. In practice, data does not travel in a vacuum. It moves across complex computer networks. If the network itself is insecure, even the strongest cryptography can be bypassed or rendered useless.

This unit examines how data is packaged, routed, and protected as it travels from one device to another across local networks and the global Internet. Because many students take this course at the same time as (or even before) a dedicated computer-networking course, we will not assume you already know networking concepts in depth. Whenever we need to discuss a protocol, header format, or routing mechanism, we will first provide a clear, concise background so you can understand both how the network works and why it is vulnerable. You might skim those background sections if you have already taken a computer networking course or are already familiar with the material. However, we recommend at least skimming them, because we will refer to these concepts frequently in our security discussions. Note that this background is not intended as a replacement for a dedicated computer networking course as we cover only the specific concepts and details needed to understand the network security topics in this unit.

The unit is organized into eight topics that follow the natural layers and functions of a network:

Topic 1 introduces core networking fundamentals so everyone starts on the same page.
Topic 2 explores vulnerabilities at the link layer (the lowest level, closest to the physical wires or wireless signals).
Topics 3 and 4 cover the transport layer, including how TCP and UDP operate and how Transport Layer Security (TLS) protects web and other traffic.
Topic 5 looks at critical Internet infrastructure, specifically the Border Gateway Protocol (BGP) that routes traffic between networks and the Domain Name System (DNS) that translates human-friendly names into IP addresses.
Topic 6 examines Denial-of-Service (DoS) attacks, one of the most common and disruptive threats on the Internet.
Topic 7 presents the primary defensive tools used in practice: firewalls and intrusion detection/prevention systems.
Topic 8 concludes the unit by discussing anonymity on the network, i.e., how users can communicate privately and what technologies (and limitations) exist today.