# HTTP Cookies

## The `Domain` Attribute

The `Domain` attribute in the `Set-Cookie` response header specifies **which hosts the cookie will be sent to**.

If specified, cookies are available on the specified server and its subdomains, otherwise it will be only sent to the domain that set the cookie.

A server can only set the Domain attribute to its own domain or a parent domain, not to a subdomain or some other domain.

An invalid `Domain` attribute will cause the user agent (e.g., browser) to disregard the cookie altogether.

### Key Points:

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

---

### Behavior of `Domain` Attribute

| Attribute Value | Cookie was set by  | Cookie will be sent to           |
| --------------- | ------------------ | -------------------------------- |
| _(Omitted)_     | `example.org`      | Only `example.org`               |
| `example.org`   | `example.org`      | `example.org` and all subdomains |
| `example.org`   | `auth.example.org` | `example.org` and all subdomains |

---

### Permitted vs. Not Permitted Values

| Request was sent to | **Permitted Values**                         | **Not Permitted Values**          |
| ------------------- | -------------------------------------------- | --------------------------------- |
| `z.com`             | _(Omitted)_, `z.com`                         | `other.com`, `y.z.com`, `com`     |
| `y.z.com`           | _(Omitted)_, `y.z.com`, `z.com`              | `other.com`, `com`                |
| `x.y.z.com`         | _(Omitted)_, `x.y.z.com`, `y.z.com`, `z.com` | `other.com`, `v.x.y.z.com`, `com` |

## The `Path` Attribute

The `Path` attribute in the `Set-Cookie` response header specifies the set of paths that must exist for the browser to send the cookie.

-   **Default Value**: If the `Path` attribute is not set, or does not start with `/`, it defaults to `/`
-   **Subpaths**
    -   The cookie is sent for all subpaths of the specified path (e.g., `/products` will match `/products`, `/products/123`, `/products/123/details`).
    -   If the Path attribute-value is `/auth/`, it will not much `/auth`
