---
title: "Lab 2.4: Notification Integration"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 5
---

# Lab 2.4: Notification Integration — Events, Not Coupling

**Duration**: 60 minutes  
**Day**: 2 (Part 2)  
**Position**: After Lab 2.3  
**Prerequisites**: Gateway from Lab 2.3

---

## Learning Objective

Extract notification logic into a shared service using event-driven patterns.

By the end of this lab, you'll understand: **Event-driven architecture decouples systems. The legacy system doesn't need to know about new services — it just publishes events.**

---

## The Problem

Look at the legacy system's notification code (scattered throughout `app.py`):

```python
# In create_order():
if user.get("email"):
    # TODO: send order confirmation email
    print(f"Would send email to {user['email']}")

# In ship_order():
# TODO: send shipping notification
notifications.append({"user_id": user_id, "message": "Your order shipped!"})
```

**Problems:**
- Notifications embedded in business logic
- No retry on failure
- No templates
- Different approaches in different places
- New system has its own notification code (duplication!)

**Solution:** Extract to a shared notification service that listens for events from both systems.

---

## Target Architecture

```
┌──────────────────┐                    ┌────────────────────┐
│  Legacy Orders   │───order.created───▶│                    │
│  (port 5001)     │───order.shipped───▶│    Event Queue     │
└──────────────────┘                    │                    │
                                        │   (in-memory for   │
┌──────────────────┐                    │    demo, Redis     │
│  Payment Service │───payment.success─▶│    in prod)        │
│  (port 8000)     │                    │                    │
└──────────────────┘                    └─────────┬──────────┘
                                                  │
                                                  ▼
                                        ┌────────────────────┐
                                        │ Notification Svc   │
                                        │                    │
                                        │ - Templates        │
                                        │ - Retry logic      │
                                        │ - Delivery         │
                                        └────────────────────┘
```

---

## Step 1: Design Event Schema (15 min)

Before building, define the contract between publishers and subscribers.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Design the event schema for order notifications:

1. What events exist? 
   - order.created (new order placed)
   - order.paid (payment successful)
   - order.shipped (order dispatched)
   - payment.completed (from Part 1 payment service)

2. What data does each event carry? (Just enough for notifications)

3. How do we ensure events are delivered? (At-least-once semantics)

Analyze notification triggers in both:
- @legacy-orderflow/app.py
- src/ (your Part 1 payment service)

Create .specify/integration/event-schema.md
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo:

```text
Design event schema in .specify/integration/event-schema.md:

Events needed:
- order.created, order.paid, order.shipped (from legacy)
- payment.completed (from Part 1 payment service)

For each event:
- Required fields
- Example payload
- Which system publishes it

Also: envelope format (event_type, event_id, timestamp, source, data).
```

</details>

### Expected Event Schema

```json
{
  "event_type": "order.created",
  "event_id": "evt_abc123def456",
  "timestamp": "2026-01-02T10:30:00Z",
  "source": "legacy-orders",
  "correlation_id": "req_xyz789",
  "data": {
    "order_id": "ord_123",
    "customer_email": "user@example.com",
    "customer_name": "Jane Doe",
    "total": 299.99,
    "items_count": 3
  }
}
```

**Event Types:**

| Event | Source | Trigger | Notification |
|-------|--------|---------|--------------|
| `order.created` | Legacy | New order placed | "Order received" email |
| `order.paid` | Legacy | Payment confirmed | "Payment confirmed" email |
| `order.shipped` | Legacy | Admin ships order | "Your order is on its way" |
| `payment.completed` | Part 1 | Payment processed | "Receipt" email |

---

## Step 2: Implement Simple Event Queue (15 min)

For the demo, we'll use an in-memory queue. In production, this would be Redis Streams, RabbitMQ, or similar.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Create src/events/queue.py:

A simple in-memory event queue with:
- publish(event) - Add event to queue and notify subscribers
- subscribe(callback) - Register a callback for all events
- subscribe_to(event_type, callback) - Register for specific event types
- Thread-safe with Lock
- Events stored in deque for bounded memory

This is demo code - production would use Redis or similar.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create src/events/queue.py:

class EventQueue:
    - _queue: deque (bounded, max 1000 events)
    - _subscribers: list of callbacks for all events
    - _typed_subscribers: dict[event_type] -> list of callbacks
    - publish(event): add to queue, call all matching subscribers
    - subscribe(callback): register for all events
    - subscribe_to(event_type, callback): register for specific type
    - Thread-safe with threading.Lock
