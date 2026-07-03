# Stored Cross-Site Scripting (XSS)

## Summary

A **Stored Cross-Site Scripting (XSS)** vulnerability exists in the **Create Value** functionality.

The application allows authenticated users to submit entity values that are stored in the backend database without adequate server-side validation or sanitization. These values are later rendered on the Dashboard without context-aware output encoding.

Although a Web Application Firewall (WAF) attempts to block common XSS payloads, it can be bypassed using JavaScript's backtick invocation syntax. Once a malicious payload is stored, it is automatically executed whenever an authenticated user visits the Dashboard.

Unlike reflected or interaction-based XSS vulnerabilities, this issue requires **no user interaction** after the malicious value has been stored. Every authenticated user—including administrators—who loads the Dashboard will automatically execute the attacker's JavaScript until the malicious data is removed.

---

## OWASP Category

**A03:2021 – Injection (Cross-Site Scripting)**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | High (8.8) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:L/A:N` |

---

## Affected Endpoints

```http
GET /entity-values/create
```

```http
GET /dashboard?db=hr
```

---

## Impact

An authenticated attacker can permanently store malicious JavaScript that is automatically executed whenever users access the Dashboard.

Successful exploitation may result in:

- Automatic execution of arbitrary JavaScript in victims' browsers.
- Theft of session tokens or other sensitive information.
- Execution of actions on behalf of authenticated users.
- Privilege escalation through compromise of administrative sessions.
- Persistent compromise affecting every authenticated user who visits the Dashboard.
- Defacement or manipulation of application content.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Navigate to:

```http
GET /entity-values/create
```

3. Enter a malicious JavaScript payload as an entity value.
4. Save the entity value.
5. Navigate to:

```http
GET /dashboard?db=hr
```

6. Observe that the stored payload is rendered without output encoding.
7. The JavaScript executes automatically as soon as the Dashboard loads.
8. Repeat the Dashboard visit using another authenticated account (including an administrator) and observe that the payload executes automatically without requiring any interaction.

---

## Proof of Concept

### Malicious Payload

```html
<IMG SRC=x onerror=alert`1`>
```

### Request

```http
POST /entity-values HTTP/1.1
Host: localhost:8000

value=<IMG SRC=x onerror=alert`1`>
```

### Result

The payload is successfully stored in the application's database.

When any authenticated user visits `/dashboard?db=hr`, the stored value is rendered as HTML without output encoding, causing the JavaScript to execute automatically on every page load.

---

## Root Cause

The application stores user-controlled entity values without adequate validation and later renders them directly into an HTML context without context-aware output encoding.

In addition, the WAF relies on signature-based filtering that blocks common payloads such as `alert(` but fails to detect JavaScript template literal invocation (`alert\`1\``), allowing malicious payloads to bypass filtering and become permanently stored.

---

## Remediation

- Sanitize all entity value inputs on the server before storing them.
- Reject HTML and JavaScript input for fields intended to contain plain text.
- Apply context-aware output encoding whenever rendering entity values.
- Implement a restrictive Content Security Policy (CSP) to reduce the impact of successful XSS attacks.
- Update WAF rules to detect JavaScript template literal invocation and other modern XSS bypass techniques.
- Review all locations where entity values are rendered to ensure proper output encoding.

---

## References

- OWASP Top 10 2021 – A03: Injection
- OWASP Cross Site Scripting Prevention Cheat Sheet
- CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')
- CWE-116: Improper Encoding or Escaping of Output
- CWE-20: Improper Input Validation
