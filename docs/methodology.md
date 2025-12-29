---
title: SDD Methodology
layout: default
nav_order: 5
permalink: /methodology/
---

# The SDD Methodology

Spec-Driven Development is a framework for getting consistent, production-ready code from AI assistants.

---

## The Core Problem

AI coding assistants are powerful but unpredictable. Ask three developers to prompt the same AI for a payment endpoint, and you'll get three different implementationsâ€”different error handling, different security patterns, different assumptions.

**In regulated environments, this is dangerous.**

---

## The SDD Solution

Write the specification first. Not documentationâ€”a machine-readable contract that:

1. **Constrains** what the AI can produce
2. **Specifies** exactly what success looks like
3. **Embeds** governance requirements directly

---

## How Governance Fits In

Traditional approach: Build first, audit later, fix findings.

**SDD approach**: Encode governance into the spec. The AI implements compliance from the start.

### Example: PCI DSS in a Payment Spec

Instead of hoping the AI knows about tokenization, your spec says:

\\\markdown
## GOVERNANCE CONSTRAINTS

### PCI DSS 4.0 Compliance
- System MUST NOT store raw card numbers (PAN)
- System MUST use tokenization via payment gateway
- All payment data MUST be logged without sensitive fields
\\\

The AI reads this and implements tokenization. No card numbers in your database. No compliance finding later.

### Example: GDPR in an Order Spec

\\\markdown
## GOVERNANCE CONSTRAINTS

### GDPR Compliance
- Personal data MUST be deletable within 30 days of request
- Order history MUST be retained for 7 years (financial regulations)
- Audit trail MUST track who accessed what, when
\\\

The AI implements soft delete with retention policies. Audit trail built in.

---

## The SDD Workflow

\\\
1. SPECIFY  â†’  Write governance-aware spec
2. RESEARCH â†’  Document technology decisions
3. PLAN     â†’  Break down into tasks
4. IMPLEMENT â†’  AI generates compliant code
5. VERIFY   â†’  Tests + security scan
\\\

Each step produces an artifact. Each artifact feeds the next step.

---

## Why This Works

1. **Specs are prompts** â€” A well-structured spec is the ultimate prompt engineering
2. **Governance is code** â€” Compliance requirements become testable assertions
3. **AI is consistent** â€” Same spec = same output across tools and sessions
4. **Humans stay in control** â€” You define what; AI implements how

---

## Best Practices

### Writing Effective Specs

âœ… **Be explicit about constraints** â€” If it's not in the spec, the AI will decide  
âœ… **Include acceptance scenarios** â€” Given/When/Then format forces clarity  
âœ… **Specify error cases** â€” How should the system fail?  
âœ… **Define boundaries** â€” What's explicitly out of scope?

### Common Mistakes

âŒ **Vague requirements** â€” "Handle errors appropriately" means nothing  
âŒ **Missing governance** â€” Compliance bolted on later is expensive  
âŒ **Implementation in specs** â€” Describe what, not how  
âŒ **No acceptance criteria** â€” If you can't test it, you can't verify it

---

## Measuring Success

After implementing SDD, teams typically see:

| Metric | Before SDD | After SDD |
|:-------|:-----------|:----------|
| Rework from code review | 40-60% | 10-15% |
| Security scan findings | 5-10 per feature | 0-2 per feature |
| Time to compliance sign-off | Weeks | Days |
| Code consistency across team | Low | High |

---

{: .note }
> This methodology is practiced hands-on in the [Labs](labs/). You'll experience the difference between AI coding without specs (Contrast Exercise) and with specs (Labs 1.1-1.6).

