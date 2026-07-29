---
title: Public-Key Encryption
parent: Cryptography
nav_order: 7
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Public-Key (Asymmetric) Encryption

## The Key-Management Problem

So far we have relied heavily on symmetric-key cryptography. All of those schemes assume that Alice and Bob already share a secret key that no one else knows. While symmetric algorithms like AES are fast and secure, they leave us with a fundamental logistical challenge known as the **key-management problem**: how can two parties who have never met securely establish a shared secret over an insecure network without an eavesdropper stealing it first?

This is exactly the problem that public-key (asymmetric) cryptography was invented to solve. Instead of requiring Alice and Bob to secretly agree on a key beforehand, each party can generate a pair of mathematically related keys: a _public key_ that can be openly distributed and a _private key_ that remains strictly secret. With this setup, Alice can encrypt a message using Bob’s public key, and only Bob (who possesses the matching private key) can decrypt it. This eliminates the need to securely share a symmetric key in advance and dramatically simplifies secure communication at global scale.

However, public-key cryptography introduces its own challenges. The mathematical operations are significantly slower than symmetric cryptography, so in practice we use a hybrid approach: public-key cryptography is used to establish a short-lived symmetric session key, and then the much faster symmetric algorithms handle the bulk of the data transfer. We will explore this hybrid pattern in detail later in the topic.

## Public-Key Encryption Overview

In a public-key (asymmetric) cryptosystem, each participant possesses a matched _key pair_:

- A **public key** that can be freely distributed to anyone.
- A **private key** that must be kept strictly secret by its owner.

The two keys are mathematically related through a _trapdoor one-way function_. This is a function that is easy to compute in one direction but computationally infeasible to reverse—unless you possess the secret “trapdoor” information (the private key).

### Basic Encryption/Decryption Workflow

Suppose Alice wants to send a confidential message to Bob:

1. Alice obtains Bob’s public key (we will discuss how she can trust this key later).
2. She encrypts her message using Bob’s public key.
3. Bob receives the ciphertext and uses his private key to decrypt it.

Only Bob can perform the decryption because he alone knows the private key. Anyone who has only the public key can encrypt messages for Bob, but cannot decrypt them. This solves the key-distribution problem that plagues symmetric cryptography: Alice and Bob no longer need a secure channel to agree on a shared secret in advance.

### Trapdoor One-Way Functions (High-Level)

The security of public-key encryption rests on the existence of trapdoor one-way functions. These functions have three key characteristics:

- Easy to compute in the forward direction (encryption).
- Computationally infeasible to invert without the trapdoor (decryption without the private key).
- Easy to invert when the trapdoor (private key) is known.

Recall from the previous section that a one-way function is a function $f$ such that given $x$, it is easy to compute $f(x)$, but given $y$, it is hard to find a value $x$ such that $f(x)=y$.

A _trapdoor one-way function_ is a function $f$ that is one-way, but also has a special backdoor that enables someone who knows the backdoor to invert the function. As before, given $x$, it is easy to compute $f(x)$, but given only $y$, it is hard to find a value $x$ such that $f(x) = y$. However, given both $y$ and the special backdoor $K$, it is now easy to compute $x$ such that $f(x) = y$.

A trapdoor one-way function can be used to construct a public encryption scheme as follows. Bob has a public key $PK$ and a secret key $SK$. He distributes $PK$ to everyone, but does not share $SK$ with anyone. We will use the trapdoor one-way function $f(x)$ as the encryption function.

Given the public key $PK$ and a plaintext message $x$, it is computationally easy to compute the encryption of the message: $y = f(x)$.

Given a ciphertext $y$ and only the public key $PK$, it is hard to find the plaintext message $x$ where $f(x) = y$. However, given ciphertext $y$ and the secret key $SK$, it becomes computationally easy to find the plaintext message $x$ such that $y=f(x)$, i.e., it is easy to compute $f^{-1}(y)$.

We can view the private key as "unlocking" the trapdoor. Given the private key $SK$, it becomes easy to compute the decryption $f^{-1}$, and it remains easy to compute the encryption $f$.

Here are two examples of trapdoor functions that will help us build public encryption schemes:

- _RSA Hardness_: Suppose $n=pq$, i.e. $n$ is the product of two large primes $p$ and $q$. Given $c = m^e \pmod{n}$ and $e$, it is computationally hard to find $m$. However, with the factorization of $n$ (i.e. $p$ or $q$), it becomes easy to find $m$.
- _Discrete log problem_: Suppose $p$ is a large prime and $g$ is a generator. Given $g$, $p$, $A = g^a \pmod{p}$, and $B = g^b \pmod{p}$, it is computationally hard to find $g^{ab} \pmod{p}$. However, with $a$ or $b$, it becomes easy to find $g^{ab} \pmod{p}$.

