---
title: Quantum Cryptography
parent: Emerging Topics in Security
nav_order: 4
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Quantum Cryptography

## Introduction

In previous units you learned how public-key cryptography (such as RSA, Elliptic Curve Cryptography, and Diffie-Hellman) works together with symmetric cryptography to protect individual messages and establish secure communication channels. These primitives form the foundation of real-world protocols such as TLS, SSH, and certificate-based authentication. They currently provide the confidentiality, integrity, and authentication that make secure communication across the modern Internet possible.

All of this security ultimately rests on a critical assumption: certain mathematical problems are computationally hard for classical computers to solve.
Unfortunately, this assumption does not hold for sufficiently powerful quantum computers.

One of the most important quantum algorithms for cryptography is **Shor’s algorithm**. It efficiently solves two mathematical problems that are believed to be extremely difficult for classical computers: integer factorization and the discrete logarithm problem.

On a classical computer, these problems are believed to be computationally hard. The best-known classical algorithms require an exponentially long time to factor very large numbers (such as those used in RSA-2048). Shor’s algorithm, however, runs in polynomial time on a sufficiently powerful quantum computer. This means that once a large-scale, fault-tolerant quantum computer becomes available, it can break RSA, Elliptic Curve Cryptography (ECC), and Diffie-Hellman key exchange in a practical amount of time. Digital signatures based on these algorithms can also be forged.

In simple terms, Shor’s algorithm turns what we currently consider “impossible to break in reasonable time” into something that becomes feasible with the right quantum hardware.

Another important quantum algorithm, **Grover’s algorithm**, provides a quadratic speedup for searching unsorted databases.
This weakens symmetric ciphers and cryptographic hash functions, although the impact is less severe.
Doubling key sizes (for example, moving from AES-128 to AES-256) largely mitigates Grover’s attack.
Public-key cryptography, however, has no such simple fix.

Although large-scale, cryptographically relevant quantum computers capable of breaking RSA-2048 are still likely 10–20 years away (with estimates varying widely), there are two strong reasons to begin preparing for quantum-resistant cryptography today.

First, nation-state adversaries and well-resourced attackers can already record large volumes of encrypted network traffic and store it for future decryption. This strategy is known as the **“Harvest Now, Decrypt Later”** attack. Once a sufficiently powerful quantum computer becomes available, all previously recorded traffic can be decrypted. This threat is especially serious for data that must remain confidential for many years, including medical and genomic records, government and military communications, intellectual property and trade secrets, long-term financial records and contracts, and personal communications that could be used for blackmail decades later.

Second, migrating cryptographic infrastructure takes a long time. Updating protocols, replacing certificates, re-issuing keys, and maintaining backward compatibility across global systems often requires many years of coordinated effort. Data encrypted today may still need strong protection well into the 2040s.

Two main families of solutions are being developed to address these challenges:

- **Quantum Key Distribution (QKD)** uses the fundamental laws of quantum mechanics to distribute symmetric keys in a way that is information-theoretically secure. Any attempt to eavesdrop on the quantum channel disturbs the quantum states and can be detected by the legitimate parties.
- **Post-Quantum Cryptography (PQC)** consists of new classical algorithms (software-based) that are believed to resist attacks from both classical and quantum computers. These algorithms can run on today’s hardware and networks without requiring specialized quantum equipment.

These two approaches are complementary rather than competing. Many organizations are planning **hybrid** deployments that combine classical cryptography, post-quantum algorithms, and (where practical) quantum key distribution to provide defense-in-depth during the transition period.

## Background: Quantum Computing Basics (Just Enough Physics)

Before we can understand how Quantum Key Distribution works, we need a small amount of background on quantum mechanics.

In classical computing, a **bit** is the basic unit of information. It can be either `0` or `1`. Once set, it stays in that state.

A **qubit** (quantum bit) is different. Thanks to the principle of **superposition**, a qubit can exist in a combination of both `0` and `1` at the same time, until it is measured. You can think of it like a spinning coin in the air: it is not heads _or_ tails, but in a state that contains both possibilities simultaneously.

