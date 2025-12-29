---
title: Pre-Course Setup
layout: default
nav_order: 4
description: "Complete these setup steps before Day 1"
---
# Pre-Course Setup Guide

**Time Required**: 30-45 minutes

---

## Prerequisites

{: .important }
> **Who This Course Is For**: This course is designed for developers with **2+ years of professional experience** who want to learn structured AI-assisted development. You should be comfortable with:
> - Git and GitHub workflows
> - Command-line tools
> - Writing and debugging code (Python experience helpful but not required)
> - Basic API concepts (REST endpoints, JSON)

**Time Investment During Course**: Plan for **6-8 hours over 2 days** (Day 1: ~4 hours, Day 2: ~3 hours). Labs build on each other, so blocking continuous time is recommended.

---

## The Big Picture

This course teaches **Spec-Driven Development**a methodology that works with any modern AI coding assistant. The specific tool you choose matters far less than learning the workflow itself.

{: .note }
> **Tool choice is not critical.** GitHub Copilot, Claude Code, and Gemini CLI all support enterprise-grade Spec-Driven Development workflows. Pick whichever you have access to or are most comfortable with.

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

That is it for this step. No SSH keys or tokens neededCodespaces handles authentication.

---

## Step 2: Development Environment

Choose ONE option. Click to expand setup instructions:

<details open markdown="block">
<summary><strong>GitHub Codespaces</strong> (Recommended - No Installation)</summary>

Cloud-based VS Code that runs in your browser. No local installation required.

**Requirements:**
- GitHub account with Codespaces enabled (free tier: 120 core-hours/month)
- Modern browser (Chrome, Firefox, Edge)
- Stable internet connection

**One-Click Setup:**

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/DataStoics/sdd-greenfield-starter?quickstart=1)

