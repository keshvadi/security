---
title: Cookies and Session Management
parent: Web Security
nav_order: 4
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Cookies and Session Management

## What Are Cookies?

HTTP is a **stateless protocol**. This means that every request a browser sends to a web server is completely independent of every other request. The server does not automatically remember who the user is or what they did in previous interactions. While this design makes HTTP simple and highly scalable, it creates a major challenge for real-world web applications. Users expect websites to remember information across multiple page loads and visits, whether they are logged in, what items are in their shopping cart, their dark mode preference, partially completed forms, or language and region settings. Without some way to maintain state, every page refresh or link click would cause the website to forget everything about the user. **HTTP cookies** were introduced specifically to solve this problem.

At a high level, you can think of cookies as small pieces of data that a web server asks the browser to store on the user’s device.
When you visit a website, the server can include a `Set-Cookie` header in its HTTP response. The browser stores the cookie (a name-value pair together with various attributes). On all subsequent requests to the same website (or matching domain and path), the browser automatically attaches the cookie in the `Cookie` header. This simple mechanism allows the server to recognize returning users and customize its responses accordingly.

For example, after a successful login a server might set a session cookie like this:

```http
Set-Cookie: session=abc123xyz; Path=/; Secure; HttpOnly; SameSite=Lax
```

The browser will then automatically include it in future requests:

```http
Cookie: session=abc123xyz
```

## Cookie Attributes and Security Best Practices

Cookie attributes are powerful settings that control how cookies behave. They determine the cookie’s scope, lifetime, and security properties. While we will explore attacks such as Cross-Site Scripting (XSS) and Cross-Site Request Forgery (CSRF) in detail in later sections, it is important to understand now how proper cookie configuration can protect against them.

The most important cookie attributes and their recommended security settings are summarized below:

| Attribute  | Purpose                                | Security Benefit               | Example                     | What the example means                                                                    |
| ---------- | -------------------------------------- | ------------------------------ | --------------------------- | ----------------------------------------------------------------------------------------- |
| Name/Value | The actual data being stored           | —                              | `sessionid=abc123def456...` | A unique, random session identifier                                                       |
| Domain     | Which domains receive the cookie       | Reduces attack surface         | `Domain=example.com`        | Cookie is sent to example.com and all its subdomains                                      |
| Path       | Which paths receive the cookie         | Reduces attack surface         | `Path=/`                    | Cookie is sent to every page on the site                                                  |
| Secure     | Only send over HTTPS                   | Prevents network eavesdropping | `Secure`                    | Cookie is never sent over unencrypted HTTP connections                                    |
| HttpOnly   | Block JavaScript access                | Protects against XSS theft     | `HttpOnly`                  | JavaScript cannot read or steal this cookie                                               |
| SameSite   | Control when cookie is sent cross-site | Protects against CSRF          | `SameSite=Lax`              | Cookie is sent during normal navigation but blocked on most dangerous cross-site requests |
| Max-Age    | How long the cookie lives              | Limits damage window if stolen | `Max-Age=86400`             | Cookie automatically expires after 24 hours                                               |

**Example of a secure session cookie**:

```http
Set-Cookie: sessionid=abc123def456...; Max-Age=86400; Path=/; Secure; HttpOnly; SameSite=Lax
```

## Cookie Scoping Rules (Domain and Path)

The browser uses two attributes to decide which cookies to send with each request: `Domain` and `Path`.

- **Domain** uses **suffix matching**: A cookie with `Domain=example.com` will be sent to `example.com` and any of its subdomains (`www.example.com`, `api.example.com`, etc.).
- **Path** uses **prefix matching**: A cookie with `Path=/app` will be sent to any URL whose path starts with `/app` (e.g., `/app`, `/app/settings`, `/app/user/123`), but **not** to `/admin` or the root path `/`.

These rules are more permissive than the Same-Origin Policy, which requires an exact match on scheme, host, and port.

For security reasons, a website can **only set cookies for its own domain or parent domains**. For example, `shop.example.com` can set a cookie for `shop.example.com` or `example.com`, but it cannot set a cookie for `google.com` or `bank.com`.

Additionally, browsers forbid setting cookies on broad top-level domains (such as `.com`, `.org`, `.edu`, or two-level TLDs like `.co.uk`). Allowing this would let one malicious site affect every website ending in that TLD.

## Session Management with Cookies

One of the most important uses of cookies is **session management**, i.e., keeping users logged in across multiple requests and page visits.
The process typically works as follows.

1. User enters username and password on the login page.
2. Server verifies the credentials.
3. If valid, the server generates a random, unpredictable session token.
4. The server stores the mapping: `session_token → user_id` (usually in a database or cache).
5. The server sends the session token to the browser in a `Set-Cookie` header.
6. On every future request, the browser automatically sends the session token cookie.
7. The server looks up the token and knows which user is making the request.

Because cookies are automatically sent by the browser, they must be handled with care. Poorly configured cookies can lead to serious vulnerabilities including session hijacking, CSRF, and XSS data theft.

**Key Best Practices**:

- Always use `HttpOnly` and `Secure` on session cookies.
- Use `SameSite=Lax` (or `Strict`) by default.
- Generate session tokens using cryptographically secure randomness.
- Set reasonable expiration times.
- Scope cookies as narrowly as possible (`Domain` and `Path`).
- Prefer modern session management libraries and frameworks that handle these details correctly.

## Cookies vs. Same-Origin Policy

It is important to understand that Cookie Policy and the Same-Origin Policy are different mechanisms that solve different problems:

- Same-Origin Policy protects the browser’s internal state (DOM access, `localStorage`, JavaScript execution context).
- Cookie Policy controls which cookies are automatically sent with HTTP requests.

Because of this difference, it is possible for two different origins to share cookies. For example, if a cookie is set with `Domain=tru.ca`, then both `eng.tru.ca` and `cs.tru.ca` can read and write it, even though they are different origins under the Same-Origin Policy.

This is why the `SameSite` attribute was introduced, to give developers more control over when cookies are sent across origins.

### Quick Comparison

| Aspect          | Same-Origin Policy                                          | Cookie Policy                                                  |
| --------------- | ----------------------------------------------------------- | -------------------------------------------------------------- |
| Matching Rule   | Strict equality (scheme + host + port)                      | Suffix matching on Domain + prefix matching on Path            |
| Path matters?   | No                                                          | Yes                                                            |
| Subdomains      | Different origins (e.g., `sub.example.com` ≠ `example.com`) | Can share cookies if `Domain` is set to parent                 |
| Scheme matters? | Yes (`http` ≠ `https`)                                      | No (cookies can be sent over both unless `Secure` flag is set) |
| Main Purpose    | Protect DOM, storage, and network requests                  | Control which cookies are sent with requests                   |
