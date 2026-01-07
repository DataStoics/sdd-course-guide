---
title: Prerequisites
layout: default
nav_order: 2
description: "Complete these setup steps before Day 1"
---
# Prerequisites

**Time Required**: ~4 days workshop (>4 hours per day depending on models used and iterations)

---

## Who This Course Is For

This course is designed for developers with **2+ years of professional experience** who want to learn structured AI-assisted development. You should be comfortable with:
- Git and GitHub workflows
- Command-line tools
- Writing and debugging code (Python experience helpful but not required)
- Basic API concepts (REST endpoints, JSON)

### What to Bring

- [ ] Laptop with working Codespace or local environment
- [ ] AI assistant tested and working (authentication complete)
- [ ] Power adapter
- [ ] Notepad for reflection notes (optional but recommended)

---

## The Big Picture

This course teaches **Spec-Driven Development** — a methodology that works with any modern AI coding assistant. The workflow matters more than the tool.

{: .note }
> **Tool choice is not critical.** GitHub Copilot (recommended), Claude Code, and Gemini CLI all support enterprise-grade Spec-Driven Development workflows. Pick whichever you have access to or are most comfortable with.

---

## Step 1: GitHub Account (5 min)

If you do not have one, create a free account at [github.com](https://github.com).

---

## Step 2: Development Environment (10 min)

We recommend **GitHub Codespaces** for a zero-install experience.

<details open markdown="block">
<summary><strong>GitHub Codespaces</strong> (Recommended)</summary>

**One-Click Setup:**

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/DataStoics/sdd-greenfield-starter?quickstart=1)

1. Click the badge above.
2. Wait 2-3 minutes for the container to build.
3. **Verify it works:** If VS Code loads and you see the terminal, you are ready.

**Save your work:** When you commit, VS Code will prompt "Publish to GitHub" — this creates your own copy.

**Official docs:** [GitHub Codespaces Quickstart](https://docs.github.com/en/codespaces/getting-started/quickstart)

</details>

<details markdown="block">
<summary><strong>Local Docker + VS Code</strong> (Optional / Offline)</summary>

For offline work or faster performance.

**Requirements:** Docker Desktop, VS Code, "Dev Containers" extension.

**Setup:**
1. Clone the template: `git clone https://github.com/DataStoics/sdd-greenfield-starter`
2. Open in VS Code.
3. Click "Reopen in Container" when prompted.

**Official docs:** [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)

</details>

---

## Step 3: AI Assistant (15 min)

Choose ONE. All three support the workflow.

<details open markdown="block">
<summary><strong>GitHub Copilot</strong> (Recommended)</summary>

Pre-installed in Codespaces.

1. Click the Copilot icon in the sidebar.
2. Sign in with GitHub.
3. **Select "Agent" mode** at the top of the chat panel.

{: .important }
> **Agent mode is required.** Regular Chat mode is not sufficient for this course.

</details>

<details markdown="block">
<summary><strong>Claude Code</strong> ($20/month)</summary>

1. Run: `npm install -g @anthropic-ai/claude-code`
2. Run: `claude` (Follow authentication prompts)

</details>

<details markdown="block">
<summary><strong>Gemini CLI</strong> (Free Tier Available)</summary>

1. Run: `npm install -g @google/gemini-cli`
2. Run: `gemini` (Follow authentication prompts)

</details>

---

## Step 4: Verify You Are Ready (2 min)

1. Open your AI Assistant.
2. Ensure you are signed in.

**Hint**: Just check you are properly authenticated against your AI assistant.

If you are logged in, you are ready for Day 1!

---

## See You on Day 1!

First activity: A hands-on contrast lab (Lab 0) where you will build something with AI — then discover why specs change everything.