## RSA Encryption

For this class, you won’t need to remember the detailed mathematical proof of why RSA works. All you need to remember is that we use the public key to encrypt messages, we use the corresponding private key to decrypt messages, and an attacker cannot break RSA encryption unless they can factor large primes, which is believed to be hard.

There is a tricky flaw in the basic RSA scheme. The scheme is deterministic, so it is not IND-CPA secure. Sending the same message multiple times causes information leakage, because an adversary can see when the same message is sent. This basic variant of RSA might work for encrypting “random” messages, but it is not IND-CPA secure. As a result, we have to add some randomness to make the RSA scheme resistant to information leakage.

RSA introduces randomness into the scheme through a padding mode. Despite the name, RSA padding modes are more similar to the IVs in block cipher modes than the padding in block cipher modes. Unlike block cipher padding, public-key padding is not a deterministic algorithm for extending a message. Instead, public-key padding is a tool for mixing in some randomness so that the ciphertext output “looks random,” but can still be decrypted to retrieve the original plaintext.

One common padding scheme is OAEP (Optimal Asymmetric Encryption Padding). This scheme effectively generates a random symmetric key, uses the random key to scramble the message, and encrypts both the scrambled message and the random key. To recover the original message, the attacker has to recover both the scrambled message and the random key in order to reverse the scrambling process.

## El Gamal Encryption

The standard Diffie-Hellman protocol doesn’t quite deliver public-key encryption directly. It allows Alice and Bob to agree on a shared secret that they could use as a symmetric key, but it doesn’t let Alice and Bob control what the shared secret is. For example, in a Diffie-Hellman exchange where Alice and Bob each choose random secrets, the shared secret is also a random value. Diffie-Hellman on its own does not let Alice and Bob send chosen encrypted messages to each other. However, there is a slight variation on Diffie-Hellman that would allow Alice and Bob to exchange encrypted messages.

In 1985, a cryptographer named Taher Elgamal invented a public-key encryption algorithm based on Diffie-Hellman. We will present a simplified form of the El Gamal encryption scheme.

The public system parameters are a large prime $p$ and a value $g$ satisfying $1 < g < p-1$. Bob chooses a random value $b$ (satisfying $0 \le b \le p-2$) and computes $B = g^b \pmod{p}$. Bob’s public key is $B$, and his private key is $b$. Bob publishes $B$ to the world, and keeps $b$ secret.

Now, suppose Alice has a message $m$ (in the range $1 \dots p-1$) she wants to send to Bob, and suppose Alice knows that Bob’s public key is $B$. To encrypt the message $m$ to Bob, Alice picks a random value $r$ (in the range $0 \dots p-2$), and forms the ciphertext:

$$
(g^r \pmod{p},\; m \times B^r \pmod{p})
$$

Note that the ciphertext is a pair of numbers, each number in the range $0 \dots p-1$.

How does Bob decrypt? Let’s say that Bob receives a ciphertext of the form $(R, S)$. To decrypt it, Bob computes:

$$
R^{-b} \times S \pmod{p}
$$

and the result is the message $m$ Alice sent him.

Why does this decryption procedure work? If $R = g^r \pmod{p}$ and $S = m \times B^r \pmod{p}$ (as should be the case if Alice encrypted the message $m$ properly), then:

$$
R^{-b} \times S = (g^r)^{-b} \times (m \times B^r) = g^{-rb} \times m \times g^{br} = m \pmod{p}
$$

If you squint your eyes just right, you might notice that El Gamal encryption is basically Diffie-Hellman, tweaked slightly. It’s a Diffie-Hellman key exchange, where Bob uses his long-term public key $B$ and where Alice uses a fresh new public key $R = g^r \pmod{p}$ chosen anew just for this exchange. They derive a shared key $K = g^{rb} = B^r = R^b \pmod{p}$. Then, Alice encrypts her message $m$ by multiplying it by the shared key $K$ modulo $p$.

