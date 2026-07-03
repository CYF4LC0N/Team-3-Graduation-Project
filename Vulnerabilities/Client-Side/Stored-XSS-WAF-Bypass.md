# Stored Cross-Site Scripting (XSS) with WAF Bypass

## Summary

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Attributes** functionality.

The application allows authenticated users to create custom attribute names, which are later rendered in multiple locations throughout the application without proper output encoding.

Although a Web Application Firewall (WAF) attempts to block common XSS payloads containing `alert(`, it fails to account for JavaScript's tagged template literal syntax. As a result, an attacker can bypass the WAF and store a malicious payload that executes whenever another user interacts with the affected functionality.

Once the malicious attribute is stored, visiting the **Create Value** page and selecting the corresponding attribute causes the payload to be rendered as HTML, resulting in arbitrary JavaScript execution in the victim's browser.

---

## OWASP Category

**A03:2021 – Injection (Cross-Site Scripting)**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | High (8.2) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:L/A:N` |

---

## Affected Endpoints

```http
GET /attributes
```

```http
GET /entity-values/create
```

---

## Impact

An authenticated attacker can store malicious JavaScript that executes in the browsers of other users interacting with the affected functionality.

Successful exploitation may result in:

- Execution of arbitrary JavaScript in victims' browsers.
- Theft of session tokens or sensitive information.
- Actions performed on behalf of authenticated users.
- Defacement or manipulation of application content.
- Privilege escalation depending on the victim's permissions.
- Compromise of administrative accounts if an administrator triggers the payload.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Navigate to:

```http
GET /attributes
```

3. Create a new attribute.
4. As the **Attribute Name**, supply the following payload:

```html
<IMG SRC=x onerror=alert`1`>
```

5. Save the attribute.
6. Navigate to:

```http
GET /entity-values/create
```

7. Locate the malicious attribute and tick its checkbox.
8. Observe that the payload is rendered as HTML and JavaScript executes immediately.

---

## Proof of Concept

### Malicious Payload

```html
<IMG SRC=x onerror=alert`1`>
```

### Request

```http
POST /attributes HTTP/1.1
Host: localhost:8000

name=<IMG SRC=x onerror=alert`1`>
type=text
```

### Result

The payload is successfully stored despite the presence of the WAF.

When another user visits `/entity-values/create` and selects the malicious attribute, the browser executes the injected JavaScript.

---

## Root Cause

The application stores user-controlled input without adequate validation and later renders it into an HTML context without proper output encoding.

Additionally, the WAF relies on signature-based filtering that blocks common payloads such as `alert(` but fails to detect JavaScript template literal invocation (`alert\`1\``), allowing the payload to bypass filtering entirely.

---

## Remediation

- Implement strict server-side validation for **Attribute Name** values.
- Treat attribute names as plain text and reject HTML or JavaScript input.
- Reject input containing dangerous characters such as `<`, `>`, `"`, `'`, and `` ` `` where appropriate.
- Apply context-aware output encoding whenever rendering user-controlled data.
- Do not rely on the WAF as the primary defense against XSS.
- Implement a restrictive Content Security Policy (CSP) to reduce the impact of successful XSS attacks.
- Review all locations where attribute names are rendered to ensure consistent output encoding.

---

## References

- OWASP Top 10 2021 – A03: Injection
- OWASP Cross Site Scripting Prevention Cheat Sheet
- CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
- CWE-116: Improper Encoding or Escaping of Output
