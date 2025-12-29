---
title: Claude Code
layout: default
parent: AI Assistants
nav_order: 2
---
# Claude Code Tips for SDD Labs

This guide helps you get the most out of Claude Code during the spec-driven development labs.

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `claude` | Start Claude Code CLI |
| `/help` | Show available commands |
| `/clear` | Clear conversation context |
| `Ctrl+C` | Cancel current operation |

---

## Using Claude Code in SDD Workflow

### 1. Spec Creation with /speckit.specify

When running the `/speckit.specify` command:

- Claude Code reads your entire workspace automatically
- Provide clear, detailed requirements in your prompt
- Review generated artifacts thoroughly â€” Claude generates comprehensive specs

**Tip**: Claude Code maintains conversation context. Build on previous responses rather than re-explaining.

### 2. Implementation with /speckit.implement

During implementation:

- Claude Code automatically reads relevant files
- Reference specific files when needed: "Look at `contracts/order-service.yaml`"
- Use iterative refinement â€” Claude responds well to "adjust X to do Y"

**Example prompt**:
```
Implement the OrderService based on contracts/order-service.yaml. 
Follow the patterns in our existing services.
```

---

## Claude Code Best Practices

### Effective Prompts for SDD

**Good**:
```
Implement the POST /checkout endpoint based on contracts/checkout-api.yaml
```

**Better**:
```
Implement POST /checkout following contracts/checkout-api.yaml.
Make sure to:
- Use Pydantic models from data-model.md
- Follow error handling patterns from error-handling.md  
- Add logging per logging-strategy.md
- Include tests in tests/test_checkout.py
```

### Multi-File Operations

Claude Code excels at coordinated changes:
```
Update the checkout endpoint AND its tests AND the API documentation 
to support a new "gift_wrap" option
```

---

## Common Issues & Solutions

### Issue: Claude makes too many changes

**Solution**: Be specific about scope
```
ONLY modify src/checkout.py. Don't change any other files.
```

### Issue: Claude doesn't follow the spec

**Solution**: Quote the relevant section
```
The spec says: "Payment must be validated before order creation"
Your implementation does order first. Please fix.
```

### Issue: Context seems lost

**Solution**: Use `/clear` and start fresh with full context
```
/clear
```
Then re-explain the task with references to spec files.

---

## Lab-Specific Tips

### Lab 1.1: Init and Spec

When running `specify init .`:
1. Select "Claude Code" when prompted
2. The generated `.claude/instructions.md` will contain SDD context
3. Claude will now understand your spec-driven workflow

### Lab 1.2: Research

Ask Claude to analyze:
```
Read research.md and summarize the key technical decisions. 
Are there any gaps we should address?
```

### Lab 1.3: Planning

Validate your plan:
```
Compare plan.md against spec.md. 
Does the plan address all acceptance criteria?
```

### Lab 1.4-1.6: Implementation

Claude Code is great for end-to-end implementation:
```
Implement the complete payment service following contracts/payment-service.yaml.
Create the model, service, API endpoint, and tests.
```

---

## Productivity Tips

1. **Incremental development**: Ask Claude to implement one component at a time
2. **Review mode**: "Review my implementation against the spec and list any gaps"
3. **Test generation**: "Generate pytest tests for all acceptance criteria in spec.md"
4. **Documentation**: "Update README.md to reflect the current implementation"

---

## Debugging with Claude Code

When tests fail:
```
test_checkout_validation is failing with this error:
[paste error]
Diagnose the issue and fix it.
```

When implementation diverges from spec:
```
Compare src/checkout.py against the acceptance criteria in spec.md.
What's missing or incorrect?
```

---

## Claude Code Strengths for SDD

1. **Context retention** â€” Remembers earlier discussion points
2. **Multi-file edits** â€” Can update related files together
3. **Comprehensive output** â€” Generates complete implementations
4. **Explanation** â€” Excellent at explaining why code is structured a certain way

---

## Further Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [Anthropic Developer Portal](https://console.anthropic.com)
