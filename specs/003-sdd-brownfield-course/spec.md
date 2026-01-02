# Part 2: Transforming Legacy Code - Course Specification

## Overview

**Course Title:** Mastering Spec-Driven Development - Part 2: Brownfield
**Duration:** 2 days (approximately 8 hours hands-on)
**Prerequisites:** Completion of Part 1: Building from Scratch

---

## The Storyline: The Acquisition Integration

### Context

Three months after Part 1's successful investor demo, the company acquired **OrderFlow Inc.**, a smaller competitor. Their order management system handles ~200 orders/day and must be integrated with the payment checkout system built in Part 1.

### The Challenge

> "Great news! We acquired OrderFlow Inc. Their system needs to integrate with ours. 
> Customers shouldn't notice the change, and we can't have downtime during transition.
> 
> **Plot twist:** Their developer left 6 months ago. There's no documentation. 
> The system 'works' but nobody knows exactly how.
> 
> Can you have a migration plan by Friday?"

### Why This Storyline Works

| Criteria | How It's Met |
|----------|--------------|
| **Realistic** | Acquisitions/integrations happen constantly in enterprise |
| **Relatable** | Every developer inherits undocumented code at some point |
| **Connects to Part 1** | Same domain (orders/payments), extends the system they built |
| **Minimal complexity** | No exotic infrastructure, just real patterns |
| **Teaches enterprise patterns** | Strangler pattern, API facades, event-driven architecture |

---

## Learning Objectives

By the end of Part 2, participants will be able to:

1. **Analyze Legacy Code**: Systematically explore undocumented codebases and identify implicit business rules
2. **Extract Specifications**: Reverse-engineer specs from working code using characterization tests
3. **Apply Strangler Pattern**: Incrementally replace legacy components without full rewrites
4. **Design Integration Facades**: Create API layers that abstract legacy complexity
5. **Handle Data Synchronization**: Manage data consistency across systems during migration
6. **Plan Safe Migrations**: Create rollback-capable cutover strategies

---

## Key Differentiators from Part 1

| Part 1 (Greenfield) | Part 2 (Brownfield) |
|---------------------|---------------------|
| Empty repository | 2000+ lines of existing code |
| Write specs first | **Extract** specs from code |
| Choose technology freely | Work within existing constraints |
| Design for compliance | Retrofit compliance |
| `/speckit.specify` (create new) | `/speckit.specify` (extract from existing) |
| Full architectural control | Navigate technical debt |
| Build clean from scratch | Make clean from messy |

---

## The Legacy Monolith

### System Overview

The inherited `legacy-monolith` is a Flask application (~2000 lines) exhibiting classic brownfield challenges:

- **Monolithic architecture**: User management, orders, payments, products all in one file
- **Global state**: In-memory dictionaries instead of proper database
- **No tests**: Zero test coverage, behavior unknown
- **Magic numbers**: Hardcoded business rules scattered throughout
- **Security issues**: MD5 passwords, no auth on admin endpoints
- **Silent failures**: Errors swallowed, inconsistent logging

### Intentional Problems (Teaching Points)

| Problem | Code Location | Learning Opportunity |
|---------|---------------|---------------------|
| MD5 password hashing | `hash_password()` | Security spec extraction |
| 90% random audit logging | `log_action()` | Compliance requirement discovery |
| No auth on admin endpoints | `/admin/*` routes | Access control specification |
| Magic numbers | `MAX_ORDER=10000`, `DISCOUNT_THRESHOLD=500` | Business rule extraction |
| Silent failures | `continue # silently skip` | Error handling specification |
| Global mutable state | `users = {}, orders = {}` | State management patterns |
| No idempotency | `create_order()` | Compare to Part 1 approach |
| Secret backdoor | `X-Admin-Override` header | Security audit finding |
| Inconsistent validation | Various endpoints | Input validation spec |

### What the Legacy System Does

**Core Features:**
- User registration and login (session-based auth)
- Product catalog management
- Shopping cart functionality
- Order creation and lifecycle (pending → paid → shipped → delivered)
- Order cancellation and refunds
- Basic admin functions (user listing, suspension)
- Sales reporting

