# HTTP Cookies

Cookies can be either session or permanent cookies:

1. **Session Cookies**

    - If a cookie has neither the `Max-Age` nor the `Expires` attributes, then it is a session cookie.
    - Session cookies are deleted when the current session ends.
    - They are also known as _in-memory cookie_, _transient cookie_ or _non-persistent cookie_.

2. **Permanent Cookies**

    - If a cookie has either the `Max-Age` or `Expires` attributes, then it is a permanent cookie.
    - Permanent cookies are deleted after the date specified in the `Expires` attribute or after the period specified in the `Max-Age`.

## Quick Overview

| Cookie Attribute | Key Takeaways                                                                                                                                                                                     | If invalid                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Expires          | Makes the cookie a permanent cookie<br><br>Calculated with respect to Date response header<br><br>Setting to past date removes the cookie<br>                                                     | Attribute is ignored                       |
| Max-Age          | Number of seconds until the cookie expires<br><br>Makes the cookie a permanent cookie<br><br>Has precedence over Expires if both are set<br><br>Setting to 0 or negative value removes the cookie | Attribute is ignored                       |
| Domain           | Can be set to origin or parent domains<br><br>Only sent to the origin if omitted<br><br>Sent to issuing domain and all subdomains if set                                                          | Cookie is discarded                        |
| Path             | Must start with /<br><br>Only sent if the value matches the prefix of the request url                                                                                                             | Attribute is ignored                       |
| Secure           | Only set and sent through HTTPS                                                                                                                                                                   | Cookie is discarded if set over HTTP       |
| HttpOnly         | Cookie is only scoped to HTTP requests<br><br>Cannot be accessed by browser API                                                                                                                   | Cookie is discarded if set by non-HTTP API |

## HTTP Headers

### `Set-Cookie` Header

-   The HTTP `Set-Cookie` is a **forbidden response header** that is used to send cookies. In other words, frontend JavaScript cannot access the header.
-   Multiple cookies can be set using multiple `Set-Cooke` headers.
-   Examples:
    -   `Set-Cookie: sessionId=38afes7a8`
    -   `Set-Cookie: id=a3fWa; Max-Age=2592000`
    -   `Set-Cookie: num=123; Secure; Domain=example.com`

### `Cookie` Header

-   The HTTP `cookie` header is a **forbidden request header** that contains the stored HTTP cookies. In other words, the header cannot be set or modified programmatically in a request.
-   The attributes of the cookies are **not** sent.
-   Multiple cookies with the same name can be present. For example, `Cookie: myCookie=myValue; myCookie=another`.
-   Examples:
    -   `Cookie: cookieName=cookieValue`
    -   `Cookie: theme=light; sid=123`

## The Cookie Name

-   It is required and case-sensitive. The cookie with name `foo` is different than a cookie with name `foO`.
-   The server should not send more than one Set-Cookie header field in the same response with the exact same cookie name.

## Cookie Attributes

### The `Secure` Attribute

The `Secure` attribute limits the scope of the cookie to secure channels. If the attribute is present, the cookie will be sent to the server only if the request is transmitted over TLS.

-   It is an optional attribute, and defaults to not secure.
-   It can only be set and will be only set through HTTPS
-   It provides confidentiality, it does NOT provide integrity.
-   Servers should encrypt and sign the contents of cookie even when sending the cookies over a secure channel.

### The `Domain` Attribute

The `Domain` attribute in the `Set-Cookie` response header specifies **which hosts the cookie will be sent to**.

If specified, cookies are available on the specified server and its subdomains, otherwise it will be only sent to the domain that set the cookie.

A server can only set the Domain attribute to its own domain or a parent domain, not to a subdomain or some other domain.

An invalid `Domain` attribute will cause the user agent (e.g., browser) to disregard the cookie altogether.

-   **Optional Attribute:**
    -   It is an optional attribute. **If `Domain` is omitted**, the cookie is only available to the exact domain that set it.
    -   The cookie is said to be `host-only cookie` in this case. This is the default behaviour.
-   **Case-Insensitive:**
    -   `Domain=EXAMPLE.com` is treated the same as `Domain=example.com`.
-   **Valid `Domain` Values:**

    -   The value can be set to:

        1. **Exact Match:**

            - Example: `Domain=example.org` when the request was sent to `example.org`.

        2. **Parent Domain (excluding top-level domains like `.com`, `.org`, `.net`):**

            - Example: `Domain=example.org` when the request was sent to `auth.example.org`.
            - Example: `Domain=example.org` when the request was sent to `admin.auth.example.org`.

-   **Invalid `Domain` Values:**

    -   **The value cannot be a subdomain.**
        -   ❌ `Domain=auth.example.org` when the request was sent to `example.org` → **Not Allowed**
    -   **The value cannot be an unrelated domain.**
        -   ❌ `Domain=othersite.com` when the request was sent to `example.org` → **Not Allowed**
    -   **The value cannot be a top-level domain (TLD).**
        -   ❌ `Domain=com`, `Domain=org`, `Domain=net` → **Not Allowed**

-   **If `Domain` is set,** the cookie is available to the specified domain and all its subdomains.

**Behavior of `Domain` Attribute**

| Attribute Value | Cookie was set by  | Cookie will be sent to           |
| --------------- | ------------------ | -------------------------------- |
| _(Omitted)_     | `example.org`      | Only `example.org`               |
| `example.org`   | `example.org`      | `example.org` and all subdomains |
| `example.org`   | `auth.example.org` | `example.org` and all subdomains |

**Permitted vs. Not Permitted Values**

| Request was sent to | **Permitted Values**                         | **Not Permitted Values**          |
| ------------------- | -------------------------------------------- | --------------------------------- |
| `z.com`             | _(Omitted)_, `z.com`                         | `other.com`, `y.z.com`, `com`     |
| `y.z.com`           | _(Omitted)_, `y.z.com`, `z.com`              | `other.com`, `com`                |
| `x.y.z.com`         | _(Omitted)_, `x.y.z.com`, `y.z.com`, `z.com` | `other.com`, `v.x.y.z.com`, `com` |

### The `Path` Attribute

The `Path` attribute in the `Set-Cookie` response header specifies the set of paths that must exist for the browser to send the cookie.

-   **Default Value**: If the `Path` attribute is not set, or does not start with `/`, it defaults to `/`
-   **Subpaths**
    -   The cookie is sent for all subpaths of the specified path (e.g., `/products` will match `/products`, `/products/123`, `/products/123/details`).
    -   If the Path attribute-value is `/auth/`, it will not much `/auth`
