---
title: Same-Origin Policy
parent: Web Security
nav_order: 3
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Same-Origin Policy

## 1. Motivation: Why Browsers Need Isolation

The modern web runs untrusted code from many different sources in the same browser. Without strong isolation, a malicious website could read your data from other sites, steal your session cookies, or perform actions on your behalf, for example, transferring money from your bank account while you have another tab open.

The **Same-Origin Policy (SOP)** is the browser’s fundamental security mechanism. At its core, SOP is an isolation and access control philosophy designed to protect data connected to one host from being accessed or manipulated by another (possibly malicious) host.

It allows websites to load useful third-party content such as images, fonts, analytics scripts, and widgets, while sandboxing that content. The basic idea is to treat anything coming from a different origin as if it were running in its own separate browser tab, completely isolated from the rest of the page. (We will define what an _origin_ is shortly.) This policy creates a clear boundary between websites: even if many pages are open at the same time (in different tabs, windows, or inside `<iframe>` elements), a page from `evil.com` cannot access or manipulate the data of `good.com` (or any other site).

## 2. What Is an Origin?

An **origin** is the fundamental unit of isolation in the browser. The Same-Origin Policy uses origins to decide what web content is allowed to interact with other web content.

The origin of a webpage or resource is defined by a tuple consisting of three parts:

**Origin = (scheme, host, port)**

- **Scheme** (also called protocol): Usually `http` or `https`. Other schemes like `ftp` or `file` also exist.
- **Host** (also called domain or hostname): The domain name or IP address of the server (e.g., `www.example.com` or `192.168.1.1`).
- **Port**: The network port the server is listening on. If no port is specified in the URL, the browser uses the **default port** for the scheme:
  - `http` → port 80
  - `https` → port 443

Two resources have the **same origin** only if **all three** components are exactly identical. Even a small difference in any one component means they have **different origins**.

Here are some common examples that illustrate whether two resources have the same origin or not:

- `http://example.com` ≠ `https://example.com` (different scheme)
- `http://example.com` ≠ `http://www.example.com` (different host)
- `http://example.com:80` = `http://example.com` (default port is implied)
- `http://example.com:8080` ≠ `http://example.com` (explicit non-default port)

It is very important to understand that the following parts of a URL are **not** included in the origin:

- Path (`/page.html`, `/api/users`)
- Query parameters (`?id=123&sort=desc`)
- Fragment / anchor (`#section-2`)
- Username and password (`user:pass@`)

This means the following two URLs have the **same origin**:

- `https://www.example.com/page1`
- `https://www.example.com/page2?user=alice#top`

## 3. Same Origin or Not? (Examples)

Determining whether two URLs have the same origin is a fundamental skill in web security. Let’s practice with some examples.

| URL A                           | URL B                     | Same Origin? | Reason                                        |
| ------------------------------- | ------------------------- | ------------ | --------------------------------------------- |
| `http://wikipedia.org/a/`       | `http://wikipedia.org/b/` | **Yes**      | Same scheme, host, and port (path is ignored) |
| `http://wikipedia.org`          | `https://wikipedia.org`   | **No**       | Different scheme (`http` vs `https`)          |
| `http://example.com`            | `http://www.example.com`  | **No**       | Different host                                |
| `http://example.com`            | `http://example.com:80`   | **Yes**      | Port 80 is the default for HTTP               |
| `http://example.com:8080`       | `http://example.com`      | **No**       | Explicit non-default port                     |
| `https://sub.example.com`       | `https://example.com`     | **No**       | Different host (subdomain vs parent)          |
| `http://192.168.1.100`          | `http://192.168.1.100:80` | **Yes**      | Default port                                  |
| `http://localhost`              | `http://localhost:3000`   | **No**       | Different port                                |
| `https://user:pass@example.com` | `https://example.com`     | **Yes**      | Username/password is not part of the origin   |

## 4. How SOP Applies to Different Web Features

The Same-Origin Policy does not apply uniformly to everything. Different parts of the web platform have slightly different rules. Understanding these nuances is essential for both developers and security researchers.