**Business Rules (Hidden in Code):**
- $100 minimum order, $10,000 maximum
- 10% discount for orders over $500
- 8% tax rate
- $5.99 flat shipping
- Credit users have $5,000 credit limit
- 30-day refund policy (with secret admin override)
- Max 100 items per line item (silently truncated)

---

## External System Integration (Minimal Set)

### Design Principle

Teach enterprise integration patterns with the **simplest possible implementations** that still demonstrate real concepts.

### Integration Points

| System | Purpose | Implementation | Pattern Taught |
|--------|---------|----------------|----------------|
| **Part 1 Payment Service** | Unified payment processing | Reuse from Part 1 | Service composition |
| **Notification Service** | Order confirmations, alerts | New microservice (simple queue) | Async messaging, event-driven |
| **Unified Reporting API** | Combined metrics | Read model endpoint | CQRS concepts, data aggregation |
| **API Gateway** | Request routing | Simple reverse proxy | Strangler pattern, feature flags |

### Explicitly NOT Included

To keep complexity manageable:

- ❌ Full Kafka/RabbitMQ infrastructure
- ❌ Real external APIs (Stripe, Twilio, SendGrid)
- ❌ Database migrations across schemas
- ❌ OAuth2/SSO integration
- ❌ Kubernetes/container orchestration
- ❌ Complex caching layers

---

## Lab Structure

### Day 1: Understand and Extract

| Lab | Duration | Focus | Outcome |
|-----|----------|-------|---------|
| **2.0: The Inherited Codebase** | 45 min | Exploration | Documented system map, identified pain points |
| **2.1: Extract Specs from Code** | 90 min | Characterization testing | Executable specs for core flows |
| **2.2: Document Business Rules** | 60 min | Rule extraction | Business rules specification |

**Day 1 Outcome:** Clear understanding of what the legacy system actually does, documented as specifications.

### Day 2: Integrate and Migrate

| Lab | Duration | Focus | Outcome |
|-----|----------|-------|---------|
| **2.3: The Strangler Facade** | 90 min | API Gateway | Unified endpoint routing both systems |
| **2.4: Notification Integration** | 60 min | Event-driven | Shared notification service |
| **2.5: Unified Reporting** | 45 min | Data aggregation | Combined metrics from both systems |
| **2.6: Migration Planning** | 45 min | Cutover strategy | Production-ready migration plan |

**Day 2 Outcome:** Working integration layer with clear migration path to eventual consolidation.

---

## Lab 2.0: The Inherited Codebase

### Learning Objective

Experience the reality of joining a team with undocumented legacy code. Learn systematic exploration techniques.

### The Scenario

You've just been given access to the OrderFlow Inc. repository. Your first task: figure out what it does.

### Activities

1. **Code Exploration Without Documentation**
   - Run the application
   - Test endpoints manually
   - Document what you discover

2. **Identify the Architecture**
   - What patterns (or anti-patterns) are used?
   - Where is state stored?
   - How do components communicate?

3. **Create a System Map**
   - List all endpoints and their purposes
   - Identify data flows
   - Mark areas of concern

### Key Takeaway

> The only specification is the current behavior. Your job is to make the implicit explicit.

---

## Lab 2.1: Extract Specs from Code

### Learning Objective

Use characterization testing to capture existing behavior as executable specifications.

### The Approach: Extract Specs from Code

Instead of writing specs first (Part 1), you **extract** specs from working code:

```text
/speckit.specify Analyze the order creation flow in legacy-monolith/app.py. 
Extract the implicit business rules and create a specification that 
documents current behavior, including edge cases and error handling.
```

### Activities

1. **Write Characterization Tests**
   - Tests that document current behavior, not desired behavior
   - Capture the order creation happy path
   - Document edge cases (min/max order, discount triggers)

