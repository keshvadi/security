---
title: Intrusion Detection
parent: Network Security
nav_order: 14
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Intrusion Detection

## Cheat sheet

- **Core Idea**: Perfect prevention is impossible. Even the best-designed systems will be attacked successfully at some point. Detection, response, and containment are therefore essential layers of defense in depth.
- **Main Detector Placements**:
  - **NIDS** (Network IDS): Sits at the choke point (router), sees all external traffic. Cheap and central, but must reconstruct TCP streams, faces evasion attacks, and is blind to encrypted (HTTPS) content without key escrow.
  - **HIDS** (Host-based IDS): Runs on the endpoint itself (antivirus, system-call monitors, instrumented servers). Sees decrypted traffic and true semantics, but must be deployed everywhere and still faces parsing inconsistencies at the application or filesystem layer.
  - **Logging**: Cheap post-facto evidence from existing server logs. Delayed, tamperable, cannot prevent damage in real time.
  - **System-call / behavioral monitoring**: Watches for the _effects_ of compromise (reading `/etc/passwd`, calling `exec`, clearing history) rather than the attack string itself.
- **Main Detection Strategies**:
  - _Signature-based_ (blacklist known bad patterns) — excellent for known attacks, useless for novel ones.
  - _Anomaly-based_ (learn "normal" with ML/stats) — can catch unknowns but training data and concept drift are hard.
  - _Specification-based_ (manually write precise "normal" rules) — low FP if specs are good, extremely labor-intensive.
  - _Behavioral_ (look for evidence of successful compromise or attacker actions) — high signal, can stop attacks in progress, independent of the exact exploit string.

---

## Why We Need Detection

Throughout this book we have studied prevention: firewalls that block unwanted traffic, TLS that protects confidentiality and integrity, DNSSEC that authenticates responses, SSH that replaces plaintext remote login, and so on. These controls are valuable, but they are never perfect. Code has bugs, configurations drift, users make mistakes, insiders turn malicious, and novel attacks appear faster than patches can be written.

A prudent organization therefore assumes that prevention will eventually fail for _some_ attack, against _some_ asset, at _some_ time. When that happens we need three additional capabilities:

- **Detection** — notice that something bad is (or was) happening.
- **Response** — stop the bleeding and limit damage.
- **Recovery / Containment** — restore service and remove the attacker's foothold.

This topic focuses on the detection problem. The running example we will use is a public web server that is supposed to serve only files under `/public/files/`. An attacker who can trick it into serving `/etc/passwd` has achieved a serious information leak. We will see how different detectors try to notice this (or similar) attacks and why each approach has characteristic blind spots.

## Network Intrusion Detection Systems (NIDS)

A **NIDS** is placed on the link between the organization's border router and the internal network (or at other strategic choke points). In principle it can see every packet that enters or leaves the site.

### Advantages

- One device protects thousands of hosts.
- No changes required on end systems ("bolt-on" security for legacy code).
- Central management and logging.
- Can be implemented in the same hardware pipeline as a firewall, keeping cost low.

### Disadvantages and Evasion

The NIDS does not have the same view of traffic that the end host has. It must reconstruct TCP connections from individual packets (handling reordering, loss, and retransmission), yet the reconstruction may differ from what the server actually receives or processes.

**Classic evasion example — path traversal with encoding**:
A naïve NIDS that simply greps for the string `../` or `/etc/passwd` in HTTP requests will miss:

- `profile=%2e%2e%2fetc%2fpasswd` (URL-encoded `../etc/passwd`)
- `profile=..%2f..%2fetc%2fpasswd` or `..%252f` (double encoding)
- `profile=/etc/./passwd` or `profile=/etc//passwd`

The web server (or its framework) will decode the request before passing the filename to the filesystem. The NIDS that never decoded, or decoded differently, misses the attack. This is an **evasion attack**.

**Encryption**:
Modern traffic is HTTPS (TLS). The NIDS sees only ciphertext and the outer TCP/IP headers. Without the session keys it cannot inspect the HTTP request at all. Some organizations solve this by terminating TLS at a reverse proxy that shares keys with the NIDS, or by installing inspection certificates on all internal clients. Both approaches have serious drawbacks for privacy, performance, and security.

**TCP/IP Stream and TTL tricks**:
Packets can be crafted so the NIDS sees them (TTL high enough) but an intermediate router or the end host drops them. Attackers can also exploit ambiguities in how different operating systems handle overlapping TCP segments or out-of-order packets. If the NIDS and the target server reassemble the TCP stream differently, the NIDS and the server will disagree on what data was actually delivered, allowing the attack payload to slip through unflagged.

