---
title: Security Principles
parent: Introduction
nav_order: 2
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Security Principles

Perfect security is unattainable. Adversaries adapt, systems grow more complex, and the environments in which software runs continually change. As a result, building secure systems requires a different approach from many other engineering disciplines. In computer security, we rely on a set of design principles that have proven effective over decades of practice.

These principles were first articulated in a systematic way by Jerome Saltzer and Michael Schroeder in the 1970s and have been refined by subsequent research and real-world experience. They are not rigid rules or a checklist that guarantees security. Rather, they are practical guidelines that help designers reason clearly about trade-offs, assumptions, and attacker capabilities.
Systems that ignore them are repeatedly compromised, while systems that apply them carefully tend to fail less often and fail more gracefully when they do.

Throughout this book, these principles serve as a primary analytical framework. When we examine an architecture, evaluate a protocol, or design a defense, we will repeatedly return to them to guide our reasoning and to identify weaknesses.

## 1. Economy of Mechanism

_(also known as Simplicity-and-Necessity, Small Trusted Base)_

Keep the design of security mechanisms as simple and small as possible. Complexity is the enemy of security: every additional feature, interface, or line of code increases the chance of errors, makes thorough analysis harder, and expands the attack surface.

This principle has two closely related aspects. First, minimize functionality. Disable unused features by default, prefer minimal installations, and avoid adding convenience features that are not essential. Second, keep the portion of the system that must be trusted, i.e., the Trusted Computing Base (TCB), as small as possible. The TCB consists of those components that must operate correctly for the system’s security goals to hold. Anything outside the TCB should be unable to compromise security even if it is buggy or malicious.

A smaller TCB is easier to write, audit, and reason about. Industry experience shows that defect rates typically fall in the range of one to five defects per thousand lines of code. Reducing the size of the trusted code therefore directly reduces the expected number of security-critical bugs. In practice this means moving as much functionality as possible out of privileged components, using strong isolation so that untrusted code cannot interfere with trusted code, and resisting the temptation to add “just one more” feature to a security-critical module.

When evaluating a design, ask: Can this component be removed from the TCB? Can this interface be narrowed or eliminated? Can this privilege be dropped after it is no longer needed? The goal is not minimalism for its own sake, but a deliberate reduction of the mechanisms that an attacker can target and that defenders must get completely right.

## 2. Fail-safe Defaults

_(also known as Safe-Defaults)_

Base access decisions on permission rather than exclusion, and choose default settings that remain secure even when users or administrators do not change them. Most people leave configuration options at their default values. Therefore the default must be the safe choice.

In access control this principle means deny-by-default: nothing is allowed unless it has been explicitly permitted. Prefer allow-lists (also called goodlists or whitelists) over deny-lists (badlists or blacklists). An allow-list states exactly who or what is permitted and rejects everything else. A deny-list attempts to enumerate what is forbidden; any omission becomes a vulnerability. Legitimate users who are incorrectly denied access will usually complain and can be added to the allow-list. Illegitimate users who are incorrectly granted access will remain silent.

The same idea applies to system behavior under failure. Security mechanisms should fail closed (deny access) rather than fail open. A firewall that crashes should stop forwarding packets, not start allowing everything.

Classic illustrations include network firewalls that block all traffic by default and only open ports that have been deliberately enabled, and modern systems that enable encryption or HTTPS by default rather than requiring users to turn these protections on. When a designer must choose a default, the question should be: “If this setting is never changed, will the system still be reasonably secure?” If the answer is no, the default is wrong.

## 3. Complete Mediation

Every access to every object must be checked for authority. The check should occur as close as possible to the moment the access is performed, not merely at the beginning of a session or at the time a reference is first obtained.

It is not enough to authenticate a user once and then allow unrestricted access thereafter. Privileges may be revoked, the user’s role may change, or an attacker may have compromised part of the system after the initial check. Therefore authorization must be re-validated on each use. In operating-system terms, this is the role of a reference monitor: a trusted component that intercepts every attempt to access a protected resource and decides whether the access should be allowed.

