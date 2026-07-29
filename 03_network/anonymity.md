---
title: Anonymity and Tor
parent: Network Security
nav_order: 16
layout: page
header-includes:
  - \pagenumbering{gobble}
---

# Anonymity and Tor

## Anonymity

Imagine you want to research a sensitive medical condition, read about a controversial political movement, or contact a journalist about something happening on campus, without the university network, your ISP, or the websites you visit being able to build a permanent record that ties the activity back to you personally. You are not trying to hide the _content_ of what you read or say (you may even want it public); you are trying to hide the _fact that it was you_.

This is the goal of **anonymous communication**. It is distinct from _confidentiality_ (encryption), which protects the _contents_ of messages from eavesdroppers. Anonymity protects the _identity_ of the participants.

The Internet Protocol was not designed with anonymity in mind. Every IP packet carries the source IP address in its header in plaintext. That address is a strong identifier: it is usually assigned by your ISP or campus network, it is often stable for hours or days, and it can be mapped to a physical location or even a specific dorm room or apartment with subpoenas or data brokers. Any server you connect to sees your IP immediately. Anyone who can observe the path between you and the server (your Wi-Fi provider, the campus border router, a backbone provider, or a nation-state with taps) can also link you to the destination.

Contrast this with a simple group messaging app scenario: if you want to post a public message without your name attached, you might send it privately to a friend and ask them to post it on your behalf. That uses _indirection_ to hide your identity. The Internet removes this natural indirection.

## Proxies: The Simplest Form of Indirection

Suppose Alice wants to visit `www.example.com` without `example.com` learning her real IP address. She can send her request to a **proxy** server. The proxy fetches the page from `example.com` and forwards the response back to Alice. From `example.com`'s perspective, the request came from the proxy's IP, not Alice's.

This works for the narrow goal of hiding Alice from the destination. It fails for broader anonymity goals:

- The proxy itself now knows _everything_: who Alice is, where she went, and (if the connection to the proxy is unencrypted) what she sent and received.
- If the proxy is compromised, subpoenaed, or malicious, Alice's privacy is completely lost.
- A single proxy is a single point of failure for both availability and trust.

A better design would distribute trust across multiple independent parties so that no one party sees both the client and the destination.

Sanity check: If Alice uses a single proxy and then uses HTTPS (TLS) to `example.com`, does the proxy still learn the _content_ of the pages? Does it still learn that Alice is talking to `example.com`?  
Answer: The proxy does not see the content (TLS is end-to-end between Alice and the server), but it still sees the destination hostname (via the TLS Server Name Indication or by observing the IP the proxy connects to) and definitely knows that Alice is its client. The proxy remains a single point that can log "Alice visited example.com at time T."

## Onion Routing: Layered Encryption Across Multiple Relays

The key insight of **onion routing** is to chain several proxies (called _relays_ or _nodes_) together and to use _layered encryption_ so that each relay only learns what it absolutely must know.

Here is the classic illustration with three relays (the number used by Tor in practice).

Alice wants to send message `M` to Bob while hiding her identity from everyone except the first relay. She chooses three relays at random: Frank (entry), Dan (middle), and Charlie (exit).

She builds the "onion" from the inside out:

