---
title: Network Security
nav_order: 3
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Network Security

Welcome to Network Security.

In the last unit, you built a strong foundation in cryptography. You learned how symmetric-key and public-key encryption, cryptographic hashes, message authentication codes, digital signatures, key-exchange protocols such as Diffie-Hellman, and certificates work together to provide confidentiality, integrity, and authentication for individual messages or files. These cryptographic tools are powerful, but they are only part of the picture. In practice, data does not travel in a vacuum. It moves across complex computer networks. If the network itself is insecure, even the strongest cryptography can be bypassed or rendered useless.

This unit examines how data is packaged, routed, and protected as it travels from one device to another across local networks and the global Internet.

Because many students take this course at the same time as (or even before) a dedicated computer-networking course, we will not assume you already know networking concepts in depth. Whenever we need to discuss a protocol, header format, or routing mechanism, we will first provide a clear, concise background so you can understand both how the network works and why it is vulnerable. You might skim those background sections if you have already taken a computer networking course or are already familiar with the material. However, we recommend at least skimming them, because we will refer to these concepts frequently in our security discussions. Note that this background is not intended as a replacement for a dedicated computer networking course as we cover only the specific concepts and details needed to understand the network security topics in this unit.
