---
title: "Lab 1.5: Integration"
layout: default
parent: Labs
nav_order: 6
---
# Lab 1.5: Integrate Payment + Order  Thursday Build Day

**Duration**: 120 minutes
**Day**: 2 (Afternoon)
**Prerequisites**: Completed Lab 1.4 with order specification

---

## Learning Objective

Build the order service, wire it to payments, and prove the full checkout flow works. By end of day Thursday, you'll have a complete demo: create order  pay  see order history.

**Key principle**: You write the spec, AI writes the code. Your job is to validate.

---

## Course Progress

![Lab 1.5 Progress](../assets/images/lab-1.5-progress.svg)

**Thursday goal**: Full checkout flow working, tests passing, demo-ready.

---

## Starting Point

- Working payment service from Lab 1.3
- Complete order specification from Lab 1.4
- Integration points documented

---

## Step 1: Generate Order Tasks (15 min)

Break down your order spec into actionable tasks:

> "/speckit.tasks"

Your AI reads `specs/002-order/spec.md` and `plan.md`, then generates `specs/002-order/tasks.md` with:

| Phase | Tasks | Description |
|-------|-------|-------------|
| **Setup** | T001-T004 | Dependencies, project structure |
| **Foundational** | T005-T006 | Database setup, error handling |
| **Order CRUD** | T007-T012 | Create, read, update order endpoints |
| **State Machine** | T013-T015 | Status transitions, validation |
| **Integration** | T016-T018 | Payment callback, audit trail |
| **Tests** | T019-T022 | Unit tests, integration tests |

### Verify Tasks File

Open `specs/002-order/tasks.md` and confirm:

- [ ] Tasks are dependency-ordered (setup before features)
- [ ] Each task has a unique ID (T001, T002, etc.)
- [ ] Parallel tasks marked with `[P]`
- [ ] Story references link back to spec

---

## Step 2: Validate Before Implementation (5 min)

Run analysis to catch issues before writing code:

> "/speckit.analyze"

**What it checks**:
- Spec requirements have corresponding plan elements
- Data model covers all entities
- No orphaned plan items without spec justification
- Constitution compliance

**Fix any issues before proceeding.**

---

## Step 3: Implement Order Models (20 min)

Now let AI generate the implementation:

> "/speckit.implement Start with the order models. Create OrderStatus enum with states from the spec (created, paid, fulfilled, cancelled, refunded, archived). Create Pydantic models for CreateOrderRequest, OrderResponse, MarkOrderPaidRequest, and OrderAuditEntry."

### What AI Generates

The AI creates `src/app/models.py` additions:

| Model | Purpose | Key Fields |
|-------|---------|------------|
| `OrderStatus` | State machine enum | created, paid, fulfilled, cancelled, refunded, archived |
| `CreateOrderRequest` | Input validation | items (list), idempotency_key |
| `OrderResponse` | API response | id, status, total_amount, items, payment_transaction_id |
| `MarkOrderPaidRequest` | Payment callback | transaction_id |
| `OrderAuditEntry` | Governance compliance | order_id, actor_id, timestamp, event_type |

### Your Verification (Critical)

Open the generated file and verify:

- [ ] `OrderStatus` matches spec state machine exactly
- [ ] All fields have proper validation (Field constraints)
- [ ] Audit entry includes timestamp and actor (governance)
- [ ] No hardcoded values that should be configurable

**Never skip verification.** AI generates code; you own correctness.

---

## Step 4: Implement State Machine (15 min)

> "/speckit.implement Create the state machine in src/app/state_machine.py. Define VALID_TRANSITIONS dict matching the spec. Include InvalidTransitionError exception and can_transition/validate_transition functions."

### Verify State Transitions

The spec defines these valid transitions:

| From State | Valid Transitions |
|------------|-------------------|
| created | paid, cancelled |
| paid | fulfilled, refunded |
| fulfilled | archived |
| cancelled | (terminal) |
| refunded | (terminal) |
| archived | (terminal) |

**Open the generated file and confirm the transitions match exactly.**

### Test the State Machine

> "Write a quick test: can we transition from 'created' to 'paid'? From 'cancelled' to 'paid'? Run it."

Expected:
-  created  paid (valid)
-  cancelled  paid (InvalidTransitionError)

---

## Step 5: Implement Order Endpoints (30 min)

> "/speckit.implement Create order endpoints in src/app/order.py:
> 1. POST /orders - create order with idempotency check (use Redis like payment service)
> 2. PATCH /orders/{id}/pay - mark order paid, validate state transition
> 3. GET /orders - list orders with pagination
> 4. GET /orders/{id} - get single order
> 
> Include audit trail logging per governance constraints. Use structlog for JSON logging with trace_id."

### What AI Should Generate

| Endpoint | Behavior | Governance |
|----------|----------|------------|
| `POST /orders` | Create with idempotency | Audit: order_created |
| `PATCH /orders/{id}/pay` | State transition validation | Audit: order_paid or invalid_transition_attempted |
| `GET /orders` | Paginated list | N/A |
| `GET /orders/{id}` | Single order | N/A |

