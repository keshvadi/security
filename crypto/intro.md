---
title: Introduction to Cryptography
parent: Cryptography
nav_order: 1
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Introduction to Cryptography

## 1. Brief History of Cryptography

The word "cryptography" comes from the Latin roots _crypt_, meaning secret, and _graphia_, meaning writing. So cryptography is quite literally the study of how to write secret messages. Today, the field is the study of techniques used to secure communication by transforming readable information into an unreadable format for unauthorized parties.

### From Classical Ciphers to the Telegraph Era

Techniques for secret communication are ancient. A well-known early example is the *Caesar cipher*, attributed to Julius Caesar, which encrypts text by shifting each letter by a fixed number of positions in the alphabet. For instance, with a shift of 3, the message *“NETWORK”* becomes *“QHWZRUN”*. While simple by modern standards, such substitution ciphers capture a timeless goal: making intercepted messages unintelligible to outsiders.

Cryptography took on new urgency in the *19th century* with the rise of *electronic long-distance communication*, especially the telegraph. Military and diplomatic organizations needed ways to protect sensitive messages traveling over infrastructure that could be monitored. Because messages often had to be encrypted and decrypted *by hand*, many “pen-and-ink” systems remained relatively simple—practical for human operators, but also more vulnerable to well-resourced codebreakers and pattern-based attacks.

### The Mechanical Era and the Enigma Story

The next major shift came with the *mechanical era*, when engineers began building machines to automate encryption. The most famous example is the German *Enigma* machine used during World War II. Enigma was an impressive engineering system that greatly increased complexity and operational speed, making it practical to encrypt large volumes of military traffic.

First, earlier breakthroughs by Polish cryptanalysts (who had already made headway against earlier Enigma procedures) and the subsequent transfer of knowledge and materials (including access to Enigma-related components and replicas) provided a valuable foundation for the later British effort. Second, the work drew on deep mathematical and analytical expertise, including contributions by figures such as *Alan Turing*. Turing’s work on the *Bombe* (an electromechanical system used to help determine Enigma settings) helped narrow the space of possible configurations and made the search for the correct key feasible in practice. His contributions also helped shape ideas that influenced modern computing, and he is widely regarded as one of the founding fathers of computer science. Third, Bletchley Park scaled the operation to an industrial level: at its height, the British codebreaking effort grew to more than 10,000 personnel and coordinated workflows that combined human expertise with mechanized assistance, enabling many searches to run in parallel, the processing of enormous volumes of intercepted traffic, and the exploitation of weaknesses introduced by real-world procedures and repeated patterns.

A central lesson from this era is that cryptographic strength is not only about the cipher design. It also depends on how systems are used, what attackers can learn, and how much effort they are willing (and able) to invest.

### Modern Cryptography: Mathematics, Computers, and Security Theory
What we now call *modern cryptography* is distinguished by its strong reliance on mathematical foundations and computer-based implementation. After World War II, *Claude Shannon* played a foundational role by formalizing ideas about secrecy and information, including analysis related to the *one-time pad* (a scheme notable for achieving perfect secrecy under the right assumptions which we will cover in this unit).

As computing became central to business and government, the need for standardized, interoperable encryption grew. In the early 1970s, the U.S. standards community (NIST) introduced the *Data Encryption Standard (DES)* to meet practical needs in areas like banking and commercial data protection. From the late 1970s onward, cryptography rapidly developed into a rigorous field with formal definitions, security models, and proofs—laying the groundwork for the cryptographic systems that secure today’s networks, applications, and digital infrastructure.


## 2. Definitions

At a glance, it is easy to see why the Caesar cipher is weak: there are only 26 possible shifts, so an attacker can simply try them all until the message becomes readable. But *how do we argue this rigorously*—and more generally, how do we reason about whether a cryptographic scheme is secure?

To study cryptography formally, we introduce a precise framework: we define the participants, define what an attacker is allowed to do, and define what it means for the attacker to “succeed.” The rest of this section introduces core terms that we will use throughout the unit.

## Definitions: Alice, Bob, Eve, and Mallory

