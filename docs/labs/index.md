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

Start with 2,500+ lines of existing code. Extract specs from working systems.

**Prerequisite:** Part 1 complete with your working payment service.

### Day 1: Understand Before You Change

**The Reality:** You've acquired a company. Their code "just works." Nobody knows why.

| Time | Lab | What You'll Build |
|:-----|:----|:------------------|
| Morning | [Lab 2.0: The Inherited Codebase](lab-2.0-inherited-codebase.md) | System map of a legacy monolith |
| Morning | [Lab 2.1: Extract Specs from Code](lab-2.1-extract-specs.md) | Characterization tests as executable docs |
| Afternoon | [Lab 2.2: Document Business Rules](lab-2.2-document-rules.md) | Magic numbers → human-readable specs |

**Day 1 Outcome:** You know what the legacy system actually does (even if nobody else did).

### Day 2: Integrate Without Breaking

**The Challenge:** Make legacy and new work together. Safely.

| Time | Lab | What You'll Build |
|:-----|:----|:------------------|
| Morning | [Lab 2.3: The Strangler Facade](lab-2.3-strangler-facade.md) | API gateway with gradual traffic shifting |
| Afternoon | [Lab 2.4: Notification Integration](lab-2.4-notifications.md) | Event-driven service across systems |
| Afternoon | [Lab 2.5: Unified Reporting](lab-2.5-unified-reporting.md) | Aggregated views from both systems |
| Afternoon | [Lab 2.6: Migration Planning](lab-2.6-migration-planning.md) | Runbook and rollback procedures |

**Day 2 Outcome:** Integration architecture with a safe migration path.

---

| Part 1 | Part 2 |
|--------|--------|
| Empty repo | Legacy codebase |
| Write the spec | Extract the spec |
| Full control | Constraints everywhere |
| Greenfield freedom | Brownfield reality |

The spec-first discipline you learn in Part 1 becomes **spec-extraction** thinking in Part 2.

---

{: .important }
> Each lab builds on the previous one. If you fall behind, ask your instructor for checkpoint help.

---

## Full Schedule at a Glance

### Part 1: Building from Scratch

| Session | Focus | Labs |
|:--------|:------|:-----|
| Day 1 Morning | Contrast + Specify | Lab 0, Lab 1.1 |
| Day 1 Afternoon | Research + Build | Labs 1.2, 1.3 |
| Day 2 Morning | Second Feature | Lab 1.4 |
| Day 2 Afternoon | Integrate + Ship | Labs 1.5, 1.6 |

### Part 2: Transforming Legacy Code

| Session | Focus | Labs |
|:--------|:------|:-----|
| Day 1 Morning | Explore + Characterize | Labs 2.0, 2.1 |
| Day 1 Afternoon | Document Rules | Lab 2.2 |
| Day 2 Morning | Gateway | Lab 2.3 |
| Day 2 Afternoon | Integrate + Plan | Labs 2.4, 2.5, 2.6 |
