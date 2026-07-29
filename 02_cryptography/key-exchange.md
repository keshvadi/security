---
title: Diffie-Hellman Key Exchange
parent: Cryptography
nav_order: 6
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Diffie-Hellman Key Exchange

So far we have relied heavily on symmetric-key cryptography (encryption, MACs, and authenticated encryption). All of those schemes assume that Alice and Bob already share a secret key that no one else knows. This raises an obvious question: how can two parties who can communicate only over an insecure channel establish such a shared secret in the first place?
It turns out there is a clever way to do it, first discovered by Whit Diffie and Martin Hellman in the 1970s.

The protocol they invented, now called Diffie-Hellman key exchange, allows two parties to agree on a shared secret even though an eavesdropper can see every message they send. The resulting secret is typically used as an _ephemeral_ session key: it is employed for a single conversation (or a short period of time) and then discarded.

**Important limitation.** Diffie-Hellman by itself provides no authentication. A passive eavesdropper (Eve) cannot compute the shared secret, but an active attacker (Mallory) who can modify messages can perform a man-in-the-middle attack and end up sharing one key with Alice and a different key with Bob. Real protocols therefore always combine Diffie-Hellman with an authentication mechanism. We will return to this point later in the section.

## Diffie-Hellman Intuition: The Paint Analogy

A helpful way to understand Diffie-Hellman is the classic paint-mixing analogy.

Alice and Bob want to end up with the same secret color, but they can communicate only over a channel that Eve can observe. They proceed as follows:

- They publicly agree on a common base color (for example, yellow).
- Alice chooses a private color (say, red) and mixes it with the public yellow. She sends the resulting mixture to Bob.
- Bob chooses his own private color (say, blue) and mixes it with the public yellow. He sends his mixture to Alice.
- Alice takes the mixture she received from Bob and adds her private red.
- Bob takes the mixture he received from Alice and adds his private blue.

Both now hold the same final color (yellow + red + blue). Eve, who saw only the two mixtures that travelled over the channel, cannot easily recover the final secret color: she would need to “unmix” a paint combination, which is assumed to be impractical.

The analogy captures the essential idea of a _one-way function_: combining (mixing) is easy, but reversing the combination is hard. In the actual Diffie-Hellman protocol the same idea is realized with modular exponentiation. Computing \(g^a \bmod p\) is easy, yet recovering the exponent \(a\) from the result (the discrete logarithm problem) is believed to be computationally infeasible when the parameters are chosen properly. The paint mixtures correspond to the public values \(g^a \bmod p\) and \(g^b \bmod p\); the final shared color corresponds to the shared secret \(g^{ab} \bmod p\).

## Discrete logarithm problem

The secret exchange in the color analogy relied on the fact that mixing two colors is easy, but separating a mixture of two colors is practically impossible. It turns out that there is a mathematical equivalent of this. We call these _one-way functions_: a function $$f$$ such that given $$x$$, it is easy to compute $$f(x)$$, but given $$y$$, it is practically impossible to find a value $$x$$ such that $$f(x) = y$$.

A one-way function is also sometimes described as the computational equivalent of a process that turns a cow into hamburger: given the cow, you can produce hamburger, but there's no way to restore the original cow from the hamburger.

There are many functions believed to be one-way functions. The simplest one is exponentiation modulo a prime: $$f(x) = g^x \pmod{p}$$, where $$p$$ is a large prime and $$g$$ is a specially-chosen generator.

Given $$x$$, it is easy to calculate $$f(x)$$. However, given $$f(x) = g^x \pmod{p}$$, there is no known efficient algorithm to solve for $$x$$. This is known as the _discrete logarithm problem_, and it is believed to be computationally hard to solve.

Using the hardness of the discrete log problem and the analogy from above, we are now ready to construct the Diffie-Hellman key exchange protocol.

## Diffie-Hellman protocol

In high-level terms, the Diffie-Hellman key exchange works like this.

Alice and Bob first establish the public parameters $$p$$ and $$g$$. Remember that $$p$$ is a large prime and $$g$$ is a generator in the range $$1<g<p-1$$. For instance, Alice could pick $$p$$ and $$g$$ and then announce it publicly to Bob. Today, $$g$$ and $$p$$ are often hardcoded or defined in a standard so they don't need to be chosen each time. These values don't need to be specific to Alice or Bob in any way, and they're not secret.

