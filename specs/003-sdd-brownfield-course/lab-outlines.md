# Part 2 Lab Outlines

Detailed structure for each lab in Part 2: Transforming Legacy Code.

---

## Lab 2.0: The Inherited Codebase (45 min)

### Metadata

- **Duration**: 45 minutes
- **Day**: 1
- **Position**: Opening lab
- **Prerequisites**: Part 1 complete, development environment ready

### Learning Objective

Experience the reality of inheriting undocumented legacy code. Develop systematic exploration techniques before attempting any changes.

### The Scenario

**Monday Morning Message:**

> "Welcome to the OrderFlow integration team! Here's the repo access. 
> The previous developer left 6 months ago. There's no documentation.
> The system handles about 200 orders/day and 'just works.'
> 
> First task: Tell us what this thing actually does."

### Step 1: First Contact (10 min)

**Run the application:**
```bash
cd legacy-monolith
pip install -r requirements.txt
python app.py
```

**Explore without reading code:**
- Hit the `/health` endpoint
- Try registering a user
- Browse products
- Attempt to create an order

**Document your findings:**
- What endpoints exist?
- What works? What fails?
- Any obvious error messages?

### Step 2: Code Archaeology (20 min)

**Read `app.py` systematically:**

1. **Imports and setup** — What frameworks/libraries?
2. **Global state** — Where is data stored?
3. **Routes** — What endpoints are defined?
4. **Helper functions** — What utility code exists?

**Create a system map:**

```markdown
## Legacy System Endpoints

| Method | Path | Purpose | Notes |
|--------|------|---------|-------|
| POST | /register | User registration | No email validation |
| POST | /login | Authentication | Returns session ID |
| GET | /products | List products | No pagination! |
| ... | ... | ... | ... |
```

### Step 3: Identify Red Flags (15 min)

Hunt for concerning patterns:

- [ ] Security issues (auth, passwords, secrets)
- [ ] Data integrity risks (validation, constraints)
- [ ] Reliability concerns (error handling, logging)
- [ ] Scalability issues (global state, no pagination)

**Document each finding:**
```markdown
## Red Flag: MD5 Password Hashing

**Location**: `hash_password()` function, line ~45
**Risk**: High - MD5 is cryptographically broken
**Impact**: User credentials at risk if database compromised
**Recommendation**: Migrate to bcrypt before production integration
```

### Success Criteria

- [ ] Application runs locally
- [ ] All endpoints documented
- [ ] System map created
- [ ] At least 5 red flags identified with locations
- [ ] No code changes made (exploration only)

### Key Takeaway

> "The most dangerous words in software: 'It works, don't touch it.'
> Before you can fix it, you must understand it."

---

## Lab 2.1: Extract Specs from Code (90 min)

### Metadata

- **Duration**: 90 minutes
- **Day**: 1
- **Position**: After Lab 2.0
- **Prerequisites**: System map from Lab 2.0

### Learning Objective

Use characterization testing to capture existing behavior as executable specifications. Learn to document what code *does*, not what it *should* do.

### The Challenge

> "We need to integrate this system, but we can't break existing behavior.
> How do we know what 'existing behavior' even is?"

### Step 1: Set Up Characterization Testing (15 min)

Create test infrastructure:

```python
# tests/test_characterization.py
"""
Characterization Tests for Legacy Order System

These tests document CURRENT BEHAVIOR, not desired behavior.
If a test fails, the system behavior changed - investigate before "fixing" the test.
"""

import pytest
from app import app, users, orders, products

@pytest.fixture
def client():
    app.config['TESTING'] = True
    # Reset global state
    users.clear()
    orders.clear()
    products.clear()
    # Initialize test products
    products['prod_001'] = {
        'id': 'prod_001',
        'name': 'Test Widget',
        'price': 100.00,
        'stock': 50,
        'category': 'test'
    }
    with app.test_client() as client:
        yield client
```

### Step 2: Characterize the Happy Path (30 min)

Write tests that capture the order creation flow:

```text
/speckit.analyze Analyze the order creation flow in the legacy system.
Create characterization tests that document:
1. User registration and login
2. Adding items to cart
3. Creating an order
4. The exact response format and side effects
```

**Expected test coverage:**
- User registration → login → session token
- Add to cart → verify cart state
- Create order → verify order created, cart cleared, stock reduced

