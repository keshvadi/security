---
title: Cryptography
nav_order: 2
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Cryptography

In our earlier discussion of the foundations of security, we introduced the core goals of confidentiality, integrity, and availability (the CIA triad). The next question is practical: how can we protect data and communication when they must traverse networks we do not control and cannot fully trust?

One of the key mechanisms is *cryptography* (literally secret writing), an engineering discipline that enables secure communication over an insecure channel (like the Internet) by turning security goals into concrete, enforceable mechanisms. In doing so, cryptography helps defend against realistic threats such as eavesdropping, tampering, and impersonation.

In this unit, you will learn how cryptographic tools support essential security services (e.g., confidentiality, integrity, and authentication), how keys and algorithms work together, and how real-world constraints such as performance, usability, and randomness shape security outcomes in practice.
