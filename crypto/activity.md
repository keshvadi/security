# Topic: Public-Key (Asymmetric) Encryption

*   **1) Overview → Why public‑key + hybrid**
*   **2) Trapdoor one‑way functions (RSA hardness, discrete log)**
*   **3) Public key distribution risks (active attacker)**
*   **4) Session keys & key separation (hybrid encryption workflow)**


## Topic learning outcomes (derived from your draft)

By the end of this topic, students should be able to:

1.  Explain how public‑key encryption addresses the **key distribution** problem and why we still use symmetric keys for bulk data. 
2.  Describe a **trapdoor one‑way function** and relate RSA/Diffie‑Hellman hardness assumptions to encryption workflows (at a conceptual level). 
3.  Identify risks of **unauthenticated key distribution** (Mallory substituting a key) and articulate the need for integrity of public keys (bridge to signatures/PKI next). 
4.  Explain **hybrid/session‑key** design and why public‑key schemes are slow and typically randomized (e.g., OAEP for IND‑CPA security) compared to symmetric ciphers. 

### A1. Reading + Quick Check — “Why Public‑Key at All? (The Key‑Management Problem)”

*   **Maps to**: §1 Overview. 
*   **Format**: Short reading with 4 MCQs (automark).
*   **Focus**: Contrast symmetric vs public‑key; “publish PK, keep SK secret”; encrypting a *symmetric* K for bulk comms. 
*   **Production**: H5P Question Set; include a small diagram image holder.

### A2. Concept Builder — Trapdoor One‑Way Functions (Match & Explain)

*   **Maps to**: §2 Trapdoor one‑way functions (RSA, discrete log). 
*   **Format**: Matching + 2 short explanations (auto for matching; show/hide for sample explanations).
*   **Focus**: “Easy forward f(x), hard inverse—unless you have the trapdoor”; connect RSA factoring notion and DH hardness intuition (no heavy math). 
*   **Production**: Accordion for definitions; Matching widget.

**Goal:** Solidify the difference between a standard one-way function (Unit 1) and a trapdoor one-way function.
**Format:** A "Scenario vs. Property" matching activity.
**Context:** Comparing a "Paper Shredder" (one-way) vs. a "Locked Safe" (trapdoor).

### **Activity 2: The Toy RSA Math (Manual Calculation)**

* 
**Goal:** Provide "Aha!" moment for how a private key "unlocks" a calculation.


* **Format:** Small-number RSA exercise.
* **Task:** Students are given $n=33$ ($p=3, q=11$) and $e=3$. They will encrypt a small digit and then see how knowing the factors ($p$ and $q$) allows for decryption.


### A5. The "Mallory-in-the-Middle" Key Swap (Diagram Walkthrough)**

* 
**Goal:** Illustrate why public key distribution is vulnerable to spoofing without integrity.


* **Format:** Interactive diagram analysis.
* 
**Context:** Alice wants Bob's key, but Mallory intercepts the request and provides his own. Students must identify at which point the **confidentiality** is lost because **integrity** was missing.

### A5-b. Interactive Scenario — **Mallory’s Key Substitution**  (Integrity of Public Keys) | **Public Key Distribution** Under Active Attack

*   **Maps to**: §3 Public Key Distribution. 
*   **Format**: Branching scenario (or carousel) where learners choose how Alice obtains Bob’s key; outcomes show when Mallory can read messages.
*   **Learning point**: Confidentiality of the *public* key is not required, but **integrity** is; foreshadows Digital Signatures/PKI next topic. 
*   **Production**: Storyline/H5P Course Presentation; include alt text.

#### 4. Activity: Thinking Like Mallory — The Public Key Spoof

**Goal:** Highlight the "Public Key Distribution" problem and the risk of active attacks.

* **Format:** Short Answer / Discussion.
* **Scenario:** Bob broadcasts his public key over an insecure channel. Mallory intercepts it and replaces it with his own.


* 
**Question:** "If Alice uses the key she *thinks* belongs to Bob, who can read her messages? What security property (CIA Triad) was violated during the key distribution phase?".


### A6. Worked Walkthrough — **Hybrid (KEM/DEM) Pattern** with Session Keys

*   **Maps to**: §4 Session Keys (multiple keys for enc/MAC and per‑direction). 
*   **Format**: Diagram + step list: generate random AES key(s), encrypt data with AES‑GCM (or AES‑CBC + HMAC to match your note), wrap keys with RSA‑OAEP.
*   **Quick Check**: 4 MCQs on “why several session keys,” “why symmetric for bulk,” and “order of operations.” 
*   **Production**: Diagram holders; automarkable MCQ.

### A8. Short Discussion (Ungraded) — “How do you trust a public key online?”

*   **Maps to**: §3. 
*   **Prompt**: Evaluate options: in‑person exchange, fingerprint on business card, pinned keys, organization directory, or “download from a website.” Contrast integrity guarantees and usability; bridge to **signatures/certificates** next. 
*   **Production**: Add your standard “online community” note from Unit 1. 

### A3-a. Micro‑Lab (Ubuntu‑in‑Docker) — **Toy RSA Encrypt/Decrypt** with OpenSSL

*   **Maps to**: §2 / §4 (practical feel for asymmetry & performance). 
*   **Format**: 15–20 min guided shell steps: generate RSA keypair, encrypt a tiny *random* file with **RSA‑OAEP**, decrypt, verify.
*   **Checks**: 3 inline MCQs (key sizes, message length limits, why OAEP).
*   **Production**: Command blocks; automark on specific outputs; notes that RSA is for *wrapping a key*, not big files. 
### A3-b. Mini‑Timing Demo — Symmetric vs Public‑Key (Feel the Cost)

*   **Maps to**: §4 (public‑key is slow). 
*   **Format**: Learners time `openssl enc` (AES‑GCM) on a 50–100MB file vs an RSA‑OAEP *key wrap* and reflect (why we don’t RSA‑encrypt large payloads).
*   **Quick Check**: 3 MCQs (performance & design takeaway). 
*   **Production**: Shell snippets; reflection box.
### Activity 3-c (Hands‑On Lab): **RSA‑OAEP in Practice** + Why We Don’t Encrypt Big Files with RSA

**Alignment:** §1 Overview, §4 Session Keys.   
**Goal:** Generate RSA keys; encrypt/decrypt a **small** secret with OAEP; observe size constraint; set up for hybrid in Activity 6. 

**Environment:** Ubuntu‑in‑Docker (same as Unit 1). 

**Steps**

1.  **Generate keypair (2048‑bit)**
    ```bash
    openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out rsa_priv.pem
    openssl pkey -in rsa_priv.pem -pubout -out rsa_pub.pem
    ```
2.  **Create a small secret to simulate a session key (32 bytes)**
    ```bash
    openssl rand 32 > session.key
    ```
3.  **Encrypt the small secret with **OAEP****
    ```bash
    openssl pkeyutl -encrypt -inkey rsa_pub.pem -pubin -in session.key -out session.key.rsa \
      -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
    ```
4.  **Decrypt with private key**
    ```bash
    openssl pkeyutl -decrypt -inkey rsa_priv.pem -in session.key.rsa -out session.key.dec \
      -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
    diff session.key session.key.dec && echo "Match"
    ```
5.  **(Demonstration)** Try encrypting a **large** file directly with RSA and observe the error (or learn to chunk—don’t do this in practice). This motivates hybrid. 

**Auto‑markable prompts**

*   Paste the first 10 hex bytes of `session.key` and `session.key.dec` (should match).
*   Confirm the modulus size of your keypair (use `openssl pkey -in rsa_priv.pem -text -noout | grep 'modulus' -n` and record bits).
*   Question: Why does RSA‑OAEP encryption fail for large inputs? (size bound ≈ key\_size − padding overhead). 

