# Lab 1.2: Research and Technology Decisions

**Duration**: 45 minutes  
**Day**: 1  
**Prerequisites**: Completed Lab 1.1 with `specs/001-payment/spec.md`

---

## Learning Objective

Use `/speckit.research` to document technology decisions and trade-offs **before** implementing. Understand that research creates an auditable record of **why** you chose specific technologies — not just what you chose.

---

## The SDD Workflow

```
✅ Lab 1.1: specify init . → spec.md (DONE)
👉 Lab 1.2: /speckit.research → research.md → plan.md (YOU ARE HERE)
   Lab 1.3: /speckit.tasks → tasks.md → implementation
```

**Why research before planning?**
- Specs define WHAT and WHY (governance)
- Research explores HOW (options, trade-offs)
- Plans commit to WHICH (decisions)

---

## Starting Point

- Repository with `specs/001-payment/spec.md` from Lab 1.1
- AI assistant configured (`specify init .` completed)
- All governance constraints defined

---

## Step 1: Generate Research Document (10 min)

Ask your AI assistant to research implementation options:

```
/speckit.research
```

Or prompt manually:

> "Based on specs/001-payment/spec.md, research technology options for implementing this payment feature. For each governance constraint, identify 2-3 implementation approaches and recommend one with justification."

The generated `specs/001-payment/research.md` should explore:

### Key Research Questions

| Governance Constraint | Research Question |
|-----------------------|-------------------|
| PCI tokenization | How to integrate with payment gateway? SDK vs REST? |
| Idempotency (60s cache) | Redis vs in-memory? Distributed vs local? |
| Audit logging | Structured logging library? Log format? |
| GDPR retention | Soft delete vs hard delete? Archival strategy? |

---

## Step 2: Review Technology Trade-offs (15 min)

Open `specs/001-payment/research.md` and verify it contains:

### 2a. Options Analysis

For each major decision, you should see alternatives considered:

**Example: Idempotency Cache**

| Option | Pros | Cons |
|--------|------|------|
| Redis | Distributed, TTL built-in, fast | External dependency |
| In-memory dict | Simple, no deps | Lost on restart, not distributed |
| Database | Persistent | Slower, overkill for 60s TTL |

**Decision**: Redis — matches our 60-second TTL requirement, handles distributed deployments

### 2b. Constraint Traceability

Every technology choice must trace to a spec constraint:

| Decision | Spec Constraint | Justification |
|----------|-----------------|---------------|
| FastAPI | Performance, OpenAPI | Async for gateway calls, auto-docs |
| Redis | Idempotency: 60s cache | Native TTL, distributed |
| httpx | External gateway calls | Async HTTP client |
| structlog | Audit trail | JSON logging, correlation IDs |

**Key Question**: Can you justify every technology from the spec?

---

## Step 3: Generate Implementation Plan (10 min)

Now that research is complete, generate the plan:

```
/speckit.plan
```

Or prompt:

> "Based on specs/001-payment/research.md decisions, create a plan.md with project structure, dependencies, and implementation phases."

The plan should include:

1. **Committed decisions** (from research)
2. **Project structure** 
3. **Dependencies list**
4. **Implementation phases**

### Example plan.md Structure

```markdown
# Payment Feature Plan

## Decisions (from research.md)
- Framework: FastAPI
- Cache: Redis  
- HTTP Client: httpx
- Logging: structlog

## Project Structure
src/app/
├── main.py          # FastAPI entry point
├── models.py        # Pydantic models
├── payment.py       # Payment service
└── config.py        # Configuration

## Dependencies
fastapi>=0.100.0
redis>=4.5.0
httpx>=0.24.0
structlog>=23.1.0
pydantic>=2.0.0

## Phases
1. Models + Config
2. Payment service
3. API endpoint
4. Tests
```

---

## Step 4: Create Project Scaffold (5 min)

Based on your plan, create the structure:

```bash
# Create source directories
mkdir -p src/app
mkdir -p tests

# Create initial files
touch src/app/__init__.py
touch src/app/main.py
touch src/app/models.py
touch src/app/payment.py
touch src/app/config.py
touch tests/__init__.py
touch tests/test_payment.py
```

---

## Step 5: Verify Infrastructure (3 min)

The template includes `docker-compose.yml`. Verify services start:

```bash
# Start services
docker-compose up -d

# Verify Redis
docker-compose exec redis redis-cli ping
# Expected: PONG

# Verify Mock Payment Gateway
curl http://localhost:8001/health
# Expected: {"status":"healthy"...}
```

---

## Step 6: Commit Your Work (2 min)

```bash
git add .
git commit -m "feat: research and plan for payment feature"
```

---

## Success Criteria

Your lab is complete when:

- [ ] `specs/001-payment/research.md` exists with technology options analyzed
- [ ] `specs/001-payment/plan.md` exists with committed decisions
- [ ] Every technology choice traces to a spec constraint
- [ ] `src/app/` directory structure created
- [ ] `docker-compose up` starts Redis and Mock Payment Gateway
- [ ] Commit includes both research.md and plan.md

### Validate Your Work

```bash
python validate_lab.py --lab 1.2 --repo .
```

---

## Research vs. Plan: Key Differences

| Research (research.md) | Plan (plan.md) |
|------------------------|----------------|
| Explores options | Commits to decisions |
| Documents trade-offs | Documents structure |
| "We could use X, Y, or Z" | "We will use X" |
| WHY we considered alternatives | WHAT we're building |
| Auditable decision record | Implementation blueprint |

**Both are governance artifacts** — if an auditor asks "why Redis?", you point to research.md.

---

## Reflection Questions

1. **Audit trail**: If a new team member asks "why not PostgreSQL for caching?", where do they find the answer?

2. **Change process**: If you need to switch from Redis to Memcached, what documents need updating? (Answer: research.md rationale, plan.md decision, then implementation)

3. **Constraint-driven**: What would happen if you chose a technology that violates a spec constraint?

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping research, jumping to plan | Research documents WHY; it's your audit trail |
| Technology not traceable to spec | Either add requirement to spec or remove technology |
| Copying boilerplate without analysis | AI should analyze YOUR spec, not generic patterns |
| research.md = plan.md | They serve different purposes; keep them separate |

---

## Advanced: Integrating External Research Sources (Optional)

The research phase isn't limited to your AI assistant's training data. Teams can integrate **live sources** for better-informed decisions:

| Source Type | Examples | Use Case |
|-------------|----------|----------|
| **Live Search** | Perplexity | Latest versions, security advisories |
| **Library Docs** | Context7 | Up-to-date API references |
| **Internal Docs** | Confluence, SharePoint | Company standards, approved tech |
| **Security DBs** | Snyk, NVD | CVE checks before adopting |

### Example with Live Sources

```
Research caching options for our idempotency requirement.
- Check Context7 for Redis best practices
- Search Perplexity for recent security advisories  
- Document findings with sources and dates
```

### Why This Matters for Audits

When using external sources, document them:

```markdown
## Sources Consulted
| Source | Date | Finding |
|--------|------|---------|
| Context7 (Redis docs) | 2025-12-29 | Use TTL for expiration |
| Perplexity search | 2025-12-29 | No critical CVEs |
```

**This creates an auditable trail** — you can prove decisions were based on current, verified information.

> 📖 **See Also**: [MCP Integration Guide](../reference/mcp-integration.md) for setup details

---

## What's Next?

In **Lab 1.3**, you'll use `/speckit.tasks` to break down the plan into implementable tasks, then use `/speckit.implement` to generate compliant payment endpoint code.

**The research and plan are your implementation contract.** Lab 1.3 implements exactly what's documented — no more, no less.
