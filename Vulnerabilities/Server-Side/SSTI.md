# Server-Side Template Injection (SSTI) Allows Remote Template Execution

## Summary

A **Server-Side Template Injection (SSTI)** vulnerability exists in the application's template rendering functionality.

The application embeds user-controlled input directly into server-side templates without proper sanitization or isolation. As a result, an authenticated attacker can inject template expressions that are evaluated by the underlying template engine.

Depending on the template engine and its configuration, successful exploitation may lead to disclosure of sensitive information, arbitrary file access, remote code execution (RCE), or complete server compromise.

---

## OWASP Category

**A03:2021 – Injection**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | Critical (9.1) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H` |

---

## Affected Endpoints

```http
GET /preview?template=
```

```http
POST /api/render
```

---

## Impact

An authenticated attacker can inject server-side template expressions that are evaluated by the template engine.

Successful exploitation may result in:

- Disclosure of sensitive application data.
- Disclosure of environment variables and configuration secrets.
- Unauthorized access to local files.
- Arbitrary command execution, depending on the template engine.
- Remote Code Execution (RCE).
- Complete compromise of the application server.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Navigate to the template preview functionality:

```http
GET /preview?template=
```

3. Supply a template expression instead of ordinary text.
4. Submit the request.
5. Observe that the template expression is evaluated by the server rather than rendered as plain text.
6. Repeat the test using the rendering API:

```http
POST /api/render
```

7. Observe that user-controlled template expressions are executed by the template engine.

---

## Proof of Concept

### Request

```http
GET /preview?template={{7*7}} HTTP/1.1
Host: localhost:8000
```

### Response

```text
49
```

The server evaluates the template expression instead of returning the literal string `{{7*7}}`, confirming the presence of Server-Side Template Injection.

---

## Root Cause

The application renders untrusted user input directly within server-side templates.

Instead of treating user input as plain text, the template engine interprets attacker-controlled expressions as executable template code, allowing arbitrary template evaluation.

---

## Remediation

- Never render untrusted input as template code.
- Treat all user input as plain text.
- Use safe template rendering APIs that separate data from templates.
- Apply strict server-side input validation.
- Enable template sandboxing where supported by the template engine.
- Avoid exposing sensitive objects, helper functions, or system APIs within the template context.
- Keep template engines and related libraries updated with the latest security patches.

---

## References

- OWASP Top 10 2021 – A03: Injection
- OWASP Server-Side Template Injection Prevention
- CWE-1336: Improper Neutralization of Special Elements Used in a Template Engine
- PortSwigger Web Security Academy – Server-Side Template Injection
