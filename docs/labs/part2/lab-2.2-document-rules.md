---
title: "Lab 2.2: Document Business Rules"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 3
---

# Lab 2.2: Document Business Rules
{: .fs-9 }

Why Does It Do That?
{: .fs-6 .fw-300 }

[~60 min]{: .label .label-blue }

---

## Learning Objective

Extract implicit business rules from code and formalize them into stakeholder-readable documentation.

> **The code is the only source of truth for legacy systems. Your job is to translate code-truth into human-truth.**
{: .highlight }

---

## The Problem

Business rules are hidden in code like this:

```python
if total > DISCOUNT_THRESHOLD:
    discount = total * DISCOUNT_RATE
```

**Questions that code doesn't answer:**
- What IS `DISCOUNT_THRESHOLD`? Who decided this value?
- Why 10% discount? Is there documentation? An approval email?
- When does this apply? All customers? Only certain orders?

**Your job**: Find all the hidden rules and make them visible.

---

## Step 1: Hunt Magic Numbers (20 min)

Find every hardcoded value with business meaning.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Scan @legacy-orderflow/app.py and extract ALL business rules:

1. **Magic Numbers**: All hardcoded numeric values (100, 10000, 0.08, 5.99, etc.)
2. **Magic Strings**: Hardcoded status values ("pending", "paid", "shipped", error messages)
3. **Implicit Thresholds**: Conditional boundaries (if total > X, if days < Y)

For each rule found, document:
- Value
- Location (line number or function name)
- Apparent purpose
- Confidence level (High/Medium/Low)

Create .specify/legacy-integration/business-rules.md
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo:

```text
Scan ../legacy-orderflow/app.py and extract ALL business rules.

Magic Numbers:
- All hardcoded numeric values (100, 10000, 0.08, 5.99, etc.)

Magic Strings:
- Status values, error messages, category names

Implicit Thresholds:
- if total > X, if days < Y, if count >= Z

For each: value, location, apparent purpose, confidence (high/medium/low).

Create .specify/legacy-integration/business-rules.md
```

</details>

### Expected Findings

| Category | Value | Location | Purpose | Confidence |
|----------|-------|----------|---------|------------|
| **Pricing** | 0.08 | `TAX_RATE` | Tax percentage (8%) | High |
| **Pricing** | 5.99 | `SHIPPING_RATE` | Flat shipping | High |
| **Pricing** | 7.99 | `EXPRESS_SHIPPING` | Express option | High |
| **Pricing** | 14.99 | `OVERNIGHT_SHIPPING` | Overnight option | High |
| **Order** | 100 | `MIN_ORDER` | Minimum order $ | High |
| **Order** | 10000 | `MAX_ORDER` | Maximum order $ | High |
| **Discount** | 500 | `DISCOUNT_THRESHOLD` | Bulk discount trigger | Medium |
| **Discount** | 0.1 | `DISCOUNT_RATE` | 10% off bulk orders | Medium |
| **Discount** | 5000 | inline | VIP threshold? | Low |
| **Discount** | 1000 | `BULK_*` | Another discount tier? | Low |
| **Refund** | 30 | inline | Refund window days | Medium |
| **Credit** | 5000 | inline | Credit limit? | Low |
| **Audit** | 0.9 | inline | 90% sampling? | Low |

{: .warning }
> **Low confidence items are dangerous.** You found the value, but don't understand its purpose. Flag these for stakeholder clarification.

---

## Step 2: Map State Transitions (20 min)

Every order has a lifecycle. Document it.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
/speckit.specify

Map the order state machine from @legacy-orderflow/app.py:

1. What status values exist? (Look for all possible order["status"] values)
2. What transitions are allowed? (What functions change status, and from what to what?)
3. What triggers each transition? (User action? Admin? System?)
4. Are any transitions blocked? (Can you cancel after shipping? Refund after 30 days?)

