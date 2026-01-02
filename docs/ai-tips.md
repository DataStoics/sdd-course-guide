---
title: AI Assistant Tips
layout: default
nav_order: 12
permalink: /ai-tips/
---

# AI Assistant Power User Guide

Get more out of your AI coding assistant with these techniques.

{: .note }
> This guide focuses on **GitHub Copilot** but principles apply to Claude Code and Gemini CLI too.

---

## The Spec-First Mindset

### Why Specs Make AI Better

Without specs, AI guesses. With specs, AI implements.

| Without Spec | With Spec |
|--------------|-----------|
| "Build a payment system" | "Implement FR-001 per `specs/001-payment/spec.md`" |
| AI invents requirements | AI follows YOUR requirements |
| Different output each time | Consistent, traceable output |
| You debug AI's assumptions | You validate against spec |

### The Reference Pattern

Always reference your spec explicitly:

```text
# Good - explicit reference
Implement the idempotency check per FR-002 in specs/001-payment/spec.md.
The scenario is "Double-Click Protection" which expects returning 
the original response when the same idempotency_key is submitted twice.

# Bad - vague
Add idempotency to the payment endpoint.
```

---

## Copilot-Specific Techniques

### Agent Mode vs. Chat Mode

| Mode | Use For | How to Access |
|------|---------|---------------|
| **Agent Mode** | Multi-step tasks, file creation, running commands | Select "Agent" at top of chat |
| **Chat Mode** | Quick questions, explanations, code review | Default mode |
| **Inline** | Single-line completions | Just type in editor |

**For SDD**: Use Agent Mode for `/speckit.*` commands. It can create files, run tests, and iterate.

### @ References

Reference files and symbols directly:

```text
# Reference a file
@specs/001-payment/spec.md implement FR-003

# Reference a symbol
@PaymentService add the validation from FR-003

# Reference the workspace
@workspace where is idempotency handled?
```

### Slash Commands

| Command | Purpose |
|---------|---------|
| `/explain` | Explain selected code |
| `/fix` | Fix errors in selected code |
| `/tests` | Generate tests for selected code |
| `/doc` | Generate documentation |
| `/new` | Create new file from description |

**SDD Pro Tip**: Combine with spec references:
```text
/tests Generate tests for @src/payment.py based on scenarios in @specs/001-payment/spec.md
```

### Multi-File Context

When working across files, open them all:

1. Open `spec.md` in a tab
2. Open `plan.md` in another tab
3. Open the file you're implementing
4. Copilot sees all open files as context

Or use explicit references:
```text
Based on @specs/001-payment/spec.md and @specs/001-payment/plan.md,
implement the payment endpoint in @src/app/payment.py
```

---

## Effective Prompting Patterns

### The STAR Pattern

**S**ituation → **T**ask → **A**ction → **R**esult

```text
# Situation
The payment endpoint currently accepts any amount.

# Task
Add validation per FR-003 (amounts must be $0.01 - $10,000).

# Action
Update the PaymentRequest model and add validation in the endpoint.

# Result
Invalid amounts should return HTTP 400 with error code "amount_invalid".
```

### The Scenario Pattern

Reference specific Given/When/Then scenarios:

```text
Implement Scenario 3 from specs/001-payment/spec.md:

Given: Customer submits invalid token "tok_invalid"
When: Payment is attempted  
Then: Return {"error": "invalid_token", "message": "Payment method is invalid"}

Make sure the error response matches this exact format.
```

### The Traceability Pattern

Request code comments linking to requirements:

```text
Implement FR-004 (audit logging) with a docstring that references 
the requirement ID and spec file location.
```

Result:
```python
def log_payment_attempt(request: PaymentRequest, result: PaymentResult):
    """Log payment attempt for audit trail.
    
    Implements FR-004: System MUST log all payment attempts with trace ID.
    Source: specs/001-payment/spec.md#FR-004
    """
```

### The Constraint Pattern

Be explicit about what NOT to do:

```text
Implement the payment validation with these constraints:
- Do NOT use external validation libraries
- Do NOT change the existing response format
- Do NOT modify the PaymentRequest model structure
- Only add validation logic in the endpoint function
```

---

## Common Mistakes & Fixes

### Mistake 1: Vague Prompts

```text
# Bad
Make it more secure

# Good
Add input validation to prevent SQL injection per OWASP guidelines.
Specifically validate the `order_id` parameter accepts only alphanumeric 
characters matching pattern: ^ORD-[0-9]{10}$
```

### Mistake 2: No Context

```text
# Bad
Fix the bug

# Good
The test test_double_click_protection is failing because the endpoint 
returns a new transaction ID instead of the cached one. Fix this by 
checking Redis for existing idempotency_key before processing.
```

### Mistake 3: Too Much at Once