This ability to be in multiple states at once gives quantum computers their enormous potential power for certain problems (such as factoring large numbers).

### Measurement and the Observer Effect

Here is the crucial security-relevant property: when you **measure** a qubit, it collapses into a definite classical state, either `0` or `1`. The act of measurement disturbs the qubit.

You cannot observe a qubit without changing it.

This is fundamentally different from classical bits. In a classical network, an eavesdropper can copy packets or read bits without the sender or receiver noticing. In the quantum world, any attempt to observe the information disturbs the system in a detectable way.

**Analogy**: Imagine a special envelope that contains a secret message written in invisible ink. If you open the envelope to read it, the ink becomes visible and the message is permanently altered. Anyone who later receives the envelope can tell that it has been opened.

### The No-Cloning Theorem

Another fundamental law of quantum mechanics is the **No-Cloning Theorem**. It states that it is impossible to make a perfect copy of an unknown quantum state.

In classical networking, an attacker can easily copy packets, duplicate bits, or sniff traffic without altering the original. In quantum communication, you cannot make an identical copy of a qubit without knowing its state in advance. Any attempt to copy it introduces detectable errors.

This theorem is one of the cornerstones that makes Quantum Key Distribution possible.

### Polarized Photons — The Practical Carrier

Most practical Quantum Key Distribution systems use **photons** (individual particles of light) rather than idealized laboratory qubits. These photons naturally exhibit quantum properties such as superposition and the fact that any measurement disturbs their state. They are transmitted through optical fiber cables, and in some advanced systems, through free space using satellites.

Photons can be polarized in different ways. The two most commonly used bases in QKD are:

- **Rectilinear basis**: Horizontal (↔) or Vertical (↕)
- **Diagonal basis**: 45° (↗) or 135° (↘)

**Polarization** refers to the direction in which the electric field of a light wave oscillates as it travels. To understand this, imagine shaking one end of a rope lying on the ground. If you move your hand up and down, the wave travels forward but vibrates vertically. If you move your hand side to side, the wave vibrates horizontally.

In the same way, a **horizontally polarized** photon has its electric field oscillating left and right, while a **vertically polarized** photon has its electric field oscillating up and down. Photons can also be polarized at other angles, such as 45° or 135°.

## Quantum Key Distribution (QKD) — The BB84 Protocol

Now that we have the necessary quantum background, we can examine the most important and widely deployed Quantum Key Distribution protocol: **BB84**, named after its inventors Charles Bennett and Gilles Brassard who proposed it in 1984.

**Quantum Key Distribution (QKD)** allows two parties, **Alice** (the sender) and **Bob** (the receiver), to generate and agree on a shared secret symmetric key over an insecure quantum channel, while detecting any eavesdropping by a third party (**Eve**).

Unlike classical key exchange protocols such as Diffie-Hellman (which rely on computational hardness), QKD provides _information-theoretic security_ which comes directly from the laws of quantum mechanics.

### How BB84 Works — Step by Step

The BB84 protocol consists of several phases, mixing quantum and classical communication:

1. **Quantum Transmission (Alice → Bob)**  
   Alice generates a random sequence of bits (raw key material) and a random sequence of bases (rectilinear or diagonal).
   For each bit, she encodes it into the polarization of a photon according to her chosen basis and sends the photon to Bob over a quantum channel (typically optical fiber).

2. **Measurement (Bob)**  
   For each incoming photon, Bob randomly selects a measurement basis (rectilinear or diagonal) and measures the photon’s polarization. He records both the chosen basis and the measurement result.

3. **Basis Reconciliation (Classical Channel)**  
   Alice and Bob communicate over a public classical channel and announce which basis they used for each photon.
   They discard all bits where their bases did not match. This process is called **sifting**.
   The classical channel is assumed to be authenticated (so Eve cannot modify messages undetected), but it can be passively eavesdropped.