2. **Identify Implicit Contracts**
   - What does `create_order()` actually guarantee?
   - What side effects occur?
   - What validation happens (or doesn't)?

3. **Generate Spec from Tests**
   - Convert test findings into formal specification
   - Mark discrepancies between expected and actual behavior

### Key Insight

> Characterization tests answer: "What does this code actually do?" 
> Not: "What should this code do?"

---

## Lab 2.2: Document Business Rules

### Learning Objective

Extract and formalize hidden business rules that exist only in code.

### The Problem

Business rules are scattered across the codebase:
- `if total > DISCOUNT_THRESHOLD:` — What's the discount policy?
- `if user['type'] == 'credit':` — What are credit user rules?
- `if datetime.now() - created > timedelta(days=30):` — What's the refund policy?

### Activities

1. **Hunt for Magic Numbers**
   - Find all hardcoded values
   - Research their meaning through code context

2. **Map Conditional Logic**
   - Trace decision trees for order states
   - Document the state machine (pending → paid → shipped → delivered)

3. **Create Business Rules Specification**
   - Formalize rules in human-readable format
   - Get stakeholder validation (simulated)

### Deliverable

A `business-rules.md` document capturing:
- Pricing rules (tax, shipping, discounts)
- Order lifecycle states and transitions
- User types and their capabilities
- Refund and cancellation policies

---

## Lab 2.3: The Strangler Facade

### Learning Objective

Implement the strangler pattern to gradually route traffic between legacy and new systems.

### The Pattern

```
┌─────────────┐     ┌──────────────┐
│   Client    │────▶│  API Gateway │
└─────────────┘     └──────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
     ┌─────────────────┐    ┌─────────────────┐
     │  Legacy System  │    │  New Services   │
     │  (OrderFlow)    │    │  (Part 1)       │
     └─────────────────┘    └─────────────────┘
```

### Activities

1. **Create API Gateway**
   - Simple routing based on path/feature flags
   - All traffic goes through gateway

2. **Implement Feature Flags**
   - Toggle between legacy and new implementation
   - Support percentage-based rollout

3. **Add Comparison Mode**
   - Call both systems, compare responses
   - Log discrepancies for analysis

### Key Decisions

- Which endpoints route to which backend?
- How to handle authentication across systems?
- What's the rollback mechanism?

---

## Lab 2.4: Notification Integration

### Learning Objective

Extract notification logic into a shared service using event-driven patterns.

### Current State

Legacy system has notification logic embedded in order processing:
- Email sending (simulated) in `create_order()`
- No retry logic
- No template management

### Target State

Both systems publish events; a shared notification service handles delivery:

```
┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│ Legacy Order │────▶│  Event Queue    │────▶│ Notification │
│ System       │     │  (in-memory)    │     │ Service      │
└──────────────┘     └─────────────────┘     └──────────────┘
        ▲                    ▲                      │
        │                    │                      ▼
┌──────────────┐            │               ┌──────────────┐
│ New Payment  │────────────┘               │ Email/SMS    │
│ Service      │                            │ (mock)       │
└──────────────┘                            └──────────────┘
```

### Activities

1. **Design Event Schema**
   - What events does each system emit?
   - What data does notification service need?

2. **Implement Simple Queue**
   - In-memory queue (no external dependencies)
   - Basic publish/subscribe pattern

3. **Build Notification Service**
   - Subscribe to order events
   - Template-based message generation
   - Retry logic for failures

### Enterprise Patterns Taught

- Event-driven architecture
- Publish/subscribe pattern
- Eventual consistency
- Retry and dead-letter handling

---

## Lab 2.5: Unified Reporting

### Learning Objective

Create a read model that aggregates data from multiple systems.

### The Problem

Management needs combined metrics:
- Total orders (both systems)
- Revenue by time period
- Order status breakdown

### The Solution: Read Model

A separate reporting endpoint that:
1. Queries both systems
2. Transforms and aggregates data
3. Returns unified response

### Activities

1. **Design Report Schema**
   - What metrics are needed?
   - What's the common data model?

2. **Implement Data Aggregation**
   - Query legacy system
   - Query new system
   - Merge and transform

3. **Handle Data Inconsistencies**
   - Different field names
   - Different status values
   - Missing data

### Key Concept: CQRS Lite

> You don't need full CQRS to benefit from separating read and write models.
> A simple aggregation endpoint is often enough.

---

## Lab 2.6: Migration Planning

### Learning Objective

Create a production-ready migration plan with rollback capabilities.

### The Deliverable

A migration plan document covering:

1. **Phase 1: Shadow Mode** (Week 1-2)
   - Gateway routes to legacy
   - New system receives copies, results compared
   - Fix discrepancies

2. **Phase 2: Canary Release** (Week 3-4)
   - 5% traffic to new system
   - Monitor for issues
   - Gradually increase

3. **Phase 3: Full Cutover** (Week 5)
   - 100% traffic to new system
   - Legacy in read-only mode
   - Rollback plan ready

4. **Phase 4: Decommission** (Week 6+)
   - Data migration complete
   - Legacy system archived
   - Documentation updated

### Activities

1. **Define Success Criteria**
   - Response time targets
   - Error rate thresholds
   - Data consistency checks

2. **Create Rollback Procedure**
   - Feature flag switches
   - Data reconciliation
   - Communication plan

3. **Risk Assessment**
   - What could go wrong?
   - How do we detect problems?
   - What's the blast radius?

---

## Success Criteria

### Lab Completion Checklist

- [ ] Legacy system explored and documented
- [ ] Characterization tests capture core flows
- [ ] Business rules extracted and formalized
- [ ] API Gateway routes traffic correctly
- [ ] Feature flags control routing
- [ ] Notification service handles events from both systems
- [ ] Unified reporting aggregates both systems
- [ ] Migration plan documents all phases
- [ ] Rollback procedure tested

### Skills Demonstrated

- [ ] Systematic legacy code exploration
- [ ] Specification extraction from working code
- [ ] Safe refactoring with characterization tests
- [ ] Strangler pattern implementation
- [ ] Event-driven integration design
- [ ] Migration planning with risk assessment

---

## Appendix: Spec-Kit Commands for Brownfield

### Same Commands, Different Context

Part 2 uses the **same spec-kit commands** as Part 1, but with brownfield-appropriate prompts:

| Command | Part 1 Usage | Part 2 Usage |
|---------|--------------|--------------|
| `/speckit.specify` | Create spec from idea | Create spec for NEW components (gateway, notification service) |
| `/speckit.clarify` | Resolve ambiguities | Validate extracted rules with stakeholders |
| `/speckit.plan` | Design greenfield architecture | Plan integration and migration work |
| `/speckit.implement` | Build from spec | Build with awareness of legacy constraints |
| `/speckit.checklist` | Validate completeness | Validate migration readiness |

### Brownfield Prompt Patterns

**Extracting specs from legacy code:**
```text
/speckit.specify Analyze the order creation flow in legacy-monolith/app.py.
Extract the implicit business rules and document them as a specification.
Include: preconditions, validation rules, calculations, side effects.
```

**Creating characterization tests:**
```text
/speckit.implement Create characterization tests for the create_order() function.
Tests should capture CURRENT behavior (not desired behavior).
Include happy path, edge cases, and error conditions.
```

**Documenting business rules:**
```text
/speckit.clarify Review the extracted order specification.
For each magic number and implicit rule, ask me to confirm:
- Is this the intended business rule?
- Should this rule be preserved in the new system?
```

**Planning integration:**
```text
/speckit.plan Plan the API Gateway implementation for the strangler pattern.
Must route to both legacy and new systems based on feature flags.
Include: routing logic, fallback behavior, comparison mode.
```

---

## Appendix: Technical Requirements

### Legacy Monolith

- Python 3.9+
- Flask
- No external database (in-memory state)
- ~2000 lines of intentionally problematic code

### New Components to Build

- API Gateway (Python/FastAPI or Flask)
- Notification Service (Python)
- Simple in-memory event queue
- Unified reporting endpoint

### Development Environment

- Same Codespaces setup as Part 1
- Part 1 payment service running alongside
- Legacy monolith as second service

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-01-02 | Course Team | Initial draft |
