# Unauthenticated Exposure of Elasticsearch REST API Leading to Unauthorized Read/Write Access

## Summary

An **Elasticsearch Security Misconfiguration** vulnerability exists where the Elasticsearch REST API is publicly exposed on port `9200` without requiring authentication or authorization.

Any attacker capable of reaching the service can directly interact with the Elasticsearch instance without possessing valid application credentials. This allows unauthorized users to enumerate indices, read indexed documents, modify existing documents, create new ones, and delete data.

Since the application dashboard retrieves its data directly from Elasticsearch, unauthorized modifications become immediately visible to application users until the synchronization process restores the affected index.

---

## OWASP Category

**A05:2021 – Security Misconfiguration**

---

## Risk Rating

| Metric | Value |
|---------|-------|
| **Severity** | High (8.6) |
| **CVSS Vector** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:L` |

---

## Affected Endpoint

```http
GET http://localhost:9200/
```

---

## Impact

An unauthenticated attacker can directly access the Elasticsearch REST API and perform unauthorized operations against the backend datastore.

Successful exploitation may result in:

- Enumeration of Elasticsearch indices.
- Disclosure of sensitive indexed documents.
- Unauthorized