**Production**

*   Plain text, CLI copy‑paste friendly. Add a short note that OAEP (with SHA‑256/MGF1) is the modern padding to avoid deterministic/plain RSA issues (you preview this without going deep into padding‑oracle history yet). 

### Activity 3-d (Hands‑On Lab): Build a **Minimal Hybrid** (RSA‑OAEP + AES‑CTR)

**Alignment:** §4 Session Keys + your earlier CTR/nonce guidance.   
**Goal:** Use RSA only to wrap a randomly generated symmetric key; use that key to encrypt a file with AES‑CTR; handle nonce/IV correctly. , 

**Steps**

1.  **Inputs**
    ```bash
    echo "Hybrid demo file" > message.txt
    openssl rand 32 > aes.key               # 256-bit symmetric key
    openssl rand 16 > aes.iv                # 128-bit IV/nonce for CTR
    ```
2.  **Encrypt message with AES‑256‑CTR (raw key/IV in hex)**
    ```bash
    KEYHEX=$(xxd -p -c 256 aes.key)
    IVHEX=$(xxd -p -c 256 aes.iv)
    openssl enc -aes-256-ctr -K "$KEYHEX" -iv "$IVHEX" -in message.txt -out message.enc
    ```
3.  **Wrap the AES key with RSA‑OAEP**
    ```bash
    openssl pkeyutl -encrypt -inkey rsa_pub.pem -pubin -in aes.key -out aes.key.rsa \
      -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
    ```
4.  **Transmit** `{aes.key.rsa, aes.iv, message.enc}` (note IV is **not secret**). 
5.  **Receiver** unwraps key and decrypts
    ```bash
    openssl pkeyutl -decrypt -inkey rsa_priv.pem -in aes.key.rsa -out aes.key.dec \
      -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -pkeyopt rsa_mgf1_md:sha256
    KEYHEX=$(xxd -p -c 256 aes.key.dec)
    IVHEX=$(xxd -p -c 256 aes.iv)
    openssl enc -d -aes-256-ctr -K "$KEYHEX" -iv "$IVHEX" -in message.enc -out message.dec.txt
    diff message.txt message.dec.txt && echo "Recovered"
    ```

**Quick Check (auto‑marked)**

*   True/False: It’s safe to reuse the same IV for CTR with the same key. **False.** 
*   Identify which of `{aes.key.rsa, aes.iv, message.enc}` must remain secret in transit. **Only the unwrapped AES key; the wrapped key is safe to send; IV can be public.** 

**Production**

*   Keep to CTR (students already saw CTR pitfalls); you can later connect this to **AEAD** (AES‑GCM) so integrity is built‑in. 
### **Activity 3-e: Hybrid Encryption Lab (OpenSSL/CLI)**

* 
**Goal:** Demonstrate the "Public Key is Slow" problem and the "Session Key" solution.


* **Format:** Hands-on lab in the Ubuntu-in-Docker environment.
* **Task:**
1. Generate a 2048-bit RSA key pair.
2. Attempt to encrypt a large file using *only* RSA (and observe the error/slowness).
3. Generate a random 128-bit AES key (the session key), encrypt the file with AES, and then "wrap" (encrypt) that AES key with the RSA public key.


### **Activity 5: Threat Sorting — Session Key Management**

* 
**Goal:** Understand why we use multiple keys (Enc vs. MAC, Alice-to-Bob vs. Bob-to-Alice).


* **Format:** MCQ/Ordering.
* 
**Context:** Designing a secure chat protocol based on the "Session Keys" section of the text.


### A9. Quick Quiz — Public‑Key vs Signatures vs Symmetric (Terminology Sweep) RSA, DH, and AES

*   **Maps to**: All sections, prepares for next topic (Digital Signatures/PKI).
*   **Format**: 8 item mixed MCQ/matching: who holds which key, what provides what property, where integrity of public keys matters, why OAEP. 
### (Concept + Matching): **Trapdoor One‑Way Functions**

**Alignment:** §2 Trapdoor one‑way functions.  
**Goal:** Distinguish one‑way vs. trapdoor one‑way; map RSA/DLP to the abstraction. 

**Instructions**

*   Read the accordion with three panels: (1) One‑way vs. trapdoor, (2) RSA hardness intuition, (3) Discrete‑log intuition. 
*   Complete a 6‑item matching.

**Matching (examples)**

*   “Easy forward, hard inverse; **with** secret backdoor inverse is easy” → **Trapdoor one‑way**. **(Answer)** 
*   “Given $$c=m^e \bmod n$$, hard to find $$m$$ without factoring $$n$$” → **RSA hardness**. **(Answer)** 
*   “Given $$g^a, g^b$$, hard to compute $$g^{ab}$$” → **Discrete log / DH basis**. **(Answer)** 

**Production**

*   Build as **matching** (auto‑marked). Keep your voice and examples.
### A9-b (Optional, “Safe or Unsafe?”): Common Pitfalls Bingo

**Goal:** Rapid identification of red flags.  
**Items (toggle grid)**

*   “Textbook RSA without OAEP because it’s simpler.” → **Unsafe**. 
*   “Encrypt entire database directly with RSA‑2048.” → **Unsafe** (use hybrid). 
*   “Post a bare public key to a forum; tell users to ‘trust it’.” → **Unsafe** (no integrity/authentication). 
*   “Reuse CTR IVs for convenience.” → **Unsafe** (keystream reuse). 
#### A9-c. Activity: Ordering the "Hybrid" Handshake (Session Key Workflow)

**Goal:** Understand why and how we combine public-key and symmetric-key systems in practice.

* **Format:** Drag-and-drop ordering.
* **Task:** Put the steps of a secure session establishment in order:
1. Alice generates a random session key $K$.


2. Alice encrypts $K$ using Bob’s Public Key.


3. Bob decrypts the package using his Private Key to recover $K$.


4. Both parties use $K$ and a symmetric algorithm (like AES) for the rest of the conversation.

************************************************************************************************
# Topic: Digital Signatures

## Topic learning outcomes (derived from your draft)

By the end of this topic, students should be able to:

1.  **Explain what a digital signature is** (who can sign/verify; purpose: integrity + authentication) and the three algorithms **KeyGen / Sign / Verify**. 
2.  **Describe the RSA‑style trapdoor idea** for signatures at a high level: hash the message, apply trapdoor inversion with the private key, and verify using the public key. (Conceptual, math‑light.) 
3.  **State a modern security goal for signatures** (unforgeability under chosen‑message attack—“signing oracle” game) and reason about what the adversary can and cannot do. 
4.  **Connect signatures to hashing** (why collision resistance matters; how weak hashes can undermine signatures), and understand the bridge to **key distribution/PKI** from the previous topic. , 

### S1. Reading + Quick Check — “What Signatures Guarantee (and What They Don’t)”

*   **Maps to**: §1, “Digital signature properties” (who can sign vs who can verify; integrity + authenticity).
*   **Format**: Short reading + 5 MCQs (auto‑marked).
*   **Focus**: Reverse of encryption: **only** the private key holder can sign; **anyone** with the public key can verify; signatures ≠ confidentiality. 
*   **Production**: H5P Question Set; include a small icon/diagram holder showing Sign/Verify arrows.

### S1b. Matching — KeyGen / Sign / Verify (Inputs & Outputs)

*   **Maps to**: §1 algorithms.
*   **Format**: Drag‑match “Inputs → Algorithm → Output” cards (e.g., `(SK, M) → Sign → S`).
*   **Goal**: Cement mental model of the three algorithms and who holds which key. 
*   **Production**: H5P Drag‑and‑Drop; include alt text.

### Activity 1c (Reading + Quick Check): What Signatures Guarantee (and How They Differ from MACs)

