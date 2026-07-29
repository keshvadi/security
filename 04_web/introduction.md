---
title: Introduction to the Web
parent: Web Security
nav_order: 1
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Introduction to the Web

## The Fundamental Challenge of Web Security

Imagine you are sitting in a café with your laptop. You open your browser, go to Amazon, and decide to buy a new pair of headphones. You click “Add to Cart” and then “Proceed to Checkout”.
What actually happens behind the scenes when you perform that click?

Your browser creates a simple **HTTP request** and sends it across the Internet to Amazon’s web server. This request contains several pieces of information:

- Which action you want to perform (e.g., “add this specific item to my cart”)
- Any data you provided (quantity, size, color, etc.)
- Technical details about your browser and connection

The Amazon server receives this request, processes it (checks inventory, updates your shopping cart in its database, and possibly records the action for analytics), and sends back an **HTTP response**. Your browser then updates the page to show that the item has been successfully added to your cart.

This _request-response cycle_ is the fundamental model of the entire World Wide Web. Every time you click a link, submit a form, or navigate to a new page, your browser initiates a fresh request to a remote server, and that server replies with the appropriate data.

Now consider you add a few more items to your cart, close your laptop, leave the café, and go home. Later that evening you open your laptop again, return to Amazon, and continue shopping. The website still knows who you are and shows your cart exactly as you left it. It does not ask you to log in again. How is this possible?

When you first logged in, Amazon’s server sent your browser a small piece of data called a _cookie_ containing your session credentials. From that moment on, your browser automatically attaches this cookie to every request it sends to Amazon. You don’t have to do anything. The browser handles it silently in the background. This mechanism allows websites to maintain state (who you are, what’s in your cart, your preferences) across many independent requests.

This convenience, however, introduces one of the most fundamental and difficult security challenges on the web.

While you still have the Amazon shopping page open in one tab, suppose you open your email in another tab and accidentally click on a malicious link. This takes you to a shady website controlled by an attacker. Here is the dangerous question: _Can that malicious website cause your browser to send a request to Amazon on your behalf?_. As we discussed earlier, clicking the checkout button simply sends an HTTP request and there is nothing magical about it. The real question is: Can another website cause the browser to send the same HTTP request? And if it does, will the browser automatically attach your Amazon cookies (and therefore your identity and session) to that request?

If the answer is “yes”, then the attacker could potentially add expensive items to your cart, change your shipping address, or even make purchases using your saved payment methods and perform similar attacks on your email, banking, or investment accounts.

The answer to both those questions is yes! The malicious website can trick your browser into making requests to Amazon (for example, by embedding an invisible image or submitting a hidden form). Because your browser automatically includes any cookies that belong to Amazon, the request will arrive at Amazon’s servers looking exactly as if _you_ had made it. The same risk also applies to your email, banking, investment accounts, or any other site where you are currently logged in.

You might think, “I will simply never visit a malicious website”. Unfortunately, the problem is even broader. Many legitimate websites load third-party JavaScript libraries, fonts, analytics scripts, or other resources from external services and content delivery networks (CDNs). If any of those third-party components is malicious or has been compromised, attacker-controlled code can execute in your browser even though you never left a trusted domain.

This is one of the hardest problems in web security. Users routinely have multiple websites open at the same time in different tabs, different windows, or even embedded inside each other using `<iframe>` elements. The central goal of web security is to ensure that a malicious website or malicious code should never be able to:

- Steal sensitive information from a legitimate site (your emails, photos, bank balance, etc.)
- Perform actions on your behalf without your knowledge or consent
- Interfere with pages belonging to a different origin

## URLs: Locating Resources on the Web

Every resource on the web (whether it is a webpage, an image, a PDF, a video, or an API endpoint) is identified by a _Uniform Resource Locator (URL)_. A URL tells your browser exactly where to find a piece of information and how to retrieve it.
Before diving into URLs, it is useful to understand three related but distinct terms:

- **URI (Uniform Resource Identifier)**: A unique string that identifies an abstract or physical resource
- **URN (Uniform Resource Name)**: A type of URI that identifies a resource by name in a particular namespace. Examples include:
  - `urn:isbn:0130460192` (a book identified by its ISBN)
  - `urn:ietf:rfc:2648` (an IETF Request for Comments document)
  - `mailto:professor@university.ca` (an email address)
  - `tel:+1-403-220-1234` (a phone number)
