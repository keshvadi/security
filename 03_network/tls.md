---
title: TLS
parent: Network Security
nav_order: 8
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# TLS: Transport Layer Security

## Cheat sheet

- **Purpose**: Provide end-to-end confidentiality, integrity, and authentication between a client and server over an untrusted network.

- **Layer**: Runs on top of TCP (Layer 4). Often described as a "Layer 4.5" protocol because it sits between the transport layer and applications.

- **Security Guarantees**:

  - Confidentiality: Data is encrypted so intermediaries cannot read it.
  - Integrity: Any modification to messages is detected.
  - Server Authentication: The client can verify it is talking to the legitimate server (via certificates).
  - Forward Secrecy: Compromise of long-term keys does not expose past session keys (mandatory in TLS 1.3).
  - Replay Protection: Old messages cannot be replayed to establish new sessions or within an existing session.

- **Key Attacks & Defenses**:
  - Downgrade Attacks (forcing weaker ciphers or older TLS versions): Mitigated in TLS 1.3 by making version and parameter negotiation explicit and verifiable.
  - TLS Stripping / HTTPS Downgrade: An attacker downgrades HTTPS links to HTTP. Defended by HTTP Strict Transport Security (HSTS).
  - Rogue Certificates / CA Compromise: Attacker obtains a fraudulent but cryptographically valid certificate. Mitigated by Certificate Transparency (CT) logs and certificate pinning.
  - Historical crypto attacks (BEAST, POODLE, CRIME, Lucky13, etc.): Mostly mitigated by modern AEAD ciphers, encrypt-then-MAC, and removal of legacy features in TLS 1.3.

---

## Why TLS?

Imagine you are checking your email at an Internet cafe. The Wi-Fi router has been compromised by a local attacker. Your traffic then travels through an ISP that is actively trying to spy on its customers. On top of that, you are in a country whose government wants to monitor or interfere with citizens' communications. Every device between your laptop and the mail server may be malicious or compromised.

In this environment, can you still communicate securely with your email provider?

This is exactly the threat model TLS was designed to defeat. We assume a powerful _network attacker_ who can:

- Eavesdrop on every packet (passive confidentiality attack)
- Modify, inject, delete, or replay packets at will (active integrity and replay attacks)
- Impersonate either the client or the server (authentication attacks)
- Control large parts of the network infrastructure, including routers, DNS servers, or even BGP routes

Despite this attacker, TLS aims to provide **end-to-end security** between your client and the _intended_ server. "End-to-end" means that no intermediary, however malicious, should be able to read your messages, modify them without detection, trick you into talking to the wrong server, or replay old messages to gain unauthorized access.

The only thing TLS requires from the underlying network is a _reliable, in-order byte stream_. TCP provides exactly that. TLS sits on top of TCP and adds the missing security properties. From the application's point of view, a TLS connection looks almost identical to a normal TCP connection; the application simply reads and writes bytes, and TLS handles all the cryptography transparently.

This is a remarkably strong guarantee. It is the reason HTTPS (HTTP over TLS) became the foundation of secure communication on the modern Internet, and why virtually every other application protocol that needs security (such as email submission using the `STARTTLS` command to upgrade a plaintext connection, VPNs, database connections, etc.) eventually adopted TLS or a close relative.

## TLS in the Network Stack and a Brief History

The original OSI seven-layer model did not include security as a distinct layer. TLS therefore sits in an awkward place: it runs on top of TCP (Layer 4) and below the application (Layer 7). It is often called a "Layer 4.5" or "Layer 6.5" protocol in informal discussion. The important point is that it provides security services to _any_ application that uses TCP, without requiring changes to the transport or network layers.

From a programmer's perspective, the TLS API is deliberately similar to the classic Berkeley sockets API. A program that already knows how to open a TCP connection and read/write bytes can usually be converted to TLS with only a few extra lines of code. This design decision was crucial to TLS's rapid adoption.

### Evolution from SSL to TLS 1.3

TLS evolved from SSL (Secure Sockets Layer), originally developed by Netscape in the mid-1990s:

- SSL 1.0 (1994): Never publicly released; internal design only.
- SSL 2.0 (1994): First public version; contained serious cryptographic and protocol flaws.
- SSL 3.0 (1996): Much improved, but still had weaknesses (notably the POODLE attack later).
- TLS 1.0 (1999): IETF standardization based on SSL 3.0; the first version officially named "TLS."
- TLS 1.1 (2006) and TLS 1.2 (2008): Incremental improvements, better ciphers, explicit IVs, etc.
- TLS 1.3 (2018): A major redesign. Removed legacy features that had caused problems, _mandated_ forward secrecy, simplified the handshake, and made the protocol much harder to attack.

