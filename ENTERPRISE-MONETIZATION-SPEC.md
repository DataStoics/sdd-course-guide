# Enterprise SDD Course Monetization Specification

**Version**: 1.0
**Last Updated**: December 29, 2025
**Status**: Production Ready

---

## Executive Summary

### Product Vision

Transform public GitHub Pages SDD course into enterprise-grade private training platform with one-time cohort licensing ($3k/participant), zero subscription costs, and seamless GitHub-native delivery (Codespaces, SVGs, spec-kit labs preserved).

### Target Outcome

Enterprises purchase bulk licenses (10-50 seats/cohort), receive automated private repo access post-Stripe payment. Course completion tracked via GitHub PRs/forks. Revenue model: $30k-150k per cohort deployment.

### Business Model

| Element | Details |
|---------|---------|
| Price Point | $3,000/participant, volume discounts (20+ = $2,500) |
| Access Model | Private GitHub repo + Codespaces (6mo license) |
| Delivery | Self-paced + optional cohort facilitation |
| Target Audience | Enterprise engineering managers, L&D teams |
| Payment Platform | Lemoon/Stripe (8.5% fee, no upfront costs) |

---

## Business Specification

### Problem Statement

Public GitHub Pages exposes $3k course content freely, preventing enterprise monetization. Current spec lacks payment gating, license management, and cohort tracking required for B2B sales.

**Root cause**: No private access + payment automation workflow.

### Solution

**Stripe → GitHub Access Pipeline**: Public teaser → Lemoon checkout → automated repo invite → enterprise cohort consumes via private Codespaces/forks.

### Value Proposition

| Stakeholder | Value |
|-------------|-------|
| **Enterprise Buyer** | Bulk licensing, GitHub-native delivery, measurable ROI via PR completion rates |
| **Engineering Manager** | Team-wide SDD upskilling, no tool adoption (uses existing GitHub) |
| **Individual Learner** | Production-grade labs, career certification via PR badges |
| **DataStoics** | Zero platform costs, 91.5% revenue retention, scales to Course 2 |

### Success Metrics

| Metric | Target |
|--------|--------|
| Conversion rate (teaser → purchase) | >15% |
| Average cohort size | 20+ participants |
| License renewal rate | >60% at 6mo |
| Payment processing time | <5min end-to-end |
| Enterprise NPS | >70 |

---

## Technical Specification

### Architecture

```
┌─────────────────┐    ┌──────────────┐    ┌──────────────────┐
│   Public Teaser │───▶│   Lemoon     │───▶│ Automated Invite │
│ GitHub Pages    │    │ Checkout     │    │   (Zapier)       │
│ datastoics.com  │    │ $3k/seat     │    │                  │
└─────────────────┘    └──────────────┘    └──────────────────┘
                                    │
                                    ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Private Repo     │◄───│ Enterprise Team  │───▶│ Codespaces Labs  │
│ sdd-course-pro   │    │ Forks/PRs        │    │ spec-kit + SVGs  │
│ (Jekyll+Codespace)│    │ Completion       │    │                  │
└──────────────────┘    └──────────────────┘    └──────────────────┘
```

### Content Migration Plan

| Current | Target | Migration Step |
|---------|--------|----------------|
| `sdd-course-guide` (public) | `sdd-course-pro` (private) | Fork → make private → update Pages source |
| Codespaces config | Identical | Copy `.devcontainer/` |
| Progress SVGs | Dynamic via Actions | Add `progress.yml` workflow |
| Starter repo | Remains public | `sdd-greenfield-starter` |

### Payment Integration

**Lemoon Product Setup**:

```
Product: "SDD Foundations Enterprise License"
Price: $3,000/seat
Quantity: 1-100
Post-purchase webhook → Zapier → GitHub API
```

**Zapier Flow** (15min setup):

1. Lemoon "Payment Success" trigger
2. Parse buyer email + seats purchased
3. GitHub API: Add buyer as repo collaborator (Outside Collaborator role)
4. Email: "Access granted: https://github.com/DataStoics/sdd-course-pro"
5. Optional: Create GitHub Project for cohort tracking

### License Management

| Duration | Enforcement | Renewal |
|----------|-------------|---------|
| 6 months | Manual repo cleanup | Automated email @5mo |
| Enterprise | Custom contract | Invoice → extend access |

---

## Implementation Roadmap

### Phase 1: MVP (Day 1, 2 hours)

| Step | Action | Time |
|------|--------|------|
| 1 | Fork `sdd-course-guide` → `sdd-course-pro` (private) | 5min |
| 2 | Create Lemoon product ($3k/seat) | 10min |
| 3 | Setup Zapier: Lemoon → GitHub invite | 30min |
| 4 | Deploy private Pages (Enterprise Cloud org) | 15min |
| 5 | Create public teaser: `datastoics.com/sdd-enterprise` | 30min |
| 6 | Test end-to-end payment flow | 30min |

### Phase 2: Enterprise Polish (Day 2, 1 hour)

| Feature | Implementation |
|---------|----------------|
| Cohort dashboards | GitHub Projects per buyer org |
| Volume discounts | Lemoon tiered pricing |
| Invoice generation | Lemoon PDF export |
| Renewal reminders | Zapier scheduled emails |

### Phase 3: Scale (Week 2)

| Feature | Value |
|---------|-------|
| Custom enterprise forks | White-label per customer |
| Jira/MCP integration specs | Course 2 prep |
| Certification badges | PR completion → GitHub Skills |

---

## Governance and Compliance

### Enterprise Requirements

| Constraint | Implementation |
|------------|----------------|
| GDPR compliance | EU Stripe processing, data deletion |
| SOC2 alignment | GitHub Enterprise Cloud |
| Audit trail | All access via GitHub collab logs |
| Contract terms | 6mo license, no resale |

### Security Checklist

- [ ] Private repo: Settings > Danger Zone > Make private
- [ ] Pages: Enterprise Cloud only (private access)
- [ ] Codespaces: Org policy enforcement
- [ ] Zapier: GitHub Personal Access Token (repo scope only)
- [ ] Lemoon: EU data residency

---

## Deployment Specification

| Environment | URL | Access |
|-------------|-----|--------|
| **Teaser (Public)** | `datastoics.com/sdd-enterprise` | Everyone |
| **Course (Private)** | `sdd-course-pro.data-stoics.pages.dev` | Paid collaborators |
| **Starter Template** | `github.com/DataStoics/sdd-greenfield-starter` | Public |
| **Payment** | `lemoon.io/datastoics/sdd-enterprise` | Checkout only |

**Deployment Command**:

```bash
# After fork/private setup
git push origin main  # Triggers private Pages build
```

---

## Success Criteria

| Metric | Acceptance Test |
|--------|-----------------|
| Payment-to-access | <5min from Stripe to repo invite |
| Codespaces launch | <30s from repo clone |
| Lab progression | SVG updates on PR merge |
| Enterprise bulk | 20 seats → 20 collab invites |

---

## Contact and Support

| Role | Contact |
|------|---------|
| Enterprise Sales | `enterprise@datastoics.io` |
| Technical Support | `support@datastoics.io` |
| License Management | `billing@datastoics.io` |
| Course Feedback | GitHub Discussions (private repo) |

---

## References

- [Lemoon GitHub Monetization Guide](https://dev.to/abubakersiddique761/put-paid-access-to-your-github-code-without-writing-a-line-of-code-4p0c)

---

*This specification operationalizes enterprise SDD monetization with zero subscription costs, preserving GitHub-native delivery while enabling $30k+ cohort revenue.*
