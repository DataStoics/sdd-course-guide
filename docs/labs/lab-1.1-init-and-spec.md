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

You'll be prompted to choose your AI assistant:

```
? Select your AI assistant:
  copilot
  claude
  gemini
  cursor-agent
```

**Select the one you're using** (the one you used in the contrast exercise).

### What Just Happened?

The init command created the spec-kit structure and slash commands for your selected assistant:

| Assistant | Commands Directory | Format |
|-----------|-------------------|--------|
| GitHub Copilot | `.github/prompts/` | Markdown |
| Claude Code | `.claude/commands/` | Markdown |
| Gemini CLI | `.gemini/commands/` | TOML |
| Cursor | `.cursor/commands/` | Markdown |

The init also created:
- `.specify/` - Core spec-kit directory with scripts, templates, and memory
- `.specify/memory/constitution.md` - Project principles (we'll define these next)
- `.specify/templates/` - Templates for specs, plans, and checklists

**Open your agent's commands directory** and review it. These slash commands enable the spec-driven workflow:
- `/speckit.specify` - Transform ideas into structured specifications
- `/speckit.clarify` - Ask questions to refine ambiguous requirements
- `/speckit.plan` - Generate implementation plans
- `/speckit.implement` - Execute implementation with traceability

### Verify Setup

Your AI assistant should now understand SDD. Test it:

```
What is spec-driven development and how should I create specs in this project?
```

If your AI responds with information about specifications and governance, you're ready to proceed.

---

## Step 2: Establish Project Principles with /speckit.constitution (5 min)

Before diving into features, establish the foundational principles that will guide all technical decisions.

### Run the Constitution Command

Simply describe what matters for your project:

```
/speckit.constitution This is an investor demo for a payment checkout system. 
Prioritize professional appearance, graceful error handling, and demo-ability. 
Code should be clean enough to show investors but pragmatic for the timeline.
```

**The beauty of spec-kit**: You don't need to structure your thoughts perfectly. Just describe what matters and the AI will help organize it into a proper constitution.

### What Gets Created?

The command updates `.specify/memory/constitution.md` with principles like:
- Code quality standards appropriate for your context
- Error handling philosophy
- Testing requirements
- Security considerations

**Review the generated constitution** and adjust if needed. This becomes the governance layer that guides all future AI-generated code.

---

## Step 3: Generate Initial Spec with /speckit.specify (10 min)

Now use the conversational specification command. **You don't need to structure your thoughts** - just describe what you're building:

```
/speckit.specify Payment checkout for our investor demo on Friday. 
Needs to handle credit card payments, show order history, and look professional. 
Should handle edge cases like double-clicks and invalid cards gracefully.
```

### What Just Happened?

The `/speckit.specify` command automatically:

1. **Created a feature branch**: `001-payment-checkout` (semantic naming from your description)
2. **Created the spec directory**: `specs/001-payment-checkout/`
3. **Generated `spec.md`**: Structured specification from your natural language input
4. **Ran quality validation**: Checked for completeness and clarity

### The Key Insight: AI Marks What It Doesn't Know

Open the generated `spec.md`. You'll see something powerful - the AI **explicitly marked its uncertainty** instead of guessing:

```markdown
## User Scenarios

### Scenario 1: Successful Payment (Priority: P1)
**Given** a customer with a valid payment token,
**When** they submit a payment for $50.00,
**Then** payment succeeds and confirmation displays.

## Functional Requirements

- **FR-001**: System MUST accept payment tokens [NEEDS CLARIFICATION: which payment gateway?]
- **FR-002**: System MUST handle duplicate submissions [NEEDS CLARIFICATION: return same response or error?]
- **FR-003**: System MUST validate payment amounts [NEEDS CLARIFICATION: min/max limits?]
- **FR-004**: System MUST display order history [NEEDS CLARIFICATION: how many past orders?]
```

**This is spec-kit's superpower**: Instead of the AI making plausible-but-wrong assumptions, it tells you exactly what's ambiguous. Those `[NEEDS CLARIFICATION]` markers are your Thursday night rework, surfaced on Monday morning.

### Review the Generated Structure

Your spec should have:
- Summary (what we're building)
- User Scenarios (Given/When/Then format)
- Functional Requirements (FR-001, FR-002... with uncertainty markers)
- Success Criteria (measurable outcomes)
- Key Entities (data structures involved)

---

## Step 4: Resolve Ambiguities with /speckit.clarify (15 min)

**This is where Thursday night rework gets prevented.** Those `[NEEDS CLARIFICATION]` markers? Let's resolve them now, not at 11pm Thursday.

### Run the Clarify Command

```
/speckit.clarify
```

The AI analyzes your spec and presents **ONE question at a time** with a recommended answer:

```
## Question 1: Payment Gateway Integration

**Context**: FR-001 specifies accepting payment tokens but doesn't specify the gateway.

**What we need to know**: Which payment gateway should we integrate with?

| Option | Answer | Implications |
|--------|--------|--------------|
| A (Recommended) | Mock Payment Gateway | Fast to implement, perfect for demo, easy to swap later |
| B | Stripe Test Mode | Real integration, but adds complexity and API keys |
| C | Direct card input | Security risk, NOT recommended for any demo |

**Your choice**: _[Type A, B, C, or provide custom answer]_
```

### Answer Each Question

Respond with your choice. The AI updates the spec and asks the next question:

- **Q1: Payment Gateway** → A (Mock gateway for demo)
- **Q2: Duplicate handling** → Return original response (idempotent)
- **Q3: Amount limits** → $0.01 - $10,000 USD
- **Q4: Order history depth** → Last 10 orders

After ~4-5 questions, your spec transforms from ambiguous to precise.

### Before vs. After Clarification

| Before | After |
|--------|-------|
| `[NEEDS CLARIFICATION: which payment gateway?]` | Mock Payment Gateway with Stripe-compatible tokens |
| `[NEEDS CLARIFICATION: return same response or error?]` | Return original confirmation (idempotent) |
| `[NEEDS CLARIFICATION: min/max limits?]` | $0.01 - $10,000 USD |

**This is the conversation that prevents Thursday rework.**

---

## Step 5: Define What "Demoable" Actually Means (10 min)

Now think about demo day. What could still embarrass you?

### The "Don't Embarrass Me" Test

| Demo Disaster | Prevention (goes in spec) |
|---------------|---------------------------|
| User clicks Pay twice - double charge | Idempotency handling ✓ (resolved in clarify) |
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

## Step 7: Verify Your Requirements (5 min)

After clarification, your requirements should be precise and testable:

```markdown
## Requirements

### Functional Requirements

- **FR-001**: Accept payment tokens from Mock Payment Gateway (no raw card data)
- **FR-002**: Validate idempotency keys, return original response for duplicates  
- **FR-003**: Log all payment attempts with timestamp, outcome, trace ID
- **FR-004**: Return helpful errors for invalid tokens, missing data
- **FR-005**: Show last 10 transactions in order history after successful payment
- **FR-006**: Validate payment amounts between $0.01 and $10,000 USD
```

Notice: **No `[NEEDS CLARIFICATION]` markers remain.** Every requirement is specific enough to implement and test.

---

## Step 8: Generate Quality Checklist with /speckit.checklist (5 min)

Instead of a manual checklist, let spec-kit generate one tailored to your spec:

```
/speckit.checklist demo-readiness
```

The AI analyzes YOUR spec and generates context-aware validation:

```markdown
# Demo Readiness Checklist: Payment Checkout

## Functional Completeness
- [ ] FR-001: Payment token acceptance - is gateway integration specified?
- [ ] FR-002: Idempotency - is duplicate detection mechanism defined?
- [ ] FR-005: Order history - is pagination/limit specified?

## Demo Safety
- [ ] Happy path scenario covers the main demo flow?
- [ ] Error scenarios won't crash or show ugly stack traces?
- [ ] Double-click protection explicitly required?

## Investor Questions
- [ ] "Is this secure?" - tokenization requirement documented?
- [ ] "Can you trace transactions?" - audit logging specified?
- [ ] "How do you handle failures?" - error recovery defined?
```

**Review each item.** Unchecked items are Thursday night rework waiting to happen.

**Pro tip**: Run `/speckit.checklist security` or `/speckit.checklist ux` for different focus areas.

---

## Step 9: Commit Your Work (5 min)

Your first commit should follow conventional commit format:

```bash
git add .
git commit -m "feat: payment checkout specification for investor demo"
```

Note: This commit includes:
- AI assistant configuration (`.github/prompts/` or equivalent)
- Project constitution (`.specify/memory/constitution.md`)
- Feature specification (`specs/001-payment-checkout/spec.md`)

---

## Success Criteria

Your lab is complete when:

- [ ] AI assistant configured (`.github/prompts/`, `.claude/commands/`, `.gemini/commands/`, or `.cursor/commands/` exists)
- [ ] Constitution defined (`.specify/memory/constitution.md` has your project principles)
- [ ] Feature branch created by `/speckit.specify` (e.g., `001-payment-checkout`)
- [ ] `specs/001-payment-checkout/spec.md` exists with structured content
- [ ] **No `[NEEDS CLARIFICATION]` markers remain** (all resolved via `/speckit.clarify`)
- [ ] Spec has at least 4 user scenarios (Given/When/Then)
- [ ] Spec has numbered requirements (FR-001, FR-002, etc.) - all specific and testable
- [ ] Quality checklist generated via `/speckit.checklist`
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

3. **Uncertainty surfacing**: The AI marked `[NEEDS CLARIFICATION]` instead of guessing. How does this change your trust in AI-generated code?

4. **Confidence**: Rate your demo confidence now vs. contrast exercise. What made the difference?

---

## Common Mistakes

| Mistake | What Happens Thursday Night |
|---------|----------------------------|
| Skipping `specify init .` | AI doesn't follow your patterns |
| Skipping `/speckit.constitution` | No governance, inconsistent decisions |
| Over-structuring prompts | Wasted time - just describe naturally |
| **Ignoring `[NEEDS CLARIFICATION]` markers** | AI guessed wrong, you're rewriting code |
| **Not running `/speckit.clarify`** | Ambiguities become Thursday bugs |
| Skipping `/speckit.checklist` | Missed edge cases surface in demo |
| Skipping spec entirely | You ARE the contrast exercise |

---

## What's Next?

In **Lab 1.2**, you'll use `/speckit.research` to make technology decisions (FastAPI vs Flask? Redis vs in-memory?) with documented trade-offs.

**The spec says WHAT. Research says HOW. Plan commits to WHICH.**

**Remember**: You're building toward Thursday. Every decision documented now is rework prevented later.