Today, modern clients and servers use TLS 1.2 or 1.3. SSL is considered obsolete and is disabled by default everywhere.

## The TLS Handshake: Establishing Trust and Secrets

The heart of TLS is the **handshake**. Before any application data is sent, the client and server must:

1. Agree on which cryptographic algorithms to use.
2. Authenticate the server (and optionally the client).
3. Establish a set of shared secret keys that no one else knows.
4. Verify that the handshake itself was not tampered with.

Only after a successful handshake does the connection switch to encrypted application data.

Because TLS runs on top of TCP, the TLS handshake begins only after the TCP three-way handshake has completed. This lets TLS treat the connection as a reliable stream of messages rather than worrying about lost packets, reordering, or retransmissions.

### ClientHello and ServerHello

The client begins by sending a `ClientHello` message containing:

- A fresh random value $R_B$ (client random).
- A list of cipher suites the client supports (each cipher suite names a key-exchange method, a bulk encryption algorithm, and an integrity/MAC algorithm).
- Optionally, the Server Name Indication (SNI) extension telling the server which hostname the client is trying to reach (important for virtual hosting).
- Supported elliptic curves, signature algorithms, and other extensions.

The server replies with a `ServerHello` that selects:

- Its own fresh random value $R_S$ (server random).
- One cipher suite from the client's list.
- The server's digital certificate (containing its public key and identity, signed by a Certificate Authority).

At this point the client can verify the server's certificate chain. If the chain ends at a root CA that the client trusts (pre-installed in the operating system or browser), and all signatures and validity periods check out, the client now has a trusted copy of the server's public key.

Important sanity check: After the ServerHello and certificate, can the client be certain it is talking to the genuine server?  
Answer: No. An active attacker could have obtained a copy of the legitimate server's certificate (by connecting to it itself) and presented that same certificate to the victim client. The client needs more than just a certificate; it needs proof that the _other party_ actually possesses the corresponding private key.

### Establishing the Premaster Secret (Key Exchange)

The next step is for the client and server to agree on a secret value called the **Premaster Secret (PS)** that only they will know. The PS must satisfy two properties:

- An eavesdropper who sees every message must not be able to compute the PS.
- Only the legitimate client and the legitimate server (the one that knows the private key corresponding to the certificate) should be able to derive the PS.

There are two historically important ways to achieve this.

1. RSA key exchange (legacy, removed in TLS 1.3)

The client generates a random PS, encrypts it under the server's public key (from the certificate), and sends the ciphertext to the server. The server decrypts it with its private key. Only the server can recover the PS.

This method is simple, but it provides **no forward secrecy**. If an attacker later steals the server's long-term private key, they can decrypt every PS that was ever sent to that server in the past, and therefore decrypt every past session.

2. Diffie-Hellman Ephemeral (DHE / ECDHE) key exchange (modern, mandatory in TLS 1.3)

The client and server perform a classic Diffie-Hellman exchange (or its elliptic-curve variant ECDHE). The server signs its half of the exchange with its long-term private key so the client can verify it came from the certified server. The resulting shared secret becomes the PS.

Because the DH exponents are freshly generated for each connection and never transmitted, an attacker who later steals the server's long-term private key still cannot reconstruct past PS values. This property is called **forward secrecy** (or perfect forward secrecy). Since TLS 1.3, RSA key exchange is forbidden precisely because it lacks forward secrecy.

Here are the diagrams from the teaching materials that illustrate the two approaches:

<img src="{{ site.baseurl }}/assets/images/tls1.png" alt="Diagram of the first part of the TLS handshake, from the ClientHello to the server certificate presentation" width="75%">

<img src="{{ site.baseurl }}/assets/images/tls2-rsa.png" alt="Diagram of the second part of the TLS handshake using RSA, from the server certificate presentation to the exchange of MACs" width="40%">

<img src="{{ site.baseurl }}/assets/images/tls2-dh.png" alt="Diagram of the second part of the TLS handshake using Diffie-Hellman, from the server certificate presentation to the exchange of MACs" width="40%">

### Deriving Session Keys and the Finished Messages

Once both sides have the PS and the two random values $R_B$ and $R_S$, they independently derive a set of symmetric keys using a key-derivation function (historically based on MD5+SHA1 or HMAC, now HKDF in TLS 1.3). Typically this produces:

- Client encryption key $C_B$
- Server encryption key $C_S$
- Client integrity (MAC) key $I_B$
- Server integrity (MAC) key $I_S$

The public random values $R_B$ and $R_S$ exchanged at the start of the handshake are critical for defending against cross-connection replay attacks. If an attacker recorded an old RSA-based TLS handshake and replayed the encrypted Premaster Secret, but $R_B$ and $R_S$ were always zero, the server would re-generate the exact same symmetric keys. By requiring new, randomly generated values for $R_B$ and $R_S$ every time, each connection results in a uniquely different set of symmetric keys, causing replay attempts to fail.

