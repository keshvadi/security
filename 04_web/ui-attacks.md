---
title: UI Attacks (Clickjacking and Phishing)
parent: Web Security
nav_order: 7
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# UI Attacks: Clickjacking and Phishing

## Introduction to UI Attacks

Most of the attacks we have studied so far (XSS, CSRF, SQL Injection, etc.) exploit technical vulnerabilities in code, browsers, or servers. However, there is another important class of attacks that instead exploits how users perceive and interact with web interfaces.

These are called **UI Attacks** (also known as **Interface Attacks** or **Visual Deception Attacks**).

**UI Attacks** trick users into performing actions they did not intend by manipulating what they _see_ on the screen. The attacker does not necessarily need to break into the server or steal data directly. Instead, they make the user _do something harmful themselves_, such as clicking a button, entering credentials, or authorizing an action, while believing they are doing something completely different.

UI attacks are powerful because they target the human user, who is often the weakest link in the security chain. They work well because:

- Humans tend to trust familiar-looking interfaces (browser chrome, login pages, buttons).
- Most users do not carefully inspect URLs or security indicators.
- Visual deception can be very convincing when done well.
- These attacks often bypass technical defenses that only protect against code-based attacks.

### Examples of UI Attacks

Two of the most important UI attacks are:

| Attack           | Main Goal                                        | Key Technique                                                |
| ---------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **Clickjacking** | Make the user click something they didn’t intend | Transparent iframes, hidden elements, fake cursors           |
| **Phishing**     | Steal credentials or deliver malware             | Fake websites, homograph domains, browser-in-browser attacks |

Both attacks rely on _visual deception_ rather than breaking cryptographic protections or finding code bugs.

## Clickjacking Attacks

**Clickjacking** (also known as **UI Redressing**) is an attack in which the attacker tricks the victim into clicking on an element that is different from what the victim believes they are clicking.

The goal is usually to make the user perform an action they did not intend, such as clicking “Delete”, “Transfer Money”, “Allow Camera”, or “Like/Share”, while thinking they are interacting with something else.

Clickjacking attacks typically use one or more of the following techniques:

**1. Transparent or Overlapping Iframes**  
The attacker loads the legitimate target website (e.g., a bank’s transfer page or a social media “Like” button) inside an invisible or semi-transparent `<iframe>`. They then place a fake, visible button or link on top of it. When the user clicks what they think is a harmless button on the attacker’s page, they are actually clicking on the hidden legitimate page.

**2. Hidden or Covered Elements**  
Using CSS tricks (`opacity: 0`, `z-index` manipulation, or precise positioning), the attacker can hide important buttons or make them appear in unexpected places. The user clicks on what looks like normal content but actually triggers a sensitive action on another site.

**3. Fake Cursor Attacks**  
The attacker draws a fake mouse cursor on the page using HTML and JavaScript. The user thinks they are clicking in one location, but their real cursor is clicking somewhere else, often on a hidden iframe containing a sensitive action.

**4. Full Browser-in-Browser Attacks**  
A more advanced and convincing technique where the attacker creates a complete fake browser window (including address bar, tabs, security indicators, and browser chrome) using HTML, CSS, and JavaScript. The victim believes they are interacting with their real browser and a legitimate website, but everything is controlled by the attacker.

## Clickjacking Defenses

There are several effective ways to defend against clickjacking. The strongest defenses are implemented on the server side using HTTP headers.

### 1. Server-Side Frame Controls (Recommended)

These are the most reliable defenses because they are enforced by the browser before any JavaScript runs.

**A. `X-Frame-Options` Header**

This older but still widely supported header tells the browser whether a page is allowed to be embedded in an iframe.

```http
X-Frame-Options: DENY           # Completely prevents the page from being framed
X-Frame-Options: SAMEORIGIN     # Only allows framing from the same origin
```

**B. Content Security Policy – `frame-ancestors` (Modern & Preferred)**

The modern and more flexible way to control framing is using the `frame-ancestors` directive in a Content Security Policy.

```http
Content-Security-Policy: frame-ancestors 'none';                    # Strongest protection
Content-Security-Policy: frame-ancestors 'self';                    # Only same origin
Content-Security-Policy: frame-ancestors https://trusted.example.com;
```

The recommendation is to use `frame-ancestors 'none'` on any page that contains sensitive actions (bank transfers, account settings, admin panels, payment pages) and use `'self'` for most other pages that should not be embedded by third parties.

### 2. Client-Side JavaScript Frame Busting (Legacy)

Older websites sometimes used JavaScript to prevent framing. For example:

```js
if (top !== self) {
  top.location = self.location;
}
```

This code checks whether the current page is being loaded inside an iframe (by comparing top and self). If it detects that it is being framed, it redirects the top-level window to the current page's location, effectively attempting to 'bust' out of the attacker's iframe.