**Alignment:** *signatures.md* §1 (“Digital signature properties”).  
**Goal:** Students internalize that signatures provide **integrity + authenticity + public verifiability** (anyone can verify), unlike MACs (shared‑key). 

**Instructions**

1.  Read a short LMS page contrasting **MAC vs. signature** and defining: **KeyGen, Sign, Verify**, public verification, and non‑repudiation (limited, system‑dependent). 
2.  Complete a 4‑item MCQ.

**Quick Check (auto‑marked)**

*   Q1: Who can **verify** a digital signature? → *Anyone who knows the public key.* **Answer**. 
*   Q2: Which guarantees do signatures provide that MACs do not by themselves? → *Public verifiability (and potential non‑repudiation).* **Answer**. 
*   Q3: What 3 algorithms form a signature scheme? → *KeyGen, Sign, Verify.* **Answer**. 
*   Q4 (T/F): If everyone has the public key, then everyone can **generate** valid signatures. → *False.* **Answer**. 

**Production:** Simple MCQ block; glossary popovers for *public verifiability*, *non‑repudiation*.

### Activity 1d: The "Who Can Do What?" Matrix (Conceptual)

* 
**Goal:** Differentiate between MACs, Public-Key Encryption, and Digital Signatures based on who holds which keys.


* **Format:** A 3x3 table matching "Operation" (Sign, Verify, Encrypt, Decrypt) to "Key Required" (Public, Private, or Shared Secret).
* 
**Context:** Reinforces that for signatures, the **private key** is for signing and the **public key** is for verification—the inverse of encryption.

### A3-0: Toy RSA Signatures (Manual Calculation)

**Goal:** Use the provided number theory facts to sign and verify a "toy" message.

**Format:** Step-by-step arithmetic exercise using small primes ($p=2, q=7$).

* **Task:** Students will:
1. Calculate $n = pq$.
2. Find $d$ given $e=3$ (using Fact 3).
3. "Sign" a small hash value $y$ by computing $S = y^d \pmod n$.
4. Verify the signature by computing $S^3 \pmod n$ and checking if it matches the original $y$.

#### A3-1. Activity: RSA Signing "Under the Hood" (The Math Walkthrough)

**Goal:** Visualize how a signature is actually computed using the hash and the trapdoor.

* **Format:** A "Click-to-Reveal" or "Step-by-Step" diagram walkthrough.
* **Steps to show:**
1. Message $M$ is hashed to $H(M)$.

2. The Signer applies the private key $d$ to the hash: $S = H(M)^d \pmod n$.
3. The Verifier takes $S$ and the public key $n$, computing $S^3 \pmod n$.
4. The final check: Does $S^3 \pmod n = H(M)$?.

### S3. Micro‑Lab (Ubuntu‑in‑Docker) — **Sign & Verify with OpenSSL**

*   **Maps to**: §1 end‑to‑end properties; connects to your existing lab conventions.
*   **What learners do (≈20–25 min):**
    1.  Generate an RSA keypair; export public key.
    2.  Sign a small text file (hash‑then‑sign via OpenSSL); verify success.
    3.  Tweak one byte; observe verification failure.
    4.  Try verifying with a **non‑matching** public key; observe failure.
    5.  Answer 4 inline MCQs on “who can sign/verify” and “why a hash is part of signing.”
*   **Why here**: It gives the *feel* of authenticity + integrity and underscores that verification fails if the content or key is wrong. (Math‑light; operationally faithful.) , 
*   **Production**: Shell blocks + automarked answers; notes that learners should use reputable libraries—not rolling their own—from your draft’s caution. 

### S3b. (Hands‑On Lab): **RSA‑PSS** Sign & Verify (OpenSSL)

**Alignment:** *signatures.md* §2–4 (hash‑then‑sign with a trapdoor).  
**Goal:** Use RSA **safely** with PSS and SHA‑256; observe that PSS is **probabilistic** (different signatures on same message). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8017)

**Environment:** Ubuntu‑in‑Docker (same toolchain as Unit 1). 

**Steps**

```bash
# 1) Generate RSA keypair
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out rsa_priv.pem
openssl pkey -in rsa_priv.pem -pubout -out rsa_pub.pem

# 2) Prepare a message
echo "I love hybrid crypto" > msg.txt

# 3) Sign with RSA-PSS + SHA-256
openssl dgst -sha256 -sigopt rsa_padding_mode:pss -sigopt rsa_pss_saltlen:-1 \
  -sign rsa_priv.pem -out msg.sig msg.txt

# 4) Verify
openssl dgst -sha256 -sigopt rsa_padding_mode:pss -sigopt rsa_pss_saltlen:-1 \
  -verify rsa_pub.pem -signature msg.sig msg.txt

# 5) Sign again (should produce a different signature due to randomized salt)
openssl dgst -sha256 -sigopt rsa_padding_mode:pss -sigopt rsa_pss_saltlen:-1 \
  -sign rsa_priv.pem -out msg2.sig msg.txt
cmp msg.sig msg2.sig || echo "Different (expected)"
```

**Auto‑mark prompts**

*   Paste the output of Step 4 (**Verified OK**).
*   Are `msg.sig` and `msg2.sig` identical? → *No (PSS is randomized).* **Answer**. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8017), [\[en.wikipedia.org\]](https://en.wikipedia.org/wiki/Probabilistic_signature_scheme)

**Production:** Plain CLI; note that **PSS** parameters (hash/MGF1/saltlen) must match on verify. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8017)

### S3c. (Hands‑On Lab): **Ed25519 (EdDSA)** Sign & Verify (OpenSSL)

**Alignment:** Modern algorithms in **FIPS 186‑5** (EdDSA), contrasts with RSA‑PSS; connects to your “trapdoor” mental model via discrete log hardness. [\[nvlpubs.nist.gov\]](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-5.pdf), [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8032)

