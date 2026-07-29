---
title: Introduction to Computer Security
parent: Introduction
nav_order: 1
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Introduction to Computer Security

## What is Computer Security?

Computer security is the discipline of protecting computing systems, networks, and data from unauthorized access, modification, disruption, or destruction. It is about keeping systems functioning as intended, free of abuse from attackers. We want data to be accessed only by the entities we choose, resources and capabilities (like printing on our printers) to be used only by authorized users, and we want to enable privacy and anonymity where needed. We need to achieve all of this in the presence of an intelligent adversary, and we have to do it on a budget. We don't have infinite resources to spend on defenses.

Unlike reliability engineering, which focuses on protecting systems against accidental failures and random faults, security is concerned with protection in the presence of intelligent adversaries who actively try to defeat the system’s protections.

At its core, security is about achieving specific _goals_ in an adversarial environment. These goals describe what we want to protect and what kinds of undesirable outcomes we want to prevent. Only after the goals are clear does it make sense to discuss the mechanisms, protocols, and design principles used to achieve them.

This unit begins by examining the classic security goals, then introduces the way of thinking that security professionals use when analyzing systems, and finally explains why these goals are so difficult to achieve in practice.

### General Themes in Computer Security

Several fundamental themes run through computer security:

- Computers do precisely what they're told.  
  They are just machines that faithfully execute instructions, with no awareness of whether those instructions are helpful or harmful. A computer can't tell the difference between a legitimate request and a malicious one if they are issued the same way.

- Code is data, and data is code.  
  The distinction we imagine isn't as clear as it seems. Programs are stored as data, and in languages like Python or JavaScript, you can take a string and evaluate it directly as code. Even exploits like buffer overflows treat carefully crafted data as executable instructions.

- By adding usefulness and features, we inevitably introduce vulnerabilities.
  The most secure computer is one that is powered off and unplugged, but that is not very useful! It is features and convenience that create vulnerabilities. This includes handy features of programming languages themselves, like `eval()` statements that make development easier but open dangerous doors if input isn't perfectly controlled (we will return to this idea later when we study code injection attacks).

- The goal is risk management, not perfection.
  Since 100% security is impossible, our goal is to invest enough in defenses to deter likely attacks while keeping the system usable and productive.

## The CIA Triad

The most widely used framework for describing security goals is the _CIA triad_: Confidentiality, Integrity, and Availability. These three properties capture the primary objectives that most security mechanisms are designed to achieve.

_Confidentiality_ is the goal of keeping information secret from those who are not authorized to see it. A system that provides confidentiality ensures that sensitive data is disclosed only to authorized parties.

Classic examples include encrypting network traffic so that eavesdroppers cannot read the contents of messages, and enforcing access-control policies so that only permitted users can read a file. Confidentiality is violated when an unauthorized party obtains access to information, whether through direct access, interception, or inference.

_Integrity_ is the goal of ensuring that data and systems are not modified in unauthorized ways. A system that provides integrity detects or prevents unauthorized changes to information or to the system itself.

Integrity covers both the content of data (for example, ensuring that a financial transaction has not been altered in transit) and the state of the system (for example, ensuring that critical configuration files or executable code have not been tampered with). Note that integrity also applies to people (e.g., bribery or corruption can compromise the "integrity" of a human operator). When integrity is violated, an adversary may be able to change records, inject malicious code, or cause the system to behave incorrectly.

_Availability_ is the goal of ensuring that systems, services, and data remain accessible and usable when needed by authorized users. A system that provides availability continues to operate correctly even in the face of attacks or failures that attempt to disrupt it.

Availability is violated by denial-of-service attacks, ransomware that encrypts data and withholds the decryption key, or any action that prevents legitimate users from accessing a resource. In many critical systems (for example, hospital networks or industrial control systems), loss of availability can have serious real-world consequences.

### Trade-offs Among Security Goals

The three goals of the CIA triad are not always independent.
Strengthening one goal can weaken another, and real systems must often make explicit trade-offs among them.

The most common tension is between confidentiality (or integrity) and availability.
Strong access controls, heavy encryption, or strict integrity checks can make a system more secure against unauthorized access or modification, but they may also make the system slower, harder to use, or more fragile. In the extreme, a system that is completely disconnected from any network achieves very high confidentiality and integrity, yet provides almost no availability to remote users.

Another frequent trade-off appears between security and usability. Mechanisms that require users to follow complex procedures (for example, frequent re-authentication or cumbersome key management) may improve confidentiality or integrity on paper, but users often find ways to circumvent them, ultimately reducing overall security.

Because perfect security is rarely achievable, designers must decide which goals matter most for a given system and accept that improving one property may come at the expense of another. Making these trade-offs consciously, rather than by accident, is an essential part of secure system design.

## The Security Mindset

The security mindset is a way of thinking about systems that focuses on how they can fail when faced with an adversary. Instead of asking only “Does this system work correctly under normal conditions?”, a person with the security mindset asks “How can this system be misused or attacked?” and “What happens if someone tries to defeat the protections?”

A useful way to apply this mindset is to analyze any system in terms of three elements:

- _Policy_: What is supposed to happen? What actions are allowed or forbidden?
- _Mechanism_: How is the policy enforced? What technical or procedural controls are in place?
- _Assumptions_: What must be true for the mechanism to correctly enforce the policy? What are we trusting?

