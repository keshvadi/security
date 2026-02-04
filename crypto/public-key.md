---
title: Public-Key Encryption
parent: Cryptography
nav_order: 7
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Public-Key (Asymmetric) Encryption

## 1. Overview

Previously we saw symmetric-key encryption, where Alice and Bob share a secret key $$K$$ and use the same key to encrypt and decrypt messages. However, symmetric-key cryptography can be inconvenient to use, because it requires Alice and Bob to coordinate somehow and establish the shared secret key. _Asymmetric cryptography_, also known as _public-key cryptography_, is designed to address this problem.

In a public-key cryptosystem, the recipient Bob has a publicly available key, his _public key_, that everyone can access. When Alice wishes to send him a message, she uses his public key to encrypt her message. Bob also has a secret key, his _private key_, that lets him decrypt these messages. Bob publishes his public key but does not tell anyone his private key (not even Alice).

Public-key cryptography provides a nice way to help with the key management problem. Alice can pick a secret key $$K$$ for some symmetric-key cryptosystem, then encrypt $$K$$ under Bob's public key and send Bob the resulting ciphertext. Bob can decrypt using his private key and recover $$K$$. Then Alice and Bob can communicate using a symmetric-key cryptosystem, with $$K$$ as their shared key, from there on.

## 2. Trapdoor One-way Functions

Public-key cryptography relies on a close variant of the one-way function. Recall from the previous section that a one-way function is a function $$f$$ such that given $$x$$, it is easy to compute $$f(x)$$, but given $$y$$, it is hard to find a value $$x$$ such that $$f(x)=y$$.

A _trapdoor one-way function_ is a function $$f$$ that is one-way, but also has a special backdoor that enables someone who knows the backdoor to invert the function. As before, given $$x$$, it is easy to compute $$f(x)$$, but given only $$y$$, it is hard to find a value $$x$$ such that $$f(x) = y$$. However, given both $$y$$ and the special backdoor $$K$$, it is now easy to compute $$x$$ such that $$f(x) = y$$.

A trapdoor one-way function can be used to construct a public encryption scheme as follows. Bob has a public key $$PK$$ and a secret key $$SK$$. He distributes $$PK$$ to everyone, but does not share $$SK$$ with anyone. We will use the trapdoor one-way function $$f(x)$$ as the encryption function.

Given the public key $$PK$$ and a plaintext message $$x$$, it is computationally easy to compute the encryption of the message: $$y = f(x)$$.

Given a ciphertext $$y$$ and only the public key $$PK$$, it is hard to find the plaintext message $$x$$ where $$f(x) = y$$. However, given ciphertext $$y$$ and the secret key $$SK$$, it becomes computationally easy to find the plaintext message $$x$$ such that $$y=f(x)$$, i.e., it is easy to compute $$f^{-1}(y)$$.

We can view the private key as "unlocking" the trapdoor. Given the private key $$SK$$, it becomes easy to compute the decryption $$f^{-1}$$, and it remains easy to compute the encryption $$f$$.

Here are two examples of trapdoor functions that will help us build public encryption schemes:

- _RSA Hardness_: Suppose $$n=pq$$, i.e. $$n$$ is the product of two large primes $$p$$ and $$q$$. Given $$c = m^e \pmod{n}$$ and $$e$$, it is computationally hard to find $$m$$. However, with the factorization of $$n$$ (i.e. $$p$$ or $$q$$), it becomes easy to find $$m$$.

- _Discrete log problem_: Suppose $$p$$ is a large prime and $$g$$ is a generator. Given $$g$$, $$p$$, $$A = g^a \pmod{p}$$, and $$B = g^b \pmod{p}$$, it is computationally hard to find $$g^{ab} \pmod{p}$$. However, with $$a$$ or $$b$$, it becomes easy to find $$g^{ab} \pmod{p}$$.

## 3. Public Key Distribution

This all sounds great---almost too good to be true. We have a way for a pair of strangers who have never met each other in person to communicate securely with each other. Unfortunately, it is indeed too good to be true. There is a slight catch. The catch is that if Alice and Bob want to communicate securely using these public-key methods, they need some way to securely learn each others' public key. The algorithms presented here don't help Alice figure out what is Bob's public key; she's on her own for that.

You might think all Bob needs to do is broadcast his public key, for Alice's benefit. However, that's not secure against _active attacks_. Mallory, the active attacker, could broadcast his own public key, pretending to be Bob: he could send a spoofed broadcast message that appears to be from Bob, but that contains a public key that Mallory generated. If Alice trustingly uses that public key to encrypt messages to Bob, then Mallory will be able to intercept Alice's encrypted messages and decrypt them using the private key Mallory chose.

What this illustrates is that Alice needs a way to obtain Bob's public key through some channel that she is confident cannot be tampered with. That channel does not need to protect the _confidentiality_ of Bob's public key, but it does need to ensure the _integrity_ of Bob's public key. It's a bit tricky to achieve this.

One possibility is for Alice and Bob to meet in person, in advance, and exchange public keys. Some computer security conferences have "key-signing parties" where like-minded security folks do just that. In a similar vein, some cryptographers print their public key on their business cards. However, this still requires Alice and Bob to meet in person in advance. Can we do any better? In the next topic (Digital Signatures), we will soon see some methods that help somewhat with that problem.

## 4. Session Keys

There is a problem with public key: it is _slow_. It is very, very slow. When encrypting a single message with a 2048b RSA key, the RSA algorithm requires exponentiation of a 2048b number to a 2048b power, modulo a 2048b number. Additionally, some public key schemes only really work to encrypt "random" messages. For example, RSA without OAEP leaks when the same message is sent twice, so it is only secure if every message sent consists of random bits.

Because public key schemes are expensive and difficult to make IND-CPA secure, we tend to only use public key cryptography to distribute one or more _session keys_. Session keys are the keys used to actually encrypt and authenticate the message. To send a message, Alice first generates a random set of session keys. Often, we generate several different session keys for different purposes. For example, we may generate one key for encryption algorithms and another key for MAC algorithms. We may also generate one key to encrypt messages from Alice to Bob, and another key to encrypt messages from Bob to Alice. (If we need different keys for each message direction and different keys for encryption and MAC, we would need a total of four symmetric keys.) Alice then encrypts the message using a symmetric algorithm with the session keys (such as AES-128-CBC-HMAC-SHA-256 [^1]) and encrypts the random session keys with Bob's public key. When he receives the ciphertext, Bob first decrypts the session keys and then uses the session keys to decrypt the original message.

<!-- ## Sample Exam Questions

Here we've compiled a list of Sample Exam Questions that cover public-key cryptography. -->

[^1]: That is, using AES with 128b keys in CBC mode and then using HMAC with SHA-256 for integrity