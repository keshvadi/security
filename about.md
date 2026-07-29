---
title: About
layout: home
permalink: /about/
nav_order: 110
---

# About This Webbook

**Computer Network Security** is an open webbook written for undergraduate students in computer science and software engineering. It assumes limited prior background in computer networking and introduces the minimum networking concepts needed to understand each security topic in context, making the material self-contained.

The book is organized into five units:

1. **Introduction** – Core foundations, the CIA triad, the security mindset, and fundamental design principles.
2. **Cryptography** – Symmetric and public-key primitives, hashes, MACs, and the threat models that underpin modern secure communication.
3. **Network Security** – Practical application of these concepts to network protocols and systems.
4. **Web Security** – Attacks and defenses in the web application layer.
5. **Emerging Topics** – Quantum cryptography, AI agents, cloud security, and the privacy, legal, and ethical responsibilities that accompany modern security practice.

This webbook evolved from lecture notes developed for teaching computer network security and software security courses. Its core structure follows the “Network Systems Security” course taught by [Prof. Joel Reardon](https://cspages.ucalgary.ca/~joel.reardon/) at the University of Calgary. Deep coverage of individual topics draws heavily on Paul C. van Oorschot’s textbook *[Computer Security and the Internet: Tools and Jewels from Malware to Bitcoin](https://people.scs.carleton.ca/~paulv/toolsjewels.html)*. The interactive webbook format was inspired by [David Wagner](https://people.eecs.berkeley.edu/~daw/)’s Computer Security course at UC Berkeley, and the hands-on approach benefits from the capture-the-flag methodologies of the [pwn.college](https://pwn.college/) team.

Many ideas and presentations were also shaped by the work and teaching of researchers including [J. Alex Halderman](https://jhalderm.com/), [Stephen Checkoway](https://checkoway.net/), [Dan Boneh](https://crypto.stanford.edu/~dabo/), [John Mitchell](https://theory.stanford.edu/people/jcm/), [Zakir Durumeric](https://zakird.com), [Hovav Shacham](https://www.cs.utexas.edu/directory/hovav-shacham), [Stefan Savage](https://cseweb.ucsd.edu/~savage/), [Deian Stefan](http://cseweb.ucsd.edu/~dstefan/), [Nadia Heninger](https://cseweb.ucsd.edu/~nadiah), [Michael Bailey](https://mdbailey.ece.illinois.edu/), and [Kirill Levchenko](https://klevchen.ece.illinois.edu/), among others.

### Online Version

The interactive, formatted version of this book is available at:  
[https://keshvadi.github.io/security/](https://keshvadi.github.io/security/)

---

## Source Code & Formatting

This repository has auto-formatting enabled. The preferred way to format source is through Prettier on your local machine. Install Node, run `npm install -g yarn`, then `yarn`. To format code, use:

```bash
yarn prettier
```

This automatically formats all `.md` and `.html` files.

There is also a GitHub Action that can format the source. Go to the **Actions** tab, find the *Auto-Format Source* workflow, and manually trigger a workflow dispatch against the target branch.

A CI check runs Prettier and fails if any formatting errors are detected.

### Running Locally

```bash
bundle exec jekyll serve
```

---

## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
