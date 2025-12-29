# Spec-Driven Development Foundations Course 1 — Project Specification

**Version**: 1.0
**Last Updated**: December 29, 2025
**Status**: Production

---

## Executive Summary

### Product Vision

A 2-day professional workshop teaching developers to build production-ready software using AI coding assistants through specification-first methodology. Learners transform vague requirements into deployable code using spec-kit, GitHub's open-source specification framework.

### Target Outcome

By end of course, participants can:
- Write governance-aware specifications from vague requirements
- Use spec-kit commands to drive AI code generation
- Build a complete checkout system (payment + orders)
- Demo production-ready software with confidence

### Business Model

| Element | Details |
|---------|---------|
| Price Point | $3,000 per participant |
| Format | 2-day intensive (6-8 hours) |
| Delivery | Self-paced online via GitHub Pages |
| Target Audience | Professional developers (2+ years experience) |
| Differentiator | Specification-first AI development methodology |

---

## Business Specification

### Problem Statement

AI coding assistants produce inconsistent, buggy code when given vague prompts:
- 1.7x more bugs than human-written code
- 2.74x more security vulnerabilities
- 23% more production incidents year-over-year
- 39% perception gap (developers think they're faster but aren't)

**Root cause**: Input quality, not AI capability.

### Solution

Teach developers to write structured specifications before AI generates code, transforming AI from "unpredictable creative tool" to "reliable specification execution engine."

### Value Proposition

| Stakeholder | Value |
|-------------|-------|
| **Individual Developer** | Career advancement (spec-first skills command premium), reduced Thursday-night rework |
| **Engineering Manager** | Consistent output across team regardless of individual skill |
| **Enterprise** | Compliance built-in, audit trails automatic, reduced incidents |

### Success Metrics

| Metric | Target |
|--------|--------|
| Course completion rate | >80% |
| Lab success rate | >90% complete all labs |
| NPS score | >50 |
| Time to completion | <8 hours over 2 days |
| Apply-at-work rate | >60% use within 2 weeks |

### Competitive Landscape

| Competitor | Positioning | Our Differentiation |
|------------|-------------|---------------------|
| Tessl | Enterprise platform ($125M funding) | We teach methodology, not sell platform |
| Cursor/Windsurf | IDE tools | Tool-agnostic principles |
| Generic AI courses | Prompt engineering | Spec-first, not prompt-first |
| Corporate training | Expensive, slow | Self-paced, immediate application |

---

## Technical Specification

### Platform Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Pages (Static Site)                    │
│                 https://datastoics.github.io/sdd-course-guide    │
├─────────────────────────────────────────────────────────────────┤
│                         Jekyll Engine                            │
│                    Theme: just-the-docs                          │
├─────────────────────────────────────────────────────────────────┤
│  docs/                                                           │
│  ├── index.md              (Home)                                │
│  ├── methodology.md        (SDD concepts)                        │
│  ├── setup.md              (Pre-course setup)                    │
│  ├── business-case.md      (ROI, career impact)                  │
│  ├── whats-next.md         (Post-course)                         │
│  ├── labs/                                                       │
│  │   ├── index.md          (Labs overview)                       │
│  │   ├── lab-0-*.md        (Contrast exercise)                   │
│  │   └── lab-1.1-1.6.md    (Core labs)                          │
│  └── assets/images/        (SVG diagrams)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Content Inventory

| Page | Nav Order | Purpose | Word Count |
|------|-----------|---------|------------|
| index.md | 1 | Course overview, value prop | ~500 |
| methodology.md | 2 | SDD concepts, tool landscape, best practices | ~3,000 |
| setup.md | 3 | Environment setup, AI assistant config | ~2,400 |
| labs/index.md | 4 | Lab overview, schedule | ~450 |
| business-case.md | 5 | ROI calculator, career impact, enterprise value | ~2,200 |
| whats-next.md | 6 | Accomplishments, Course 2, resources | ~900 |

### Lab Structure

| Lab | Duration | Learning Objective | Deliverable |
|-----|----------|-------------------|-------------|
| Lab 0 | 45 min | Experience AI without specs | Broken/incomplete code |
| Lab 1.1 | 90 min | Initialize spec-kit, write first spec | `spec.md` with no ambiguity markers |
| Lab 1.2 | 75 min | Technical planning, research | `plan.md` with technology decisions |
| Lab 1.3 | 60 min | Implementation from spec | Working payment endpoint |
| Lab 1.4 | 75 min | Second feature specification | Order service spec |
| Lab 1.5 | 75 min | Cross-feature integration | Checkout flow working |
| Lab 1.6 | 75 min | Production readiness | Docker container, CI/CD |

**Total**: ~8.25 hours (fits 2-day intensive)

### Visual Assets

| Asset | Purpose | Format |
|-------|---------|--------|
| sdd-workflow.svg | 8-phase SDD workflow diagram | Static SVG |
| sdd-enterprise-integrations.svg | MCP integration points | Static SVG |
| lab-1.1-progress.svg | Progress tracker Lab 1.1 | Static SVG |
| lab-1.2-progress.svg | Progress tracker Lab 1.2 | Static SVG |
| lab-1.3-progress.svg | Progress tracker Lab 1.3 | Static SVG |
| lab-1.4-progress.svg | Progress tracker Lab 1.4 | Static SVG |
| lab-1.5-progress.svg | Progress tracker Lab 1.5 | Static SVG |
| lab-1.6-progress.svg | Progress tracker Lab 1.6 | Static SVG |

