---
title: "Lab 2.1: Extract Specs from Code"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 2
---

# Lab 2.1: Extract Specs from Code
{: .fs-9 }

What Does It Actually Do?
{: .fs-6 .fw-300 }

~90 min
{: .label .label-blue }

Implementation-Heavy
{: .label .label-yellow }

---

## Learning Objective

Use characterization testing to capture existing behavior as executable specifications. Learn to document what code *does*, not what it *should* do.

> **Tests don't prove code is correct. They prove code hasn't changed.**
{: .highlight }

---

## The Challenge

Your PM asks a reasonable question:

> "We need to integrate this system, but we can't break existing behavior.
> How do we know what 'existing behavior' even is?"

**The answer**: Characterization tests. Tests that capture *current behavior*, not *desired behavior*.

If a characterization test fails after a change, you changed behavior -- investigate before "fixing" the test.

---

## The SDD Approach to Legacy Code

In Part 1, the flow was:

```
Spec → Plan → Code → Tests
```

In Part 2 (brownfield), we reverse it:

```
Code → Tests → Spec (extracted)
```

You discover the implicit specification by testing what the code actually does.

---

## Step 1: Set Up Characterization Testing (15 min)

Create test infrastructure in the **legacy repo** (yes, we're adding tests to their code):

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Create a characterization test file at @legacy-orderflow/tests/test_characterization.py:
1. Import pytest, app, and all global state (users, orders, products, etc.)
2. Create a fixture that resets global state and seeds test products
3. Add a docstring explaining these are characterization tests (document current behavior)
4. Include a helper function to register and login a test user
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From legacy-orderflow directory:

```text
Create tests/test_characterization.py:
1. Import pytest, app, and all global state (users, orders, products, etc.)
2. Create a fixture that resets global state and seeds test products
3. Add a docstring explaining these are characterization tests (document current behavior)
4. Include a helper function to register and login a test user
```

</details>

### Expected Test Setup

```python
"""
Characterization Tests for Legacy Order System

These tests document CURRENT BEHAVIOR, not desired behavior.
If a test fails, the system behavior CHANGED - investigate before "fixing" the test.

WARNING: Do not modify these tests to make them pass. 
If behavior changed, update the spec and get approval first.
"""

import pytest
from app import app, users, orders, products, sessions, carts

@pytest.fixture
def client():
    """Reset all global state for isolated tests."""
    app.config['TESTING'] = True
    users.clear()
    orders.clear()
    products.clear()
    sessions.clear()
    carts.clear()
    # Seed test data...
    with app.test_client() as client:
        yield client
```

---

## Step 2: Characterize the Happy Path (30 min)

Document the order creation flow -- the core behavior that must not break.

### Extract the Order Flow Spec

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Analyze the order creation flow in @legacy-orderflow/app.py.
Extract the implicit specification by documenting:
1. User registration and login (exact payload formats, response shapes)
2. Adding items to cart (rules, limits, validation)
3. Creating an order (business rules: min/max amounts, tax, shipping, discounts)
4. The exact response format and side effects (cart cleared, stock reduced, etc.)

Create the spec in .specify/legacy-integration/order-creation.md
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo:

```text
/speckit.specify

Analyze ../legacy-orderflow/app.py and extract the order creation flow.
Document: 
- Registration/login payload formats and responses
- Cart operations (add, update, remove, limits)
- Order creation rules (min $100, max $10K, tax, shipping, discounts)
- Exact response shapes and side effects

Create .specify/legacy-integration/order-creation.md
```

</details>

### Write Tests That Document Behavior

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Based on the order flow documented in .specify/legacy-integration/order-creation.md,
create characterization tests in @legacy-orderflow/tests/test_characterization.py:

1. test_order_happy_path - Complete flow: register → login → add to cart → create order
2. test_order_response_shape - Verify exact response fields
3. test_order_side_effects - Verify: cart cleared, stock reduced, order ID generated

Each test should assert on EXACT current behavior (don't assume what's "correct").
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From legacy-orderflow directory:

```text
Based on app.py, create characterization tests in tests/test_characterization.py:

1. test_order_happy_path - Complete flow: register → login → add to cart → create order
2. test_order_response_shape - Verify exact response fields  
3. test_order_side_effects - Verify: cart cleared, stock reduced, order ID generated

Assert on EXACT current behavior. Don't assume what's "correct" - document what IS.
```

</details>

### Run Your Tests

```bash
cd legacy-orderflow
pip install pytest
pytest tests/test_characterization.py -v
```

{: .important }
> All tests should **pass against current code**. If a test fails, you misunderstood the behavior -- adjust the test to match reality.

---

## Step 3: Characterize Edge Cases (30 min)

Now the interesting part: what happens at the boundaries?

### Discover Boundary Behavior

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.clarify

Based on .specify/legacy-integration/order-creation.md, clarify edge case behavior 
by analyzing @legacy-orderflow/app.py:

- What happens when order total is exactly $100 (minimum)?
- What happens when order total is exactly $10,000 (maximum)?
- What happens when order total is exactly $500 (discount threshold)?
- What happens when cart contains a product with 0 stock?
- What happens when a user's cart exceeds the max items limit?
- What happens if user tries to order while not logged in?

For each: what does the code ACTUALLY do? (Not what it should do.)
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo:

```text
Analyze ../legacy-orderflow/app.py for edge case behavior:

- Order total exactly $100 (minimum)? Does it pass or fail?
- Order total exactly $10,000 (maximum)? Pass or fail?
- Order total exactly $500 (discount threshold)? Is discount applied?
- Cart with 0-stock product? What error?
- Cart exceeds max items? What happens?
- Order without login? What error code/message?

Document what the code ACTUALLY does, not what seems right.
```

</details>

### Write Edge Case Tests

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Add edge case characterization tests to @legacy-orderflow/tests/test_characterization.py:

1. test_order_minimum_amount - $100 exactly (boundary)
2. test_order_maximum_amount - $10,000 exactly (boundary)
3. test_order_discount_threshold - $500.01 should get 10% off
4. test_order_below_discount - $499.99 no discount
5. test_order_out_of_stock - What error when stock=0?
6. test_order_without_login - What status code? What message?
7. test_order_empty_cart - Can you create an order with empty cart?

Document current behavior, even if it seems wrong.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From legacy-orderflow directory:

```text
Add edge case tests to tests/test_characterization.py:

- test_order_minimum_amount ($100 exactly)
- test_order_maximum_amount ($10,000 exactly)  
- test_order_discount_threshold ($500.01 gets 10% off)
- test_order_below_discount ($499.99 no discount)
- test_order_out_of_stock (what error?)
- test_order_without_login (what status code?)
- test_order_empty_cart (is it allowed?)

Document current behavior even if it seems buggy.
```

</details>

### Run All Tests

```bash
pytest tests/test_characterization.py -v
```

---

## Step 4: Document Surprising Behavior (15 min)

Some things you discover won't match expectations. Document them!

### Update Your Spec with Discoveries

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Based on the characterization tests, update .specify/legacy-integration/order-creation.md 
with a "Surprising Behaviors" section documenting:

1. Any boundary conditions that behaved unexpectedly
2. Any validation that's missing or inconsistent
3. Any side effects that weren't obvious from reading the code
4. Any error messages that are confusing or unhelpful

Format each as: What we expected → What actually happens → Risk level
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Update .specify/legacy-integration/order-creation.md with "Surprising Behaviors":

For each unexpected behavior found in characterization tests:
- What we expected
- What actually happens  
- Risk level (high/medium/low)
- Whether this blocks integration

Examples: boundary off-by-one, missing validation, confusing errors, etc.
```

</details>

### Expected Surprises

You might find things like:

| Expected | Actual | Risk |
|----------|--------|------|
| Order total ≥ $100 | Actually requires > $100 (off-by-one) | Medium |
| Empty cart → error | Silently creates $0 order | High |
| Invalid product → error | Crashes with 500 | High |
| Double-click → idempotent | Creates duplicate order | Critical |

---

## Step 5: Commit Your Work (5 min)

### In the Legacy Repo

```bash
cd legacy-orderflow
git add tests/
git commit -m "test: add characterization tests for order flow"
```

### In Your Greenfield Repo

```bash
cd ~/labs/sdd-greenfield-starter
git add .specify/legacy-integration/
git commit -m "docs: extract order creation spec from legacy system"
```

---

## Success Criteria

- [ ] `legacy-orderflow/tests/test_characterization.py` exists with 10+ tests
- [ ] All characterization tests pass against current code
- [ ] Happy path fully documented with expected values
- [ ] At least 5 edge cases captured
- [ ] `.specify/legacy-integration/order-creation.md` exists in your greenfield repo
- [ ] "Surprising Behaviors" section documents unexpected findings
- [ ] Both repos committed

---

## The Characterization Testing Mindset

| Unit Tests | Characterization Tests |
|------------|----------------------|
| Verify code is correct | Verify code hasn't changed |
| Written from spec | Written from observation |
| Fail = bug in code | Fail = behavior changed |
| "Fix the code" | "Investigate the change" |
| Document intent | Document reality |

**Key insight**: Characterization tests are a safety net for refactoring. They tell you "this is how it works today." Breaking one means you changed behavior -- intentionally or not.

---

## Reflection Questions

1. **Surprising behavior**: What was the most unexpected behavior you discovered through characterization testing? Would you have found it during integration?

2. **Test mindset shift**: How does "testing to document reality" feel different from "testing to verify correctness"? Which requires more humility?

3. **Safety net value**: If you had to refactor `place_order()` tomorrow, how confident would you be with these tests vs. without them?

4. **Part 1 contrast**: In Part 1, your tests came from specs you wrote. Here, tests document code someone else wrote. Which approach gives you more confidence?

---

## Key Takeaways

1. **Test reality, not assumptions** -- The code is the only source of truth
2. **Boundaries reveal bugs** -- Test at min, max, and threshold values
3. **Document surprises** -- Unexpected behavior is integration risk
4. **Tests are executable specs** -- They prove what the system does today

---

## What's Next?

In **Lab 2.2**, you'll extract the business rules hiding in the code -- magic numbers, implicit thresholds, and undocumented policies that the previous developer never wrote down.

**Lab 2.1 answered "What does it do?" Lab 2.2 answers "Why does it do that?"**
