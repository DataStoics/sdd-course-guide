---
title: "Lab 0: AI Without Specs"
layout: default
parent: "Part 1: Building from Scratch"
nav_order: 2
---

# Lab 0: AI Without Specs
{: .fs-9 }

Experience the chaos of unstructured AI development.
{: .fs-6 .fw-300 }

45 min
{: .label .label-yellow }

Hands-On
{: .label .label-green }

---

## The Scenario

It's **Monday morning**. Your PM sends this message:

> "Hey! We're pitching to investors on Friday. Need a working checkout demo -- payments, order history, the works. Something that looks real. Can you have it ready by end of day Thursday?"

You've got **4 days** to ship something demoable.

---

## The Reality Check

For this exercise, you have **30 minutes** instead of 4 days. Same pressure, compressed timeline.

Your challenge: Build a checkout feature that you'd be confident demoing to investors.

{: .note }
> **Where does your code go?** You're working in your Codespace. All code you create stays in your repository at `github.com/YOUR-USERNAME/sdd-greenfield-starter`. We'll compare this Lab 0 output to the spec-driven approach in Lab 1.3.

### Your Prompt

> "Build an e-commerce checkout system with payment processing and order management."

That's all the guidance you get. No spec. No constraints. Just you, your AI assistant, and a deadline.

---

## Instructions

### Step 1: Open Your AI Assistant (2 minutes)

Depending on which tool you're using:

<details open markdown="block">
<summary><strong>GitHub Copilot</strong> (Recommended)</summary>

1. Open Copilot Chat: `Ctrl+Alt+I` (Windows/Linux) or `Cmd+Alt+I` (Mac)
2. Select **Agent** mode at the top of the chat panel
3. Ensure you're signed in to GitHub

</details>

<details markdown="block">
<summary><strong>Gemini CLI</strong></summary>

1. Open terminal in VS Code: `Ctrl+` ` (backtick)
2. Run: `gemini`
3. Sign in with your Google account if prompted

</details>

<details markdown="block">
<summary><strong>Claude Code</strong></summary>

1. Open terminal in VS Code: `Ctrl+` ` (backtick)
2. Run: `claude`
3. Sign in with your Anthropic account if prompted

</details>

### Step 2: Start Building (25 minutes)

**Remember the scenario**: Investors are watching on Friday. You need something that:
- Looks professional
- Actually processes a payment (even if mocked)
- Shows order history
- Doesn't crash during the demo

Work naturally. Iterate with your AI. Add features. Fix bugs. Do whatever you'd normally do with a 4-day deadline.

### Step 3: The Friday Test (3 minutes)

With 3 minutes left, ask yourself:
- **Would I demo this?** Would you stand in front of investors with this code?
- **What's missing?** What would break during a live demo?
- **What would you rewrite?** If PM says "great, now make it production-ready" -- how much stays?

---

## Time Check

| Time | Scenario |
|------|----------|
| 0:00 | Monday AM -- you start coding |
| 0:15 | Wednesday -- halfway to demo |
| 0:25 | Thursday night -- final push |
| 0:30 | **FRIDAY MORNING** -- Demo time |

---

## What to Save

Before the timer ends, make sure you have:

1. **At least one working endpoint** (even if basic)
2. **Some kind of UI or API response** to demo
3. **Your chat history** with the AI (we'll discuss approaches)

---

## When the Timer Ends

**It's Friday. Demo time.**

Don't clean up. Don't add comments. Don't fix that one bug.

**Would you ship this? Would you demo this to investors?**

The state of your code right now IS the lesson.

---

## Debrief Discussion (15 minutes)

Your instructor will lead a discussion using these questions:

### The Demo Test
- **Confidence check**: On a scale of 1-10, how confident are you in demoing this to investors?
- **Crash risk**: What's the #1 thing that would break during a live demo?
- **The "one more day" fallacy**: If you had one more day, what would you fix first?

### The Rework Question
- **PM follow-up**: PM says "Great demo! Now make it production-ready." How much do you keep?
- **Estimate**: How many hours to go from demo code to production code?
- **The trap**: Did building fast actually save time, or create rework?

### The Pattern
- **Look around**: How many different approaches are in the room?
- **Why?**: Everyone got the same prompt. Why did everyone build something different?
- **What was missing?**: What information would have helped you ship faster AND better?

---

## Key Takeaways

{: .highlight }
> **The mess is the lesson.** This lab reveals a universal problem — not a personal failure.

1. **Speed is an illusion** — Code appears fast, but debug time adds up
2. **Consistency beats velocity** — Same prompt, different results across the room
3. **Thursday reveals the truth** — The code that ships Friday isn't the code you started Monday
4. **"Works" ≠ "Demoable"** — "Works on my machine" isn't "works on demo day"

### The Speed Trap (Visual)

| Feels Fast | Actually Slow |
|------------|---------------|
| Code appears in seconds | Debug time adds up |
| Features pile up quickly | Integration breaks things |
| "Almost done" by Wednesday | Thursday night rewrites |

---

## What's Next?

In **Lab 1.1**, you'll replay this scenario with one change:

**Before touching code, you'll spend 30 minutes writing a spec.**

You'll see how:
- AI follows YOUR architecture, not its own
- Edge cases get handled the first time
- Demo day code IS production-path code
- You ship Thursday with time to spare

**The contrast between this lab and Lab 1.3 output is why we call this "contrast-first" learning.**

---

## Clean Up: Discard Lab 0 Code

Before starting Lab 1.1, **discard all the code you just created**. Run this in your terminal:

```bash
git checkout -- .
git clean -fd
```

{: .warning }
> **Why discard?** Lab 0 code was for learning, not building on. Lab 1.1 starts fresh with the spec-driven approach. Keeping Lab 0 code would pollute your clean slate.

---

## Notes for Participants

- **This is normal** -- everyone struggles with this lab
- **The mess is the point** -- it reveals a universal problem
- **Your experience matters** -- share honestly in the debrief

> **Pro tip**: Save your chat history. We'll analyze what prompts worked and which created rework.
