---
title: Certificates
parent: Cryptography
nav_order: 9
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Certificates

## The Public-Key Authenticity Problem

Public-key cryptography is a powerful tool. It allows two parties who have never met to establish confidentiality, integrity, and authenticity over an insecure network. While it solves the classic key-distribution problem of symmetric cryptography (which requires a shared secret to be established in advance), all of these guarantees rest on a single critical assumption: each party must be using the _correct_ public key of the other party.

If Alice encrypts a message under a public key that she believes belongs to Bob, but that key actually belongs to an attacker, the security of the entire system collapses. The cryptography itself may be perfect; the failure lies in the authenticity of the key.

In other words, public-key cryptography does not eliminate the need for trust; it merely shifts it. Instead of needing a secret channel to exchange a symmetric key, Alice now needs an _authentic_ channel (or some other reliable mechanism) to obtain Bob’s public key.

### The Classic Man-in-the-Middle Attack

Consider the following naïve protocol. Alice wants to communicate securely with Bob but does not yet know his public key. She simply asks him to send it over the network:

1. Alice → Bob: “Please send me your public key.”
2. Bob → Alice: Bob’s public key $PK_B$.
3. Alice encrypts her message under the key she just received and sends the ciphertext to Bob.

An active attacker (Mallory) can easily subvert this exchange:

1. Alice sends her request for Bob’s public key.
2. Mallory intercepts Bob’s reply and replaces $PK_B$ with Mallory’s own public key $PK_M$.
3. Alice, believing she has received Bob’s key, encrypts her secret message under $PK_M$.
4. Mallory intercepts the ciphertext, decrypts it with his private key, reads (and possibly modifies) the message, then re-encrypts it under Bob’s real public key $PK_B$ and forwards it to Bob.

Neither Alice nor Bob detects any problem. Alice thinks she is talking securely to Bob; Bob thinks he is receiving a message that only he can read. In reality, Mallory sits in the middle, able to read and alter every message. This is the classic man-in-the-middle (MITM) attack on public-key exchange.

The same attack works in both directions if Alice and Bob exchange public keys with each other. Mallory simply substitutes his own key in each direction and ends up sharing one key with Alice and a different key with Bob.

### You Cannot Bootstrap Trust

The attack succeeds because Alice has no way to verify that the public key she received actually belongs to Bob. Cryptography can protect the _confidentiality_ and _integrity_ of data once the correct keys are in place, but it cannot, by itself, create the initial authentic association between a public key and an identity.

This is a fundamental limitation: **you cannot bootstrap trust from nothing**. Trust has to start somewhere. Once a small kernel of trust exists (for example, a correctly known public key of a trusted party), that kernel can be used to establish further trusted keys, and those keys can then be used to secure everything else.

In other words, cryptography can _protect_ and _extend_ trust, but it cannot _originate_ it. Somewhere in the system there must already exist a small, trusted piece of information: a _kernel of trust_.

The rest of this topic explores the practical mechanisms that attempt to provide that initial kernel of trust in the real world.

## Solution 1: Trusted Directory Service

One natural approach to this key management problem is to use a trusted directory service: some organization that maintains an association between the name of each participant and their public key. Suppose everyone trusts Dirk the Director to maintain this association. Then any time Alice wants to communicate with someone, say Bob, she can contact Dirk to ask him for Bob’s public key. This is only safe if Alice trusts Dirk to respond correctly to those queries (e.g., not to lie to her, and to avoid being fooled by imposters pretending to be Bob): if Dirk is malicious or incompetent, Alice’s security can be compromised.

On first thought, it sounds like a trusted directory service doesn’t help, because it just pushes the problem around. If Alice communicates with the trusted directory service over an insecure communication channel, the entire scheme is insecure, because an active attacker can tamper with messages involving the directory service. To protect against this threat, Alice needs to know the directory service’s public key, but where does she get that from? One potential answer might be to hardcode the public key of the directory service in the source code of all applications that rely upon the directory service. So this objection can be overcome.

A trusted directory service might sound like an appealing solution, but it has a number of shortcomings:

