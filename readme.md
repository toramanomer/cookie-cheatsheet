# HTTP Cookies

## The `Domain` Attribute

The `Domain` attribute in the `Set-Cookie` response header specifies **which hosts the cookie will be sent to**.

An invalid `Domain` attribute will cause the user agent (e.g., browser) to disregard the cookie altogether.

### Key Points:

-   **Optional Attribute:**
    -   It is an optional attribute. **If `Domain` is omitted**, the cookie is only available to the exact domain that set it.
-   **Case-Insensitive:**
    -   `Domain=EXAMPLE.com` is treated the same as `Domain=example.com`.
-   **Valid `Domain` Values:**

    -   The value must be a **domain** that the current host is **a part of**.
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
