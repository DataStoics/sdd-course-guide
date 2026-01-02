---
title: "Lab 1.5: Integrate Payment + Order – Thursday Build Day"
layout: default
parent: "Part 1: Building from Scratch"
nav_order: 7
---

# Lab 1.5: Integrate Payment + Order -- Thursday Build Day

**Duration**: 150 minutes  
**Day**: 2 (Afternoon)  
**Prerequisites**: Completed Lab 1.4 with order specification

---

## Learning Objective

Build the order service, wire it to payments, and prove the full checkout flow works. By end of day Thursday, you'll have a complete demo: create order - pay - see order history.

---

## Course Progress

![Lab 1.5 Progress](../assets/images/lab-1.5-progress.svg)

---

## Where We Are in the Week

- **Monday**: [DONE] Payment spec + plan
- **Tuesday**: [DONE] Payment implementation
- **Wednesday**: [DONE] Order spec (Lab 1.4)
- **Thursday**: [HERE] Build + Integrate
- **Friday**: Demo day

**Thursday goal**: Full checkout flow working, tests passing, demo-ready.

---

## Starting Point

- Working payment service from Lab 1.3
- Complete order specification from Lab 1.4
- Integration points documented

---

## Step 1: Generate Order Tasks (15 min)

Run the tasks command for the order spec:

```
/speckit.tasks
```

Your `specs/002-order/tasks.md` should include:

1. **Database setup tasks**: Models, migrations
2. **State machine tasks**: Status enum, transition logic
3. **Endpoint tasks**: Create, update status, get history
4. **Audit tasks**: Audit entry creation
5. **Integration tasks**: Payment callback handling
6. **Test tasks**: Unit and integration tests

---

## Step 2: Implement Order Models (20 min)

Ask your AI to implement the order models:

> "Based on the order spec and data model, implement the Order models in `src/app/models.py`. Include OrderStatus enum with states from the spec (created, paid, fulfilled, cancelled, refunded, archived), OrderItem, CreateOrderRequest, OrderResponse, MarkOrderPaidRequest, and OrderAuditEntry for the immutable audit trail."

The AI will create Pydantic models that match your spec's data model.

---

## Step 3: Implement State Machine (15 min)

Ask your AI to implement the state machine:

> "Create a state machine module at `src/app/state_machine.py` that enforces valid order transitions based on the spec. Include a VALID_TRANSITIONS dictionary mapping each OrderStatus to its allowed next states, an InvalidTransitionError exception, and validate_transition function."

The AI will generate:
- Transition rules matching your spec's state diagram
- Clear error messages for invalid transitions
- Helper functions for checking transitions

---

## Step 4: Implement Order Endpoints (40 min)

Ask your AI to implement the order service:

> "Implement the order endpoints in `src/app/order.py` based on the order spec. Include: POST /orders to create an order with idempotency, PATCH /orders/{id}/pay to mark as paid with state machine validation, GET /orders for paginated history, GET /orders/{id} for single order. Each state change must create an audit entry per governance constraints."

The AI will create endpoints that:
- Use Redis for idempotency (like the payment service)
- Enforce state transitions via the state machine
- Create immutable audit entries for every change
- Return proper error codes for invalid operations

---

## Step 5: Register Order Router (5 min)

Ask your AI:

> "Register the order router in main.py so the order endpoints are available."

---

## Step 6: Write Order Unit Tests (25 min)

Ask your AI to generate unit tests:

> "Create unit tests in `tests/test_order.py` for the order endpoints. Test create order success and validation, mark order paid with state transitions, and order history retrieval. Reference the acceptance scenarios from the spec."

The AI will generate tests covering:
- **TestCreateOrder**: Valid order creation, empty items rejection, idempotency
- **TestMarkOrderPaid**: State transition from created to paid, transaction ID linking
- **TestOrderHistory**: Empty history, pagination

---

## Step 7: Write Integration Tests (30 min)

Ask your AI to generate end-to-end tests:

> "Create integration tests in `tests/test_integration.py` for the complete checkout flow: create order, process payment, mark order paid, verify final state. Also test edge cases: cannot pay cancelled order, double payment rejected."

The AI will generate E2E tests covering:
- **Complete checkout flow**: Order created → Payment processed → Order marked paid
- **State machine enforcement**: Cannot pay a cancelled order
- **Idempotency**: Double payment returns error, not duplicate charge

---

## Step 8: Run Coverage Check (10 min)

Ask your AI:

> "Run pytest with coverage and verify we have at least 80% coverage. Show me what's missing."

**Target**: 80%+ coverage

If coverage is below 80%, ask:
> "Add tests to cover the missing lines, especially edge cases for order not found, invalid items, and audit trail entries."

---

## Step 9: Run Security Scan (5 min)

Ask your AI:

> "Run security scans on the source code using semgrep and bandit. Report any critical or high severity findings."

**Pass Criteria**: 0 CRITICAL + 0 HIGH

---

## Step 10: Commit Your Work (2 min)

Ask your AI:

> "Commit all the order implementation and integration tests with a conventional commit message."

---

## Success Criteria

Your lab is complete when:

- [ ] `src/app/order.py` exists with order endpoints
- [ ] `src/app/models.py` includes Order models and OrderStatus enum
- [ ] `tests/test_order.py` exists with order unit tests
- [ ] `tests/test_integration.py` exists with e2e checkout flow test
- [ ] State machine enforces valid transitions only
- [ ] `pytest --cov` shows 80%+ coverage
- [ ] E2E test passes: order - payment - order paid

### Validate Your Work

Ask your AI:

> "Run the lab validation script for Lab 1.5 with coverage check and show me the results."

---

## Reflection Questions

1. **Demo readiness**: Could you demo this to investors right now? What would you polish tonight?

2. **Integration confidence**: Without the integration spec in Lab 1.4, how long would debugging "payment and order don't connect" have taken?

3. **State machine win**: Did your AI generate valid transition logic? How much did the state diagram in the spec help?

4. **Lab 0 callback**: In the Lab 0 approach, where would you be right now? (Hint: probably fixing integration bugs)

---

## Common Mistakes

| Mistake | Demo Day Impact |
|---------|-----------------|
| No state machine validation | Invalid orders crash the demo |
| Missing audit entries | "Can you trace this?" -- "Uh..." |
| Tests mock everything | "Run it live" -- fails |
| Forgot to register router | 404 on demo day |
| Coverage < 80% | Untested path crashes |

---

## What's Next?

It's **Thursday evening**. Your checkout flow works. Tests pass. Demo scenarios verified.

In **Lab 1.6**, you'll package for production and do a final check. If all goes well, you'll have time for a beer before Friday.

**The Lab 0 code you would've been rewriting from scratch right now.**