- Trust: It requires complete trust in the trusted directory service. Another way of putting this is that everyone’s security is contingent upon the correct and honest operation of the directory service.
- Scalability: The directory service becomes a bottleneck. Everyone has to contact the directory service at the beginning of any communication with anyone new, so the directory service is going to be getting a lot of requests. It had better be able to answer requests very quickly, lest everyone’s communications suffer.
- Reliability: The directory service becomes a single central point of failure. If it becomes unavailable, then no one can communicate with anyone not known to them. Moreover, the service becomes a single point of vulnerability to denial-of-service attacks: if an attacker can mount a successful DoS attack on the directory service, the effects will be felt globally.
- Online: Users will not be able to use this service while they are disconnected. If Alice is composing an email offline (say while traveling), and wants to encrypt it to Bob, her email client will not be able to look up Bob’s public key and encrypt the email until she has connectivity again. As another example, suppose Bob and Alice are meeting in person in the same room, and Alice wants to use her phone to beam a file to Bob over infrared or Bluetooth. If she doesn’t have general Internet connectivity, she’s out of luck: she can’t use the directory service to look up Bob’s public key.
- Security: The directory service needs to be available in real time to answer these queries. That means that the machines running the directory service need to be Internet-connected at all times, so they will need to be carefully secured against remote attacks.

Because of these limitations, the trusted directory service concept is not widely used in practice, except in the context of messengers (such as Signal), where in order to send a message, Alice already has to be online.

In this case, the best approach is described as “trust but verify” using a key transparency mechanism. Suppose Alice and Bob discovered each others keys through the central keyserver. If they are ever in person, they can examine their devices to ensure that Alice actually has the correct key for Bob and vice versa. Although inconvenient, this acts as a check on a rogue keyserver, as the rogue keyserver would know there is at least a chance of getting caught.

## Solution 2: Trust on First Use (TOFU / Leap-of-Faith)

One pragmatic approach to the public-key authenticity problem is known as _Trust on First Use_ (TOFU), also called _leap-of-faith authentication_ or _key continuity management_.

### How TOFU Works

The idea is simple. The first time Alice encounters Bob (or a server), she has no prior authentic information about his public key. She therefore takes a “leap of faith”: she accepts whatever public key is presented to her on that first contact and stores it locally. From that moment onward she treats the stored key as authoritative. On every subsequent connection she verifies that the key presented by the other party matches the one she previously recorded. If the key ever changes, she treats the change as a potential attack and refuses the connection (or at least raises a strong warning).

In effect, TOFU converts an unauthenticated first contact into a long-term association. Security after the first use is strong, provided the key is never changed by the legitimate party without notice.

### SSH: The Classic Real-World Example

The most widely deployed example of TOFU is the Secure Shell (SSH) protocol. When a user connects to a new server for the first time, the SSH client displays a message similar to:

> The authenticity of host 'example.com' can't be established.  
> ECDSA key fingerprint is SHA256:….  
> Are you sure you want to continue connecting (yes/no)?

If the user answers “yes”, the client stores the server’s public key in a local file (commonly `~/.ssh/known_hosts`). On all future connections the client checks that the server still presents the same key. If a different key appears, SSH aborts the connection and prints a prominent warning that the host identification has changed (exactly the behaviour expected under a TOFU policy).

Because manually verifying every server’s key out-of-band is often impractical, SSH relies on this leap-of-faith model for ordinary use.

### Strengths and Weaknesses

**Strengths**

- Extremely easy for users; almost no cryptographic knowledge is required.
- After the first successful contact, the association is protected against later man-in-the-middle attacks.
- No reliance on a central authority or online directory.
- Works well for long-lived servers whose keys change infrequently.

**Weaknesses**

- Completely vulnerable on the _first_ connection. An attacker who is present at the moment of first contact can substitute their own public key and thereafter impersonate the server indefinitely.
- Key changes by the legitimate party (for example, after a server re-installation) produce the same warning as an attack, creating usability friction and the risk that users will simply click through the warning.
- Provides no protection against an attacker who can persistently intercept the initial connection.

TOFU is therefore a deliberate engineering trade-off: it sacrifices security on the very first contact in exchange for simplicity and good security thereafter.

### Usable Security