However, this technique is unreliable and can be bypassed by attackers using sandboxed iframes, `X-Frame-Options`, or clever timing attacks. It should only be used as a defense-in-depth measure, not as the primary protection.

### 3. User Experience and Design Defenses

In addition to technical controls, you can make clickjacking harder through good interface design:

- **Confirmation dialogs** for dangerous actions (“Are you sure you want to transfer $5,000?”)
- **UI randomization** which randomly change the position or appearance of critical buttons
- **Attention-directing techniques** which highlight the area around important buttons or temporarily disable other parts of the page
- **Click delay** that require the user to hover over a sensitive button for a short time before it becomes clickable

## Phishing Attacks

**Phishing** is a social engineering attack in which the attacker creates a fake website, email, or message that appears legitimate in order to trick the victim into revealing sensitive information (such as passwords, credit card numbers, or 2FA codes) or performing harmful actions (such as installing malware or authorizing a transaction).

While phishing has existed for decades, web-based phishing has become increasingly sophisticated and difficult to detect.

### Common Web Phishing Techniques

**1. Homograph Attacks (IDN Homographs)**  
Attackers register domain names that look almost identical to legitimate ones by using visually similar characters from different alphabets. For example:

- `paypaI.com` (using capital “I” instead of lowercase “l”)
- `аррle.com` (using Cyrillic “а” which looks identical to Latin “a”)
- `g00gle.com` (using zeros instead of the letter “o”)

These domains can obtain valid SSL certificates, making the phishing site appear completely legitimate to the user.

**2. Browser-in-Browser Attacks**  
The attacker creates a complete fake browser window inside the page using HTML, CSS, and JavaScript. This fake browser includes:

- A realistic address bar with a green padlock icon
- Tabs and browser chrome (minimize, maximize, close buttons)
- A convincing login page for a popular service (Google, Microsoft, bank, etc.)

**3. Certificate and Trust Indicator Abuse**  
Attackers obtain legitimate SSL certificates for lookalike domains. They exploit users’ tendency to trust the green padlock icon without checking the actual domain name.
Some advanced attacks even spoof Extended Validation (EV) certificate indicators.

**4. Other Common Techniques**

- Typosquatting (registering common misspellings of popular domains)
- Using URL shorteners to hide the real destination
- Creating highly convincing replicas of login pages

Phishing succeeds because it exploits several human tendencies:

- Trust in familiar visual cues (logos, colors, padlock icons)
- Inability or unwillingness to carefully inspect URLs
- Sense of urgency created by the attacker (“Your account will be locked in 24 hours!”)
- Fatigue and distraction while browsing

Even technically sophisticated users can fall victim to well-crafted phishing attacks.

## Defenses Against Phishing

Phishing is difficult to eliminate completely because it exploits human psychology, but a combination of technical and non-technical defenses can significantly reduce its effectiveness.

### Technical Defenses

**1. HTTPS Enforcement + Certificate Transparency**  
Force all sensitive pages and login forms to use HTTPS with HTTP Strict Transport Security (HSTS). Modern browsers also check Certificate Transparency logs, which helps detect suspicious or fraudulent certificates.

**Limitation**: While HTTPS prevents certain network-level attacks (such as man-in-the-middle), it does not stop attackers from registering lookalike domains and obtaining valid SSL certificates for them.

**2. Improved Browser URL Display and Warnings**  
Modern browsers highlight the most important part of the domain name (e.g., making `bank.com` bold) and show clearer warnings when visiting suspicious or newly registered domains.

**Limitation**: These improvements help, but many users still do not carefully inspect the URL bar, especially on mobile devices where the full address is often hidden.

**3. Password Managers**  
Good password managers only autofill credentials on the exact matching domain. This prevents most phishing sites from capturing passwords, even if the user is tricked into visiting a fake login page.

**Limitation**: This defense only works if users actually adopt and use a password manager. Many users still type passwords manually or reuse the same password across sites.

**4. Phishing-Resistant Multi-Factor Authentication (MFA)**  
Traditional SMS or app-based one-time passwords can still be phished because attackers can forward or intercept them in real time. Stronger options include hardware security keys (YubiKey, Google Titan), WebAuthn / FIDO2, and passkeys. These methods are much harder to phish because cryptographic credentials are bound to the legitimate domain.

**Limitation**: Phishing-resistant MFA provides excellent protection but can be more expensive to deploy at scale and may require additional user training or hardware distribution.

### Non-Technical Defenses

- **Security Awareness Training**
  Teach users to always check the full URL (especially the domain), be suspicious of unexpected requests for credentials or urgent actions, and hover over links before clicking.

- **Simulated Phishing Exercises**
  Many organizations regularly send fake phishing emails or messages to train employees and measure awareness levels.

**Limitation of User Education**: While training raises short-term awareness, studies consistently show that its long-term effectiveness is limited. Users tend to forget training over time, especially when they are busy or distracted.
