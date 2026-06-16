---
name: rayu-api-design
description: Conventions for designing clean, consistent HTTP/REST APIs — resources, request/response schemas, status codes, error envelopes, pagination, versioning, auth-per-route, and idempotency. Use when defining a backend contract. Pairs with the backend-design subagent.
---

# Rayu API Design

Design the contract before the implementation. A good API is predictable: a developer who has seen three endpoints can guess the fourth.

## 1. Resources and routes

- Model **nouns**, not verbs: `/users`, `/users/{id}`, `/users/{id}/orders`.
- Use HTTP methods for the verb: `GET` (read, safe), `POST` (create), `PUT`/`PATCH` (replace/update), `DELETE` (remove).
- Plural collection names; lowercase, hyphenated paths. Nest only one level deep — prefer `/orders?userId=…` over deep nesting.
- Keep responses consistent in shape across endpoints.

## 2. Request / response schemas

Define every field with name, type, required/optional, and validation. Keep wire names consistent (pick `camelCase` or `snake_case` and never mix). Echo created/updated resources back in the response so the client doesn't need a second round-trip.

```http
POST /api/v1/users
Content-Type: application/json

{ "email": "a@example.com", "password": "••••••", "name": "Ada" }

201 Created
{ "id": "usr_123", "email": "a@example.com", "name": "Ada", "createdAt": "2026-01-01T00:00:00Z" }
```

## 3. Status codes (use them precisely)

- `200` OK, `201` Created (+ `Location`), `204` No Content (deletes).
- `400` malformed, `401` unauthenticated, `403` authenticated-but-forbidden, `404` not found, `409` conflict, `422` validation failed, `429` rate-limited.
- `500` server error (never leak internals), `503` unavailable.

## 4. One error envelope everywhere

Pick a single error shape and use it for *every* error:

```json
{ "error": { "code": "validation_failed",
             "message": "Email is already registered.",
             "details": [{ "field": "email", "issue": "duplicate" }] } }
```

Machine-readable `code`, human `message`, optional `details`. Never return HTML errors from a JSON API.

## 5. Pagination, filtering, sorting

- Cursor pagination for large/append-heavy sets (`?limit=20&cursor=…`), offset for small/admin lists (`?page=2&pageSize=20`).
- Return pagination metadata (`nextCursor` or `total`/`page`).
- Standardize filtering (`?status=active`) and sorting (`?sort=-createdAt`).

## 6. Versioning

Version from day one: `/api/v1/…` (or an `Accept` header). Only break within a new version; add fields additively within a version (new optional fields aren't breaking).

## 7. Auth per route (ties into security)

For each endpoint declare: public vs authenticated, and the role/permission required. Use `Authorization: Bearer <token>`; return `401` (no/invalid token) vs `403` (valid token, insufficient role). Keep the auth model centralized (middleware/guards), not per-handler. The `security` collaborator owns the auth decisions — design routes to fit them.

## 8. Idempotency & safety

- `GET`/`PUT`/`DELETE` are idempotent; `POST` is not — for money/critical creates, accept an `Idempotency-Key` header and dedupe.
- Validate and normalize all input at the boundary; never trust the client.
- Rate-limit public/auth endpoints; return `429` + `Retry-After`.

## Working in the rayu swarm

The `backend-design` subagent emits the API contract + data model PRD using these conventions; the `backend` collaborator implements against it, and the `frontend`/`mobile` collaborators code their API layer to the same shapes. Keep the contract in `.rayu/swarm/` so every domain agrees on it.