Key continuity management exemplifies several design principles for “usable security”. One principle is that “there should be only one mode of operation, and it should be secure.” In other words, users should not have to configure their software specially to be secure. Also, users should not have to take an explicit step to enable security protections; the security should be ever-present and enabled automatically, in all cases. Arguably, users should not even have the power to disable the security protections, because that opens up the risk of social engineering attacks, where the attacker tries to persuade the user to turn off the cryptography.

Another design principle: “Users shouldn’t have to understand cryptography to use the system securely.” While it’s reasonable to ask the designers of the system to understand cryptographic concepts, it is not reasonable to expect users to know anything about cryptography.

## Solution 3: Web of Trust

Another approach is the so-called web of trust, which was pioneered by PGP, a software package for email encryption. The idea is to democratize the process of public key verification so that it does not rely upon any single central trusted authority. In this approach, each person can issue certificates for their friends, colleagues, and others whom they know.

Suppose Alice wants to contact Doug, but she doesn’t know Doug. In the simplest case, if she can find someone she knows and trusts who has issued Doug a certificate, then she has a certificate for Doug, and everything is easy.

If that doesn’t work, things get more interesting. Suppose Alice knows and trusts Bob, who has issued a certificate to Carol, who has in turn issued a certificate to Doug. In this case, PGP will use this certificate chain to identify Doug’s public key.

In the latter scenario, is this a reasonable way for Alice to securely obtain a copy of Doug’s public key? It’s hard to say. For example, Bob might have carefully checked Carol’s identity before issuing her a certificate, but that doesn’t necessarily indicate how careful or honest Carol will be in signing other people’s keys. In other words, Bob’s signature on the certificate for Carol might attest to Carol’s identity, but not necessarily her honesty, integrity, or competence. If Carol is sloppy or malicious, she might sign a certificate that purports to identify Doug’s public key, but actually contains some imposter’s public key instead of Doug’s public key. That would be bad.

This example illustrates two challenges:

- **Trust isn’t transitive.** Just because Alice trusts Bob, and Bob trusts Carol, it doesn’t necessarily follow that Alice trusts Carol. (More precisely: Alice might consider Bob trustworthy, and Bob might consider Carol trustworthy, but Alice might not consider Carol trustworthy.)
- **Trust isn’t absolute.** We often trust a person for a specific purpose, without necessarily placing absolute trust in them. To quote one security expert: “I trust my bank with my money but not with my children; I trust my relatives with my children but not with my money.” Similarly, Alice might trust that Bob will not deliberately act with malicious intent, but it’s another question whether Alice trusts Bob to very diligently check the identity of everyone whose certificate he signs; and it’s yet another question entirely whether Alice trusts Bob to have good judgement about whether third parties are trustworthy.

The web-of-trust model doesn’t capture these two facets of human behavior very well.

The PGP software takes the web of trust a bit further. PGP certificate servers store these certificates and make it easier to find an intermediary who can help you in this way. PGP then tries to find multiple paths from the sender to the recipient. The idea is that the more paths we find, and the shorter they are, the greater the trust we can have in the resulting public key. It’s not clear, however, whether there is any principled basis for this theory, or whether this really addresses the issues raised above.

One criticism of the web-of-trust approach is that, empirically, many users find it hard to understand. Most users are not experts in cryptography, and it remains to be seen whether the web of trust can be made to work well for non-experts. To date, the track record has not been one of strong success. Even in the security community, it is only partially used—not due to lack of understanding, but due to usability hurdles, including lack of integration into mainstream tools such as mail readers.

## Solution 4: Digital Certificates

Trust on First Use is simple, but it leaves the first contact unprotected. A more systematic approach is to introduce a trusted third party that _vouches_ for the binding between an identity and a public key. That voucher is called a _digital certificate_.

### What a Certificate Is

A digital certificate is a digitally signed statement of the form:

> “This public key $PK$ belongs to the entity named $X$.”

The statement is signed by a trusted party using its private key. Anyone who possesses the corresponding public key of the signer can verify the signature and thereby gain confidence that the signer really made the assertion. Because the certificate is just data, it can be freely copied, stored, and transmitted over insecure channels; its authenticity is protected by the signature, not by the channel.

