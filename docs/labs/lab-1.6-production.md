---
title: "Lab 1.6: Production"
layout: default
parent: Labs
nav_order: 7
---
# Lab 1.6: Production Readiness â€” Friday Morning Polish

**Duration**: 90 minutes  
**Day**: 2 (Final Lab)  
**Prerequisites**: Completed Lab 1.5 with integrated features

---

## Learning Objective

Package for production, run final checks, and make sure nothing embarrassing happens during the demo. By end of this lab, you're ready for the Friday investor pitch.

---

## Where We Are in the Week

```
Monday:      âœ… Spec + Plan
Tuesday:     âœ… Payment built
Wednesday:   âœ… Order spec
Thursday:    âœ… Integration complete
Friday AM:   ðŸ‘‰ YOU ARE HERE â€” Final polish
Friday PM:   Demo day ðŸŽ¯
```

**Friday morning goal**: Production container, security check, demo rehearsal.

---

## Starting Point

- Working payment + order integration from Lab 1.5
- All tests passing with 80%+ coverage
- Demo scenarios verified

---

## Step 1: Create Production Dockerfile (20 min)

Create a multi-stage `Dockerfile` in the project root:

```dockerfile
# =============================================================================
# Stage 1: Builder
# =============================================================================
FROM python:3.11-slim as builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Create virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt


# =============================================================================
# Stage 2: Production
# =============================================================================
FROM python:3.11-slim as production

# Create non-root user for security
RUN groupadd --gid 1000 appgroup && \
    useradd --uid 1000 --gid appgroup --shell /bin/bash --create-home appuser

WORKDIR /app

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy application code
COPY --chown=appuser:appgroup src/ ./src/

# Set Python environment
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

# Expose port
EXPOSE 8000

# Run the application
CMD ["uvicorn", "src.app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Key Production Patterns

| Pattern | Reason |
|---------|--------|
| Multi-stage build | Smaller final image (no build tools) |
| Non-root user | Security best practice |
| HEALTHCHECK | Container orchestration support |
| PYTHONDONTWRITEBYTECODE | Avoid .pyc files in container |

---

## Step 2: Build and Test Container (15 min)

```bash
# Build the production image
docker build -t sdd-workshop:latest .

# Run the container
docker run -d --name sdd-test \
  -p 8000:8000 \
  -e REDIS_URL=redis://host.docker.internal:6379 \
  -e PAYMENT_GATEWAY_URL=http://host.docker.internal:8001 \
  sdd-workshop:latest

# Test health endpoint
curl http://localhost:8000/health

# View logs
docker logs sdd-test

# Clean up
docker stop sdd-test && docker rm sdd-test
```

### Verify Container Size

```bash
docker images sdd-workshop:latest --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Target**: < 200MB (Python slim base + dependencies)

---

## Step 3: Final Security Scan (15 min)

### Semgrep Scan

```bash
semgrep --config p/security-audit src/
```

**Pass Criteria**: 0 CRITICAL + 0 HIGH findings

### Bandit Scan

```bash
bandit -r src/ -ll
```

**Pass Criteria**: No high-severity issues

### Container Security

```bash
# Scan the built image (if Trivy is available)
trivy image sdd-workshop:latest
```

---

## Step 4: CI/CD Pipeline Verification (15 min)

Verify your `.github/workflows/ci.yml` includes production checks:

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install ruff
      - name: Run linter
        run: ruff check src/

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/security-audit
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install Bandit
        run: pip install bandit
      - name: Run Bandit
        run: bandit -r src/ -ll

  test:
    runs-on: ubuntu-latest
    services:
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests with coverage
        run: pytest --cov=src --cov-report=xml --cov-fail-under=80
        env:
          REDIS_URL: redis://localhost:6379

  build:
    runs-on: ubuntu-latest
    needs: [lint, security, test]
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t sdd-workshop:${{ github.sha }} .
```

Push a commit and verify all jobs pass.

---

## Step 5: Demo Day Checklist (10 min)

**The "Would I Demo This?" Checklist**:

### Security (investor Q: "Is this secure?")

- [ ] No hardcoded secrets in code
- [ ] Environment variables for config
- [ ] Non-root user in container
- [ ] Security scan passing

### Reliability (investor Q: "Will it crash?")

- [ ] Health check endpoint works
- [ ] Double-click handled gracefully
- [ ] Error responses are helpful

### Professionalism (investor Q: "Is this real?")

- [ ] Structured JSON logging
- [ ] Trace IDs work
- [ ] API docs auto-generated

---

## Step 6: Demo Rehearsal (10 min)

Run through the demo script:

```bash
# Start services
docker-compose up -d

