---
title: Passwords
parent: Cryptography
nav_order: 10
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Passwords

## Authentication

_Authentication_ is the process of using supporting evidence to corroborate an asserted identity. In other words, a party claims to be a particular entity, and authentication is the act of checking whether that claim is true.

It is useful to distinguish authentication from two closely related concepts:

- **Identification** (or recognition) is the process of establishing an identity from available information _without_ an explicit claim. For example, a system might recognize a returning user from a cookie or a biometric template.
- **Authorization** is the process of deciding whether a request should be granted, given the identity of the requester. Authentication answers “Who are you?”; authorization answers “What are you allowed to do?”

In practice the three processes are often combined, but keeping them conceptually separate helps avoid design mistakes.

Authentication mechanisms are traditionally classified according to the kind of evidence they rely on:

- **Something you know** — a secret such as a password, PIN, or passphrase.
- **Something you have** — a physical object such as a hardware token, smart card, or mobile phone that can receive a one-time code.
- **Something you are** — a biometric characteristic such as a fingerprint, face, or iris pattern.

Systems that combine two or more of these factors are called _multi-factor authentication_. Passwords belong to the first category (“something you know”) and remain the most widely deployed authentication mechanism on the web, despite their well-known limitations. The rest of this topic examines those limitations and the practical defenses that can be applied.

## Passwords as an Authentication Mechanism

A password is a secret associated with a public user identity (a userid or username). In the simplest form of password authentication the user sends both the userid and the password to the server. The server looks up the stored credential for that userid and checks whether the supplied password matches. If it does, the server accepts the claimed identity.

Despite decades of criticism, passwords remain the dominant authentication mechanism on the web and in many other systems. They require no special hardware, work across virtually every platform, and are conceptually simple for both users and developers. Deploying an alternative (smart cards, hardware tokens, biometrics, or pure public-key authentication) usually involves extra cost, extra software, or extra user education. As a result, passwords continue to be the default choice for the great majority of online services.

The fundamental tension in password systems is easy to state but hard to resolve:

- For security, passwords should be long, random, and unique to each service.
- For usability, passwords must be memorable (or at least easy to type and manage).

Users who try to satisfy the security requirement often end up writing passwords down, reusing them across sites, or choosing simpler ones that they can remember. Users who prioritize memorability tend to choose short, predictable, or reused passwords that are vulnerable to guessing. Almost every practical problem discussed in the rest of this topic is a consequence of this tension.

## Threats to Password Security

Password authentication is exposed to a wide range of attacks. The most important threats are the following.

### Online Guessing Attacks

An attacker repeatedly tries to log in to an account by submitting candidate passwords. If the password is weak or chosen from a small set of common choices, the attacker may eventually succeed. Because each guess requires interaction with the live server, the attack is limited by network latency and by any rate-limiting the server may impose.

Researchers have studied the statistics of passwords as used in the field, and the results suggest that online guessing attacks are a realistic threat. According to one source, the five most commonly used passwords are 123456, password, 12345678, qwerty, and abc123. A careful measurement study found that with a dictionary of the 10 most common passwords, you can expect to find about 1% of users’ passwords. Furthermore, with a dictionary of the $2^{20}$ most commonly used passwords, you can expect to guess about 50% of users’ passwords.

This implies that we can distinguish targeted from untargeted attacks. For an untargeted attack, where the attacker just wants to hack _some_ account (e.g., to send spam), the attacker might try 10 guesses against each of a large list of accounts. The attacker can expect to have to try about 100 accounts, making a total of about 1000 login attempts, to guess one user’s password correctly. Since this can be automated, resistance against untargeted attacks is very low. For a targeted attack, if the attacker is mildly lucky, they might succeed after about one million guesses (which happens half of the time). If each attempt takes 1 second, making $2^{20}$ guesses will take about 11 days, and the attack is very noticeable.

### Offline Guessing Attacks (After Server Compromise)

If an attacker obtains a copy of the server’s password database, the guessing process can be performed entirely locally, without further contact with the server. This is called an _offline_ attack. When passwords are stored only as fast cryptographic hashes, modern hardware can test billions of candidate passwords per second. Consequently, even moderately complex passwords can often be recovered in a short time once the database has been stolen.