1. The innermost layer is what only Charlie (the exit) must know: the real destination and the message.

   - `inner = Enc(K_Charlie, (M, Bob))`
   - _(Note: For true end-to-end security, the message `M` itself should also be encrypted with Bob's public key before being wrapped in the onion, ensuring that even Charlie cannot read the payload)._

2. The next layer is what Dan must know: "send the following blob to Charlie."

   - `middle = Enc(K_Dan, (inner, Charlie))`

3. The outermost layer is what Frank must know: "send the following blob to Dan."
   - `outer = Enc(K_Frank, (middle, Dan))`

Alice sends `outer` to Frank. Frank decrypts one layer with his private key, learns he should forward the result to Dan, and does so. Dan decrypts his layer, learns he should forward to Charlie, and does so. Charlie decrypts the last layer, learns the real destination Bob, and delivers `M` to Bob.

**What each party knows**:

- Frank knows Alice's IP but only that the next hop is Dan. He has no idea the final destination is Bob or what `M` contains.
- Dan knows only that traffic arrived from Frank and should go to Charlie. He sees neither Alice nor Bob nor the content.
- Charlie knows the content (if not encrypted with Bob's key) and the final destination Bob, but sees the previous hop as Dan, not Alice.
- Bob sees only Charlie's IP address.
- No single relay knows both the true source and the true destination.

The scheme generalizes to any number of hops. As long as _at least one_ relay in the path is honest and not colluding with the others, the adversary cannot link Alice to Bob with certainty. In a busy network with thousands of simultaneous connections, even partial collusion is hard to exploit because of the large "crowd."

Tor implements bidirectional onion routing (circuits support traffic in both directions) and uses it for web browsing, SSH, chat, and many other TCP-based applications.

Sanity check: Suppose Frank and Charlie are both malicious and colluding, but Dan is honest. Can they link Alice to Bob?  
Answer: Frank sees Alice and knows the circuit goes to Dan. Charlie sees Dan and knows the destination is Bob. Because they never see each other's view directly and Dan is honest, they cannot confirm that the circuit fragment Frank observed is the same one Charlie observed—unless they use timing or volume correlation across the whole path. In a high-traffic network the correlation is noisy.

## How Tor Actually Works Today

Tor is the most widely used realization of onion routing. Key practical elements:

- Volunteer relays. Anyone can run a relay. There are several thousand relays at any time, run by individuals, universities, companies, and activists around the world. Relays are not all equal: some are configured only as middle relays, some allow exit traffic (the dangerous and legally risky role), and some are _bridges_ (unpublished entry points used to circumvent censorship).

- Directory authorities. A small set of trusted, well-known servers maintain and digitally sign the current list of relays, their IP addresses, public keys, exit policies, and measured bandwidth. Clients download this consensus document and use it to pick relays.

- Circuits, not paths per packet. Tor clients build _circuits_ (encrypted tunnels) that last for ~10 minutes. All TCP streams (web, SSH, etc.) from the client are multiplexed over the same circuit for that period. This reduces setup cost and helps hide the number of distinct destinations.

- Typical circuit length = 3 hops. Entry (guard) → middle → exit. Three is a deliberate engineering compromise: enough hops to provide meaningful protection against single-relay compromise, few enough that latency and throughput remain usable for interactive applications.

- Entry guards. Instead of picking a fresh random entry relay for every circuit, Tor clients pick a small set of _guards_ and stick with them for weeks or months (subject to certain rotation rules). This dramatically reduces the probability that a client will ever choose a malicious relay as its first hop. An adversary who wants to see the client's real IP must either control one of the client's guards or wait a very long time.

- TLS between relays. Every hop-to-hop link is protected by TLS (just like HTTPS). The onion layers ride inside these TLS connections. Fixed-size cells (originally 512 bytes of payload) carry commands and data; this uniformity helps frustrate traffic analysis.

- Exit policies. Every exit relay publishes which destination ports and IP ranges it is willing to connect to. This lets clients avoid relays that cannot reach the service they want and lets relay operators limit their legal exposure.

When you use the Tor Browser, it automatically configures itself as a Tor client, builds circuits, and sends your traffic through them. The destination server sees an exit relay's IP address, not yours.

## Threat Model: What Tor Protects and What It Does Not

Tor was designed against a strong but not omniscient adversary. Understanding the limits is crucial for using it safely.

**Tor protects you against**:

- A destination server learning your real IP address or rough geographic location.
- Your local network administrator or ISP learning which websites or hidden services you are visiting.
- Any single relay (or small set of non-colluding relays) from learning both who you are and where you are going.
- Simple logging by intermediate networks.

**Tor does _not_ protect you against**:

- A powerful global passive adversary who can observe (or compel observation of) both the entry and exit segments of your circuits and then correlate timing, packet sizes, or traffic volume. In the real world this is extremely difficult for most users but feasible for nation-states against high-value targets.
- A malicious or compromised exit relay. If you send unencrypted HTTP to a site, the exit relay sees the full request and response in plaintext. (Always use HTTPS, SSH, etc., _on top of_ Tor.)
- Browser or application fingerprinting. If your browser sends a unique combination of fonts, plugins, screen size, etc., the destination can track you across visits even if your IP changes.
- Endpoint compromise. If malware on your laptop already has your files or keystrokes, Tor cannot help.
- Confirmation attacks when you are the _only_ Tor user doing something distinctive at a particular time.
- The fact that you are using Tor at all (your ISP sees TLS connections to known Tor relay IPs—unless you use bridges).

Sanity check: You are a TRU student using Tor to post an anonymous tip about campus safety to a journalist's SecureDrop site. The journalist's server is reached via an exit relay. The campus border router sees you talking to a Tor guard. Can the campus IT department prove you sent the tip?  
Answer: Not from network traffic alone, provided you used the Tor Browser (or equivalent) correctly, avoided fingerprintable behavior, and the tip itself does not contain identifying information. The campus sees only "this student is using Tor"; the exit relay sees "someone sent this tip via Tor"; neither sees both facts together.

## Onion Services: Anonymous Servers as Well as Clients

So far we have discussed _client_ anonymity: hiding the identity of the person who initiates a connection. Tor also supports **onion services** (formerly called hidden services), which allow the _server_ to be anonymous as well.

An onion service has a long-term public key and is reachable via a `.onion` address derived from that key (e.g., `facebookcorewwwi.onion` was a real Facebook onion address). The service picks several relays to act as _introduction points_ and publishes their descriptors (signed by the service's key) to a distributed hash table.

When a client wants to connect:

1. The client also picks a _rendezvous point_ (a random relay) and tells the onion service (via one of its introduction points) "meet me at this rendezvous relay; here is a one-time cookie."
2. The service connects to the rendezvous point from its own side.
3. The rendezvous point simply forwards data between the two circuits.

Neither the client nor the service ever learns the other's real IP address. The rendezvous point and introduction points learn even less. This design lets whistleblowers, journalists, dissidents, and even ordinary people run web sites, chat servers, or SSH servers without revealing where the server is physically located.

Running an onion service is not free of risk (the content itself may still be illegal or dangerous), but the network location is hidden.

## Performance, Attacks, and Operational Realities

**Performance**. Each additional hop adds latency (propagation + crypto + queuing) and reduces throughput. Tor's three-hop design plus the volunteer nature of relays means that Tor is noticeably slower than a direct connection—sometimes dramatically so during congestion. However, the trade-off is mathematically highly favorable: assuming at least a fraction of the available relays are honest, the security against collusion increases _exponentially_ with each added hop, while the performance penalty (latency) increases only _linearly_.

**Attacks that have been demonstrated or theorized**:

- Entry-exit correlation (the fundamental traffic-analysis attack). A perfect defense against timing correlation requires padding all messages (which Tor does by using fixed-size cells) _and_ introducing significant, randomized delays at each proxy. Because Tor is designed for low-latency interactive applications (like web browsing), it deliberately avoids injecting long delays, leaving it vulnerable to timing correlation by a global observer.
- Sybil attacks (an adversary runs many relays hoping to be chosen for many circuits).
- Bad exit attacks (malicious exits injecting or monitoring traffic).
- Congestion and latency attacks that fingerprint circuits.
- Application-level leaks (DNS queries sent outside Tor, WebRTC, etc.—the Tor Browser tries hard to close these).

Tor's developers and researchers have added many mitigations over the years: entry guards, better path selection, padding in some cases, improved congestion control, and active measurement of relays. Perfect protection against a global adversary remains information-theoretically difficult for low-latency systems; Tor's goal is to raise the bar high enough that only the most powerful adversaries can succeed, and even then only against a small number of high-value targets.

**Censorship circumvention**. Because Tor is effective at hiding destinations, censors often try to block Tor itself. The Tor Project responds with _bridges_ (unpublished relays), _pluggable transports_ (obfs4, Snowflake, etc.) that disguise Tor traffic as random or as other protocols (WebRTC, HTTPS, etc.), and meek (domain fronting). These are essential tools for users in repressive regimes.

## Using Tor Safely

If you decide to use Tor on campus or at home:

- Use the official Tor Browser (or Tails live USB for stronger isolation). Do not try to "Tor-ify" a normal browser yourself.
- Keep the Tor Browser updated.
- Prefer onion services when they exist (they give you end-to-end anonymity for both sides).
- Always use HTTPS (or SSH, etc.) on top of Tor for any sensitive session.
- Do not log into personal accounts or mix Tor and non-Tor browsing in ways that link your identities.
- Be aware that some services (banks, Google, Cloudflare) treat Tor exit IPs as suspicious and may present CAPTCHAs or blocks.
- Understand that "using Tor" is itself a signal in some environments; bridges and pluggable transports reduce that signal.

Anonymity is fragile. One mistake, enabling JavaScript that phones home, pasting a document with metadata, or using a unique browser configuration, can deanonymize you even if the network layer is perfect.
