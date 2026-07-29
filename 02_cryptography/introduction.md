---
title: Introduction to Cryptography
parent: Cryptography
nav_order: 1
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Introduction to Cryptography

## Brief History of Cryptography

The word _cryptography_ comes from the Greek roots _kryptos_ (hidden) and _graphein_ (to write). At its core, cryptography is the study of techniques for secure communication in the presence of adversaries.

### From Classical Ciphers to the Telegraph Era

Techniques for secret communication are ancient. One of the best-known early examples is the _Caesar cipher_, attributed to Julius Caesar, which encrypts text by shifting each letter a fixed number of positions in the alphabet. With a shift of 3, for instance, the message “NETWORK” becomes “QHWZRUN”. Although trivial by modern standards, such substitution ciphers already embody a central goal of cryptography: making intercepted messages unintelligible to unauthorized parties.

Cryptography gained new importance in the nineteenth century with the rise of long-distance electronic communication, especially the telegraph. Military and diplomatic organizations needed reliable ways to protect messages that traveled over infrastructure that could be monitored. Because encryption and decryption were still performed by hand, many “pen-and-ink” systems remained relatively simple. Practical for human operators, but also more vulnerable to well-resourced codebreakers and pattern-based attacks.

### The Mechanical Era and the Enigma Story

The next major shift came with the _mechanical era_, when engineers began building machines to automate encryption. The most famous example is the German _Enigma_ machine used during World War II. Enigma greatly increased the complexity and speed of encryption, making it practical to protect large volumes of military traffic.

Breaking Enigma required a combination of mathematical insight, engineering, and large-scale organization.

First, earlier breakthroughs by Polish cryptanalysts (who had already made headway against earlier Enigma procedures) and the subsequent transfer of knowledge and materials (including access to Enigma-related components and replicas) provided a valuable foundation for the later British effort.

Second, the work drew on deep mathematical and analytical expertise, including contributions by figures such as _Alan Turing_. Turing’s work on the _Bombe_ (an electromechanical system used to help determine Enigma settings) helped narrow the space of possible configurations and made the search for the correct key feasible in practice. His contributions also helped shape ideas that influenced modern computing, and he is widely regarded as one of the founding fathers of computer science.

Third, Bletchley Park scaled the operation to an industrial level: at its height, the British codebreaking effort grew to more than 10,000 personnel and coordinated workflows that combined human expertise with mechanized assistance, enabling many searches to run in parallel, the processing of enormous volumes of intercepted traffic, and the exploitation of weaknesses introduced by real-world procedures and repeated patterns.

A lasting lesson from this era is that cryptographic strength depends not only on the design of the cipher itself, but also on how the system is used, what information an attacker can obtain, and how much effort the attacker is willing and able to invest.

### Modern Cryptography: Mathematics, Computers, and Security Theory

What we now call _modern cryptography_ is distinguished by its strong reliance on mathematical foundations and computer-based implementation. After World War II, _Claude Shannon_ played a foundational role by formalizing ideas about secrecy and information, including analysis related to the _one-time pad_ (a scheme notable for achieving perfect secrecy under the right assumptions which we will cover in this unit).

As computers became central to government and commerce, the need for standardized, interoperable encryption grew. In the 1970s the U.S. National Bureau of Standards (now NIST) published the Data _Encryption Standard (DES)_ to meet practical needs in areas like banking and commercial data protection. From the late 1970s onward, cryptography rapidly matured into a rigorous scientific field with precise security models and proofs. The tools and concepts developed in this period (e.g., symmetric encryption, public-key cryptography, cryptographic hash functions, and formal notions of security) form the foundation of the systems that protect today’s networks and applications.

## Definitions

At first glance it is easy to see why the Caesar cipher is weak: there are only 26 possible shifts, so an attacker can simply try them all.
But _how do we argue this rigorously_, and more generally, how do we reason about whether a cryptographic scheme is secure?

To do so we need a precise framework. We must define the participants, specify what an attacker is allowed to do, and state clearly what it means for the attacker to succeed.