Click the badge above, or go to [github.com/DataStoics/sdd-greenfield-starter](https://github.com/DataStoics/sdd-greenfield-starter) and click **Code** > **Codespaces** > **Create codespace on main**.

Wait 2-3 minutes for the container to build. Gemini CLI is pre-installed.

**Verify it works:**
```bash
python --version
gemini --version
```

Expected output: Python 3.12.x and Gemini CLI version number.

**Save your work later:** When you commit, VS Code will prompt "Publish to GitHub"this creates your own copy.

**Official docs:** [GitHub Codespaces Quickstart](https://docs.github.com/en/codespaces/getting-started/quickstart)

</details>

<details markdown="block">
<summary><strong>Local Docker + VS Code</strong> (For Offline Work)</summary>

For offline work or faster performance.

**Requirements:**
- Docker Desktop ([download](https://www.docker.com/products/docker-desktop/))
- VS Code ([download](https://code.visualstudio.com/))
- VS Code "Dev Containers" extension
- 8GB RAM, 10GB free disk space

**Setup:**
1. Install Docker Desktop and VS Code
2. Install the Dev Containers extension in VS Code
3. Clone the template: `git clone https://github.com/DataStoics/sdd-greenfield-starter`
4. Open folder in VS Code
5. Click "Reopen in Container" when prompted

**Verify it works:**
```bash
python --version
gemini --version
```

Expected output: Python 3.12.x and Gemini CLI version number.

**Official docs:**
- [Docker Desktop Installation](https://docs.docker.com/desktop/)
- [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)

</details>

---

## How You Will Work During the Course

{: .important }
> **Critical: Use Terminal-Based AI Assistants**
>
> During this course, you will interact with your AI assistant **in the terminal**, not through VS Code's built-in chat panel. The spec-kit commands (`/speckit.specify`, `/speckit.clarify`, etc.) work through CLI tools, not IDE extensions.

You will have **two browser tabs** open:

| Tab | Purpose |
|:----|:--------|
| **Tab 1: Course Guide** | [datastoics.github.io/sdd-course-guide](https://datastoics.github.io/sdd-course-guide)read lab instructions |
| **Tab 2: Codespace** | Your development environmentwrite code with AI assistant |

**Workflow:**
1. Read the lab instructions in Tab 1
2. Switch to Tab 2 and **open the integrated terminal**
3. Run your AI assistant CLI (e.g., `gemini`)
4. Use spec-kit commands as instructed
5. Verify results match the lab's success criteria

{: .tip }
> Keep both tabs visible if you have a large monitor, or use split screen.

---

## Step 3: AI Coding Assistant

Choose ONE. All three support the Spec-Driven Development workflow we teach. Click to expand setup instructions:

<details open markdown="block">
<summary><strong>Gemini CLI</strong> (Recommended - Free)</summary>

Best choice for this course. Free tier with generous limits.

| Tier | Cost | Notes |
|:-----|:-----|:------|
| Free | $0 | 1,000 requests/day, 60/minute |
| Pro/Ultra | $20+/month | Higher limits |

**If using Codespaces:** Already installed! Continue to authentication below.

**If using local setup:**
```bash
npm install -g @google/gemini-cli
```

### Authenticate with Google

{: .warning }
> **Required Step**: Gemini CLI requires Google account authentication before use.

1. **Start Gemini CLI:**
   ```bash
   gemini
   ```

2. **Follow the authentication prompt:**
   - A URL will appear in your terminal
   - Open that URL in your browser (or Ctrl+click in Codespaces)
   - Sign in with your Google account
   - Grant the requested permissions
   - Return to your terminalauthentication completes automatically

3. **Verify authentication succeeded:**
   ```
   You should see the Gemini prompt () ready for input
   ```

**If authentication fails:**
- Ensure you're using a personal Google account (some corporate accounts block third-party apps)
- Try `gemini auth login` to re-authenticate
- Check your firewall isn't blocking the auth callback

**Official docs:** [Gemini CLI on GitHub](https://github.com/google-gemini/gemini-cli)

</details>

<details markdown="block">
<summary><strong>GitHub Copilot</strong> ($10/month)</summary>

Best if you already use GitHub or want the tightest VS Code integration.

| Tier | Cost | Notes |
|:-----|:-----|:------|
| Free | $0 | 50 agent requests/monthmay be limiting |
| Pro | $10/month | 300 agent requests/month |
| Business | $19/user/month | Enterprise features, admin controls |

**Setup:**
1. Install "GitHub Copilot" extension in VS Code
2. Install "GitHub Copilot Chat" extension in VS Code
3. Sign in with your GitHub account
4. Select "Agent" mode in the Chat panel (not just "Chat")

{: .warning }
> **For this course**: While Copilot works in the VS Code sidebar, you'll get better results using the **terminal-based agent mode** by pressing `Ctrl+Shift+`\`` to open the terminal, then using Copilot CLI commands.

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
```

### Authenticate with Anthropic

1. **Start Claude Code:**
   ```bash
   claude
   ```

2. **Follow the authentication prompt:**
   - You'll be prompted for your Anthropic API key or to sign in
   - If using API key: Get it from [console.anthropic.com](https://console.anthropic.com)
   - If signing in: Follow the browser flow similar to Gemini

3. **Verify authentication succeeded:**
   ```
   You should see Claude ready for input
   ```

**Official docs:** [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)

</details>

---

## Step 4: Verify Everything Works

Test your AI assistant by asking it to create a simple file.

{: .important }
> **Use the terminal, not VS Code's chat panel.** The spec-kit workflow runs through CLI tools.

<details open markdown="block">
<summary><strong>Gemini CLI</strong> (click to expand)</summary>

**In your Codespace terminal, run:**
```bash
gemini
```

**After authentication completes**, type this prompt:
```
Create a file called hello.py with a function that prints 'Hello SDD'
```

**Success looks like:**
```
Created hello.py
```

The file appears in your file explorer. Type `/quit` to exit Gemini CLI.

</details>

<details markdown="block">
<summary><strong>GitHub Copilot</strong> (click to expand)</summary>

**Open a terminal** in your Codespace (Ctrl+Shift+`).

**For CLI mode**, use:
```bash
gh copilot suggest "Create a file called hello.py with a function that prints Hello SDD"
```

**Alternatively, in VS Code Chat:**
1. Open Copilot Chat (click the Copilot icon or press `Ctrl+Alt+I`)
2. Select **"Agent"** mode at the top of the chat panel
3. Type your prompt

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

**After authentication completes**, type this prompt:
```
Create a file called hello.py with a function that prints 'Hello SDD'
```

**Success looks like:**
```
I'll create that file for you.

hello.py
 Created
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
| Gemini CLI says "not authenticated" | Run `gemini auth login` and complete browser flow |
| Gemini auth URL doesn't open | Copy the URL manually and paste in browser |
| Corporate Google account blocked | Try a personal Google account instead |
| Claude Code API key rejected | Verify key at console.anthropic.com, check for typos |
| AI only shows code, will not create files | You may be in chat-only modeuse terminal CLI instead |
| Rate limit errors | Wait a few minutes, or upgrade to paid tier |
| `/speckit.specify` command not found | Run `specify init .` first (covered in Lab 1.1) |

For tool-specific issues, refer to the official documentation linked abovethey are kept current by the tool maintainers.

---

## What to Bring on Day 1

- [ ] Laptop with working Codespace or local environment
- [ ] AI assistant tested and working (authentication complete)
- [ ] Power adapter
- [ ] Notepad for reflection notes (optional but recommended)

---

## Questions?

Post in the [course discussion forum](https://github.com/DataStoics/sdd-course-guide/discussions) or contact your instructor.

**Resolve issues at least 24 hours before Day 1** so we can start on time.

---

## See You on Day 1!

First activity: A hands-on contrast lab (Lab 0) where you will build something with AIthen discover why specs change everything.