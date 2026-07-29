---
title: Cross-Site Request Forgery (CSRF)
parent: Web Security
nav_order: 4
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Cross-Site Request Forgery (CSRF)

## What is Cross-Site Request Forgery (CSRF)?

**Cross-Site Request Forgery (CSRF)** is an attack where a malicious website tricks a victim’s browser into making an unintended request to a legitimate website where the victim is already logged in.

Imagine this scenario: You are logged into your online banking website in one browser tab. In another tab, you visit a malicious website (perhaps through a phishing link or a malicious advertisement). That malicious site can silently cause your browser to send a request to the bank (for example, to transfer money to the attacker’s account) all without you clicking anything suspicious or even knowing it happened.

In simple terms, the attacker causes the victim’s browser to perform actions _on the victim’s behalf_ without the victim’s knowledge or consent. CSRF attacks can have serious consequences, such as transferring money from the victim’s bank account, changing their password or email address, making purchases on their behalf, deleting important data, or performing any action the victim is authorized to do on the target website.

Although modern defenses, especially `SameSite` cookies and CSRF tokens, have made CSRF harder to exploit, it remains a relevant threat. Many older applications and APIs still lack proper protection, developers sometimes forget to secure new endpoints (especially JSON APIs and microservices), some frameworks do not enable CSRF protection by default, and sophisticated attacks can still bypass weak implementations.

## 2. How CSRF Attacks Work

A CSRF attack follows a simple pattern: the attacker tricks the victim’s browser into sending a request to a legitimate website while the victim is authenticated.
Let’s look at two classic attack techniques.

### Attack 1: GET-Based CSRF (Image Tag Trick)

Many websites used to allow state-changing actions via simple `GET` requests (this is now considered bad practice).

**Example Scenario**:  
A bank allows money transfers using a URL like this:

```
https://bank.com/transfer?amount=1000&to=attacker@evil.com
```

**Malicious Code** (hosted on `https://evil.com`):

```html
<img
  src="https://bank.com/transfer?amount=1000&to=attacker@evil.com"
  style="display:none;"
/>
```

**What happens**:

1. The victim is logged into `bank.com` (has a valid session cookie).
2. The victim visits `evil.com` (perhaps through a phishing link).
3. The browser automatically loads the hidden `<img>` tag.
4. The browser sends a `GET` request to `bank.com/transfer` **with the victim’s session cookie attached**.
5. The bank processes the transfer as if the victim made it intentionally.

The victim never sees anything suspicious. The image simply fails to load (because it returns HTML instead of an image), but the damage is already done.

### Attack 2: POST-Based CSRF (Auto-Submitting Form)

Most modern applications use `POST` requests for state-changing actions. CSRF is still possible using a hidden auto-submitting form.

**Malicious Code** (on `https://evil.com`):

```html
<form id="evilform" action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="amount" value="1000" />
  <input type="hidden" name="to" value="attacker@evil.com" />
</form>

<script>
  document.getElementById("evilform").submit();
</script>
```

**What happens**:

1. The victim visits `evil.com`.
2. The JavaScript automatically submits the hidden form.
3. The browser sends a `POST` request to `bank.com/transfer` **with the victim’s session cookie**.
4. The bank processes the transfer.

Again, the victim has no idea this happened.

These attacks succeed simply because:

- The victim’s browser **automatically attaches cookies** (including session cookies) to requests made to `bank.com`.
- The bank trusts any request that has a valid session cookie.
- There is **no verification** that the request actually came from the victim’s intentional action on the bank’s own website.

### Why CSRF is Possible

It is important to understand that the Same-Origin Policy does not prevent CSRF attacks. HTTP is a stateless protocol, so websites rely on cookies (especially session cookies) to keep users logged in. The browser automatically attaches all relevant cookies to every request it sends to a domain, regardless of where the request originated. While the Same-Origin Policy prevents JavaScript from reading or modifying data across different origins, it does nothing to stop the browser from sending simple requests (such as loading an image or submitting a form). As long as the server accepts the request and processes it, the attack succeeds. This is why CSRF is fundamentally a _server-side vulnerability_. The browser will happily send the authenticated request, and the responsibility for defending against it falls entirely on the web application itself.