Because of these problems, a production NIDS must contain a full TCP/IP stack plus application-layer parsers (HTTP, DNS, SMTP, etc.) and must reason about how each _particular_ end host would interpret the stream. That is a great deal of complexity.

## Host-Based Intrusion Detection Systems (HIDS)

A **HIDS** runs on the endpoint itself. Antivirus products, OSSEC, OS query monitors, and instrumented web-server modules are all examples.

### Advantages

- Sees traffic _after_ TLS decryption and _after_ the application has parsed it.
- Has direct access to filesystem semantics, process state, and system-call arguments.
- Can therefore detect path-traversal attacks that fool a NIDS (it can actually ask "would opening this filename reach `/etc/passwd`?").
- Works for encrypted traffic with no extra effort.

### Disadvantages

- Must be installed, configured, and updated on every machine.
- Different operating systems and applications need different HIDS logic.
- Still not perfect: the HIDS must understand the _application's_ idea of "normal" (e.g., a web server may legitimately read many files; distinguishing a traversal from a normal request may require deep knowledge of that specific application).
- Can consume CPU, memory, and disk on production systems.

## Logging as a Detection Mechanism

Most servers already produce logs: web access logs, authentication logs, application logs, `syslog`, etc. A simple detector can be a nightly (or real-time) script that scans those logs for suspicious patterns.

### Advantages

- Extremely cheap — the logging code is already there.
- No performance impact on the NIDS or the need to instrument every binary.
- Provides forensic evidence after the fact ("who was logged in when the password file was read?").

### Disadvantages

- Detection is delayed. By the time the log is examined the attacker may already have used the stolen passwords to log in and install a backdoor.
- Logs can be tampered with by a successful attacker (clearing `/var/log/`, editing `lastlog`, etc.).
- Still requires the same semantic understanding as a HIDS to avoid being fooled by evasion in the logged events.

## Monitoring System Calls and Behavioral Evidence

Instead of looking at the _input_ the attacker sends, we can look at the _effects_ the attacker produces once code is running.

Example: a C program is never supposed to call `execve("/bin/sh", ...)`. A detector that watches system calls can notice the call and raise an alert (or even kill the process) regardless of how the attacker got the shellcode there—buffer overflow, format string, ROP, etc.

Other behavioral signals:

- A web server process suddenly reading `/etc/passwd` or `/etc/shadow`.
- A user process clearing its shell history (`unset HISTFILE`, `echo > ~/.bash_history`).
- Unexpected outgoing connections, new listening sockets, or privilege-escalation syscalls.

**Advantages**: High signal-to-noise (successful compromise is rarer than attack _attempts_), works across many exploit variants, can sometimes stop the attack in progress.

**Disadvantages**: The attack has already begun; volume of system-call events is huge; legitimate programs sometimes do "suspicious" things (cron jobs reading password files, developers running `su`, etc.); false positives are easy if the policy is too broad.

## Comparing the Approaches

| Aspect                          | NIDS                            | HIDS / System-call monitor         | Logging                        |
| ------------------------------- | ------------------------------- | ---------------------------------- | ------------------------------ |
| Deployment cost                 | Low (one or few devices)        | High (every host)                  | Very low (already present)     |
| Encrypted traffic               | Blind without key sharing       | Sees plaintext at endpoint         | Sees whatever the app logs     |
| Evasion difficulty              | High (many TCP/encoding tricks) | Lower (sees real semantics)        | Medium                         |
| Real-time prevention            | Possible (can kill connections) | Possible (can kill processes)      | No (post-facto)                |
| Visibility into non-net attacks | None                            | Good (local malware, insiders)     | Depends on what is logged      |
| Subversion risk                 | Attacker must compromise NIDS   | Attacker must compromise each host | Attacker can edit logs on host |

Real organizations use _all_ of them in combination.

## False Positives, False Negatives, and Why Scale Matters

A **false negative** (FN) is a missed attack. A **false positive** (FP) is an alarm on benign activity.

It is trivial to build a detector with 0% FN: "always raise an alarm." It is equally trivial to build one with 0% FP: "never raise an alarm." Both are useless.

Any useful detector lives on a curve: improving one metric usually worsens the other. The right operating point depends on the _cost_ of each kind of error in your environment.

Even more important is the **base rate** — the fraction of events that are actually attacks. Suppose a detector has a 0.1% FP rate and a 2% FN rate.

- If you receive 1,000 web requests per day and 5 are attacks, you expect roughly 1 false positive per day (0.1% of 995 benign requests).
- If you receive 10,000,000 requests per day and still only 5 attacks, you expect roughly 10,000 false positives per day.