### JavaScript and the DOM

JavaScript executes with the **origin of the page that loads it**, not the origin of the script file itself. This means that even if a webpage on `https://example.com` loads a script from a completely different domain (for example, `<script src="https://cdn.example.com/app.js">`), that script still runs with the origin of `https://example.com`.

This design has important security implications for developers. When building a web application, developers often include many third-party libraries (such as jQuery, React, analytics tools, or fonts from CDNs). All of these scripts run with the **same origin** as the main page. Therefore, if any one of these libraries is malicious or becomes compromised later, it gains full access to the page’s DOM, cookies, and other sensitive data, as if it were part of the website itself.

A key rule of the Same-Origin Policy is that JavaScript from one origin cannot read or modify the DOM of a page with a different origin.
For example, a malicious script running on `https://evil.com` cannot access or read the contents of your inbox on `https://gmail.com`, even if you have both pages open in the same browser. The browser strictly enforces this boundary to prevent cross-site data theft.

### Storage APIs

Modern browsers provide several client-side storage mechanisms that allow websites to save data locally on the user’s device. These storage APIs are strictly governed by the Same-Origin Policy:

- `localStorage`: A simple key-value store that persists data even after the browser is closed (until manually cleared).
- `sessionStorage`: Similar to `localStorage`, but the data is automatically cleared when the tab or browser window is closed.
- `IndexedDB`: A more powerful, database-like storage system for storing larger amounts of structured data.
- `Cache API`: Allows websites (especially Progressive Web Apps) to store copies of files and resources for offline use.
- `Service Workers`: Scripts that run in the background and control caching, push notifications, and offline functionality (their registration and scope are origin-restricted).

This means that a webpage on `https://example.com` cannot read or write data stored by `https://other.com`, and vice versa. Each origin gets its own isolated storage space. This design is critical for security — it prevents a malicious site from accessing sensitive data (such as authentication tokens or user preferences) stored by legitimate websites.

### Network Requests (`fetch` and `XMLHttpRequest`)

By default, the browser **blocks** cross-origin network requests. For example, a webpage on `https://example.com` cannot use `fetch()` or `XMLHttpRequest` to read data from `https://api.other.com` unless the target server explicitly permits the request using **CORS** (Cross-Origin Resource Sharing). This restriction prevents a malicious website from silently reading sensitive information from other origins.

### Iframes and Frames

An `<iframe>` creates a new browsing context with its own independent origin, determined by the URL of the page loaded inside the iframe. Even when an iframe is embedded in another page, the parent page and the iframe are treated as having separate origins (unless their URLs are exactly the same).

This separation means the parent page cannot directly read from or modify the DOM of the iframe, and the iframe cannot access the parent page’s DOM. This strong boundary is known as **frame isolation**. It prevents malicious websites from embedding legitimate sites (such as your banking page) inside hidden iframes and stealing information from them.

### Images, CSS, Fonts, and Other Subresources

Many types of subresources can be loaded from other origins, but they are subject to important restrictions under the Same-Origin Policy.
Images can generally be loaded cross-origin, but the browser prevents JavaScript from reading their pixel data (for example, using the `<canvas>` element) unless the image server explicitly allows it by sending the proper CORS headers. For example, suppose a page on `https://example.com` includes an image from another domain:

```html
<img src="https://photos.cdn.com/vacation-beach.jpg" />
```

The browser will successfully download and display the image. In this case, the image itself has the origin of `https://photos.cdn.com`. However, JavaScript running on `https://example.com` is not allowed to read the pixel data of this image (for instance, by drawing it onto a `<canvas>` element) unless the server at `photos.cdn.com` explicitly permits it through proper CORS headers.

Similarly, CSS stylesheets and web fonts can usually be loaded from other origins. However, they often require the target server to send appropriate CORS headers, especially in the case of fonts, to prevent certain types of information leakage.

Other subresources, such as videos, audio files, and favicons, generally follow similar rules: they can be loaded cross-origin for display or playback, but JavaScript access to their content is restricted.