A common violation of this principle occurs when an access decision is made early and then cached or reused later without re-checking. This creates time-of-check-to-time-of-use (TOCTTOU) vulnerabilities, in which the conditions that justified the original decision no longer hold at the moment the resource is actually used.

Another frequent mistake is performing mediation only on the client side. Client-side checks can improve usability and reduce unnecessary server load, but they provide no security. A malicious client can simply bypass them. All security-relevant mediation must be enforced by a trustworthy component that the attacker cannot control.

When reviewing a design, ask of every operation that touches a sensitive resource: “Where exactly is the authority check performed, and can that check be bypassed, skipped, or reused after it is no longer valid?” If the answer is unclear, complete mediation has not been achieved.

## 4. Open Design

_(also known as Shannon’s Maxim, Kerckhoffs’ Principle)_

Do not rely on the secrecy of the design itself for security. The security of a system should rest on the secrecy of a small amount of information (typically cryptographic keys) rather than on the attacker remaining ignorant of how the system works.

This idea is often summarized by Kerckhoffs’ principle: a cryptographic system should remain secure even if everything about the system, except the key, is public knowledge.

Claude Shannon later expressed the same thought more bluntly: “the enemy knows the system”. History has repeatedly shown that security through obscurity is brittle. Designs, algorithms, and even source code that were once secret eventually become known through reverse engineering, leaks, insider disclosure, or simple determined analysis. Systems that depended on that secrecy have usually failed once the design was exposed.

Open design does not require that every detail of a system be published. There is rarely any advantage in advertising the exact floor plan of a building or the precise schedule of security patrols. The principle does require that the fundamental mechanisms be able to withstand scrutiny. Prefer algorithms and protocols that have been widely reviewed over proprietary constructions whose strength depends on remaining hidden. When a secret must be kept, make sure it is something that can be changed easily (a key) rather than something that is expensive or impossible to change (the entire design).

When evaluating a system, ask: “If an attacker obtained a complete description of the design tomorrow, would the system still meet its security goals?” If the answer is no, the design violates the principle of open design.

## 5. Separation of Privilege

_(also known as Separation of responsibility, Isolated-Compartments, Modular-Design)_

Require more than one independent condition, key, or party before a critical action is authorized. No single person or component should be able to complete a sensitive operation on its own.

This principle limits the damage that can be caused by a single failure, error, or act of malice. If two independent officers must turn their keys simultaneously to launch a missile, a single traitor or coerced individual cannot succeed. If two different employees must approve a large financial transfer, collusion becomes necessary. In software systems the same idea appears as modular design and strong isolation: privileges are divided among distinct components so that the compromise of one component does not automatically grant all the privileges needed for a serious attack.

Separation of privilege is closely related to the notions of isolation and compartmentalization. Components should interact only through well-defined, narrow interfaces. The more strongly the system is divided into independent compartments, the harder it becomes for an attacker who has gained a foothold in one part of the system to expand control to the rest.

When reviewing a design, identify the most sensitive operations and ask: “Can a single compromised person, process, or component perform this action alone?” If the answer is yes, consider whether the operation can be split so that two or more independent authorizations are required.

## 6. Least Privilege

Grant every user, process, and component only the privileges it actually needs to perform its legitimate function, and grant those privileges only for the minimum time necessary.

The goal is to reduce the potential damage that can result from error, compromise, or malice. If a program is exploited, the attacker inherits only the privileges that program possessed. The fewer those privileges, the smaller the harm. A text editor that needs to edit a single file should not receive the power to read, modify, or delete every file belonging to the user. A web server that only needs to bind to port 80 and read files from a specific directory should not run as root for its entire lifetime.