That last step is in effect a funny kind of one-time pad, where we use multiplication modulo $p$ instead of XOR: here $K$ is the key material for the one-time pad, and $m$ is the message, and the ciphertext is $S = m \times K = m \times B^r \pmod{p}$. Since Alice chooses a new value $r$ independently for each message she encrypts, we can see that the key material is indeed used only once. And a one-time pad using modular multiplication is just as secure as XOR, for essentially the same reason that a one-time pad with XOR is secure: given any ciphertext $S$ and a hypothesized message $m$, there is exactly one key $K$ that is consistent with this hypothesis (i.e., exactly one value of $K$ satisfying $S = m \times K \pmod{p}$).

Another way you can view El Gamal is using the discrete log trapdoor one-way function defined above: Alice encrypts the message with $B^r = g^{br} \pmod{p}$. Given only $g$, $p$, $R = g^r \pmod{p}$, and $B = g^b \pmod{p}$, it is hard for an attacker to learn $g^{-br} \pmod{p}$ and decrypt the message. However, with Bob’s secret key $b$, Bob can easily calculate $g^{-br} \pmod{p}$ and decrypt the message.

_(Note: For technical reasons, this simplified El Gamal scheme is actually not semantically secure on its own. With some tweaks, the scheme can be made semantically secure.)_

### Summary of El Gamal Encryption

- **System parameters:** A 2048-bit prime $p$, and a value $g$ in the range $2 \dots p-2$. Both are arbitrary, fixed, and public.
- **Key generation:** Bob picks $b$ in the range $0 \dots p-2$ randomly, and computes $B = g^b \pmod{p}$. His public key is $B$ and his private key is $b$.
- **Encryption:** $E_B(m) = (g^r \pmod{p},\; m \times B^r \pmod{p})$ where $r$ is chosen randomly from $0 \dots p-2$.
- **Decryption:** $D_b(R, S) = R^{-b} \times S \pmod{p}$.

## Public Key Distribution

This all sounds great—almost too good to be true. We have a way for a pair of strangers who have never met each other in person to communicate securely with each other. Unfortunately, it is indeed too good to be true. There is a slight catch. The catch is that if Alice and Bob want to communicate securely using these public-key methods, they need some way to securely learn each others' public key. The algorithms presented here don't help Alice figure out what is Bob's public key; she's on her own for that.

You might think all Bob needs to do is broadcast his public key, for Alice's benefit. However, that's not secure against _active attacks_. Mallory, the active attacker, could broadcast his own public key, pretending to be Bob: he could send a spoofed broadcast message that appears to be from Bob, but that contains a public key that Mallory generated. If Alice trustingly uses that public key to encrypt messages to Bob, then Mallory will be able to intercept Alice's encrypted messages and decrypt them using the private key Mallory chose.

What this illustrates is that Alice needs a way to obtain Bob's public key through some channel that she is confident cannot be tampered with. That channel does not need to protect the _confidentiality_ of Bob's public key, but it does need to ensure the _integrity_ of Bob's public key. It's a bit tricky to achieve this.

One possibility is for Alice and Bob to meet in person, in advance, and exchange public keys. Some computer security conferences have "key-signing parties" where like-minded security folks do just that. In a similar vein, some cryptographers print their public key on their business cards. However, this still requires Alice and Bob to meet in person in advance. Can we do any better? In the next topic (Digital Signatures), we will soon see some methods that help somewhat with that problem.

## Session Keys

There is a problem with public key: it is _slow_. It is very, very slow. When encrypting a single message with a 2048-bit RSA key, the RSA algorithm requires exponentiation of a 2048-bit number to a 2048-bit power, modulo a 2048-bit number. Additionally, some public key schemes only really work to encrypt "random" messages. For example, RSA without OAEP leaks when the same message is sent twice, so it is only secure if every message sent consists of random bits. In the simplified El Gamal scheme shown in these notes, it is easy for an attacker to substitute the message $M' = 2M$. If the messages have meaning, this malleability is a serious problem.

Because public key schemes are expensive and difficult to make IND-CPA secure, we tend to only use public key cryptography to distribute one or more _session keys_. Session keys are the keys used to actually encrypt and authenticate the message. To send a message, Alice first generates a random set of session keys. Often, we generate several different session keys for different purposes. For example, we may generate one key for encryption algorithms and another key for MAC algorithms. We may also generate one key to encrypt messages from Alice to Bob, and another key to encrypt messages from Bob to Alice. (If we need different keys for each message direction and different keys for encryption and MAC, we would need a total of four symmetric keys.)

Alice then encrypts the message using a symmetric algorithm with the session keys (such as AES-128-CBC-HMAC-SHA-256) and encrypts the random session keys with Bob's public key. When he receives the ciphertext, Bob first decrypts the session keys and then uses the session keys to decrypt the original message.