Then, Alice picks a secret value $$a$$ at random from the set $$\{0,1,\dots,p-2\}$$, and she computes $$A=g^a \bmod p$$. At the same time, Bob randomly picks a secret value $$b$$ and computes $$B=g^b \bmod p$$.

Now Alice announces the value $$A$$ (keeping $$a$$ secret), and Bob announces $$B$$ (keeping $$b$$ secret). Alice uses her knowledge of $$B$$ and $$a$$ to compute

$$S = B^a = (g^b)^a = g^{ba} \pmod p.$$

Symmetrically, Bob uses his knowledge of $$A$$ and $$b$$ to compute

$$S = A^b = (g^a)^b = g^{ab} \pmod p.$$

Note that $$g^{ba} = g^{ab} \pmod{p}$$, so both Alice and Bob end up with the same result, $$S$$.

Finally, Alice and Bob can use $$S$$ as a shared key for a symmetric-key cryptosystem (in practice, we would apply some hash function to $$S$$ first and use the result as our shared key, for technical reasons).

The amazing thing is that Alice and Bob's conversation is entirely public, and from this public conversation, they both learn this secret value $$S$$---yet eavesdroppers who hear their entire conversation cannot learn $$S$$.

As far as we know, there is no efficient algorithm to deduce $$S=g^{ab} \bmod p$$ from the values Eve sees, namely $$A=g^a \bmod p$$, $$B=g^b \bmod p$$, $$g$$, and $$p$$. The hardness of this problem is closely related to the discrete log problem discussed above. In particular, the fastest known algorithms for solving this problem take $$2^{cn^{1/3} (\log n)^{2/3}}$$ time, if $$p$$ is a $$n$$-bit prime. For $$n=2048$$, these algorithms are far too slow to allow reasonable attacks.

Here is how this applies to secure communication among computers. In a computer network, each participant could pick a secret value $$x$$, compute $$X=g^x \bmod p$$, and publish $$X$$ for all time. Then any pair of participants who want to hold a conversation could look up each other's public value and use the Diffie-Hellman scheme to agree on a secret key known only to those two parties. This means that the work of picking $$p$$, $$g$$, $$x$$, and $$X$$ can be done in advance, and each time a new pair of parties want to communicate, they each perform only one modular exponentiation. Thus, this can be an efficient way to set up shared keys.

Here is a summary of Diffie-Hellman key exchange:

- _System parameters:_ a 2048-bit prime $$p$$, a value $$g$$ in the range $$2\ldots p-2$$. Both are arbitrary, fixed, and public.

- _Key agreement protocol:_ Alice randomly picks $$a$$ in the range $$1\ldots p-2$$ and sends $$A=g^a \bmod p$$ to Bob. Bob randomly picks $$b$$ in the range $$1\ldots p-2$$ and sends $$B=g^b \bmod p$$ to Alice. Alice computes $$K=B^a \bmod p$$. Bob computes $$K=A^b \bmod p$$. Alice and Bob both end up with the same random secret key $$K$$, yet as far as we know no eavesdropper can recover $$K$$ in any reasonable amount of time.

## A Toy Example

To make the protocol concrete, consider the following small (and deliberately insecure) parameters:

- Prime modulus \( p = 23 \)
- Generator \( g = 5 \)
- Alice’s secret \( a = 6 \)
- Bob’s secret \( b = 15 \)

Alice computes her public value:

\[
A = g^a \bmod p = 5^6 \bmod 23 = 8
\]

Bob computes his public value:

\[
B = g^b \bmod p = 5^{15} \bmod 23 = 19
\]

Alice now uses Bob’s public value to obtain the shared secret:

\[
S = B^a \bmod p = 19^6 \bmod 23 = 2
\]

Bob uses Alice’s public value and obtains the same result:

\[
S = A^b \bmod p = 8^{15} \bmod 23 = 2
\]

Both parties arrive at the shared secret \( S = 2 \).

Of course a 5-bit prime is trivial to break by exhaustive search. In real systems the prime \( p \) is typically 2048 bits long (or an elliptic-curve group of comparable strength is used), making the discrete-logarithm problem far beyond the reach of current computation.

## Elliptic-Curve Diffie-Hellman

