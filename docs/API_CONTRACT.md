# Ticket QR Code Generator Worker — API Contract

**Ticket:** ENG-139055  
**Epic:** Core Infrastructure Overhaul  
**Priority:** P1  
**Document Type:** API Architecture / Capstone 1  
**Implementation Status:** Design only

---

## 1. Purpose

This document defines the proposed REST API contract for the Ticket QR Code Generator Worker.

The API is **not implemented in the current Capstone 1 phase**.

The purpose of this document is to establish a stable contract between a future client application and backend service before feature development begins.

---

## 2. API Conventions

### Base URL

Production base URL:

```text
https://<deployment-host>/api/v1
```

Local development example:

```text
http://localhost:5000/api/v1
```

The actual deployment host will be determined during implementation.

### Content Type

Requests containing a body should use:

```http
Content-Type: application/json
```

Responses should use:

```http
Content-Type: application/json
```

---

## 3. Standard Response Envelope

All endpoints should use a consistent response structure.

### Successful response

```json
{
  "success": true,
  "data": {}
}
```

### Error response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fields": {}
  }
}
```

This consistency allows the future frontend to handle API responses through a predictable interface.

---

# 4. Endpoint Summary

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Service health check |
| `POST` | `/tickets` | Create a ticket |
| `GET` | `/tickets/:ticketId` | Retrieve a ticket |
| `GET` | `/tickets` | List/search tickets |
| `POST` | `/tickets/:ticketId/qr` | Generate a QR-code record |
| `GET` | `/qr/:qrId` | Retrieve a QR-code record |
| `GET` | `/tickets/:ticketId/qr` | Retrieve QR generation history for a ticket |

---

# 5. Health Check

## `GET /health`

Checks whether the API service is available.

### Request

No request body.

### Success

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "status": "ok"
  }
}
```

### Failure

**HTTP 503**

```json
{
  "success": false,
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service is temporarily unavailable"
  }
}
```

### Purpose

The endpoint can be used by:

- deployment platforms
- monitoring systems
- developers
- operational health checks

---

# 6. Create Ticket

## `POST /tickets`

Creates a new ticket.

### Request

```json
{
  "ticketNumber": "TKT-10001",
  "title": "Printer installation request",
  "description": "Install a printer in floor 2 reception",
  "status": "OPEN",
  "priority": "MEDIUM",
  "createdBy": 101
}
```

### Required fields

```text
ticketNumber
title
description
status
priority
createdBy
```

### Success

**HTTP 201**

```json
{
  "success": true,
  "data": {
    "id": 1001,
    "ticketNumber": "TKT-10001",
    "title": "Printer installation request",
    "description": "Install a printer in floor 2 reception",
    "status": "OPEN",
    "priority": "MEDIUM",
    "createdBy": 101,
    "createdAt": "2026-09-28T12:00:00.000Z",
    "updatedAt": "2026-09-28T12:00:00.000Z"
  }
}
```

### Validation errors

**HTTP 400**

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fields": {
      "ticketNumber": "Ticket number is required",
      "title": "Title is required"
    }
  }
}
```

### Duplicate ticket number

**HTTP 409**

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_TICKET",
    "message": "A ticket with this ticket number already exists"
  }
}
```

---

# 7. Get Ticket

## `GET /tickets/:ticketId`

Retrieves a specific ticket.

### Example

```http
GET /api/v1/tickets/1001
```

### Success

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "id": 1001,
    "ticketNumber": "TKT-10001",
    "title": "Printer installation request",
    "description": "Install a printer in floor 2 reception",
    "status": "OPEN",
    "priority": "MEDIUM",
    "createdBy": 101,
    "createdAt": "2026-09-28T12:00:00.000Z",
    "updatedAt": "2026-09-28T12:00:00.000Z"
  }
}
```

### Not found

**HTTP 404**

```json
{
  "success": false,
  "error": {
    "code": "TICKET_NOT_FOUND",
    "message": "Ticket not found"
  }
}
```

---

# 8. List / Search Tickets

## `GET /tickets`

Retrieves tickets.

### Basic request

```http
GET /api/v1/tickets
```

### Optional query parameters

```text
status
priority
search
page
limit
```

Example:

```http
GET /api/v1/tickets?status=OPEN&priority=HIGH&page=1&limit=20
```

### Success

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1001,
        "ticketNumber": "TKT-10001",
        "title": "Printer installation request",
        "status": "OPEN",
        "priority": "HIGH"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 1,
      "totalPages": 1
    }
  }
}
```

