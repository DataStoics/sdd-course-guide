---
title: MCP Integration
parent: Reference
nav_order: 3
---
# Integrating External Systems with SDD

> **ðŸ“ Course 2 Preview**: This guide introduces concepts covered in depth in **SDD Foundations Course 2: Brownfield Repo Flow**. In Course 1, these are optional enhancements. In Course 2, they become essential for working with legacy systems.

---

## Why This Matters for Brownfield

In Course 1 (Greenfield), you build from scratch â€” your AI assistant's training data is often sufficient.

In Course 2 (Brownfield), you'll work with:
- **Existing codebases** that need context
- **Internal documentation** (Confluence, wikis, ADRs)
- **Company standards** (approved tech lists, security policies)
- **Legacy APIs** that aren't in public training data

MCP (Model Context Protocol) servers bridge this gap by connecting your AI assistant to live, internal, and current sources.

---

## Course 1 vs Course 2

| Aspect | Course 1 (Greenfield) | Course 2 (Brownfield) |
|--------|----------------------|----------------------|
| Codebase | Empty, you create it | 5000+ lines existing |
| Documentation | You write specs first | Must discover existing patterns |
| Tech decisions | Research options freely | Must align with existing stack |
| External sources | Nice to have | Essential |

---

## MCP Servers You'll Use in Course 2

### For Understanding Legacy Code

| Server | Purpose |
|--------|---------|
| **GitHub/GitLab** | Search existing repos, find patterns |
| **Context7** | Get current docs for legacy dependencies |

### For Aligning with Company Standards

| Server | Purpose |
|--------|---------|
| **Confluence** | Internal wikis, architecture decisions |
| **Custom REST** | Tech radar, approved libraries API |

### For Modernization Research

| Server | Purpose |
|--------|---------|
| **Perplexity** | Migration guides, deprecation notices |
| **Snyk/NVD** | Security vulnerabilities in legacy deps |

---

## Preview: Course 2 Workflow

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ 1. DISCOVER (understand legacy)     â”‚
â”‚    - Connect to existing codebase   â”‚
â”‚    - Query internal documentation   â”‚
â”‚    - Map undocumented behaviors     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
               â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ 2. SPECIFY (create missing specs)   â”‚
â”‚    - Generate specs FROM code       â”‚
â”‚    - Validate against internal docs â”‚
â”‚    - Fill governance gaps           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
               â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ 3. MODERNIZE (guided by specs)      â”‚
â”‚    - Research migration paths       â”‚
â”‚    - Check security implications    â”‚
â”‚    - Align with current standards   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## Quick Setup (Optional for Course 1)

If you want to experiment now, basic MCP config structure:

```json
{
  "mcpServers": {
    "perplexity": { "command": "...", "env": { "API_KEY": "..." } },
    "context7": { "command": "...", "env": { "API_KEY": "..." } }
  }
}
```

- **Claude Code**: `~/.config/claude/mcp_servers.json`
- **VS Code**: Extension-specific settings

See each server's documentation for setup details.

---

## What You'll Learn in Course 2

- Connecting AI assistants to internal knowledge bases
- Generating specs from undocumented legacy code
- Using live research for migration decisions
- Integrating security scanning into the spec workflow
- Building "spec wrappers" for legacy APIs

---

## See Also

- [Course 2 Preview Slides](../../slides/course2-preview.md)
- [Lab 1.6: Production Readiness](../lab-instructions/lab-1.6-production-ready.md) â€” includes Course 2 teaser
- [MCP Specification](https://modelcontextprotocol.io)