The most basic problem in cryptography is securing communication over an *insecure channel*. We typically describe this using a standard cast of characters. *Alice* and *Bob* are the parties who want to communicate securely, as if they were in the same room or connected by a private, untappable link. In real systems, Alice and Bob are not necessarily people. For example, Alice could be a web browser or mobile app, and Bob could be a web server or API.

Unfortunately, Alice and Bob must communicate over a channel such as a telephone line or the Internet that an adversary may be able to access. A passive attacker is traditionally called *Eve*, the *eavesdropper*, who can listen to or record traffic but does not modify it (for example, someone within the same Wi‑Fi range capturing wireless packets). In other settings, we consider an active attacker called *Mallory*, who can do everything Eve can *and also* tamper with communication (e.g., modify, inject, replay, or block messages). For instance, Mallory might be an entity positioned “on the path”, such as a compromised router, a malicious access point, or an operator with privileged access to networking infrastructure (e.g., an ISP-level device). In this sense, Mallory is a *superset* of Eve: an active attacker includes all passive capabilities plus additional control over the communication channel.

The goal of cryptographic design is to “scramble” communication so that:
- Eve learns nothing useful about the message contents, and
- Mallory cannot alter the communication without the change being detected.

In other words, we aim to simulate an ideal communication channel using only the insecure channel that is actually available.

## Definitions: Keys

A central building block of nearly every cryptographic system (a *cryptosystem*) is the *key*. A key is a value that controls a cryptographic algorithm, enabling it to provide security services such as confidentiality and authentication. In practice, many cryptographic algorithms are designed to be public, and the *key* is what must remain secret.

There are two main key models in modern cryptography. In the _symmetric-key_ model, Alice and Bob share a single secret key and use that shared secret to protect communication. In the _asymmetric-key_ (public-key) model, each party has a *key pair* consisting of a *private key* (kept secret) and a *public key* (shared openly). Public keys enable secure communication and verification without requiring Alice and Bob to share a secret in advance.

We will explore both models in depth in the upcoming topics.

## Definitions: Kerckhoff's Principle

To analyze security, we must be explicit about the *threat model*: what can attackers like Eve and Mallory see and do, and what information do they have?
A foundational assumption in modern cryptography is *Kerckhoffs’s Principle*, which can be summarized as follows:

> A cryptosystem should remain secure even if attackers know how it works.  
> The system should depend on the secrecy of the key, not the secrecy of the algorithm, and it should be possible to recover from leaks by rotating keys rather than redesigning the entire system.  
> This idea is closely related to the maxim “don’t rely on security through obscurity”.

Accordingly, throughout this unit we will assume that adversaries may know the encryption and decryption algorithms and may understand how the system is implemented.[^1] The only information the attacker is missing is the secret key(s).

## 3. Confidentiality, Integrity, Authenticity

In the previous unit, you learned the *CIA triad*: confidentiality, integrity, and availability. In this cryptography unit, we focus on how cryptographic tools help achieve *confidentiality* and *integrity*, and how they support *authentication* (which is closely tied to integrity). Availability is an essential security goal, but it is usually addressed through system design and operational defenses rather than cryptographic mechanisms alone.

At a high level, cryptography provides these security services using a small set of *cryptographic primitives* (basic building blocks). You will learn the details of these primitives in the next topics. For now, we introduce the goals they are designed to achieve.

### Confidentiality

As mentioned before, *Confidentiality* means an adversary should not be able to learn the contents of private data. For messages, confidentiality aims to ensure that an attacker who observes traffic still cannot understand what was sent.
A useful mental model is a *lockbox*. Alice places her message inside a locked box and sends it over an insecure channel. Eve can see (and even copy) the box, but without the right key she cannot open it. Bob, who has the key, can unlock the box and read the message.

In modern systems, the “lockbox” is *encryption*. The original message is called *plaintext*. After encryption, it becomes *ciphertext*, which should look meaningless to an attacker. Bob uses *decryption* to turn ciphertext back into the original plaintext.
Even if Eve captures the ciphertext, she should not be able to recover the plaintext without the appropriate key.

### Integrity
*Integrity* means an adversary should not be able to change data without being detected. If a message has integrity, then modifications (whether accidental or malicious) can be identified by the receiver.
Integrity is especially important against an active attacker like Mallory, who may attempt to modify messages in transit or inject new ones.

