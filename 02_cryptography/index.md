---
title: Cryptography
nav_order: 2
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Cryptography

You're sending your banking details, a private message, or even your health records over the Internet. Every packet races across untrusted wires, routers controlled by who-knows-who, and servers that might be compromised. Without protection, a nosy ISP, a hacker on public Wi-Fi, or a nation-state could read, alter, or steal it all in seconds.
Yet billions of these transactions happen every day, securely. How?

Cryptography is the engineering discipline that solves this problem. It lets us build confidentiality (so eavesdroppers can’t read your data), integrity (so attackers can’t change it without detection), and authentication (so you know who you’re really talking to), in the most hostile environment on Earth: the open Internet.
Cryptography powers everything from HTTPS locks in your browser (protecting trillions in daily e-commerce) to end-to-end encrypted chats (keeping your messages private from everyone except the recipient), blockchain transactions, VPN tunnels, and even quantum-resistant defenses against tomorrow's threats. When it fails (think massive data breaches or weak legacy encryption), lives, money, and trust collapse.

Over the coming topics, we’ll gradually unpack the layered cryptographic mechanisms that power today’s Internet: symmetric encryption, asymmetric keys, hash functions, digital signatures, certificates, and more. By the end, you’ll see the familiar web browser lock icon, secure messaging apps, VPNs, and blockchain transactions in a completely new light.