You might wonder: why does the browser attach the victim’s `bank.com` session cookie to a request that was triggered while the user was visiting `evil.com`? The answer is that cookies are scoped by **domain**, not by the page that caused the request. Whenever the browser makes any HTTP request to `bank.com`, whether the user clicked a link on the bank’s own site, loaded an image, or the request was initiated by a completely different website, it automatically includes all cookies that belong to `bank.com`. This behavior is by design: it makes many legitimate features of the web work smoothly, such as third-party images, fonts, and analytics. Unfortunately, it is also the exact reason CSRF attacks are possible.

## Historical Real-World CSRF Vulnerabilities

Cross-Site Request Forgery (CSRF) is one of the most common and persistent web security vulnerabilities, and attacks of this type continue to be discovered even today. The following real-world examples illustrate how dangerous and widespread this attack has been over the years.

In 2006, Netflix suffered from several serious CSRF vulnerabilities. An attacker could create a malicious webpage that, when visited by a logged-in user, would add DVDs to the victim’s rental queue, change the shipping address on the account, or even change the account password.

In 2008, a significant CSRF vulnerability was discovered in Gmail. While a user was logged into their Gmail account, an attacker could trick them into visiting a malicious website (often through a phishing email or malicious advertisement). The malicious page contained hidden form submissions that automatically created a new mail filter in the victim’s Gmail settings. This filter silently forwarded all incoming emails to an attacker-controlled address, giving the attacker full access to the victim’s private correspondence without the user ever noticing.

Banking websites have been frequent targets of CSRF attacks. In several well-documented cases during the late 2000s and early 2010s, banks allowed important actions such as money transfers through simple GET or POST requests that lacked proper CSRF protection. An attacker would create a malicious webpage containing an invisible form or image tag that triggered a transfer from the victim’s account to the attacker’s account. When a logged-in user accidentally visited the malicious site, their browser would automatically send the transfer request along with the victim’s valid session cookie, resulting in real financial losses.

Social media platforms have also suffered from serious CSRF vulnerabilities. In multiple incidents, attackers were able to force logged-in users to perform actions on their behalf, such as posting status updates, liking pages, sending friend requests, or even changing account privacy settings. Because the user was already authenticated, the malicious site could silently submit these requests using the victim’s session cookie. These attacks not only compromised user privacy but also enabled widespread social engineering and reputation damage.

In 2020, a CSRF vulnerability was found in TikTok that affected users who had signed up using third-party accounts (such as Google or Apple). The vulnerability allowed an attacker to silently reset the password on the victim’s TikTok account. When combined with another bug, it enabled a one-click account takeover, giving the attacker full control over the victim’s account.

## Defenses Against CSRF Attacks

Web applications use several complementary defenses to protect against CSRF attacks. The most effective and widely recommended defense is the use of **CSRF tokens** (also known as the synchronizer token pattern). Two other important protections are the **`SameSite`** cookie attribute and validation of the **`Referer`** or **`Origin`** HTTP headers. While each technique works differently and has its own strengths and limitations, combining multiple defenses usually provides the strongest protection.

### Primary Defense: CSRF Tokens (Synchronizer Token Pattern)

The most effective and widely used defense against CSRF is the **CSRF Token** (also called Synchronizer Token Pattern).

Here is how CSRF tokens work:

1. When the legitimate website generates a page containing a form (or any state-changing action), the server creates a random, unpredictable token.
2. The server includes this token in the HTML form as a hidden field.
3. The server also stores the token (usually associated with the user’s session).
4. When the user submits the form, the browser sends both the form data **and** the CSRF token.
5. The server verifies that the submitted token matches the one it expects for that session.
6. If the token is missing or invalid, the server rejects the request.

Because an attacker on a different origin cannot read the token from the legitimate page (due to the Same-Origin Policy), they cannot forge a valid request.

Here is how a CSRF token is typically used in practice:

**Legitimate Form (generated by the server):**

```html
<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="a7f3k9x2mPq8vL5nRjT6wYbH" />
  <input type="text" name="amount" placeholder="Amount" />
  <input type="text" name="to" placeholder="Recipient" />
  <button type="submit">Transfer</button>
</form>
```

**Server-side validation (conceptual):**

```python
if request.form.get('csrf_token') != session.get('csrf_token'):
    return "Invalid CSRF token", 403
# Proceed with the transfer...
```

