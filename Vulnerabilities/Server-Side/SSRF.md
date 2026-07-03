# Server-Side Request Forgery (SSRF) Allows Access to Internal Network Resources

## Summary

A **Server-Side Request Forgery (SSRF)** vulnerability exists in the application's URL fetching functionality.

The application accepts user-controlled URLs and performs server-side requests without adequately validating the destination. As a result, an authenticated attacker can force the server to issue requests to unintended locations, including internal services, localhost, private network resources, and cloud metadata endpoints that are not directly accessible from the external network.

Depending on the environment, successful exploitation may expose sensitive information, facilitate lateral movement, or enable further attacks against internal infrastructure.

---

## OWASP Category

**A10:2021 – Server-Side Request Forgery (SSRF)**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | High (8.6) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:L/A:L` |

---

## Affected Endpoints

```http
GET /fetch?url=
```

```http
GET /api/proxy?url=
```

---

## Impact

An authenticated attacker can abuse the application as a proxy to send requests to arbitrary destinations from the server.

Successful exploitation may result in:

- Access to internal network services.
- Disclosure of sensitive information from internal applications.
- Access to cloud metadata services and instance credentials.
- Circumvention of network segmentation.
- Enumeration of internal infrastructure.
- Potential chaining with additional vulnerabilities to further compromise the environment.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Navigate to the URL fetching functionality.
3. Supply a URL pointing to an internal resource, for example:

```text
http://127.0.0.1:8000/
```

or

```text
http://169.254.169.254/
```

4. Submit the request.
5. Observe that the application performs the request on behalf of the user.
6. Verify that the server successfully retrieves the response from the supplied destination.

---

## Proof of Concept

### Request

```http
GET /fetch?url=http://127.0.0.1:8000 HTTP/1.1
Host: localhost:8000
```

### Response

```http
HTTP/1.1 200 OK

<!DOCTYPE html>
<html>
...
</html>
```

The application successfully performs a server-side request to a user-controlled destination, demonstrating that arbitrary URLs can be requested without sufficient validation. Depending on the deployment environment, this behavior may allow access to internal services, cloud metadata endpoints, or other restricted resources.

---

## Root Cause

The application performs server-side requests using user-controlled URLs without validating the destination or restricting access to trusted resources.

The absence of destination validation and network restrictions allows attackers to abuse the server as a proxy for accessing internal or otherwise inaccessible resources.

---

## Remediation

- Implement a strict allowlist of permitted hosts, domains, and protocols.
- Block requests to localhost, loopback addresses, private IP ranges, and cloud metadata services.
- Disable unnecessary URL schemes (such as `file://`, `gopher://`, and similar protocols) where supported.
- Validate and sanitize all user-supplied URLs before processing.
- Enforce network-level egress filtering to restrict outbound connections.
- Avoid making server-side requests directly from untrusted user input whenever possible.
- Log and monitor outbound requests for suspicious activity.

---

## References

- OWASP Top 10 2021 – A10: Server-Side Request Forgery (SSRF)
- OWASP SSRF Prevention Cheat Sheet
- CWE-918: Server-Side Request Forgery (SSRF)