```

</details>

### Expected Implementation

```python
# src/events/queue.py
from collections import deque
from threading import Lock
from typing import Callable, Dict, List
import logging

logger = logging.getLogger(__name__)

class EventQueue:
    """Simple in-memory event queue for demo purposes."""
    
    def __init__(self, maxlen: int = 1000):
        self._queue = deque(maxlen=maxlen)
        self._lock = Lock()
        self._subscribers: List[Callable] = []
        self._typed_subscribers: Dict[str, List[Callable]] = {}
    
    def publish(self, event: dict) -> None:
        """Publish an event to all subscribers."""
        with self._lock:
            self._queue.append(event)
            event_type = event.get("event_type", "unknown")
            
            logger.info({"event": "published", "type": event_type, "id": event.get("event_id")})
            
            # Notify all subscribers
            for callback in self._subscribers:
                try:
                    callback(event)
                except Exception as e:
                    logger.error({"event": "subscriber_error", "error": str(e)})
            
            # Notify type-specific subscribers
            for callback in self._typed_subscribers.get(event_type, []):
                try:
                    callback(event)
                except Exception as e:
                    logger.error({"event": "subscriber_error", "error": str(e)})
    
    def subscribe(self, callback: Callable) -> None:
        """Subscribe to all events."""
        self._subscribers.append(callback)
    
    def subscribe_to(self, event_type: str, callback: Callable) -> None:
        """Subscribe to a specific event type."""
        if event_type not in self._typed_subscribers:
            self._typed_subscribers[event_type] = []
        self._typed_subscribers[event_type].append(callback)

# Global instance (in production, this would be Redis connection)
event_queue = EventQueue()
```

---

## Step 3: Build Notification Service (20 min)

A service that listens for events and sends (logs) notifications.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.implement

Create src/notifications/service.py based on .specify/integration/event-schema.md:

1. Subscribe to order.created, order.paid, order.shipped, payment.completed
2. Select appropriate template based on event type
3. Format notification with event data
4. "Send" notification (log for demo, actual email in production)
5. Retry failed notifications up to 3 times with exponential backoff

Templates:
- order.created: "Hi {name}, your order {order_id} has been received!"
- order.shipped: "Great news! Order {order_id} is on its way!"
- payment.completed: "Payment of ${amount} confirmed. Transaction: {transaction_id}"
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create src/notifications/service.py:

TEMPLATES = {
    "order.created": "Hi {customer_name}, order {order_id} received! Total: ${total}",
    "order.shipped": "Order {order_id} is on its way!",
    "payment.completed": "Payment of ${amount} confirmed. Transaction: {transaction_id}",
}

class NotificationService:
    def __init__(self, event_queue):
        # Subscribe to relevant events
        event_queue.subscribe_to("order.created", self.handle_order_created)
        # etc.
    
    def handle_order_created(self, event):
        template = TEMPLATES["order.created"]
        message = template.format(**event["data"])
        self.send(event["data"]["customer_email"], message)
    
    def send(self, to, message, retries=3):
        # For demo: just log
        # In production: send email via SendGrid/SES
        logger.info({"event": "notification_sent", "to": to, "message": message})
```

</details>

### Expected Implementation

