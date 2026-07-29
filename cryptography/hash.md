---
title: Cryptographic Hashes
parent: Cryptography
nav_order: 3
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Cryptographic Hashes

## Introduction to Cryptographic Hash Functions

A _cryptographic hash function_ is a function that takes an input of arbitrary length (a message, a file, a password, etc.) and produces a fixed-size output called a _hash value_, _hash_, _message digest_, or _digital fingerprint_.

No matter how long or short the input is (whether a single word, a few sentences, or an entire disk image), the hash function always returns a digest of the same predetermined length (for example, 256 bits). The same input always produces exactly the same hash value, but the hash value itself does not reveal the original input.

Intuitively, a cryptographic hash acts like a compact digital fingerprint of the data. Just as a human fingerprint is a short identifier that is extremely likely to be unique to one person, a good hash value is a short string that is extremely likely to be unique to one particular input. Changing even a single bit of the input produces an entirely different, unpredictable hash.

## Core Security Properties

Cryptographic hash functions are required to satisfy three main security properties. We denote a hash function by \(H\).

### Preimage Resistance (One-Way Property)

Given a hash value \(y\), it should be computationally infeasible to find any input \(x\) such that \(H(x) = y\).

In other words, it is easy to compute the hash of a given message, but it is hard to go backwards: recovering _any_ input that produces a given hash should be impractical. This is also called the _one-way property_.

### Second-Preimage Resistance

Given an input \(x\), it should be computationally infeasible to find a _different_ input \(x' \neq x\) such that \(H(x') = H(x)\).

Here the adversary already knows one valid input and is trying to find another input that collides with it. The starting point is fixed; the adversary cannot choose it freely.

### Collision Resistance

It should be computationally infeasible to find _any_ two distinct inputs \(x\) and \(x'\) such that \(H(x) = H(x')\).

In this case the adversary is free to choose both inputs. Finding even a single colliding pair should be impractical.

### Relationships Between the Properties

Collision resistance is the strongest of the three properties. If a hash function is collision-resistant, then it is also second-preimage resistant. The converse is not necessarily true: a function can be second-preimage resistant without being fully collision-resistant.

In practice, hash functions that provide collision resistance also provide preimage resistance, although there exist artificial counter-examples.

### What “Infeasible” Means

When we say a task is “infeasible”, we mean it is computationally infeasible: there is no known method that succeeds with non-negligible probability using a realistic amount of computing power and time. This is the notion of _computational security_. We do not require information-theoretic impossibility (security against an adversary with unlimited resources); we only require that the attack is impractical for any real-world adversary.

## Why These Properties Matter (Applications)

The three security properties of cryptographic hash functions are not abstract requirements; each is needed for concrete applications.

### Integrity Checking / Modification Detection

Hash functions are widely used to detect whether data has been altered. Suppose a software vendor publishes the hash of an installation file. A user who downloads the file can recompute the hash and compare it with the published value. If the hashes match, the file is extremely likely to be unmodified. Because of collision resistance (and second-preimage resistance), an attacker cannot substitute a malicious file that produces the same hash.

The same idea is used by operating systems and security tools that maintain an allow-list of known-good hashes of critical system files and periodically re-check them.

### Password Hashing

Instead of storing user passwords in plaintext, servers store the hash of each password. When a user logs in, the server hashes the password that was just entered and compares it with the stored hash. Preimage resistance is essential here: even if an attacker steals the password database, it should be infeasible to recover the original passwords from the hashes.

### Digital Signatures

Most digital signature schemes operate on fixed-size inputs. Signing a long message directly would be inefficient. In practice, one first computes a hash of the message and then signs the short hash value. Collision resistance is required: if an attacker could find two messages with the same hash, they might obtain a signature on one message and later claim it is a signature on the other.

### Commitment Schemes with Hash Functions

A _commitment scheme_ lets one party (the committer) commit to a value while keeping it secret, and later open the commitment to prove that the revealed value is the one that was originally chosen. A good commitment scheme has two essential properties:

- **Hiding**: the commitment itself reveals no information about the committed value.
- **Binding**: once the commitment has been published, the committer cannot open it to a different value.

Cryptographic hash functions give a simple and widely used way to build commitments. To commit to a message \(m\), the committer chooses a fresh random string \(r\) (called the _opening_ or _nonce_) and publishes

\[
c = H(m \,\|\, r).
\]

Later, to open the commitment, the committer reveals both \(m\) and \(r\). Anyone can recompute the hash and check that it matches the previously published \(c\). Because \(H\) is one-way and collision-resistant, the committer cannot find a different pair \((m', r')\) that produces the same hash, and an observer who sees only \(c\) learns nothing useful about \(m\).

#### Example: The Magician’s Prediction

A magician asks a volunteer to think of any number between 1 and 100 and to keep it secret. The magician then writes something on a piece of paper, folds it, and places it in plain view (this is the commitment). After the volunteer announces the chosen number, the magician opens the paper and shows that it contains exactly that number.

In cryptographic terms the magician did the following:

1. When the volunteer selected the number \(m\), the magician chose a random string \(r\) and computed \(c = H(m \,\|\, r)\).
2. The magician published (or displayed) only the commitment \(c\).
3. After the volunteer revealed \(m\), the magician opened the commitment by revealing \(r\), allowing everyone to verify that \(H(m \,\|\, r) = c\).