### Step 3: Characterize Edge Cases (30 min)

Document boundary behavior:

```text
/speckit.characterize What happens when:
- Order total is exactly $100 (minimum)?
- Order total is exactly $10,000 (maximum)?
- Order total is exactly $500 (discount threshold)?
- Cart contains product with 0 stock?
- User is a 'credit' type approaching $5,000 limit?
```

**Key discovery:** Does the system handle edge cases gracefully or fail silently?

### Step 4: Generate Specification (15 min)

Convert test findings into formal spec:

```text
/speckit.specify Based on the characterization tests, create a specification
for the order creation feature documenting current behavior including:
- Preconditions (user logged in, cart not empty)
- Business rules (min/max amounts, discounts, tax)
- Postconditions (order created, cart cleared, stock updated)
- Error conditions (what returns 400, 401, 500)
```

### Deliverables

1. `tests/test_characterization.py` — Executable behavior documentation
2. `specs/legacy-order-creation/spec.md` — Formal specification extracted from tests

### Success Criteria

- [ ] Characterization tests pass against current code
- [ ] Happy path fully documented with expected values
- [ ] At least 5 edge cases captured
- [ ] Spec accurately reflects actual (not desired) behavior
- [ ] Any "surprising" behaviors documented

### Key Insight

> "Characterization tests are a safety net for refactoring.
> They tell you: 'This is how it works today.'
> Breaking one means you changed behavior — intentionally or not."

---

## Lab 2.2: Document Business Rules (60 min)

### Metadata

- **Duration**: 60 minutes
- **Day**: 1
- **Position**: After Lab 2.1
- **Prerequisites**: Characterization tests from Lab 2.1

### Learning Objective

Extract implicit business rules from code and formalize them into stakeholder-readable documentation.

### The Problem

Business rules are hidden in code:

```python
# What IS this? Why 500? Who decided 10%?
if total > DISCOUNT_THRESHOLD:
    discount = total * DISCOUNT_RATE
```

### Step 1: Hunt Magic Numbers (20 min)

Find all hardcoded values:

```text
/speckit.extract-rules Scan the legacy codebase for:
1. Magic numbers (hardcoded numeric values)
2. Magic strings (hardcoded status values, error messages)
3. Implicit thresholds (conditional boundaries)

For each, identify: value, location, apparent purpose, confidence level.
```

**Expected findings:**

| Constant | Value | Location | Purpose | Confidence |
|----------|-------|----------|---------|------------|
| MAX_ORDER | 10000 | line 15 | Maximum order amount | High |
| MIN_ORDER | 100 | line 14 | Minimum order amount | High |
| TAX_RATE | 0.08 | line 16 | Tax percentage | High |
| SHIPPING | 5.99 | line 17 | Flat shipping rate | High |
| DISCOUNT_THRESHOLD | 500 | line 18 | Discount eligibility | Medium |
| DISCOUNT_RATE | 0.1 | line 19 | Discount percentage | Medium |
| 5000 | (inline) | credit check | Credit limit? | Low |
| 30 | (inline) | refund check | Refund window days? | Low |

### Step 2: Map State Transitions (20 min)

Document the order lifecycle:

```text
/speckit.analyze Map the order state machine from the legacy code.
What states exist? What transitions are allowed? What triggers each transition?
```

**Expected state diagram:**

```
                    ┌──────────────┐
                    │   pending    │
                    └──────────────┘
                           │
              ┌────────────┼────────────┐
              │ pay        │            │ cancel
              ▼            │            ▼
       ┌──────────┐        │     ┌────────────┐
       │   paid   │        │     │ cancelled  │
       └──────────┘        │     └────────────┘
              │            │
              │ ship       │
              ▼            │
       ┌──────────┐        │
       │ shipped  │────────┘ (can't cancel after ship)
       └──────────┘
              │
              │ deliver
              ▼
       ┌──────────┐
       │delivered │
       └──────────┘
              │
              │ refund (within 30 days)
              ▼
       ┌──────────┐
       │ refunded │
       └──────────┘
```

### Step 3: Create Business Rules Document (20 min)

Formalize findings:

```text
/speckit.specify Create a business-rules.md document that captures
all discovered business rules in stakeholder-friendly language.
Include: pricing rules, order policies, user types, refund policies.
Flag any rules that seem inconsistent or risky.
```

### Deliverable

