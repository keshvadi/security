---
title: Web Security
nav_order: 4
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Web Security

The modern world runs on the web. From banking and healthcare to social media, e-commerce, and critical infrastructure, nearly every important service we use is delivered through a web browser. This convenience comes with a significant attack surface: web applications are complex, distributed systems that run untrusted code in users' browsers while handling sensitive data on remote servers.

**Web security** is the study of the attacks that target these systems and the defenses used to protect them. It focuses on the unique threats that arise because code from different origins runs together in the same browser, because HTTP is stateless, and because web applications must safely handle user input from untrusted sources.

In this unit we will examine the core technologies that power the web, the fundamental security mechanisms built into browsers (most notably the Same-Origin Policy and cookie rules), and the major classes of attacks that exploit weaknesses in web applications.

We will also study the practical defenses that real-world developers and browser vendors use to mitigate these threats.

By the end of this unit, you will understand how these attacks work, why they are possible, and how to design and build web applications that are resilient against them.
