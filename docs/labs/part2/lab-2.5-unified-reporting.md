---
title: "Lab 2.5: Unified Reporting"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 6
---

# Lab 2.5: Unified Reporting — One View, Two Systems

**Duration**: 45 minutes  
**Day**: 2 (Part 2)  
**Position**: After Lab 2.4  
**Prerequisites**: Gateway and both systems running

---

## Learning Objective

Create a read model that aggregates data from multiple systems into a unified view.

By the end of this lab, you'll understand: **You don't need to migrate data to unify reporting. Query both, transform once.**

---

## The Business Need

Your PM has a request from leadership:

> "I need ONE dashboard that shows:
> - Total orders (from both systems)
> - Revenue by period  
> - Order status breakdown
> - Where orders are coming from (legacy vs new)
>
> I can't have two separate reports."

**The challenge:** Data lives in two places with different schemas. Leadership needs one view.

---

## The Approach: Query-Time Aggregation

Instead of migrating data (risky) or syncing databases (complex), we'll:

1. Query both systems via their APIs
2. Transform responses to a common schema
3. Aggregate and return unified results

This is a lightweight version of CQRS (Command Query Responsibility Segregation) — separate models for reads and writes.

---

## Step 1: Design the Unified Schema (10 min)

Map fields from both systems to a common format.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Design a unified order schema that works for both legacy and new systems:

Legacy (@legacy-orderflow/app.py):
- order["id"], order["total"], order["status"], order["created"]
- order["user_id"], order["items"]

New (src/ - your Part 1 payment service):
- order.order_id, order.amount, order.order_status, order.created_at
- order.customer_id, order.line_items

Create .specify/reports/unified-schema.md with:
- Common field mapping table
- Transformation rules
- Handling for missing/null values
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create .specify/reports/unified-schema.md:

Map these to common schema:

Legacy:
- id → order_id
- total → order_total
- status → status (may need value mapping)
- created → created_timestamp (parse to ISO8601)

New:
- order_id → order_id
- amount → order_total
- order_status → status
- created_at → created_timestamp

Include: field mappings, value transformations, null handling.
```

</details>

### Expected Mapping

| Unified Field | Legacy Field | New Field | Transform |
|--------------|--------------|-----------|-----------|
| `order_id` | `id` | `order_id` | Prefix with source (leg_/new_) |
| `order_total` | `total` | `amount` | Float, 2 decimals |
| `status` | `status` | `order_status` | Normalize values |
| `created_timestamp` | `created` | `created_at` | ISO8601 UTC |
| `customer_id` | `user_id` | `customer_id` | String |
| `source` | — | — | "legacy" or "new" |

**Status Value Mapping:**

| Legacy Status | New Status | Unified Status |
|--------------|------------|----------------|
| pending | pending | pending |
| paid | completed | paid |
| shipped | shipped | shipped |
| delivered | delivered | delivered |
| cancelled | canceled | cancelled |
| refunded | refunded | refunded |

---

## Step 2: Implement the Aggregation Endpoint (25 min)

Add a reporting endpoint to your gateway that queries both systems.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.implement

Add GET /reports/sales to src/gateway/main.py based on .specify/reports/unified-schema.md:

1. Accept query params: start_date, end_date (optional, default last 30 days)
2. Fetch orders from legacy system (GET http://localhost:5001/admin/orders)
3. Fetch orders from new system (GET http://localhost:8000/orders)
4. Transform both to unified schema
5. Calculate aggregates:
   - total_orders: count
   - total_revenue: sum of order_total
   - by_status: count per status
   - by_source: count per source (legacy/new)
6. Return combined results
7. Handle errors gracefully (if one system down, return partial data)
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Add to src/gateway/main.py:

@app.get("/reports/sales")
async def unified_sales_report(
    start_date: date = None,  # Default: 30 days ago
    end_date: date = None     # Default: today
):
    # Fetch from both systems (parallel)
    legacy_orders = await fetch_legacy_orders(start_date, end_date)
    new_orders = await fetch_new_orders(start_date, end_date)
    
    # Transform to common schema
    all_orders = [
        *transform_legacy_orders(legacy_orders),
        *transform_new_orders(new_orders)
    ]
    
    # Aggregate
    return {
        "period": {"start": start_date, "end": end_date},
        "total_orders": len(all_orders),
        "total_revenue": sum(o["order_total"] for o in all_orders),
        "by_status": aggregate_by_field(all_orders, "status"),
        "by_source": aggregate_by_field(all_orders, "source"),
        "orders": all_orders  # Optional: include raw data
    }
```

