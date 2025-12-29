---
title: Spec Template
parent: Reference
nav_order: 1
---
# Specification Template

**Feature Branch**: `XXX-feature-name`  
**Created**: YYYY-MM-DD  
**Status**: Draft  
**Input**: [Brief description of what this feature/spec covers]

---

## Overview

[One paragraph explaining what this specification covers and why it matters.]

---

## GOVERNANCE CONSTRAINTS

> âš ï¸ **REQUIRED SECTION** â€” Every specification MUST define explicit governance constraints.
> These constraints ensure AI-generated code meets compliance, security, and audit requirements.

### Compliance Requirements

| Constraint | Type | Requirement | Verification |
|------------|------|-------------|--------------|
| [Name] | PCI / GDPR / SOX / etc. | [Specific requirement] | [How to verify] |

### Security Requirements

- [ ] **Data Classification**: [Public / Internal / Confidential / Restricted]
- [ ] **Encryption**: [At rest / In transit / Both / None required]
- [ ] **Authentication**: [Required / Not required]
- [ ] **Authorization**: [Role-based / Attribute-based / None]

### Audit Requirements

- [ ] **Logging**: What must be logged (actions, actors, timestamps)
- [ ] **Retention**: How long logs/data must be kept
- [ ] **Access Trail**: Who accessed what, when

### Idempotency Requirements

- [ ] **Duplicate Prevention**: How to prevent duplicate operations
- [ ] **Cache Duration**: How long to cache idempotency keys

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - [Story Name] (Priority: P1)

[One paragraph describing what the user wants to accomplish and why.]

**Why this priority**: [Explain why this is P1/P2/P3]

**Independent Test**: [How to verify this story works independently]

**Acceptance Scenarios**:

1. **Given** [precondition], **When** [action], **Then** [expected result]
2. **Given** [precondition], **When** [action], **Then** [expected result]
3. **Given** [precondition], **When** [action], **Then** [expected result]

---

### Edge Cases

- What happens when [edge case 1]? â†’ [Expected behavior]
- What happens when [edge case 2]? â†’ [Expected behavior]
- What happens when [edge case 3]? â†’ [Expected behavior]

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: [Requirement description]
- **FR-002**: [Requirement description]
- **FR-003**: [Requirement description]

### Key Entities

- **[Entity 1]**: [Description]
- **[Entity 2]**: [Description]

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: [Measurable success criterion]
- **SC-002**: [Measurable success criterion]
- **SC-003**: [Measurable success criterion]

---

## Assumptions

- [Assumption 1]
- [Assumption 2]

---

## Dependencies

- [Dependency 1]
- [Dependency 2]

---

## Out of Scope

- [What this spec explicitly does NOT cover]
- [Deferred to future work]

---

## Clarifications

### Session YYYY-MM-DD

- Q: [Question asked] â†’ A: [Answer provided]

---

## Template Usage Notes

1. **GOVERNANCE CONSTRAINTS is mandatory** â€” Never skip this section
2. **Be specific** â€” Vague specs produce vague code
3. **Think compliance first** â€” What would an auditor need to see?
4. **Testable criteria** â€” Every requirement must be verifiable
5. **No implementation details** â€” Specs describe WHAT, not HOW