**Why Ed25519?** Standardized in **RFC 8032**; small keys/signatures, fast, deterministic nonces (robust for constrained devices). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8032), [\[cryptobook.nakov.com\]](https://cryptobook.nakov.com/digital-signatures/eddsa-and-ed25519)

**Steps**

```bash
# 1) Generate Ed25519 keypair
openssl genpkey -algorithm ed25519 -out ed25519_priv.pem
openssl pkey -in ed25519_priv.pem -pubout -out ed25519_pub.pem

# 2) Sign (deterministic for same key+message)
openssl pkeyutl -sign -inkey ed25519_priv.pem -in msg.txt -out msg.ed.sig

# 3) Verify
openssl pkeyutl -verify -pubin -inkey ed25519_pub.pem \
  -in msg.txt -sigfile msg.ed.sig

# 4) Re‑sign and compare (expected: identical bytes for same msg/key with Ed25519)
openssl pkeyutl -sign -inkey ed25519_priv.pem -in msg.txt -out msg2.ed.sig
cmp msg.ed.sig msg2.ed.sig && echo "Identical (expected)"
```

**Quick Check**

*   Are two Ed25519 signatures over the *same* message with the *same* key **identical**? → *Yes (deterministic design).* **Answer**. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8032)
*   Name one advantage of Ed25519 vs. ECDSA P‑256 in practice. → *Smaller/faster and deterministic nonce generation; good for IoT.* **Answer**. [\[cryptobook.nakov.com\]](https://cryptobook.nakov.com/digital-signatures/eddsa-and-ed25519), [\[eprint.iacr.org\]](https://eprint.iacr.org/2021/471.pdf)

**Production:** Short note on **Ed25519/Ed448** being included in **FIPS 186‑5** as EdDSA. [\[nvlpubs.nist.gov\]](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-5.pdf)

### Activity 3e: Lab – Signing and Verifying with OpenSSL (CLI)

* **Goal:** Practical application of RSA signatures in the Linux environment.
* **Format:** Hands-on lab using the Ubuntu-in-Docker container.
* **Task:**
1. Generate an RSA private key.
2. Extract the public key.
3. Create a text file and sign it using `openssl dgst -sha256 -sign`.


4. Verify the signature with `openssl dgst -sha256 -verify` and observe what happens if the text file is modified by even a single character (the avalanche effect in the hash function).




### S4. Scenario Carousel — **Unforgeability Game (Chosen‑Message Attack)**

*   **Maps to**: §5 security definition.
*   **Format**: A short “game” where the learner plays Georgia (adversary) asking a “signing oracle” for signatures on chosen messages, then tries to output a new `(M, S)` pair not previously signed.
*   **Quick Check**: 5 MCQs about why seeing many valid signatures *still* shouldn’t let you forge a signature on a *new* message; what “wins” the game. 
*   **Production**: H5P Course Presentation/Branching (simple states), with Answer Key.

### S5. Reading + Quick Check — **Why Hash Choice Matters for Signatures**

*   **Maps to**: §5 (“Signatures have been broken… when the hash function… is compromised”).
*   **Format**: Short reading + 4 MCQs.
*   **Focus**: If a signature scheme signs `H(M)`, then **collision resistance** is critical; weak hashes can enable practical attacks. Link this concept back to your prior hashing topic (SHA‑1 collision demo and the **Flame** case in Unit 1). , 
*   **Production**: H5P MCQs; optional “click to reveal” recap of your earlier SHA‑1 collision activity reference.

### S6. Hands‑On Mini‑Demo — “Hash Swap Thought Experiment”

*   **Maps to**: §5 dependency on `H(·)`.
*   **Format**: Learners compute SHA‑256 of a file, sign it, then **re‑hash** and verify; they see that verification binds the **exact** message (not just the filename).
*   **Quick Check**: Why an admitted SHA‑1 collision (hypothetical for the course) would be dangerous if a system still used SHA‑1 for signing.
*   **Production**: Shell snippets; automarkable answers; keep strictly conceptual—no real forgery required—aligning with your prior caution to use reputable libraries. 

### Activity 7 (Diagram Walkthrough + Matching): Hash‑then‑Sign Encodings — **RSA‑PSS vs RSASSA‑PKCS1‑v1\_5**

**Alignment:** §2–4; modern recommendations.  
**Goal:** Students can articulate **why PSS is preferred** for new deployments and how **parameters** (hash, MGF1, salt length) bind verify behavior. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8017), [\[quarkslab.github.io\]](https://quarkslab.github.io/crypto-condor/latest/method/RSASSA.html)

**Instructions**

*   View a two‑panel diagram: PSS (randomized salt) vs PKCS1‑v1\_5 (deterministic).
*   Matching: algorithm ↔ properties (probabilistic/deterministic; parameterization; robustness).
*   One MCQ: *Which scheme should you choose for new apps?* → **PSS**. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc8017), [\[quarkslab.github.io\]](https://quarkslab.github.io/crypto-condor/latest/method/RSASSA.html)


### S7. Misuse Clinic — 6 Short Scenarios (Signatures Done Wrong)

*   **Maps to**: §1/§5 principles.
*   **Format**: MCQ/matching; pick the *primary* failure.
*   **Examples**:
    *   Using **the public key** to “sign.” (Impossible; public verifies only.) 
    *   Publishing only a **hash** of the file and not a signature (attacker can replace both file and hash). (Connect to your Unit‑1 hash‑integrity discussion.) 
    *   Verifying signatures **with the wrong key** (key distribution/PKI problem from the prior topic). 
*   **Production**: H5P Question Set.

### S8. Bridge Discussion — **From Signatures to PKI**

*   **Maps to**: Prior “Public‑Key Distribution” topic + this unit’s “anyone can verify” property.
*   **Prompt**: “How do signatures help me trust a downloaded public key? When would I rely on a certificate vs an out‑of‑band fingerprint?” (Preps learners for your next PKI content.) 
*   **Production**: Use your **ungraded community** wording pattern from Unit‑1 discussions. 

### S9. Short Quiz — Terminology & Properties Sweep

*   **Maps to**: Entire topic.
*   **Format**: 8–10 items: who holds which key, what Verify checks, what it means to “win” the forgery game, why the hash must be collision‑resistant. 

#### S9-b. Activity: Digital Signatures vs. MACs (The Comparison Lab)

**Goal:** Solidify the "Asymmetric vs. Symmetric" distinction for integrity.

* **Format:** Comparative Table / "Which Tool for the Job?"
* **Scenarios:**
1. 
*Scenario A:* A software vendor wants to publish a single update that 1 million users can verify independently. (Best tool: **Digital Signature**).


2. *Scenario B:* Two routers on a private link need to quickly verify every packet they send to each other. (Best tool: **MAC**).

************************************************************************************************************************************************
# Topic: Certificates

MITM setup → Trusted Directory → Digital Certificates → PKI → Chains → Revocation → Web‑of‑Trust → SSH TOFU

## Topic learning outcomes (derived from your draft)

By the end of this topic, students should be able to:

1.  Explain why **MITM** persists when public keys lack authenticity and why any solution must ensure **integrity of public keys**. 
2.  Describe how **digital certificates** bind identities to public keys and how applications verify them **without contacting a live directory** (self‑validating when you trust the signer). 
3.  Outline how **PKI and certificate chains** scale trust (root → intermediates → end‑entity) and the risk that **any trusted CA** can affect everyone. 
4.  Compare **revocation strategies** (validity periods vs revocation lists) and their trade‑offs. 
5.  Contrast **Web‑of‑Trust** and **TOFU/SSH** with hierarchical PKI, noting non‑transitivity of trust and usability trade‑offs. 

### C1. Reading + Quick Check — **Why MITM Still Works Without Key Authenticity**

*   **Maps to**: §13.1 Man‑in‑the‑Middle Attacks.
*   **Focus**: Show that swapping a public key lets Mallory read/alter traffic *without* breaking crypto; explain why Bob may not notice.
*   **Format**: Short reading + 4 MCQs (automark).
*   **Production**: H5P Question Set + one diagram holder (Alice—Mallory—Bob). 

### Activity 1-b (Concept + Quick Check): Why MITM Persists Without Key Authentication

**Alignment:** §13.1 *Man‑in‑the‑middle Attacks*  
**Goal:** Make students fluent with *active attacker* vs *passive eavesdropper* and how MITM is defeated only if public keys are authenticated.

**Instructions**

1.  Read the short scenario (Alice, Bob, Mallory) mirroring your text.
2.  Complete a 5‑item MCQ/Matching set:
    *   *Which steps Mallory modifies?*
    *   *What property is missing?* (authenticity of Bob’s public key)
    *   *Which defenses address it?* (certificates; authenticated key directories; TOFU)
3.  **Answer Key**: provided.

**Production:** Simple MCQ block. (No external dependencies.)

***

### C1-c. Concept Builder — **Trusted Directory vs Certificates**

*   **Maps to**: §13.2 Trusted Directory Service, §13.3 Digital Certificates.
*   **Focus**: Compare properties and limitations (trust, scalability, reliability, online requirement) of a live directory vs offline **self‑validating** certificates (verify signature if you trust the issuer’s public key).
*   **Format**: Matching + 2 short justifications (show/hide).
*   **Production**: Matching widget; brief accordion recap. 

### Activity 1-d (Reading + Quick Check): Trusted Directory vs. Digital Certificates

**Alignment:** §13.2 *Trusted Directory Service* → §13.3 *Digital Certificates*  
**Goal:** Contrast a *live* directory with *offline, verifiable* certificates; emphasize that a cert is “self‑verifying” **if** you already trust the signer’s public key.

**Instructions**

1.  Read: 2‑panel explainer
    *   Panel A—“Trusted directory”: bottlenecks, reliability, availability.
    *   Panel B—“Digital certificates (X.509)”: signed bindings of *name ↔ public key*; verification by a known CA key.
2.  Quick Check (MCQ):
    *   Why is it safe to download a certificate over an insecure channel? → *Because we verify the CA signature.*
    *   What must be pre‑trusted? → *The CA’s public key.*

**Production:** Two slides + 4 MCQs.  
**Note (Standards box):** Certificates used on the Internet follow the **X.509** profile defined in **RFC 5280** (format, extensions, path validation, CRLs). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc5280)