**Design standard**: Blue (#2563eb) = current, Green (#22c55e) = completed, Gray (#f3f4f6) = not started

### Development Environment

| Component | Specification |
|-----------|---------------|
| **Primary** | GitHub Codespaces (cloud) |
| **Alternative** | Local Docker + VS Code |
| **Template repo** | DataStoics/sdd-greenfield-starter |
| **Python version** | 3.12.x |
| **Pre-installed** | Gemini CLI, Node.js, spec-kit |

### Supported AI Assistants

| Assistant | Interface | Cost | Recommendation |
|-----------|-----------|------|----------------|
| Gemini CLI | Terminal | Free | Primary (course default) |
| GitHub Copilot | VS Code/Terminal | $10/mo | Alternative |
| Claude Code | Terminal | $20/mo | Alternative |

### spec-kit Integration

Commands used in course:

| Command | Lab Introduced | Purpose |
|---------|----------------|---------|
| `specify init .` | 1.1 | Initialize project structure |
| `/speckit.constitution` | 1.1 | Define project principles |
| `/speckit.specify` | 1.1 | Generate spec from description |
| `/speckit.clarify` | 1.1 | Resolve ambiguities |
| `/speckit.plan` | 1.2 | Generate technical plan |
| `/speckit.implement` | 1.3 | Generate code from spec |
| `/speckit.checklist` | 1.4 | Validate completeness |
| `/speckit.tasks` | 1.5 | Generate task breakdown |
| `/speckit.analyze` | 1.6 | Analyze implementation |

---

## Content Design Principles

### Pedagogical Approach

1. **Contrast-first**: Show the problem (Lab 0) before the solution (Labs 1.x)
2. **Natural language prompts**: Labs describe WHAT to ask AI, not prescriptive code
3. **Human verification**: Every AI step followed by verification checklist
4. **Scenario-based**: Friday investor demo provides authentic pressure
5. **Cumulative**: Each lab builds on previous deliverables

### Writing Standards

| Element | Standard |
|---------|----------|
| Titles | Expanded form (no abbreviations except established terms) |
| Time estimates | Every lab shows duration |
| Success criteria | Checkboxes showing "done" state |
| Code blocks | Minimal—prefer natural language prompts |
| Tables | Used for comparisons, checklists, options |
| Callouts | warning (red), note (blue), tip (green), important (yellow) |

### Encoding Requirements

- **File encoding**: UTF-8 WITHOUT BOM
- **Line endings**: LF (Git handles CRLF conversion)
- **Diagrams**: Static SVG only (no Mermaid—rendering issues)

---

## Governance and Compliance

### Content Governance

| Constraint | Rationale |
|------------|-----------|
| No hardcoded code in labs | AI should generate; humans verify |
| All external links must be current | Outdated links damage credibility |
| No vendor lock-in | Tool-agnostic methodology |
| Explicit prerequisites | 2+ years dev experience |

### Quality Checklist

Before release:
- [ ] All labs have SVG progress diagrams
- [ ] No `[NEEDS CLARIFICATION]` markers in content
- [ ] All external links verified
- [ ] Time estimates accurate
- [ ] Success criteria testable
- [ ] Jekyll builds without errors
- [ ] Mobile-responsive layout verified

---

## Future Roadmap

### Course 2: Brownfield Legacy Flow (Planned)

| Module | Skill |
|--------|-------|
| Reverse Specification | Extract specs from existing code |
| Strangler Fig Pattern | Incremental legacy replacement |
| Compliance Remediation | Fix governance gaps |
| MCP Integration | Connect to Jira, Confluence, GitHub |

### Potential Enhancements

| Enhancement | Priority | Effort |
|-------------|----------|--------|
| Demo videos (3-5 min each) | High | Medium |
| Interactive progress tracking | Medium | High |
| Assessment/quiz system | Low | Medium |
| Multi-language support | Low | High |

---

## Repository Structure

```
sdd-course-guide/
├── docs/
│   ├── _config.yml              # Jekyll configuration
│   ├── index.md                 # Home page
│   ├── methodology.md           # SDD methodology
│   ├── setup.md                 # Pre-course setup
│   ├── business-case.md         # ROI and career impact
│   ├── whats-next.md            # Post-course resources
│   ├── labs/
│   │   ├── index.md             # Labs overview
│   │   ├── lab-0-ai-without-specs.md
│   │   ├── lab-1.1-init-and-spec.md
│   │   ├── lab-1.2-plan.md
│   │   ├── lab-1.3-implementation.md
│   │   ├── lab-1.4-order-spec.md
│   │   ├── lab-1.5-integration.md
│   │   └── lab-1.6-production.md
│   └── assets/
│       └── images/
│           ├── sdd-workflow.svg
│           ├── sdd-enterprise-integrations.svg
│           └── lab-1.x-progress.svg (6 files)
├── PROJECT-SPEC.md              # This document
└── README.md                    # Repository readme
```

---

## Deployment

| Environment | URL | Branch |
|-------------|-----|--------|
| Production | https://datastoics.github.io/sdd-course-guide | main |
| Starter Template | https://github.com/DataStoics/sdd-greenfield-starter | main |

**Deployment method**: GitHub Pages with Jekyll, automatic on push to main.

---

## Contact

| Role | Contact |
|------|---------|
| Training inquiries | training@datastoics.io |
| Enterprise sales | enterprise@datastoics.io |
| Course 2 interest | training@datastoics.io (subject: "Course 2 Interest") |

---

*This specification serves as the authoritative reference for the Spec-Driven Development Foundations Course 1 project.*
