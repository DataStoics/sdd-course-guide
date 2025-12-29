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

## Why Specs Matter (What You Saw in Lab 0)

Remember Lab 0? You asked your AI to build a checkout feature using natural language - no specs, no structure. The result:

- Code that works in isolation but breaks when integrated
- Missing edge cases (double-clicks, error handling)
- No way to answer "Is this secure?" confidently
- Thursday night energy spent on rework, not polish

**This lab shows you what happens when you spec first.**

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

**Select the one you're using** (the same AI assistant you used in Lab 0).

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

## Step 5: Expand Scenarios with /speckit.clarify (15 min)

Remember - `/speckit.specify` already generated User Scenarios in your spec. Now let's **expand them for demo coverage** using natural language:

```
/speckit.clarify I need scenarios that will impress investors. 
What happens if someone double-clicks the Pay button? 
What if payment fails - do we show ugly errors? 
Can we prove the system is auditable?
```

The AI analyzes your concerns and updates the spec with additional scenarios:

```
## Scenario Coverage Analysis

Your spec currently has 1 scenario (happy path). For investor demo, I recommend:

**Adding Scenario 2: Double-Click Protection**
Would you like me to add a scenario where a duplicate payment attempt returns the original confirmation?

| Option | Answer | Demo Value |
|--------|--------|------------|
| A (Recommended) | Yes, add idempotency scenario | Shows robustness, prevents demo embarrassment |
| B | No, skip for MVP | Risk: demo could show duplicate charge |

**Your choice**: _[Type A or B]_
```

### The Conversation Expands Your Spec

As you answer, the AI adds scenarios to your spec:

- **Q1: Double-click?** → A (Yes, add idempotency scenario)
- **Q2: Error handling?** → A (Yes, show graceful recovery)
- **Q3: Audit trail?** → A (Yes, prove enterprise-readiness)

### Review Your Expanded Scenarios

Open `spec.md` and you'll see the AI generated these scenarios **in proper Given/When/Then format**:

```markdown
## User Scenarios

### Scenario 1: Successful Payment (Priority: P1)
**Given** a customer with a valid payment token,
**When** they submit a payment for $50.00,
**Then** payment succeeds and they see confirmation with transaction ID.

*Demo moment: "Here's a customer completing checkout..."*

### Scenario 2: Double-Click Protection (Priority: P1)
**Given** a payment was just processed,
**When** the customer clicks Pay again,
**Then** they see the same confirmation (not a duplicate charge).

*Demo moment: "Watch what happens if I click Pay twice..."*

### Scenario 3: Graceful Error Recovery (Priority: P2)
**Given** a customer submits an invalid payment token,
**When** the payment is attempted,
**Then** they see a helpful error message and can retry.

*Demo moment: "And if something goes wrong, users get a clear message..."*

### Scenario 4: Audit Trail (Priority: P2)
**Given** a completed transaction,
**When** an auditor requests the trace,
**Then** system shows: timestamp, customer, amount, outcome, trace ID.

*Demo moment: "Every transaction is fully auditable from day one."*
```

**Key insight**: You described what you needed in plain English. The AI structured it into testable scenarios. No manual Given/When/Then writing required.

---

## Step 6: Generate Quality Checklist with /speckit.checklist (5 min)

**Why checklists?** Your spec captures requirements, but checklists catch *gaps* - the things you forgot to specify that will bite you Thursday night.

Run:

```
/speckit.checklist demo-readiness
```

The AI reads YOUR spec and generates questions specific to your feature:

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

**How to use it**: Review each item. If you can't check it off, go back to your spec and add the missing detail. Unchecked items = Thursday night rework.

**Different focus areas**:
- `/speckit.checklist security` - authentication, authorization, data protection
- `/speckit.checklist ux` - user flows, error messages, edge cases
- `/speckit.checklist testing` - testability, acceptance criteria coverage

---

## Step 7: Commit Your Work (5 min)

**Where does this commit go?** When you created a Codespace from the template, GitHub made YOUR OWN COPY of the repository under your account. Your commits go to YOUR repo, not the original template.

Commit using conventional format:

```bash
git add .
git commit -m "feat: payment checkout specification for investor demo"
```

This commit includes:
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

---

## AI Without Specs vs. AI With Specs

| Aspect | AI Without Specs (Lab 0) | AI With Specs (Lab 1.1) |
|--------|--------------------------|-------------------------|
| Time spent | 30 min coding | 30 min spec, then coding |
| Double-click | Causes duplicate charge | Returns original response |
| Bad input | Crash or ugly error | Helpful error message |
| "Is this secure?" | "Uh... I think so?" | "Yes, here's the spec" |
| Thursday night | Reworking everything | Polishing details |
| Friday demo | Hoping nothing breaks | Confident walkthrough |

**Same effort, different outcome.**

---

## Key Takeaways

1. **Specs are conversations, not documents** - You described what you needed naturally; the AI structured it.

2. **`[NEEDS CLARIFICATION]` is a feature** - The AI asking instead of guessing prevents Thursday rewrites.

3. **Checklists catch what specs miss** - Requirements say what to build; checklists ask "did you forget anything?"

4. **30 minutes now saves hours later** - Every ambiguity resolved in spec is a bug that won't appear Thursday night.

---

## What's Next?

In **Lab 1.2**, you'll use `/speckit.plan` to make technology decisions (FastAPI vs Flask? Redis vs in-memory?) with documented trade-offs.

**The spec says WHAT. The plan says HOW.**

**Remember**: You're building toward Thursday. Every decision documented now is rework prevented later.