</details>

### Expected Implementation

```python
# Add to src/gateway/main.py

from datetime import date, datetime, timedelta
import httpx

@app.get("/reports/sales")
async def unified_sales_report(
    start_date: date = None,
    end_date: date = None
):
    """Aggregate sales data from legacy and new systems."""
    # Default to last 30 days
    end_date = end_date or date.today()
    start_date = start_date or (end_date - timedelta(days=30))
    
    # Fetch from both systems (handle failures gracefully)
    legacy_orders = []
    new_orders = []
    errors = []
    
    async with httpx.AsyncClient(timeout=10.0) as client:
        try:
            resp = await client.get(f"{LEGACY_URL}/admin/orders")
            if resp.status_code == 200:
                legacy_orders = resp.json().get("orders", [])
        except Exception as e:
            errors.append({"source": "legacy", "error": str(e)})
        
        try:
            resp = await client.get(f"{NEW_URL}/orders")
            if resp.status_code == 200:
                new_orders = resp.json().get("orders", [])
        except Exception as e:
            errors.append({"source": "new", "error": str(e)})
    
    # Transform to unified schema
    unified = []
    unified.extend(transform_legacy_orders(legacy_orders, start_date, end_date))
    unified.extend(transform_new_orders(new_orders, start_date, end_date))
    
    # Aggregate
    return {
        "period": {
            "start": start_date.isoformat(),
            "end": end_date.isoformat()
        },
        "summary": {
            "total_orders": len(unified),
            "total_revenue": round(sum(o["order_total"] for o in unified), 2),
            "average_order": round(
                sum(o["order_total"] for o in unified) / len(unified), 2
            ) if unified else 0
        },
        "by_status": aggregate_by_field(unified, "status"),
        "by_source": aggregate_by_field(unified, "source"),
        "errors": errors if errors else None,
        "orders": unified
    }


def transform_legacy_orders(orders: list, start: date, end: date) -> list:
    """Transform legacy orders to unified schema."""
    unified = []
    for order in orders:
        try:
            created = datetime.fromisoformat(order["created"].replace("Z", "+00:00"))
            if not (start <= created.date() <= end):
                continue
            
            unified.append({
                "order_id": f"leg_{order['id']}",
                "order_total": float(order.get("total", 0)),
                "status": normalize_status(order.get("status", "unknown")),
                "created_timestamp": order["created"],
                "customer_id": str(order.get("user_id", "")),
                "source": "legacy"
            })
        except (KeyError, ValueError) as e:
            logger.warning({"event": "transform_error", "source": "legacy", "error": str(e)})
    return unified


def transform_new_orders(orders: list, start: date, end: date) -> list:
    """Transform new system orders to unified schema."""
    unified = []
    for order in orders:
        try:
            created = datetime.fromisoformat(order["created_at"].replace("Z", "+00:00"))
            if not (start <= created.date() <= end):
                continue
            
            unified.append({
                "order_id": f"new_{order['order_id']}",
                "order_total": float(order.get("amount", 0)),
                "status": normalize_status(order.get("order_status", "unknown")),
                "created_timestamp": order["created_at"],
                "customer_id": str(order.get("customer_id", "")),
                "source": "new"
            })
        except (KeyError, ValueError) as e:
            logger.warning({"event": "transform_error", "source": "new", "error": str(e)})
    return unified


STATUS_MAPPING = {
    # Legacy -> Unified
    "pending": "pending",
    "paid": "paid",
    "shipped": "shipped",
    "delivered": "delivered",
    "cancelled": "cancelled",
    "refunded": "refunded",
    # New -> Unified
    "completed": "paid",
    "canceled": "cancelled",
}

def normalize_status(status: str) -> str:
    """Map various status values to unified format."""
    return STATUS_MAPPING.get(status.lower(), status.lower())


def aggregate_by_field(orders: list, field: str) -> dict:
    """Count orders by a given field."""
    result = {}
    for order in orders:
        value = order.get(field, "unknown")
        result[value] = result.get(value, 0) + 1
    return result
```

---

## Step 3: Handle Edge Cases (10 min)

Real systems have messy data. Handle it gracefully.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Update the unified reporting code to handle:

1. **Missing fields**: Use defaults (0 for amounts, "unknown" for status)
2. **Different date formats**: Parse multiple formats, normalize to ISO8601
3. **Duplicate detection**: Orders might appear in both systems during migration
   - Detect by: same customer + same amount + same timestamp (within 1 min)
   - Mark duplicates, don't double-count