In practice this means starting with minimal privileges and adding rights only when required, then dropping them again as soon as they are no longer needed. It also means designing systems so that high privileges are not inherited by default. Many traditional operating systems violate the principle by giving every process the full rights of the user who launched it. Modern mobile platforms do better by placing each application in its own sandbox with tightly restricted permissions.

When evaluating a design, examine each component and ask two questions: “Does this component really need every privilege it currently holds?” and “Can any of those privileges be dropped after initialization or after a specific task is completed?” The more often the answer leads to a reduction in privilege, the closer the system comes to satisfying the principle of least privilege.

## 7. Least Common Mechanism

Minimize the mechanisms that are common to more than one user or process and on which all of them depend. Every shared component is a potential channel through which one party can influence or interfere with another.

When multiple users or security domains rely on the same piece of code, data, or system service, that shared element becomes part of the trusted computing base for all of them. A vulnerability or malicious modification in the shared mechanism can compromise every party that depends on it. Shared state can also create subtle side channels or unintended communication paths that bypass normal access controls.

Classic examples include shared variables, world-writable files, global configuration databases, and common system utilities that run with elevated privilege. In modern systems the same concern appears with shared libraries, inter-process communication facilities, and cloud services that multiplex many customers on the same infrastructure. The more a mechanism is shared, the higher the cost of any flaw it contains and the harder it becomes to reason about the isolation between users.

The practical guidance is to reduce sharing wherever possible. Prefer per-user or per-process instances over a single global instance. When sharing cannot be avoided, make the shared interface as narrow and carefully mediated as possible, and treat the shared component as part of the TCB. When reviewing a design, look for resources that many different parties depend on and ask whether that sharing is truly necessary or whether it can be eliminated or more strongly isolated.

## 8. Psychological Acceptability

_(also known as Least-Surprise, User-Buy-In, Consider Human Factors)_

Design security mechanisms so that they are easy for people to use correctly and hard to use incorrectly. The human interface must match the mental models that ordinary users already possess. When a protection mechanism is too awkward, slow, or confusing, people will find ways to circumvent it, often defeating the very security it was intended to provide.

This principle has two complementary sides.
The first is “least surprise”: the system should behave as users expect. Interfaces that violate expectations produce mistakes, especially under time pressure or when the consequences of an error are irreversible.
The second is “user buy-in”: the secure way of doing a task should also be the easiest way. If the secure option requires extra steps, users will choose the convenient but insecure alternative.

Classic failures include password rules so complex that users write the passwords on sticky notes, security warnings that appear so frequently they are ignored, and update prompts that interrupt work and are therefore postponed indefinitely. Positive examples include physical keys shaped like ordinary door keys for cryptographic tokens (so that the correct action is intuitively obvious) and single-sign-on systems that reduce the number of passwords a person must manage.

Security mechanisms are ultimately operated by people. A theoretically perfect control that is routinely bypassed in practice provides no security. When evaluating a design, ask: “Will ordinary users understand this? Will they find it easier to do the right thing than the wrong thing?” If the answer is no, the mechanism violates the principle of psychological acceptability.

## 9. Time-Tested Tools

Prefer security mechanisms, protocols, cryptographic primitives, and libraries that have already withstood extensive public scrutiny and real-world use. Designing new security mechanisms is difficult even for experts; the probability of introducing subtle but fatal flaws is high.

This principle is often summarized by the maxim “don’t roll your own crypto”. History is filled with examples of proprietary or newly invented encryption schemes, authentication protocols, and random-number generators that looked strong to their designers yet collapsed once exposed to serious analysis. In contrast, algorithms and protocols that have survived years of examination by the global research community (AES, TLS 1.3, established libraries such as OpenSSL or libsodium, well-reviewed authentication frameworks, etc.) offer far higher confidence.

The underlying reasoning is statistical as much as technical. A widely used, heavily reviewed mechanism is far more likely to have had its weaknesses discovered and corrected than a one-off construction that has received little external attention. The longer a tool has been “soaking” under real attack and analysis, the more trustworthy it generally becomes.