***

### Activity 1-e (Diagram Walkthrough): **PKI at Scale** — CAs, Chains, and Path Validation

**Alignment:** §13.4 *PKI* and §13.5 *Certificate Chains & Hierarchical PKI*  
**Goal:** Read a chain, identify **root → intermediate → leaf**, and explain how a client validates the path and names.

**Instructions**

1.  View a three‑panel diagram:
    *   Panel 1: **Root CA** (self‑signed), **intermediate CA**, **server cert**.
    *   Panel 2: Path validation steps (issuer/subject linkage, signature checks, validity periods, key usage/EKUs, name constraints).
    *   Panel 3: “Any‑CA‑for‑any‑domain” risk & why **browser root programs** are strict.
2.  Quick Check (Matching):
    *   *Basic Constraints, Key Usage, EKU, Name Constraints* → *purpose in path validation* (from **RFC 5280**). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc5280)
3.  Reflection prompt (1–2 sentences): “Why is a single mis‑issuance by any trusted CA dangerous in the WebPKI?”

**Production notes**

*   Add a small “Root Programs” info box with links to **CA/Browser Forum Baseline Requirements** and **Mozilla Root Store Policy v3.0**. Emphasize that client platforms enforce additional policies on top of RFC 5280. [\[cabforum.org\]](https://cabforum.org/working-groups/server/baseline-requirements/requirements/), [\[mozilla.org\]](https://www.mozilla.org/en-US/about/governance/policies/security-group/certs/policy/)

***


### C3. Micro‑Lab (Ubuntu‑in‑Docker) — **Inspect a Real Site’s Certificate Chain**

*   **Maps to**: §13.4 PKI, §13.5 Certificate Chains.
*   **What learners do (≈20–25 min)**:
    1.  Use `openssl s_client -connect example.com:443 -showcerts` to view **end‑entity + chain**; extract subjects/issuers.
    2.  Identify **root vs intermediate** and note that trust anchors are in the OS/browser store.
    3.  Check **validity period** (notBefore/notAfter).
    4.  Answer 4 MCQs: chain purpose, path building idea, why *any* trusted CA can issue for any domain (risk model), and why chains reduce the load on a single “Jerry‑like” signer.
*   **Production**: Command blocks; automark on concept questions (not on dynamic outputs). 

### C3-b. Mini‑Lab (Optional) — **Peek Into Trust Stores**

*   **Maps to**: §13.4 (pre‑trusted CAs).
*   **What learners do (≈10–15 min)**:
    *   On Linux container: examine `/etc/ssl/certs` or `update-ca-certificates` outputs; count anchors.
    *   Reflection: What does it mean that you likely don’t recognize many CA names?
*   **Production**: Shell snippets + 2 reflection prompts (ungraded). 

### C3-c. Walkthrough — **Path Validation, Step‑by‑Step**

*   **Maps to**: §13.5 Certificate Chains and Hierarchical PKI.
*   **Focus**: Order the essential checks conceptually: (a) chain links (issuer→subject), (b) signatures verify, (c) time validity, (d) name matches site, (e) trust anchor present.
*   **Format**: Ordering + 3 MCQs.
*   **Production**: H5P; concise diagram showing chain links. 

### Activity 3-d (Hands‑On Lab, offline): **Build a Mini‑PKI** with OpenSSL—Root, Intermediate, Server, Chain

**Alignment:** §13.4–13.5; prepares students for protocol labs later.  
**Goal:** Create a lab‑local root & intermediate, issue a server cert, verify the chain and **extensions** (BC, KU, EKU), then break & fix validation.

**Environment:** Ubuntu‑in‑Docker (same as Unit 1).  
**Steps (abridged)**

```bash
# 1) Create Root CA key/cert (self-signed) with CA:TRUE
openssl genrsa -out root.key 4096
openssl req -x509 -new -key root.key -days 3650 -sha256 \
  -subj "/CN=Mini-Root" -out root.crt -extensions v3_ca \
  -config <(printf "[req]\ndistinguished_name=dn\n[dn]\n[v3_ca]\nbasicConstraints=critical,CA:TRUE\nkeyUsage=critical,keyCertSign,cRLSign\n")

# 2) Create Intermediate CA signed by Root (CA:TRUE; pathLen=0)
# 3) Create server CSR and issue leaf cert (CA:FALSE; EKU=serverAuth)
# 4) Build chain: cat server.crt inter.crt > chain.pem
# 5) Verify:
openssl verify -CAfile root.crt -untrusted inter.crt server.crt
```

**Auto‑mark prompts**

*   Paste the output of `openssl verify` (**OK** expected).
*   Show `openssl x509 -in server.crt -text` and answer: *Is Basic Constraints CA:FALSE?* **Yes.** (Short free‑text).

**Standards note:** Field meanings follow **RFC 5280** (BasicConstraints, KeyUsage, EKU, Name constraints). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc5280)

***

### Activity 3-e (Hands‑On Lab, offline): **Revocation in Practice** — Create a CRL, Revoke, and Re‑verify

**Alignment:** §13.6 *Revocation* (CRLs), trade‑offs.  
**Goal:** Experience the operational side: publish a **CRL**, verify rejection; discuss **OCSP** conceptually and **OCSP stapling**.

**Steps (abridged)**

```bash
# 1) Revoke the server cert at the Intermediate CA
# (create index.txt/serial files as per OpenSSL CA layout)
openssl ca -config inter.cnf -revoke server.crt -crl_reason keyCompromise

# 2) Issue a CRL from the Intermediate CA
openssl ca -config inter.cnf -gencrl -out inter.crl

# 3) Verify server cert against chain + CRL
openssl verify -CAfile root.crt -untrusted inter.crt \
  -CRLfile inter.crl -crl_check server.crt
```

**Expected:** verification **fails** due to revocation.

**Concept add‑on (reading snippet):**

