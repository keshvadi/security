---
title: CAPTCHAs
parent: Web Security
nav_order: 8
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# CAPTCHAs

## 1. The Purpose of CAPTCHAs

Imagine you run a popular website that allows users to create accounts, post comments, or search for content. Without any protection, an attacker could easily write a simple script (a _bot_) to:

- Create thousands of fake accounts to send spam or manipulate online votes
- Repeatedly submit forms to overload your servers (a denial-of-service attack)
- Scrape your content, prices, or user data at high speed
- Brute-force login attempts on user accounts

These automated attacks are cheap, fast, and can cause serious damage to your service and your users.

**CAPTCHAs** were created to solve this problem. **CAPTCHA** stands for **Completely Automated Public Turing test to tell Computers and Humans Apart**.

The core idea is that a CAPTCHA presents a challenge that is easy for most humans to solve but difficult for current computer programs. By requiring users to complete this challenge before performing certain actions (such as creating an account or submitting a form), websites can significantly reduce automated abuse while still allowing real people through.

From a security perspective, CAPTCHAs serve as an important line of defense in the broader web security ecosystem:

- They help prevent mass account creation used for spam, fraud, and phishing campaigns
- They slow down brute-force attacks on login forms
- They protect computationally expensive operations (image processing, complex searches, etc.) from being abused
- They make large-scale web scraping more difficult and expensive

## 2. Traditional CAPTCHAs (Distorted Text)

The earliest and most well-known CAPTCHAs presented users with _distorted text_ that they had to type correctly.
A typical traditional CAPTCHA would:

1. Generate a random string of letters and numbers (e.g., `X7K9P2`)
2. Apply heavy visual distortions:
   - Warping and twisting the letters
   - Adding random lines, dots, or noise in the background
   - Varying fonts, sizes, and colors
   - Rotating or overlapping characters
3. Display the distorted image to the user
4. Ask the user to type the characters they see

Only if the user typed the correct string would the form submission be accepted.

When CAPTCHAs were first introduced in the early 2000s, they worked well because:

- _Optical Character Recognition (OCR)_ technology was not advanced enough to reliably read heavily distorted text
- The distortions exploited weaknesses in early computer vision algorithms
- Adding random noise and lines broke the assumptions that OCR systems made about clean, straight text

At the time, these challenges were genuinely difficult for machines but relatively easy for humans (most people could solve them in a few seconds).

To improve accessibility for visually impaired users, many sites also offered **audio CAPTCHAs**:

- A distorted audio clip would play a sequence of numbers or words
- Background noise (static, voices, music) was added to make it hard for speech recognition software
- Users would type what they heard

While helpful, audio CAPTCHAs were often even more frustrating for users than visual ones.

### The Problem with Traditional CAPTCHAs

Over time, traditional distorted-text CAPTCHAs became less effective because:

- OCR technology improved dramatically (thanks in part to data collected from CAPTCHA-solving itself)
- Attackers began using human farms i.e., paying people in low-wage countries small amounts of money to solve CAPTCHAs in bulk
- Users found them increasingly annoying and time-consuming

This led to the development of more advanced, modern CAPTCHA systems such as Google reCAPTCHA to address these limitations.

## 3. Modern CAPTCHAs (reCAPTCHA and Others)

As traditional distorted-text CAPTCHAs became less effective, modern CAPTCHAs shifted away from “prove you’re human by solving this puzzle” toward invisible, behavior-based systems that determine humanity by observing user interaction.
This new approach offers several advantages: it is far less annoying for legitimate users, more difficult for attackers to bypass at scale, and can adapt as attack techniques evolve.

The most widely used today is Google reCAPTCHA.

**reCAPTCHA v2** introduced several improvements over older systems:

- "I'm not a robot" checkbox: Often no challenge is shown at all if Google's risk analysis determines the user is likely human.
- Image selection challenges: When a challenge is shown, users are asked to select all images containing a specific object (cars, traffic lights, crosswalks, etc.).
- Behavioral analysis: The system silently tracks mouse movements, typing patterns, and other signals to assess whether the interaction looks human.

**reCAPTCHA v3** takes a completely different approach:

- It runs completely in the background with no user interaction required in most cases.
- It assigns a risk score (0.0 = very likely a bot, 1.0 = very likely a human) based on:
  - How the user interacts with the page
  - Their browsing history and behavior across sites
  - Device and network signals
- Website owners can then decide what action to take based on the score (allow, show extra verification, block, etc.).

However, it also raises new concerns about privacy (especially with systems that track behavior across many websites) and false positives (legitimate users sometimes being flagged as suspicious).
Several alternatives to reCAPTCHA have gained popularity, as summarized in the table below:

| System                   | Key Features                              | Notable Advantage                      |
| ------------------------ | ----------------------------------------- | -------------------------------------- |
| **hCaptcha**             | Image challenges, privacy-focused         | More privacy-friendly than Google      |
| **Cloudflare Turnstile** | Invisible + visible modes                 | Strong privacy + good accessibility    |
| **Arkose Labs**          | Game-like challenges, adaptive difficulty | Very strong against sophisticated bots |
| **Friendly Captcha**     | Proof-of-work (no images)                 | No visual challenges, good for privacy |