The rest of this section introduces the core terms we will use throughout the unit.

### Alice, Bob, Eve, and Mallory

The most basic problem in cryptography is securing communication over an _insecure channel_. We typically describe this using a standard cast of characters. _Alice_ and _Bob_ are the parties who want to communicate securely, as if they were in the same room or connected by a private, untappable link. In real systems, Alice and Bob are not necessarily people. For example, Alice could be a web browser or mobile app, and Bob could be a web server or API.

Unfortunately, Alice and Bob must communicate over a channel such as a telephone line or the Internet that an adversary may be able to access. A passive attacker is traditionally called _Eve_, the _eavesdropper_, who can listen to or record traffic but does not modify it (for example, someone within the same Wi‑Fi range capturing wireless packets). In other settings, we consider an active attacker called _Mallory_, who can do everything Eve can _and also_ tamper with communication (e.g., modify, inject, replay, or block messages). For instance, Mallory might be an entity positioned “on the path”, such as a compromised router, a malicious access point, or an operator with privileged access to networking infrastructure (e.g., an ISP-level device). In this sense, Mallory is a _superset_ of Eve: an active attacker includes all passive capabilities plus additional control over the communication channel.

The goal of cryptographic design is to make the insecure channel behave as if it were a private, untappable link.
The goal is to “scramble” communication so that:

- Eve learns nothing useful about the message contents, and
- Mallory cannot alter the communication without the change being detected.

In other words, we aim to simulate an ideal communication channel using only the insecure channel that is actually available.

### Keys

Almost every cryptographic system is controlled by a _key_. A key is a secret value that parameterizes a cryptographic algorithm. Modern practice assumes that the algorithms themselves are public; security rests on the secrecy of the key.

There are two main models.
In the _symmetric-key_ model, Alice and Bob share a single secret key and use it for both protecting and verifying messages.
In the _asymmetric_ (public-key) model, each party possesses a _key pair_ consisting of a private key (kept secret) and a corresponding public key (which may be distributed openly). We will study both models in detail later in this unit.

### Kerckhoffs’s Principle

A foundational assumption of modern cryptography is _Kerckhoffs’s principle_: a cryptosystem should remain secure even if everything about the system, except the secret key, is known to the attacker. Security must not depend on the secrecy of the algorithm or its implementation details. This is the cryptographic expression of the broader design principle “do not rely on security through obscurity”.

Throughout this unit we therefore assume that adversaries know the algorithms, the protocols, and how the system is built. The only information they lack is the secret key (or keys).

## Confidentiality, Integrity, Authenticity

In the previous unit you learned the classic security goals of confidentiality, integrity, and availability (the CIA triad). In this cryptography unit we focus on how cryptographic tools help achieve _confidentiality_ and _integrity_, and how they support _authentication_ (a goal closely related to integrity).

Cryptography provides these services through a small set of basic building blocks called _cryptographic primitives_. You will study the primitives themselves in later topics. Here we first clarify the security goals they are designed to achieve.

### Confidentiality

_Confidentiality_ means that an adversary who observes a message should learn nothing useful about its content. In other words, even if an attacker can see the data traveling over the network, the data should remain unintelligible.

A useful mental model is a locked box. Alice places her message inside the box, locks it, and sends the locked box over an insecure channel. Eve can see the box and even make a perfect copy of it, but without the correct key she cannot open it. Bob, who possesses the key, can unlock the box and read the original message.

In modern systems the locked box is realized by _encryption_. The original readable message is called _plaintext_. Encryption transforms the plaintext into _ciphertext_, which should appear random and meaningless to anyone who does not hold the appropriate key. Decryption reverses the process and recovers the plaintext. The security of the scheme rests on the secrecy of the key, not on keeping the encryption algorithm itself secret.

### Integrity

_Integrity_ means that an adversary should not be able to modify data without the change being detected. If a message has integrity protection, the receiver can determine whether the message arrived exactly as it was sent, or whether it was altered in transit (accidentally or maliciously).

