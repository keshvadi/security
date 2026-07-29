---
title: Introduction
nav_order: 1
has_children: true
layout: default
header-includes:
  - \pagenumbering{gobble}
---

# Introduction to Security

Security is fundamentally about protecting systems and data in the presence of adversaries.
Before examining specific technologies or attack techniques, it is essential to understand what we are trying to protect, what “secure” actually means, and why achieving security is difficult in practice.

This section introduces the core concepts that underpin all of computer security. We start by defining the primary goals of security through the _CIA Triad (Confidentiality, Integrity, and Availability)_. We then explore the _security mindset_: shifting your perspective to anticipate how adversaries will target a system, while framing our defenses in terms of policies (what is allowed), mechanisms (how it is enforced), and assumptions (what we trust).

We also examine the practical challenges of securing complex systems.
Finally, we introduce a set of general _security design principles_, concepts such as least privilege, defense in depth, and complete mediation, that appear repeatedly throughout this book and form the basis of real-world secure system design.
