---
title: Testing in SDD
layout: default
nav_order: 13
permalink: /testing/
---

# Testing in Spec-Driven Development

How specifications drive your test strategy and why "spec-derived tests" are different from "AI-invented tests."

---

## The SDD Testing Philosophy

### Tests Come From Specs, Not Imagination

| Traditional Approach | SDD Approach |
|---------------------|--------------|
| Developer invents test cases | Tests derive from spec scenarios |
| "What might go wrong?" | "What did we specify?" |
| Coverage is a guessing game | Coverage maps to requirements |
| Tests validate assumptions | Tests validate contracts |

**Key insight**: Your Given/When/Then scenarios ARE your test definitions. The AI doesn't guess what to test—it implements exactly what you specified.

---

## The TDD Cycle in SDD

```
    ┌──────────────────────────────────────────────────────────────┐
    │                    Spec-Driven TDD                           │
    │                                                              │
    │   SPEC                                                       │
    │   ────                                                       │
    │   Scenario 2: Double-Click Protection                        │
    │   Given: Payment processed with key "order-123"              │
    │   When: Same request submitted again                         │
    │   Then: Original response returned (no duplicate)            │
    │                          │                                   │
    │                          ▼                                   │
    │   RED ─────────────────────────────────────────────────────  │
    │   Write test from scenario (fails - no implementation)       │
    │                          │                                   │
    │                          ▼                                   │
    │   GREEN ───────────────────────────────────────────────────  │
    │   Implement minimum code to pass test                        │
    │                          │                                   │
    │                          ▼                                   │
    │   REFACTOR ────────────────────────────────────────────────  │
    │   Clean up while keeping tests green                         │
    │                          │                                   │
    │                          ▼                                   │
    │   COMMIT ──────────────────────────────────────────────────  │
    │   Lock in progress with traceable commit                     │
    │                                                              │
    └──────────────────────────────────────────────────────────────┘
```

---

## From Scenario to Test

### Spec Scenario

```markdown
### Scenario 2: Double-Click Protection (P1)
- **Given** a payment was just processed with idempotency key `order-123`
- **When** the same request is submitted again
- **Then** the original response is returned (no duplicate charge)
- **And** no additional audit entry is created
```

### Generated Test

```python
"""Test: Scenario 2 - Double-Click Protection

Implements: specs/001-payment/spec.md#Scenario-2
Requirement: FR-002 (idempotent payment processing)
"""

import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)


class TestDoubleClickProtection:
    """Scenario 2: Double-Click Protection (P1)
    
    Verifies that duplicate payment requests with the same 
    idempotency_key return the original response without 
    creating a duplicate charge.
    """
    
    def test_duplicate_request_returns_original_response(self):
        """When same request submitted again, original response returned."""
        # Given: First payment processed
        first_response = client.post("/pay", json={
            "amount": 50.00,
            "token": "tok_valid",
            "idempotency_key": "order-123"
        })
        assert first_response.status_code == 200
        first_transaction_id = first_response.json()["transaction_id"]
        
        # When: Same request submitted again
        second_response = client.post("/pay", json={
            "amount": 50.00,
            "token": "tok_valid",
            "idempotency_key": "order-123"
        })
        
        # Then: Original response returned (no duplicate charge)
        assert second_response.status_code == 200
        assert second_response.json()["transaction_id"] == first_transaction_id
    
    def test_no_additional_audit_entry_on_duplicate(self, audit_log_spy):
        """No additional audit entry created for duplicate request."""
        # Given: First payment processed
        client.post("/pay", json={
            "amount": 50.00,
            "token": "tok_valid",
            "idempotency_key": "order-456"
        })
        initial_log_count = audit_log_spy.call_count
        
        # When: Duplicate request
        client.post("/pay", json={
            "amount": 50.00,
            "token": "tok_valid",
            "idempotency_key": "order-456"
        })
        
        # Then: No additional audit entry
        assert audit_log_spy.call_count == initial_log_count
```