- **URL (Uniform Resource Locator)**: The most common type of URI. A URL not only _identifies_ a resource but also provides instructions on _how to locate and retrieve_ it.

A basic URL consists of three mandatory parts and several optional parts. Consider the following example:

<p style="text-align: center">
  <code>
    <span style="color:blue">https</span
    >://<span style="color:green">www.example.com</span
    ><span style="color:red">/index.html</span>
  </code>
</p>

- **Scheme (or Protocol)** — shown in <span style="color:blue">blue</span>  
  The scheme tells the browser _how_ to communicate with the server. The two most important schemes for this course are `http` and `https` (the secure version of HTTP that uses TLS encryption). Other schemes include `ftp://`, `file://`, and `mailto:`.

- **Host (or Domain / Location)** — shown in <span style="color:green">green</span>  
  This part identifies _which computer_ on the Internet should be contacted. It usually consists of a domain name such as `www.example.com`.  
  A domain name is a series of dot-separated labels. The rightmost label is the **top-level domain (TLD)** (e.g., `.com`, `.org`, `.edu`, `.ca`). Labels to the left are subdomains. For example, in `www.eng.tru.ca`:

  - `.ca` is the TLD
  - `tru` is a second-level domain
  - `eng` is a third-level domain
  - `www` is the specific server

  The host may optionally include a **username** (followed by `@`) and a **port number** (preceded by `:`). If no port is specified, the browser uses the default port for the scheme (port 80 for `http`, port 443 for `https`).

- **Path** — shown in <span style="color:red">red</span>  
  The path tells the server _which specific resource_ to return. It is conceptually similar to a file path on a computer’s filesystem. Every URL must have at least the root path `/`.

After the path, two optional components may appear:

- **Query Parameters** (after a `?` character)  
  These allow you to send additional data to the server. Parameters are key-value pairs separated by `&`. For example:  
  `https://www.google.com/search?q=web+security&hl=en`

- **Fragment (or Anchor)** (after a `#` character)  
  The fragment identifies a specific part of the resource (for example, a section heading on a webpage). The fragment is not sent to the server. It is handled entirely by the browser (usually to scroll to that part of the page).

A complete URL with all parts may look like this:

<p style="text-align: center">
  <code>
    <span style="color:blue">https://</span
    ><span style="color:red">user@</span
    ><span style="color:blue">www.example.org</span
    ><span style="color:red">:8443</span
    ><span style="color:blue">/path/to/page</span
    ><span style="color:red">?id=123&amp;sort=desc</span
    ><span style="color:blue">#section-2</span>
  </code>
</p>

URLs are not just addresses. They are also a common attack surface. Attackers frequently manipulate URLs to:

- Inject malicious parameters (reflected XSS)
- Trick users with look-alike domains (phishing and homograph attacks)
- Bypass security checks that rely on URL parsing

## How the Web Actually Works

At its core, the World Wide Web is a _distributed client-server system_. Your browser acts as the _client_, while remote computers running web server software act as the _servers_.

### HTTP (Hypertext Transfer Protocol)

The communication between the browser and the server is governed by the **Hypertext Transfer Protocol (HTTP)**. HTTP follows a simple but powerful **request-response model**:

1. The browser (client) opens a connection to the server and sends an **HTTP request**.
2. The server processes the request — often by querying databases, performing computations, or generating dynamic content — and sends back an **HTTP response**.
3. The connection is usually closed (or kept alive for efficiency in modern versions of HTTP).

A modern webpage is no longer just a static document. It is a **distributed application** with two cooperating parts:

- **Server-side code** running on the web server (written in languages such as Python, Node.js, PHP, Ruby, Go, Java, etc.).
- **Client-side code** running inside your browser, primarily JavaScript.

When you visit a site, the server performs some computation, generates HTML (and possibly structured data such as JSON), and sends it to your browser. Your browser then parses the HTML, applies CSS for styling, executes JavaScript for interactivity, and renders the final interactive page you see.

### Stateless by Default

One of the most important characteristics of HTTP is that it is **stateless**. This means every request your browser makes is completely independent of every other request. The server does not automatically remember who you are or what you did in previous requests.

This design choice has major implications. On the one hand, it makes the web simple and highly scalable because servers do not have to maintain state for millions of simultaneous users. On the other hand, it creates significant challenges for building real applications. How do you keep a user logged in across multiple page loads? How do you remember the items in a shopping cart? How do you maintain user preferences? These challenges are solved using mechanisms we will study in detail later in this unit, most notably **cookies** and **session management**.

