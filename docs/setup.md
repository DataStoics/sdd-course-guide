---
title: Pre-Course Setup
layout: default
nav_order: 2
description: "Complete these setup steps before Day 1"
---
# Pre-Course Setup Guide

**Time Required**: 30-45 minutes

---

## The Big Picture

This course teaches **Spec-Driven Development** -- a methodology that works with any modern AI coding assistant. The specific tool you choose matters far less than learning the SDD workflow itself.

{: .note }
> **Tool choice is not critical.** GitHub Copilot, Claude Code, and Gemini CLI all support enterprise-grade SDD workflows. Pick whichever you have access to or are most comfortable with.

By the end of setup, you need:
1. A GitHub account
2. A development environment (Codespaces recommended)
3. An AI coding assistant in **agentic mode**

---

## What is "Agentic Mode"?

Your AI assistant must be able to:
- Create and edit files autonomously
- Run terminal commands
- Work across multiple files in one task

**This is NOT sufficient:**
- Chat-only interfaces (ChatGPT web, Claude.ai web)
- Basic autocomplete (Copilot without Chat)
- Copy-paste workflows

If your AI can create a file when you ask without you copying code, you have agentic mode.

---

## Step 1: GitHub Account

If you do not have one, create a free account at [github.com](https://github.com).

That is it for this step. No SSH keys or tokens needed -- Codespaces handles authentication.

---

## Step 2: Development Environment

Choose ONE option:

### Option A: GitHub Codespaces (Recommended)

Cloud-based VS Code that runs in your browser. No local installation required.

**Requirements:**
- GitHub account with Codespaces enabled (free tier: 120 core-hours/month)
- Modern browser (Chrome, Firefox, Edge)
- Stable internet connection

**One-Click Setup:**

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/DataStoics/sdd-greenfield-starter?quickstart=1)