The principle does not forbid research or experimentation. Building a new cryptographic protocol is an excellent way to learn. It does, however, strongly discourage deploying home-grown security mechanisms in production systems when mature, reviewed alternatives already exist. When a new mechanism is unavoidable, it should be subjected to the same open, prolonged scrutiny that older tools have already endured.

When evaluating a design, ask: “Is this security-critical component something we invented ourselves, or is it a well-studied tool that has already survived years of attack and review?” Whenever the former is true, treat the decision with extreme caution.

## 10. Evidence Production

Record security-relevant system activities so that it is possible to determine after the fact what happened, who was responsible, and what the effects were. Good evidence supports accountability, intrusion detection, forensic analysis, and recovery.

In the physical world we rely on witnesses, paper trails, and human memory. In computer systems these natural sources of evidence are usually absent. Therefore the system itself must be designed to generate reliable records: authentication events, privilege escalations, access to sensitive files, configuration changes, installation of software, insertion of removable media, and other actions that could be part of an attack or policy violation. Logs should include enough detail (who, what, when, where, and often from where) to reconstruct the sequence of events.

Evidence production also includes deliberate deceptions that reveal unauthorized activity. Honey files, honey tokens, and canary values are resources that legitimate users should never touch; any access to them is strong evidence of malicious or exploratory behavior.

The mere existence of logs is not enough. The logs themselves must be protected from tampering or deletion by an attacker who has gained elevated privileges. Common techniques include writing logs to append-only storage, sending them immediately to a remote system that the attacker does not control, or using cryptographic techniques to detect modification.

When reviewing a design, ask: “If a serious incident occurred tonight, would we have enough trustworthy evidence to understand what happened and to support a proper response?” If the answer is no, the system is deficient in evidence production.

## 11. Detect if You Can’t Prevent

When it is impossible or impractical to prevent an attack, design the system so that the attack will at least be detected. Detection without a subsequent response is of limited value, so the ability to detect must be paired with plans for response and recovery.

Perfect prevention is rarely achievable. Some attacks are too expensive to block completely, some physical or social avenues cannot be fully closed, and some residual risk must simply be accepted. In these situations the next best objective is timely detection. A system that cannot stop an attacker should still generate clear evidence that an attack has occurred, so that operators can respond, contain the damage, and restore the system to a secure state.

A classic illustration appears in hardware security modules. The highest FIPS 140 level requires strong tamper resistance, attacks should be extremely difficult to carry out. Lower levels require only tamper evidence: if someone opens the device, a seal is broken or a visible indication is left behind. The cheaper, more widely deployable devices do not prevent every physical attack, but they make the attack detectable.

The same reasoning applies to software and networks. Not every intrusion can be blocked at the perimeter. Logging, intrusion-detection systems, file-integrity monitoring, and anomaly detection exist precisely so that successful or attempted attacks do not remain invisible. Once detection occurs, predefined response procedures (isolation, credential revocation, recovery from clean backups, etc.) become possible.

When reviewing a design, identify the attacks that cannot be reliably prevented and ask: “If this attack succeeded, would we know about it quickly, and do we have a workable plan for responding?” If either answer is no, the principle has not been satisfied.

## 12. Remnant Removal

When sensitive data is no longer needed, remove it completely so that it cannot be recovered by an attacker. Residual copies left in memory, on disk, in caches, or in log files often become the easiest path to compromise.

Ordinary deletion is rarely sufficient. Most file systems simply mark directory entries as free; the underlying data blocks remain until they are overwritten. Memory that once held cryptographic keys, passwords, or decrypted documents may retain those values long after the program believes it has finished with them. Swap files, hibernation images, crash dumps, and cloud snapshots can preserve sensitive material indefinitely. Attackers routinely search for exactly these remnants.

