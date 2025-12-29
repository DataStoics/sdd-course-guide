# Governance Checklist

**Purpose**: Validate that specifications contain proper governance constraints before proceeding to implementation.

---

## How to Use This Checklist

Before running `/speckit.plan`, verify your specification passes ALL applicable checks below. Mark each item as you verify.

---

## PCI DSS Requirements (Payment Card Industry)

*Apply these checks if your spec involves payment processing, card data, or financial transactions.*

### Data Handling

- [ ] **No raw card storage**: Spec prohibits storing full card numbers, CVV, or magnetic stripe data
- [ ] **Tokenization required**: Spec specifies use of payment tokens instead of raw card data
- [ ] **Encryption specified**: Spec defines encryption requirements for any stored payment data

### Access Control

- [ ] **Authentication required**: Spec requires authentication for payment operations
- [ ] **Role separation**: Spec defines who can perform what payment actions
- [ ] **Audit logging**: Spec requires logging of all payment-related actions

### Transmission Security

- [ ] **TLS required**: Spec mandates encrypted transmission (HTTPS)
- [ ] **No sensitive data in URLs**: Spec prohibits card data in query strings or URLs

---

## GDPR Requirements (General Data Protection Regulation)

*Apply these checks if your spec involves personal data of EU residents.*

### Data Collection

- [ ] **Purpose limitation**: Spec defines WHY personal data is collected
- [ ] **Data minimization**: Spec collects only necessary data
- [ ] **Consent mechanism**: Spec defines how consent is obtained (if required)

### Data Storage

- [ ] **Retention policy defined**: Spec states how long data is kept
- [ ] **Deletion capability**: Spec defines how data can be deleted on request
- [ ] **Storage location**: Spec identifies where data is stored (EU/US/etc.)

### Data Subject Rights

- [ ] **Access mechanism**: Spec defines how users can access their data
- [ ] **Correction capability**: Spec allows users to correct their data
- [ ] **Portability**: Spec defines data export format (if applicable)

### Audit Trail

- [ ] **Processing records**: Spec requires logging of who processed personal data
- [ ] **Timestamp logging**: Spec requires timestamps on all data operations
- [ ] **Actor identification**: Spec requires identification of who performed actions

---

## Idempotency Requirements

*Apply these checks if your spec involves operations that should not be duplicated.*

### Duplicate Prevention

- [ ] **Idempotency key specified**: Spec requires unique keys for deduplicable operations
- [ ] **Key format defined**: Spec defines the format/source of idempotency keys
- [ ] **Cache duration set**: Spec defines how long to remember previous requests

### Behavior Specification

- [ ] **Duplicate response defined**: Spec states what happens on duplicate requests
- [ ] **Partial failure handling**: Spec defines behavior when operation partially completes

---

## Security Requirements

*Apply these checks to ALL specifications.*

### Authentication & Authorization

- [ ] **Auth requirements stated**: Spec defines what requires authentication
- [ ] **Authorization model defined**: Spec defines who can do what
- [ ] **Session handling**: Spec defines session timeout/refresh (if applicable)

### Input Validation

- [ ] **Input constraints defined**: Spec defines valid input ranges/formats
- [ ] **Error handling specified**: Spec defines how invalid input is handled
- [ ] **Injection prevention**: Spec prohibits raw user input in sensitive contexts

### Output Protection

- [ ] **Sensitive data masking**: Spec defines what data must be masked in responses
- [ ] **Error message security**: Spec prohibits exposing internal details in errors

---

## Audit Requirements

*Apply these checks to ALL specifications.*

### Logging

- [ ] **What to log defined**: Spec lists events that must be logged
- [ ] **Log format specified**: Spec defines log structure (timestamp, actor, action, etc.)
- [ ] **Log retention defined**: Spec states how long logs are kept

### Traceability

- [ ] **Request correlation**: Spec requires trace IDs for request tracking
- [ ] **User attribution**: Spec requires identifying which user performed actions

---

## Checklist Validation

### Before `/speckit.plan`

Count your checked items:

| Category | Checked | Required | Status |
|----------|---------|----------|--------|
| PCI (if applicable) | [ ] / 8 | All applicable | ❓ |
| GDPR (if applicable) | [ ] / 10 | All applicable | ❓ |
| Idempotency (if applicable) | [ ] / 5 | All applicable | ❓ |
| Security | [ ] / 7 | All | ❓ |
| Audit | [ ] / 4 | All | ❓ |

### Result

- [ ] **PASS** — All applicable checks verified, proceed to `/speckit.plan`
- [ ] **FAIL** — Update specification to address unchecked items

---

## Common Failures & Fixes

| Missing Item | Quick Fix |
|--------------|-----------|
| No retention policy | Add: "Data retained for X years per [regulation]" |
| No tokenization | Add: "Raw card data never stored; use payment tokens" |
| No audit logging | Add: "All operations logged with timestamp + actor" |
| No idempotency | Add: "Duplicate requests return original response" |
| Vague access control | Add: "Only [role] can perform [action]" |

---

## Remember

> **Specs are governance infrastructure, not documentation.**
>
> If an auditor asked "how do you ensure X?", your spec should have the answer.
