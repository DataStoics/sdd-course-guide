---
title: "Lab 2.6: Migration Planning"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 7
---

# Lab 2.6: Migration Planning
{: .fs-9 }

From Legacy to Done
{: .fs-6 .fw-300 }

[~45 min]{: .label .label-blue }
[Final Lab]{: .label .label-green }

---

## Learning Objective

Create a production-ready migration plan with rollback capabilities and risk mitigation.

> **The migration plan is a spec for the migration itself. Just like code specs prevent rework, migration specs prevent outages.**
{: .highlight }

---

## The Deliverable

A migration plan that could actually be executed. This is what you'd present to leadership to get approval for the cutover.

---

## Step 1: Define Migration Phases (15 min)

Every safe migration follows phases. Each phase has entry criteria, success metrics, and rollback triggers.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Create a phased migration plan for transitioning from legacy-orderflow to the integrated system:

Phase 0: Preparation (current state)
- Characterization tests passing
- Gateway deployed with 0% new traffic
- Monitoring and alerting configured

Phase 1: Shadow Mode (Week 1)
- Compare responses without switching traffic
- Log all discrepancies
- Success: <0.1% discrepancy rate

Phase 2: Canary Release (Week 2-3)
- 5% → 10% → 25% traffic to new system
- Monitor error rates and latency
- Success: Error rate <1%, latency within 10% of legacy

Phase 3: Gradual Rollout (Week 4-5)
- 50% → 75% → 90% traffic to new system
- Success: No customer complaints, metrics stable

Phase 4: Full Cutover (Week 6)
- 100% traffic to new system
- Legacy running in standby
- Success: 48 hours stable operation

Phase 5: Legacy Decommission (Week 8+)
- Archive legacy data
- Shutdown legacy services
- Success: Clean shutdown, no dependencies

For each phase: duration, entry criteria, success metrics, rollback trigger.
Create .specify/migration/migration-plan.md
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create .specify/migration/migration-plan.md with 6 phases:

0. Preparation (done) - characterization tests, gateway, monitoring
1. Shadow Mode - compare without routing
2. Canary (5-25%) - small traffic
3. Gradual (50-90%) - increasing traffic
4. Cutover (100%) - legacy standby
5. Decommission - archive and shutdown