Click the badge above, or go to [github.com/DataStoics/sdd-greenfield-starter](https://github.com/DataStoics/sdd-greenfield-starter) and click **Code** > **Codespaces** > **Create codespace on main**.

Wait 2-3 minutes for the container to build. Gemini CLI is pre-installed.

<!-- SCREENSHOT: Show the "Create codespace" dropdown menu -->

**Verify it works:**
```bash
python --version    # Should show 3.12.x
gemini --version    # Should show installed version
```

**Save your work later:** When you commit, VS Code will prompt "Publish to GitHub" -- this creates your own copy.

**Official docs:** [GitHub Codespaces Quickstart](https://docs.github.com/en/codespaces/getting-started/quickstart)

---

## How You'll Work During the Course

You'll have **two browser tabs** open:

| Tab | Purpose |
|:----|:--------|
| **Tab 1: Course Guide** | [datastoics.github.io/sdd-course-guide](https://datastoics.github.io/sdd-course-guide) -- read lab instructions |
| **Tab 2: Codespace** | Your development environment -- write code with AI assistant |

**Workflow:**
1. Read the lab instructions in Tab 1
2. Switch to Tab 2 and tell your AI assistant what to do
3. Verify results match the lab's success criteria
4. Move to next section

{: .tip }
> Keep both tabs visible if you have a large monitor, or use split screen.

---

### Option B: Local Docker + VS Code

For offline work or faster performance.

**Requirements:**
- Docker Desktop ([download](https://www.docker.com/products/docker-desktop/))
- VS Code ([download](https://code.visualstudio.com/))
- VS Code "Dev Containers" extension
- 8GB RAM, 10GB free disk space

**Setup:**
1. Install Docker Desktop and VS Code
2. Install the Dev Containers extension in VS Code
3. Clone the course template repository
4. Open folder in VS Code
5. Click "Reopen in Container" when prompted

<!-- SCREENSHOT: Show the "Reopen in Container" notification in VS Code -->

**Official docs:**
- [Docker Desktop Installation](https://docs.docker.com/desktop/)
- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)

---

## Step 3: AI Coding Assistant

Choose ONE. All three support the SDD workflow we teach. Click to expand setup instructions:

<details markdown="block">
<summary><strong>Gemini CLI</strong> (Recommended - Free)</summary>

Best choice for this course. Free tier with generous limits.

| Tier | Cost | Notes |
|:-----|:-----|:------|
| Free | $0 | 1,000 requests/day, 60/minute |
| Pro/Ultra | $20+/month | Higher limits |

**If using Codespaces:** Already installed! Just run `gemini` and sign in with your Google account.

**If using local setup:**
```bash
npm install -g @google/gemini-cli
gemini
```

**Official docs:** [Gemini CLI on GitHub](https://github.com/google-gemini/gemini-cli)

</details>

<details markdown="block">
<summary><strong>GitHub Copilot</strong> ($10/month)</summary>

Best if you already use GitHub or want the tightest VS Code integration.

| Tier | Cost | Notes |
|:-----|:-----|:------|
| Free | $0 | 50 agent requests/month -- may be limiting |
| Pro | $10/month | 300 agent requests/month |
| Business | $19/user/month | Enterprise features, admin controls |

**Setup:**
1. Install "GitHub Copilot" extension in VS Code
2. Install "GitHub Copilot Chat" extension in VS Code
3. Sign in with your GitHub account
4. Select "Agent" mode in the Chat panel (not just "Chat")

**Official docs:** [GitHub Copilot Quickstart](https://docs.github.com/en/copilot/quickstart)

</details>

<details markdown="block">
<summary><strong>Claude Code</strong> ($20/month required)</summary>

Best for complex reasoning and long context windows. Requires paid subscription.

| Tier | Cost | Notes |
|:-----|:-----|:------|
| Pro | $20/month | Required for Claude Code CLI |
| Max | $100+/month | Higher usage limits |

**Setup:**
```bash
npm install -g @anthropic-ai/claude-code
claude
```

Sign in with your Anthropic account when prompted.

**Official docs:** [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

</details>

---

## Step 4: Verify Everything Works

Test your AI assistant by asking it to create a simple file. Click your assistant below for specific instructions:

<details markdown="block">
<summary><strong>Gemini CLI</strong> (click to expand)</summary>

**In your Codespace terminal, run:**
```bash
gemini
```

**Sign in when prompted**, then type this prompt:
```
Create a file called hello.py with a function that prints 'Hello SDD'
```

**Success looks like:**
```
âœ“ Created hello.py
```

The file appears in your file explorer. Type `/quit` to exit Gemini CLI.

</details>

<details markdown="block">
<summary><strong>GitHub Copilot</strong> (click to expand)</summary>

**Open Copilot Chat** (click the Copilot icon in the sidebar or press `Ctrl+Alt+I`).

**Select "Agent" mode** at the top of the chat panel, then type:
```
Create a file called hello.py with a function that prints 'Hello SDD'
```

**Success looks like:**
- Copilot shows "Created hello.py" or similar
- The file appears in your file explorer
- You did NOT have to copy-paste code

If Copilot only shows code without creating the file, you may be in "Chat" mode instead of "Agent" mode.

</details>

<details markdown="block">
<summary><strong>Claude Code</strong> (click to expand)</summary>

**In your Codespace terminal, run:**
```bash
claude
```

**Sign in when prompted**, then type this prompt:
```
Create a file called hello.py with a function that prints 'Hello SDD'
```

**Success looks like:**
```
I'll create that file for you.

hello.py
â”œâ”€â”€ Created
```

The file appears in your file explorer. Type `/exit` to exit Claude Code.

</details>

---

**After verification:** Delete the test file to start fresh:
```bash
rm hello.py
```

If this works, you are ready for Day 1.

---

## Troubleshooting

| Problem | Solution |
|:--------|:---------|
| Codespace will not start | Check your quota at GitHub Settings > Billing > Codespaces |
| Docker not running | Start Docker Desktop application |
| AI only shows code, will not create files | You may be in chat-only mode -- check tool docs for agentic/agent mode |
| Rate limit errors | Wait a few minutes, or upgrade to paid tier |

For tool-specific issues, refer to the official documentation linked above -- they are kept current by the tool maintainers.

---

## What to Bring on Day 1

- [ ] Laptop with working Codespace or local environment
- [ ] AI assistant tested and working in agentic mode
- [ ] Power adapter

---

## Questions?

Post in the course discussion forum or contact your instructor.

**Resolve issues at least 24 hours before Day 1** so we can start on time.

---

## See You on Day 1!

First activity: A hands-on contrast exercise where you will build something with AI -- then discover why specs change everything.