4. **Error Estimation**  
   Alice and Bob publicly compare a random subset of their sifted bits to estimate the **Quantum Bit Error Rate (QBER)**.
   If the error rate exceeds a certain threshold, they abort the protocol, as this may indicate the presence of an eavesdropper.

5. **Key Distillation (Error Correction and Privacy Amplification)**  
   If the error rate is acceptable, Alice and Bob perform **error correction** to reconcile their sifted keys into identical strings.
   They then apply **privacy amplification** (using techniques such as universal hashing) to reduce any information that Eve may have gained, resulting in a shorter but highly secret final key.

### Simple Example Walkthrough

To understand how BB84 works in practice, let’s go through a concrete example with 8 photons.
Suppose Alice wants to establish a key with Bob. We will define the encoding as follows:

- **Rectilinear basis (R)**:  
  Horizontal (↔) = **0**, Vertical (↕) = **1**

- **Diagonal basis (D)**:  
  45° (↗) = **0**, 135° (↘) = **1**

Alice wants to send the following random bits:  
**Alice’s bits**: `0 1 0 1 1 0 1 0`

She also randomly chooses a basis for each bit:  
**Alice’s bases**: `R D R D R D R D`

She encodes each bit into the polarization of a photon and sends it to Bob.

Bob randomly chooses his own measurement bases:  
**Bob’s bases**: `R R D D R D D R`

After measuring the photons, Bob obtains the following results:  
**Bob’s measurements**: `0 0 1 1 1 0 0 1`

Here is the full process:

| Photon | Alice’s Bit | Alice’s Basis | Polarization Sent | Bob’s Basis | Bob’s Measurement | Basis Match? | Sifted Bit |
| ------ | ----------- | ------------- | ----------------- | ----------- | ----------------- | ------------ | ---------- |
| 1      | 0           | R             | ↔                 | R           | 0                 | **Yes**      | 0          |
| 2      | 1           | D             | ↗                 | R           | 0                 | No           | Discard    |
| 3      | 0           | R             | ↔                 | D           | 1                 | No           | Discard    |
| 4      | 1           | D             | ↘                 | D           | 1                 | **Yes**      | 1          |
| 5      | 1           | R             | ↕                 | R           | 1                 | **Yes**      | 1          |
| 6      | 0           | D             | ↗                 | D           | 0                 | **Yes**      | 0          |
| 7      | 1           | R             | ↕                 | D           | 0                 | No           | Discard    |
| 8      | 0           | D             | ↘                 | R           | 1                 | No           | Discard    |

After all photons are sent and measured, Alice and Bob **publicly announce** the basis they used for each photon over a classical channel (they do **not** reveal the bit values yet). They then discard all photons where their bases did not match. This process is called **sifting**.

In this example, they keep photons 1, 4, 5, and 6. Their **sifted keys** are now:

- Alice: `0 1 1 0`
- Bob: `0 1 1 0`

To check whether an eavesdropper was present, Alice and Bob **publicly compare a random subset** of their sifted bits.
For example, they might agree to reveal the values of photons 1 and 5. If these bits match, the error rate in the sample is low.
If many bits disagree, they conclude that an eavesdropper (Eve) likely interfered and they abort the protocol.

If the error rate is acceptably low, they proceed with **error correction** (to fix any remaining differences) and **privacy amplification** (to remove any information Eve might have obtained), resulting in a final secure key.

### How Eavesdropping is Detected

The security of BB84 relies on the fact that any measurement by Eve disturbs the quantum states:

- If Eve measures a photon in the wrong basis, she introduces errors.
- Because of the no-cloning theorem, she cannot copy the photon perfectly and forward an undisturbed version to Bob.
- When Alice and Bob compare a sample of their bits, they will detect a higher-than-expected error rate if Eve was listening.

This is fundamentally different from classical networks. In classical cryptography, passive eavesdropping is undetectable. In QKD, **any eavesdropping is detectable**.

## Security Properties and Proofs of QKD

The real power of Quantum Key Distribution lies not just in how it works, but in **what it guarantees**.
Unlike every classical key exchange protocol, QKD offers a fundamentally different kind of security.