# Demo Flow 1: Create order and pay
curl -X POST http://localhost:8000/orders -H "Content-Type: application/json" \
  -d '{"items": [{"product_id": "p1", "quantity": 2, "unit_price": 2500}], "idempotency_key": "demo-001"}'

# Get token and pay
curl -X POST http://localhost:8001/tokenize -d '...'
curl -X POST http://localhost:8000/pay -d '...'

# Demo Flow 2: View order history
curl http://localhost:8000/orders

# Demo Flow 3: Show audit trail
curl http://localhost:8000/orders/{id}/audit
```

**Rehearse until it's smooth.** Know what to say at each step.

---

## Step 7: Commit and Breathe (5 min)

```bash
git add .
git commit -m "feat: production-ready for demo day"
```

**ðŸŽ¯ You're ready for the Friday demo.**

---

## Success Criteria

Your lab is complete when:

- [ ] `Dockerfile` exists with multi-stage build
- [ ] `docker build` succeeds  
- [ ] `docker run` starts the application
- [ ] Health check returns healthy
- [ ] Demo rehearsal runs smoothly
- [ ] You'd confidently demo this to investors

### Validate Your Work

```bash
python validate_lab.py --lab 1.6 --repo . --security-scan
```

---

## The Week in Review

| Day | What Happened | Without SDD |
|-----|---------------|-------------|
| **Monday** | Wrote specs, made decisions | Started coding, made assumptions |
| **Tuesday** | Built payment, tests pass | Built something, "works on my machine" |
| **Wednesday** | Order spec in 30 min | "One more thing" panic |
| **Thursday** | Integration, full flow works | Still debugging payment |
| **Friday** | Demo rehearsal, confident | Hoping nothing crashes |

**The spec-first approach didn't slow you down. It prevented the Thursday night rewrite.**

---

## Course 2 Preview: What About Legacy Code?

This course was the easy path: empty repo, full control.

In Course 2, you'll face the real world:

```
            Course 1                    Course 2
       â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”        â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
       â”‚  Greenfield     â”‚        â”‚  Brownfield     â”‚
       â”‚  Empty repo     â”‚        â”‚  5000+ lines    â”‚
       â”‚  You write spec â”‚        â”‚  Spec is hidden â”‚
       â”‚  Full control   â”‚        â”‚  Tech debt      â”‚
       â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜        â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              â†“                          â†“
       Monday spec,         vs    Extract spec FROM
       Friday ship                 existing code
```

### Course 2 Challenges

- **Extract implicit specs** from working code
- **Refactor without breaking** production  
- **Retrofit governance** to legacy systems
- **Work with MCP servers** for internal knowledge

The spec-first thinking you learned here becomes **spec-extraction** thinking there.

---

## Reflection: Contrast Exercise Callback

Remember Monday morning? The contrast exercise?

| Then | Now |
|------|-----|
| 30 min of coding, wouldn't demo it | 2 days, confident demo |
| No idempotency | Double-click safe |
| "Is this secure?" â€” "Uh..." | "Yes, here's the spec" |
| Thursday night panic | Thursday night beer |

**The difference wasn't skill. It was approach.**

---

## What You've Accomplished

In two days, you transformed a vague PM request into:

âœ… Authoritative specifications with governance constraints  
âœ… Researched technology decisions with documented trade-offs  
âœ… Working payment endpoint with idempotency  
âœ… Order service with state machine  
âœ… Integrated checkout flow  
âœ… 80%+ test coverage  
âœ… Security scan passing  
âœ… Production-ready container  
âœ… **A demo you're confident giving**

**Same effort as the contrast exercise approach. Different outcome.**

---

## Next Steps

1. **Demo day**: Nail the investor pitch
2. **Apply SDD**: Use this on your next work project  
3. **Share**: Teach a teammate the spec-first approach
4. **Course 2**: Learn to extract specs from legacy code

**Thank you for participating in SDD Foundations Course 1!**

ðŸŽ‰ **Now go demo that checkout flow.**
