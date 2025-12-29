---
title: "Lab 1.5: Integration"
layout: default
parent: Labs
nav_order: 6
---
# Lab 1.5: Integrate Payment + Order -- Thursday Build Day

**Duration**: 150 minutes  
**Day**: 2 (Afternoon)  
**Prerequisites**: Completed Lab 1.4 with order specification

---

## Learning Objective

Build the order service, wire it to payments, and prove the full checkout flow works. By end of day Thursday, you'll have a complete demo: create order - pay - see order history.

---

## Where We Are in the Week

```
Monday:      [DONE] Payment spec + plan
Tuesday:     [DONE] Payment implementation
Wednesday:   [DONE] Order spec (Lab 1.4)
Thursday:    [HERE] Build + Integrate
Friday:      Demo day
```

**Thursday goal**: Full checkout flow working, tests passing, demo-ready.

---

## Starting Point

- Working payment service from Lab 1.3
- Complete order specification from Lab 1.4
- Integration points documented

---

## Step 1: Generate Order Tasks (15 min)

Run the tasks command for the order spec:

```bash
/speckit.tasks
```

> "Based on specs/002-order/spec.md and plan.md, create a tasks.md with implementation tasks for the order service."

Your `specs/002-order/tasks.md` should include:

1. **Database setup tasks**: Models, migrations
2. **State machine tasks**: Status enum, transition logic
3. **Endpoint tasks**: Create, update status, get history
4. **Audit tasks**: Audit entry creation
5. **Integration tasks**: Payment callback handling
6. **Test tasks**: Unit and integration tests

---

## Step 2: Implement Order Models (20 min)

Update `src/app/models.py` with order models:

```python
from enum import Enum
from typing import List, Optional
from datetime import datetime
from pydantic import BaseModel, Field


class OrderStatus(str, Enum):
    """Order lifecycle states per spec."""
    CREATED = "created"
    PAID = "paid"
    FULFILLED = "fulfilled"
    CANCELLED = "cancelled"
    REFUNDED = "refunded"
    ARCHIVED = "archived"


class OrderItem(BaseModel):
    """Line item in an order."""
    product_id: str
    quantity: int = Field(..., gt=0)
    unit_price: int = Field(..., gt=0, description="Price in cents")
    
    @property
    def total_price(self) -> int:
        return self.quantity * self.unit_price


class CreateOrderRequest(BaseModel):
    """Request to create a new order."""
    items: List[OrderItem] = Field(..., min_length=1)
    idempotency_key: str = Field(..., min_length=1, max_length=255)


class OrderResponse(BaseModel):
    """Order response with summary information."""
    id: str
    status: OrderStatus
    total_amount: int
    currency: str = "usd"
    items: List[OrderItem]
    payment_transaction_id: Optional[str] = None
    created_at: datetime
    updated_at: datetime


class MarkOrderPaidRequest(BaseModel):
    """Request to mark order as paid."""
    transaction_id: str = Field(..., description="Payment transaction ID")


class OrderAuditEntry(BaseModel):
    """Immutable audit record per governance constraints."""
    order_id: str
    actor_id: str  # Who
    timestamp: datetime  # When
    event_type: str
    previous_state: Optional[dict] = None  # What (before)
    new_state: dict  # What (after)
    trace_id: Optional[str] = None
```

---

## Step 3: Implement State Machine (15 min)

Create `src/app/state_machine.py`:

```python
"""
Order State Machine

Enforces valid transitions per specs/002-order/spec.md governance.
"""

from typing import Set
from src.app.models import OrderStatus


VALID_TRANSITIONS: dict[OrderStatus, Set[OrderStatus]] = {
    OrderStatus.CREATED: {OrderStatus.PAID, OrderStatus.CANCELLED},
    OrderStatus.PAID: {OrderStatus.FULFILLED, OrderStatus.REFUNDED},
    OrderStatus.FULFILLED: {OrderStatus.ARCHIVED},
    OrderStatus.CANCELLED: set(),  # terminal state
    OrderStatus.REFUNDED: set(),   # terminal state
    OrderStatus.ARCHIVED: set(),   # terminal state
}


class InvalidTransitionError(Exception):
    """Raised when an invalid state transition is attempted."""
    
    def __init__(self, current: OrderStatus, target: OrderStatus):
        self.current = current
        self.target = target
        super().__init__(
            f"Cannot transition from '{current.value}' to '{target.value}'"
        )


def can_transition(current: OrderStatus, target: OrderStatus) -> bool:
    """Check if a state transition is valid."""
    return target in VALID_TRANSITIONS.get(current, set())


def validate_transition(current: OrderStatus, target: OrderStatus) -> None:
    """Validate transition, raise if invalid."""
    if not can_transition(current, target):
        raise InvalidTransitionError(current, target)
```

