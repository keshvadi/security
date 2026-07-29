---
title: DNSSEC
parent: Network Security
nav_order: 11
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# DNSSEC: Cryptographic Authentication for DNS

## Cheat sheet

- **Purpose**: Add integrity and authentication (but not confidentiality) to DNS responses so resolvers can detect tampering and forgery.
- **Core Mechanism**: Each zone signs its records with private keys. Resolvers validate signatures using public keys obtained through a chain of trust anchored at the DNS root.
- **Key Record Types**:
  - **DNSKEY**: Public key used to verify signatures. Typically split into a **Key Signing Key (KSK)** and a **Zone Signing Key (ZSK)**.
  - **RRSIG**: Signature over a resource record set (RRset).
  - **DS**: Delegation Signer — a hash of a child zone's KSK, stored in the parent to establish the chain of trust.
  - **NSEC / NSEC3**: Prove the _non-existence_ of a name (negative proof) without revealing the entire zone.
- **Benefits**: Detects cache poisoning, on-path and off-path forgery, and some configuration errors. Does _not_ hide domain names from observers.
- **Deployment Status (mid-2020s)**: All root and most TLDs are signed. A growing but still minority fraction of second-level domains are signed. Resolver-side validation is available but not universal (major public resolvers do validate; many ISP resolvers do not).
- **Limitations**: Does not protect against every attack (e.g., authoritative server compromise, certain replay within TTL, or attacks on the signing ceremony itself). Operational complexity and key-rollover mistakes have caused real outages.

---

## DNSSEC

The previous topic showed that classic DNS has no meaningful authentication or integrity protection. An attacker who can win a race (or who is on the path) can inject arbitrary records, and resolvers will happily cache and serve them. This breaks every higher-layer security assumption that depends on "when I look up the name, I get the real address".

DNSSEC (DNS Security Extensions) adds exactly the missing properties, i.e., _integrity_ and _authentication_, using digital signatures. It lets a resolver prove that:

- The records it received for a name really came from the zone that is authoritative for that name.
- The records have not been altered in transit.
- A "no such name" (NXDOMAIN) answer is genuine and not a forgery.

It does _not_ provide confidentiality. DNS queries and the names in signed answers are still visible to observers. (Confidentiality is addressed by separate mechanisms: DNS over TLS / HTTPS and query-name minimization.)

Sanity check: Why do we deliberately _not_ try to hide the domain names themselves in DNSSEC?  
Answer: Because the primary job of DNS is to publish public mapping information. Hiding the names would require every resolver to have a prior relationship with every zone (impossible at Internet scale). DNSSEC's goal is to make the _published_ data trustworthy, not secret.

## The Basic Idea: Sign Every Record

The simplest mental model is:

1. Every zone generates public/private key pairs. In practice, to make key management easier, zones use **two** key pairs: a **Key Signing Key (KSK)** and a **Zone Signing Key (ZSK)**. The KSK is used only to sign the ZSK, while the ZSK is used to sign the actual zone records. This separation means a zone can frequently rotate its ZSK for security without needing to ask its parent zone to update the delegation records.
2. For every resource record set (RRset) the zone publishes (all the A records for a name, all the MX records, etc.), the zone computes a digital signature over the RRset using its private ZSK.
3. The signature is stored in a new record type: **RRSIG**.
4. The public keys are published in **DNSKEY** records inside the same zone.
5. A resolver that receives an answer also receives the RRSIG. It fetches the DNSKEY, verifies the signature, and accepts the data only if the signature is valid and was produced by a key that the resolver has reason to trust.

Because only the holder of the private key can create a valid RRSIG, a network attacker (on-path or off-path) cannot forge or modify records without invalidating the signature. Caching is safe because the signature itself can be cached and re-validated later.

## Who Do You Trust? (Delegating Trust from the Root)

If every zone simply signed its own data, we would still have the "who signs the signers" problem. A malicious authoritative server for `bank.com` could generate its own key, sign malicious records with it, and the resolver would have no way to know the key is not legitimate.

DNSSEC solves this with **delegated trust** anchored at the **DNS root**.

### The Root as Trust Anchor

Every validating resolver is pre-configured with the public key(s) of the DNS root zone (the "trust anchor"). The root is operated under extremely strict, transparent, and audited procedures (the famous root signing ceremonies that involve multiple people, smart cards, and physical security). Compromising the root is considered practically infeasible for all but the most powerful nation-state actors with sustained physical access.

### The Chain of Trust

Trust flows downward:

- The root signs a **DS (Delegation Signer)** record for each signed TLD (e.g., `.com`, `.ca`). The DS record contains a hash of the TLD's KSK.
- The TLD signs DS records for each signed second-level domain beneath it.
- Each child signs its own ZSK with its KSK, signs its own data with the ZSK, and publishes its DNSKEYs.

A resolver validating `www.bank.com` performs a "validation walk":

1. Start with the root trust anchor (known a priori).
2. Fetch the root's DNSKEY and verify it against the trust anchor.
3. Fetch the DS record for `.com` (signed by the root) and verify the signature using the root DNSKEY.
4. Use the hash in the DS to verify that the `.com` KSK the resolver just fetched is the correct one.
5. Repeat: fetch the DS for `bank.com` from `.com`, verify, obtain `bank.com`'s keys, verify the data for `www.bank.com`.

If any signature fails or a DS/DNSKEY hash does not match, the resolver treats the answer as _bogus_ and returns an error (usually SERVFAIL) to the application. The application never sees the forged data.