### Empty result

An empty result is still a successful request.

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "items": [],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 0,
      "totalPages": 0
    }
  }
}
```

The future client must display a user-friendly message such as:

```text
No data found
```

rather than rendering a blank screen.

---

# 9. Generate QR Code

## `POST /tickets/:ticketId/qr`

Creates a QR-code generation record for an existing ticket.

### Request

```json
{
  "generatedBy": 101
}
```

### Success

**HTTP 201**

```json
{
  "success": true,
  "data": {
    "id": 5001,
    "ticketId": 1001,
    "payload": "TICKET:1001",
    "generatedBy": 101,
    "generatedAt": "2026-09-28T12:05:00.000Z"
  }
}
```

### Important design decision

The API returns the **QR payload**, not a stored image.

The eventual frontend can use this payload to render the QR code visually.

### Ticket not found

**HTTP 404**

```json
{
  "success": false,
  "error": {
    "code": "TICKET_NOT_FOUND",
    "message": "Cannot generate QR code because the ticket does not exist"
  }
}
```

### Invalid generator

**HTTP 400**

```json
{
  "success": false,
  "error": {
    "code": "INVALID_USER",
    "message": "The QR code generator is invalid"
  }
}
```

---

# 10. Get QR Code

## `GET /qr/:qrId`

Retrieves a previously generated QR-code record.

### Example

```http
GET /api/v1/qr/5001
```

### Success

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "id": 5001,
    "ticketId": 1001,
    "payload": "TICKET:1001",
    "generatedBy": 101,
    "generatedAt": "2026-09-28T12:05:00.000Z"
  }
}
```

### Not found

**HTTP 404**

```json
{
  "success": false,
  "error": {
    "code": "QR_CODE_NOT_FOUND",
    "message": "QR code record not found"
  }
}
```

---

# 11. QR Generation History

## `GET /tickets/:ticketId/qr`

Returns all QR-code generation records associated with a ticket.

### Example

```http
GET /api/v1/tickets/1001/qr
```

### Success

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 5001,
        "ticketId": 1001,
        "payload": "TICKET:1001",
        "generatedBy": 101,
        "generatedAt": "2026-09-28T12:05:00.000Z"
      },
      {
        "id": 5002,
        "ticketId": 1001,
        "payload": "TICKET:1001",
        "generatedBy": 102,
        "generatedAt": "2026-09-28T12:10:00.000Z"
      }
    ]
  }
}
```

### No QR records

**HTTP 200**

```json
{
  "success": true,
  "data": {
    "items": []
  }
}
```

The future UI should represent this as an empty state rather than a blank page.

---

# 12. HTTP Status Code Convention

| Status | Meaning |
|---|---|
| `200` | Successful read/request |
| `201` | Resource successfully created |
| `400` | Invalid request or validation failure |
| `401` | Authentication required/invalid |
| `403` | Authenticated but not authorized |
| `404` | Requested resource does not exist |
| `409` | Resource conflict, such as duplicate ticket number |
| `429` | Rate limit exceeded |
| `500` | Unexpected server error |
| `503` | Service temporarily unavailable |

Authentication and authorization endpoints are outside the current Capstone 1 scope, but these status codes are reserved for the eventual implementation.

---

# 13. Common Error Contract

All API errors should follow the same structure:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "fields": {}
  }
}
```

`fields` should only be returned when field-level validation information is useful.

