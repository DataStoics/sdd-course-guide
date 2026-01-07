---
title: "Lab 2.3: The Strangler Facade"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 4
---

# Lab 2.3: The Strangler Facade
{: .fs-9 }

Route, Don't Rewrite
{: .fs-6 .fw-300 }

[~90 min]{: .label .label-purple }
[Implementation-Heavy]{: .label .label-yellow }

---

## Learning Objective

Implement the strangler pattern to gradually route traffic between legacy and new systems without downtime.

> **The safest migration is one you can reverse at any moment.**
{: .highlight }

---

## The Strangler Pattern

Instead of a risky "big bang" rewrite:

```
Before:  Client → Legacy System (100% traffic)

After:   Client → Gateway → Legacy (some %)
                         → New System (some %)
```

**The pattern:**
1. Put a facade (gateway) in front of both systems
2. Route all traffic through the gateway
3. Gradually shift traffic to the new system
4. Rollback instantly if anything breaks
5. Eventually retire the legacy system

**Why "strangler"?** Like a strangler fig that gradually wraps around a host tree, eventually replacing it entirely.

---

## Starting Point

Make sure both systems are running:

```bash
# Terminal 1: Legacy system (port 5001)
cd ~/labs/legacy-orderflow
docker-compose up -d  # or: python app.py

# Terminal 2: Your greenfield system (port 8000)
cd ~/labs/sdd-greenfield-starter
docker-compose up -d  # or: uvicorn src.main:app
```

Verify both are accessible:

```bash
curl http://localhost:5001/health  # Legacy
curl http://localhost:8000/health  # New (from Part 1)
```

---

## Step 1: Create the Gateway Spec (20 min)

Before building, specify what the gateway needs to do.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Create a specification for an API Gateway that:

1. Accepts ALL requests currently handled by the legacy system 
   (reference @legacy-orderflow/app.py for endpoint inventory)
2. Routes to either legacy (port 5001) or new system (port 8000) based on feature flags
3. Supports percentage-based traffic splitting (0-100%)
4. Logs every routing decision for analysis
5. Falls back to legacy if new system returns 5xx error
6. Adds correlation ID to all proxied requests for tracing

Create .specify/gateway/strangler-facade.md with:
- Endpoint routing table (which endpoints route where)
- Feature flag schema
- Fallback behavior spec
- Logging requirements
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo:

```text
/speckit.specify

Create .specify/gateway/strangler-facade.md for an API Gateway that:

1. List all endpoints from ../legacy-orderflow/app.py that need proxying
2. Route to legacy (5001) or new (8000) via feature flags
3. Support percentage-based traffic splitting
4. Log routing decisions as JSON
5. Fall back to legacy on 5xx errors from new system
6. Add X-Correlation-ID to all requests

Include: routing table, feature flag schema, fallback spec.
```

</details>

### Expected Routing Table

| Endpoint Pattern | Initial Routing | Notes |
|-----------------|-----------------|-------|
| `/health` | Both (compare) | Verify systems agree |
| `/products*` | Legacy | Read-only, low risk |
| `/register`, `/login` | Legacy | Auth stays on legacy |
| `/pay*` | New (Part 1) | Your payment service! |
| `/orders*` | Legacy | High risk, migrate later |
| `/admin/*` | Legacy | Needs auth fix first |
| `/cart/*` | Legacy | Depends on orders |

---

## Step 2: Implement the Gateway (40 min)

Build a FastAPI gateway that proxies requests based on feature flags.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.plan

Plan the API Gateway implementation based on .specify/gateway/strangler-facade.md:

Technology:
- FastAPI (consistent with Part 1)
- httpx for async HTTP proxying
- Structured JSON logging

Create implementation plan for:
- src/gateway/main.py - Gateway app
- src/gateway/router.py - Routing logic with feature flags
- src/gateway/proxy.py - HTTP proxying with fallback
- src/gateway/config.py - Feature flag configuration
```

Then:

```text
/speckit.implement

Implement the gateway based on the plan. Start with src/gateway/main.py and config.py.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create src/gateway/main.py:

- FastAPI app on port 8080
- Catch-all route that proxies to legacy or new system
- Feature flags dict in config.py controls routing
- Log every request: path, destination, response time
- Add X-Correlation-ID if not present
```

Then:

```text
Create src/gateway/proxy.py:

- async function proxy_request(request, target_url)
- Copy all headers, body, method
- Return response with same status, headers, body
- On 5xx from target: log error, optionally retry other target
```

</details>

### Core Gateway Code

Your AI should generate something like:

```python
# src/gateway/main.py
from fastapi import FastAPI, Request
from .config import get_target, LEGACY_URL, NEW_URL
from .proxy import proxy_request
import uuid
import logging

app = FastAPI(title="Strangler Gateway")
logger = logging.getLogger(__name__)

@app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
async def gateway(request: Request, path: str):
    # Add correlation ID for tracing
    correlation_id = request.headers.get("X-Correlation-ID", str(uuid.uuid4()))
    
    # Determine target based on feature flags
    target = get_target(path)
    target_url = NEW_URL if target == "new" else LEGACY_URL
    
    logger.info({
        "event": "routing",
        "path": path,
        "target": target,
        "correlation_id": correlation_id
    })
    
    # Proxy the request
    response = await proxy_request(request, f"{target_url}/{path}", correlation_id)
    
    # Fallback to legacy if new system fails
    if target == "new" and response.status_code >= 500:
        logger.warning({"event": "fallback", "path": path, "reason": "5xx from new"})
        response = await proxy_request(request, f"{LEGACY_URL}/{path}", correlation_id)
    
    return response