**Quantum Key Distribution** provides **information-theoretic security** (also called unconditional security). Its security does not depend on unproven mathematical assumptions or limits on the attacker’s computing power. Instead, it is based directly on the laws of quantum mechanics. Even an adversary with unlimited computational resources and perfect technology cannot break the protocol without being detected.

This is a major philosophical and practical leap forward for network security.

BB84 delivers three powerful guarantees:

1. **Detectable Eavesdropping**  
   Any attempt by Eve to measure the quantum states disturbs them.
   Because of the observer effect and the no-cloning theorem, she cannot gain useful information without introducing errors into the channel.

2. **Bounded Information Leakage**  
   Alice and Bob can accurately estimate how much information Eve might have obtained by measuring the Quantum Bit Error Rate (QBER).
   If the error rate is low enough, they know Eve’s knowledge is limited.

3. **Privacy Amplification**  
   Through classical post-processing, Alice and Bob can shrink their raw key into a much shorter final key from which Eve has essentially zero information.

If the protocol completes successfully, Alice and Bob share a secret key that is secure against **any** eavesdropper, classical or quantum.

### Important Assumptions and Limitations

The theoretical security proofs for protocols such as BB84 rely on several idealized assumptions that are difficult to achieve in real hardware:

- **Perfect single-photon sources**: The sender must emit exactly one photon per pulse. In practice, most sources occasionally emit multiple photons, which can leak information to an eavesdropper.
- **Perfect detectors with no dark counts**: Detectors must register photons with 100% efficiency and produce no false detections when no photon arrives.
- **No side-channel attacks**: The implementation must not leak information through timing, power consumption, electromagnetic emissions, or other physical characteristics of the hardware.
- **An authenticated classical channel**: Alice and Bob need a way to authenticate their classical communication (for basis reconciliation, error correction, and privacy amplification). Without authentication, an attacker could perform a man-in-the-middle attack on the classical channel.

In practice, the last point is especially important: QKD still needs a way for Alice and Bob to authenticate their classical messages.
This is usually achieved with pre-shared keys or post-quantum signatures.

## Practical Challenges, Limitations, and Attacks on Real QKD Systems

While the theoretical security of protocols like BB84 is extremely strong, real-world Quantum Key Distribution systems face significant engineering and security challenges that limit their practicality today.

- **Distance and Signal Loss**  
  The biggest practical limitation of QKD is distance. Photons traveling through optical fiber gradually lose intensity due to absorption and scattering (attenuation). Because of the no-cloning theorem, quantum signals cannot be amplified like classical signals without destroying their quantum properties. As a result, current commercial fiber links are typically limited to _50–150 km_ before the error rate becomes too high to generate usable secret keys. Longer distances currently require trusted relay nodes or satellite-based links. This makes QKD well-suited for metro-area or campus networks, but difficult to scale for global communication without additional infrastructure.

- **Trusted Nodes Problem**  
  To extend QKD over long distances, current networks rely on _trusted relay nodes_. Each node receives the key from the previous segment and forwards it to the next. While this extends range, it breaks true end-to-end security: if any intermediate node is compromised, the attacker obtains the full key. This introduces new points of trust (and potential attack) into the network and is one of the most criticized aspects of today’s QKD deployments.

- **Hardware Requirements and Cost**  
  QKD systems require specialized and expensive hardware that is far more demanding than classical networking equipment. Most practical systems use attenuated lasers rather than true single-photon sources (which can occasionally emit multiple photons). They also need high-precision single-photon detectors, often superconducting nanowire detectors that must be cooled to cryogenic temperatures. Maintaining stable optical alignment and polarization control adds further complexity. As a result, QKD systems are bulky, power-hungry, and significantly more expensive than traditional network hardware. Deployment is currently justified only for high-value links (such as government, military, banking, or critical infrastructure) where the security requirements outweigh the cost.