The detector has not changed; the environment has. At Internet or campus scale, even an excellent detector can flood analysts with alerts. This is the **base-rate fallacy** in action. Real detection systems therefore invest heavily in _tuning_, _prioritization_, _aggregation_, and _automated response_ so that human analysts only ever see the highest-confidence, highest-impact events.

**Sanity check**: A TRU network carries 50 million HTTP requests on a typical weekday; a handful are malicious. A signature-based NIDS with a 0.01% FP rate will still produce thousands of alerts per day. Why is this not a reason to abandon detection entirely?  
**Answer**: Because the goal is not "zero alerts." The goal is to surface the _real_ attacks with high enough priority and low enough volume that the security team can actually investigate and respond before damage compounds. Tuning, whitelisting known-good traffic, behavioral correlation, and automated playbooks all help turn an unmanageable firehose into actionable intelligence.

## Four Fundamental Detection Strategies

To understand how these strategies differ, consider how each one might be used to catch a classic _buffer overflow attack_:

### Signature-Based (Blacklisting Known Bad)

Maintain a database of known attack patterns ("if you ever see this exact sequence of bytes or this exact sequence of syscalls, raise an alert").

- Example: If an attack relies on a specific sequence of junk bytes followed by a known shellcode string, the detector can search for that exact byte pattern.
- Pros: Easy to implement, extremely effective against the attacks that are already in the wild and shared by the community (Snort rules, YARA signatures, ClamAV, etc.).
- Cons: Blind to zero-days and to any variant that differs enough to miss the signature. Attackers deliberately mutate their exploits (polymorphism, encoding, junk insertion) to evade signatures.

### Anomaly-Based (Learning "Normal")

Build a statistical or machine-learning model of what legitimate traffic or behavior looks like, then flag statistically unlikely events.

- Example: A C program expects a user's name as input. A trained model observes that normal activity consists entirely of printable keyboard characters. If an attacker attempts a buffer overflow using raw memory addresses and executable shellcode (non-printable characters), the model flags this as highly abnormal.
- Pros: Can detect previously unseen attacks.
- Cons: Requires high-quality training data that is free of attacks; models suffer from concept drift (normal behavior changes over semesters, after new software deployments, etc.); high FP rates are common when attacks are rare in the training set.

### Specification-Based (Manually Declaring "Normal")

A human writes precise rules that describe acceptable behavior for a particular program or protocol.

- Example: A programmer writes a specification stating that a specific input field accepts only numerical digits (e.g., an age). If an attacker sends buffer overflow shellcode (raw bytes that are not numbers), it violates the specification and is blocked.
- Pros: If the specification is accurate, FP rates can be driven extremely low; novel attacks are still caught.
- Cons: Extremely labor-intensive to write and maintain specifications for every important program; specs can contain errors that become permanent false negatives.

### Behavioral / "Evidence of Compromise"

Do not try to recognize the attack _input_. Instead, look for the _consequences_ an attacker would produce after a successful exploit, or for actions that are characteristic of attackers (clearing history, installing a new SSH key, beaconing home, etc.).

- Example: A C program is designed never to call the `exec()` function. If a buffer overflow successfully hijacks control flow and attempts to spawn a shell, the detector notices the unauthorized `exec()` call and flags this behavior as an attack.
- Pros: High signal, largely independent of the exploit vector and of the exact bytes the attacker sent.
- Cons: The attack is only detected after it has started.

## Honeypots and Deception

A **honeypot** is a system (or entire network) that has no production purpose. Any connection or login attempt is therefore, by definition, either a mistake or an attack.

Honeypots can:

- Give early warning that someone is scanning or targeting your organization.
- Divert attackers away from real assets.
- Allow defenders to study attacker tools, tactics, and procedures in a controlled environment.
- Feed high-quality attack data back into signature and anomaly systems.

**Challenges**: making the honeypot believable (attackers quickly learn to recognize obvious fakes), legal and ethical issues around monitoring, and the risk that a compromised honeypot becomes a launch pad for attacks on others.

Modern "honeytokens" (fake credentials, fake database records, canary files) extend the same idea to data rather than whole machines.

## ctical Takeaways

- Deploy NIDS at the border and at sensitive internal boundaries, but do not rely on it alone.
- Instrument critical servers with HIDS or at least detailed behavioral logging.
- Keep logs long enough for meaningful forensics, protect them (remote syslog, append-only storage, WORM), and monitor for log tampering.
- Combine signature, behavioral, and (where feasible) specification-based techniques.
- Accept that some attacks will be detected only after the fact; have incident-response playbooks ready.
- Tune aggressively and measure your real FP/FN rates against your actual traffic mix.
- Consider low-interaction honeypots or honeytokens on high-value networks for early warning.
- Remember the base-rate problem: detection at scale is as much an operations and data-science problem as a security problem.
