# Open Redirect

## Summary

An **Open Redirect** vulnerability exists in the application's dashboard routing functionality.

The application accepts user-controlled input as part of the `/dashboard/{any}` route and uses it to construct redirect destinations without validating the target hostname.

If the path segment begins with a double slash (`//`), the application prepends `https:` to the supplied value and issues an HTTP redirect. Because no validation is performed, an attacker can craft a URL on the trusted application domain that redirects victims to an arbitrary external website.

This behavior can be abused in phishing attacks, credential theft campaigns, or to increase the credibility of malicious links by leveraging the application's trusted domain.

---

## OWASP Category

**A03:2021 – Injection / Unvalidated Redirects and Forwards**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | Medium (6.1) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` |

---

## Affected Endpoint

```http
GET /dashboard/{any}
```

---

## Impact

An attacker can craft malicious URLs hosted on the trusted application domain that redirect victims to arbitrary external websites.

Successful exploitation may result in:

- Phishing attacks using trusted application URLs.
- Credential theft by redirecting users to fake login pages.
- Increased credibility of malicious links.
- Delivery of malware or other malicious content.
- User trust abuse through redirection to attacker-controlled websites.

---

## Steps to Reproduce

1. Open a browser or intercept the following request:

```http
GET /dashboard//evil.com HTTP/1.1
Host: localhost:8000
```

2. Send the request.
3. Observe that the server responds with an HTTP **302 Found** status.
4. Inspect the `Location` response header.
5. Observe that the application redirects the browser to the attacker-controlled domain.

---

## Proof of Concept

### Request

```http
GET /dashboard//evil.com HTTP/1.1
Host: localhost:8000
```

### Response

```http
HTTP/1.1 302 Found
Location: https://evil.com
```

The application constructs the redirect destination directly from user-controlled input without validating the target hostname, allowing arbitrary external redirects.

---

## Root Cause

The application constructs absolute redirect URLs using untrusted user input from the request path.

No validation is performed to verify whether the destination belongs to a trusted domain, allowing attackers to redirect users to arbitrary external websites.

---

## Remediation

- Reject any path beginning with `//` by returning **HTTP 400 Bad Request**.
- If external redirects are required, parse the destination using `parse_url()` (or an equivalent API) and validate the hostname against a strict allowlist of trusted domains.
- Never construct redirect destinations directly from user-controlled input.
- Prefer using server-side route names or predefined redirect mappings instead of dynamic URLs.
- Log and monitor unexpected redirect attempts for potential abuse.

---

## References

- OWASP Top 10 2021 – A03: Injection
- OWASP Unvalidated Redirects and Forwards Cheat Sheet
- CWE-601: URL Redirection to Untrusted Site ('Open Redirect')