`specs/legacy-system/business-rules.md`:

```markdown
# OrderFlow Inc. Business Rules

## Pricing

| Rule | Value | Source |
|------|-------|--------|
| Tax Rate | 8% | `TAX_RATE = 0.08` |
| Shipping | $5.99 flat | `SHIPPING = 5.99` |
| Volume Discount | 10% off orders > $500 | `DISCOUNT_*` constants |

## Order Constraints

- Minimum order: $100
- Maximum order: $10,000
- Maximum items per line: 100 (silently truncated!)

## User Types

- `regular`: Standard checkout
- `credit`: Can accumulate balance up to $5,000

## Refund Policy

- 30-day window for refunds
- Admin override available (security concern!)
- Refunds restore user credit balance

## ⚠️ Flagged Issues

1. **Silent truncation**: Orders >100 items truncated without warning
2. **Admin backdoor**: `X-Admin-Override` header bypasses refund window
3. **Random logging**: Only 90% of actions logged (compliance risk)
```

### Success Criteria

- [ ] All magic numbers identified with purposes
- [ ] State machine documented
- [ ] Business rules formalized in stakeholder language
- [ ] Risky or inconsistent rules flagged
- [ ] Document suitable for business review

### Key Insight

> "The code is the only source of truth for legacy systems.
> Your job is to translate code-truth into human-truth."

---

## Lab 2.3: The Strangler Facade (90 min)

### Metadata

- **Duration**: 90 minutes
- **Day**: 2
- **Position**: Opening lab Day 2
- **Prerequisites**: Labs 2.0-2.2 complete

### Learning Objective

Implement the strangler pattern to gradually route traffic between legacy and new systems without downtime.

### The Pattern

Instead of a risky big-bang migration:

1. Put a facade (API gateway) in front of both systems
2. Route all traffic through the gateway
3. Gradually shift traffic to the new system
4. Eventually retire the legacy system

### Step 1: Create the Gateway Spec (20 min)

```text
/speckit.specify Create a specification for an API Gateway that:
1. Accepts all requests currently handled by legacy system
2. Routes to either legacy or new system based on feature flags
3. Supports percentage-based traffic splitting
4. Logs routing decisions for analysis
5. Can fallback to legacy on new system errors
```

### Step 2: Implement Basic Gateway (40 min)

```text
/speckit.plan Plan the API Gateway implementation:
- Technology: FastAPI (matches Part 1 stack)
- Routing: Path-based + feature flag
- Feature flags: Environment variables initially
- Logging: Structured JSON for analysis
```

**Core routing logic:**

```python
@app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE"])
async def gateway(request: Request, path: str):
    # Determine target based on feature flags
    if should_route_to_new_system(path):
        return await proxy_to_new_system(request, path)
    else:
        return await proxy_to_legacy_system(request, path)
```

### Step 3: Add Feature Flags (15 min)

Implement routing control:

```python
FEATURE_FLAGS = {
    "payment": {
        "enabled": True,
        "percentage": 100  # All payment traffic to new system
    },
    "orders": {
        "enabled": True,
        "percentage": 0  # All order traffic to legacy (for now)
    }
}
```

### Step 4: Implement Comparison Mode (15 min)

For validation before cutover:

```python
async def compare_mode(request, path):
    """Call both systems, compare responses, log discrepancies."""
    legacy_response = await proxy_to_legacy(request, path)
    new_response = await proxy_to_new(request, path)
    
    if not responses_match(legacy_response, new_response):
        log_discrepancy(path, legacy_response, new_response)
    
    return legacy_response  # Always return legacy during comparison
```

### Success Criteria

- [ ] Gateway proxies requests to legacy system
- [ ] Feature flags control routing
- [ ] Payment endpoints route to Part 1 service
- [ ] Order endpoints route to legacy
- [ ] Comparison mode logs discrepancies
- [ ] Fallback works when new system fails

### Key Insight

> "The strangler pattern is about reducing risk, not achieving perfection.
> You can always route back to legacy if something goes wrong."

---

## Lab 2.4: Notification Integration (60 min)

### Metadata

- **Duration**: 60 minutes
- **Day**: 2
- **Position**: After Lab 2.3
- **Prerequisites**: Gateway from Lab 2.3

### Learning Objective

Extract notification logic into a shared service using event-driven patterns.

### Current State Problem

