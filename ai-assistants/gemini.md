---
title: Gemini CLI
layout: default
parent: AI Assistants
nav_order: 3
---
# Gemini CLI Tips for SDD Labs

This guide helps you get the most out of Gemini CLI during the spec-driven development labs.

---

## Setup

### 1. Get API Key

1. Go to https://aistudio.google.com/apikey
2. Sign in with your Google account
3. Click "Create API Key"
4. Copy the key (starts with `AIza...`)

### 2. Install Gemini CLI

```bash
# Using npm (Node.js 18+ required)
npm install -g @anthropics/gemini-cli

# Or download from GitHub releases:
# https://github.com/google-gemini/gemini-cli/releases
```

### 3. Configure

```bash
# Set API key for current session
export GEMINI_API_KEY="your-api-key-here"

# Or add to shell profile for persistence
echo 'export GEMINI_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc
```

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `gemini` | Start Gemini CLI |
| `/help` | Show available commands |
| `/clear` | Clear conversation context |
| `Ctrl+C` | Cancel current operation |

---

## Using Gemini CLI in SDD Workflow

### 1. Spec Creation with /speckit.specify

When running the `/speckit.specify` command:

- Gemini reads workspace context automatically
- Provide detailed requirements â€” Gemini benefits from explicit structure
- Review and refine generated specs

**Tip**: Structure your prompts with clear sections. Gemini responds well to organized input.

### 2. Implementation with /speckit.implement

During implementation:

- Reference files explicitly: "Based on `contracts/order-service.yaml`..."
- Break complex tasks into steps
- Use follow-up prompts to refine output

**Example prompt**:
```
Step 1: Read contracts/order-service.yaml
Step 2: Implement the OrderService class following that contract
Step 3: Create matching pytest tests
```

---

## Gemini CLI Best Practices

### Effective Prompts for SDD

**Good**:
```
Implement the POST /checkout endpoint based on contracts/checkout-api.yaml
```

**Better**:
```
Task: Implement POST /checkout endpoint

Requirements:
- Follow contracts/checkout-api.yaml exactly
- Use Pydantic models from data-model.md
- Implement error handling from error-handling.md
- Add structured logging per logging-strategy.md

Output:
- src/routes/checkout.py
- tests/test_checkout.py
```

### Structured Prompting

Gemini excels with structured prompts:
```
## Context
We're building an e-commerce checkout system using FastAPI.

## Specification
See spec.md for full requirements.

## Task
Implement the payment validation logic.

## Constraints
- Must support Stripe and PayPal
- Must validate before charging
- Must log all attempts
```

---

## Common Issues & Solutions

### Issue: Output too verbose

**Solution**: Request specific format
```
Implement the function. Output ONLY the code, no explanations.
```

### Issue: Gemini doesn't follow spec patterns

**Solution**: Show examples
```
Following this pattern from our codebase:
[paste example]

Implement the checkout endpoint the same way.
```

### Issue: Incomplete implementation

**Solution**: Ask for completion
```
Continue implementing. Still need:
- Error handling
- Input validation  
- Tests
```

---

## Lab-Specific Tips

### Lab 1.1: Init and Spec

When running `specify init .`:
1. Select "Gemini CLI" when prompted
2. The generated `.gemini/instructions.md` will contain SDD context
3. Gemini will now understand your spec-driven workflow

### Lab 1.2: Research

Ask Gemini to analyze options:
```
Research the best approach for payment processing given our constraints in spec.md.
Compare Stripe vs PayPal vs Square.
Document in research.md format.
```

### Lab 1.3: Planning

Validate architecture:
```
Review plan.md and spec.md.

Questions:
1. Does the architecture support all requirements?
2. Are there any missing components?
3. What risks should we document?
```

### Lab 1.4-1.6: Implementation

Break implementation into clear phases:
```
Phase 1: Create the Pydantic models for checkout
Phase 2: Implement the service layer
Phase 3: Create the API endpoint
Phase 4: Write tests

Start with Phase 1.
```

---

## Productivity Tips

1. **Numbered steps**: Gemini tracks multi-step instructions well
2. **Examples first**: Show a pattern, then ask for similar code
3. **Explicit format**: "Output as markdown" or "Output as Python code only"
4. **Iterative refinement**: Build up complexity gradually

---

## Debugging with Gemini CLI

When tests fail:
```
Test failure:
```
[paste error]
```

File: src/checkout.py
Spec: spec.md

Diagnose and fix the issue.
```

When implementation diverges from spec:
```
Compare these:
1. Spec requirements (spec.md section 3)
2. Current implementation (src/checkout.py)

List all discrepancies.
```

---

## Gemini CLI Strengths for SDD

1. **Structured reasoning** â€” Handles complex, multi-part prompts well
2. **Comparison tasks** â€” Excellent at comparing spec vs implementation
3. **Research** â€” Strong at analyzing options and trade-offs
4. **Long context** â€” Can process large specifications effectively

---

## Further Resources

- [Gemini CLI Documentation](https://github.com/google-gemini/gemini-cli)
- [Google AI Studio](https://aistudio.google.com/)
- [Gemini API Documentation](https://ai.google.dev/docs)
