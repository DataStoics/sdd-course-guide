---
title: GitHub Copilot
layout: default
parent: AI Assistants
nav_order: 1
---
# GitHub Copilot Tips for SDD Labs

This guide helps you get the most out of GitHub Copilot during the spec-driven development labs.

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `Ctrl+I` (Windows) / `Cmd+I` (Mac) | Open Copilot Chat inline |
| `Ctrl+Enter` | Accept suggestion |
| `Tab` | Accept inline completion |
| `Esc` | Dismiss suggestion |

---

## Using Copilot in SDD Workflow

### 1. Spec Creation with /speckit.specify

When running the `/speckit.specify` command:

- **Be specific** â€” Copilot uses your spec as context
- **Review generated artifacts** â€” Check spec.md, research.md
- **Iterate** â€” Refine if something is unclear

**Tip**: Keep the spec visible in a split pane while coding. Copilot references open files.

### 2. Implementation with /speckit.implement

During implementation:

- **Open related spec files** â€” Copilot uses them as context
- **Reference contracts** â€” When generating API code, open the relevant contract file
- **Use @workspace** â€” Query the entire workspace for context

**Example prompt**:
```
@workspace Implement the OrderService based on the spec in contracts/order-service.yaml
```

---

## Copilot Chat Best Practices

### Context Commands

| Command | Use When |
|---------|----------|
| `@workspace` | Need broad codebase context |
| `@file` | Reference specific file |
| `#selection` | Work with selected code |

### Effective Prompts for SDD

**Good**:
```
Based on contracts/checkout-api.yaml, implement the POST /checkout endpoint 
with proper error handling and validation
```

**Better**:
```
@workspace Implement POST /checkout following the contract in contracts/checkout-api.yaml. 
Include:
- Pydantic validation from data-model.md
- Error responses from error-handling.md
- Logging per logging-strategy.md
```

---

## Common Issues & Solutions

### Issue: Copilot ignores spec files

**Solution**: Explicitly reference them
```
@file:specs/001-sdd-foundations-greenfield/spec.md
Generate tests based on the acceptance criteria above
```

### Issue: Generated code doesn't match contract

**Solution**: Open the contract file in editor first, then prompt
```
With checkout-api.yaml open, implement the response schema exactly as specified
```

### Issue: Suggestions are generic, not spec-specific

**Solution**: Use the `.github/copilot-instructions.md` file (created by `specify init .`) to provide persistent context

---

## Lab-Specific Tips

### Lab 1.1: Init and Spec

When running `specify init .`:
1. Select "GitHub Copilot" when prompted
2. The generated `.github/copilot-instructions.md` will contain SDD instructions
3. Copilot will now understand your spec-driven workflow

### Lab 1.2: Research

Use Copilot Chat to explore:
```
@workspace What are the key technical decisions documented in research.md?
```

### Lab 1.3: Planning

Ask Copilot to validate your plan:
```
@workspace Does plan.md align with all requirements in spec.md?
```

### Lab 1.4-1.6: Implementation

Reference specific contracts:
```
@file:contracts/payment-service.yaml Implement this contract with FastAPI
```

---

## Productivity Shortcuts

1. **Multi-file context**: Open all related files, then prompt with `@workspace`
2. **Inline suggestions**: Start typing a function signature and let Copilot complete
3. **Chat history**: Use up arrow to recall previous prompts
4. **Explain code**: Select code, use `Explain this` in Copilot Chat

---

## Debugging with Copilot

When tests fail:
```
@workspace Why is test_checkout_validation failing? 
Here's the error: [paste error]
```

When implementation diverges from spec:
```
@file:spec.md @file:src/checkout.py 
Does the implementation match the acceptance criteria in spec.md?
```

---

## Further Resources

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [VS Code Copilot Extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
- [Copilot Chat Commands](https://docs.github.com/en/copilot/using-github-copilot/asking-github-copilot-questions-in-your-ide)