4. **One system down**: Return partial data with error indicator
5. **Timeout**: 10 second timeout, return what we have
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Add edge case handling to the reporting endpoint:

1. Missing fields: default values
2. Parse multiple date formats (ISO, timestamp, US format)
3. Detect duplicates: same amount + customer + timestamp within 60s
4. Partial results if one system fails
5. Include "data_quality" section showing issues found
```

</details>

### Data Quality Reporting

```python
def assess_data_quality(orders: list) -> dict:
    """Report on data quality issues found."""
    issues = {
        "missing_customer": 0,
        "missing_amount": 0,
        "unknown_status": 0,
        "potential_duplicates": 0
    }
    
    seen = set()
    for order in orders:
        if not order.get("customer_id"):
            issues["missing_customer"] += 1
        if order.get("order_total", 0) == 0:
            issues["missing_amount"] += 1
        if order.get("status") == "unknown":
            issues["unknown_status"] += 1
        
        # Duplicate detection
        key = f"{order['customer_id']}_{order['order_total']}_{order['created_timestamp'][:16]}"
        if key in seen:
            issues["potential_duplicates"] += 1
        seen.add(key)
    
    return {k: v for k, v in issues.items() if v > 0}  # Only non-zero issues
```

---

## Step 4: Test the Report (5 min)

```bash
# Make sure both systems have some data
# Legacy should have orders from Lab 2.0-2.1 testing

# Query the unified report
curl "http://localhost:8080/reports/sales?start_date=2026-01-01&end_date=2026-01-31" | jq
```

**Expected Response:**

```json
{
  "period": {
    "start": "2026-01-01",
    "end": "2026-01-31"
  },
  "summary": {
    "total_orders": 15,
    "total_revenue": 4523.47,
    "average_order": 301.56
  },
  "by_status": {
    "pending": 3,
    "paid": 8,
    "shipped": 2,
    "delivered": 2
  },
  "by_source": {
    "legacy": 12,
    "new": 3
  },
  "data_quality": {
    "unknown_status": 1
  },
  "orders": [...]
}
```

---

## Success Criteria

- [ ] `GET /reports/sales` returns aggregated data from both systems
- [ ] Data transformed to unified schema correctly
- [ ] Status values normalized across systems
- [ ] Source attribution (legacy/new) included
- [ ] Partial results returned if one system fails
- [ ] Date filtering works correctly
- [ ] `.specify/reports/unified-schema.md` documents the mapping

---

## CQRS Lite: The Pattern

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Gateway (port 8080)                        │
│                                                                     │
│   WRITES (mutate data)              READS (query data)              │
│   ────────────────────              ──────────────────              │
│   POST /orders → Legacy             GET /reports/sales              │
│   POST /pay → New                        │                          │
│                                          ▼                          │
│                               ┌─────────────────────┐               │
│                               │  Aggregation Layer  │               │
│                               │                     │               │
│                               │  - Query both       │               │
│                               │  - Transform        │               │
│                               │  - Aggregate        │               │
│                               └─────────────────────┘               │
│                                    │         │                      │
└────────────────────────────────────┼─────────┼──────────────────────┘
                                     │         │
                              ┌──────┴───┐ ┌───┴──────┐
                              │  Legacy  │ │   New    │
                              │  (5001)  │ │  (8000)  │
                              └──────────┘ └──────────┘
```

**Key insight:** You don't need full CQRS to benefit from read/write separation. A simple aggregation layer gives you unified views without rewriting either system.

---

## Reflection Questions

1. **Schema mapping complexity**: What surprised you about mapping "the same data" from two systems? Why do you think the original developers used different field names?

2. **Graceful degradation value**: The report returns partial data if one system fails. When would partial data be better than no data? When would it be worse?

3. **Duplicate detection challenge**: Orders might exist in both systems during migration. How confident are you in your duplicate detection logic? What edge cases might it miss?

4. **Part 1 contrast**: In Part 1, all your data was in one schema you designed. Here, you're aggregating schemas you don't control. How does that change your specs?

---

## Key Takeaways

1. **Query, don't migrate** — Unify at read time, not by moving data
2. **Schema mapping is the contract** — Document transformations explicitly
3. **Graceful degradation** — Partial data is better than no data
4. **Source attribution matters** — Always know where data came from

---

## What's Next?

In **Lab 2.6**, you'll create a migration plan — the operational document that guides a safe transition from legacy to new system.

**You can see unified data. Now plan how to make everything new.**
