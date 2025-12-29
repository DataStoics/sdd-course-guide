---
title: "Lab 1.1: Init and Spec"
layout: default
parent: Labs
nav_order: 2
---
# Lab 1.1: Init and Spec -- Ship Thursday, Demo Friday

**Duration**: 90 minutes  
**Day**: 1  
**Position**: Immediately after Lab 0  
**Prerequisites**: Working development environment (Codespaces or local Docker)

---

## Learning Objective

Set up your AI assistant using `specify init .`, then transform the vague "Friday Demo" requirement into a spec that gets you to Thursday with demoable, production-path code.

By the end of this lab, you'll understand: **30 minutes of spec writing saves hours of Thursday night rework.**

---

## Connection to Contrast Exercise

Remember the Friday Demo scenario? You had 30 minutes (simulating 4 days) and ended up with:

- Code you wouldn't confidently demo
- Features that work individually but break together  
- Thursday night energy spent on rework, not polish

**This lab shows you the Monday morning that leads to a calm Thursday.**

---

## Starting Point

- GitHub repository cloned from `sdd-greenfield-template`
- Working dev container environment
- AI assistant (GitHub Copilot, Claude Code, or Gemini CLI)

---

## The Scenario (Continued)

Same PM message. Same deadline. Different approach.

> "Hey! We're pitching to investors on Friday. Need a working checkout demo -- payments, order history, the works. Something that looks real. Can you have it ready by end of day Thursday?"

**Your move**: Instead of jumping into code, you clarify what "looks real" means.

---

## Step 1: Initialize Your AI Assistant (10 min)

Before writing any code or specs, set up your AI assistant with the SDD workflow.

### Run the Init Command

From your repository root, run:

```bash
specify init .
```

### Select Your AI Assistant

The init wizard will display a menu. Use arrow keys to select your AI assistant:

```
â•­â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€ Choose your AI assistant: â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â•®
â”‚                                                                                                         â”‚
â”‚         copilot (GitHub Copilot)                                                                        â”‚
â”‚         claude (Claude Code)                                                                            â”‚
â”‚  â–¶      gemini (Gemini CLI)                                                                             â”‚
â”‚         cursor-agent (Cursor)                                                                           â”‚
â”‚         ...                                                                                             â”‚
```

**Select the one you're using** (Gemini CLI is recommended for this course).

### What Just Happened?

The init command created a `.specify/` directory with configuration for your selected assistant:

| Assistant | Config Folder Created |
|-----------|----------------------|
| GitHub Copilot | `.github/` with `copilot-instructions.md` |
| Claude Code | `.claude/` with `settings.json` |
| Gemini CLI | `.gemini/` with `settings.json` |

It also created:
- `.specify/` - Contains templates, scripts, and governance rules
- `.specify/templates/` - Spec templates for consistent documentation
- `.specify/scripts/` - Helper scripts for the SDD workflow

**Open the generated instructions file** and review it. This file tells your AI assistant:
- How to work with specifications
- Where to find governance constraints
- How to structure generated code
- What patterns to follow

### Verify Setup

Your AI assistant should now understand SDD. Test it:

```
What is spec-driven development and how should I create specs in this project?
```

If your AI responds with information about specifications and governance, you're ready to proceed.

---

## Step 2: Create the Spec Directory (5 min)

Create the specification structure:

```bash
mkdir -p specs/001-payment
touch specs/001-payment/spec.md
```

Open `specs/001-payment/spec.md` in your editor.

---

## Step 3: Generate Initial Spec with /speckit.specify (10 min)

Now use the AI-assisted specification command:

```
/speckit.specify "payment checkout for investor demo - must handle edge cases gracefully, look professional, and be demoable by Friday"
```

Notice how different this is from the contrast exercise prompt. You're telling the AI:
- **What** you're building (payment checkout)
- **What context** matters (investor demo)
- **What "done" means** (demoable, handles edge cases)

Review the generated spec structure. You should have sections for:
- Summary
- Demo Requirements (what investors will see)
- Edge Cases (what could break the demo)
- Technical Constraints

---

## Step 4: Define What "Demoable" Actually Means (25 min)

**This is where Thursday night rework gets prevented.** What would embarrass you in front of investors?

### The "Don't Embarrass Me" Test

Think about demo day. What could go wrong?

| Demo Disaster | Prevention (goes in spec) |
|---------------|---------------------------|
| User clicks Pay twice - double charge | Idempotency handling |
| Payment fails - ugly error | Graceful error responses |
| "Is this secure?" question | Tokenization, no raw cards |
| Demo with test data - crashes | Validation for all inputs |

### Translating to Constraints

These demo concerns become technical constraints:

```markdown
## CONSTRAINTS

### Payment Safety (prevents demo disasters)
- **No raw card storage**: Use tokens from Mock Payment Gateway (also: real security)
- **Idempotency required**: Handle "Pay" button double-clicks gracefully
- **Validation**: Reject bad input with helpful errors, not crashes
```

### Data Handling (answers investor questions)

If an investor asks "How do you handle data?", you want a confident answer:

```markdown
### Data Handling
- **Retention policy**: Transactions kept 7 years (audit-ready from day 1)
- **Failed attempts**: Cleaned up after 14 days (not cluttering DB)
- **User data**: Deletable on request (GDPR-ready)
```

### Audit Trail (proves it's real)

Investors spot fake demos. Logging proves the system actually works:

```markdown
### Audit Trail
- **All transactions logged**: Timestamp, actor, outcome
- **Correlation IDs**: Can trace any request end-to-end
- **Immutable**: Logs can't be faked
```

---

## Step 5: Write Demo Scenarios (20 min)

These are the moments you'll show investors. Each scenario becomes a test AND a demo script.

### Scenario 1: The Happy Path (this is the main demo)

```markdown
**Given** a customer with a valid payment token,
**When** they submit a payment for $50.00,
**Then** payment succeeds,
**And** they see a confirmation with transaction ID,
**And** the transaction appears in their order history.
```

*Demo moment: "Here's a customer completing checkout..."*

### Scenario 2: The Double-Click (proves robustness)

```markdown
**Given** a payment was just processed,
**When** the customer clicks Pay again (network hiccup, impatient user),
**Then** they see the same confirmation (not a duplicate charge),
**And** we log it as a duplicate attempt.
```

*Demo moment: "Watch what happens if I click Pay twice..."*

### Scenario 3: The Error Recovery (proves professionalism)

```markdown
**Given** a customer submits an invalid payment token,
**When** the payment is attempted,
**Then** they see a helpful error ("Payment method expired, please update"),
**And** no partial charge occurs,
**And** they can retry with a new token.
```

*Demo moment: "And if something goes wrong, users get a clear message..."*

### Scenario 4: The Audit Question (proves enterprise-readiness)

```markdown
**Given** a completed transaction,
**When** an investor asks "Can you trace transactions?",
**Then** we can show: timestamp, customer, amount, outcome, trace ID.
```

*Demo moment: "Every transaction is fully auditable from day one."*

---

## Step 6: Define Requirements (10 min)

Turn scenarios into numbered requirements (your AI will reference these):

```markdown
## Requirements

### Functional Requirements

- **FR-001**: Accept payment tokens from Mock Payment Gateway (no raw card data)
- **FR-002**: Validate idempotency keys, return original response for duplicates
- **FR-003**: Log all payment attempts with timestamp, outcome, trace ID
- **FR-004**: Return helpful errors for invalid tokens, missing data
- **FR-005**: Show transaction in order history after successful payment
```

---

## Step 7: The "Ship Thursday" Checklist (5 min)

Verify your spec prevents Thursday night rework:

- [ ] **Double-click safe?** Idempotency handling specified
- [ ] **Error states?** Graceful failures, not crashes
- [ ] **Audit-ready?** Logging requirements defined
- [ ] **Demoable?** Happy path clearly specified
- [ ] **Defensible?** Can answer "is this secure?"

**If any item is unchecked, that's your Thursday night rework waiting to happen.**

---

## Step 8: Commit Your Work (5 min)

Your first commit should follow conventional commit format:

```bash
git add .
git commit -m "feat: payment specification for investor demo"
```

Note: We're adding all files (including the AI config from `specify init .`).

---

## Success Criteria

Your lab is complete when:

- [ ] AI assistant configured (`.github/copilot-instructions.md`, `.claude/instructions.md`, or `.gemini/instructions.md` exists)
- [ ] `specs/001-payment/spec.md` exists
- [ ] Spec contains constraints section (idempotency, data handling, audit)
- [ ] Spec has at least 4 demo scenarios (Given/When/Then)
- [ ] Spec has numbered requirements (FR-001, FR-002, etc.)
- [ ] Commit message follows convention: `feat: payment specification`

### Validate Your Work

Run the validation script:

```bash
python validate_lab.py --lab 1.1 --repo .
```

All checks should pass.

---

## Contrast Exercise vs. Lab 1.1

| Aspect | Contrast Exercise (Monday AM) | Lab 1.1 (Monday, after spec) |
|--------|-------------------------------|------------------------------|
| Time spent | 30 min coding | 30 min spec, then coding |
| Double-click | Causes duplicate charge | Returns original response |
| Bad input | Crash or ugly error | Helpful error message |
| "Is this secure?" | "Uh... I think so?" | "Yes, here's the spec" |
| Thursday night | Reworking everything | Polishing details |
| Friday demo | Hoping nothing breaks | Confident walkthrough |

**Same effort, different outcome.**

---

## Reflection Questions

Before moving to Lab 1.2, consider:

1. **Time math**: You spent ~30 min on spec. In the contrast exercise, how much time would you have spent Thursday night fixing edge cases? Was the spec investment worth it?

2. **AI behavior**: With the spec, your AI will generate idempotency handling automatically. Without it, the AI "didn't know" you needed it. What changed?

3. **Confidence**: Rate your demo confidence now vs. contrast exercise. What made the difference?

---

## Common Mistakes

| Mistake | What Happens Thursday Night |
|---------|----------------------------|
| Skipping `specify init .` | AI doesn't follow your patterns |
| Vague constraints ("handle errors") | AI picks random error patterns |
| No idempotency handling | Demo crashes on double-click |
| Untestable scenarios | Can't prove it works |
| Skipping spec entirely | You ARE the contrast exercise |

---

## What's Next?

In **Lab 1.2**, you'll use `/speckit.research` to make technology decisions (FastAPI vs Flask? Redis vs in-memory?) with documented trade-offs.

**The spec says WHAT. Research says HOW. Plan commits to WHICH.**

**Remember**: You're building toward Thursday. Every decision documented now is rework prevented later.
