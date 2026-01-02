---
title: Specification Gallery
layout: default
nav_order: 11
permalink: /spec-gallery/
---

# Specification Gallery

Real-world spec examples across different domains. Use these as templates and inspiration.

{: .note }
> These specs demonstrate the **structure and thinking** behind good specifications. Adapt them to your domain, don't copy them verbatim.

---

## Payment Processing (Course Example)

This is the spec you build in Part 1, Labs 1.1-1.3.

### Overview
Payment processing for an e-commerce checkout demo. Must handle credit card tokens, prevent duplicate charges, and provide audit trail.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST accept payment tokens from mock gateway | P1 |
| FR-002 | System MUST return identical response for duplicate idempotency keys | P1 |
| FR-003 | System MUST validate payment amounts ($0.01 - $10,000 USD) | P1 |
| FR-004 | System MUST log all payment attempts with trace ID | P1 |
| FR-005 | System SHOULD return structured error messages (not stack traces) | P1 |

### User Scenarios

**Scenario 1: Successful Payment (P1)**
- **Given** a customer with valid payment token `tok_valid_visa`
- **When** they submit payment for $50.00 with idempotency key `order-123`
- **Then** payment succeeds with confirmation containing transaction ID
- **And** audit log entry is created

**Scenario 2: Double-Click Protection (P1)**
- **Given** a payment was just processed with idempotency key `order-123`
- **When** the same request is submitted again
- **Then** the original response is returned (no duplicate charge)
- **And** no additional audit entry is created

**Scenario 3: Invalid Token (P1)**
- **Given** a customer with invalid token `tok_invalid`
- **When** they submit payment
- **Then** they receive error `{"error": "invalid_token", "message": "Payment method is invalid"}`
- **And** HTTP status is 400

**Scenario 4: Amount Validation (P2)**
- **Given** a payment request for $15,000
- **When** submitted
- **Then** error returned: `{"error": "amount_exceeded", "message": "Maximum payment is $10,000"}`

### Data Model

```
PaymentRequest {
  amount: decimal (required, 0.01-10000)
  token: string (required, from payment gateway)
  idempotency_key: string (required, unique per transaction)
  metadata: object (optional)
}

PaymentResponse {
  transaction_id: string (unique identifier)
  status: enum [succeeded, failed]
  amount: decimal
  created_at: timestamp
  trace_id: string (for debugging)
}
```

### Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-001 | Response time | < 200ms (p95) |
| NFR-002 | Availability | 99.9% |
| NFR-003 | Idempotency window | 24 hours |

---

## User Authentication (Web App)

Common pattern for session-based authentication.

### Overview
User authentication system with email/password login, session management, and password reset flow.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST hash passwords using bcrypt with cost factor ≥ 12 | P1 |
| FR-002 | System MUST NOT reveal whether email exists during login failure | P1 |
| FR-003 | System MUST lock account after 5 failed login attempts | P1 |
| FR-004 | System MUST expire sessions after 24 hours of inactivity | P1 |
| FR-005 | System MUST invalidate all sessions on password change | P1 |
| FR-006 | Password reset tokens MUST expire after 1 hour | P1 |

### User Scenarios

**Scenario 1: Successful Login (P1)**
- **Given** a registered user with email `user@example.com`
- **When** they submit correct password
- **Then** session cookie is set (HttpOnly, Secure, SameSite=Strict)
- **And** user is redirected to dashboard

**Scenario 2: Failed Login - Wrong Password (P1)**
- **Given** a registered user with email `user@example.com`
- **When** they submit incorrect password
- **Then** error displayed: "Invalid email or password"
- **And** failed attempt counter increments
- **And** no indication whether email exists

**Scenario 3: Account Lockout (P1)**
- **Given** 5 consecutive failed login attempts
- **When** user attempts 6th login
- **Then** account is locked for 15 minutes
- **And** email notification sent to user

**Scenario 4: Password Reset Flow (P1)**
- **Given** user requests password reset for `user@example.com`
- **When** email exists in system
- **Then** reset email sent with single-use token (expires 1 hour)
- **When** email does NOT exist
- **Then** same success message shown (no enumeration)

### Security Constraints

| Constraint | Rationale |
|------------|-----------|
| bcrypt cost ≥ 12 | Slows brute force attacks |
| Generic error messages | Prevents user enumeration |
| HttpOnly cookies | Prevents XSS token theft |
| SameSite=Strict | Prevents CSRF attacks |
| Session invalidation on password change | Limits breach impact |

---

## Order Management (E-commerce)