### What to Notice

1. **Docstring links to spec**: Easy to trace test back to requirement
2. **Test name describes behavior**: `test_duplicate_request_returns_original_response`
3. **Given/When/Then structure**: Comments mirror the spec
4. **Two tests for compound Then**: "original response" and "no audit entry" are separate

---

## Test Organization

### Directory Structure

```
tests/
├── conftest.py              # Shared fixtures
├── test_payment.py          # Payment feature tests
│   ├── TestSuccessfulPayment    # Scenario 1
│   ├── TestDoubleClickProtection # Scenario 2
│   ├── TestInvalidToken         # Scenario 3
│   └── TestAmountValidation     # Scenario 4
├── test_order.py            # Order feature tests
└── test_integration.py      # Cross-feature tests
```

### Naming Convention

| Spec Element | Test Name |
|--------------|-----------|
| Scenario 1: Successful Payment | `class TestSuccessfulPayment` |
| Given valid token | `def test_valid_token_accepted` |
| When payment submitted | (in test body) |
| Then payment succeeds | `assert response.status_code == 200` |

### Fixtures Map to Given Clauses

```python
# conftest.py

@pytest.fixture
def valid_payment_token():
    """Given: Customer with valid payment token."""
    return "tok_valid_visa"


@pytest.fixture
def processed_payment(client, valid_payment_token):
    """Given: Payment was just processed."""
    response = client.post("/pay", json={
        "amount": 50.00,
        "token": valid_payment_token,
        "idempotency_key": "order-fixture-123"
    })
    return response.json()


@pytest.fixture
def order_in_pending_state(client):
    """Given: Order in pending status."""
    response = client.post("/orders", json={
        "items": [{"sku": "ITEM-1", "qty": 2}]
    })
    return response.json()["order_id"]
```

---

## Coverage That Matters

### Scenario-Based Coverage

| Scenario | Tests | Coverage |
|----------|-------|----------|
| 1: Successful Payment | 2 tests | ✓ Happy path |
| 2: Double-Click | 2 tests | ✓ Idempotency |
| 3: Invalid Token | 1 test | ✓ Error handling |
| 4: Amount Validation | 3 tests | ✓ Edge cases |
| **Total** | **8 tests** | **100% scenarios** |

### Why 80% Code Coverage Works

When tests derive from specs:
- Each requirement has tests
- Edge cases are documented and tested
- The 20% uncovered is usually infrastructure (logging, config)

**Anti-pattern**: Chasing 100% coverage by testing implementation details instead of behavior.

---

## Test Types in SDD

### Unit Tests (from Scenarios)

Test individual behaviors in isolation:

```python
def test_amount_validation_rejects_negative():
    """FR-003: Amounts must be positive."""
    with pytest.raises(ValidationError):
        PaymentRequest(amount=-50.00, token="tok", idempotency_key="key")
```

### Integration Tests (from Workflows)

Test features working together:

```python
def test_checkout_flow(client):
    """Integration: Order → Payment → Confirmation.
    
    Covers the demo day scenario from Lab 1.5.
    """
    # Create order
    order = client.post("/orders", json={"items": [...]})
    order_id = order.json()["order_id"]
    
    # Process payment
    payment = client.post("/pay", json={"order_id": order_id, ...})
    transaction_id = payment.json()["transaction_id"]
    
    # Link payment to order
    confirm = client.patch(f"/orders/{order_id}/pay", json={
        "transaction_id": transaction_id
    })
    
    # Verify order state
    order_status = client.get(f"/orders/{order_id}")
    assert order_status.json()["status"] == "paid"
```

### Characterization Tests (Part 2)

Document existing behavior, not desired behavior:

```python
def test_order_with_negative_quantity_is_accepted():
    """CHARACTERIZATION: Legacy system accepts negative quantities.
    
    This documents CURRENT behavior, not correct behavior.
    Bug? Maybe. But changing it might break production.
    
    See: .specify/legacy-integration/order-creation.md#surprising-behaviors
    """
    response = client.post("/create_order", json={
        "item_id": 1,
        "quantity": -5  # This should probably fail...
    })
    # But it doesn't!
    assert response.status_code == 200
```

