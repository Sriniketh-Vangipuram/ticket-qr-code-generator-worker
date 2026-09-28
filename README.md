# Ticket QR Code Generator Worker

**Ticket:** ENG-139055
**Epic:** Core Infrastructure Overhaul
**Priority:** P1
**Story Points:** 5
**Owner:** Vangipuram Sriniketh [PDIT-INT-1097]

## Overview

The Ticket QR Code Generator Worker is a proposed digital workflow for staff who need to generate QR-code records for operational tickets.

The current client requirement is for **Capstone 1 architectural planning only**. Therefore, this repository defines the database architecture and API contract without implementing frontend or backend features.

The goal of this phase is to establish a clear, consistent foundation that can be implemented later without redesigning the core data model or API boundaries.

---

## Current Scope

### Included

* Relational database schema
* Entity Relationship Diagram (ERD)
* Primary and foreign key definitions
* Relationships and cardinality
* Data integrity rules
* Recommended database indexes
* REST API contract
* Request/response structures
* HTTP status-code conventions
* Validation and error behavior
* Empty-state behavior
* Connectivity considerations
* Security considerations
* Accessibility considerations
* Telemetry requirements
* AI prompt traceability

### Not Included

* Frontend source code
* Backend source code
* Database migrations
* ORM configuration
* QR-code generation library
* Authentication implementation
* Production database provisioning
* Application deployment
* Automated feature implementation

This scope follows the Technical Requirements Document instruction:

> "No feature code yet—just architectural planning."

---

## Architecture

The proposed domain model contains three core entities:

```text
USER
  │
  ├──────────────< TICKET
  │                   │
  │                   │
  └──────────────< QR_CODE
                       │
                       │
TICKET ────────────────┘
```

### Relationships

```text
USER 1 ─────────── N TICKET

USER 1 ─────────── N QR_CODE

TICKET 1 ──────── N QR_CODE
```

### Core entities

#### Users

Represents staff members interacting with the worker.

Important attributes:

* `id`
* `name`
* `email`
* `role`
* `created_at`
* `updated_at`

#### Tickets

Represents operational tickets.

Important attributes:

* `id`
* `ticket_number`
* `title`
* `description`
* `status`
* `priority`
* `created_by`
* `created_at`
* `updated_at`

#### QR Codes

Represents QR-code generation events.

Important attributes:

* `id`
* `ticket_id`
* `payload`
* `generated_by`
* `generated_at`

The design stores the **QR payload rather than a generated image**. This keeps the relational database focused on business data and allows a future client application to render the QR code from the payload.

---

## Repository Structure

```text
ticket-qr-code-generator-worker/
│
├── docs/
│   ├── ERD.md
│   └── API_CONTRACT.md
│
├── PROMPTS.md
├── README.md
└── .gitignore
```

---

## Documentation

### Database Architecture

See:

`docs/ERD.md`

The ERD document contains:

* entity definitions
* columns and data types
* primary keys
* foreign keys
* unique constraints
* relationships
* cardinality
* recommended indexes
* data integrity rules
* security considerations
* auditability considerations
* future extensibility

### API Contract

See:

`docs/API_CONTRACT.md`

The API contract contains the proposed endpoints for a future implementation.

| Method | Endpoint                       | Purpose                        |
| ------ | ------------------------------ | ------------------------------ |
| `GET`  | `/api/v1/health`               | Service health check           |
| `POST` | `/api/v1/tickets`              | Create a ticket                |
| `GET`  | `/api/v1/tickets/:ticketId`    | Retrieve a ticket              |
| `GET`  | `/api/v1/tickets`              | List/search tickets            |
| `POST` | `/api/v1/tickets/:ticketId/qr` | Generate a QR-code record      |
| `GET`  | `/api/v1/qr/:qrId`             | Retrieve a QR-code record      |
| `GET`  | `/api/v1/tickets/:ticketId/qr` | Retrieve QR generation history |

These are **contracts only** and are not implemented in this repository.

---

## Requirements Mapping

### Happy Path

| Requirement                  | Architectural support                                              |
| ---------------------------- | ------------------------------------------------------------------ |
| Clear access to worker       | Defined future API boundary and domain model                       |
| Immediate response to inputs | API contract uses predictable request/response structures          |
| Consistent data              | Relational schema, constraints, foreign keys, and validation rules |

### Unhappy Path

| Requirement      | Architectural support                                                                |
| ---------------- | ------------------------------------------------------------------------------------ |
| Empty states     | API returns successful empty collections instead of ambiguous responses              |
| Bad connectivity | API contract defines deterministic responses and client loading/error considerations |
| Invalid inputs   | Field-level validation error contract is defined                                     |