Create .specify/legacy-integration/order-states.md with:
- ASCII state diagram
- Transition table (from → to → trigger → conditions)
- Edge cases (blocked transitions)
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Map the order state machine in ../legacy-orderflow/app.py:

- What status values exist?
- What functions change status?
- What conditions allow/block transitions?
- Who can trigger each transition (user/admin/system)?

Create .specify/legacy-integration/order-states.md with ASCII diagram.
```

</details>

### Expected State Diagram

```
                     ┌─────────────┐
                     │   pending   │◄────────────────────────┐
                     └──────┬──────┘                         │
                            │                                │
              ┌─────────────┼─────────────┐                  │
              │ /pay        │             │ /cancel          │
              ▼             │             ▼                  │
        ┌──────────┐        │       ┌────────────┐           │
        │   paid   │        │       │ cancelled  │           │
        └────┬─────┘        │       └────────────┘           │
             │              │                                │
             │ /ship        │                                │
             ▼              │                                │
        ┌──────────┐        │                                │
        │ shipped  │────────┴──(can't cancel after ship)     │
        └────┬─────┘                                         │
             │                                               │
             │ /deliver                                      │
             ▼                                               │
        ┌───────────┐                                        │
        │ delivered │                                        │
        └────┬──────┘                                        │
             │                                               │
             │ /refund (within 30 days)                      │
             ▼                                               │
        ┌──────────┐                                         │
        │ refunded │─────────────────────────────────────────┘
        └──────────┘    (order can be re-placed)
```

### Transition Table

| From | To | Trigger | Conditions | Who |
|------|-----|---------|------------|-----|
| pending | paid | `/orders/{id}/pay` | Payment succeeds | User |
| pending | cancelled | `/orders/{id}/cancel` | None | User/Admin |
| paid | shipped | `/admin/orders/{id}/ship` | Admin only | Admin |
| paid | cancelled | `/orders/{id}/cancel` | Must be admin | Admin |
| shipped | delivered | `/admin/orders/{id}/deliver` | Admin only | Admin |
| delivered | refunded | `/orders/{id}/refund` | Within 30 days | User |
| delivered | refunded | `/orders/{id}/refund` + X-Admin-Override | Any time | Admin |

{: .warning }
> Notice the admin backdoor: `X-Admin-Override` header bypasses the 30-day refund window. This is a security concern!

---

## Step 3: Document User Types and Permissions (10 min)

Who can do what?

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Analyze @legacy-orderflow/app.py for user types and permissions:

1. What user types exist? (Look for user["type"] or role checks)
2. What can each type do? (Different endpoints, different limits)
3. How is admin determined? (Is there an admin user type? Header check?)
4. What's the auth model? (Sessions? Tokens? How long do they last?)

Add to .specify/legacy-integration/business-rules.md under "User Types & Permissions"
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Analyze ../legacy-orderflow/app.py for user permissions:

- User types? (regular, credit, vip, admin?)
- What can each type do?
- How is admin status determined?
- Session/token model?

Add to .specify/legacy-integration/business-rules.md
```

</details>

### Expected Findings

| User Type | Description | Special Abilities |
|-----------|-------------|-------------------|
| `regular` | Default users | Standard checkout |
| `credit` | Credit account | Can accumulate balance up to $5,000 |
| VIP (implicit) | Lifetime spend > $5,000 | 5% discount on all orders |

**Admin Access** (⚠️ Security Issue):
- No admin user type in system
- Admin functions check for `X-Admin-Override: true` header
- **Anyone who knows the header can act as admin!**

---

## Step 4: Flag Risky Rules (10 min)

Some business rules are dangerous. Highlight them.

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Review .specify/legacy-integration/business-rules.md and add a "⚠️ Flagged Issues" section:

Look for:
1. Security risks (auth bypasses, exposed secrets)
2. Data integrity risks (silent truncation, missing validation)
3. Compliance risks (incomplete audit logging, hardcoded retention)
4. Business logic inconsistencies (conflicting rules, unclear thresholds)

For each: what's the issue, where is it, what's the risk level, what should we do about it.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Add "⚠️ Flagged Issues" section to .specify/legacy-integration/business-rules.md:

From your analysis, what's risky?
- Security issues (admin backdoor, debug endpoints)
- Data issues (90% audit sampling, silent failures)
- Business logic bugs (boundary conditions)

For each: issue, location, risk level, recommended action.
```

</details>

### Expected Flagged Issues

```markdown
## ⚠️ Flagged Issues

### Critical

1. **Admin Backdoor**: `X-Admin-Override` header grants admin access to anyone
   - Location: Multiple admin endpoints
   - Risk: Unauthorized administrative actions
   - Action: Replace with proper role-based auth before production

2. **Debug Endpoints in Production**: `/debug/*` endpoints expose secrets
   - `/debug/env` — leaks all environment variables including API keys
   - `/debug/users` — exposes password hashes
   - `/debug/sessions` — exposes all active session tokens
   - Action: Remove or protect before integration

3. **MD5 Password Hashing**: Cryptographically broken
   - Location: `hash_password()` function
   - Risk: Password compromise if database leaked
   - Action: Migrate to bcrypt (requires password reset flow)

### High

4. **Incomplete Audit Logging**: Only 90% of actions logged
   - Location: `should_log()` returns `random.random() > 0.1`
   - Risk: Compliance failure, missing audit trail
   - Action: Log 100% of security-relevant actions

5. **Unused Security Decorator**: `@require_admin` defined but never used
   - Location: Decorator exists but admin endpoints don't use it
   - Risk: False sense of security
   - Action: Apply decorator to all admin endpoints

### Medium

6. **Silent Cart Truncation**: Orders > MAX_ITEMS_PER_ORDER silently truncated
   - Risk: Customer confusion, partial orders
   - Action: Return error instead of silent truncation
```

---

## Success Criteria

- [ ] `.specify/legacy-integration/business-rules.md` contains:
  - Pricing rules (tax, shipping, discounts)
  - Order constraints (min/max amounts, item limits)
  - User types and permissions
  - Flagged issues with severity levels
- [ ] `.specify/legacy-integration/order-states.md` contains:
  - State diagram (ASCII)
  - Transition table
  - Blocked transitions documented
- [ ] At least 10 magic numbers documented with purposes
- [ ] At least 5 flagged issues with risk levels
- [ ] Document is readable by non-technical stakeholders

---

## The Business Rules Document: Your Integration Contract

This document serves multiple purposes:

| Audience | What They Learn |
|----------|----------------|
| **Developers** | What rules to preserve during integration |
| **QA** | What edge cases to test |
| **Product** | What business logic exists (they may not have known!) |
| **Security** | What vulnerabilities need fixing |
| **Compliance** | What audit gaps exist |

**This is the conversation starter with stakeholders.** Before integration, get sign-off that these rules are correct (or should be changed).

---

## Reflection Questions

1. **Hidden knowledge**: How many business rules did you find that probably only existed in the original developer's head? What would happen during integration without this document?

2. **State diagram value**: Could you have drawn the order state machine by reading the code alone? How does visualizing it change your understanding?

3. **Security vs. timeline**: You found security issues. In a real project, would you fix them now, or document them for later? What factors influence that decision?

4. **Part 1 contrast**: In Part 1, you specified business rules before coding. Here, you're extracting rules from code. Which approach produces better documentation?

---

## Key Takeaways

1. **Magic numbers have meaning** — Every hardcoded value is a business decision someone made
2. **State machines hide complexity** — Order lifecycles are more complex than they look
3. **Security debt accumulates** — Legacy code collects auth shortcuts and debug backdoors
4. **Document the dangers** — Flagged issues inform what to fix vs. what to accept

---

## What's Next?

In **Lab 2.3**, you'll implement the strangler pattern — an API gateway that lets you gradually route traffic between legacy and new systems without downtime or big-bang migrations.

**You've documented what the legacy system does. Now you'll learn how to safely replace it.**