Before switching to encrypted traffic, the client and server exchange `Finished` messages. Each side computes a MAC (or HMAC or AEAD tag in modern TLS) over _every handshake message seen so far_, using its integrity key, and sends the result to the other party. The other party verifies the MAC using the corresponding key it derived.

Sanity check: Up until the Finished messages, every byte of the handshake has been sent in plaintext. Why is this not a fatal problem?  
Answer: The Finished MAC covers the entire transcript. If an attacker tampered with any earlier message (for example, removing a strong cipher suite from the ClientHello), the Finished MACs will not match and the connection will be aborted.

At the end of a successful handshake we have:

1. The client has verified that it is talking to a server whose identity matches the certificate and that possesses the corresponding private key.
2. Both sides have confirmed that the handshake transcript was not modified.
3. Both sides share fresh, secret symmetric keys known only to them.

### Protecting Application Data: The Record Protocol

After the handshake, every application message is processed by the _TLS record protocol_ before being handed to TCP:

- The data is optionally compressed (removed in TLS 1.3 because compression enabled CRIME and BREACH attacks).
- A MAC or AEAD tag is computed over the data (plus a sequence number to prevent replay within the connection).
- The data + MAC is encrypted with the appropriate symmetric key.
- A record header (type, version, length) is prepended.

In TLS 1.2 and earlier the record layer used a "MAC-then-encrypt" construction with CBC or RC4, which proved fragile (BEAST, Lucky13, POODLE). TLS 1.3 mandates AEAD algorithms (AES-GCM, ChaCha20-Poly1305) that combine encryption and authentication in a single, provably secure primitive. The record layer is now much simpler and safer.

A TLS record on the wire looks roughly like: [Content Type (1 byte)] [Protocol Version] [Length (2 bytes)] [Encrypted+Authenticated Payload]. The payload contains the application data (or handshake messages, alerts, etc.) plus the integrity tag. Sequence numbers (implicit, not sent on the wire) are included in the authenticated data so that an attacker cannot reorder or drop records without detection.

## Downgrade Attacks

Even when both client and server support strong cryptography, an active attacker can try to force them to use weaker options. These are **downgrade attacks**.