```python
# src/notifications/service.py
import logging
import time
from typing import Dict
from ..events.queue import event_queue

logger = logging.getLogger(__name__)

TEMPLATES = {
    "order.created": "Hi {customer_name}, your order #{order_id} has been received! Total: ${total:.2f}",
    "order.paid": "Payment confirmed for order #{order_id}. We're preparing your items!",
    "order.shipped": "Great news! Order #{order_id} is on its way! Track: {tracking_number}",
    "payment.completed": "Payment of ${amount:.2f} confirmed. Transaction ID: {transaction_id}",
}

class NotificationService:
    """Handles notifications for order and payment events."""
    
    def __init__(self):
        # Subscribe to all notification-worthy events
        event_queue.subscribe_to("order.created", self._handle_event)
        event_queue.subscribe_to("order.paid", self._handle_event)
        event_queue.subscribe_to("order.shipped", self._handle_event)
        event_queue.subscribe_to("payment.completed", self._handle_event)
        logger.info("NotificationService started and subscribed to events")
    
    def _handle_event(self, event: dict) -> None:
        """Route event to appropriate handler."""
        event_type = event.get("event_type")
        data = event.get("data", {})
        
        template = TEMPLATES.get(event_type)
        if not template:
            logger.warning({"event": "no_template", "type": event_type})
            return
        
        try:
            message = template.format(**data)
            recipient = data.get("customer_email", "unknown")
            self._send_with_retry(recipient, message, event.get("event_id"))
        except KeyError as e:
            logger.error({"event": "template_error", "missing_key": str(e)})
    
    def _send_with_retry(self, to: str, message: str, event_id: str, max_retries: int = 3):
        """Send notification with exponential backoff retry."""
        for attempt in range(max_retries):
            try:
                self._send(to, message)
                logger.info({
                    "event": "notification_sent",
                    "to": to,
                    "event_id": event_id,
                    "attempt": attempt + 1
                })
                return
            except Exception as e:
                wait_time = 2 ** attempt  # 1, 2, 4 seconds
                logger.warning({
                    "event": "notification_retry",
                    "attempt": attempt + 1,
                    "wait_seconds": wait_time,
                    "error": str(e)
                })
                time.sleep(wait_time)
        
        logger.error({"event": "notification_failed", "to": to, "event_id": event_id})
    
    def _send(self, to: str, message: str) -> None:
        """Send notification. For demo, just logs. Production: email/SMS."""
        # In production: sendgrid.send(to, message) or similar
        print(f"📧 NOTIFICATION to {to}: {message}")

# Initialize service (subscribes to events)
notification_service = NotificationService()
```

---

## Step 4: Wire Up the Publishers (10 min)

Add event publishing to your payment service (legacy will publish via webhook in production).

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Update your Part 1 payment endpoint to publish a payment.completed event:

After successful payment processing:
1. Create event with schema from .specify/integration/event-schema.md
2. Publish to event_queue
3. Include correlation_id from request headers for tracing
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Update src/payment.py (your Part 1 payment endpoint):

After payment succeeds, add:

from .events.queue import event_queue
import uuid

event = {
    "event_type": "payment.completed",
    "event_id": f"evt_{uuid.uuid4().hex[:12]}",
    "timestamp": datetime.utcnow().isoformat() + "Z",
    "source": "payment-service",
    "correlation_id": request.headers.get("X-Correlation-ID", ""),
    "data": {
        "transaction_id": payment_result["id"],
        "amount": payment_request.amount,
        "customer_email": payment_request.email,
    }
}
event_queue.publish(event)
```

</details>

---

## Step 5: Test the Integration (5 min)

Verify events flow from payment to notifications.

```bash
# In one terminal, watch the logs
cd ~/labs/sdd-greenfield-starter
uvicorn src.main:app --port 8000 --log-level debug

# In another terminal, send a payment
curl http://localhost:8000/pay -X POST \
  -H "Content-Type: application/json" \
  -H "X-Correlation-ID: test-123" \
  -d '{"amount": 99.99, "token": "tok_test", "email": "user@example.com"}'
```

**Expected output:**
```
{"event": "published", "type": "payment.completed", "id": "evt_abc123..."}
{"event": "notification_sent", "to": "user@example.com", "event_id": "evt_abc123..."}
📧 NOTIFICATION to user@example.com: Payment of $99.99 confirmed. Transaction ID: pay_xyz...
```

---

## Success Criteria

- [ ] `src/events/queue.py` implements in-memory event queue
- [ ] `src/notifications/service.py` handles 4 event types
- [ ] Payment endpoint publishes `payment.completed` event
- [ ] Notifications logged with correct template formatting
- [ ] Retry logic works (can test by throwing exception in `_send`)
- [ ] `.specify/integration/event-schema.md` documents event contracts

---

## The Event-Driven Advantage

| Before (Coupled) | After (Events) |
|-----------------|----------------|
| Payment code calls notification code | Payment publishes event, doesn't know about notifications |
| Legacy duplicates notification logic | Both systems publish to same queue |
| Adding SMS requires changing payment code | Add SMS subscriber, no payment changes |
| Failed email blocks payment response | Async handling, payment returns immediately |
| Testing requires mocking emails | Test events in isolation |

---

## Key Takeaways

1. **Publish, don't call** — Systems publish events; they don't know who's listening
2. **Schema is contract** — Event schema defines the interface between systems
3. **Retry is mandatory** — Notifications fail; have a strategy
4. **Correlation IDs trace everything** — One ID follows request through all systems

---

## What's Next?

In **Lab 2.5**, you'll create a unified reporting endpoint that aggregates data from both legacy and new systems — seeing the whole picture from one API.

**Events move data changes. Reports aggregate data views.**