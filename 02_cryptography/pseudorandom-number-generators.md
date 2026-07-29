---
title: Pseudorandom Number Generators
parent: Cryptography
nav_order: 5
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Pseudorandom Number Generators

## Why Cryptography Needs Randomness

Modern cryptography depends heavily on randomness. Many of the values that keep systems secure must be unpredictable to an adversary. Common examples include:

- Secret keys for symmetric encryption and message authentication
- Initialization vectors (IVs) and nonces used in encryption modes
- Salts for password hashing
- Random challenges in authentication protocols
- Session identifiers and authentication cookies
- One-time codes (for example, SMS or app-based second-factor codes)

If any of these values can be guessed or predicted by an attacker, the security guarantees of the surrounding cryptographic mechanisms often collapse. An attacker who can predict a key can decrypt traffic; an attacker who can predict a nonce may be able to force keystream reuse; an attacker who can predict a password salt can pre-compute tables more effectively.

It is therefore not enough for a value to _look_ random. A sequence may pass simple statistical tests and still be highly predictable to someone who knows how it was generated. What cryptography actually requires is _unpredictability_: given all the information an adversary is likely to have, the next value should be impossible to guess with probability significantly better than chance. In the rest of this topic we examine how real systems try to achieve that property.

## True Randomness versus Pseudorandomness

True randomness is obtained by measuring physical processes that are believed to be unpredictable. Typical sources include thermal noise in electrical circuits, timing jitter of interrupts and device drivers, mouse movements, keyboard press timings, and specialized hardware random-number generators.

The amount of uncertainty contained in such measurements is called _entropy_. Not every random source is equally useful. Consider two coins:

- A fair coin that lands heads or tails with equal probability has high entropy: each toss is maximally uncertain.
- A biased coin that lands heads 99 % of the time is still random, yet it has very low entropy. An observer who always guesses “heads” will be correct almost always.

For cryptographic purposes we need the high-entropy kind of randomness. A key generated from the biased coin would be far easier for an attacker to guess than a key generated from a fair coin. In general, the uniform distribution (every outcome equally likely) yields the greatest entropy, which is why cryptography prefers bits that behave like independent fair coin tosses.

Collecting high-quality entropy from the physical environment is relatively slow and can be costly. Operating systems therefore maintain an _entropy pool_ that accumulates these measurements over time. When a program needs random bytes, the system rarely returns the raw physical samples directly; doing so would quickly exhaust the available entropy.

Instead, almost all practical systems use a _pseudorandom number generator_ (PRNG). A PRNG is a deterministic algorithm that takes a short, truly random _seed_ (drawn from the entropy pool) and expands it into a long sequence of bits that appear random. As long as the seed remains secret and contains sufficient entropy, the output of a cryptographically secure PRNG is computationally indistinguishable from true random bits for any adversary with realistic resources. The expensive true randomness is used only sparingly—to seed or reseed the generator—while the bulk of the random bits needed by cryptographic algorithms are produced quickly and cheaply by the PRNG.

## Pseudorandom Number Generators (PRNGs)

A _pseudorandom number generator_ (PRNG) is a deterministic algorithm that expands a short, truly random value called the _seed_ into a long sequence of bits that are computationally indistinguishable from true random bits. The generator maintains an _internal state_. At any moment this state completely determines all future output. When the PRNG is asked for new bits it updates its state and emits the next block of the sequence.

Because the algorithm is deterministic, two parties who start with the identical seed (and therefore the identical internal state) will produce exactly the same stream of bits. This property is essential for many cryptographic constructions: both the sender and the receiver can independently generate the same keystream or the same nonce sequence. At the same time it implies that the seed and the evolving internal state must be kept secret; anyone who obtains them can reproduce the entire output stream.

A practical cryptographic PRNG is normally described by three operations:

- **Seed** (or initialize). The generator is given a block of high-entropy material drawn from the system’s entropy pool. This material is mixed into the internal state, after which the generator is ready to produce output.
- **Reseed**. Additional fresh entropy may be mixed into the current state at any later time. Reseeding restores unpredictability if the previous state might have been compromised and also replenishes entropy that has been consumed.
- **Generate**. The caller requests a specified number of pseudorandom bits. The generator updates its internal state and returns the requested bits. Some designs also allow a caller to supply extra entropy at the moment of generation.

Taken together, these three operations let a system collect true randomness only occasionally while still being able to supply large quantities of high-quality pseudorandom bits on demand.

## Security Properties of Cryptographic PRNGs

A PRNG that is suitable for cryptographic use must satisfy several carefully defined security properties.

The most basic requirement is _computational indistinguishability_ from true randomness. An adversary who does not know the seed or the internal state should be unable to distinguish the generator’s output from a sequence of genuinely random bits by any efficient test. In other words, for all practical purposes the bits look uniformly random.

It is important to understand the precise meaning of this claim. A PRNG is a deterministic algorithm that expands a finite seed of \(n\) bits. In principle, an attacker with unlimited computational power could try all \(2^n\) possible seeds, generate the corresponding output streams, and see whether any of them matches the observed bits. Such an exhaustive search would distinguish the PRNG from true randomness. Cryptographic security therefore does not claim information-theoretic indistinguishability; it only claims that no _efficient_ adversary (one limited to realistic amounts of time and computing power) can tell the two apart.

