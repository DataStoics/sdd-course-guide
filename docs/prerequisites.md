---
title: Prerequisites
layout: default
nav_order: 2
description: "Complete these setup steps before Day 1"
---
# Prerequisites
{: .fs-9 }

Get your environment ready before the workshop begins.
{: .fs-6 .fw-300 }

**Setup Time**: 15-30 minutes
{: .label .label-blue }

---

## Who This Course Is For

{: .highlight }
> Developers with **2+ years of professional experience** who want to learn structured AI-assisted development.

You should be comfortable with:

| Skill | Why It Matters |
|:------|:---------------|
| Git & GitHub | Version control for all lab work |
| Command-line tools | Running spec-kit commands |
| Writing/debugging code | Python helpful, not required |
| REST API basics | Building endpoints in labs |

### What to Bring

- [ ] Laptop with working Codespace or local environment
- [ ] AI assistant tested and working
- [ ] Power adapter
- [ ] Notepad for reflection notes *(optional)*

---

## The Big Picture

This course teaches **Spec-Driven Development** — a methodology that works with any modern AI coding assistant.

{: .note }
> **Tool choice is flexible.** GitHub Copilot (recommended), Claude Code, and Gemini CLI all support the SDD workflow. Pick whichever you have access to.

---

## Step 1: GitHub Account
{: .d-inline-block }

5 min
{: .label .label-green }

If you don't have one, create a free account at [github.com](https://github.com).

{: .tip }
> That's it for this step. Codespaces handles authentication automatically.

---

## Step 2: Development Environment
{: .d-inline-block }

5-10 min
{: .label .label-green }

We recommend **GitHub Codespaces** for a zero-install experience.

<details open markdown="block">
<summary><strong>GitHub Codespaces</strong> (Recommended)</summary>

**One-Click Setup:**

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/DataStoics/sdd-greenfield-starter?quickstart=1)

1. Click the badge above
2. Wait 2-3 minutes for the container to build
3. **Verify:** VS Code loads with a terminal prompt

{: .tip }
> **Save your work:** When you commit, VS Code prompts "Publish to GitHub" — this creates your own copy.

📖 [GitHub Codespaces Quickstart](https://docs.github.com/en/codespaces/getting-started/quickstart)

</details>

<details markdown="block">
<summary><strong>Local Docker + VS Code</strong> (Optional)</summary>

For offline work or faster performance.

**Requirements:** Docker Desktop, VS Code, "Dev Containers" extension

**Setup:**
```bash
git clone https://github.com/DataStoics/sdd-greenfield-starter
code sdd-greenfield-starter
# Click "Reopen in Container" when prompted
```

📖 [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)

</details>

---

## Step 3: AI Assistant
{: .d-inline-block }

5-15 min
{: .label .label-green }

Choose **one** assistant. All three support the SDD workflow.

<details open markdown="block">
<summary><strong>GitHub Copilot</strong> (Recommended)</summary>

Pre-installed in Codespaces. Just sign in.

1. Click the **Copilot icon** in the sidebar (or `Ctrl+Alt+I`)
2. Sign in with your GitHub account
3. Select **"Agent"** mode at the top of the chat panel

{: .warning }
> **Agent mode is required.** Regular Chat mode cannot create files or run commands — you need Agent mode for this course.

| Tier | Cost | Limits |
|:-----|:-----|:-------|
| Free | $0 | 2,000 completions + 50 chats/month |
| Pro | $10/month | Unlimited completions, 300 chats |
| Business | $19/user | Enterprise features |

📖 [GitHub Copilot Quickstart](https://docs.github.com/en/copilot/quickstart)

</details>

<details markdown="block">
<summary><strong>Claude Code</strong></summary>

Requires paid subscription ($20/month Pro).

```bash
npm install -g @anthropic-ai/claude-code
claude
# Follow authentication prompts
```

📖 [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

</details>

<details markdown="block">
<summary><strong>Gemini CLI</strong></summary>

Free tier available (1,000 requests/day).

```bash
npm install -g @google/gemini-cli
gemini
# Follow authentication prompts
```

📖 [Gemini CLI on GitHub](https://github.com/google-gemini/gemini-cli)

</details>

---

## Step 4: Verify Setup
{: .d-inline-block }

2 min
{: .label .label-green }

Quick check that everything works:

1. Open your AI assistant
2. Confirm you're signed in
3. *(Optional)* Ask it to create a test file

{: .highlight }
> ✅ **If you're logged in, you're ready for Day 1!**

---

## Troubleshooting

| Problem | Solution |
|:--------|:---------|
| Codespace won't start | Check quota at GitHub Settings → Billing → Codespaces |
| Copilot not responding | Ensure you're signed into GitHub in VS Code |
| Copilot shows code but won't create files | Switch to **Agent** mode in the Chat panel |
| Rate limit errors | Wait a few minutes, or upgrade to paid tier |

---

## Ready to Begin?

First activity: A hands-on contrast lab where you'll build something with AI — then discover why specs change everything.

[Start Part 1: Greenfield Development](labs/part1/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 }