The Diffie-Hellman protocol can be instantiated with any cyclic group in which the discrete-logarithm problem is hard. The most important modern variant replaces modular arithmetic with elliptic-curve groups and is therefore called _Elliptic-Curve Diffie-Hellman_ (ECDH).

The Diffie-Hellman protocol can be realized with any suitable one-way function. The most widely used alternative to modular exponentiation is based on [elliptic curves](https://en.wikipedia.org/wiki/Elliptic_curve) and is called _Elliptic-Curve Diffie-Hellman_ (ECDH).

The protocol itself is identical:

- Alice and Bob agree on a public base point \(G\) on an elliptic curve.
- Alice chooses a secret integer \(a\) and sends the point \(A = a \cdot G\).
- Bob chooses a secret integer \(b\) and sends the point \(B = b \cdot G\).
- Each party multiplies the received point by its own secret, obtaining the same shared point \(S = a \cdot b \cdot G\).

Only the underlying mathematical one-way function has changed. The security of ECDH rests on the elliptic-curve discrete-logarithm problem rather than on the classic discrete-logarithm problem modulo a prime. The practical advantage is that ECDH achieves roughly the same security level as finite-field Diffie-Hellman with substantially smaller keys and faster arithmetic (for example, a 256-bit elliptic curve is considered comparable to a 2048-bit prime modulus).

For the purposes of this course it is enough to remember that the _idea_ of the key exchange remains identical: each party combines its own secret with the other party’s public value, and an eavesdropper who sees only the public values cannot efficiently compute the shared secret.

## Difficulty in Bits

It is generally believed that the discrete log problem is hard, but how hard? In practice, it is generally believed that computing the discrete log modulo a 2048b prime, computing the elliptic curve discrete log on a 256b curve, and brute forcing a 128b symmetric key algorithm are all roughly the same difficulty. (Brute-forcing the 128b key is believed to be slightly harder than the other two.)

Thus, if we are using Diffie-Hellman key exchange with other cryptoschemes, we try to relate the difficulty of the schemes so that it is equally difficult for an attacker to break any scheme. For example, 128b AES tends to be used with SHA-256 and either 256b elliptic curves or 2048b primes for Diffie-Hellman. Similarly, for top-secret use, the NSA uses 256b AES, 384b Elliptic Curves, SHA-384, and 3096b Diffie-Hellman and RSA.

## Attacks on Diffie-Hellman: The Man-in-the-Middle

Diffie-Hellman is secure against a passive eavesdropper (Eve) who can only observe the messages exchanged by Alice and Bob. It is _not_ secure against an active attacker (Mallory) who can modify, inject, or suppress messages.

Mallory can perform a classic man-in-the-middle attack as follows. When Alice sends her public value \(A = g^a \bmod p\), Mallory replaces it with her own value \(M = g^m \bmod p\). Likewise, when Bob sends \(B = g^b \bmod p\), Mallory replaces it with the same \(M\). After the exchange:

- Alice computes what she believes is the shared secret: \(K_A = M^a = g^{ma} \bmod p\)
- Bob computes a different value: \(K_B = M^b = g^{mb} \bmod p\)

Alice and Bob therefore do not share a key with each other. Instead, Alice shares a key with Mallory and Bob shares a different key with Mallory. Mallory, knowing \(m\), can compute both of those keys. She can now decrypt everything Alice sends, read or modify it, re-encrypt it under Bob’s key, and forward it, completely transparently.

The root cause of the attack is the absence of _authentication_ (and integrity protection) for the messages that carry the Diffie-Hellman public values. Nothing in the basic protocol lets Alice verify that the value she received really came from Bob, and vice versa.

A common misconception is that “we can just encrypt the subsequent traffic, so the attack does not matter”. This is incorrect. If the Diffie-Hellman exchange itself is unauthenticated, Alice and Bob end up encrypting under keys that Mallory already knows. Encryption performed _after_ a successful man-in-the-middle attack protects nothing; Mallory can still read and modify every message.

The standard defence is to authenticate the Diffie-Hellman exchange. In modern protocols this is typically achieved by having each party digitally sign the transcript of the key-exchange messages (or by binding the exchange to already-authenticated identities). Only when the public values are authenticated can Alice and Bob be confident that they share a secret with each other and not with Mallory.
