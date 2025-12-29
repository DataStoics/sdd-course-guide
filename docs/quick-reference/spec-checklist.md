---
title: Spec Checklist
parent: Quick Reference
nav_order: 2
---
# Specification Quality Checklist

Use this checklist to evaluate specification quality before proceeding to implementation.

---

## Essential Sections

### âœ… Business Context
- [ ] Clear problem statement
- [ ] Who benefits (users/stakeholders)
- [ ] Success metrics defined
- [ ] Scope boundaries (what's NOT included)

### âœ… Functional Requirements
- [ ] Each requirement has unique ID (FR-001, FR-002)
- [ ] Requirements are atomic (one thing each)
- [ ] Requirements are testable
- [ ] No implementation details (HOW) â€” only WHAT

### âœ… GOVERNANCE CONSTRAINTS
- [ ] Regulatory requirements identified (PCI, GDPR, HIPAA, SOC2)
- [ ] Data classification (what's sensitive)
- [ ] Retention requirements (how long to keep)
- [ ] Audit requirements (what to log)
- [ ] Access control requirements

### âœ… Acceptance Scenarios
- [ ] Given/When/Then format
- [ ] Cover happy path
- [ ] Cover error cases
- [ ] Cover edge cases
- [ ] Specific values (not "valid" or "correct")

### âœ… Non-Functional Requirements
- [ ] Performance targets (response time, throughput)
- [ ] Availability expectations
- [ ] Security requirements (authentication, encryption)
- [ ] Scalability considerations

### âœ… Data Requirements
- [ ] Entities defined
- [ ] Required fields identified
- [ ] Relationships documented
- [ ] Validation rules specified

---

## Quality Indicators

### ðŸŸ¢ Good Spec Signs
- Can write tests directly from acceptance scenarios
- No questions needed to start implementing
- Security/compliance built into requirements
- Clear success criteria

### ðŸ”´ Poor Spec Signs
- Uses vague words: "should", "appropriate", "reasonable"
- Missing acceptance scenarios
- No governance constraints
- Implementation details mixed with requirements

---

## Vague vs. Specific

| Vague âŒ | Specific âœ… |
|----------|-------------|
| "Handle errors appropriately" | "Return HTTP 400 with error code and message for invalid input" |
| "Secure the endpoint" | "Require Bearer token; reject with 401 if missing or invalid" |
| "Store user data" | "Store email, created_at; hash password with bcrypt cost 12" |
| "Respond quickly" | "P95 response time < 200ms under 100 concurrent users" |
| "Log important events" | "Log: user_id, action, timestamp, ip_address for audit" |

---

## Pre-Plan Validation

Before running `/speckit.plan`, verify:

1. **Testability**: Can I write a test for each requirement?
2. **Completeness**: Are all scenarios covered?
3. **Clarity**: Would a new team member understand this?
4. **Compliance**: Would this pass an audit?
5. **Feasibility**: Is this technically achievable?

---

## Quick Spec Template

```markdown
# Feature: [Name]

## Business Context
[Problem statement and value]

## Functional Requirements
- FR-001: [Requirement]
- FR-002: [Requirement]

## GOVERNANCE CONSTRAINTS
- GC-001: [Constraint with regulation reference]
- GC-002: [Constraint]

## Acceptance Scenarios

### Scenario 1: [Name]
**Given** [initial state]
**When** [action]
**Then** [expected outcome with specific values]

## Non-Functional Requirements
- NFR-001: [Performance/security/availability requirement]

## Data Requirements
- Entity: [Fields, types, constraints]
```

---

## Common Missing Items

When reviewing specs, check for these frequently missed items:

1. **Idempotency** â€” How do we handle duplicate requests?
2. **Error messages** â€” What do users see when things fail?
3. **Rate limiting** â€” How do we prevent abuse?
4. **Timeouts** â€” What happens when dependencies are slow?
5. **Partial failure** â€” What if step 2 of 3 fails?
6. **Audit trail** â€” Who did what when?
7. **Data cleanup** â€” When/how is data deleted?