In practice, a certificate also contains additional information such as a validity period, the algorithms used, and various extensions, but the essential content remains the binding of an identity to a public key, attested by a signature.

### The Role of the Certificate Authority

The party that creates and signs certificates is called a _Certificate Authority_ (CA). The CA’s job is to perform whatever checks are necessary to convince itself that the claimed identity really controls the public key being certified, and then to issue a signed certificate recording that fact.

Once the CA has issued the certificate, it no longer needs to be online for ordinary use. The certificate can be presented by its owner (or by anyone else) to any relying party. The CA’s ongoing role is mainly administrative: deciding whom to certify, managing the lifecycle of certificates, and handling revocation when necessary.

### Offline Verification

Suppose Alice already possesses an authentic copy of the CA’s public key (we will discuss how you get that later). When she receives a certificate that claims to bind Bob’s name to a public key $PK_B$, she performs the following steps:

1. She verifies the digital signature on the certificate using the CA’s public key.
2. If the signature is valid, she knows the CA really issued that statement.
3. If she trusts the CA to have performed its checks correctly, she now believes that $PK_B$ belongs to Bob.
4. She can thereafter use $PK_B$ to encrypt messages to Bob or to verify signatures from Bob.

All of these steps can be performed _offline_. Alice does not need to contact the CA at the moment she wants to communicate with Bob; she only needs the certificate and the CA’s public key. This is the crucial practical advantage of certificates over an online trusted directory: once the initial trust in the CA’s public key is established, further key authentications can be performed without real-time interaction with the CA.

## The Root of Trust

Trust has to start somewhere. A relying party must ultimately possess at least one public key that it already believes to be authentic. In modern systems, that initial belief is established by _hardcoding_.

### How Trust Is Bootstrapped in Practice

When you install an operating system or a web browser, the software comes pre-loaded with a collection of public keys belonging to a set of well-known Certificate Authorities. These pre-installed keys are called _root certificates_ (or simply _roots_). Because they are embedded in the software by the vendor, they travel to the user through the same channel that delivered the operating system or browser itself (an out-of-band path that is assumed to be authentic at installation time).

Once the roots are present on the machine, they serve as the starting points for all subsequent certificate validation. Any certificate chain (we will discuss in the next section) that terminates at one of these roots can be verified offline, exactly as described in the previous section.

### The Trust Store

The collection of root certificates is known as the _trust store_ (sometimes called the root store). Every major operating system and browser maintains its own trust store:

- Windows, macOS, and the various Linux distributions each ship with a system-wide set of trusted roots.
- Browsers such as Firefox maintain an independent trust store of their own.
- Mobile operating systems likewise embed a list of roots.

The trust store is the concrete realization of the “kernel of trust” discussed earlier. Everything the machine accepts as a valid public key ultimately derives its credibility from one of the keys in this store.

## Hierarchical PKI and Certificate Chains

A single Certificate Authority cannot realistically issue certificates for every person, organization, or website in the world. There are simply too many entities, and no one organization can personally verify the identity of all of them. Even a national government or a large commercial CA would be overwhelmed if it tried to sign every certificate itself.

The practical solution is _delegation_. A CA can issue a certificate not only to end users, but also to other CAs, authorizing them to issue certificates on its behalf. The subordinate CA is then free to certify further parties (or even further subordinate CAs). Trust is thereby extended through a hierarchy rather than through a single central authority.

### Certificate Chains

When trust is delegated, a relying party (like Alice) typically receives a _certificate chain_ (also called a certification path). A certificate chain is a sequence of certificates in which:

- The first certificate (the _end-entity_ or _leaf_ certificate) binds the public key of the party Alice actually wants to communicate with (for example, a web server).
- Each subsequent certificate binds the public key of the CA that signed the previous certificate.
- The chain ends at a certificate whose signer is a CA that Alice already trusts (a _root_ CA).

In the simplest non-trivial case, the chain has three certificates:

> Root CA → Intermediate CA → End-entity (leaf)

Longer chains are possible; a root may authorize an intermediate, which authorizes another intermediate, and so on, until the final leaf certificate is reached.

### How a Browser Validates a Chain

When a web browser (or any other relying party) receives a certificate chain, it performs roughly the following checks:

1. It verifies the signature on the leaf certificate using the public key contained in the next certificate in the chain.
2. It verifies the signature on that intermediate certificate using the public key of the certificate above it.
3. It continues walking up the chain until it reaches a certificate that is signed by a CA whose public key is already present in the browser’s local _trust store_ (the set of pre-installed root CA public keys).
4. Along the way, it also checks validity periods, key-usage extensions, name constraints, and other policy rules.

If every signature verifies correctly and the chain terminates at a trusted root, the browser accepts the leaf public key as authentic for the claimed identity. If any link in the chain fails, the entire chain is rejected.

This hierarchical design gives Public-Key Infrastructure its scalability: the root CAs need only certify a manageable number of intermediate CAs, while the intermediate CAs handle the large volume of end-entity certificates. At the same time, it preserves the core idea that trust ultimately rests on a small set of root keys that the relying party already possesses.

### Implications of Trusting Many Root CAs

In practice, the trust stores of modern systems contain a large number of roots, often well over one hundred. This design has a serious consequence: the browser (or operating system) will accept a certificate issued by _any_ of those roots. There is no notion of “partial” or “proportional” trust. A certificate signed by a small, little-known CA is treated with exactly the same authority as one signed by a large, globally recognized CA.

The security of the entire system is therefore only as strong as its weakest root. If any single CA in the trust store is compromised, behaves maliciously, or is coerced into issuing a fraudulent certificate, that certificate will be accepted by every relying party that trusts the corresponding root. An attacker who obtains a fraudulent certificate for a major website can then perform a man-in-the-middle attack against users of that site, and the users’ browsers will display the familiar padlock because the certificate chains to a trusted root.

This _weakest-link_ property is an inherent characteristic of the current CA model. It is one of the main reasons the public-key infrastructure used on the web continues to attract criticism and ongoing efforts at improvement.

## Revocation

A digital certificate is a signed statement. Once issued, it remains cryptographically valid until its expiration date (or indefinitely if no expiration is specified). In the real world, however, circumstances change. A private key may be stolen, a certificate may have been issued by mistake, or an organization may no longer wish to be associated with a particular key. In all of these cases the certificate must be _revoked_, that is, relying parties must be told to stop trusting it.

Without a revocation mechanism, a compromised or erroneously issued certificate can continue to be accepted until it naturally expires, which may be months or years later. History has shown that this is a serious problem; several well-known incidents involved fraudulent certificates that could not be promptly invalidated.

This problem has arisen in practice. A number of years ago, Verisign issued bogus certificates for “Microsoft Corporation” to … someone other than Microsoft. It turned out that Verisign had no way to revoke those bogus certificates. This was a serious security breach, because it provided the person who received those certificates with the ability to run software with all the privileges that would be accorded to the real Microsoft. How was this problem finally resolved? In the end, Microsoft issued a special patch to the Windows operating system that revoked those specific bogus certificates. The patch contained a hardcoded copy of the bogus certificates and inserted an extra check into the certificate-checking code: if the certificate matches one of the bogus certificates, then treat it as invalid. This addressed the particular issue, but was only feasible because Microsoft was in a special position to push out software to address the problem. What would we have done if a trusted CA had handed out a bogus certificate for Amazon.com, or Paypal.com, or BankofAmerica.com, instead of for Microsoft.com?

This example illustrates the need to consider revocation when designing a PKI system. There are two standard approaches to revocation:

### Two Main Approaches

**Short validity periods.**  
Certificates are issued with a deliberately short lifetime (days or even hours). When a certificate approaches expiration, its owner must request a new one from the CA. If the CA has learned that a certificate should no longer be trusted, it simply refuses to issue a replacement. Once the short lifetime elapses, the old certificate becomes invalid everywhere. This approach requires no special revocation infrastructure; ordinary expiration does the work.

**Certificate Revocation Lists (CRLs).**  
The CA maintains a list of all certificates it has revoked, signs the list, and publishes it periodically. Relying parties download the latest CRL and, when validating a certificate, check that its serial number does not appear on the list. Because the CRL is signed, it can be distributed over insecure channels. A certificate can be placed on the CRL as soon as the CA decides it should be revoked, giving the possibility of relatively prompt invalidation.

