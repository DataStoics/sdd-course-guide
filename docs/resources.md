---
title: Further Resources
layout: default
nav_order: 99
permalink: /resources/
---

# Further Resources

Curated reading materials to deepen your Spec-Driven Development practice.

---

## Recommended Reading

| Resource | Why Read It |
|:---------|:------------|
| [GitHub spec-kit Repository](https://github.com/github/spec-kit) | Official toolkit — explore advanced features and examples |
| [Anthropic: Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Deep dive on managing AI context effectively |
| [Martin Fowler: Exploring SDD Tools](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) | Compare spec-kit, Kiro, Tessl approaches |
| [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) | If you're using Claude for implementation |
| [GitHub Copilot Documentation](https://docs.github.com/en/copilot) | Official docs for the recommended AI assistant |

---

## Quick Reference

### spec-kit Commands

| Command | Purpose |
|---------|---------|
| `specify init .` | Initialize project |
| `/speckit.constitution` | Define project principles |
| `/speckit.specify` | Generate spec from description |
| `/speckit.clarify` | Resolve ambiguities |
| `/speckit.plan` | Create technical plan |
| `/speckit.implement` | Generate code from spec |
| `/speckit.checklist` | Validate completeness |

### The "Would I Demo This?" Test

Before shipping, ask:
- [ ] Is this secure? (Can I prove it?)
- [ ] What happens on double-click? (Idempotent?)
- [ ] What happens on error? (Graceful?)
- [ ] Where's the audit trail? (Traceable?)
- [ ] Would I confidently demo this to investors?

---

## Research & Industry Reports

| Report | Key Finding |
|:-------|:------------|
| [State of AI Code Generation (CodeRabbit, 2025)](https://www.coderabbit.ai/blog/state-of-ai-code-generation-2025) | AI-generated code has 1.7x more bugs without structured specifications |
| [GitHub Copilot Enterprise Study (2024)](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-in-the-enterprise-with-accenture/) | +15% PR merge rate, 84% more successful builds with AI assistance |
| [Engineering in the Age of AI (Cortex, 2025)](https://www.cortex.io/post/engineering-in-the-age-of-ai-2026-benchmark-report) | Teams with specification discipline show 30% lower change failure rate |

---

**Remember**: The spec is the contract. AI implements it. You validate it.
