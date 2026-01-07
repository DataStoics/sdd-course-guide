---
title: Quick Reference
layout: default
nav_order: 10
permalink: /quick-reference/
---

# Quick Reference

Everything you need on one page. Print this or keep it in a second tab.

---

## The SDD Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SDD Workflow                                 │
│                                                                      │
│   1. SPECIFY        2. CLARIFY       3. PLAN         4. IMPLEMENT   │
│   ───────────       ──────────       ─────           ───────────    │
│   What to build?    Remove fog       How to build?   Build it       │
│                                                                      │
│   Vague idea ──▶ Structured spec ──▶ Tech decisions ──▶ Code       │
│                                                                      │
│   [Natural Language] → [Requirements] → [Architecture] → [Tests+Code]│
└─────────────────────────────────────────────────────────────────────┘
```

---

## spec-kit Commands

### CLI Commands (run in terminal)

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `specify init . --ai copilot` | Initialize project | Once, at project start |
| `specify version` | Check installed version | Troubleshooting |

### Slash Commands (in AI chat)

| Command | Purpose | Output |
|---------|---------|--------|
| `/speckit.constitution` | Define project principles | `.specify/memory/constitution.md` |
| `/speckit.specify` | Transform idea → spec | `specs/NNN-feature/spec.md` |
| `/speckit.clarify` | Resolve ambiguities | Updates `spec.md` |
| `/speckit.plan` | Technology decisions | `plan.md`, `research.md`, `data-model.md` |
| `/speckit.tasks` | Break down into tasks | `tasks.md` |
| `/speckit.analyze` | Check consistency | Validation report |
| `/speckit.implement` | Generate code | `src/`, `tests/` |
| `/speckit.checklist <type>` | Validate completeness | Checklist report |

### Checklist Types

```
/speckit.checklist security       # Security scan
/speckit.checklist demo-readiness # Pre-demo validation
/speckit.checklist production     # Production checklist
```

---

## Spec Structure Template

```markdown
# Feature: [Name]

## Overview
One paragraph describing the feature and its purpose.

## Functional Requirements
- **FR-001**: System MUST [do something specific and testable]
- **FR-002**: System MUST [another requirement]
- **FR-003**: System SHOULD [nice-to-have requirement]

## Non-Functional Requirements
- **NFR-001**: Response time < 200ms for 95th percentile
- **NFR-002**: Handle 1000 concurrent requests
- **NFR-003**: 99.9% uptime

## User Scenarios

### Scenario 1: [Happy Path] (Priority: P1)
- **Given** [initial context]
- **When** [action taken]
- **Then** [expected outcome]

### Scenario 2: [Edge Case] (Priority: P1)
- **Given** [context with edge condition]
- **When** [action taken]
- **Then** [graceful handling]

## Data Model
| Entity | Fields | Relationships |
|--------|--------|--------------|
| Order | id, status, total | has many Items |
| Item | id, name, price | belongs to Order |

## Constraints & Assumptions
- Using Redis for session storage (decision: X)
- Mock payment gateway for demo
- Single-region deployment initially

## Open Questions
- [ ] What's the refund policy? → Need PM input
```

---

## Requirement Keywords

| Keyword | Meaning | Example |
|---------|---------|---------|
| **MUST** | Required, no exceptions | System MUST validate input |
| **MUST NOT** | Prohibited | System MUST NOT store plain passwords |
| **SHOULD** | Recommended, but negotiable | System SHOULD log all requests |
| **SHOULD NOT** | Discouraged | System SHOULD NOT expose internal IDs |
| **MAY** | Optional | System MAY cache responses |

---

## The "Would I Demo This?" Checklist

### Security (Investor asks: "Is this secure?")
- [ ] No hardcoded secrets in code
- [ ] Environment variables for configuration
- [ ] Input validation on all endpoints
- [ ] Non-root user in container
- [ ] Security scan passing (semgrep, bandit)

### Reliability (Investor asks: "Will it crash?")
- [ ] Health check endpoint works
- [ ] Double-click handled gracefully (idempotency)
- [ ] Error responses are helpful, not stack traces
- [ ] Tests pass with 80%+ coverage

### Professionalism (Investor asks: "Is this real?")
- [ ] Structured JSON logging
- [ ] Trace IDs propagate through requests
- [ ] API docs auto-generated at /docs
- [ ] Container size < 200MB

---

## Common Patterns

### Idempotency Pattern

```python
# Store idempotency key with result
@app.post("/pay")
async def process_payment(request: PaymentRequest):
    # Check if we've seen this key before
    existing = await redis.get(f"idem:{request.idempotency_key}")
    if existing:
        return json.loads(existing)  # Return cached result
    
    # Process payment
    result = await process_new_payment(request)
    
    # Cache result (expire after 24h)
    await redis.setex(
        f"idem:{request.idempotency_key}",
        86400,
        json.dumps(result)
    )
    return result
