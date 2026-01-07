---
title: "Part 2: Transforming Legacy Code"
layout: default
nav_order: 4
has_children: true
permalink: /labs/part2/
---

# Part 2: Transforming Legacy Code

{: .note }
> **Coming Soon** — Most real-world work isn't greenfield. Part 2 tackles the hard stuff.

---

## The Storyline: The Acquisition Integration

Three months after your successful Friday demo, the company acquired **OrderFlow Inc.** — a smaller competitor with ~200 orders/day. Now you need to integrate their system with yours.

### The Challenge

> "Great news! We acquired OrderFlow Inc. Their order system needs to integrate with our payment checkout.
> 
> **Plot twist:** Their developer left 6 months ago. There's no documentation. The system 'works' but nobody knows exactly how.
> 
> Can you have a migration plan by Friday?"

---

## What's Different About Brownfield?

| Part 1: Building from Scratch | Part 2: Transforming Legacy Code |
|:------------------------------|:---------------------------------|
| Empty repository | 2,000+ lines of existing code |
| You write the spec | You **extract** the spec |
| `/speckit.specify` (create) | `/speckit.analyze` (reverse-engineer) |
| Full technology control | Tech debt constrains choices |
| Build compliance in | Retrofit compliance |
| Design clean architecture | Navigate existing complexity |

---

## What You'll Build

### Day 1: Understand and Extract

| Lab | Duration | What You'll Learn |
|:----|:---------|:------------------|
| **Lab 2.0: The Inherited Codebase** | 45 min | Systematic legacy code exploration |
| **Lab 2.1: Extract Specs from Code** | 90 min | Characterization testing, behavior documentation |
| **Lab 2.2: Document Business Rules** | 60 min | Finding hidden rules in magic numbers |

**Day 1 Outcome:** Clear specification of what the legacy system actually does.

### Day 2: Integrate and Migrate

| Lab | Duration | What You'll Build |
|:----|:---------|:------------------|
| **Lab 2.3: The Strangler Facade** | 90 min | API Gateway routing to both systems |
| **Lab 2.4: Notification Integration** | 60 min | Shared event-driven notification service |
| **Lab 2.5: Unified Reporting** | 45 min | Combined metrics from both systems |
| **Lab 2.6: Migration Planning** | 45 min | Production-ready cutover strategy |

**Day 2 Outcome:** Working integration layer with clear migration path.

---

## The Legacy Monolith

You'll work with a Flask application exhibiting classic brownfield challenges:

| Problem | Teaching Opportunity |
|---------|---------------------|
| MD5 password hashing | Security spec extraction |
| 90% random audit logging | Compliance requirement discovery |
| No auth on admin endpoints | Access control specification |
| Magic numbers everywhere | Business rule extraction |
| Silent failures | Error handling specification |
| No tests | Characterization testing |

### Business Rules Hidden in Code

```python
# What IS this? Why 500? Who decided 10%?
if total > DISCOUNT_THRESHOLD:
    discount = total * DISCOUNT_RATE
```

Your job: Make the implicit explicit.

---

## Enterprise Integration (Minimal Set)

Part 2 teaches integration patterns with **minimal external dependencies**:

| Integration | Pattern Taught |
|-------------|----------------|
| Part 1 Payment Service | Service composition |
| Notification Service | Event-driven architecture |
| Unified Reporting | CQRS-lite, data aggregation |
| API Gateway | Strangler pattern, feature flags |

**Not included** (keeps training focused):
- ❌ Full Kafka/RabbitMQ infrastructure
- ❌ Real external APIs (Stripe, Twilio)
- ❌ Complex database migrations
- ❌ Kubernetes orchestration

---

## Key Skills

By completing Part 2, you'll be able to:

1. **Analyze Legacy Code** — Systematically explore undocumented systems
2. **Extract Specifications** — Reverse-engineer specs using characterization tests
3. **Apply Strangler Pattern** — Incrementally replace legacy without rewrites
4. **Design Integration Facades** — Abstract legacy complexity behind clean APIs
5. **Handle Data Synchronization** — Manage consistency across systems
6. **Plan Safe Migrations** — Create rollback-capable cutover strategies

---

## Prerequisites

{: .important }
> Complete [Part 1: Building from Scratch](../part1/) first. The greenfield discipline is prerequisite knowledge for brownfield work.

The spec-first discipline you learned in Part 1 becomes **spec-extraction** thinking in Part 2.

---

**Part 2 labs coming soon.**

[Review Part 1](../part1/){: .btn .btn-outline .fs-5 .mb-4 .mb-md-0 }