Legacy system has notification logic embedded:
- In `create_order()`: sends order confirmation
- In `ship_order()`: sends shipping notification
- No retry logic, no templates, no unified approach

### Target Architecture

```
┌──────────────┐          ┌─────────────┐          ┌──────────────┐
│ Legacy       │──event──▶│   Event     │──event──▶│ Notification │
│ Order System │          │   Queue     │          │ Service      │
└──────────────┘          └─────────────┘          └──────────────┘
                                ▲
┌──────────────┐               │
│ New Payment  │───event───────┘
│ Service      │
└──────────────┘
```

### Step 1: Design Event Schema (15 min)

```text
/speckit.specify Design the event schema for order notifications:
- What events exist? (order.created, order.paid, order.shipped)
- What data does each event carry?
- How do we ensure events are delivered?
```

**Event schema:**

```json
{
  "event_type": "order.created",
  "event_id": "evt_abc123",
  "timestamp": "2026-01-02T10:30:00Z",
  "source": "legacy-orders",
  "data": {
    "order_id": "ord_xyz789",
    "customer_email": "user@example.com",
    "total": 299.99,
    "items": [...]
  }
}
```

### Step 2: Implement Simple Event Queue (20 min)

No Kafka/RabbitMQ — just demonstrate the pattern:

```python
# event_queue.py
from collections import deque
from threading import Lock

class SimpleEventQueue:
    def __init__(self):
        self._queue = deque()
        self._lock = Lock()
        self._subscribers = []
    
    def publish(self, event):
        with self._lock:
            self._queue.append(event)
            for subscriber in self._subscribers:
                subscriber(event)
    
    def subscribe(self, callback):
        self._subscribers.append(callback)
```

### Step 3: Build Notification Service (25 min)

```text
/speckit.implement Create a notification service that:
1. Subscribes to order events
2. Selects appropriate template based on event type
3. Formats and "sends" notification (log for demo)
4. Handles failures with retry logic
```

**Template approach:**

```python
TEMPLATES = {
    "order.created": "Your order {order_id} has been received! Total: ${total}",
    "order.shipped": "Your order {order_id} is on its way! Tracking: {tracking}",
    "payment.completed": "Payment of ${amount} received. Thank you!"
}
```

### Success Criteria

- [ ] Event queue accepts and delivers events
- [ ] Legacy system publishes events (minimal code change)
- [ ] New payment service publishes events
- [ ] Notification service processes events from both sources
- [ ] Failed notifications are retried

### Key Insight

> "Event-driven architecture decouples systems.
> The legacy system doesn't need to know about the notification service.
> It just publishes events; someone else handles delivery."

---

## Lab 2.5: Unified Reporting (45 min)

### Metadata

- **Duration**: 45 minutes
- **Day**: 2
- **Position**: After Lab 2.4
- **Prerequisites**: Gateway and both systems running

### Learning Objective

Create a read model that aggregates data from multiple systems into a unified view.

### The Business Need

Management wants ONE dashboard showing:
- Total orders (from both systems)
- Revenue by period
- Order status breakdown
- Customer metrics

### Step 1: Design Unified Schema (10 min)

Map data models from both systems:

| Legacy Field | New System Field | Unified Field |
|--------------|------------------|---------------|
| `total` | `amount` | `order_total` |
| `status` | `order_status` | `status` |
| `created` | `created_at` | `created_timestamp` |

### Step 2: Implement Aggregation Endpoint (25 min)

```text
/speckit.implement Create a reporting endpoint in the gateway that:
1. Queries both legacy and new order systems
2. Transforms data to common schema
3. Aggregates metrics (totals, counts, averages)
4. Returns unified JSON response
```

**Implementation approach:**

```python
@app.get("/reports/sales")
async def unified_sales_report(start_date: date, end_date: date):
    # Query both systems
    legacy_data = await fetch_legacy_orders(start_date, end_date)
    new_data = await fetch_new_orders(start_date, end_date)
    
    # Transform to common format
    all_orders = [
        *transform_legacy_orders(legacy_data),
        *transform_new_orders(new_data)
    ]
    
    # Aggregate
    return {
        "total_orders": len(all_orders),
        "total_revenue": sum(o["order_total"] for o in all_orders),
        "by_status": aggregate_by_status(all_orders),
        "by_source": {
            "legacy": len(legacy_data),
            "new": len(new_data)
        }
    }
```