Security failures often occur when one of these three elements is incomplete or incorrect. The policy may be unclear or too weak, the mechanism may not fully enforce the policy, or the assumptions may not hold in the real world. By making the policy, mechanism, and assumptions explicit, we can more systematically identify weaknesses before an adversary does.

Developing the security mindset means habitually looking for the gap between what a system is intended to do and what an adversary can actually make it do.

### Example A: The Liquor Store

- **Policy**: “No minors are allowed to purchase alcohol.”
- **Mechanism**: A cashier inspects a government-issued ID at the register.
- **Assumption**: IDs are difficult to forge, and forged IDs are easy to detect.
- **The vulnerability**: If an attacker has a high-quality fake ID, the mechanism fails because the assumption was wrong.
  Have you ever heard of a minor successfully buying age-restricted products? If even one minor succeeds, it suggests the system relies on an assumption that is not always true.

### Example B: Phone Banking

- **Policy**: “Only the account holder is allowed to access the bank account.”
- **Mechanism**: The bank verifies identity using a Date of Birth and Home Address.
- **Assumption**: Date of Birth and Home Address are “secrets” known only to the account holder.
- **The vulnerability**: This information is often public or discoverable, so the mechanism can work exactly as designed and still fail to provide security.
  Who else might know your address and date of birth? Employers, schools, family members, friends, or social media contacts? If this information is known (or easily discoverable) by others, the mechanism relies on a false assumption and therefore does not reliably enforce the policy.

Developers often work forward: they define a policy and then build a mechanism to enforce it.  
Security auditing often works backward: you start from what you can observe and infer what the system is trying to protect, and what it assumes.

## Why Security Is Hard

We often ask, “Why can’t we just build secure computer systems?” If we can build bridges that stand for centuries and airplanes that fly safely for millions of miles, why is system security so difficult?

The answer lies in the unique nature of the field. In civil engineering, gravity is a constant force, it does not change its rules to make your bridge collapse. In computer security, we face an intelligent adversary who actively analyzes our defenses and changes their tactics to defeat them. Security challenges are not just technical; they are economic, human, and asymmetric.
Achieving security is difficult for several fundamental reasons, some of which are provided below.

### 1. The Asymmetry of Defense

The most fundamental challenge in security is the imbalance between the defender (you) and the attacker. A defender must secure every possible entry point—every door, window, and software port. The attacker only needs to find one weakness to succeed.

Defenses are usually visible (locks, guards, firewalls), allowing attackers to study them at their leisure. Attacks, however, are planned in secret. The attacker can abort or change plans without you ever knowing they were there.

Defenders spend huge budgets building static defenses (like a moat). Attackers are nimble; if they invent a “boat”, your expensive moat becomes less effective overnight. Defenders must follow laws, protocols (like HTTP), and ethical standards. Criminals are not bound by these constraints. In addition, the internet facilitates attacks of great scale at little cost, allowing attackers to strike from anywhere on the planet with minimal deterrence.

### 2. The Intelligent and Adaptive Adversary

An attacker can induce “zero-probability” faults, forcing the system into states that would never happen during normal operation. They can exhibit arbitrary behavior, such as feeding a program malicious inputs that valid users would never type.

Furthermore, this field is an arms race. The adversary evolves alongside our defenses. Every time we deploy a new security measure, attackers analyze it and develop a new method to bypass it. This constant evolution means that a system considered “secure” today may be vulnerable tomorrow.

### 3. Complexity and Abstraction

Modern computer systems are built on layers of abstractions. When we write code or design networks, we often forget the gritty details of the hardware or lower-level protocols that sit beneath us. However, attackers do not forget. They exploit the specific details and quirks that these abstractions try to hide.

To make matters worse, computers evolve faster than security. We constantly demand new features, patches, and backwards compatibility. As complexity grows, the number of vulnerabilities tends to outscale the lines of code. We are often building defenses for yesterday’s technology while trying to secure tomorrow’s features.

### 4. The Economics of Security

Security is fundamentally an economic problem. First, security has costs. It adds overhead, burdens users, and takes time to deploy. Second, it is hard to measure. How do you measure the value of a “lack of disaster”? It is difficult to determine if a security investment was worth it, especially since breaches are often discovered long after the attack occurred.

Finally, market economics often fail in security. Those in a position to fix the problem (like software vendors) often don’t bear the cost of the failure (the users do). Security becomes a “tax” we all pay (e.g., store security raises prices for everyone), but the incentives to pay that tax are often misaligned.

### 5. The Human Factor

A major challenge is that security often gets in the way. If a security mechanism is inconvenient or hard to use, users will bypass it or undermine it without realizing the risk. This leads to the “Dancing Pigs” problem: users will often choose a fun or convenient feature over a security warning.

Because no formal training is required to use a computer, we cannot expect users to behave securely. This makes social engineering—manipulating people into breaking their own security procedures—a highly effective and persistent attack vector.

> **Note**: The “Dancing Pigs” problem is a classic metaphor in computer security: “Given a choice between dancing pigs and security, users will pick dancing pigs every time”. It comes from a famous quote by security expert Gary McGraw (often attributed to Edward Felten).

### 6. Regulatory and Political Challenges

Finally, we face government obstacles. Governments have a dual desire: they want to protect their citizens’ data, but they also want to monitor communications for law enforcement and intelligence. This often leads to conflicting policies, such as pushes to weaken encryption or install “backdoors”, which can inadvertently weaken security for everyone.
