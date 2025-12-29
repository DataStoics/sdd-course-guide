# SDD Workflow Quick Reference

## The Four-Phase SDD Workflow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   SPECIFY   │ ──▶ │    PLAN     │ ──▶ │  IMPLEMENT  │ ──▶ │   VERIFY    │
│             │     │             │     │             │     │             │
│ Create spec │     │ AI creates  │     │ AI executes │     │ Tests +     │
│ with human  │     │ impl plan   │     │ tasks from  │     │ security    │
│ governance  │     │ from spec   │     │ plan        │     │ scans       │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

## Phase 1: SPECIFY

**Goal**: Create governance-ready specification

**Command**: `/speckit.specify` or manual spec creation

**Checklist**:
- [ ] Clear business requirement statement
- [ ] GOVERNANCE CONSTRAINTS section
- [ ] Acceptance scenarios (Given/When/Then)
- [ ] Security requirements (explicit)
- [ ] Data requirements (what's stored, for how long)
- [ ] Error handling expectations

**Key Question**: "Would an auditor know what we're building and why?"

---

## Phase 2: PLAN

**Goal**: Generate implementation roadmap from spec

**Command**: `/speckit.plan`

**Checklist**:
- [ ] Technology choices trace to spec requirements
- [ ] Tasks are atomic and testable
- [ ] Dependencies identified
- [ ] Security tasks included (not afterthought)
- [ ] Test tasks parallel implementation

**Key Question**: "Does every plan item trace back to a spec requirement?"

---

## Phase 3: IMPLEMENT

**Goal**: Execute plan with AI assistance

**Command**: `/speckit.implement` or task-by-task

**Checklist**:
- [ ] Follow task order from plan
- [ ] Reference spec in AI prompts
- [ ] Run tests after each task
- [ ] Don't skip security tasks
- [ ] Document deviations

**Key Question**: "Is this code doing what the spec says?"

---

## Phase 4: VERIFY

**Goal**: Confirm implementation meets spec

**Commands**: 
- `pytest --cov` (tests + coverage)
- `semgrep --config=auto` (security)
- `bandit -r src/` (Python security)

**Checklist**:
- [ ] All acceptance scenarios have passing tests
- [ ] Coverage ≥ 80%
- [ ] Security scan: 0 CRITICAL, 0 HIGH
- [ ] Governance constraints verified
- [ ] Documentation updated

**Key Question**: "Can we prove this implementation meets the spec?"

---

## Spec-Constrained Prompting

### Bad Prompt (Vague)
```
Create a payment endpoint
```

### Good Prompt (Spec-Constrained)
```
Implement POST /pay per specs/001-payment/spec.md FR-001:
- Accept idempotency_key (required)
- Never log card_number
- Return structured PaymentResult
- Add audit log entry for compliance
```

---

## When Things Go Wrong

| Problem | Solution |
|---------|----------|
| AI generates wrong code | Spec may be vague — add detail |
| Security scan fails | Add security requirement to spec |
| Tests don't match behavior | Check acceptance scenarios |
| Plan doesn't match needs | Review spec completeness |

---

## Golden Rules

1. **Spec First**: Never implement without a spec
2. **Trace Everything**: Every code decision → spec requirement
3. **Test Continuously**: Run tests after every change
4. **Security Built-in**: Security in spec, not bolted on
5. **Iterate**: Specs evolve — update and regenerate

---

## Command Cheat Sheet

```bash
# Specify
/speckit.specify "new feature description"

# Clarify
/speckit.clarify

# Plan  
/speckit.plan

# Implement (per task)
/speckit.implement T001

# Verify
pytest --cov --cov-report=term-missing
semgrep --config=auto src/
```