```text
# Bad
Build the entire order service with all endpoints, tests, models, 
state machine, payment integration, and audit logging.

# Good (step by step)
Step 1: Create the Order model with fields from data-model.md
Step 2: Implement the state machine transitions from spec.md
Step 3: Add the POST /orders endpoint per FR-001
[Continue one step at a time]
```

### Mistake 4: Accepting Without Validation

```text
# Bad approach
AI generates code → Ship it

# Good approach
AI generates code → Run tests → Review against spec → 
Check edge cases → Then ship
```

---

## Advanced Techniques

### Iterative Refinement

Build in stages, validating each:

```text
# Stage 1
Create the basic endpoint structure for POST /orders.
Just the route handler with request/response models.

[Review, test, approve]

# Stage 2
Now add the state machine validation from spec.md.
The initial state should be "pending".

[Review, test, approve]

# Stage 3
Now integrate with the payment service per FR-003.
Use the existing PaymentClient from src/services/.

[Review, test, approve]
```

### Test-First Prompting

Ask for tests before implementation:

```text
# Step 1: Generate test
Write a test for Scenario 2 (Double-Click Protection) from spec.md.
The test should verify that identical idempotency_key returns 
identical response without creating duplicate payment.

[Review test, ensure it matches spec]

# Step 2: Implement to pass
Now implement the idempotency check to make this test pass.
Use Redis to store the mapping from idempotency_key to response.
```

### Comparison Prompting

For brownfield work, compare implementations:

```text
Compare how order creation is handled in:
1. @legacy-orderflow/app.py (search for /create_order route)
2. The spec in @specs/order-creation.md

List the differences and flag any behaviors in legacy 
that aren't documented in the spec.
```

### Security Review Prompting

```text
Review @src/app/payment.py for security issues:
1. Input validation gaps
2. Authentication/authorization issues
3. Injection vulnerabilities
4. Sensitive data exposure

For each issue found, reference the relevant OWASP category 
and suggest a fix.
```

---

## Workspace Organization for AI

### File Structure That Helps AI

```
project/
├── specs/
│   └── 001-payment/
│       ├── spec.md          # What to build
│       ├── plan.md          # How to build it
│       ├── tasks.md         # Step-by-step breakdown
│       └── data-model.md    # Entity definitions
├── src/
│   └── app/
│       ├── payment.py       # Implementation
│       └── models.py        # Data models
└── tests/
    └── test_payment.py      # Tests from scenarios
```

**Why this helps**: AI can trace `spec.md` → `tasks.md` → `payment.py` → `test_payment.py`

### Naming Conventions

| Pattern | Example | AI Benefit |
|---------|---------|------------|
| Numbered specs | `001-payment/` | Clear ordering |
| Requirement IDs | `FR-001`, `NFR-001` | Traceable references |
| Scenario names | `test_double_click_protection` | Maps to spec scenarios |
| Explicit models | `PaymentRequest`, `PaymentResponse` | Clear domain language |

---

## Tool-Specific Tips

### GitHub Copilot

- **Workspace indexing**: Copilot indexes your workspace. Keep specs in the repo.
- **`.github/copilot-instructions.md`**: Add project-specific context that persists.
- **Copilot Chat history**: Conversations are saved. Reference previous discussions.

### Claude Code

- **CLAUDE.md**: Create this file in repo root with project context.
- **Memory**: Claude remembers conversation context within a session.
- **MCP servers**: Connect to GitHub, databases, APIs for live context.

### Gemini CLI

- **Context window**: Very large. Can process entire specs at once.
- **Grounding**: Can search Google for current documentation.
- **GEMINI.md**: Similar to CLAUDE.md for project context.

---

## Quick Reference: Prompt Templates

### Implement a Requirement
```text
Implement {FR-XXX} from specs/{feature}/spec.md.

The requirement states: {paste requirement text}

Expected behavior from Scenario {N}:
- Given: {context}
- When: {action}
- Then: {outcome}

Add tests that verify this scenario.
```

### Fix a Failing Test
```text
The test `{test_name}` is failing with:
{paste error message}

The test is based on Scenario {N} from spec.md which expects:
{paste scenario}

The current implementation in {file} does:
{brief description}

Fix the implementation to match the spec.
```

### Add Error Handling
```text
Add error handling to {function/endpoint} per Scenario {N} in spec.md.

When {error condition}, the system should:
- Return HTTP {status code}
- Response body: {exact JSON format}
- Log: {what to log}

Do NOT change the happy path behavior.
```

### Review Code Against Spec
```text
Review @{file} against @specs/{feature}/spec.md.

Check:
1. All FR-XXX requirements are implemented
2. All scenarios have corresponding tests
3. Error handling matches spec
4. Data model matches data-model.md

List any gaps or discrepancies.
```

---

**Remember**: AI is your implementation partner, not your requirements analyst. You own the spec. AI implements it.