This is the spec you build in Part 1, Labs 1.4-1.5.

### Overview
Order lifecycle management from creation through fulfillment, integrated with payment processing.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST create orders with unique order ID | P1 |
| FR-002 | System MUST enforce valid state transitions only | P1 |
| FR-003 | System MUST link orders to payment transactions | P1 |
| FR-004 | System MUST maintain order history for customer | P1 |
| FR-005 | System SHOULD support order cancellation before payment | P2 |

### State Machine

```
                    ┌─────────────────────────────────────────┐
                    │            Order States                  │
                    │                                          │
   [Create] ──────▶ │  PENDING ──────────▶ PAID               │
                    │     │                  │                 │
                    │     │                  ▼                 │
                    │     │              SHIPPED               │
                    │     │                  │                 │
                    │     ▼                  ▼                 │
                    │  CANCELLED         DELIVERED             │
                    │                        │                 │
                    │                        ▼                 │
                    │                    REFUNDED              │
                    └─────────────────────────────────────────┘
```

### Valid Transitions

| From | To | Trigger |
|------|----|---------|
| pending | paid | Payment confirmed |
| pending | cancelled | Customer cancels / timeout |
| paid | shipped | Fulfillment started |
| paid | refunded | Refund requested |
| shipped | delivered | Delivery confirmed |
| delivered | refunded | Return processed |

### User Scenarios

**Scenario 1: Order Creation (P1)**
- **Given** a customer with items in cart
- **When** they submit order
- **Then** order created with status `pending`
- **And** order ID returned in format `ORD-{timestamp}-{random}`

**Scenario 2: Payment Integration (P1)**
- **Given** an order in `pending` status
- **When** payment succeeds (transaction ID received)
- **Then** order transitions to `paid`
- **And** payment transaction linked to order

**Scenario 3: Invalid State Transition (P1)**
- **Given** an order in `delivered` status
- **When** attempting to transition to `pending`
- **Then** error returned: "Invalid transition from delivered to pending"
- **And** order state unchanged

---

## File Upload Service (Cloud Storage)

Common pattern for secure file handling.

### Overview
Secure file upload with virus scanning, size limits, and CDN distribution.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST scan uploads for malware before storing | P1 |
| FR-002 | System MUST reject files > 100MB | P1 |
| FR-003 | System MUST validate file type matches extension | P1 |
| FR-004 | System MUST generate unique, non-guessable file URLs | P1 |
| FR-005 | System MUST NOT execute uploaded files | P1 |
| FR-006 | System SHOULD generate thumbnails for images | P2 |

### Allowed File Types