Each phase needs:
- Duration
- Entry criteria (what must be true to start)
- Success metrics (how we know it's working)
- Rollback triggers (when to abort)
- Rollback procedure (how to go back)
```

</details>

### Expected Phase Structure

```markdown
## Phase 2: Canary Release

**Duration**: 2 weeks
**Traffic Split**: 5% → 10% → 25% to new system

### Entry Criteria
- [ ] Phase 1 complete with <0.1% discrepancy rate
- [ ] Rollback tested in staging
- [ ] On-call team briefed and scheduled
- [ ] Runbook reviewed by team

### Success Metrics
- [ ] Error rate <1% on new system
- [ ] P99 latency within 10% of legacy baseline
- [ ] No data inconsistencies detected in unified reports
- [ ] Customer support tickets within normal range

### Traffic Ramp Schedule
| Day | New Traffic % | Decision Point |
|-----|--------------|----------------|
| 1-3 | 5% | Monitor, adjust |
| 4-7 | 10% | Review metrics |
| 8-10 | 25% | Hold if issues |
| 11-14 | 25% (hold) | Final review |

### Rollback Triggers (Any One)
- Error rate >5% sustained for 5 minutes
- Any data corruption or inconsistency detected
- P99 latency >2x legacy baseline
- Critical customer complaint related to migration
- Any on-call page related to new system

### Rollback Procedure
1. Set `FEATURE_FLAGS["orders"]["percentage"] = 0`
2. Verify: `curl gateway/debug/routing` shows 100% legacy
3. Monitor: Error rate returns to baseline
4. Notify: #migration-channel "Rollback complete, investigating"
5. Do NOT proceed until root cause identified and fixed
```

---

## Step 2: Risk Assessment (15 min)

What could go wrong? Plan for it.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Add a Risk Assessment section to .specify/migration/migration-plan.md:

For each risk:
- Description
- Likelihood (High/Medium/Low)
- Impact (Critical/High/Medium/Low)
- Mitigation strategy
- Contingency if it happens

Consider risks in these categories:
1. Technical (data sync, performance, compatibility)
2. Operational (on-call capacity, deployment issues)
3. Business (revenue impact, customer complaints)
4. Timeline (delays, dependencies)
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Add Risk Assessment to .specify/migration/migration-plan.md:

| Risk | Likelihood | Impact | Mitigation | Contingency |
|------|------------|--------|------------|-------------|
| Data sync failure | Medium | High | Comparison mode | Rollback, fix sync |
| Performance degradation | Low | Medium | Canary catches early | Rollback, optimize |
| Feature parity gap | Medium | High | Characterization tests | Rollback, add feature |
| Rollback fails | Low | Critical | Test in staging | Manual DNS switch |
```

</details>

### Expected Risk Matrix

| Risk | L | I | Mitigation | Contingency |
|------|---|---|------------|-------------|
| **Data sync failure** | M | H | Comparison mode validates responses match before traffic shift | Rollback immediately; investigate discrepancy logs |
| **Performance degradation** | L | M | Canary phase catches issues at low traffic | Rollback; profile new system; optimize |
| **Feature parity gap** | M | H | Characterization tests ensure behavior match | Rollback; add missing feature to new system |
| **Rollback failure** | L | C | Test rollback procedure in staging before each phase | Manual feature flag update; DNS failover as backup |
| **Cascading failure** | L | C | Circuit breakers in gateway; fallback to legacy | Disable gateway; direct traffic to legacy |
| **Data corruption** | VL | C | Unified reports detect inconsistencies | Stop writes; restore from backup; post-mortem |
| **On-call overload** | M | M | Stagger rollout; clear escalation path | Slow down rollout; add on-call staff |
| **Customer complaints** | M | M | Support team briefed; FAQ prepared | Rollback if >5 complaints in 1 hour |

---

## Step 3: Create the Runbook (15 min)

Operational documentation for the team executing the migration.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Create .specify/migration/runbook.md with operational procedures:

1. **Health Checks**
   - How to verify legacy is healthy
   - How to verify new system is healthy
   - How to verify gateway is healthy
   - Dashboard links

2. **Traffic Control**
   - How to check current routing percentages
   - How to increase traffic to new system
   - How to decrease traffic (rollback)
   - Feature flag locations and format

3. **Emergency Procedures**
   - Full rollback (route everything to legacy)
   - Gateway bypass (direct to legacy)
   - Data recovery steps

4. **Monitoring & Alerts**
   - Key metrics to watch
   - Alert thresholds
   - Who gets paged for what

5. **Escalation**
   - L1: On-call engineer (rollback authority)
   - L2: Tech lead (extend rollout decision)
   - L3: Engineering director (abort migration decision)
   - Contact info for each level
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Create .specify/migration/runbook.md:

Sections:
1. Health Checks (curl commands for each service)
2. Traffic Control (how to modify feature flags)
3. Emergency Rollback (step by step)
4. Monitoring (what to watch, where)
5. Escalation (who to call)

Include actual commands, not just descriptions.
```

</details>

### Expected Runbook Content

```markdown
# Migration Runbook

## Health Checks

### Legacy System (port 5001)
```bash
curl http://localhost:5001/health
# Expected: {"status": "ok", "version": "2.3.1"}
```

### New System (port 8000)
```bash
curl http://localhost:8000/health
# Expected: {"status": "healthy", "version": "1.0.0"}
```

### Gateway (port 8080)
```bash
curl http://localhost:8080/health
curl http://localhost:8080/debug/routing  # Shows current traffic split
```

## Traffic Control

### Check Current Routing
```bash
curl http://localhost:8080/debug/config | jq '.feature_flags'
```

### Increase Traffic to New System
```bash
# Edit src/gateway/config.py or use API if implemented:
# Change: "orders": {"enabled": True, "percentage": 10}
# To:     "orders": {"enabled": True, "percentage": 25}

# Restart gateway or call config reload endpoint
curl -X POST http://localhost:8080/admin/reload-config
```

### Emergency Rollback (Route ALL to Legacy)
```bash
# Option 1: Feature flag
curl -X POST http://localhost:8080/admin/rollback-all

# Option 2: Manual config
# Set ALL percentages to 0 in config.py
# Restart gateway

# Option 3: Gateway bypass
# Update DNS/load balancer to point directly to legacy:5001
```

## Escalation Contacts

| Level | Role | Contact | Authority |
|-------|------|---------|-----------|
| L1 | On-call engineer | PagerDuty | Rollback |
| L2 | Tech lead | [name] | Pause rollout |
| L3 | Eng director | [name] | Abort migration |
```

---

## Step 4: Commit and Review (5 min)

```text
Commit the migration plan and runbook with message:
docs: add migration plan and operations runbook for legacy integration
```

### Final Review Checklist

Before presenting to leadership:

- [ ] All phases have clear entry/exit criteria
- [ ] Rollback procedures tested (at least on paper)
- [ ] Risk matrix reviewed with team
- [ ] Runbook commands verified to work
- [ ] On-call schedule confirmed for migration period
- [ ] Communication plan for customers/support
- [ ] Success metrics measurable (not vague)

---

## Success Criteria

- [ ] `.specify/migration/migration-plan.md` contains:
  - 6 phases with clear criteria
  - Risk assessment matrix
  - Timeline estimate
- [ ] `.specify/migration/runbook.md` contains:
  - Health check commands
  - Traffic control procedures
  - Emergency rollback steps
  - Escalation contacts
- [ ] Plan reviewed by at least one other person
- [ ] Rollback procedure tested conceptually

---

## Part 2 Complete: What You've Built

Over the past 6 labs, you've transformed an undocumented legacy system into a manageable, integrated architecture:

| Lab | Artifact | Value |
|-----|----------|-------|
| 2.0 | System analysis | Know what you're dealing with |
| 2.1 | Characterization tests | Executable documentation |
| 2.2 | Business rules doc | Human-readable specs |
| 2.3 | Strangler gateway | Safe routing infrastructure |
| 2.4 | Event-driven notifications | Decoupled integration |
| 2.5 | Unified reporting | Single view of truth |
| 2.6 | Migration plan | Roadmap to completion |

---

## The Part 1 → Part 2 Skill Transfer

| Part 1 Skill | Part 2 Application |
|--------------|-------------------|
| Write specs before code | Extract specs from existing code |
| Plan before building | Plan migrations with rollback |
| Test-driven development | Characterization testing |
| Clean architecture | Strangler pattern integration |
| API design | Unified facades and aggregation |
| Deployment | Phased migration with monitoring |

**Same spec-first discipline. Different starting point.**

---

## The Brownfield Reality

Most software work isn't greenfield. Most of your career will involve:

- Code you didn't write
- Decisions you don't understand
- Systems that "just work" (until they don't)
- Migrations that can't fail

The skills you've learned in Part 2 -- systematic exploration, characterization testing, safe migration patterns -- are the skills that make senior engineers valuable.

---

## Key Takeaways

1. **Specs work both directions** -- Write specs for new code, extract specs from existing code
2. **Migration is a spec** -- The plan is the authoritative document for the migration
3. **Rollback is not failure** -- It's a successful risk mitigation
4. **Phase gates protect you** -- Don't skip entry criteria, even under pressure

---

## Course Complete!

You've completed **SDD Part 2: Transforming Legacy Code**.

**What you can now do:**
- Systematically explore undocumented codebases
- Extract implicit specifications from existing behavior
- Safely integrate legacy and modern systems
- Plan and execute migrations with confidence

**What's next?**
- Apply these skills to your own legacy systems
- Share the characterization testing pattern with your team
- Consider: What legacy system at work needs this treatment?

---

## Reflection: The Full Journey

| Day | Part 1: Greenfield | Part 2: Brownfield |
|-----|-------------------|-------------------|
| Morning | AI chaos → SDD intro | System archaeology |
| Midday | First spec | Characterization tests |
| Afternoon | Implementation | Business rules extraction |
| Evening | Integration | Gateway + unified views |
| Next AM | Production prep | Migration planning |
| Next PM | Demo day! | Migration complete |

**Two days. Two very different challenges. Same disciplined approach.**

---

**Congratulations on completing the SDD Foundations course!**

**Now go make your legacy systems better.**