Example:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fields": {
      "title": "Title is required",
      "priority": "Priority must be LOW, MEDIUM, HIGH, or CRITICAL"
    }
  }
}
```

---

# 14. Invalid Input Handling

The future API implementation must validate incoming requests before persistence.

Examples include:

### Missing ticket title

```text
POST /tickets
title = ""
```

Expected:

```text
HTTP 400
VALIDATION_ERROR
```

### Unsupported priority

```text
priority = "URGENT123"
```

Expected:

```text
HTTP 400
VALIDATION_ERROR
```

### Invalid ticket ID

```text
GET /tickets/abc
```

Expected:

```text
HTTP 400
INVALID_TICKET_ID
```

### Non-existent ticket

```text
GET /tickets/999999
```

Expected:

```text
HTTP 404
TICKET_NOT_FOUND
```

---

# 15. XSS / Text Sanitization

The future implementation must treat all user-supplied text as untrusted.

Relevant fields include:

```text
name
title
description
search
```

The API/application layer should:

1. Validate input types.
2. Apply appropriate length limits.
3. Sanitize text where required.
4. Never execute user-provided HTML or scripts.
5. Use parameterized queries or an ORM for database operations.
6. Encode output appropriately when rendering user-controlled content.

Example malicious input:

```html
<script>alert("xss")</script>
```

must never be executed by the application.

---

# 16. Slow Connectivity

The client must assume that API requests may occur over slow or unreliable networks.

The future client should:

- display a loading indicator during asynchronous requests
- disable duplicate primary actions while a request is in progress
- show a clear network error when a request fails
- avoid rendering a blank screen
- preserve already loaded data where practical
- allow safe retry of failed operations

The API should return deterministic responses so that retry behavior can be implemented safely.

---

# 17. Idempotency Consideration

QR generation can potentially be triggered multiple times because of network retries or repeated user actions.

During implementation, the API should consider an idempotency mechanism for QR-generation requests.

A future request could include:

```http
Idempotency-Key: <unique-request-id>
```

The backend could then prevent accidental duplicate processing of the same logical request.

This is documented as a design consideration rather than implemented in the current Capstone 1 scope.

---

# 18. Accessibility Considerations

Accessibility is primarily a client concern, but the API contract supports it by returning:

- predictable success responses
- predictable validation errors
- field-specific validation messages
- explicit empty collections
- machine-readable error codes

The future frontend can map these responses to accessible form errors and status messages.

---

# 19. Telemetry Contract

The future client should simulate the required analytics event after the primary action succeeds.

Expected console message:

```text
[Analytics] User interacted with Ticket QR Code Generator Worker
```

This is a client-side telemetry simulation and does not require a dedicated analytics API in the current scope.

---

# 20. Security Considerations

The eventual implementation should:

- authenticate protected operations
- authorize QR generation based on user role/permissions
- validate all input
- sanitize untrusted text
- use parameterized database operations
- avoid returning unnecessary sensitive information
- avoid hardcoding secrets or API keys
- use HTTPS in production
- apply appropriate rate limiting

Authentication implementation is intentionally outside this architecture-only phase.

---

# 21. API-to-Database Mapping

| API Operation | Primary Table(s) |
|---|---|
| Create ticket | `users`, `tickets` |
| Get ticket | `tickets`, `users` |
| List tickets | `tickets` |
| Generate QR | `tickets`, `users`, `qr_codes` |
| Get QR | `qr_codes` |
| QR history | `qr_codes`, `tickets` |
| Health check | No business table |

---

# 22. Scope Boundary

### Included

- REST endpoint definitions
- HTTP methods
- Request/response contracts
- Validation behavior
- Error conventions
- Empty-state behavior
- Connectivity considerations
- Security considerations
- Telemetry considerations
- API/database mapping

### Not included

- Backend source code
- Controllers
- Services
- Routes
- Database migrations
- ORM implementation
- Authentication implementation
- QR-code library integration
- Frontend implementation
- Deployment configuration

This document is an **API architecture contract only**, as required for the current Capstone 1 phase.