Integrity is especially important against an active adversary (Mallory), who may try to change messages, inject new ones, or replay old ones. Without integrity protection, even a perfectly confidential channel can be manipulated in dangerous ways.

### Authenticity

_Authenticity_ (or data-origin authentication) means the receiver can verify _who_ created a message. If a message is authentic, Bob can be confident that it originated from the claimed sender (Alice) rather than from an attacker.

Authenticity and integrity are closely related. It is difficult to trust that a message “came from Alice” unless you also have a reliable way to detect whether the message was altered after she created it. In practice, the mechanisms that provide authenticity almost always provide integrity as well. The two properties are not identical, however, and later topics will show situations in which the distinction matters.

A helpful physical analogy is a tamper-evident seal on an envelope. Alice applies a special seal before sending the message. If Mallory opens or alters the envelope, the seal is broken and the tampering becomes visible. Without the correct secret (or private key), Mallory cannot produce a new seal that will pass inspection.

In cryptographic terms, Alice attaches a short value, commonly called a _tag_ or a _digital signature_, that Bob can verify. If the message has been modified, verification fails and Bob rejects the message. An attacker who does not possess the appropriate key should be unable to forge a valid tag or signature on a message of their choosing.

### Non-Repudiation versus Deniability

In addition to confidentiality, integrity, and authenticity, two related properties are often discussed: _non-repudiation_ and _deniability_.

_Non-repudiation_ means that a sender cannot later deny having created a particular message. Digital signatures provide this property. Because only Alice knows her private key, a valid signature on a message serves as strong evidence that the message originated from her. A third party (for example, a judge) can verify the signature using Alice’s public key and be convinced that Alice produced it.

The opposite property is _deniability_ (sometimes called plausible deniability). In many symmetric-key settings, Alice and Bob share the same secret key used to compute authentication tags. Because either party could have created a valid tag, a third party cannot be sure which of the two actually produced a given message. Bob can therefore plausibly deny authorship: Alice could have forged the message herself. The message still has integrity (Mallory could not have altered it undetected), but it lacks non-repudiation.

In short, digital signatures give non-repudiation, while symmetric message authentication codes typically provide authenticity and integrity without non-repudiation.

## Overview of Cryptographic Schemes

**Note.** The goal of this section is not to study specific algorithms. It is only to give you a high-level roadmap of which cryptographic tools are typically used for which security goals. Detailed definitions, constructions, and real protocol examples appear in later topics.

Cryptographic primitives are the basic building blocks that let us achieve confidentiality, integrity, and authenticity. We study them in two main settings: symmetric-key cryptography and public-key (asymmetric) cryptography.

### Symmetric-Key Cryptography

In the symmetric-key setting, Alice and Bob share a single secret key in advance.

- **Confidentiality.** Alice encrypts a message with the shared key; Bob decrypts it with the same key. Anyone who does not know the key should be unable to recover the plaintext.
- **Integrity and authenticity.** Alice can also compute a short authentication tag (a Message Authentication Code, or MAC) using the shared key. Bob verifies the tag with the same key. Successful verification gives Bob confidence that the message was not modified and that it was produced by someone who knows the secret key.

Symmetric-key algorithms are generally fast and are the workhorses for protecting large amounts of data.

### Public-Key (Asymmetric) Cryptography

In the public-key setting, each participant has a matched _key pair_: a public key that can be distributed openly and a private key that must be kept secret. The two keys are mathematically linked so that an operation performed with one key can be inverted or verified only with the other.

- **Confidentiality.** To send a confidential message to Bob, Alice encrypts it under Bob’s _public_ key. Only Bob, who possesses the corresponding private key, can decrypt it.
- **Integrity and authenticity (digital signatures).** Alice can sign a message with her _private_ key. Anyone who has Alice’s public key can verify the signature. Successful verification convinces the verifier that the message has not been altered and that it was signed by the holder of Alice’s private key.

It is important to keep the two public-key operations distinct. Encrypting with a private key does **not** provide confidentiality, because anyone who knows the corresponding public key can reverse the operation.