---

## Step 4: Implement Order Endpoints (40 min)

Create `src/app/order.py`:

```python
"""
Order Management Endpoints

Implements order lifecycle per specs/002-order/spec.md.
"""

import json
import logging
import uuid
from datetime import datetime
from typing import List, Optional

import redis.asyncio as redis
from fastapi import APIRouter, HTTPException, Request, Query

from src.app.config import settings
from src.app.models import (
    CreateOrderRequest,
    MarkOrderPaidRequest,
    OrderResponse,
    OrderStatus,
    OrderAuditEntry,
)
from src.app.state_machine import validate_transition, InvalidTransitionError

router = APIRouter(prefix="/orders", tags=["orders"])
logger = logging.getLogger(__name__)

# In-memory storage for lab purposes (use database in production)
orders: dict[str, dict] = {}
audit_entries: List[dict] = []
_redis_client: Optional[redis.Redis] = None


async def get_redis() -> redis.Redis:
    """Get or create Redis client for idempotency."""
    global _redis_client
    if _redis_client is None:
        _redis_client = redis.from_url(settings.redis_url, decode_responses=True)
    return _redis_client


def generate_order_id() -> str:
    """Generate unique order ID per FR-001."""
    return f"ord_{uuid.uuid4().hex[:12]}"


async def create_audit_entry(
    order_id: str,
    actor_id: str,
    event_type: str,
    previous_state: Optional[dict],
    new_state: dict,
    trace_id: Optional[str] = None,
) -> OrderAuditEntry:
    """Create immutable audit entry per governance constraints."""
    entry = OrderAuditEntry(
        order_id=order_id,
        actor_id=actor_id,
        timestamp=datetime.utcnow(),
        event_type=event_type,
        previous_state=previous_state,
        new_state=new_state,
        trace_id=trace_id,
    )
    audit_entries.append(entry.model_dump())
    
    logger.info(
        json.dumps({
            "event": "audit_entry_created",
            "order_id": order_id,
            "event_type": event_type,
            "actor_id": actor_id,
            "timestamp": entry.timestamp.isoformat(),
        })
    )
    
    return entry


@router.post("", response_model=OrderResponse)
async def create_order(request: CreateOrderRequest, http_request: Request):
    """
    Create a new order.
    
    Acceptance Scenario 1: Valid items - order created with status "created".
    Acceptance Scenario 2: Same idempotency key - original order returned.
    """
    trace_id = http_request.headers.get("X-Trace-ID", str(uuid.uuid4()))
    actor_id = http_request.headers.get("X-User-ID", "anonymous")
    
    redis_client = await get_redis()
    
    # Check idempotency
    cache_key = f"order_idempotency:{request.idempotency_key}"
    cached_order_id = await redis_client.get(cache_key)
    
    if cached_order_id and cached_order_id in orders:
        logger.info(json.dumps({
            "event": "duplicate_order_request",
            "order_id": cached_order_id,
            "idempotency_key": request.idempotency_key,
        }))
        order = orders[cached_order_id]
        return OrderResponse(**order)
    
    # Create order
    order_id = generate_order_id()
    now = datetime.utcnow()
    
    total_amount = sum(item.quantity * item.unit_price for item in request.items)
    
    order = {
        "id": order_id,
        "status": OrderStatus.CREATED,
        "total_amount": total_amount,
        "currency": "usd",
        "items": [item.model_dump() for item in request.items],
        "payment_transaction_id": None,
        "created_at": now,
        "updated_at": now,
        "version": 1,
    }
    
    orders[order_id] = order
    
    # Cache for idempotency
    await redis_client.setex(cache_key, 60, order_id)
    
    # Create audit entry (FR-004)
    await create_audit_entry(
        order_id=order_id,
        actor_id=actor_id,
        event_type="order_created",
        previous_state=None,
        new_state={"status": order["status"].value, "total_amount": total_amount},
        trace_id=trace_id,
    )
    
    return OrderResponse(**order)


@router.patch("/{order_id}/pay", response_model=OrderResponse)
async def mark_order_paid(
    order_id: str,
    request: MarkOrderPaidRequest,
    http_request: Request,
):
    """
    Mark order as paid after successful payment.
    
    Acceptance Scenario 1: created - paid with transaction_id.
    Acceptance Scenario 2: Already paid - error.
    """
    trace_id = http_request.headers.get("X-Trace-ID", str(uuid.uuid4()))
    actor_id = http_request.headers.get("X-User-ID", "system")
    
    if order_id not in orders:
        raise HTTPException(status_code=404, detail={"error_code": "order_not_found"})
    
    order = orders[order_id]
    current_status = order["status"]
    
    # Validate transition (FR-005)
    try:
        validate_transition(current_status, OrderStatus.PAID)
    except InvalidTransitionError as e:
        # Log attempted invalid transition
        await create_audit_entry(
            order_id=order_id,
            actor_id=actor_id,
            event_type="invalid_transition_attempted",
            previous_state={"status": current_status.value},
            new_state={"attempted_status": "paid"},
            trace_id=trace_id,
        )
        
        if current_status == OrderStatus.PAID:
            raise HTTPException(
                status_code=400,
                detail={"error_code": "order_already_paid"}
            )
        raise HTTPException(
            status_code=400,
            detail={
                "error_code": "invalid_transition",
                "current_status": current_status.value,
                "requested_status": "paid",
            }
        )
    
    # Update order
    previous_state = {"status": current_status.value}
    order["status"] = OrderStatus.PAID
    order["payment_transaction_id"] = request.transaction_id
    order["updated_at"] = datetime.utcnow()
    order["version"] += 1
    
    # Create audit entry
    await create_audit_entry(
        order_id=order_id,
        actor_id=actor_id,
        event_type="order_paid",
        previous_state=previous_state,
        new_state={
            "status": "paid",
            "payment_transaction_id": request.transaction_id,
        },
        trace_id=trace_id,
    )
    
    return OrderResponse(**order)


@router.get("", response_model=dict)
async def get_order_history(
    http_request: Request,
    limit: int = Query(default=20, ge=1, le=100),
    offset: int = Query(default=0, ge=0),
):
    """
    Get user's order history.
    
    Acceptance Scenario 1: Returns paginated list sorted by created_at DESC.
    """
    user_id = http_request.headers.get("X-User-ID", "anonymous")
    
    # Filter orders by user (in production, this would be a DB query)
    # For lab purposes, return all orders
    user_orders = list(orders.values())
    user_orders.sort(key=lambda x: x["created_at"], reverse=True)
    
    paginated = user_orders[offset:offset + limit]
    
    return {
        "orders": [OrderResponse(**o).model_dump() for o in paginated],
        "total_count": len(user_orders),
        "limit": limit,
        "offset": offset,
    }


@router.get("/{order_id}", response_model=OrderResponse)
async def get_order(order_id: str):
    """Get order by ID."""
    if order_id not in orders:
        raise HTTPException(status_code=404, detail={"error_code": "order_not_found"})
    
    return OrderResponse(**orders[order_id])
```