### Eavesdropping

If the password is transmitted in cleartext (for example over an unencrypted HTTP connection or an open Wi-Fi network), an eavesdropper can simply record it. The standard defense is to send the password only over a TLS-protected channel (HTTPS).

Another possible defense would be to use more advanced cryptographic protocols. For instance, one could imagine a challenge-response protocol where the server sends your browser a random challenge $r$; then the browser takes the user’s password $w$, computes $H(w, r)$ where $H$ is a cryptographic hash, and sends the result to the server. In this scheme, the user’s password never leaves the browser, defending against eavesdroppers. While such a scheme could be implemented today with Javascript on the login page, it has little or no advantage over SSL (and has some shortcomings compared to it), so the standard defense is simply to use SSL/TLS.

### Phishing and Social Engineering

An attacker may trick the user into revealing the password directly—most commonly by presenting a forged login page that looks identical to the legitimate site. Because the user voluntarily types the password into the attacker’s page, cryptographic protections on the real site provide no help.

### Client-Side Malware

Keyloggers and other forms of malware running on the user’s device can capture the password as it is typed or retrieved from a password manager. To defend against keyloggers, some have proposed using randomized virtual keyboards: a keyboard is displayed on the screen with the order of letters randomly permuted, and the user clicks the characters of their password. However, it is easy for malware to defeat this scheme by recording the location of each mouse click and taking a screenshot.

Once the endpoint is compromised, the password is essentially available to the attacker; no server-side measure can fully prevent this. Passwords are fundamentally insecure in this threat model, making multi-factor authentication the main practical defense.

### Password Reuse Across Sites

Users frequently reuse the same password on multiple services. When one of those services is breached and the password is recovered (especially if it was stored poorly), the attacker can try the same credential on many other sites. A single server compromise can therefore cascade into the compromise of accounts that the user holds elsewhere.

These threats are not mutually exclusive; real attacks often combine several of them. The defenses discussed in the following sections address different subsets of the threat model and must be understood in that light.

## Defenses Against Online Guessing

Because online guessing requires interaction with the live server, the server can observe and throttle the attack. Several practical defenses are commonly used.

### Rate Limiting

The server limits the number of failed login attempts that may be made against an account in a given time window (for example, five failures per hour). Once the limit is reached, further attempts are delayed or rejected. Rate limiting makes a targeted attack that needs millions of guesses impractically slow, while still allowing legitimate users who occasionally mistype their password to recover. For instance, if we limit each account to 5 incorrect guesses per hour, making $2^{20}$ guesses would take at least 24 years—making at least half the user population immune to targeted attacks. Unfortunately, one research study found that only about 20% of major web sites currently use rate-limiting.

### Account Lockout and Its Denial-of-Service Risk

A stricter policy locks the account after a small number of consecutive failures and requires an out-of-band recovery step (e-mail confirmation, phone call, etc.). Lockout is effective against concentrated guessing, but it introduces a denial-of-service opportunity: an attacker who knows a valid userid can deliberately trigger the lock and prevent the legitimate user from logging in. Many services therefore prefer soft rate limiting over hard lockout, or they combine a modest lockout threshold with additional signals (IP reputation, device history, etc.).

Furthermore, rate-limiting and lockouts are not effective defenses against untargeted attacks. An attacker who makes 1 guess against each of 1000 accounts can expect to break into at least one of them, without ever triggering the rate limit.

### CAPTCHAs and Their Limitations

After one or more failed attempts the server may require the client to solve a CAPTCHA before further guesses are accepted. The intention is to raise the cost of automated attack scripts. In practice the protection is limited. Commercial CAPTCHA-solving services employ human workers in countries with low wages and charge only about $1–$2 per thousand CAPTCHAs solved (0.1–0.2 cents each), so a determined attacker can still afford large numbers of guesses. In addition, an untargeted attack that makes only one or two guesses per account may never trigger the CAPTCHA at all.

### Password Strength Requirements and Nudges

