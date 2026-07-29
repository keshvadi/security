---
title: Secure Shell (SSH)
parent: Network Security
nav_order: 17
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Secure Shell (SSH)

## Remote Access

The fundamental idea of remote access is to use a local keyboard and monitor to control another computer across a network. This is essential for managing servers, headless machines, and routers.

In the early days of the Internet, remote administration was accomplished using **Telnet** (designed in 1969) or the Berkeley "r-commands" suite introduced in 1982 (`rlogin`, `rcp`, and `rexec`). **These protocols provided absolutely no security.** Usernames, passwords, and every keystroke were transmitted across the network in plaintext. Anyone sitting on the same network path could effortlessly observe the traffic, capture credentials, and read the server's output.

In 1995, Tatu Ylönen witnessed a password-sniffing attack in his university network. An eavesdropper gathered logins and passwords by looking at the traffic. Ylönen read about cryptography and invented SSH (Secure Shell) as a replacement. It quickly became the standard way to log into remote machines.

Today, SSH provides much more than just a plain remote terminal. It establishes a general-purpose secure channel that other tools rely on. When you push code to GitHub, mount a remote directory using `sshfs`, or securely copy a file with `scp`, you are utilizing the secure transport layer established by SSH.

## The Original SSH-1 Protocol

The first version of SSH used a clever but ultimately flawed design. It is still worth studying because it provides a foundational understanding of historical key exchange and authentication mechanisms.

When a client initiated an SSH-1 connection, the following sequence occurred:

1. The client contacts the server ("host") and provides its SSH protocol version and implementation version.
2. The server replies with:
   - Its public RSA "host-key" (1024 bits), which is permanent and stored in a config file.
   - A random RSA "server-key" (768 bits) that is changed hourly and never saved to a file.
   - Eight random bytes.
   - A list of supported ciphers.
3. Both the client and server compute a session identifier using an MD5 hash of the host key, server key, and random bytes.
4. The client checks a local cache of host keys.
   - If the host is not in the cache, it shows the key to the user and asks to add it (the connection may abort here).
   - If the host is in the cache and the host key matches, all is good.
   - If the host is in the cache but the key is different, it warns the user that the key has changed and that someone could be doing something nasty (the connection may abort here).
5. The client randomly generates a session key based on supported ciphers offered by the server. It encrypts this key with the server key and the host key, then sends it to the server. Double encryption forced the attacker to break both keys to read recorded sessions.
6. The client and server now have a secure channel.
7. The client now has to authenticate to the server, supporting Kerberos, password, or public key.

**Client Authentication in SSH-1**:

- Kerberos: The user gets a ticket to log into the system, treating the SSH server just like a printer or other service.
- Password: The client types their password into the terminal, sends it to the server over the encrypted channel, and the server checks that it is the password for that user on this machine.
- Public Key: The client sends the server the public key they want to use. The server checks if that key is authorized for that user in `/home/user/.ssh/authorized_keys`; if it is not listed in that file, it rejects it. If authorized, the server challenges the client by generating a random 256-bit string, encrypting it with the client's public key, and sending it. The client decrypts it with their private key, combines the challenge with the session identifier, hashes the result with MD5, and sends it back to answer the challenge.

## SSH-2: The Modern Protocol

As cryptographic standards evolved, the flaws in SSH-1 became apparent (most notably, its lack of strong message integrity and reliance on MD5).
**SSH-2** is a complete, incompatible rewrite of the protocol designed to cleanly separate the connection, transport, and authentication layers.

SSH-2 is what almost every system runs today. Its major improvements include:

- Diffie-Hellman Key Exchange: Replaced the hourly RSA "Server Key" with a proper Diffie-Hellman negotiation, providing Perfect Forward Secrecy (PFS).
- Stronger Primitives: Added support for modern symmetric ciphers like AES.
- HMAC for Integrity: Addressed SSH-1's lack of real integrity by using Hash-based Message Authentication Codes (HMAC) to ensure packets cannot be tampered with.
- Rekeying Support: Allows the session keys to be periodically renegotiated during long-lived connections, limiting the amount of ciphertext an adversary can analyze under a single key.
- Directional Keys: Uses different cryptographic keys for sending, receiving, and integrity checking.
- Improved Authentication: Dropped the outdated Kerberos support and upgraded the public-key challenge so that the client signs the session identifier along with other connection metadata.

## Public-Key Authentication

A key difference between SSH and protocols like TLS is that the SSH client is _always_ authenticated (either by password or by key).

Public-key authentication is the preferred method for logging into SSH servers. It allows you to authenticate without transmitting a password, and it enables automated systems (like Git or continuous integration pipelines) to securely access restricted resources.

**How it works**:
Your machine holds a private key (stored on disk and usually protected by a local passphrase). You copy the matching public key to the remote server, placing it in a specific file: `~/.ssh/authorized_keys`.

When you attempt to log in, the server generates a random challenge. You use your local private key to digitally sign this challenge and send it back. The server verifies the signature using the public key listed in `authorized_keys`.
Your private key never leaves your local machine, and no private material is ever stored on the server.

## Advanced SSH Features and Tunneling

SSH is much more than a remote shell. It is a Swiss Army knife for networking.

### Port Forwarding (Tunneling)

SSH can encapsulate and securely forward other network protocols over its encrypted channel.

- Local Port Forwarding (`-L`): Forwards a port on your local machine to a destination reachable by the remote server.
  _Syntax_: `ssh -L 8080:db.internal.tru.ca:5432 user@bastion-host`
  _Result_: Any traffic you send to `localhost:8080` on your laptop is encrypted, sent to the bastion host, and forwarded to the internal PostgreSQL database.
- Remote Port Forwarding (`-R`): Does the exact reverse, exposing a port from your local machine to the remote server.
- Dynamic Port Forwarding (`-D`): Creates a local SOCKS proxy.
  _Syntax_: `ssh -D 1080 user@remote-server`
  _Result_: You can configure your web browser to route all traffic through `localhost:1080`. Your traffic will exit onto the Internet from the remote server, acting as a lightweight VPN.

### Agent Forwarding and Jump Hosts

If you use a private key to log into Server A, and from Server A you need to log into Server B, you would traditionally need to copy your private key to Server A. This is a severe security risk.

- ssh-agent: A background program that holds your decrypted private keys in local memory so you do not have to type your passphrase for every connection.
- Agent Forwarding (`-A`): `ssh -A` forwards your local `ssh-agent` socket to the remote machine. This allows you to hop to Server B without copying your keys to Server A.

**The Danger of Agent Forwarding** If Server A is compromised, an attacker with root privileges can hijack your forwarded agent socket and authenticate _as you_ to any other machine that accepts your keys. You should only use `-A` with machines you completely trust.

**The Modern Alternative (`-J`)** The `ssh -J` (Jump host) flag is much safer. It instructs SSH to establish a secure tunnel _through_ Server A directly to Server B, utilizing your local keys without ever exposing an agent socket to Server A.

### Configuration and File Transfer

- X11 Forwarding (`-X` or `-Y`): Allows you to run graphical GUI applications on a remote Linux compute server and have the application windows render seamlessly on your local desktop.
- Configuration Files: Rather than typing long commands, you can define shortcuts, custom ports, forced ciphers, and usernames in `~/.ssh/config`. Server-side security policies (such as disabling password authentication entirely) are managed in `/etc/ssh/sshd_config`.
- File Transfer: While `scp` and `sftp` are native SSH tools for moving files, modern workflows often rely on `rsync -e ssh` for efficient delta-transfers or `git` over SSH for version control.