```

### State Machine Pattern

```python
# Define valid transitions
VALID_TRANSITIONS = {
    "pending": ["paid", "cancelled"],
    "paid": ["shipped", "refunded"],
    "shipped": ["delivered"],
    "delivered": [],  # Terminal state
    "cancelled": [],  # Terminal state
    "refunded": [],   # Terminal state
}

def can_transition(current: str, target: str) -> bool:
    return target in VALID_TRANSITIONS.get(current, [])
```

### Graceful Error Pattern

```python
# Return helpful errors, not stack traces
class PaymentError(Exception):
    def __init__(self, code: str, message: str, retry_after: int = None):
        self.code = code
        self.message = message
        self.retry_after = retry_after

@app.exception_handler(PaymentError)
async def payment_error_handler(request, exc):
    return JSONResponse(
        status_code=400,
        content={
            "error": exc.code,
            "message": exc.message,
            "retry_after": exc.retry_after,
        }
    )
```

---

## Traceability Markers

Every piece of code should trace back to a requirement.

```python
# Good: traceable
def validate_amount(amount: float) -> bool:
    """Validate payment amount per FR-003.
    
    Business rule: $0.01 - $10,000 USD
    Source: specs/001-payment/spec.md#FR-003
    """
    return 0.01 <= amount <= 10000.00

# Bad: untraceable
def validate_amount(amount):
    return amount > 0 and amount < 10000
```

---

## TDD Cycle (AI-Assisted)

```
    ┌─────────────────────────────────────────┐
    │                                          │
    │   1. RED: Write failing test from spec   │
    │         (AI reads scenario)              │
    │                ↓                         │
    │   2. GREEN: Implement to pass test       │
    │         (AI writes minimum code)         │
    │                ↓                         │
    │   3. REFACTOR: Clean up                  │
    │         (AI improves structure)          │
    │                ↓                         │
    │   4. COMMIT: Lock in progress            │
    │         (You validate)                   │
    │                ↓                         │
    │         [Next Task]                      │
    │                                          │
    └─────────────────────────────────────────┘
```

---

## Part 1 vs Part 2 Comparison

| Aspect | Part 1: Greenfield | Part 2: Brownfield |
|--------|-------------------|-------------------|
| Starting point | Empty repo | 2,500+ lines of code |
| Spec source | You write it | Extract from code |
| First step | `/speckit.specify` | Characterization tests |
| Tech choices | Full control | Inherited constraints |
| Testing mindset | "Verify correctness" | "Document reality" |
| Risk profile | Low (you control it) | High (unknown unknowns) |

---

## Key Metrics

| Metric | Target | Why |
|--------|--------|-----|
| Test coverage | ≥ 80% | Confidence threshold |
| Security findings | 0 CRITICAL, 0 HIGH | Demo safety |
| Response time | < 200ms (p95) | User experience |
| Container size | < 200MB | Deployment efficiency |
| Build time | < 5 min | Developer velocity |

---

## Troubleshooting

### "/speckit.* commands not found"
```bash
# You must run specify init before slash commands work
specify init . --ai copilot

# Verify initialization succeeded
ls -la .specify/
ls -la .github/prompts/  # For Copilot

# Re-initialize if needed
specify init . --ai copilot --force
```

### "Copilot keeps asking for approval"
This is normal behavior in Agent mode:
- Click "Allow" or "Continue" to approve actions
- "Continue to iterate?" prompts appear during long implementations
- For faster workflow, approve commands in batches when offered

### "Implementation seems stuck"
- Check file explorer for newly created files
- Ask: "What's the current implementation status?"
- If no progress for 2+ minutes, try: `/speckit.implement --continue`
- Long implementations (60+ minutes) are normal for complex specs

### "AI ignoring my spec"
- Check: Is spec file in `specs/NNN-feature/spec.md`?
- Check: Did you run `/speckit.tasks` first?
- Try: Reference spec explicitly: "According to FR-001 in specs/..."

### "Tests failing unexpectedly"
- Check: Are you testing the spec or testing assumptions?
- Check: Did characterization tests change? (Part 2)
- Try: Re-read the scenario -- does code match spec?

### "Docker won't build"
```bash
# Clean up
docker system prune -f
docker-compose down -v
docker-compose up -d --build
```

---

## AI Prompting Tips

### Be Specific
```
# Bad
"Make it better"

# Good  
"Update the payment endpoint to return HTTP 409 
 when idempotency_key already exists, per FR-002"
```

### Reference Specs
```
# Bad
"Add error handling"

# Good
"Add error handling for Scenario 3 in 
 specs/001-payment/spec.md (graceful recovery)"
```

### Request Traceability
```
"Implement FR-004 with a comment linking 
 to the spec requirement"
```

---

## Emergency Commands

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Discard all changes
git checkout -- .
git clean -fd

# Check container status
docker-compose ps
docker-compose logs -f

# Run tests with verbose output
pytest -v --tb=long

# Security scan
semgrep --config=p/security-audit src/
bandit -r src/
```

---

**The spec is the contract. AI implements it. You validate and demo.**