The hiding property prevents the audience from learning the number in advance, while the binding property prevents the magician from changing the prediction after hearing the volunteer’s choice.

### Other Uses

Hash functions appear in many additional places in security systems, including key derivation, proof-of-work constructions, and the construction of message authentication codes (MACs). In each case one or more of the three core properties is essential for security.

## Birthday Paradox and Collision Resistance

How many people must be in a room before it becomes likely that two of them share a birthday? Most people guess a number close to 365. The correct answer is only 23. At 23 people the probability of at least one shared birthday already exceeds 50 %. With 50 people the probability rises above 97 %.

The reason is that we are not looking for a match with one specific day; any matching pair counts. The number of possible pairs among \(n\) people grows roughly as \(n^2\), so collisions appear much sooner than intuition suggests.

The same mathematics applies to hash functions. Finding a second preimage for a _fixed_ input \(x\) requires trying on the order of \(2^n\) candidates for an \(n\)-bit hash. Finding _any_ colliding pair (collision resistance), however, is analogous to the birthday problem: an attacker needs only about \(2^{n/2}\) hashes before a collision is expected.

Consequently, collision resistance is substantially harder to achieve (or, equivalently, easier for an attacker to break) than second-preimage resistance. A hash function that is merely second-preimage resistant may still be vulnerable to birthday-style collision attacks.

### Practical Implication for Hash Output Length

Because of the birthday bound, the output length of a hash function must be chosen roughly twice as long as the security level one wishes to achieve against collision attacks. For example, to obtain a 128-bit security level against collisions one normally selects a 256-bit hash function (such as SHA-256). This is why modern systems prefer hashes with 256-bit or longer outputs.

## Hash Function Algorithms in Practice

Cryptographic hashes have evolved over time. One of the earliest hash functions, MD5 (Message Digest 5) was broken years ago. The slightly more recent SHA1 (Secure Hash Algorithm) was broken in 2017, although by then it was already suspected to be insecure. Systems which rely on MD5 or SHA1 actually resisting attackers are thus considered insecure. Outdated hashes have also proven problematic in non-cryptographic systems. The `git` version control program, for example, identifies identical files by checking if the files produce the same SHA1 hash. This worked just fine until someone discovered how to produce SHA1 collisions.

Today, there are two primary "families" of hash algorithms in common use that are believed to be secure: SHA2 and SHA3. Within each family, there are differing output lengths. SHA-256, SHA-384, and SHA-512 are all instances of the SHA2 family with outputs of 256b, 384b, and 512b respectively, while SHA3-256, SHA3-384, and SHA3-512 are the SHA3 family members.

For the purposes of the course, we don't care which of SHA2 or SHA3 we use, although they are in practice very different functions. The only significant difference is that SHA2 is vulnerable to a _length extension attack_. Given $$H(M)$$ and the length of the message, but not $$M$$ itself, there are circumstances where an attacker can compute $$H(M \Vert M')$$ for an arbitrary $$M'$$ of the attacker's choosing. This is because SHA2's output reflects all of its internal state, while SHA3 includes internal state that is never outputted but only used in the calculation of subsequent hashes. While this does not violate any of the aforementioned properties of hash functions, it is undesirable in some circumstances.

In general, we prefer using a hash function that is related to the length of any associated symmetric key algorithm. By relating the hash function's output length with the symmetric encryption algorithm's key length, we can ensure that it is equally difficult for an attacker to break either scheme. For example, if we are using AES-128, we should use SHA-256 or SHA3-256. Assuming both functions are secure, it takes $$2^{128}$$ operations to brute-force a 128 bit key and $$2^{128}$$ operations to generate a hash collision on a 256 bit hash function. For longer key lengths, however, we may relax the hash difficulty. For example, with 256b AES, the NSA uses SHA-384, not SHA-512, because, let's face it, $$2^{192}$$ operations is already a hugely impractical amount of computation.

## Example Application: Proving Possession of a Large Number of Records

Cryptographic hashes can also be used in clever statistical protocols. Consider the following scenario. A hacker contacts a journalist and claims to have stolen 150 million records. The hacker does not want to hand over the entire dataset, yet still wants to convince the journalist that the claim is roughly true.

Because a good cryptographic hash function produces outputs that look random, the set of all hash values of the records behaves like a large collection of random numbers. If someone really hashed 150 million different records, the _lowest_ few hash values among them should be extremely small. If the lowest hashes the hacker can produce are only moderately small, it is strong statistical evidence that far fewer than 150 million records were hashed.

A simple protocol is therefore:

1. The journalist asks the hacker to compute the hash of every record and to return the 10 _lowest_ hash values together with the corresponding records.
2. The journalist checks that those 10 hashes are as small as one would statistically expect from a set of 150 million random values, and also verifies that the supplied records are well-formed and consistent with the claimed dataset.

An attacker who only possesses, say, 15 million records will find it extremely difficult to produce 10 hashes that look as small as the lowest 10 out of 150 million.

Two straightforward countermeasures make cheating even harder:

- The journalist requires the actual records that produced the low hashes (so the attacker cannot simply invent low hash values).
- The journalist sends a fresh random nonce at the beginning of the interaction. The hacker must hash each record _concatenated with that nonce_. This prevents the attacker from pre-computing a large table of low-hash fake records in advance.

The technique does not give a mathematical proof, but it supplies strong probabilistic evidence that the claimed number of records is approximately correct, while revealing only a tiny fraction of the data.