### Step 3: Handle Inconsistencies (10 min)

What happens when data doesn't match?

- Missing fields → default values
- Different date formats → normalize
- Different status values → mapping table
- Duplicate detection → order ID prefix by source

### Success Criteria

- [ ] Single endpoint returns data from both systems
- [ ] Data correctly transformed to common schema
- [ ] Aggregations calculate correctly
- [ ] Source attribution visible (legacy vs new)
- [ ] Handles missing/malformed data gracefully

### Key Insight (CQRS Lite)

> "You don't need full CQRS to benefit from read/write separation.
> A simple aggregation layer gives you unified views without 
> rewriting either system."

---

## Lab 2.6: Migration Planning (45 min)

### Metadata

- **Duration**: 45 minutes
- **Day**: 2
- **Position**: Final lab
- **Prerequisites**: All previous labs complete

### Learning Objective

Create a production-ready migration plan with rollback capabilities and risk mitigation.

### The Deliverable

A migration plan document that could actually be executed.

### Step 1: Define Migration Phases (15 min)

```text
/speckit.specify Create a phased migration plan:
- Phase 1: Shadow mode (compare without switching)
- Phase 2: Canary release (small percentage)
- Phase 3: Gradual rollout (increasing percentage)
- Phase 4: Full cutover
- Phase 5: Legacy decommission

For each phase: duration, success criteria, rollback trigger.
```

**Phase template:**

```markdown
## Phase 2: Canary Release (Week 3-4)

**Duration**: 2 weeks
**Traffic Split**: 5% → 25% new system

### Entry Criteria
- [ ] Phase 1 complete with <0.1% discrepancy rate
- [ ] Rollback tested and documented
- [ ] On-call team briefed

### Success Criteria
- [ ] Error rate <1%
- [ ] P99 latency within 10% of legacy
- [ ] No data inconsistencies detected
- [ ] Customer complaints within normal range

### Rollback Trigger
- Error rate >5% sustained for 5 minutes
- Any data corruption detected
- Critical customer complaint

### Rollback Procedure
1. Set feature flag `orders.enabled = false`
2. Verify all traffic routes to legacy
3. Investigate root cause
4. Do NOT proceed until resolved
```

### Step 2: Risk Assessment (15 min)

Identify what could go wrong:

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Data sync failure | Medium | High | Comparison mode catches discrepancies before cutover |
| Performance degradation | Low | Medium | Canary catches issues at low traffic |
| Feature parity gap | Medium | High | Characterization tests ensure behavior match |
| Rollback failure | Low | Critical | Test rollback in staging before each phase |

### Step 3: Create Runbook (15 min)

Operational documentation:

```text
/speckit.specify Create an operations runbook covering:
1. How to check system health during migration
2. How to adjust traffic percentages
3. How to perform emergency rollback
4. How to investigate discrepancies
5. Escalation contacts and procedures
```

### Success Criteria

- [ ] All phases documented with clear criteria
- [ ] Rollback procedures tested
- [ ] Risk assessment complete
- [ ] Runbook covers operational scenarios
- [ ] Plan reviewed and approved (simulated)

### Key Insight

> "The migration plan is a spec for the migration itself.
> Just like code specs prevent rework, migration specs prevent outages."

---

## Summary: Part 2 Progression

```
Day 1: UNDERSTAND                    Day 2: INTEGRATE
┌─────────────────────────────┐     ┌─────────────────────────────┐
│ Lab 2.0: Explore Legacy     │     │ Lab 2.3: Strangler Facade   │
│ Lab 2.1: Extract Specs      │────▶│ Lab 2.4: Notifications      │
│ Lab 2.2: Document Rules     │     │ Lab 2.5: Unified Reporting  │
└─────────────────────────────┘     │ Lab 2.6: Migration Plan     │
                                    └─────────────────────────────┘
           ▼                                     ▼
   "What does this do?"              "How do we safely change it?"
```

### Part 1 vs Part 2 Skill Transfer

| Part 1 Skill | Part 2 Application |
|--------------|-------------------|
| Write clear specs | Extract specs from code |
| Plan before building | Plan migrations, not greenfield |
| Test-driven development | Characterization testing |
| Clean architecture | Strangler pattern integration |
| API design | Unified facades and aggregation |
| Deployment | Migration planning with rollback |