### Authenticity (Authentication)
*Authenticity* means the receiver can determine *who* created a message. If a message is authentic, Bob can be confident that it was produced by the claimed sender (e.g., Alice), rather than by an attacker.
Authenticity and integrity are closely related: it is hard to believe a message “came from Alice” unless you also have a way to detect whether it was altered. In that sense, integrity is often a prerequisite for authenticity. However, the two are not identical, and we will see later where the distinction matters.

A helpful analogy for authenticaion is a *tamper-evident seal* on an envelope. Alice attaches a special seal to her message before sending it. If Mallory tampers with the message, the seal breaks. Without the right key (or the right private key, in public-key systems), Mallory cannot create a seal that will pass verification.
In cryptographic terms, Alice attaches a value that the receiver can check. In some schemes, Alice computes a short *tag* or computes a *digital signature* that Bob can verify.
If the message is modified, verification should fail, allowing Bob to detect tampering. An attacker should not be able to generate valid tags or signatures for forged messages.

> **Note:** In the upcoming topics, you will learn the specific primitives that provide these services (e.g., encryption for confidentiality, and integrity/authentication mechanisms such as MACs and digital signatures), and how real protocols combine them.

### Non-Repudiation vs. Deniability

In addition to confidentiality and integrity, we often consider a property known as *non-repudiation*. This is a security service that ensures a sender cannot later deny having sent a message. In the _asymmetric-key_ model, because only Alice possesses her private key, a digital signature provides a "mathematical thumbprint" that proves to a third party that the message must have originated from her.

A related, opposing property is *deniability* (or "plausible deniability"). Informally, deniability means that even if Alice provides a transcript of a conversation, there is no definitive cryptographic proof that Bob authored a particular message.

