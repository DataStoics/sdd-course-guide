---
title: What's Next
layout: default
nav_order: 6
permalink: /whats-next/
---

# What's Next

You've completed Course 1. You can now build compliant software from scratch using SDD.

---

## Course 2: Brownfield Legacy Flow

Most real-world work isn't greenfield. You inherit legacy systems with implicit requirements, undocumented business logic, and technical debt.

| Course 1 (Greenfield) | Course 2 (Brownfield) |
|:----------------------|:----------------------|
| Empty repository | 5,000+ lines of existing code |
| You write the spec | You extract the spec |
| Full control | Constraints everywhere |
| Build compliance in | Retrofit compliance |

### What You'll Learn

1. **Reverse Specification** - Extract implicit specs from working code
2. **Strangler Fig Pattern** - Incrementally replace legacy with SDD-compliant code
3. **Compliance Remediation** - Find and fix governance gaps in existing systems
4. **Risk-Based Refactoring** - Prioritize what to fix based on business impact

Interested? Contact us at [training@datastoics.io](mailto:training@datastoics.io).

---

## Optional Reading: AI Coding Best Practices

### From the Source

| Resource | Why Read It |
|:---------|:------------|
| [Claude Code: Best Practices for Agentic Coding](https://www.anthropic.com/engineering/claude-code-best-practices) | Anthropic's official guide to working with Claude. Covers explore-plan-code-commit workflow, CLAUDE.md files, and when to use subagents. |
| [OpenAI Prompt Engineering Guide](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api) | Foundational techniques: be specific, show examples, use leading words. Still relevant for all models. |
| [Google Cloud: Five Best Practices for AI Coding Assistants](https://cloud.google.com/blog/topics/developers-practitioners/five-best-practices-for-using-ai-coding-assistants) | Train the tool, make a plan, connect context between sessions. Practical enterprise advice. |

### Community Guides

| Resource | Why Read It |
|:---------|:------------|
| [Prompt Engineering for Coding Assistants](https://stoltzstack.com/blog/prompt-engineering-coding-assistants) | Developer-focused guide covering the "follow the pattern" and "persona" techniques. Good mental models. |
| [12 Prompt Engineering Tips for Claude](https://www.vellum.ai/blog/prompt-engineering-tips-for-claude) | Claude-specific tips: use XML tags, let Claude say "I don't know", break complex tasks into steps. |

---

## Optional Reading: SDD Best Practices

### Industry Perspectives

| Resource | Why Read It |
|:---------|:------------|
| [Martin Fowler: Exploring SDD Tools](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) | Thoughtworks analysis comparing Kiro, Spec-kit, and Tessl. Defines spec-first vs spec-anchored vs spec-as-source. |
| [ThoughtWorks: What Is A Spec?](https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices) | Defines what makes a good spec. Connects SDD to context engineering and BDD heritage. |
| [SDD Best Practices](https://intent-driven.dev/knowledge/best-practices/) | Practical implementation guide: prioritize human reviewability, start minimal, decompose meaningfully, prevent spec drift. |

### Tools and Frameworks

| Resource | Why Read It |
|:---------|:------------|
| [GitHub Spec Kit](https://github.com/github/spec-kit) | Official GitHub toolkit for SDD. Constitution files, slash commands (/specify, /plan, /tasks), templates. |
| [Microsoft: Diving Into Spec-Driven Development](https://developer.microsoft.com/blog/spec-driven-development-spec-kit) | Microsoft's introduction to Spec Kit. Explains the workflow and why detailed first prompts matter. |
| [nvisia: Building Better Software with AI-First Architecture](https://www.nvisia.com/insights/spec-driven-development-building-better-software-with-ai-first-architecture) | Enterprise perspective on SDD. Includes practical lessons from a fraud detection proof-of-concept. |

---

## Optional Reading: The Future of AI Coding

### Where Things Are Heading

| Resource | Why Read It |
|:---------|:------------|
| [GitLab: Agentic AI Trends Reshaping Software Development](https://about.gitlab.com/the-source/ai/emerging-agentic-ai-trends-reshaping-software-development/) | AI agents as orchestration layer, enterprise guardrails, tackling technical debt at scale. |
| [GitLab: AI Trends for 2025](https://about.gitlab.com/the-source/ai/ai-trends-for-2025-agentic-ai-self-hosted-models-and-more/) | Proactive AI collaborators, on-premises models, AI assistants becoming central hubs. |
| [Svitla: Agentic AI Trends 2025](https://svitla.com/blog/agentic-ai-trends-2025/) | Multi-agent collaboration, tool/API integration, autonomous business workflows. Good technical depth. |

### The Big Picture

The shift from **generative AI** (responds to prompts) to **agentic AI** (plans and executes autonomously) is accelerating. But autonomy without structure produces chaos.

**SDD is how you stay in control.**

Specs become the governance layer that constrains what agents can do. As AI gets more capable, clear specifications become more valuable - not less.

---

## Apply It This Week

The best way to solidify these skills:

1. **Pick a real feature** at work (something small)
2. **Write the spec first** (30-60 minutes)
3. **Have AI implement from the spec**
4. **Compare** the result to your usual AI coding
5. **Note the differences** in quality, rework, and review time

{: .tip }
> Skills stick when you use them immediately. Don't wait for the "perfect" project.

---

## Stay Connected

- [Course Discussion Forum](https://github.com/DataStoics/sdd-course-guide/discussions)
- [Starter Template](https://github.com/DataStoics/sdd-greenfield-starter)
- [Checkpoint Repositories](https://github.com/DataStoics?q=lab-checkpoint)