### Your Verification Checklist

- [ ] Idempotency uses Redis with 60s TTL (matches payment service)
- [ ] State transitions use validate_transition()  not inline if/else
- [ ] Audit entries created for state changes
- [ ] Error responses use structured format: `{"error_code": "..."}`
- [ ] Trace ID propagated from request headers

---

## Step 6: Register Router and Test Health (5 min)

> "/speckit.implement Add the order router to main.py. Verify the app starts and /health returns healthy."

Start the service:

> "Start the development server and hit the health endpoint."

Expected: `{"status": "healthy"}`

---

## Step 7: Write Unit Tests (25 min)

> "/speckit.implement Write unit tests in tests/test_order.py:
> 1. test_create_order_success - valid items create order with 'created' status
> 2. test_create_order_idempotency - same idempotency_key returns same order
> 3. test_mark_order_paid - created order transitions to paid
> 4. test_cannot_pay_cancelled_order - invalid transition returns 400
> 5. test_double_payment_rejected - already paid order returns error
>
> Mock Redis for idempotency. Use pytest-asyncio."

### Verify Test Coverage

Run:

> "Run pytest with coverage. Target: 80%+ on src/app/order.py"

If below 80%, ask AI to add tests for uncovered paths:

> "Coverage is below 80%. Add tests for: [list uncovered lines]"

---

## Step 8: Write Integration Tests (25 min)

This is the critical test  does the full checkout flow work?

> "/speckit.implement Write an end-to-end integration test in tests/test_integration.py:
>
> Test: test_complete_checkout_flow
> 1. POST /orders - create order (assert status='created')
> 2. POST /pay - process payment (mock gateway, assert status='succeeded')
> 3. PATCH /orders/{id}/pay - mark order paid (assert status='paid')
> 4. GET /orders/{id} - verify final state
>
> This proves payment and order services integrate correctly."

### Run the Integration Test

> "Run pytest tests/test_integration.py -v"

**All tests must pass.** If they fail:

1. Read the error message
2. Ask AI to diagnose: "This test failed with [error]. What's wrong?"
3. Fix and re-run

---

## Step 9: Run Security Scan (10 min)

> "Run semgrep security scan on src/. Report any CRITICAL or HIGH findings."

**Pass criteria**: 0 CRITICAL, 0 HIGH

If issues found:

> "Fix this security finding: [paste finding]"

---

## Step 10: Demo Rehearsal (10 min)

Run through the full demo manually:

> "Start the services. I want to test the checkout flow manually:
> 1. Create an order with 2 items
> 2. Process payment for the order total
> 3. Mark the order as paid
> 4. View the order to confirm final state"

**Demo script** (practice saying this):

1. "Customer adds items to cart... creates order... order ID generated"
2. "Customer submits payment... payment processed... transaction ID returned"
3. "System marks order paid... status updated"
4. "Customer views order history... sees the paid order"

---

## Step 11: Commit Your Work (5 min)

> "Commit all changes with message: feat: order management with payment integration"

---

## Success Criteria

Your lab is complete when:

- [ ] `specs/002-order/tasks.md` exists with 15+ tasks
- [ ] `src/app/order.py` exists with 4 endpoints
- [ ] `src/app/state_machine.py` enforces valid transitions
- [ ] `tests/test_order.py` has 5+ unit tests
- [ ] `tests/test_integration.py` has checkout flow test
- [ ] `pytest --cov` shows 80%+ coverage
- [ ] Security scan: 0 CRITICAL, 0 HIGH
- [ ] Manual checkout flow works end-to-end

### Validate Your Work

> "/speckit.checklist"

---

## The AI-Human Partnership

| AI Does | You Verify |
|---------|-----------|
| Generate models from spec | Fields match spec exactly |
| Write state machine logic | Transitions match spec diagram |
| Create endpoint code | Error handling is correct |
| Write test scaffolding | Tests cover edge cases |
| Produce boilerplate | Security scan passes |

**The spec is your contract. AI implements it. You validate the implementation.**

---

## Reflection Questions

1. **AI accuracy**: Did your AI generate correct state transitions on the first try? What did you have to fix?

2. **Verification time**: How much time did you spend writing prompts vs. verifying output? (Target: 30% prompting, 70% verification)

3. **Integration confidence**: Without the spec from Lab 1.4, how would you have known the payment-order integration was correct?

4. **Coverage gaps**: What did the coverage report reveal that your initial tests missed?

---

## Common Mistakes

| Mistake | Impact | Prevention |
|---------|--------|------------|
| Trusting AI output without verification | Bugs in production | Always verify against spec |
| Skipping security scan | Vulnerabilities shipped | Run semgrep before commit |
| Not testing state machine edge cases | Invalid transitions allowed | Write negative tests |
| Hardcoding values AI generates | Config drift | Extract to config.py |

---

## What's Next?

It's **Thursday evening**. Your checkout flow works. Tests pass. Demo scenarios verified.

In **Lab 1.6**, you'll package for production and do a final check. If all goes well, you'll have time for a beer before Friday.

**The AI wrote the code. You wrote the spec that made it correct.**