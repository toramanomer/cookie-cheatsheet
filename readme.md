For personal use as a quick reference and notes on HTTP cookies.

# Table of Contents

1. [HTTP Cookies](#http-cookies)
2. [Quick Overview](#quick-overview)
3. [HTTP Headers](#http-headers)
    - [`Set-Cookie` Header](#set-cookie-header)
    - [`Cookie` Header](#cookie-header)
4. [The Cookie Name](#the-cookie-name)
    - [Name Prefix](#name-prefix)
5. [Cookie Attributes](#cookie-attributes)
    - [The `Secure` Attribute](#the-secure-attribute)
    - [The `Domain` Attribute](#the-domain-attribute)
    - [The `Path` Attribute](#the-path-attribute)

# HTTP Cookies

Cookies can be either session or permanent cookies:

1. **Session Cookies**

    - If a cookie has neither the `Max-Age` nor the `Expires` attributes, then it is a session cookie.
    - Session cookies are deleted when the current session ends.
    - They are also known as _in-memory cookie_, _transient cookie_ or _non-persistent cookie_.

2. **Permanent Cookies**

    - If a cookie has either the `Max-Age` or `Expires` attributes, then it is a permanent cookie.
    - Permanent cookies are deleted after the date specified in the `Expires` attribute or after the period specified in the `Max-Age`.

The `Path` and `Domain` attributes must match the values used when the cookie was created when removing a cookie or changing the value of an existing cookie.

The cookie can be be removed by sending a Set-Cookie header by setting the value of the `Expires` attribute to a date in the past or by setting the value of the `Max-Age` attribute to zero or negative number, given that the name, domain and path match.

## Quick Overview

| Cookie Attribute | Key Takeaways                                                                                                                                                                                     | If invalid                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `Expires`        | Makes the cookie a permanent cookie<br><br>Calculated with respect to Date response header<br><br>Setting to past date removes the cookie<br>                                                     | Attribute is ignored                       |
| `Max-Age`        | Number of seconds until the cookie expires<br><br>Makes the cookie a permanent cookie<br><br>Has precedence over Expires if both are set<br><br>Setting to 0 or negative value removes the cookie | Attribute is ignored                       |
| `Domain`         | Can be set to origin or parent domains<br><br>Only sent to the origin if omitted<br><br>Sent to issuing domain and all subdomains if set                                                          | Cookie is discarded                        |
| `Path`           | Must start with /<br><br>Only sent if the value matches the prefix of the request url                                                                                                             | Attribute is ignored                       |
| `Secure`         | Only set and sent through HTTPS                                                                                                                                                                   | Cookie is discarded if set over HTTP       |
| `HttpOnly`       | Cookie is only scoped to HTTP requests<br><br>Cannot be accessed by browser API                                                                                                                   | Cookie is discarded if set by non-HTTP API |

| Cookie Name Prefix | Cookie Attribute Requirements       |
| ------------------ | ----------------------------------- |
| `__Secure-`        | `Secure`                            |
| `__Host-`          | `Secure`<br>`Path=/`<br>No `Domain` |

## HTTP Headers

### `Set-Cookie` Header

-   The HTTP `Set-Cookie` is a **forbidden response header** that is used to send cookies. In other words, frontend JavaScript cannot access the header.
-   Multiple cookies can be set using multiple `Set-Cooke` headers. Multiple Set-Cookie header fields must not be combined into a single header field.
-   Examples:
    -   `Set-Cookie: sessionId=38afes7a8`
    -   `Set-Cookie: id=a3fWa; Max-Age=2592000`
    -   `Set-Cookie: num=123; Secure; Domain=example.com`

### `Cookie` Header

-   The HTTP `cookie` header is a **forbidden request header** that contains the stored HTTP cookies. In other words, the header cannot be set or modified programmatically in a request.
-   The attributes of the cookies are **not** sent.
-   Multiple cookies with the same name can be present. For example, `Cookie: myCookie=myValue; myCookie=another`. (Because they could have been set with different domain-path attribute combinations.)
-   Examples:
    -   `Cookie: cookieName=cookieValue`
    -   `Cookie: theme=light; sid=123`

## The Cookie Name

-   It is required and case-sensitive. The cookie with name `foo` is different than a cookie with name `foO`.
-   It must not be empty.
-   The server should not send more than one Set-Cookie header field in the same response with the exact same cookie name.

### Name Prefix

As user agents do not send the cookies with attributes, it is impossible for a server to have confidence that a given cookie was set with a particular set of attributes.
In order to provide such confidence, two cooke name prefixes are defined, both of which require case-sensitive match.

1. `__Secure-`:

    Ensures that the cookie was set with `Secure` attribute.

2. `__Host-`:

    Ensures that the cookie was set with `Secure` attribute, `Path` as `/` and no `Domain` attribute, which would mean that the cookie was set by the same domain, and not any subdomains!

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