```

```python
# src/gateway/config.py
import os
import random

LEGACY_URL = os.getenv("LEGACY_URL", "http://localhost:5001")
NEW_URL = os.getenv("NEW_URL", "http://localhost:8000")

# Feature flags: which paths route to new system
FEATURE_FLAGS = {
    "payment": {"enabled": True, "percentage": 100},   # All payment to new
    "orders": {"enabled": True, "percentage": 0},      # All orders to legacy
    "products": {"enabled": True, "percentage": 0},    # Products on legacy
    "health": {"enabled": True, "percentage": 50},     # 50/50 for testing
}

def get_target(path: str) -> str:
    """Determine routing target based on path and feature flags."""
    # Find matching flag
    for prefix, config in FEATURE_FLAGS.items():
        if path.startswith(prefix) or f"/{prefix}" in path:
            if config["enabled"] and random.random() * 100 < config["percentage"]:
                return "new"
    return "legacy"
```

---

## Step 3: Add Comparison Mode (15 min)

Before cutting over, validate that both systems return equivalent responses.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Add comparison mode to src/gateway/proxy.py:

When COMPARE_MODE=true:
1. Call BOTH legacy and new system for the same request
2. Compare responses (status code, key response fields)
3. Log any discrepancies with details
4. Always return the legacy response (comparison is passive)

This lets us validate the new system without risk.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Add compare_mode() function to src/gateway/proxy.py:

async def compare_mode(request, path):
    # Call both systems in parallel
    legacy_response = await proxy_to(LEGACY_URL, request, path)
    new_response = await proxy_to(NEW_URL, request, path)
    
    # Compare and log discrepancies
    if not responses_match(legacy_response, new_response):
        log_discrepancy(path, legacy_response, new_response)
    
    # Always return legacy (safe)
    return legacy_response
```

</details>

### Comparison Logging

```python
def log_discrepancy(path: str, legacy: Response, new: Response):
    """Log differences between legacy and new responses."""
    logger.warning({
        "event": "discrepancy",
        "path": path,
        "legacy_status": legacy.status_code,
        "new_status": new.status_code,
        "legacy_body_preview": str(legacy.json())[:200] if legacy.ok else None,
        "new_body_preview": str(new.json())[:200] if new.ok else None,
    })
```

---

## Step 4: Test the Gateway (15 min)

Run the gateway and verify routing works.

```bash
# Start the gateway (port 8080)
cd ~/labs/sdd-greenfield-starter
uvicorn src.gateway.main:app --port 8080
```

### Test Routing

```bash
# Should route to legacy (orders stay on legacy)
curl http://localhost:8080/products
curl http://localhost:8080/orders

# Should route to new (payment on your Part 1 service)
curl http://localhost:8080/pay -X POST -H "Content-Type: application/json" \
  -d '{"amount": 100, "token": "tok_test"}'

# Health should be 50/50 - run multiple times
curl http://localhost:8080/health
curl http://localhost:8080/health
curl http://localhost:8080/health
```

### Check Routing Logs

```bash
# Look for routing decisions in logs
# You should see: {"event": "routing", "path": "...", "target": "legacy|new"}
```

---

## Step 5: Commit Your Work (5 min)

```text
Commit the gateway implementation with message: 
feat: implement strangler facade for legacy integration
```

---

## Success Criteria

- [ ] Gateway runs on port 8080
- [ ] Requests proxy correctly to legacy (5001)
- [ ] Payment endpoints route to new system (8000)
- [ ] Feature flags control routing percentages
- [ ] 5xx from new system falls back to legacy
- [ ] Routing decisions are logged
- [ ] Comparison mode logs discrepancies
- [ ] `.specify/gateway/strangler-facade.md` documents the design

---

## The Gateway Architecture

```
                    ┌─────────────────────────────────────────────────┐
                    │              Gateway (port 8080)                │
                    │                                                 │
  Client ────────▶  │  ┌─────────────┐    ┌────────────────────────┐  │
                    │  │   Router    │───▶│  Proxy + Fallback      │  │
                    │  │  (flags)    │    │  (retry on 5xx)        │  │
                    │  └─────────────┘    └────────────────────────┘  │
                    │         │                    │     │            │
                    │         ▼                    ▼     ▼            │
                    └─────────────────────────────────────────────────┘
                              │                    │     │
                    ┌─────────┴─────────┐          │     │
                    ▼                   ▼          ▼     ▼
             ┌──────────┐        ┌──────────┐  ┌──────────┐
             │ Feature  │        │  Legacy  │  │   New    │
             │  Flags   │        │  (5001)  │  │  (8000)  │
             └──────────┘        └──────────┘  └──────────┘
```

---

## Reflection Questions

1. **Gradual migration value**: What would "big bang" migration look like without the strangler pattern? What could go wrong?

2. **Fallback confidence**: The gateway falls back to legacy on 5xx errors. How does this change your risk assessment of deploying new code?

3. **Comparison mode insight**: Why run both systems in parallel and compare responses? What bugs would this catch that tests miss?

4. **Part 1 contrast**: In Part 1, you built from scratch with full control. Here, you're routing between systems you don't fully control. How does that change your spec needs?

---

## Key Takeaways

1. **Facade before rewrite** -- Put a gateway in front before changing anything
2. **Gradual is safe** -- 0% → 1% → 10% → 50% → 100% with validation at each step
3. **Fallback is mandatory** -- If new fails, legacy saves you
4. **Compare before commit** -- Validate responses match before real traffic

---

## What's Next?

In **Lab 2.4**, you'll extract notification logic from the legacy system into a shared event-driven service that both systems can use.

**The gateway routes requests. Events route data changes.**