**Ciphersuite downgrade**: The attacker removes strong cipher suites from the ClientHello (or the server's selection) so that only weak ones remain. In TLS 1.2 and earlier this was hard to detect because the attacker could simply rewrite the list before the Finished MACs were computed.

**Version downgrade**: The attacker forces the use of TLS 1.0 or 1.1 even though both sides support 1.3. Older versions had known weaknesses.

TLS 1.3 defends against version downgrade by embedding a special value (the "downgrade sentinel") in the ServerHello random field when the server is _willing_ to negotiate an older version. A genuine TLS 1.3 client that sees this sentinel in a ServerHello that claims to be TLS 1.3 immediately aborts. Because the sentinel is inside the ServerHello (which is covered by the Finished MAC and, in 1.3, by the signature), the attacker cannot forge it without detection.

## TLS Stripping and HTTP Strict Transport Security (HSTS)

Even a perfect TLS handshake is useless if the client never attempts TLS in the first place. An attacker who can modify HTTP responses can simply rewrite an `https://` link into `http://` or remove the `https` redirect. This is a **TLS stripping** or **HTTPS downgrade** attack.

The primary defense is **HTTP Strict Transport Security (HSTS)**. A server that wants to be HTTPS-only can send the header:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

This tells the browser: "For the next year (or whatever the max-age says), never connect to this domain over plain HTTP. If you ever see an `http://` link, automatically upgrade it to `https://` before sending the request."

Once the browser has seen and stored this policy, even a later attacker who strips the header cannot force a plaintext connection—the browser itself enforces HTTPS.

Limitations and the bootstrap problem: On the very first visit, or after the max-age expires, the browser has no stored policy. An attacker can still strip during that window. The _HSTS preload list_ (shipped with browsers) solves this for important sites by baking the policy in at compile time. Subdomain coverage requires the `includeSubDomains` directive.

HSTS, especially with preload, is the most effective practical defense against stripping today.

## The Certificate Ecosystem: The Real Achilles' Heel

All the beautiful cryptography in TLS is worthless if the client cannot be sure that the public key in the certificate actually belongs to the server it intended to reach. This is the _identity problem_, and TLS solves it with _public-key certificates_ issued by _Certificate Authorities (CAs)_ (discussed in detail in previous unit).

As we learnd, a CA is an organization that is trusted (by virtue of being included in the browser/OS root store) to verify that a particular public key belongs to a particular domain name. When you request a certificate for `example.com`, the CA performs some validation (domain control via DNS, HTTP, or email) and then signs a certificate containing your public key and the name `example.com`.

Browsers trust a few hundred root CAs by default. Any of those roots (or any intermediate CA they have signed) can issue a certificate for _any_ domain. This is _delegated trust_ at global scale: instead of every user verifying every key, a small number of CAs do the verification on everyone's behalf.

The system is only as strong as its weakest CA. If any trusted CA is compromised or acts maliciously, it can issue a cryptographically valid certificate for any domain, and every browser will accept it without warning.

**DigiNotar (2011)**: Attackers compromised a Dutch CA and issued rogue certificates for Google and other high-value domains. Because DigiNotar's root was trusted everywhere, the certificates worked for man-in-the-middle attacks on hundreds of thousands of users. Once discovered, every major browser removed DigiNotar from its trust store; the company went bankrupt within weeks.

**TURKTRUST (2013)**: A configuration error caused a Turkish CA to issue two intermediate CA certificates to ordinary customers. One of those intermediates ended up on a corporate firewall, giving that organization the ability to generate on-the-fly valid certificates for any domain. Browsers blacklisted the specific misissued intermediates rather than removing the entire CA.

These incidents (and others) demonstrated that the CA trust model is a single point of catastrophic failure.

### Modern Mitigations: Certificate Transparency, Short-Lived Certificates, Automation

**Certificate Transparency (CT)** requires CAs to publish every certificate they issue to publicly auditable, append-only logs (implemented cryptographically as a hash chain). Domain owners can monitor the logs (via services or their own tools) and quickly detect fraudulent certificates for their domains. CT does not prevent issuance, but it makes secret issuance much harder and provides evidence for revocation.

**Certificate Revocation** is the process of publishing lists of no-longer-accepted certificates (Certificate Revocation Lists, or CRLs) so that browsers can verify if a certificate is still trusted. **OCSP stapling** improves this by letting the server itself provide a fresh, signed proof of validity with every handshake, improving both privacy and reliability compared with clients querying OCSP servers directly.

**Short-lived certificates** (days or weeks instead of years) reduce the value of a stolen or misissued certificate and shrink the window during which revocation matters.

**Let's Encrypt** (launched 2015) made certificates free and completely automatic via the ACME protocol. A server operator runs a small agent that proves control of the domain (by placing a file or DNS record) and obtains a certificate in seconds. This automation, combined with short lifetimes, has driven the massive increase in HTTPS adoption.

## Historical Cryptographic Attacks on TLS

TLS 1.2 and earlier contained a number of cryptographic weaknesses that were discovered over time. Understanding them helps explain why TLS 1.3 looks the way it does.

- BEAST (2011): Exploited predictable IVs in CBC-mode encryption with TLS 1.0. Allowed an attacker to decrypt sensitive cookies byte-by-byte.
- CRIME (2012) and BREACH (2013): Abused TLS compression (or HTTP compression inside TLS) to leak secrets via length side-channels.
- POODLE (2014): Downgraded connections to SSL 3.0 and exploited padding-oracle weaknesses in CBC.
- Lucky13 (2013) and related padding-oracle attacks: Showed that even with explicit IVs, CBC-mode MAC-then-encrypt constructions leak information through timing.
- Heartbleed (2014): A catastrophic implementation bug in OpenSSL's heartbeat extension that allowed attackers to read server memory (including private keys) without any cryptographic attack.

TLS 1.3 removed CBC mode entirely, removed compression, removed the heartbeat extension from the standard, mandated AEAD, and simplified the handshake to reduce the attack surface. Many of the attacks above are simply impossible against a correct TLS 1.3 implementation.

## What TLS Does _Not_ Protect

It is equally important to understand the limits of TLS:

- Metadata: IP addresses, port numbers, DNS queries, packet sizes, and timing are still visible to every network observer. Even perfect encryption does not hide _who is talking to whom_ or _when_.
- Traffic analysis and website fingerprinting: With enough packets, an observer can often identify which website (or even which page) is being visited despite encryption.
- DNS and BGP: TLS cannot protect you if an attacker has already redirected your traffic to a server under their control via DNS spoofing or BGP hijacking (see the BGP chapter). You will happily complete a TLS handshake with the _wrong_ server if its certificate is valid or if you have been tricked into accepting it.
- Client identity: By default TLS authenticates only the server. Client certificates are optional and rarely used for web browsing.
- 0-RTT data in TLS 1.3: For performance, TLS 1.3 allows clients to send data in the very first flight (0-RTT). This data is encrypted but not forward-secret and is vulnerable to replay. Applications must be careful not to use 0-RTT for non-idempotent operations.

These limitations are why defense in depth, combining TLS with HSTS, CT, secure DNS (DNSSEC, DoH), and network-layer protections, remains necessary.