### Structure of an HTTP Request

Here is a typical HTTP request:

```
GET /search?q=web+security HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

Every HTTP request has the following parts:

- **Request Line** (first line): Contains three pieces of information: the **method** (`GET`), the **path** (`/search?q=web+security`), and the **HTTP version** (`HTTP/1.1`).
- **Request Headers** (subsequent lines): Key-value pairs that provide additional information about the request (e.g., which host is being contacted, what kind of content the browser accepts, authentication details, cookies, etc.).
- **Blank Line**: Separates the headers from the optional body.
- **Request Body** (optional): Present only in certain request types (primarily `POST` and `PUT`). Contains data being sent to the server (for example, form submissions or JSON payloads).

### Structure of an HTTP Response

Here is a typical HTTP response:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 4521
Date: Sat, 30 May 2026 17:10:00 GMT
Server: gws
Set-Cookie: NID=1234567890; expires=...
Connection: keep-alive

<!DOCTYPE html>
<html>
... (the actual HTML content of the page)
```

Every HTTP response contains:

- **Status Line** (first line): Includes the HTTP version, a **status code** (e.g., `200`), and a **reason phrase** (`OK`).
- **Response Headers**: Provide metadata about the response (content type, length, caching instructions, cookies to set, security headers, etc.).
- **Blank Line**
- **Response Body**: The actual content being returned (HTML, JSON, image data, etc.).

The two most common HTTP methods are `GET` and `POST`.

`GET` is used to retrieve data from the server. According to the HTTP specification, GET requests are intended to be safe (they should not modify any data on the server) and idempotent (making the same request multiple times should produce the same result). All parameters are included directly in the URL, which means they appear in the browser’s address bar, browser history, and server logs. GET requests cannot contain a request body.

`POST` is used to send data to the server that will modify its state, for example, logging in, submitting a form, uploading a file, or completing a purchase. The data is placed in the request body instead of the URL, so it remains hidden from the address bar. Unlike GET requests, POST requests are neither safe nor idempotent.

Because GET parameters are part of the visible URL, they are easily logged, bookmarked, and shared. Sensitive information such as passwords, session tokens, or personal data should never be sent using GET requests. We will later see that many reflected cross-site scripting attacks exploit GET parameters.

## The Building Blocks of a Web Page

When a browser receives an HTTP response, the body is usually an HTML document. Modern webpages, however, are built from three core technologies working together: HTML, CSS, and JavaScript. These three pillars define the structure, appearance, and behavior of almost every website on the Internet.

### HTML

**HTML (Hypertext Markup Language)** provides the structure and content of a webpage. It allows us to create headings, paragraphs, links, images, forms, tables, and many other elements.
You do not need to be an HTML expert for this course, but you must recognize several elements that have important security implications:

- **Links**: `<a href="http://example.com">Click me</a>`
- **Images**: `<img src="http://example.com/photo.jpg">`
- **Scripts**: `<script>alert("Hello");</script>` or `<script src="http://example.com/app.js"></script>`
- **Frames/Iframes**: `<iframe src="http://example.com"></iframe>`
- **Forms**: `<form action="/submit" method="POST">...</form>`

The `<iframe>` element is particularly important. When one webpage embeds another using an iframe, the browser enforces frame isolation. The outer page cannot read or modify the content of the inner page, and vice versa. This is an early form of the same-origin policy we will study in detail later.

### CSS

**CSS (Cascading Style Sheets)** controls the visual appearance of an HTML page like fonts, colors, layout, spacing, animations, and more.
While CSS is primarily a styling language, it is surprisingly powerful when used maliciously. An attacker who can inject malicious CSS can:

- Steal information by reading computed styles
- Perform UI redressing (clickjacking) attacks
- Exfiltrate data using clever CSS tricks

In practice, if an attacker can force a victim to load malicious CSS, the effect is often comparable to forcing the victim to load malicious JavaScript.

### JavaScript

**JavaScript** is a full-featured programming language that runs directly inside the browser. It is currently the only programming language that all modern web browsers can execute natively. This means that no matter which client-side framework or language you use (such as React, Vue, Angular, Svelte, or TypeScript), the code is ultimately translated or compiled into plain JavaScript that the browser can run. Even server-side frameworks (like Ruby on Rails, Django, or PHP) typically generate HTML and JavaScript bundles that the browser executes.