---

## Test Prompts for AI

### Generate Tests from Scenario

```text
Write pytest tests for Scenario 2 (Double-Click Protection) 
from specs/001-payment/spec.md.

Requirements:
- Use pytest with TestClient
- Include docstring linking to spec
- Follow Given/When/Then structure in comments
- Test both assertions in the Then clause separately
```

### Test Error Scenarios

```text
Generate tests for all error scenarios in specs/001-payment/spec.md.

For each error case:
- Test the exact HTTP status code
- Test the exact error response body
- Test that no side effects occur (no payment processed, no audit log)
```

### Test State Machine

```text
Generate tests for the order state machine from specs/002-order/spec.md.

Test:
- All valid transitions
- All invalid transitions (should fail with 422)
- The error message for invalid transitions
```

### Coverage Analysis

```text
Compare tests in tests/test_payment.py against scenarios 
in specs/001-payment/spec.md.

List:
- Scenarios with tests ✓
- Scenarios missing tests ✗
- Tests without corresponding scenarios ⚠️
```

---

## Common Testing Mistakes

### Mistake 1: Testing Implementation, Not Behavior

```python
# Bad: Tests internal implementation
def test_redis_cache_used():
    payment_service._cache.set.assert_called_once()

# Good: Tests specified behavior
def test_duplicate_returns_same_response():
    first = process_payment(key="123")
    second = process_payment(key="123")
    assert first == second
```

### Mistake 2: Inventing Scenarios

```python
# Bad: Made-up scenario not in spec
def test_payment_with_emoji_in_token():
    response = pay(token="💳")  # Why are we testing this?

# Good: Test from spec scenario
def test_invalid_token_returns_helpful_error():
    """Scenario 3 from spec.md"""
    response = pay(token="tok_invalid")
    assert response.json()["error"] == "invalid_token"
```

### Mistake 3: Mocking Everything

```python
# Bad: So much mocking, what are we even testing?
@mock.patch("app.redis")
@mock.patch("app.gateway")
@mock.patch("app.logger")
def test_payment(mock_logger, mock_gateway, mock_redis):
    ...

# Good: Use real dependencies, mock only external services
def test_payment(redis_container, mock_payment_gateway):
    """Integration test with real Redis, mocked gateway."""
    ...
```

### Mistake 4: No Traceability

```python
# Bad: Can't trace to requirements
def test_1():
    assert foo() == "bar"

# Good: Clear traceability
def test_idempotency_returns_cached_response():
    """FR-002: Duplicate submissions return original response.
    
    Scenario: Double-Click Protection
    Source: specs/001-payment/spec.md
    """
    ...
```

---

## Testing Checklist

Before considering a feature "tested":

- [ ] Every spec scenario has at least one test
- [ ] Every FR-xxx requirement is exercised
- [ ] Error scenarios return correct status codes and messages
- [ ] State machine transitions are tested (valid and invalid)
- [ ] Integration tests cover the demo flow
- [ ] Tests have docstrings linking to specs
- [ ] Coverage is ≥ 80%
- [ ] No tests without corresponding spec scenarios

---

## Quick Reference: pytest Commands

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest tests/test_payment.py

# Run specific test class
pytest tests/test_payment.py::TestDoubleClickProtection

# Run specific test
pytest tests/test_payment.py::TestDoubleClickProtection::test_duplicate_returns_original

# Run with coverage
pytest --cov=src --cov-report=html

# Fail if coverage below threshold
pytest --cov=src --cov-fail-under=80

# Run tests matching pattern
pytest -k "idempotency"

# Show print statements
pytest -s

# Stop on first failure
pytest -x
```

---

**Remember**: Tests validate contracts, not code. The spec is the contract.
