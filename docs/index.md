---
title: Home
layout: home
nav_order: 1
description: "Mastering Spec-Driven Development - Learn to build production-ready software with AI assistants"
permalink: /
---

# Mastering Spec-Driven Development
{: .fs-9 }

Build production-ready software with AI assistants using specification-first methodology.
{: .fs-6 .fw-300 }

Stop hoping AI understands you. Start giving it what it needs to succeed.
{: .fs-5 .fw-300 }

[Get Started](prerequisites.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

## Why Spec-Driven Development?

AI coding assistants are powerful — but most teams struggle to get consistent results. Sound familiar?

| The Pain | What Happens |
|:---------|:-------------|
| **"It works... differently every time"** | Same prompt, different implementations. Inconsistent architecture, naming, and patterns. |
| **"AI keeps going off the rails"** | It builds features you didn't ask for, ignores constraints, or chooses wrong tech. |
| **"We spend more time fixing than building"** | AI-generated code requires constant rework because requirements were unclear. |
| **"I can't explain what I want"** | Translating business needs to technical specs is hard — AI amplifies ambiguity. |
| **"Security and compliance are afterthoughts"** | Without explicit constraints, AI takes shortcuts that fail audits. |

**The root cause:** We ask AI to implement vague ideas instead of validated specifications.

---

## What is Spec-Driven Development?

Spec-Driven Development makes the thinking explicit through **human validation at every phase**. You clarify intent *before* AI writes code.

![Spec-Driven Development Workflow](assets/images/sdd-workflow.svg)

**What makes this different:** The spec is not just documentation — it is a **conversation** that surfaces hidden assumptions before they become bugs.

---

## From Vague to Validated

The magic of SDD is the **clarification process**. AI does not guess -- you decide.

| You Say | AI Asks | You Clarify | Spec Captures |
|:--------|:--------|:------------|:--------------|
| "Build a payment system" | "What payment methods?" | "Credit cards via Stripe" | Stripe tokenization, no raw PAN storage |
| | "What happens on timeout?" | "Retry 3x, then fail gracefully" | 30s timeout, exponential backoff, user notification |
| | "What's out of scope?" | "No refunds yet" | Explicit boundary prevents scope creep |
| | "How do we know it works?" | "Charge goes through, get transaction ID" | Testable acceptance criteria |

**Result:** A spec that any AI assistant can implement consistently.

---

## Your Learning Path

| Part | Focus | What You'll Build |
|:-----|:------|:------------------|
| [**Part 1: Building from Scratch**](labs/part1/) | Greenfield development | Working checkout system from empty repo |
| [**Part 2: Transforming Legacy Code**](labs/part2/) | Brownfield modernization | *(Coming Soon)* Extract specs from existing code |

### Part 1: This Week's Journey

| Phase | What Happens | The Shift |
|:------|:-------------|:----------|
| **Contrast** | Try AI coding without specs | Feel the frustration of inconsistent output |
| **Specify** | Write your first real spec | Experience how questions clarify intent |
| **Plan** | Make technical decisions explicit | See how constraints guide AI choices |
| **Implement** | AI generates compliant code | Watch the spec translate to working software |
| **Integrate** | Add a second feature | Prove the approach scales |
| **Ship** | Production-ready artifact | Demo something you'd actually deploy |

By Friday, you'll have a working checkout system and a repeatable process.

{: .important }
> **Complete [Prerequisites](prerequisites.html) before Day 1.** You'll need your environment ready.

[Start Part 1](labs/part1/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View Resources](/resources/){: .btn .fs-5 .mb-4 .mb-md-0 }