Sites often impose composition rules (minimum length, required character classes, banned common passwords). Strict rules improve resistance to guessing but frequently harm usability; users respond by writing passwords down, reusing them, or choosing predictable patterns that satisfy the rules while remaining weak. A gentler alternative is a _password meter_ displayed during account creation. Empirical studies show that real-time visual feedback encourages many users to choose longer and less predictable passwords without the frustration of hard rejection.

No single defense eliminates online guessing. In practice a combination of rate limiting, careful lockout policy, and user-friendly strength guidance provides a reasonable balance between security and usability.

## Defenses Against Server Compromise (Password Storage)

When an attacker obtains a copy of the server’s authentication database, the security of every stored password is at risk. How the server stores those passwords determines how much damage the breach causes.

### Why Storing Passwords in Cleartext Is Catastrophic

The simplest (and worst) approach is to store the password itself in the database. A single breach then reveals every user’s credential in usable form. Because many people reuse passwords across sites, the attacker can immediately try the stolen passwords on other services. Cleartext storage is therefore considered a fundamental failure of security engineering. For example, in 2009, the Rockyou social network got hacked, and the hackers stole the passwords of all 32 million of their users and posted them on the Internet. Despite this, one study estimates that about 30–40% of sites still store passwords in the clear.

### Basic Password Hashing

A better approach is to store a cryptographic hash of the password rather than the password itself. When a user creates an account with password $w$, the server computes $H(w)$ (using a one-way hash such as SHA-256) and stores only the hash. On later logins the server hashes the offered password and compares the result with the stored value. Because a cryptographic hash is one-way, the original password cannot be recovered directly from the stored value.

### Why Plain Hashing Is Still Weak

Two problems remain. First, modern hardware can evaluate fast hashes such as SHA-256 at rates of hundreds of millions or billions of candidates per second. An attacker who has stolen the hash database can therefore test enormous numbers of guesses offline.

Second, the same password always produces the same hash. Consequently, an attacker can perform a dramatically sped up _amortized_ attack that avoids unnecessarily repeating work. They can compute a list of $2^{20}$ pairs $(H(g), g)$ for the $2^{20}$ most common passwords, and sort this list by the hash value. Now, for each user in the database, they check if the user's hash is in the sorted list. Using a binary search, checking the list takes extremely efficiently ($\lg 2^{20} = 20$ random accesses). This attack requires computing $2^{20}$ hashes (about one millisecond), sorting the list (fractions of a second), and doing binary searches (seconds or minutes). This recovers many passwords with almost no extra work.

### Salting

The amortization problem is solved by _salting_. When a new account is created the server generates a fresh random value $s$ (the salt) and stores the pair

$$
(s,\ H(w,s))
$$

Even if two users choose the identical password, their salts differ, so their stored hashes differ. An attacker can no longer pre-compute one dictionary of hashes and apply it to the whole database; each user’s salt forces a separate set of computations. The salt does not need to be secret and is stored in cleartext.

### Slow / Iterated Hashing

Salting alone is not enough while the underlying hash remains fast. For instance, when LinkedIn had a security breach that exposed their password hashes, it was discovered they were using SHA256; consequently, one researcher recovered 90% of their users’ passwords in just 6 days.

The standard remedy is to replace the ordinary hash with a deliberately slow function. One way to take a fast hash function and make it slower is by iterating it:

$$
F(x) = H(H(H(\cdots(H(x))\cdots)))
$$

Common constructions iterate a fast hash many times or use memory-hard algorithms:

- PBKDF2 (Password-Based Key Derivation Function 2)
- bcrypt
- scrypt
- Argon2 (the winner of the Password Hashing Competition)

These functions take a tunable cost parameter so that computing one hash requires a non-trivial amount of time or memory (typically tens to hundreds of milliseconds on the server). We must choose this parameter to provide security without slowing down the legitimate server. For instance, suppose we have a site that expects at most 10 logins per second. We could tune the hash to take one millisecond; the server expects to spend 1% of its CPU power on hashes—a small hit. But for an attacker, trying $2^{20}$ candidate passwords against 100 million users would now take about 3000 machine-years instead of 1 day.

### Recommended Modern Practice