### Non-Functional Requirements

| Requirement    | Architectural support                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Accessibility  | Predictable API responses and field-level errors support accessible client states                                                                 |
| Telemetry      | Required analytics message is documented as a future client-side event                                                                            |
| XSS protection | Input validation and sanitization responsibilities are documented                                                                                 |
| Security       | Authentication, authorization, validation, parameterized DB access, HTTPS, and rate limiting are identified as future implementation requirements |

---

## Error Handling Contract

The proposed API uses a consistent error envelope:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fields": {
      "title": "Title is required"
    }
  }
}
```

This allows a future frontend to map backend validation errors to individual fields.

---

## Empty State Contract

An empty collection is treated as a valid response rather than an error.

Example:

```json
{
  "success": true,
  "data": {
    "items": []
  }
}
```

The future client should display:

```text
No data found
```

instead of rendering a blank screen.

---

## Security Principles

The future implementation should:

* validate all incoming data
* sanitize untrusted text
* prevent XSS execution
* use parameterized queries or an ORM
* authenticate protected operations
* authorize QR-generation actions
* avoid hardcoded secrets
* use HTTPS in production
* apply appropriate rate limiting
* avoid exposing unnecessary sensitive information

Security implementation is outside the current Capstone 1 scope.

---

## QR Generation Design

The database stores the logical QR payload:

```text
TICKET:1001
```

rather than a QR image.

A future client can transform the payload into a visual QR code.

This separation provides:

* smaller database records
* easier regeneration
* simpler API responses
* separation between business data and presentation
* preservation of QR generation history

Multiple QR records may exist for the same ticket so that regeneration events are not silently overwritten.

---

## Connectivity Considerations

The client requirement specifies that users may have unreliable connectivity.

The future application should therefore provide:

* visible loading indicators
* clear network-error messages
* safe retry behavior
* duplicate-action prevention
* preserved previously loaded data where practical
* explicit empty states

The API contract provides deterministic response and error structures to support these behaviors.

---

## Design System Considerations

The eventual user interface should follow the requested monochromatic corporate design system.

The current repository contains no UI implementation.

When UI implementation begins, the design should use consistent spacing based around the requested 16px/32px scale and avoid arbitrary or rogue colors.

---

## Testing Strategy

The Technical Requirements Document requests a TDD workflow for the eventual implementation.

Because this repository is currently architecture-only, executable tests are not included.

The future implementation should begin with tests covering:

### Happy path

* worker loads successfully
* valid ticket data is accepted
* QR generation succeeds
* API response follows the defined contract
* telemetry event is emitted after the primary action

### Unhappy path

* empty ticket collection
* slow asynchronous operation
* failed network request
* missing required fields
* malformed input
* invalid ticket ID
* non-existent ticket
* duplicate ticket number
* XSS-style malicious input
* duplicate QR-generation requests

### Accessibility

The future frontend should verify:

* keyboard navigation
* accessible labels
* accessible form errors
* accessible loading/status messages
* sufficient semantic structure
* Lighthouse accessibility target of 100%

---

## AI-Assisted Engineering

This project is being developed under the client's approved AI-assisted engineering workflow.

The prompt sequence used for the architecture work is documented in:

`PROMPTS.md`

The prompt log records the architectural guidance used to establish:

1. project scope
2. database model
3. ERD relationships
4. API contract
5. edge-case handling
6. security considerations
7. final consistency review

---

## Implementation Roadmap

The architecture can later progress through the following stages:

```text
Capstone 1
    │
    ├── Database Schema
    ├── ERD
    └── API Contract
           │
           ▼
Future Implementation
    │
    ├── Database Migration
    ├── Backend API
    ├── QR Generation
    ├── Frontend Worker UI
    ├── Automated Tests
    ├── Accessibility Verification
    └── Deployment
```

These implementation stages are intentionally outside the current deliverable.

---

## Definition of Done — Current Phase

* [x] Architecture repository created
* [x] Database schema designed
* [x] ERD documented
* [x] API contract documented
* [x] Happy-path requirements mapped
* [x] Unhappy-path requirements mapped
* [x] Security considerations documented
* [x] Accessibility considerations documented
* [x] `PROMPTS.md` included
* [x] No real API keys or sensitive PII included
* [x] No unnecessary feature code introduced

---

## Final Status

**Status: Capstone 1 Architecture Complete**

This repository currently represents the **design and architecture phase** of ENG-139055.

No frontend or backend feature code is intentionally included because the current Technical Requirements Document explicitly limits this phase to database schema/ERD and API-contract planning.