This often occurs in the _symmetric-key_ setting. Since Alice and Bob share the exact same secret key to generate authentication codes, Alice is technically capable of forging a message that looks exactly like one Bob would send. In that case, a third party (such as a judge) cannot be certain the message originated from Bob, because Alice could have created it as well. In this context, while the message has integrity (we know it wasn't changed by Mallory), it lacks non-repudiation because Bob can plausibly deny involvement.

## 4: Overview of Cryptographic Schemes

**Note**: At this stage, the goal is not to learn specific primitives or algorithms. Instead, this section gives you a roadmap of which primitives are typically used for which purpose. You will see the details and real protocol examples in the next topics.

In this unit, we will study cryptographic primitives, the basic building blocks that help us achieve three core security services in communication: confidentiality, integrity, and authentication. 
We will revisit these ideas in two settings:
- **Symmetric-key cryptography**, where Alice and Bob share the same secret key. Alice encrypts a message using the shared secret key, and Bob decrypts using the same key (Confidentiality). In addition, Alice can compute an authentication tag using the shared secret key, and Bob verifies it using the same key. If verification succeeds, Bob gains confidence that the message was not modified in transit and that it came from someone who knows the shared secret (Integrity & authentication).

- **Asymmetric-key (public-key) cryptography**, where a participant has a public/private key pair. In a public-key system, each participant has a *matched key pair*: a *public key* and a *private key*. These keys are constructed so that an operation performed with one key can be correctly “undone” or *checked* only with the *other* key (depending on the primitive). Bob generates a *public key* and a *private key*. He publishes the public key and keeps the private key secret. Alice encrypts to Bob using Bob’s public key, and only Bob can decrypt using his private key (Confidentiality).
In addition, Alice can sign a message using her *private key*, and anyone (including Bob) can verify the signature using Alice’s *public key*. If verification succeeds, Bob gains confidence that the message was not altered and that it was signed by the holder of Alice’s private key (Integrity & authentication (digital signatures)).

Importantly, public-key encryption and digital signatures are different schemes with different security goals. So you should not think of “encrypting with the private key” as a way to achieve confidentiality, since anyone who has your public key could reverse the operation and recover the message.

Symmetric-key techniques are typically efficient and are widely used for protecting large volumes of data, while public-key techniques make it possible to establish security and identity without first sharing a secret in advance. Real systems often combine both approaches.

## 5. Definitions: Threat models

When we claim that an encryption scheme is “secure”, we must be explicit about the *threat model*, meaning what Eve can observe and what interactions Eve is allowed to have with the system. Different levels of attacker access lead to different kinds of attacks and different security guarantees. 
Below are common threat models used when analyzing *confidentiality*. They are listed roughly from weaker to stronger attacker capabilities.

1. Eve has managed to intercept a single encrypted message and wishes to recover the plaintext (the original message). This is known as a _ciphertext-only attack_.

2. Eve has intercepted an encrypted message and also already has some partial information about the plaintext, which helps with deducing the nature of the encryption. This case is a _known plaintext attack_. In this case Eve's knowledge of the plaintext is partial, but often we instead consider complete knowledge of one instance of plaintext.

3. Eve can capture an encrypted message from Alice to Bob and re-send the encrypted message to Bob again. This is known as a _replay attack_. For example, Eve captures the encryption of the message "Hey Bob's Automatic Payment System: pay Eve \$$100$" and sends it repeatedly to Bob so Eve gets paid multiple times. Eve might not know the decryption of the message, but she can still send the encryption repeatedly to carry out the attack.

4. Eve can trick Alice to encrypt arbitrary messages of Eve's choice, for which Eve can then observe the resulting ciphertexts. (This might happen if Eve has access to the encryption system, or can generate external events that will lead Alice to sending predictable messages in response.) At some other point in time, Alice encrypts a message that is unknown to Eve; Eve intercepts the encryption of Alice's message and aims to recover the message given what Eve has observed about previous encryptions. This case is known as a _chosen-plaintext attack_.

5. Eve can trick Bob into decrypting some ciphertexts. Eve would like to use this to learn the decryption of some other ciphertext (different from the ciphertexts Eve tricked Bob into decrypting). This case is known as a _chosen-ciphertext attack_.

6. A combination of the previous two cases: Eve can trick Alice into encrypting some messages of Eve's choosing, and can trick Bob into decrypting some ciphertexts of Eve's choosing. Eve would like to learn the decryption of some other ciphertext that was sent by Alice. (To avoid making this case trivial, Eve is not allowed to trick Bob into decrypting the ciphertext sent by Alice.) This case is known as a _chosen-plaintext/ciphertext attack_, and is the most serious threat model.

In practice, modern systems aim for very strong security (often including resistance to chosen-ciphertext attacks). In this course, we use chosen-plaintext security (CPA) as our baseline model for confidentiality because it is the simplest rigorous starting point. Later topics on integrity and authenticated encryption explain how real-world designs achieve stronger protections, and we will reference stronger models when they are relevant.

## 6. Disclaimer: Don’t Roll Your Own Crypto!

A famous rule in security engineering is: **don’t roll your own crypto**. In this unit, you will learn the core ideas and building blocks of cryptography enough to understand what the tools are designed to do, how they fit together, and why modern systems can (and sometimes cannot) achieve security.
That said, cryptography is notoriously easy to get wrong in practice. Real implementations must handle many details that are outside the scope of an introductory unit, including secure randomness, key generation and storage, parameter choices, error handling, side-channel leakage, safe composition of primitives, and interoperability. A design that looks correct on paper can fail due to a subtle implementation bug, an unexpected interaction between components, or a flawed operational assumption. For these reasons, you should *not implement your own cryptographic algorithms* based solely on what you learn here.

Instead, you should be able to understand what a tool is *supposed* to provide (confidentiality, integrity, authentication), what assumptions it relies on, and how to compare alternatives at a high level. As an scientist/engineer working in secure environments, you should learn when to rely on established, well-vetted cryptographic libraries and protocols, how to read security documentation critically, and how to recognize red flags such as “custom crypto”, hidden algorithms, or vague security claims.

Cryptographic primitives also appear outside traditional security use cases (for example, producing fingerprints of data or generating values that look random). Even then, the safe approach is the same: *use standard, widely reviewed tools* and treat “rolling your own” cryptography as a serious risk.

In short, this unit will teach you enough cryptography to understand and assess modern systems, but not enough to responsibly build industrial-strength cryptography from scratch. That is by design.

[^1]: The story of the Enigma gives one possible justification for this assumption: given how widely the Enigma was used, it was inevitable that sooner or later the Allies would get their hands on an Enigma machine, and indeed they did.