The currently accepted best practice is:

1. Generate a unique, cryptographically random salt for every password.
2. Apply a modern, slow, memory-hard password hashing function (Argon2id is the preferred choice; bcrypt or scrypt remain acceptable).
3. Store only the salt and the resulting slow hash.
4. Choose the cost parameter so that legitimate authentication remains fast enough for the service while offline guessing becomes prohibitively expensive.

When these steps are followed, a database breach still reveals that an account exists, but recovering the actual passwords becomes far more difficult and costly for the attacker.

## Password-Based Cryptography (Implications)

It is tempting to turn a human-memorable password into a cryptographic key by simply hashing it. For example, a file-encryption tool might compute $k = H(w)$ and then encrypt the file under the symmetric key $k$. This practice is dangerous.

Because ordinary cryptographic hashes are fast, an attacker who obtains the ciphertext can try the most common passwords, hash each candidate, and test whether decryption yields plausible plaintext. With a dictionary of a million likely passwords the attack often succeeds in seconds or minutes. Using a slow hash improves the situation only modestly. Suppose we use a slow hash tuned to take 1 millisecond. The attacker can still make 1000 guesses per second, and it will take only about 15 minutes to try all $2^{20}$ most likely passwords. Fifteen minutes to have a 50% chance of breaking the crypto is very weak security.

The practical conclusion is that passwords and passphrases should not be used as the sole source of cryptographic keys whenever strong security is required. It is far safer to generate a truly random key and to protect that key by other means (for example, by storing it in a hardware token or by encrypting it under a randomly generated key that is itself protected by a password).

A more sophisticated alternative is _password-authenticated key exchange_ (PAKE). PAKE protocols allow two parties who share only a low-entropy password to establish a high-entropy session key while resisting offline dictionary attacks. Protocols such as EKE, SPEKE, and the modern OPAQUE construction achieve this property, but they are considerably more complex than simple hashing and are still relatively uncommon in everyday applications. For most systems the prudent rule remains: treat passwords as authentication secrets, not as cryptographic keys.

## Alternatives to Passwords

Passwords remain ubiquitous, yet they are frequently insufficient on their own. When the risk associated with a compromised password is high, or when the threat model includes phishing, malware, or large-scale credential stuffing, stronger mechanisms are required.

### Multi-Factor Authentication

Multi-factor authentication (MFA) combines two or more independent authentication factors. The most common pattern pairs a password (“something you know”) with a second factor that is harder for an attacker to obtain—typically a one-time code delivered to a phone or generated by a hardware device (“something you have”). Even if the password is stolen, the attacker still needs the second factor to succeed. MFA dramatically raises the cost of many practical attacks and is now considered a baseline requirement for sensitive accounts.

### One-Time Codes and Hardware Tokens

One-time passwords (OTPs) can be delivered by SMS, generated by an authenticator app, or produced by a dedicated hardware token. Because each code is valid for only a short window and cannot be reused, interception or theft of a single code has limited value. Hardware tokens that implement challenge-response protocols (for example FIDO2/WebAuthn security keys) further reduce the risk of phishing, because the token authenticates the specific origin of the request rather than merely proving possession of a shared secret.

### Public-Key Based Authentication

Public-key methods eliminate the shared secret altogether. In SSH, for example, the server stores only the user’s public key; authentication consists of proving possession of the corresponding private key. Similar techniques appear in client-certificate TLS authentication and in modern passwordless schemes built on WebAuthn. When the private key is held in a secure element or hardware token, the attack surface associated with password storage and transmission disappears.

### When Passwords Are No Longer Enough

Passwords alone are inadequate whenever any of the following conditions hold:

- the account protects high-value assets or sensitive personal data,
- the user population is a likely target of phishing or targeted malware,
- regulatory or industry standards mandate stronger authentication,
- the service must defend against large-scale automated credential stuffing.

In these settings the recommended practice is to treat the password (if it is retained at all) as only one component of a multi-factor or passwordless design. The long-term direction of the industry is toward phishing-resistant authenticators that rely on public-key cryptography and hardware-backed key storage rather than on human-memorable secrets.
