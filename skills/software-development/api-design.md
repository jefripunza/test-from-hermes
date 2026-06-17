---
name: api-design
description: "API design expert: REST, OpenAPI/Swagger, versioning, pagination, filtering, sorting, rate limiting. Standard response format, validation, error strategy."
metadata:
  hermes:
    tags: [api, rest, openapi, swagger, design, backend]
---

# API Design — REST API Expert

REST API design. OpenAPI specs, versioning, pagination, filtering, sorting.

## Expertise

- RESTful API design
- OpenAPI 3.x / Swagger
- API versioning strategies
- Pagination (cursor, offset)
- Filtering & sorting
- Rate limiting
- Validation strategy
- Error handling
- Request/Response examples

## Standard Response Envelope

```json
{
  "success": true,
  "data": {},
  "message": "",
  "meta": {}
}
```

Error response:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "reason": "must be a valid email address"
    }
  }
}
```

## Pagination

```json
// Request: GET /api/users?page=2&per_page=20
// Response:
{
  "success": true,
  "data": [...],
  "meta": {
    "page": 2,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": true
  }
}
```

Cursor-based for real-time:

```json
{
  "data": [...],
  "meta": {
    "cursor": "eyJpZCI6MTIzfQ==",
    "has_more": true
  }
}
```

## Filtering & Sorting

```text
GET /api/users?role=admin&status=active
GET /api/users?search=john
GET /api/users?sort=-created_at,name
```

Prefix `-` for descending. Comma-separated for multiple.

## Versioning

```text
URL path:    /api/v1/users
Header:      Accept: application/vnd.api+json;version=1
```

Prefer URL path — explicit, cache-friendly.

## Always Provide

- Request examples (curl, JSON body)
- Response examples (success + error)
- Validation strategy (what fields, what rules)
- Error strategy (error codes, messages, HTTP status mapping)
- Rate limit headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`)

## HTTP Status Code Usage

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Success response with body |
| 201 | Created | Resource created |
| 204 | No Content | Delete success |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | No auth / invalid token |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate, state conflict |
| 422 | Unprocessable | Validation error |
| 429 | Too Many | Rate limit exceeded |
| 500 | Internal | Server error (don't leak details) |