The token is random and unpredictable, so an attacker cannot guess it. It is tied to the user’s specific session and cannot be reused by others. Most importantly, because the token is embedded in the legitimate page, an attacker on a different website cannot read it due to the Same-Origin Policy. As a result, the attacker cannot include a valid token in any forged request.

### Defense: SameSite Cookies

Another effective defense against CSRF is the **`SameSite`** cookie attribute. This attribute tells the browser when it is allowed to send the cookie along with cross-site requests.

When setting a cookie, the server can include the `SameSite` attribute:

```http
Set-Cookie: sessionid=abc123; SameSite=Lax; Secure; HttpOnly
```

The `SameSite` attribute supports three possible values:

| Value    | Behavior                                                                                                                                           |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Strict` | The cookie is **never** sent with any cross-site request (even when the user clicks a link). Most secure, but can break some legitimate workflows. |
| `Lax`    | The cookie is sent with **safe** cross-site requests (typically top-level GET navigations). This is the **default** in all modern browsers.        |
| `None`   | The cookie is sent with **all** cross-site requests. Must be combined with the `Secure` flag. Provides no CSRF protection.                         |

With `SameSite=Lax` or `Strict`, a malicious site on `evil.com` cannot cause the victim’s browser to send a request to `bank.com` **with the victim’s session cookie attached**. Because the cookie is not sent, the bank’s server will treat the request as unauthenticated and reject it.

SameSite is a powerful defense because it works automatically in the browser with no extra code required on forms or in JavaScript.
However, it has some limitations: it does not protect against same-site attacks (such as a subdomain compromise), it may not be supported in very old browsers, and it only protects cookie-based authentication (it has no effect if the application uses other methods such as JWTs stored in `localStorage`).

### Defense: Referer / Origin Header Validation

Another defense against CSRF is to check the `Referer` or `Origin` HTTP header on incoming requests. When a browser makes a request, it usually includes one of these headers: the `Referer` header contains the full URL of the page that caused the request, while the `Origin` header contains only the origin (scheme + host + port) of the requesting page. The server can inspect these headers and reject any request that does not come from a trusted origin.

Here is a simple example of how a server might validate the Referer header:

```python
referer = request.headers.get('Referer', '')
if not referer.startswith('https://www.bank.com'):
    return "CSRF attempt detected", 403
```

Referer and Origin header validation is simple to implement on the server side and can effectively catch many basic CSRF attacks. The `Origin` header is especially useful because it is more reliable and harder for attackers to spoof or strip than the `Referer` header.

However, this approach has several important limitations. The `Referer` header can be missing or deliberately stripped by browsers, privacy tools, proxies, or mobile applications, and some legitimate requests may not include these headers at all. In certain cases, attackers can manipulate or spoof the headers, and overly strict validation can break legitimate user flows such as arriving from search engines or external links. For these reasons, `Referer` and `Origin` header validation is usually used only as a secondary defense (defense-in-depth) rather than the primary protection.

### Other Defenses and Modern Best Practices

While CSRF tokens and SameSite cookies are the primary and most effective defenses, several additional techniques can further strengthen protection against CSRF attacks.

#### Custom Request Headers

Many modern web applications and JavaScript frameworks protect APIs by requiring a custom HTTP header, such as `X-Requested-With: XMLHttpRequest` or a dedicated `X-CSRF-Token` header. Because simple cross-origin requests cannot include custom headers (due to CORS preflight requirements), this technique effectively blocks many CSRF attacks on API endpoints. It works particularly well for AJAX-heavy applications but provides little protection for traditional HTML form submissions.

#### Double-Submit Cookie Pattern

The double-submit cookie pattern is a useful defense for stateless applications. The server generates a random value and sets it as a cookie. The same value is also included in the request body or a custom header when the form is submitted. On the server side, the application compares the value received in the cookie with the value submitted in the request. If the two values match, the request is accepted. Because an attacker cannot set cookies for the target domain (due to the Same-Origin Policy), they cannot forge a matching pair.

#### Short Session Timeouts and Re-authentication

For highly sensitive actions — such as changing a password, making large money transfers, or deleting an account — many applications require the user to re-authenticate by re-entering their password or completing a second factor of authentication. This additional step significantly reduces the practicality of CSRF attacks, even if other defenses are bypassed.