- **Side-Channel and Implementation Attacks**  
  Real QKD systems are vulnerable to attacks that exploit weaknesses in the physical implementation rather than flaws in the protocol itself. Researchers have demonstrated several practical attacks, including:
  - _Photon Number Splitting (PNS)_ attacks that exploit multi-photon pulses from imperfect sources.
  - _Detector Blinding Attacks_, where bright light is used to blind Bob’s detectors and force predictable behavior.
  - _Trojan Horse Attacks_, in which an attacker injects light into Alice’s or Bob’s equipment to learn internal settings.
  - _Timing side-channel attacks_ that analyze small differences in detector response times.

## Real-World Deployments

Quantum Key Distribution has moved beyond laboratory experiments. Several countries and organizations have built operational quantum communication networks, though these are still limited in scale and primarily used for high-security applications.

Current quantum networks are typically built in one of three ways:

- **Point-to-Point Links**
  The simplest and most secure configuration. Alice and Bob are directly connected by a dedicated optical fiber. This approach provides the strongest security guarantees but is limited by distance (typically up to around 100–150 km in fiber).

- **Trusted-Node Networks**
  To cover longer distances, networks use intermediate trusted relay nodes. Each node receives a key from the previous segment and generates a new key for the next segment. While this extends the reachable distance, it introduces points of trust: compromising any intermediate node can expose the entire key.

- **Satellite-Based QKD (Free-Space Links)**
  Satellites can establish quantum links over very long distances with relatively low loss. China’s Micius satellite famously demonstrated this by distributing keys between ground stations more than 2,000 km apart.

The most advanced large-scale quantum network currently in operation is in **China**:

- A fiber backbone exceeding 2,000 km connects Beijing and Shanghai, using multiple trusted nodes along the route.
- The Micius quantum satellite has successfully performed QKD with multiple ground stations across China and has even enabled intercontinental key distribution experiments.
- These networks are primarily used for government communications and the exchange of highly sensitive financial data.

Smaller quantum network testbeds and pilot projects also exist in Europe (notably through the EuroQCI initiative), Japan, South Korea, and the United States.
At present, most real-world QKD deployments focus on high-security environments such as government, defense, and critical infrastructure rather than general commercial or consumer internet traffic.

## Post-Quantum Cryptography (PQC)

While Quantum Key Distribution offers elegant, physics-based security, its practical limitations make it difficult to deploy widely today.
The more immediate and scalable solution for most networks is **Post-Quantum Cryptography (PQC)**.

Post-Quantum Cryptography refers to public-key algorithms that are believed to be secure even against large-scale quantum computers running Shor’s algorithm.
Unlike QKD, PQC requires no new hardware or fiber infrastructure.
It runs on existing CPUs, works over the current Internet, and can be deployed through software updates.

PQC focuses on two main needs:

- **Key Encapsulation Mechanisms (KEMs)** which is used for secure key exchange (replacing Diffie-Hellman)
- **Digital Signature Algorithms** which is used for authentication and certificates (replacing RSA and ECDSA)

In 2016, NIST launched a global competition to standardize post-quantum algorithms.
After several rounds of public review, the first standards were announced in 2024:

- **ML-KEM (Kyber)**: A lattice-based key encapsulation mechanism, selected for general encryption and key exchange.
- **ML-DSA (Dilithium)**: A lattice-based digital signature scheme, selected as the primary signature algorithm.
- **SLH-DSA (SPHINCS+)**: A hash-based signature scheme, selected as a backup with very strong security assumptions.

These algorithms are now being integrated into protocols and libraries.

The recommended migration strategy is **hybrid cryptography**: using both a classical algorithm and a post-quantum algorithm together.
This protects against both today’s threats and future quantum computers.

### Current Recommendations

Most security experts and standards bodies recommend the following for network operators:

- Begin testing PQC algorithms in non-production environments now.
- Implement hybrid key exchange in TLS and VPNs as soon as libraries support it.
- Plan a gradual migration of certificates and long-lived keys.
- Monitor NIST and IETF developments for updated guidance.
- Use QKD only for specific high-value links where its unique properties justify the cost.