### Combining the Two Approaches

Symmetric-key techniques are efficient and well-suited to bulk data protection. Public-key techniques solve the difficult problem of establishing security and identity without first sharing a secret. In practice, almost all real systems combine both: public-key cryptography is used to authenticate parties and to establish a fresh symmetric key, after which the actual data is protected with fast symmetric algorithms.

## Threat Models for Confidentiality

When we say that an encryption scheme is “secure”, we must be explicit about the _threat model_: what the adversary is allowed to observe and what interactions the adversary may have with the system. Different levels of attacker power lead to different attack models and different security guarantees.

Below are the classic threat models used when analyzing the confidentiality of encryption schemes.
They are ordered roughly from weaker to stronger attacker capabilities.

1. **Ciphertext-only attack**  
   The adversary sees only ciphertext and tries to recover the corresponding plaintext (or the key). This is the weakest and most basic attack model.

2. **Known-plaintext attack**  
   The adversary obtains some plaintext–ciphertext pairs (for the same key) and tries to decrypt other ciphertexts encrypted under that key, or to recover the key itself.

3. **Chosen-plaintext attack (CPA)**  
   The adversary can choose arbitrary plaintexts and obtain the corresponding ciphertexts (for example, by tricking Alice into encrypting messages of the adversary’s choice). Later the adversary is given a new ciphertext and tries to recover its plaintext. Resistance to chosen-plaintext attacks is a fundamental requirement for modern encryption schemes.

4. **Chosen-ciphertext attack (CCA)**  
   The adversary can also submit ciphertexts of its choice and receive the corresponding plaintexts (with the usual restriction that it may not ask for the decryption of the particular challenge ciphertext it is trying to break). This is a stronger and more realistic model for many practical settings.

5. **Chosen-plaintext/ciphertext attack**  
   The adversary can both obtain encryptions of chosen plaintexts and obtain decryptions of chosen ciphertexts. This is the most powerful of the classical models.

A related but distinct attack is the _replay attack_, in which the adversary simply re-sends a previously observed ciphertext. Replay does not require breaking the encryption itself. It exploits the lack of freshness or message uniqueness in a protocol. We will return to replay protection when we discuss integrity and authenticated encryption.

In this course we take resistance to chosen-plaintext attacks (CPA security) as the basic baseline for confidentiality. Later topics on message authentication and authenticated encryption show how real systems achieve still stronger guarantees, including protection against chosen-ciphertext attacks and active tampering.

## Don’t Roll Your Own Crypto!

A famous rule in security engineering is: **don’t roll your own crypto**. In this unit, you will learn the core ideas and building blocks of cryptography enough to understand what the tools are designed to do, how they fit together, and why modern systems can (and sometimes cannot) achieve security.
That said, cryptography is notoriously easy to get wrong in practice. Real implementations must handle many details that are outside the scope of an introductory unit, including secure randomness, key generation and storage, parameter choices, error handling, side-channel leakage, safe composition of primitives, and interoperability. A design that looks correct on paper can fail due to a subtle implementation bug, an unexpected interaction between components, or a flawed operational assumption. For these reasons, you should _not implement your own cryptographic algorithms_ based solely on what you learn here.

Instead, you should be able to understand what a tool is _supposed_ to provide (confidentiality, integrity, authentication), what assumptions it relies on, and how to compare alternatives at a high level. As an scientist/engineer working in secure environments, you should learn when to rely on established, well-vetted cryptographic libraries and protocols, how to read security documentation critically, and how to recognize red flags such as “custom crypto”, hidden algorithms, or vague security claims.

Cryptographic primitives also appear outside traditional security use cases (for example, producing fingerprints of data or generating values that look random). Even then, the safe approach is the same: _use standard, widely reviewed tools_ and treat “rolling your own” cryptography as a serious risk.

In short, this unit will teach you enough cryptography to understand and assess modern systems, but not enough to responsibly build industrial-strength cryptography from scratch. That is by design.
