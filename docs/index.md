---
title: Home
layout: home
nav_order: 1
description: "SDD Foundations Course 1 - Learn to build production-ready software with AI assistants"
permalink: /
---

# SDD Foundations Course 1
{: .fs-9 }

From vague idea to production code in a structured workflow.
{: .fs-6 .fw-300 }

Stop hoping AI understands you. Start giving it what it needs to succeed.
{: .fs-5 .fw-300 }

[Get Started](setup){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View Labs](labs/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## The SDD Workflow

Most AI coding fails because we skip the thinking. SDD makes the thinking explicit.

\\\mermaid
flowchart LR
    subgraph C["CONSTITUTION"]
        C1["Immutable Rules"]
        C2["Security Standards"]
        C3["Compliance Reqs"]
    end
    
    subgraph S["SPECIFY"]
        S1["Interview & Clarify"]
        S2["What does 'fast' mean?"]
        S3["Edge cases?"]
    end
    
    subgraph P["PLAN"]
        P1["Technical Decisions"]
        P2["Which DB? What libs?"]
        P3["API design?"]
    end
    
    subgraph I["IMPLEMENT"]
        I1["AI Generates Code"]
        I2["Tests pass"]
        I3["Scan clean. Ship it."]
    end
    
    C --> S --> P --> I
\\\

**Each phase has validation.** You don't move forward until the spec is clear.

---

## How Specs Get Better

The magic isn't in "adding lines to a spec." It's in the clarification process.

| What You Start With | What SDD Produces |
|:--------------------|:------------------|
| "Build a payment system" | **Functional**: Process payments via Stripe tokenization |
| | **Constraints**: Never store raw card numbers (PCI DSS) |
| | **Error Handling**: Timeout after 30s, retry 3x, then fail gracefully |
| | **Acceptance**: Given valid card, when charged, then return transaction ID |
| | **Out of Scope**: Refunds, subscriptions, multi-currency |

The AI didn't guess these requirements. **You defined them through structured questions.**

---

## Your Journey This Week

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
> **Complete [Pre-Course Setup](setup) before Day 1.** You'll need your environment ready.

[Start the Labs](labs/){: .btn .btn-outline .fs-5 }