| Type | Extensions | Max Size | MIME Types |
|------|------------|----------|------------|
| Images | .jpg, .png, .gif, .webp | 10MB | image/* |
| Documents | .pdf, .doc, .docx | 50MB | application/pdf, application/msword |
| Archives | .zip | 100MB | application/zip |

### User Scenarios

**Scenario 1: Successful Upload (P1)**
- **Given** a valid 5MB JPEG image
- **When** uploaded via multipart form
- **Then** file is scanned (< 30 seconds)
- **And** stored with UUID filename
- **And** CDN URL returned

**Scenario 2: Malware Detected (P1)**
- **Given** a file containing known malware signature
- **When** upload scan completes
- **Then** file is quarantined (not stored)
- **And** error returned: `{"error": "malware_detected"}`
- **And** security team notified

**Scenario 3: Type Mismatch (P1)**
- **Given** an executable renamed to `.jpg`
- **When** magic bytes checked
- **Then** rejected: "File content does not match extension"

**Scenario 4: Size Exceeded (P1)**
- **Given** a 150MB file
- **When** upload attempted
- **Then** rejected immediately (before full upload)
- **And** error: "File exceeds maximum size of 100MB"

### Security Constraints

| Constraint | Implementation |
|------------|----------------|
| No execution | Store outside webroot, Content-Disposition: attachment |
| Non-guessable URLs | UUID v4 in path, signed URLs for private files |
| Malware scanning | ClamAV or cloud service (e.g., VirusTotal) |
| Type validation | Check magic bytes, not just extension |

---

## Notification Service (Event-Driven)

This is the spec you build in Part 2, Lab 2.4.

### Overview
Event-driven notification service that sends emails, SMS, and push notifications based on system events.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST subscribe to events from event queue | P1 |
| FR-002 | System MUST support multiple channels (email, SMS, push) | P1 |
| FR-003 | System MUST retry failed notifications (3 attempts, exponential backoff) | P1 |
| FR-004 | System MUST respect user notification preferences | P1 |
| FR-005 | System MUST NOT send duplicate notifications for same event | P1 |

### Event Schema

```json
{
  "event_id": "evt_abc123",
  "event_type": "order.shipped",
  "timestamp": "2024-01-15T10:30:00Z",
  "correlation_id": "req_xyz789",
  "data": {
    "order_id": "ORD-123",
    "customer_email": "user@example.com",
    "customer_phone": "+1234567890",
    "tracking_number": "1Z999AA10123456784"
  }
}
```

### Supported Events

| Event Type | Default Channel | Template |
|------------|-----------------|----------|
| order.created | email | "Your order #{order_id} has been received" |
| order.paid | email | "Payment confirmed for order #{order_id}" |
| order.shipped | email + SMS | "Order shipped! Track: {tracking_number}" |
| payment.failed | email | "Payment failed. Please update payment method" |
| account.locked | email + SMS | "Security alert: Account locked" |

### User Scenarios

**Scenario 1: Successful Notification (P1)**
- **Given** event `order.shipped` published
- **When** notification service receives event
- **Then** email sent to customer
- **And** SMS sent if phone number present
- **And** event marked as processed

**Scenario 2: Retry on Failure (P1)**
- **Given** email provider returns 503
- **When** first delivery attempt fails
- **Then** retry after 1 minute
- **When** second attempt fails
- **Then** retry after 5 minutes
- **When** third attempt fails
- **Then** move to dead letter queue
- **And** alert operations team

**Scenario 3: User Preference Respected (P1)**
- **Given** user has disabled SMS notifications
- **When** `order.shipped` event received
- **Then** only email sent (no SMS)

---

## API Rate Limiting (Platform)

Common pattern for protecting APIs from abuse.

### Overview
Rate limiting middleware that protects API endpoints from abuse while allowing legitimate burst traffic.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST limit requests per API key | P1 |
| FR-002 | System MUST return 429 with Retry-After header when limit exceeded | P1 |
| FR-003 | System MUST use sliding window algorithm | P1 |
| FR-004 | System MUST support different limits per tier | P1 |
| FR-005 | System SHOULD allow burst traffic within reason | P2 |

### Rate Limit Tiers

| Tier | Requests/min | Burst | Use Case |
|------|--------------|-------|----------|
| Free | 60 | 10 | Development, testing |
| Basic | 600 | 50 | Small applications |
| Pro | 6000 | 200 | Production workloads |
| Enterprise | Custom | Custom | High-volume partners |

### Response Headers

```
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 599
X-RateLimit-Reset: 1705312800
```

### User Scenarios

**Scenario 1: Normal Request (P1)**
- **Given** API key with 600/min limit, 100 requests used
- **When** request received
- **Then** request processed
- **And** headers show Remaining: 499

**Scenario 2: Rate Limited (P1)**
- **Given** API key has exhausted limit
- **When** request received
- **Then** HTTP 429 returned
- **And** body: `{"error": "rate_limit_exceeded", "retry_after": 45}`
- **And** header: `Retry-After: 45`

**Scenario 3: Burst Allowance (P2)**
- **Given** API key with 60/min limit, 0 requests in last minute
- **When** 70 requests arrive in 1 second
- **Then** first 70 processed (burst allowance)
- **And** subsequent requests rate limited

---

## Brownfield: Legacy System Integration

This is the spec pattern you use in Part 2 when documenting existing systems.

### Overview
**This spec documents OBSERVED behavior, not desired behavior.** It's extracted from characterization testing and code analysis of the legacy OrderFlow system.

### Observed Business Rules

| Rule ID | Description | Source |
|---------|-------------|--------|
| BR-001 | Tax rate: 8.5% on subtotal | `app.py:234` |
| BR-002 | Free shipping threshold: $50 | `app.py:245` |
| BR-003 | Max items per order: 100 | `app.py:189` |
| BR-004 | Session timeout: 30 minutes | `app.py:67` |
| BR-005 | Admin bypass: `?debug=true` skips auth | `app.py:412` ⚠️ SECURITY |

### Observed State Transitions

| From | To | Condition | Surprising? |
|------|----|-----------|-------------|
| pending | paid | Any payment amount | No |
| paid | shipped | Manual trigger only | No |
| shipped | delivered | No validation | ⚠️ Yes |
| ANY | pending | Admin override | ⚠️ Yes |

### Flagged Issues

| Severity | Issue | Location | Risk |
|----------|-------|----------|------|
| CRITICAL | Debug auth bypass | `app.py:412` | Production exploit |
| HIGH | No input validation on amount | `app.py:198` | Negative payments possible |
| MEDIUM | Hardcoded credentials | `app.py:89` | Secret exposure |
| LOW | Inconsistent date formats | Various | Data quality |

### Integration Constraints

When integrating with this system:

1. **Must preserve**: Tax calculation (BR-001), shipping threshold (BR-002)
2. **Must fix before production**: Debug bypass (BR-005)
3. **Consider preserving**: Admin state override (may be intentional)
4. **Needs PM decision**: Max items limit - is 100 correct?

---

## Template: Start Here

Copy this template for new specs:

```markdown
# Feature: [Name]

## Overview
[One paragraph: What is this feature? Why does it exist?]

## Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | System MUST [specific, testable action] | P1 |
| FR-002 | System MUST [another requirement] | P1 |
| FR-003 | System SHOULD [nice-to-have] | P2 |

## User Scenarios

### Scenario 1: [Happy Path] (P1)
- **Given** [initial context]
- **When** [action taken]
- **Then** [expected outcome]

### Scenario 2: [Edge Case] (P1)
- **Given** [context with edge condition]
- **When** [action taken]
- **Then** [graceful handling]

### Scenario 3: [Error Case] (P1)
- **Given** [context that causes failure]
- **When** [action taken]
- **Then** [clear error message]

## Data Model

[Entities, fields, relationships]

## Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-001 | Response time | < Xms (p95) |
| NFR-002 | Availability | X% |

## Constraints & Assumptions
- [Technology constraints]
- [Business constraints]
- [Assumptions made]

## Open Questions
- [ ] [Question needing PM/stakeholder input]
```

---

## Error Scenario Patterns

Specifying error handling is often overlooked but crucial for production-ready code. Here are patterns for common error categories.

### Input Validation Errors

**Pattern**: Validate early, fail fast, return helpful messages.

```markdown
### Scenario: Invalid Email Format
- **Given** a registration request with email "not-an-email"
- **When** validation runs
- **Then** HTTP 400 returned
- **And** body: `{"error": "validation_error", "field": "email", "message": "Invalid email format"}`
- **And** request is NOT processed

### Scenario: Missing Required Field
- **Given** a payment request without `amount` field
- **When** validation runs
- **Then** HTTP 400 returned
- **And** body: `{"error": "missing_field", "field": "amount", "message": "Amount is required"}`

### Scenario: Value Out of Range
- **Given** a payment request with amount = -50.00
- **When** validation runs
- **Then** HTTP 400 returned
- **And** body: `{"error": "invalid_value", "field": "amount", "message": "Amount must be between $0.01 and $10,000"}`
```

**Error Response Schema**:
```json
{
  "error": "string (error_code)",
  "field": "string (optional, which field failed)",
  "message": "string (human-readable)",
  "details": "object (optional, additional context)"
}
```

### Authentication Errors

**Pattern**: Never reveal existence of users, always use generic messages.

```markdown
### Scenario: Invalid Credentials
- **Given** a login request with wrong password
- **When** authentication fails
- **Then** HTTP 401 returned
- **And** body: `{"error": "authentication_failed", "message": "Invalid email or password"}`
- **And** message is IDENTICAL for wrong email vs wrong password (no user enumeration)

### Scenario: Expired Token
- **Given** a request with expired JWT
- **When** token validation runs
- **Then** HTTP 401 returned
- **And** body: `{"error": "token_expired", "message": "Your session has expired. Please log in again"}`

### Scenario: Missing Authentication
- **Given** a request to protected endpoint without Authorization header
- **When** auth middleware runs
- **Then** HTTP 401 returned
- **And** body: `{"error": "authentication_required", "message": "Authentication required"}`
```

### Authorization Errors

**Pattern**: Acknowledge the request but deny access without revealing resource existence.

```markdown
### Scenario: Insufficient Permissions
- **Given** a user with role "viewer" accessing admin endpoint
- **When** authorization check runs
- **Then** HTTP 403 returned
- **And** body: `{"error": "forbidden", "message": "You don't have permission to perform this action"}`

### Scenario: Resource Not Owned
- **Given** user A trying to access user B's order
- **When** ownership check runs
- **Then** HTTP 404 returned (NOT 403, to avoid revealing existence)
- **And** body: `{"error": "not_found", "message": "Order not found"}`
```

### Resource Errors

**Pattern**: Clear 404s, helpful 409s for conflicts.

```markdown
### Scenario: Resource Not Found
- **Given** a request for order ID "ORD-nonexistent"
- **When** lookup runs
- **Then** HTTP 404 returned
- **And** body: `{"error": "not_found", "resource": "order", "message": "Order not found"}`

### Scenario: Resource Already Exists (Conflict)
- **Given** a create request for username that exists
- **When** uniqueness check runs
- **Then** HTTP 409 returned
- **And** body: `{"error": "conflict", "field": "username", "message": "Username already taken"}`

### Scenario: Stale Resource (Optimistic Locking)
- **Given** user A and B both loaded order version 1
- **When** user A saves changes (now version 2)
- **And** user B tries to save changes to version 1
- **Then** HTTP 409 returned
- **And** body: `{"error": "conflict", "message": "Resource was modified. Please reload and try again"}`
```

### Rate Limiting Errors

**Pattern**: Always include Retry-After header.

```markdown
### Scenario: Rate Limit Exceeded
- **Given** API key has made 100 requests in last minute (limit: 60)
- **When** next request arrives
- **Then** HTTP 429 returned
- **And** header: `Retry-After: 45`
- **And** body: `{"error": "rate_limit_exceeded", "retry_after": 45, "message": "Too many requests. Try again in 45 seconds"}`
```

### External Service Errors

**Pattern**: Hide internal details, provide actionable messages, include correlation ID.

```markdown
### Scenario: Payment Gateway Timeout
- **Given** payment gateway doesn't respond within 10 seconds
- **When** timeout occurs
- **Then** HTTP 503 returned
- **And** body: `{"error": "service_unavailable", "message": "Payment service temporarily unavailable. Please try again.", "correlation_id": "req-abc123"}`
- **And** internal error is logged with full details

### Scenario: Payment Gateway Error
- **Given** payment gateway returns unexpected error
- **When** error is caught
- **Then** HTTP 502 returned
- **And** body: `{"error": "upstream_error", "message": "Unable to process payment. Please try again or contact support.", "correlation_id": "req-abc123"}`
- **And** actual error is NOT exposed to client

### Scenario: Database Connection Lost
- **Given** database connection fails mid-request
- **When** query fails
- **Then** HTTP 503 returned
- **And** body: `{"error": "service_unavailable", "message": "Service temporarily unavailable. Please try again.", "correlation_id": "req-abc123"}`
- **And** alert sent to ops team
```

### Idempotency Errors

**Pattern**: Handle duplicates gracefully.

```markdown
### Scenario: Duplicate Request (Idempotent)
- **Given** a payment was processed with idempotency_key "order-123"
- **When** same request with same key arrives
- **Then** HTTP 200 returned (NOT 201 or 409)
- **And** body: original response (exact same)
- **And** no duplicate processing occurs

### Scenario: Conflicting Idempotency Key
- **Given** idempotency_key "order-123" was used for $50 payment
- **When** request with same key but amount $100 arrives
- **Then** HTTP 422 returned
- **And** body: `{"error": "idempotency_conflict", "message": "Idempotency key was used with different parameters"}`
```

### State Machine Errors

**Pattern**: Explain what's wrong and what's allowed.

```markdown
### Scenario: Invalid State Transition
- **Given** an order in "delivered" status
- **When** attempting to transition to "pending"
- **Then** HTTP 422 returned
- **And** body: 
  ```json
  {
    "error": "invalid_transition",
    "current_state": "delivered",
    "requested_state": "pending",
    "allowed_transitions": ["refunded"],
    "message": "Cannot transition from 'delivered' to 'pending'. Allowed: refunded"
  }
  ```
```

### Error Response Best Practices

| Do | Don't |
|----|-------|
| Use consistent error schema | Different formats per endpoint |
| Include correlation/trace ID | Expose internal error messages |
| Provide actionable messages | "An error occurred" |
| Use appropriate HTTP codes | 200 for everything |
| Log full details internally | Expose stack traces to clients |
| Include `Retry-After` for 429/503 | Force clients to guess |

### HTTP Status Code Quick Reference

| Code | When to Use |
|------|-------------|
| 400 | Invalid input, validation failed |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource not found (also for unauthorized access to hide existence) |
| 409 | Conflict (duplicate, version mismatch) |
| 422 | Semantically invalid (e.g., state machine violation) |
| 429 | Rate limit exceeded |
| 500 | Unexpected server error (bug) |
| 502 | Upstream service returned error |
| 503 | Service temporarily unavailable |
| 504 | Upstream service timeout |

---

**Remember**: Specs are living documents. Update them as you learn more.