The principle therefore requires deliberate action: overwrite keys and passwords in memory as soon as they are no longer required, use secure-deletion tools or encryption-with-key-disposal for files, disable or encrypt swap space when appropriate, and ensure that crash dumps and temporary files do not become unexpected archives of secrets. The same discipline applies to long-term logs: retain what is needed for accountability and incident response, but sanitize or avoid recording material that would itself become a valuable target.

Remnant removal must be balanced with the earlier principle of evidence production. The goal is not to erase everything, but to ensure that data that has outlived its legitimate purpose does not remain available to an adversary. When reviewing a system, ask of every location that may hold sensitive information: “How long does this data live, and what concrete steps guarantee its secure removal once that lifetime ends?”

## 13. Reluctant Allocation

Be reluctant to allocate resources, expend effort, or extend privileges, especially when dealing with unauthenticated or external parties. Place the burden of proof on the party that initiates an interaction.

Systems and people have finite resources. An attacker who can force the defender to perform expensive work, open new connections, create accounts, grant temporary privileges, or even just engage in prolonged conversation has already gained an advantage. The principle of reluctant allocation therefore counsels a conservative posture: do not offer services, consume CPU, memory, or bandwidth, or escalate privileges until the requester has been adequately authenticated and authorized.

In human protocols the same idea appears as a simple rule of caution. The person who places a telephone call should not be the one demanding that the recipient prove their identity. If someone claims to be calling from your bank, the safe response is to hang up and place a new call to a number you already know belongs to the bank. In software the principle manifests as rejecting unauthenticated requests early, requiring proof of work or other inexpensive client-side effort before performing expensive server-side operations, and refusing to act as a confused deputy that blindly extends its own privileges to an unverified third party.

When evaluating a design, ask: “What work or privilege does this system grant to a party that has not yet proven its identity or authorization? Can that allocation be delayed, reduced, or made conditional on stronger evidence?” The more reluctant the system is to spend its resources on untrusted requesters, the more resilient it becomes to both denial-of-service and social-engineering style attacks.

## 14. Security by Design

_(also known as Design security in from the start, Design-for-Evolution)_

Security must be treated as a primary design objective from the earliest stages of a system’s development, not as a feature to be added after the architecture, interfaces, and implementation are already fixed.

Once fundamental design decisions have been made (e.g., how privileges are divided, how components communicate, what is trusted, what data formats are used) it becomes extremely difficult to retrofit strong security. Later attempts usually result in incomplete protections, awkward work-arounds, or the forced preservation of insecure legacy behavior for the sake of compatibility. The most effective moment to apply the other principles in this chapter (least privilege, complete mediation, economy of mechanism, etc.) is while the architecture is still fluid.

Two practical disciplines follow from this principle. First, explicitly record the security goals of the system, the threats it is intended to address, and, equally important, the threats or properties it is _not_ designed to handle. Second, state every significant trust assumption (what is assumed about users, administrators, external services, physical environment, etc.). Assumptions that remain implicit are rarely examined and frequently turn out to be false.

A closely related concern is design for evolution. Threats, algorithms, and deployment environments change. Systems should be built so that cryptographic algorithms can be replaced (algorithm agility), software can be updated securely, and security policies can be revised without requiring a complete redesign. A system that cannot evolve eventually becomes insecure even if its original design was sound.

When reviewing a project, ask: “Were the security requirements and trust assumptions written down before the major architectural decisions were locked in? Can the system adapt when a cryptographic primitive is broken or a new class of attack appears?” If the answers are negative, security was not truly designed in from the start.

## 15. Security is Economics

Security is not an absolute property that can be purchased once and for all. It is a continuous cost-benefit trade-off. Resources spent on protection should be proportional to the value of what is being protected and to the expected cost of the attacks that must be resisted.

No practical system can be made invulnerable to every possible attack. Stronger defenses almost always cost more—in money, performance, usability, or development time. The rational goal is therefore not perfect security, but security that is “good enough” relative to the threat and the asset. There is little point in installing a high-end safe that costs more than the contents it is meant to protect. Conversely, under-investing in the protection of high-value assets is equally irrational.