### Trade-offs

Each approach involves a tension among three goals: _promptness_ of revocation, _reliability_ of the system, and _load_ on the infrastructure.

- Short lifetimes give automatic, eventual revocation without any extra messages, but they impose a continuous burden on both the CA and the certificate holders. If the CA is unreachable when a certificate needs renewal, legitimate parties may be locked out. Very short lifetimes also increase the risk that temporary network problems will disrupt service.
- CRLs allow a CA to revoke a certificate almost immediately, but they introduce their own difficulties. The lists can become large, clients must obtain fresh copies regularly, and an attacker who can deny access to the CRL distribution point may be able to keep a revoked certificate looking valid. Clients then face an awkward policy choice: should they accept a certificate when they cannot obtain a recent CRL (risking the use of a revoked certificate), or should they reject it (risking a denial-of-service attack on the whole system)?

In practice both mechanisms are used, often in combination. Modern systems also employ online protocols such as the Online Certificate Status Protocol (OCSP) that let a client ask a CA (or a designated responder) about the status of an individual certificate in real time. Nevertheless, the fundamental trade-offs among promptness, reliability, and load remain, and no revocation scheme is perfect.

## Practical Realities and Limitations of the CA Model

The public-key infrastructure used on the web is a remarkable engineering achievement: it allows billions of users to establish encrypted connections to millions of servers without any prior arrangement. At the same time, it has well-known structural limitations that every security practitioner should understand.

### Many Trusted Roots, Increased Risk

As we mentioned, modern browsers and operating systems ship with a large trust store, often more than one hundred root certificates. Because a certificate issued by _any_ of those roots is accepted, the security of the entire system is only as strong as its weakest root. A single compromised, coerced, or negligent CA can issue a fraudulent certificate for any domain and have that certificate accepted worldwide. The more roots a system trusts, the larger the attack surface.

History has repeatedly demonstrated the consequences of this design.

In 2011 the Dutch CA DigiNotar was completely compromised. Attackers issued hundreds of rogue certificates, including certificates for Google domains that were subsequently used to intercept the traffic of approximately 300,000 users. The breach was so severe that DigiNotar’s roots were removed from major trust stores and the company later went bankrupt.

In 2012–2013 the Turkish CA TURKTRUST mistakenly issued intermediate CA certificates to customers instead of ordinary end-entity certificates. Those intermediate certificates conferred the power to issue further certificates for arbitrary domains, an error that effectively gave the recipients the authority of a root CA. The incident again required emergency updates to browser trust stores.

These events (and several others of similar character) show that the theoretical weakest-link property is not merely theoretical.

### Validation Levels

Not every certificate is issued with the same degree of scrutiny. Certificate Authorities commonly distinguish three levels of validation:

- **Domain Validated (DV).** The CA verifies only that the applicant controls the domain in question (for example by e-mail challenge, DNS record, or HTTP file). Issuance can be fully automated and completed in minutes. DV certificates prove control of a domain name, nothing more.
- **Organization Validated (OV).** The CA additionally checks the legal existence of the organization, typically by consulting business registries and contacting the organization. More information appears in the certificate, but most browsers still display only a generic padlock.
- **Extended Validation (EV).** The highest commercial level. The CA follows strict, audited procedures that include human review and verification against government databases. EV certificates were introduced largely to combat phishing sites that obtained cheap DV certificates.

In practice, the visual difference among these levels has largely disappeared from modern browsers; users see essentially the same padlock regardless of the validation type.

### Why the Padlock Is Not Absolute Proof of Safety

The familiar padlock icon indicates that the browser successfully validated a certificate chain ending at a trusted root and that the subsequent traffic is encrypted. It does _not_ guarantee:

- that the CA performed thorough identity checks,
- that the private key corresponding to the certificate has never been compromised,
- that the server operator is trustworthy or competent, or
- that the content of the site is free from malware or phishing.

A fraudulent certificate that chains to any trusted root will produce a padlock. Consequently, the padlock is evidence of a successful cryptographic handshake under the current PKI rules; it is not a complete assurance of safety. Users and developers must still apply independent judgment about the parties with whom they communicate.