## 4. The Arms Race and Limitations

CAPTCHAs exist in a constant _arms race_ between defenders and attackers. As soon as a new type of CAPTCHA becomes popular, attackers develop new ways to bypass it.

In addition, advances in machine learning and computer vision have forced CAPTCHA designers to create increasingly difficult challenges for humans just to stay ahead of bots. This has created several problems: many people now find modern image CAPTCHAs annoying and time-consuming, visually impaired users often struggle with image-based challenges (even with audio alternatives), and difficulty levels vary wildly where sometimes trivially easy, other times nearly impossible for humans.

One of the most surprising developments in the CAPTCHA arms race is the rise of _human CAPTCHA farms_. Companies in certain countries pay workers very low wages (sometimes just a few cents per hour) to solve CAPTCHAs in bulk. Attackers can buy solved CAPTCHAs for as little as $0.10 to $0.50 per 1,000 solutions. This makes it economically viable for attackers to bypass even sophisticated CAPTCHAs at scale.

This reality has led some researchers to describe modern CAPTCHAs as _self-defeating_. They often end up training the very AI systems they were designed to defeat, while simultaneously creating a low-wage labor market for solving them.

The limitations of CAPTCHAs are summarized in the table below:

| Limitation                   | Description                                                                                                                                                                                                                 |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **User Experience**          | CAPTCHAs interrupt user workflows, often causing frustration, repeated attempts, and higher abandonment rates.                                                                                                              |
| **False Positives**          | Legitimate users, particularly those using VPNs, privacy tools, Tor, shared IPs, or non-standard devices, are sometimes incorrectly flagged or blocked.                                                                     |
| **Privacy Concerns**         | Behavioral CAPTCHAs (especially Google reCAPTCHA) collect and analyze user interaction data, often across multiple websites, raising significant privacy and data protection issues.                                        |
| **Accessibility**            | Many CAPTCHAs remain difficult or impossible for users with visual, auditory, motor, or cognitive disabilities, despite the existence of audio alternatives.                                                                |
| **Declining Effectiveness**  | Advances in AI and computer vision have narrowed or eliminated the gap between challenges that are "easy for humans but hard for machines". Modern bots frequently match or exceed human performance on many CAPTCHA types. |
| **CAPTCHA Solving Services** | Attackers can bypass CAPTCHAs at scale by using commercial solving services or low-wage CAPTCHA farms.                                                                                                                      |

## 5. Alternatives and Best Practices

There is a deep irony at the heart of CAPTCHAs: the better they become at distinguishing humans from machines, the more effectively they help train AI systems to close that very gap. As machine learning and computer vision continue to advance, this irony has become increasingly difficult to ignore. Many security experts now question whether traditional “prove you are human” challenges will remain viable as a primary defense in the long term.

Despite these limitations, CAPTCHAs are still appropriate in certain situations. They are most useful when strong protection against automated abuse is needed on publicly accessible forms, when the action being protected is high-value or high-risk (such as account creation, financial transactions, or content uploads), and when the application faces targeted attacks from sophisticated bots.

As AI capabilities grow, the industry is gradually shifting away from explicit challenge-response tests toward continuous, invisible verification methods. These newer approaches rely on behavioral signals, device characteristics, and risk scoring rather than forcing users to solve puzzles. In this evolving landscape, CAPTCHAs are increasingly viewed as one tool among many rather than the central line of defense.

Modern web applications therefore often combine CAPTCHAs with (or replace them by) other anti-bot techniques. In many cases, the following approaches can be more effective or less intrusive:

| Technique                       | How It Works                                                                                                       | Best Used For                                        | User Impact |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------- | ----------- |
| **Rate Limiting**               | Restricts the number of requests an IP address or user can make within a given time window                         | Most forms, login pages, and APIs                    | Low         |
| **Behavioral Analysis**         | Identifies non-human patterns in mouse movements, timing, scrolling, keystroke dynamics, and interaction sequences | Login, checkout, high-value forms, and API endpoints | Very Low    |
| **Device Fingerprinting**       | Collects device and browser characteristics to create a persistent identifier across sessions                      | High-risk actions and fraud prevention               | Low         |
| **Proof-of-Work**               | Requires the browser to solve a small computational puzzle before submitting a request                             | APIs, high-traffic endpoints, and registration forms | Low         |
| **Honeypot**                    | Places hidden form fields that legitimate users cannot see or interact with, but automated scripts often complete  | Comment forms, registration, and contact forms       | Very Low    |
| **Multi-Factor Authentication** | Requires a second verification factor (authenticator app, hardware key, or SMS) after initial credentials          | Login and account recovery                           | Medium      |
| **Email or Phone Verification** | Sends a one-time code or link that the user must enter or click to confirm ownership                               | Account creation and high-risk registrations         | Medium      |