A direct corollary is that effort should be concentrated on the weakest links. Attackers follow the path of least resistance. Strengthening an already strong component while leaving an obvious weak point unaddressed yields little improvement in overall security. The same reasoning supports the idea of conservative design: evaluate a system under assumptions favorable to the attacker and consider any plausible failure mode. Yet even this caution must be tempered by economics; not every theoretically possible attack is worth defending against.

When making design or investment decisions, ask two questions: “What is the value of the asset or the expected loss if this attack succeeds?” and “Does the cost of the proposed defense stand in reasonable proportion to that loss?” Security mechanisms that fail this simple economic test are unlikely to be sustainable or effective in the long run.

## 16. Defence in Depth

_(also known as Defense-in-Depth)_

Do not rely on any single defensive mechanism. Layer multiple, independent defenses so that an attacker must defeat several of them in succession to achieve a successful compromise.

A single control, no matter how strong, will eventually fail through a previously unknown vulnerability, a configuration error, an insider action, or simple operational fatigue. When defenses are stacked, the failure of one layer does not immediately result in total compromise. The attacker is forced to expend additional time, skill, and resources on each subsequent layer, increasing the likelihood of detection and response.

Classic physical analogies include a medieval castle that combines a moat, high walls, a drawbridge, and inner keeps, or a modern building that uses perimeter fencing, locked doors, access badges, cameras, and security guards. In computer systems the same idea appears as the combination of network firewalls, host-based hardening, application sandboxing, least-privilege execution, encryption of data at rest and in transit, and continuous monitoring. Each layer is imperfect; together they raise the cost and complexity of an attack.

Two cautions are important. First, the layers should be independent; a single flaw that simultaneously defeats several layers provides little real depth. Second, diminishing returns apply. Beyond a certain point the marginal security gained by an additional layer may not justify its cost in complexity, performance, or usability (see Security is Economics). The goal is thoughtful layering, not the indiscriminate accumulation of every possible control.

When evaluating a design, identify the critical assets and ask: “If the outermost defense is completely bypassed, what remaining controls still stand in the attacker’s way?” The more independent and substantial those remaining controls are, the better the system satisfies the principle of defence in depth.

## 17. Know Your Adversary

_(also known as Know your threat model)_

Explicitly identify who might attack the system, why they would do so, and what capabilities they are likely to possess. Security mechanisms that are not grounded in a realistic threat model are either insufficient or wastefully excessive.

Different adversaries have different goals, resources, and levels of sophistication. A teenager seeking amusement, a criminal seeking financial gain, a malicious insider, a competitor engaging in industrial espionage, and a well-funded nation-state intelligence service all pose distinct threats. Defenses appropriate for one may be irrelevant or inadequate for another. Therefore the first step in any serious security analysis is to state the threat model: the class of attackers under consideration and the assumptions made about their knowledge, access, and persistence.

Several working assumptions are commonly adopted for capable adversaries:

- The attacker can interact with the system without immediate detection.
- The attacker knows the system’s design, algorithms, and common implementation weaknesses (consistent with the principle of open design).
- The attacker is persistent and willing to try many times; an attack that succeeds only rarely may still be practical.
- The attacker can combine multiple techniques and target several components in a coordinated fashion.
- Almost any networked device can become an entry point.

Threat models are not static. Assumptions that were reasonable when a system was first designed may become false as technology, deployment scale, or the attacker population changes. Early Internet protocols, for example, were created in an environment of relative mutual trust among a small academic community; those same protocols later proved fragile when the Internet became a global network containing many malicious actors.

When evaluating or designing a system, ask: “Who are we defending against, what do they want, and what can they realistically do?” A clear answer to that question turns the other principles in this chapter from abstract guidelines into concrete, prioritised design decisions. Without it, security work risks becoming both unfocused and ineffective.