This creates a cryptographically verifiable chain from the root down to the leaf zone. No single compromised intermediate zone can forge data for a zone it does not control, because its DS record in the parent would not match the key it is using.

### Seeing DNSSEC in Action

You can observe these records in practice using the `dig +dnssec <domain>` command-line utility. The `+dnssec` flag adds an `OPT PSEUDOSECTION` to the query with the `DO` (DNSSEC OK) bit set, telling the server to include cryptographic records in the response.

For example, querying the `tru.ca` name server for its keys returns the public ZSK, the public KSK, and an RRSIG over both keys (signed by the KSK):

```shell
$ dig +norecurse +dnssec DNSKEY tru.ca @128.32.136.3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 1220
;; QUESTION SECTION:
;tru.ca.         IN  DNSKEY

;; ANSWER SECTION:
tru.ca.  172800  IN  DNSKEY  256 {ZSK of tru.ca}
tru.ca.  172800  IN  DNSKEY  257 {KSK of tru.ca}
tru.ca.  172800  IN  RRSIG   DNSKEY {signature on DNSKEY records}
```

## Proving Non-Existence: NSEC and NSEC3

A complete secure DNS must also be able to prove that a name _does not exist_ (or that a particular record type does not exist for a name). Without this, an attacker could simply suppress the real answer and claim "no such name".

Why is this difficult? Because public-key cryptography is computationally slow, name servers cannot dynamically generate signatures on the fly for every randomly generated non-existent domain query (doing so would create a massive Denial-of-Service vulnerability). Instead, all signatures must be computed offline in advance. Since a server cannot pre-compute signatures for an infinite number of non-existent names, DNSSEC pre-computes signatures for the gaps between existing names.

DNSSEC uses **NSEC** (or the privacy-preserving **NSEC3**) records. An NSEC record says, in effect: "There are no names between b.example.com and l.example.com in this zone." By returning the pre-signed NSEC record that alphabetically covers the queried non-existent name, the server proves that the name is genuinely absent without needing to compute a new signature online.

NSEC has a privacy side-effect: it reveals the entire list of names in the zone to anyone who asks for a non-existent name (zone walking). NSEC3 hashes the names and adds salt so that an attacker cannot easily enumerate the zone while still allowing resolvers to verify absence.

## Real-World Deployment and Operational Reality

As of the mid-2020s:

- The root and virtually all TLDs are DNSSEC-signed.
- Many large second-level domains (especially those run by security-conscious organizations, banks, and governments) are signed.
- Overall signed domain percentage is still well below 50% for the general web, but growing steadily.
- Major public resolvers (Google, Cloudflare, Quad9, etc.) perform DNSSEC validation by default. Many large ISPs do not, or do so only for a subset of customers.

Operational gotchas that have caused real outages:

- Forgetting to publish a new DS record in the parent when rolling the child's DNSKEY.
- Letting signatures expire (RRSIGs have their own validity periods).
- Misconfiguring NSEC3 parameters or key rollover timing.
- "DNSSEC stripping" by middleboxes or misbehaving recursive resolvers that remove or ignore the AD (Authenticated Data) bit.

DNSSEC, like any cryptographic system, requires careful key management and monitoring. When done correctly, however, it eliminates an entire class of previously unfixable attacks.

## What DNSSEC Does and Does Not Protect

**Protects**:

- Integrity and authenticity of DNS data against network attackers (the original poisoning and Kaminsky-style attacks).
- Negative answers (you really are at the right server, or the name really does not exist).
- Some accidental misconfigurations (a mis-typed delegation will fail validation).

**Does not protect**:

- Confidentiality of queries or answers (use DoH/DoT for that).
- Availability (an attacker can still drop packets or flood the resolver).
- Compromise of the authoritative server or its signing keys (if the zone's private key is stolen, the attacker can sign anything until the DS is updated in the parent).
- The root or TLD operators themselves (the trust anchor assumption).

DNSSEC is therefore a powerful but partial defense. It is most effective when combined with encrypted transport for the stub-to-recursive leg and with application-layer protections (TLS + certificate transparency).

## DNSSEC in Practice

DNSSEC is a beautiful example of "designing for the long term". The protocol was standardized in the late 1990s and early 2000s, but only became widely deployable after more than a decade of operational experience, tool improvements, and the painful lessons of large-scale outages caused by key-rollover mistakes.

The key insights that transfer to other systems are:

- **Authentication without confidentiality is sometimes exactly what you need.** DNS must be public; hiding the data would break the service. Signatures give you the security property you actually require without forcing a new architecture.
- **Delegation of trust must be explicit and verifiable.** The DS record is the technical embodiment of "I, the parent, vouch for this child's key." Without the DS link, the chain is broken.
- **Negative proofs are hard.** Proving that something does _not_ exist without revealing everything else is a recurring challenge (see also certificate revocation, non-membership in sets, etc.). NSEC/NSEC3 is one elegant solution.
- **Cryptography is easy; key management is hard.** The outages in DNSSEC have almost never been due to broken crypto. They have been due to humans forgetting to publish a DS, letting a signature expire, or mismanaging the rollover of a key that is used by millions of resolvers.

For the student, the practical takeaway is simple: turn on DNSSEC validation wherever you control a resolver, sign your own zones if you operate authoritative servers, and treat any DNS answer that fails validation as a serious security event rather than "just another SERVFAIL."