---

## Step 5: Register Order Router (5 min)

Update `src/app/main.py`:

```python
from src.app.order import router as order_router

# Add after payment router
app.include_router(order_router)
```

---

## Step 6: Write Order Unit Tests (25 min)

Create `tests/test_order.py`:

```python
"""
Order Endpoint Tests

Tests for order lifecycle per specs/002-order/spec.md.
"""

import pytest
from unittest.mock import AsyncMock, patch
from httpx import AsyncClient


class TestCreateOrder:
    """POST /orders tests."""

    @pytest.mark.asyncio
    async def test_create_order_success(self, client: AsyncClient):
        """Acceptance Scenario 1: Valid items - order created."""
        mock_redis = AsyncMock()
        mock_redis.get.return_value = None
        mock_redis.setex.return_value = True
        
        with patch("src.app.order.get_redis", return_value=mock_redis):
            response = await client.post(
                "/orders",
                json={
                    "items": [
                        {"product_id": "prod_123", "quantity": 2, "unit_price": 2500}
                    ],
                    "idempotency_key": "test-order-001",
                },
            )
        
        assert response.status_code == 200
        data = response.json()
        assert data["status"] == "created"
        assert data["total_amount"] == 5000
        assert data["id"].startswith("ord_")

    @pytest.mark.asyncio
    async def test_create_order_empty_items_rejected(self, client: AsyncClient):
        """Empty items list should be rejected."""
        response = await client.post(
            "/orders",
            json={
                "items": [],
                "idempotency_key": "test-order-002",
            },
        )
        
        assert response.status_code == 422


class TestMarkOrderPaid:
    """PATCH /orders/{id}/pay tests."""

    @pytest.mark.asyncio
    async def test_mark_created_order_paid(self, client: AsyncClient):
        """Acceptance Scenario 1: created - paid."""
        # First create an order
        mock_redis = AsyncMock()
        mock_redis.get.return_value = None
        mock_redis.setex.return_value = True
        
        with patch("src.app.order.get_redis", return_value=mock_redis):
            create_response = await client.post(
                "/orders",
                json={
                    "items": [{"product_id": "prod_123", "quantity": 1, "unit_price": 5000}],
                    "idempotency_key": "test-pay-001",
                },
            )
        
        order_id = create_response.json()["id"]
        
        # Mark as paid
        with patch("src.app.order.get_redis", return_value=mock_redis):
            pay_response = await client.patch(
                f"/orders/{order_id}/pay",
                json={"transaction_id": "txn_test_123"},
            )
        
        assert pay_response.status_code == 200
        data = pay_response.json()
        assert data["status"] == "paid"
        assert data["payment_transaction_id"] == "txn_test_123"


class TestOrderHistory:
    """GET /orders tests."""

    @pytest.mark.asyncio
    async def test_get_empty_history(self, client: AsyncClient):
        """Empty order history returns empty list."""
        # Clear orders for clean test
        from src.app.order import orders
        orders.clear()
        
        response = await client.get("/orders")
        
        assert response.status_code == 200
        data = response.json()
        assert data["orders"] == []
        assert data["total_count"] == 0
```