A second, independent property is _backtracking resistance_ (also called rollback resistance). Suppose an attacker somehow learns the internal state of the generator at a particular moment. Backtracking resistance requires that the attacker still cannot recover any of the bits that were output _before_ that moment. The past outputs remain computationally indistinguishable from random even though the present state is known. This property is important because a single compromise should not retrospectively expose earlier keys, nonces, or other secrets that the generator has already produced.

A third desirable property is _prediction resistance_. After a compromise of the internal state, the generator should be able to recover security once fresh entropy is mixed in by a reseed operation. Once reseeding has occurred, future outputs must again be unpredictable to the attacker who knew the earlier state. Prediction resistance therefore limits the damage of a temporary state compromise to the window between the compromise and the next successful reseed.

Not every algorithm that produces “random-looking” numbers provides these guarantees. Only generators that have been explicitly designed and analysed for cryptographic use—commonly called cryptographically secure PRNGs or deterministic random bit generators (DRBGs)—are appropriate for generating keys, nonces, and other secret values.

## Practical Example: HMAC-DRBG

One widely deployed cryptographically secure PRNG is HMAC-DRBG (Deterministic Random Bit Generator based on HMAC). Its design illustrates how a simple, well-analysed primitive can be turned into a full-featured generator.

HMAC-DRBG maintains a short internal state consisting of two values, $$K$$ and $$V$$. $$K$$ serves as the HMAC key and $$V$$ serves as the message input. To produce pseudorandom output the generator repeatedly computes

$$
V \leftarrow \operatorname{HMAC}(K, V)
$$

and concatenates successive values of \($$V$$\) until enough bits have been obtained. Because the output of a secure HMAC is indistinguishable from random to anyone who does not know the key \(K\), the resulting bit string is also indistinguishable from random.

Seeding and reseeding are performed by feeding fresh entropy into additional HMAC calls that update both \(K\) and \(V\). After each generation request the state is deliberately advanced with further HMAC computations so that knowledge of the current state does not reveal earlier output (backtracking resistance). Fresh entropy can be mixed in at any time, restoring unpredictability after a possible compromise (prediction resistance).

A practical advantage of this construction is its robustness to imperfect entropy sources. Because the seeding process is based on a cryptographic hash, HMAC-DRBG can accept a long input string even when each individual bit of that string carries only a small amount of entropy. For example, if every bit of the supplied seed material has only 0.1 bits of true entropy, feeding a few thousand such bits still yields a fully seeded generator with 256 bits of security. Moreover, mixing in additional data that contains no entropy at all (for instance a string of zeros or a constant such as \(\pi\)) does not degrade the internal state. The generator simply extracts whatever entropy is present and ignores the rest.

The concrete details of the state updates are less important than the underlying idea: HMAC-DRBG inherits its security from the assumed pseudorandomness of HMAC. As long as the initial seed ultimately contributes sufficient entropy and the HMAC key remains secret, the generator meets the standard cryptographic requirements for a PRNG.

## From PRNGs to Stream Ciphers

A cryptographically secure PRNG can be used to construct a practical encryption scheme that behaves like a one-time pad. The idea is straightforward: the secret key (together with a unique per-message value) is used to seed the PRNG, the PRNG produces a long keystream of pseudorandom bits, and the plaintext is XORed with that keystream. Decryption is performed by regenerating the identical keystream and XORing again. Because a secure PRNG’s output is indistinguishable from true randomness, the resulting scheme inherits the confidentiality properties of the one-time pad—provided the keystream is never reused.

Keystream reuse is catastrophic for the same reason that one-time-pad key reuse is catastrophic. If two plaintexts $$P_1$$ and $$P_2$$ are encrypted under the same keystream $$S$$, an observer who obtains the two ciphertexts can compute

$$
C_1 \oplus C_2 = (P_1 \oplus S) \oplus (P_2 \oplus S) = P_1 \oplus P_2.
$$

The keystream cancels, and the attacker learns the relationship between the two plaintexts. In many practical settings this leakage is enough to recover substantial amounts of data.

To ensure that the keystream is different for every message, the generator is seeded not only with the long-term secret key but also with a fresh _nonce_ or _initialization vector_ (IV). As long as the nonce never repeats under the same key, the PRNG produces a unique keystream for each encryption.

In principle any cryptographically secure PRNG (including HMAC-DRBG) could be used this way. In practice, however, dedicated stream ciphers such as AES in counter mode (AES-CTR) and ChaCha20 are strongly preferred. The reason is efficiency of random access. Both AES-CTR and ChaCha20 incorporate an explicit counter into the keystream generation process. Consequently, any block of a long ciphertext can be decrypted in isolation simply by setting the counter to the appropriate value.

By contrast, a general-purpose PRNG such as HMAC-DRBG has no counter; its state evolves only by successive steps. To decrypt the final megabyte of a one-terabyte file one would first have to generate the preceding terabyte of keystream—an entirely impractical cost. The very feature that makes a counter undesirable in a general PRNG (the ability to jump forward or backward in the output stream) is exactly what makes AES-CTR and ChaCha20 efficient and convenient as stream ciphers.

Thus, while the theoretical construction “PRNG + XOR” explains how stream ciphers achieve security, real systems use purpose-designed algorithms that add a counter precisely so that large encrypted objects can be accessed efficiently.
