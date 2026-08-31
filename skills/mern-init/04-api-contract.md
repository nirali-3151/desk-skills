# API contract

How the server returns data and status, and how the client reads it.

See also: [01-project-structure.md](01-project-structure.md) · [02-client-structure.md](02-client-structure.md) · [03-server-structure.md](03-server-structure.md)

---

## Who does what

Helpers: `src/utils/apiResponse.ts`  
`AppError`: `src/errors/AppError.ts`  
Error middleware: `src/middleware/error.ts`

Never `res.status().json()` in services or repositories.  
Never return a raw Mongoose doc or a bare array.  
Never send password, hashes, or stack traces.

---

## Response shape

**Success**

```json
{ "success": true, "data": {}, "message": "Login successful" }
```

**List** (optional `meta`)

```json
{ "success": true, "data": [], "meta": { "page": 1, "limit": 20, "total": 100 } }
```

**Error**

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [{ "field": "email", "message": "Required" }]
  }
}
```

**Helpers**

- `ok(res, data, message?)` → `200`
- `created(res, data, message?)` → `201`
- `noContent(res)` → `204`

**Errors:** `throw new AppError(statusCode, code, message, details?)`  
Middleware: `res.status(err.statusCode).json({ success: false, error })`  
Unknown errors → `500`.

---

## HTTP status

HTTP status is the source of truth. JSON is extra detail.

| Status | When | Who sets it |
|---|---|---|
| `200` | Read / update / login | `ok()` |
| `201` | Create / register | `created()` |
| `204` | Delete, no body | `noContent()` |
| `400` | Bad input, Zod fail | `AppError` / validation middleware |
| `401` | Missing or invalid token | `AppError` |
| `403` | Authenticated but not allowed | `AppError` |
| `404` | Resource missing | `AppError` |
| `409` | Duplicate (email taken) | `AppError` |
| `500` | Uncaught / DB crash | error middleware |

2xx must pair with `success: true`.  
4xx/5xx must pair with `success: false`.

---

## Client handling

`shared/lib/api.ts`:

- Use `response.ok` (2xx) as success. Do not trust `success` alone.
- Success → return `data`.
- Failure → throw `{ status, code, message, details }`.
- `401` → logout / send to login.