---

## Step 7: Write Integration Tests (30 min)

Create `tests/test_integration.py`:

```python
"""
End-to-End Integration Tests

Tests for complete checkout flow: order - payment - order paid.
"""

import pytest
from unittest.mock import AsyncMock, patch, MagicMock
from httpx import AsyncClient


class TestCheckoutFlow:
    """E2E checkout flow tests."""

    @pytest.mark.asyncio
    async def test_complete_checkout_flow(self, client: AsyncClient):
        """
        E2E Test: Create order - Process payment - Order marked paid.
        
        This tests the integration between order and payment services.
        """
        mock_redis = AsyncMock()
        mock_redis.get.return_value = None
        mock_redis.setex.return_value = True
        
        # Step 1: Create order
        with patch("src.app.order.get_redis", return_value=mock_redis):
            order_response = await client.post(
                "/orders",
                json={
                    "items": [
                        {"product_id": "prod_widget", "quantity": 2, "unit_price": 2500}
                    ],
                    "idempotency_key": "checkout-e2e-001",
                },
            )
        
        assert order_response.status_code == 200
        order_data = order_response.json()
        order_id = order_data["id"]
        assert order_data["status"] == "created"
        assert order_data["total_amount"] == 5000
        
        # Step 2: Process payment (mock gateway response)
        mock_gateway_response = MagicMock()
        mock_gateway_response.status_code = 200
        mock_gateway_response.json.return_value = {
            "id": "txn_checkout_e2e_001",
            "status": "succeeded",
            "amount": 5000,
            "currency": "usd",
            "created_at": "2025-01-01T12:00:00",
            "idempotency_key": "payment-e2e-001",
        }
        
        with patch("src.app.payment.get_redis", return_value=mock_redis):
            with patch("httpx.AsyncClient.post", return_value=mock_gateway_response):
                payment_response = await client.post(
                    "/pay",
                    json={
                        "token": "tok_e2e_test",
                        "amount": 5000,
                        "currency": "usd",
                        "idempotency_key": "payment-e2e-001",
                    },
                )
        
        assert payment_response.status_code == 200
        payment_data = payment_response.json()
        transaction_id = payment_data["transaction_id"]
        assert payment_data["status"] == "succeeded"
        
        # Step 3: Mark order as paid
        with patch("src.app.order.get_redis", return_value=mock_redis):
            mark_paid_response = await client.patch(
                f"/orders/{order_id}/pay",
                json={"transaction_id": transaction_id},
            )
        
        assert mark_paid_response.status_code == 200
        final_order = mark_paid_response.json()
        assert final_order["status"] == "paid"
        assert final_order["payment_transaction_id"] == transaction_id
        
        # Step 4: Verify order state via GET
        get_response = await client.get(f"/orders/{order_id}")
        assert get_response.status_code == 200
        assert get_response.json()["status"] == "paid"

    @pytest.mark.asyncio
    async def test_cannot_pay_cancelled_order(self, client: AsyncClient):
        """
        Edge Case: Cannot pay a cancelled order.
        
        Tests state machine enforcement.
        """
        mock_redis = AsyncMock()
        mock_redis.get.return_value = None
        mock_redis.setex.return_value = True
        
        # Create order
        with patch("src.app.order.get_redis", return_value=mock_redis):
            order_response = await client.post(
                "/orders",
                json={
                    "items": [{"product_id": "prod_123", "quantity": 1, "unit_price": 1000}],
                    "idempotency_key": "cancelled-order-001",
                },
            )
        
        order_id = order_response.json()["id"]
        
        # Manually cancel order (simulating cancellation)
        from src.app.order import orders
        from src.app.models import OrderStatus
        orders[order_id]["status"] = OrderStatus.CANCELLED
        
        # Attempt to pay cancelled order
        with patch("src.app.order.get_redis", return_value=mock_redis):
            pay_response = await client.patch(
                f"/orders/{order_id}/pay",
                json={"transaction_id": "txn_should_fail"},
            )
        
        assert pay_response.status_code == 400
        assert pay_response.json()["detail"]["error_code"] == "invalid_transition"

    @pytest.mark.asyncio
    async def test_double_payment_rejected(self, client: AsyncClient):
        """
        Acceptance Scenario 2: Already paid - error.
        """
        mock_redis = AsyncMock()
        mock_redis.get.return_value = None
        mock_redis.setex.return_value = True
        
        # Create and pay order
        with patch("src.app.order.get_redis", return_value=mock_redis):
            order_response = await client.post(
                "/orders",
                json={
                    "items": [{"product_id": "prod_123", "quantity": 1, "unit_price": 1000}],
                    "idempotency_key": "double-pay-001",
                },
            )
            order_id = order_response.json()["id"]
            
            # First payment
            await client.patch(
                f"/orders/{order_id}/pay",
                json={"transaction_id": "txn_first"},
            )
            
            # Second payment attempt
            second_pay = await client.patch(
                f"/orders/{order_id}/pay",
                json={"transaction_id": "txn_second"},
            )
        
        assert second_pay.status_code == 400
        assert second_pay.json()["detail"]["error_code"] == "order_already_paid"
```

---

## Step 8: Run Coverage Check (10 min)

```bash
pytest --cov=src --cov-report=term-missing --cov-fail-under=80
```

**Target**: 80%+ coverage

If coverage is below 80%:
1. Add tests for edge cases (order not found, invalid items)
2. Add tests for audit trail entries
3. Add tests for pagination

---

## Step 9: Run Security Scan (5 min)

```bash
semgrep --config p/security-audit src/
bandit -r src/
```

**Pass Criteria**: 0 CRITICAL + 0 HIGH

---

## Step 10: Commit Your Work (2 min)

```bash
git add .
git commit -m "feat: order management with payment integration"
```

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

```bash
python validate_lab.py --lab 1.5 --repo . --coverage-check
```

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
