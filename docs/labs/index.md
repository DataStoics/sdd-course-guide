---
title: Lab Schedule
layout: default
nav_exclude: true
permalink: /labs/
---

# Lab Schedule & Resources

Hands-on exercises that build on each other. By the end, you'll have a production-ready checkout system.

---

## Part 1: Building from Scratch

Start with an empty repo and full control. Learn the spec-first discipline.

### Day 1: Spec Monday  Build Tuesday

**The Setup:** Experience what happens without specs, then learn the better way.

| Time | Lab | What You'll Build |
|:-----|:----|:------------------|
| Morning | [Lab 0: AI Without Specs](lab-0-ai-without-specs.md) | 30 min of chaos  AI coding without guidance |
| Morning | [Lab 1.1: Init and Spec](lab-1.1-init-and-spec.md) | Your first governance-ready specification |
| Afternoon | [Lab 1.2: Plan](lab-1.2-plan.md) | Technology decisions with trade-off docs |
| Afternoon | [Lab 1.3: Implementation](lab-1.3-implementation.md) | Working payment endpoint with tests |

**Day 1 Outcome:** Payment service that handles idempotency and passes security scan.

### Day 2: Wednesday Changes  Thursday Demo-Ready

**The Plot Twist:** Your PM wants order tracking too. With specs, it's manageable.

| Time | Lab | What You'll Build |
|:-----|:----|:------------------|
| Morning | [Lab 1.4: Order Spec](lab-1.4-order-spec.md) | Second feature spec with state machine |
| Afternoon | [Lab 1.5: Integration](lab-1.5-integration.md) | Full checkout flow: order  payment  confirmation |
| Afternoon | [Lab 1.6: Production](lab-1.6-production.md) | Docker, CI/CD, final security check |

**Day 2 Outcome:** Complete checkout system ready for the Friday investor demo.

---

## Part 2: Transforming Legacy Code

{: .note }
> **Coming Soon**  Start with 5000+ lines of existing code. Extract specs from working systems.

| Part 1 | Part 2 |
|--------|--------|
| Empty repo | Legacy codebase |
| Write the spec | Extract the spec |
| Full control | Constraints everywhere |
| Greenfield freedom | Brownfield reality |

The spec-first discipline you learn in Part 1 becomes **spec-extraction** thinking in Part 2.

---

{: .important }
> Each lab builds on the previous one. If you fall behind, use checkpoint repositories to catch up.

---

## Part 1 at a Glance

| Session | Focus | Labs |
|:--------|:------|:-----|
| Day 1 Morning | Contrast + Specify | Lab 0, Lab 1.1 |
| Day 1 Afternoon | Research + Build | Labs 1.2, 1.3 |
| Day 2 Morning | Second Feature | Lab 1.4 |
| Day 2 Afternoon | Integrate + Ship | Labs 1.5, 1.6 |
