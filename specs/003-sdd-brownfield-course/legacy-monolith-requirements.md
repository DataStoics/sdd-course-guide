# Legacy Monolith Requirements

Technical requirements for the legacy codebase used in Part 2.

---

## Overview

The legacy monolith must be **realistic enough to be frustrating** but **simple enough for a 2-day workshop**.

---

## Size and Complexity

| Metric | Target | Rationale |
|--------|--------|-----------|
| Lines of code | 1,500-2,500 | Explorable in 45 min, not overwhelming |
| Files | 1-3 | Single entry point simplifies exploration |
| Endpoints | 15-20 | Enough complexity, still mappable |
| Business rules | 10-15 | Extractable in one session |

---

## Required Anti-Patterns (Teaching Points)

### Security Issues

| Issue | Implementation | Learning |
|-------|----------------|----------|
| Weak password hashing | MD5 instead of bcrypt | Security spec extraction |
| Missing authentication | Admin endpoints unprotected | Access control specification |
| Hardcoded secrets | `SECRET = "super_secret_key_123"` | Security audit finding |
| Backdoor access | `X-Admin-Override` header | Compliance concern |

### Code Quality Issues

| Issue | Implementation | Learning |
|-------|----------------|----------|
| Magic numbers | `MAX_ORDER = 10000`, inline `5000` | Business rule extraction |
| Global mutable state | `users = {}, orders = {}` | State management discussion |
| No separation of concerns | All logic in one file | Modular refactoring |
| Cryptic naming | `proc_tm`, `stat_cd` | Code archaeology |
| Missing error handling | `continue # silently skip` | Error specification |

### Testing Issues

| Issue | Implementation | Learning |
|-------|----------------|----------|
| Zero tests | No test files | Characterization testing |
| No test infrastructure | No pytest setup | Setting up test harness |
| Untestable code | Tight coupling to global state | Seam identification |

### Documentation Issues

| Issue | Implementation | Learning |
|-------|----------------|----------|
| No docstrings | Functions undocumented | Reverse documentation |
| Misleading comments | `# don't change this` | Code archaeology |
| Missing README details | Just "it works" | System mapping |
| No API documentation | Endpoints undocumented | Contract extraction |

---

## Required Features (Domain Coverage)

### User Management
- [x] Registration with email/password
- [x] Login with session tokens
- [x] User types (regular, credit)
- [x] Account suspension

### Product Catalog
- [x] Product listing
- [x] Product creation
- [x] Stock tracking

### Shopping Cart
- [x] Add to cart
- [x] Cart persistence per user
- [x] Stock validation

### Order Processing
- [x] Order creation from cart
- [x] Minimum/maximum order amounts
- [x] Discount calculation
- [x] Tax calculation
- [x] Shipping calculation

### Order Lifecycle
- [x] Status: pending → paid → shipped → delivered
- [x] Cancellation with stock restoration
- [x] Refund with time limit
- [x] Credit balance management

### Admin Functions
- [x] User listing
- [x] User suspension
- [x] Sales reporting

---

## Business Rules to Hide in Code

### Pricing Rules
```python
TAX_RATE = 0.08           # 8% tax
SHIPPING = 5.99           # Flat shipping
DISCOUNT_THRESHOLD = 500  # Orders over $500
DISCOUNT_RATE = 0.1       # Get 10% off
```

### Order Constraints
```python
MIN_ORDER = 100           # $100 minimum
MAX_ORDER = 10000         # $10,000 maximum
# Hidden: max 100 items per line (silently truncated)
```

### User Rules
```python
# Credit users have $5,000 limit (inline, not constant)
if user['type'] == 'credit':
    if user['balance'] + total > 5000:
        return error
```

### Refund Policy
```python
# 30-day refund window (inline)
if datetime.now() - created > timedelta(days=30):
    # But secret admin override exists
    if request.headers.get('X-Admin-Override') != SECRET:
        return error
```

### Logging Quirk
```python
def log_action(action, user_id=None, data=None):
    if random.random() > 0.1:  # Only log 90% (compliance issue!)
        audit_log.append(...)
```

---

## Order State Machine

```
         create
            │
            ▼
      ┌──────────┐
      │ pending  │◀────────┐
      └──────────┘         │
            │              │
       pay  │              │ cancel
            ▼              │
      ┌──────────┐         │
      │   paid   │─────────┤
      └──────────┘         │
            │              │
       ship │              │
            ▼              │
      ┌──────────┐         │
      │ shipped  │ (cannot cancel after ship)
      └──────────┘
            │
     deliver│
            ▼
      ┌──────────┐
      │delivered │
      └──────────┘
            │
     refund │ (30-day limit)
            ▼
      ┌──────────┐
      │ refunded │
      └──────────┘
      
Additional states:
      ┌───────────┐
      │ cancelled │
      └───────────┘
```

---

## API Endpoints Required

### Authentication
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/register` | Create user account |
| POST | `/login` | Authenticate, get session |

### Products
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/products` | List all products |
| POST | `/products` | Create product (no auth!) |

### Cart
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/cart/add` | Add item to cart |

### Orders
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/order` | Create order from cart |
| POST | `/order/{id}/pay` | Process payment |
| POST | `/order/{id}/ship` | Mark as shipped |
| POST | `/order/{id}/cancel` | Cancel order |
| POST | `/order/{id}/refund` | Refund order |

### Admin
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/admin/users` | List all users (no auth!) |
| POST | `/admin/suspend/{id}` | Suspend user (no auth!) |

### Reporting
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/report/sales` | Sales summary |

### System
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | Health check |

---

## Technical Stack

### Required
- Python 3.9+
- Flask
- No external database (in-memory)

### Explicitly NOT Using
- SQLAlchemy or any ORM
- Redis or caching
- Celery or task queues
- External services

### Why These Choices

1. **In-memory state**: Simpler setup, no database migrations during labs
2. **Flask**: Minimal framework, all logic visible in routes
3. **No dependencies**: Easy to run anywhere, focus on code not infrastructure

---

## Initial Data

On startup, pre-populate:

```python
products['prod_001'] = {
    'id': 'prod_001',
    'name': 'Widget A',
    'price': 29.99,
    'stock': 100,
    'category': 'widgets'
}
products['prod_002'] = {
    'id': 'prod_002', 
    'name': 'Widget B',
    'price': 49.99,
    'stock': 50,
    'category': 'widgets'
}
products['prod_003'] = {
    'id': 'prod_003',
    'name': 'Gadget X',
    'price': 99.99,
    'stock': 25,
    'category': 'gadgets'
}
```

---

## Validation Checklist

Before using in training:

- [ ] Application starts with `python app.py`
- [ ] Health endpoint returns 200
- [ ] Can register user
- [ ] Can login and get session
- [ ] Can add items to cart
- [ ] Can create order (verify all calculations)
- [ ] Can pay, ship, cancel, refund orders
- [ ] Admin endpoints work (without auth)
- [ ] Sales report calculates correctly
- [ ] At least 10 "intentional problems" are discoverable
- [ ] Code is frustrating but not impossible to understand

---

## Differences from Current `legacy-monolith`

The existing monolith at `repos/legacy-monolith/app.py` is close but needs review:

| Current | Target | Action |
|---------|--------|--------|
| ~400 lines | 1,500-2,500 lines | Expand with more edge cases |
| Good structure | Intentionally messy | Add more anti-patterns |
| Some comments | Misleading/missing comments | Remove helpful ones |
| Working tests | No tests | Remove any test files |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-01-02 | Course Team | Initial requirements |