*   **CRLs** (revocation lists) and **OCSP** (per‑serial status protocol) are defined for WebPKI; see **RFC 5280** (CRLs) and **RFC 6960** (OCSP). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc5280), [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc6960)
*   **OCSP stapling** delivers a recent, CA‑signed OCSP response in the TLS handshake (TLS *status\_request* / *status\_request\_v2*). [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc6066), [\[datatracker.ietf.org\]](https://datatracker.ietf.org/doc/html/rfc6961)

**Current‑practice box (read‑only):**

*   The WebPKI is moving **away from client OCSP checks** toward CRLs and browser‑side mechanisms (e.g., CRLite), and some major CAs (e.g., Let’s Encrypt/ISRG) **turned off OCSP** responders in 2025 for privacy/operational reasons. Learners won’t configure this now, but they should know the direction of travel. [\[letsencrypt.org\]](https://letsencrypt.org/2024/12/05/ending-ocsp), [\[abetterinternet.org\]](https://www.abetterinternet.org/post/ending-ocsp/)

**Answer Key:** rationale questions (CRL vs OCSP; “soft‑fail” risks for OCSP).


### Activity 3-f: Decoding a Real Certificate (CLI Lab)

* **Goal:** Understand the structure of a digital certificate (Identity + Public Key + Signature).
* **Format:** Hands-on lab using the Ubuntu-in-Docker container.
* **Task:** Students use `openssl x509 -in cert.pem -text -noout` to inspect a provided certificate.
* **Identify the "Subject":** Who is this for?
* **Identify the "Issuer":** Who is the CA?
* **Check Validity:** Is it expired?
* **Find the Signature:** Observe how the CA's signature binds the identity to the key.




### C4. Scenario Carousel — **“Any CA Can Break Me” (RISKS of Many Trust Anchors)**

*   **Maps to**: §13.4 PKI (browsers accept any trusted CA; unfamiliar CA names; risk amplification).
*   **Focus**: Interactive vignettes: if **any** of the pre‑trusted CAs mis‑issues a cert, users are exposed; more trusted CAs ⇒ larger attack surface.
*   **Format**: Carousel with short decisions + feedback.
*   **Production**: Storyline/H5P Course Presentation; include an accessibility‑described figure of a “CA trust store” and a “lock icon” caveat. 

### Activity 4-b (Case Study Discussion): **Any‑CA Risk & Browser Root Programs**

**Alignment:** §13.4–13.5  
**Goal:** Discuss system‑level governance: **Baseline Requirements (CABF)**, **root store policies**, and why automation + short lifetimes are emphasized.

**Seed prompts**

*   “Any trusted root can issue for any domain”—what mitigations exist? (audits, CT, BRs, root program enforcement, short lifetimes). [\[cabforum.org\]](https://cabforum.org/working-groups/server/baseline-requirements/requirements/), [\[mozilla.org\]](https://www.mozilla.org/en-US/about/governance/policies/security-group/certs/policy/)
*   Why are certificate **validity periods shrinking** (policy trend toward \~200 → 100 → lower days over coming years)? (reduces blast radius; makes revocation less central). [\[postquantum.com\]](https://postquantum.com/security-pqc/sc081v3-47day-certificate/)
*   What does automation buy us? (Chrome’s push and ecosystem guidance). [\[blog.chromium.org\]](https://blog.chromium.org/2023/10/unlocking-power-of-tls-certificate.html)

**Production:** Ungraded forum; add your “community norms” text.

### Activity 4-c: The CA "Trust" Audit (Browser Exploration)**

* **Goal:** Critically evaluate the "Trust" model in modern browsers.
* **Format:** Reflection and Research task.
* **Task:** Students follow your instructions to find the CA list in their own browser.
* **Prompt:** "Find a CA you have never heard of. Research where they are based. Do you feel comfortable with this entity being able to issue a certificate for your bank?"
* **Discussion:** Connect this to the "Any CA can sign for Any Domain" problem.


### Activity 4-d (Concept Contrast + Quick Check): **Web‑of‑Trust (PGP)** and **TOFU/SSH**

**Alignment:** §13.7 *Web of Trust*, §13.8 *Leap‑of‑Faith (TOFU)*  
**Goal:** Show that not all ecosystems use CA hierarchies; compare usability & threat models.

**Instructions**

*   2‑panel reading:
    *   PGP **web‑of‑trust**—local endorsements, non‑transitive trust pitfalls (as in your text).
    *   **SSH TOFU**—store‑on‑first‑seen host keys; defends against future attacks but not first‑contact MITM.
*   Quick Check (MCQ): *Who/what you must trust, and when, in each model.*
*   Short reflection: “Where in your projects would TOFU be ‘good enough,’ and where would you insist on CA‑backed identity?”

**Production:** MCQ + 2‑sentence reflection.


### C6. Reading + Quick Check — **Revocation Trade‑offs**

*   **Maps to**: §13.6 Revocation.
*   **Focus**: Compare **validity periods** (short‑lived certs) vs **revocation lists** (signed lists; update frequency; DoS and staleness dilemmas).
*   **Format**: 5 MCQs (automark).
*   **Production**: H5P; include a simple table image holder that illustrates the trade‑off (no live CRL/OCSP coding needed here). 

### Activity 6-b (Short Reading + Quick Check): **Certificate Transparency (CT)** in one page

**Alignment:** §13.4–13.6 (web PKI realities).  
**Goal:** Understand that WebPKI adds *public logs* so mis‑issuance can be detected; know the artifact: **SCT**.

**Instructions**

1.  Read a one‑page explainer:
    *   What CT logs are; **SCT** in cert/handshake; *monitoring & auditing*; CT v2.0 (**RFC 9162**) replaces v1.0 (**RFC 6962**). [\[datatracker.ietf.org\]](https://datatracker.ietf.org/doc/rfc9162/), [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc9162.pdf)
2.  Quick Check (MCQ):
    *   Why CT exists alongside PKI? → *Detect mis‑issuance and provide accountability.* **Answer**. [\[datatracker.ietf.org\]](https://datatracker.ietf.org/doc/rfc9162/)

**Optional research (ungraded):** Look up a domain on **crt.sh** to see its issued certs and SCTs (classroom note with URLs). *(No dependency in grading.)*

### Activity 6-c: Revocation Dilemma: CRL vs. OCSP (Decision Matrix)

* **Goal:** Compare the trade-offs of different revocation methods.
* **Format:** Comparison Table / MCQ.
* **Context:** Students are given scenarios (e.g., "A high-security government facility" vs. "A public Wi-Fi portal") and must decide if **Certificate Revocation Lists (CRLs)** or short **Validity Periods** are more appropriate based on bandwidth and security requirements.



### C6-b. Case Discussion (Ungraded) — **Web‑of‑Trust: Non‑Transitive & Purpose‑Bound Trust**

*   **Maps to**: §13.7 Web of Trust.
*   **Prompt**: Evaluate a path Alice→Bob→Carol→Doug; discuss why *trust isn’t transitive* and *not absolute* (“I trust my bank with my money but not with my children”).
*   **Outcome**: Short post (150–200 words) on whether multiple short paths improve assurance and when usability defeats the model.
*   **Production**: Use your “ungraded online community” wording pattern from Unit‑1 discussions. , 

### C8. Compare & Contrast — **SSH TOFU vs Hierarchical PKI**

*   **Maps to**: §13.8 Leap‑of‑Faith (TOFU).
*   **Focus**: Pros/cons of **Trust‑On‑First‑Use**: defends against passive eavesdroppers and attackers not present on first contact; still vulnerable to first‑session MITM; emphasizes *usable security* principles (“one secure mode; user needn’t understand crypto”).
*   **Format**: Two‑column drag‑sort (benefits vs risks) + 3 MCQs.
*   **Production**: H5P drag‑and‑drop. 

### Activity 8-b: TOFU vs. PKI (Pros/Cons Sorting)

* **Goal:** Compare the "Leap-of-Faith" (SSH) model with the "Certificate" (HTTPS) model.
* **Format:** Sorting categorization activity.
* **Categories:** Usability, Scalability, Resistance to first-time MITM, Complexity.
* **Context:** Helps students understand that "Usable Security" sometimes involves trade-offs in absolute rigor.


### C10. Short Quiz — **Certificates, Chains, Revocation, Alternatives**

*   **Maps to**: whole topic; prepares them for protocol‑level use later.
*   **Format**: 10 mixed items (MCQ/matching/order).
*   **Production**: H5P; can be graded low‑stakes or formative. 

************************************************************************
# Topic: Passwords


## Topic learning outcomes (derived from your draft)

By the end of this topic, students should be able to:

1.  Describe the **risk landscape** for passwords (eavesdropping, phishing, online guessing, client‑side malware, server compromise). 
2.  Explain and evaluate **mitigations** (TLS for transport; rate‑limiting/CAPTCHAs nudges for online guessing; slow, salted hashing for server storage). 
3.  Implement and reason about **password hashing “done right”** (unique per‑user salts + slow hash via iteration; e.g., PBKDF2/Bcrypt/Scrypt). 
4.  Analyze **why password‑derived keys are weak** and articulate better alternatives or multi‑factor approaches. 

### P1. Reading + Quick Check — “What Can Go Wrong with Passwords?”

*   **Maps to**: §1 Risks and weaknesses.
*   **Format**: Concise reading + 6 MCQs (automark).
*   **Focus**: Distinguish **online guessing**, **phishing/social engineering**, **eavesdropping**, **client‑side malware**, **server compromise**, and consequences of **reuse**. 
*   **Production**: H5P Question Set; one small risk diagram image holder.

### Activity 1 (Reading + Quick Check): Threats & Where Passwords Fail

**Alignment:** §1 Risks & weaknesses  
**Goal:** Make students map each risk (online guessing, phishing, eavesdropping, client malware, server compromise) to a first‑line mitigation, and recognize where passwords fundamentally can’t help (malware, phishing at scale).

**Instructions**

1.  Read a short LMS page summarizing your §1 list with two columns: *threat* → *primary defense*.
2.  Complete 6 MCQs (one per risk + “password reuse”/credential‑stuffing).

**Answer key (examples)**

*   *Eavesdropping* → **TLS/HTTPS** for the login flow and account recovery flows. **(Answer)** [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
*   *Server compromise* → **Salted + slow, memory‑hard hashing**; never store plaintext. **(Answer)** [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
*   *Phishing* → **Phishing‑resistant MFA** (e.g., WebAuthn/FIDO2 passkeys). **(Answer)** [\[w3.org\]](https://www.w3.org/TR/webauthn-2/)

**Production notes**

*   Use your own wording from §1; our MCQ feedback can link to a one‑line mitigation summary.

***


### P2. Concept Check — Eavesdropping vs. TLS (Why HTTPS Is the Default Defense)

*   **Maps to**: §2 Mitigations for eavesdropping.
*   **Format**: 4 MCQs + 1 short “explain in 2 lines” (show/hide sample answer).
*   **Focus**: Why TLS beats DIY challenge–response in the browser; passwords traveling in clear vs encrypted channel. 
*   **Production**: H5P; figure holder for “http” vs “https” lock metaphor with alt text.

### Activity 2 (Reading + Short Exercise): TLS Is Table‑Stakes

**Alignment:** §2 Eavesdropping  
**Goal:** Recognize that *all* credential entry and recovery must be over HTTPS, HSTS is preferred, and mixed content must be eliminated.

**Instructions**

*   Mini‑checklist: “Is my login safe?” → HTTPS only, secure cookies, HSTS for the domain, no mixed active content on the auth pages. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
*   Exercise (offline): Given three synthetic `curl` headers from a mock site, identify the one that correctly enforces HTTPS (HSTS + secure cookies).

**Answer key:** Show expected headers and why.

***


### P3. Mini‑Scenario — Why Client‑Side Malware Breaks Passwords

*   **Maps to**: §3 Client‑side malware.
*   **Format**: 3 micro‑scenarios (keylogger; virtual keyboard; screen capture) → per‑scenario MCQ on whether the password remains safe and why **2FA** helps. 
*   **Production**: H5P Course Presentation; short call‑out that passwords are **fundamentally insecure** in this threat model. 

### Activity 3 (Case + Quick Check): Client Malware & What Actually Helps

**Alignment:** §3 Client‑side malware  
**Goal:** Accept that passwords can’t survive malware; distinguish weak MFA (e.g., SMS) vs. phishing‑resistant MFA (WebAuthn passkeys).

**Instructions**

*   Read: 1‑page explainer on authenticator assurance levels and authenticator types. NIST guidance allows multiple factors; SMS has risks; synced authenticators (“passkeys”) are now explicitly recognized in NIST’s supplemental guidance. [\[csrc.nist.gov\]](https://csrc.nist.gov/pubs/sp/800/63/b/upd2/final), [\[nist.gov\]](https://www.nist.gov/identity-access-management/projects/nist-special-publication-800-63-digital-identity-guidelines)
*   Quick Check (3 MCQs): rank SMS vs TOTP vs **WebAuthn** for phishing/malware resistance; when to require a second factor. [\[w3.org\]](https://www.w3.org/TR/webauthn-2/)

***


### P4. Interactive Estimation — Targeted vs. Untargeted Online Guessing

*   **Maps to**: §4 Online guessing attacks (statistics, targeted vs untargeted).
*   **Format**: Learners choose strategies; we compute **expected attempts** and time (using the order‑of‑magnitude figures from your text) and ask 4 MCQs. 
*   **Production**: H5P; clear notes on assumptions; no external data.

### Activity 4 (Reading + Quick Check): Online Guessing, Rate‑Limiting & “Breached Password” Checks

**Alignment:** §4 Online guessing + §5 mitigations  
**Goal:** Move students from composition rules to **length + screening + throttling**, aligned with modern guidance.

**Instructions**

*   “What actually works” list:
    *   **Rate limiting / progressive delays / risk‑based lockouts** for targeted attacks. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
    *   **Check new passwords against lists of commonly‑used or breached passwords** (e.g., via a privacy‑preserving k‑anonymity API pattern). [\[csrc.nist.gov\]](https://csrc.nist.gov/pubs/sp/800/63/b/upd2/final), [\[blog.cloudflare.com\]](https://blog.cloudflare.com/validating-leaked-passwords-with-k-anonymity/)
    *   Avoid arbitrary composition rules; allow paste; no periodic forced resets without evidence of compromise (per NIST). [\[csrc.nist.gov\]](https://csrc.nist.gov/pubs/sp/800/63/b/upd2/final)
*   MCQs reinforce each item above.

**Production notes**

*   Keep the tone pragmatic (tie back to your §5 “rate‑limiting is good but not a panacea”).

***

### P5. Reading + Quick Check — Rate‑Limiting, CAPTCHAs, and Nudges

*   **Maps to**: §5 Mitigations for online guessing.
*   **Format**: 6 MCQs (automark).
*   **Focus**: When **rate‑limiting** helps (targeted), its **DoS downside**; why **CAPTCHAs** raise cost but don’t stop broad attacks; why **password meters/nudges** can improve choices. 
*   **Production**: H5P; include a small pros/cons table (accessible).

### **Activity 5: Rate-Limiting vs. DoS (Scenario Reflection)**

* 
**Goal:** Analyze the trade-off between protecting against guessing and enabling Denial-of-Service.


* **Format:** Multiple Choice / Short Answer.
* **Context:** A developer proposes locking an account for 24 hours after 3 failed attempts. Students must identify the specific risk (Mallory can lock Bob out of his own account) and suggest a more balanced approach (e.g., progressive delays or CAPTCHAs).


### P6. Micro‑Lab (Ubuntu‑in‑Docker) — **Why Plain Hashing is Dangerous**

*   **Maps to**: §7 Password hashing (fast hash pitfalls) + §6 Server compromise.
*   **What learners do (≈20–25 min):**
    1.  Hash two common candidate passwords with SHA‑256 and time 1e6 iterations (`openssl dgst` or Python `hashlib`) to **feel how fast SHA‑256 is**.
    2.  Observe identical hashes for identical inputs across users → motivates **amortized guessing**.
    3.  Answer 4 MCQs about **offline guessing** and **amortization**. 
*   **Production**: Shell + optional Python blocks; automarked conceptual questions; reinforce “do not store plain SHA‑256.” 

### P6-b. Hands‑On Lab (Ubuntu‑in‑Docker) — **Salting and Slow Hashing Done Right**

*   **Maps to**: §8 Password hashing, done right.
*   **What learners do (≈30–40 min):**
    *   Use Python `hashlib`’s `pbkdf2_hmac` to compute `PBKDF2(SHA‑256, salt, iterations)` for one password, **changing only the salt** to see output diversity; then **vary the iteration count** (e.g., 10k → 200k) and time the effect.
    *   Compare to a **plain SHA‑256** path.
    *   Answer 6 MCQs + 1 short calculation (how iteration count influences server CPU budget at \~10 logins/sec). 
*   **Production**: Step‑by‑step commands; automark conceptual answers; notes that in practice teams should use **PBKDF2/Bcrypt/Scrypt** from reputable libs (not roll‑your‑own). 

### Activity 5 (Hands‑On Lab, offline): **Salts + Slow KDFs**—Tuning for Defense

**Alignment:** §6–§8 (server compromise, hashing right)  
**Goal:** See—by measurement—why we use per‑user salts and **slow / memory‑hard** KDFs (PBKDF2, scrypt), and how parameter tuning trades server CPU for attacker time.

**Environment:** Ubuntu‑in‑Docker (Python 3 is available).

**Steps (ready to paste)**

```python
# passwords_kdf_lab.py
import os, time, hashlib, binascii, secrets

pwd = b"CorrectHorseBatteryStaple!"          # demo password
salt = secrets.token_bytes(16)               # 128-bit random salt

# 1) PBKDF2-HMAC-SHA256 with ~600k iterations (OWASP 2026 min)
t0 = time.time()
dk_pbkdf2 = hashlib.pbkdf2_hmac("sha256", pwd, salt, 600_000, dklen=32)
t1 = time.time()

# 2) scrypt with OWASP-minimum-like params (adjust to your CPU)
#   N=2^17, r=8, p=1  (may take noticeable time on student VMs)
t2 = time.time()
dk_scrypt = hashlib.scrypt(pwd, salt=salt, n=131072, r=8, p=1, dklen=32)
t3 = time.time()

def hx(b): return binascii.hexlify(b).decode()

print("Salt(hex):", hx(salt))
print("PBKDF2 600k iters:", hx(dk_pbkdf2), "time(s):", round(t1-t0,3))
print("scrypt N=2^17,r=8,p=1:", hx(dk_scrypt), "time(s):", round(t3-t2,3))
```

**Prompts (auto‑marked)**

*   Paste the runtimes—did scrypt cost more CPU/time than PBKDF2 on your VM? Why is that desirable against GPUs/ASICs? **Expected:** memory hardness raises attacker cost. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc9106.html)
*   Explain why **unique per‑user salts** defeat amortized guessing/rainbow tables. **Expected:** same password ≠ same hash across users. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

**Standards box**

*   **Preferred today:** **Argon2id** (RFC 9106) where available; otherwise **scrypt** or **bcrypt**; **PBKDF2** only for FIPS contexts—use high iterations (e.g., ≥600k for HMAC‑SHA‑256) per OWASP 2026. [\[rfc-editor.org\]](https://www.rfc-editor.org/rfc/rfc9106.html), [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
*   PBKDF2 is standardized in **RFC 8018**. [\[datatracker.ietf.org\]](https://datatracker.ietf.org/doc/html/rfc8018)

**Production notes**

*   If student VMs are slow, allow them to reduce scrypt `n` to `2^16` for runtime, then discuss trade‑offs.

***
### **Activity 4: Lab – Hashing "Done Right" with OpenSSL (CLI Lab)**

* 
**Goal:** Compare the speed of standard hashes vs. slow (iterated) hashes.


* **Format:** Hands-on lab in the Ubuntu-in-Docker environment.
* **Task:**
1. Time how long it takes to compute 1 million SHA-256 hashes using a script.
2. Use a tool (or a simple loop) to implement **Iterated Hashing** (e.g., $n=10,000$) and observe the performance difference.


3. Generate a **salted hash** and observe that identical passwords now result in different stored values.


### Activity 6 (Mini‑Lab, offline): **Pepper & Verification Flow** (architecture exercise)

**Alignment:** §8 “hashing done right”  
**Goal:** Understand how a **server‑held pepper** complements per‑user salts and where to store it.

**Instructions**

*   Provide a one‑page architecture diagram: password → KDF(pwd || salt || **pepper**) → store `salt, hash`. Pepper lives in app secret store / HSM, rotated under incident, never in the user row. (OWASP lists pepper as optional “defense‑in‑depth”.) [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
*   Short answer: *What breaks if the database leaks but the pepper remains secret?* → offline cracking cost rises; at scale, all hashes become invalid without the pepper.

**Answer key** included.

***

### P6-c. Walkthrough — Breaking the Amortized Attack with Salts

*   **Maps to**: §8 (per‑user random salt stops precomputed lookups; raises attacker work).
*   **Format**: Ordering + 4 MCQs; one mini “proof‑by‑counterexample” where students see why a single precomputed table won’t match multiple salts. 
*   **Production**: H5P Ordering/MCQ.

### P6-d. Thought Experiment — Tuning the Iteration Count

*   **Maps to**: §8 choosing `n` (slow hash cost trade‑off).
*   **Format**: 3 param sets; students decide which meets **1% CPU** target for 10 logins/sec and gives acceptable **attack slow‑down**, with immediate feedback and an answer key. 
*   **Production**: H5P; clear numeric examples drawn from your text narrative. 

### P10. Concept Check — Why Password‑Derived Keys Are Weak

*   **Maps to**: §9 Implications for cryptography.
*   **Format**: 5 MCQs + 1 short answer; students contrast **key from random 128‑bit** vs **key from human password via slow hash**; time‑to‑crack intuition (2^20 common candidates). 
*   **Production**: H5P; emphasize “prefer random keys; if you must use passwords, understand the limits.” 
### Activity 8 (Reading + Poll): From Passwords to **Passkeys (WebAuthn/FIDO2)**

**Alignment:** §10 Alternatives  
**Goal:** Give learners the “why now” for passkeys and how they’re phish‑resistant and increasingly standardized.

**Reading nuggets**

*   **WebAuthn** is a W3C Recommendation (Level 2 in 2021; Level 3 is a 2026 Candidate Snapshot); credentials (passkeys) bind to a **relying party** and are **phishing‑resistant** because signatures are scoped to the origin. [\[w3.org\]](https://www.w3.org/TR/webauthn-2/), [\[w3.org\]](https://www.w3.org/TR/webauthn-3/)
*   Platform vendors (Apple/Google/Microsoft) jointly committed (May 5, 2022) to expanded **FIDO** support so passkeys sync across devices—removing many UX blockers. [\[apple.com\]](https://www.apple.com/newsroom/2022/05/apple-google-and-microsoft-commit-to-expanded-support-for-fido-standard/), [\[fidoalliance.org\]](https://fidoalliance.org/apple-google-and-microsoft-commit-to-expanded-support-for-fido-standard-to-accelerate-availability-of-passwordless-sign-ins/)

**Poll:** “Where would you deploy passkeys first in your projects?” (banking, campus SSO, admin portals, etc.)

**Production notes**

*   Keep it short; main passkey implementation work will be in your later authentication module.

***


### P11. Compare & Contrast — Alternatives to Passwords

*   **Maps to**: §10 Alternatives (2FA, one‑time PINs, SSH keys, persistent cookies).
*   **Format**: Drag‑sort into “Adds a factor,” “Replaces password,” “Session convenience”; 4 MCQs about where each fits. 
*   **Production**: H5P drag‑and‑drop; brief notes only (as in your section).

### P12. Short Quiz — End‑of‑Topic Sweep

*   **Maps to**: Whole topic.
*   **Format**: 10 mixed items (MCQ/matching/ordering) covering threats, mitigations, hashing best practices, and limitations.
*   **Production**: H5P; can be graded low‑stakes or formative, per your pattern. 

************************************************************************************************