JavaScript is what makes modern websites truly interactive. It enables features such as buttons that respond to clicks, content that updates dynamically without reloading the page (AJAX), real-time form validation, animations, and complex single-page applications like Gmail, Google Docs, and online banking portals. Because of its power and flexibility, JavaScript has moved far beyond the browser. Many modern desktop applications such as Slack, Discord, and Visual Studio Code are actually built using web technologies through frameworks like Electron.

Because JavaScript is so powerful, modern browsers run it inside a strict **sandbox** to protect users. This sandbox limits what JavaScript code can access on the user’s computer. However, this protection breaks down if malicious JavaScript manages to run _inside_ a legitimate website (for example, through a Cross-Site Scripting or XSS attack). In that case, the malicious code executes with the full privileges of the legitimate page. It can steal session cookies, read sensitive form data, make requests on the user’s behalf, and silently exfiltrate information back to the attacker.

To understand how JavaScript interacts with a webpage, we need to look at the **Document Object Model (DOM)**. When the browser receives an HTML document, it first parses the HTML into an internal tree-like representation called the DOM. JavaScript can then read from and modify this DOM in real time. It can change content, update styles, respond to user events (clicks, typing, scrolling), and make additional network requests.

**Where JavaScript Can Be Placed**

JavaScript can be included in a webpage in several ways:

- Embedded directly: `<script>alert("Hello");</script>`
- Loaded from an external file: `<script src="app.js"></script>`
- As an event handler: `<button onclick="doSomething()">Click</button>`
- Using the `javascript:` pseudo-protocol (rare and dangerous)

Because JavaScript is so powerful, controlling where it comes from and what it is allowed to do is a central goal of web security.

## How a Webpage Actually Loads

Now that we have covered URLs, HTTP, and the three pillars of web content, let’s see how everything works together when you visit a website.
When you type a URL into your browser and press Enter, the following steps occur (simplified):

1. DNS Resolution: The browser asks the DNS to translate the hostname (e.g., `www.example.com`) into an IP address.
2. TCP Connection: The browser opens a TCP connection to the server’s IP address on the appropriate port (usually 443 for HTTPS or 80 for HTTP).
3. TLS Handshake (for HTTPS): If the URL uses `https`, the browser and server perform a TLS handshake to establish an encrypted connection.
4. HTTP Request: The browser sends an HTTP request, usually a `GET` request for the main page.
5. Server Response: The server processes the request and returns an HTTP response, typically containing HTML in the body.
6. HTML Parsing and DOM Construction: The browser parses the HTML and builds the DOM, an internal tree representation of the page.
7. Subresource Loading: While parsing the HTML, the browser discovers references to other resources:

   - CSS files (`<link rel="stylesheet">`)
   - JavaScript files (`<script src="...">`)
   - Images (`<img src="...">`)
   - Fonts, videos, iframes, etc.

   The browser makes additional HTTP requests (often in parallel) to fetch these subresources.

8. CSS and JavaScript Processing:
   - CSS is applied to style the page.
   - JavaScript is executed (in the order it appears or as specified by `async`/`defer` attributes).
   - JavaScript can further modify the DOM and trigger additional network requests.
9. Rendering: The browser renders the final visual result on the screen. The page is now interactive.

Many of the vulnerabilities we will study in the rest of this unit directly target specific stages of this process. **Phishing** and **reflected XSS** attacks often begin with carefully crafted malicious URLs that abuse steps 1 (DNS resolution) and 4 (HTTP request). **Stored XSS** succeeds when an attacker injects malicious HTML or JavaScript that is later parsed and executed during steps 6–8. **CSRF** attacks rely on the browser automatically attaching cookies to forged requests in step 4. **Clickjacking** manipulates the visual rendering and framing behavior in steps 7–9, while **session hijacking** frequently targets cookies transmitted during the request and response phases (steps 4 and 5).

In addition, modern browsers and newer versions of HTTP (HTTP/2 and HTTP/3) introduce several important optimizations. These include **multiplexing**, which allows multiple requests and responses to be sent over a single connection at the same time, **server push** capabilities in HTTP/2, and the **QUIC** protocol in HTTP/3 for faster performance. Browsers and servers also make heavy use of caching, controlled by headers such as `Cache-Control` and `ETag`, to avoid re-downloading resources that have not changed.
These optimizations improve speed but do not fundamentally change the security model and are out of the scope of our study.
