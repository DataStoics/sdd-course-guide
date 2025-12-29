---
title: Pre-Course Setup
layout: default
nav_order: 2
description: "Complete these setup steps before Day 1"
---
# Pre-Course Setup Guide

**Course**: SDD Foundations Course 1 (Greenfield Repo Flow)  
**Time Required**: 45-60 minutes  
**Difficulty**: Beginner-friendly

---

## Overview

Before the course begins, you must have a working development environment with AI assistant access in **agentic mode**. This guide walks you through setup and verification.

**Why agentic mode?** This course teaches spec-driven development where AI agents execute multi-step tasks from specifications autonomously. Chat-only AI tools cannot perform this workflow.

---

## Requirements Checklist

### 1. GitHub Account

- [ ] GitHub account created at https://github.com
- [ ] Able to fork public repositories

**That's it!** No SSH keys, no personal access tokens needed for this course. You'll fork the template repository and work in GitHub Codespaces, which handles authentication automatically.

### 2. Development Environment (Choose One)

#### Option A: GitHub Codespaces (Recommended)

**Best for**: Quick setup, limited local resources, Windows users

**Requirements**:
- [ ] Codespaces enabled on your GitHub account
- [ ] Browser access to github.com (Chrome/Firefox/Edge recommended)
- [ ] Stable internet connection (Codespaces runs in the cloud)

**Free tier**: 60 core-hours/month (plenty for this 2-day course)

#### Option B: Local Docker

**Best for**: Offline work, faster performance, experienced developers

**Requirements**:
- [ ] Docker Desktop installed and running
  - Windows: https://docs.docker.com/desktop/install/windows-install/
  - Mac: https://docs.docker.com/desktop/install/mac-install/
  - Linux: https://docs.docker.com/desktop/install/linux-install/
- [ ] VS Code installed: https://code.visualstudio.com/
- [ ] VS Code "Dev Containers" extension installed
- [ ] Git installed: https://git-scm.com/downloads
- [ ] At least 8GB RAM, 10GB free disk space

### 3. AI Assistant â€” Bring Your Own (REQUIRED)

**âš ï¸ CRITICAL**: This course requires an AI coding assistant. You'll bring your own â€” choose ONE from the supported options below.

#### Supported Options (Choose One):

| Tool | Free Tier | Paid Tier | Setup Guide |
|------|-----------|-----------|-------------|
| **GitHub Copilot** | 30-day trial, free for students/educators/OSS | $10/month individual | [Copilot Setup](#github-copilot-setup) |
| **Claude Code** | Free tier with limits | $20/month Pro | [Claude Code Setup](#claude-code-setup) |
| **Gemini CLI** | Free tier (generous) | Pay-as-you-go | [Gemini CLI Setup](#gemini-cli-setup) |

**Recommendation**: If you don't have a preference, start with **GitHub Copilot** (30-day free trial) or **Gemini CLI** (generous free tier).

#### NOT Sufficient:

| Tool | Why Not |
|------|---------|
| âŒ Basic Copilot autocomplete | Only suggests code in current file |
| âŒ ChatGPT web interface | Cannot access your codebase |
| âŒ Claude.ai web interface | Cannot execute file operations |
| âŒ Chat-only AI assistants | Cannot perform multi-file autonomous tasks |

**How to tell if your AI is agentic**:
- Can it create new files without you copying/pasting?
- Can it run terminal commands?
- Can it make changes across multiple files in one task?

If yes to all â†’ You have agentic mode âœ…

---

## Step-by-Step Setup Instructions

### GitHub Codespaces Setup (Option A)

**Time**: ~10 minutes

#### Step 1: Fork the Template Repository

1. Navigate to the course template: https://github.com/[instructor]/sdd-greenfield-template
   *(Instructor will provide the exact URL)*
2. Click **"Fork"** button (top right)
3. Keep default settings, click **"Create fork"**
4. You now have your own copy at `github.com/[your-username]/sdd-greenfield-template`

#### Step 2: Create Codespace

1. Go to YOUR forked repository (`github.com/[your-username]/sdd-greenfield-template`)
2. Click green **"Code"** button
3. Select **"Codespaces"** tab
4. Click **"Create codespace on main"**

   ![Codespace creation](images/codespace-create.png)
   *(If image doesn't exist, instructor will demonstrate)*

5. Wait for dev container to build (~2-3 minutes)
   - You'll see VS Code loading in browser
   - Terminal will show container setup progress

#### Step 3: Verify Environment

Open the integrated terminal (View â†’ Terminal or Ctrl+`) and run:

```bash
# Check Python version (should be 3.11+)
python --version
# Expected: Python 3.11.x

# Check Docker
docker --version
# Expected: Docker version 24.x.x or higher

# Check Docker Compose
docker-compose --version
# Expected: Docker Compose version v2.x.x

# Start services and verify Redis
docker-compose up -d
redis-cli ping
# Expected: PONG

# Verify pytest
pytest --version
# Expected: pytest 7.x.x or higher
```

**All commands should succeed.** If any fail, see Troubleshooting section below.

#### Step 4: Verify AI Assistant

**For Claude Code agent**:
1. Open Claude Code panel in VS Code
2. Type: "Create a file called hello.py with a function that prints 'Hello SDD'"
3. Verify it:
   - Creates the file autonomously (not just shows code)
   - The file appears in your file explorer
   - Contains working Python code

**For GitHub Copilot**:
1. Open any Python file in your Codespace
2. Open Copilot Chat: press `Ctrl+Shift+I` (or `Cmd+Shift+I` on Mac)
3. Type: "Create a file called hello.py with a function that prints 'Hello SDD'"
4. Verify it creates the file (not just shows code)

**For Gemini CLI**:
1. Open the terminal in your Codespace
2. Ensure your API key is set: `echo $GEMINI_API_KEY`
3. Run `gemini`
4. Type: "Create a file called hello.py with a function that prints 'Hello SDD'"
5. Verify the file appears in your file explorer

**Success indicator**: AI can create files and run commands without you copying/pasting.

---

### AI Assistant Setup Guides

#### GitHub Copilot Setup

**Time**: ~10 minutes

1. **Get Access**:
   - Free trial: https://github.com/features/copilot (30 days)
   - Students/educators: https://education.github.com (free)
   - Individual: $10/month at github.com/settings/copilot

2. **Install Extension**:
   - In VS Code (or Codespaces), go to Extensions
   - Search "GitHub Copilot"
   - Install both "GitHub Copilot" and "GitHub Copilot Chat"
   - Sign in when prompted

3. **Verify**:
   - Open any `.py` file
   - Start typing a function â€” Copilot should suggest completions
   - Open Copilot Chat (Ctrl+Shift+I) and ask it to create a file

#### Claude Code Setup

**Time**: ~15 minutes

1. **Get Access**:
   - Free tier: https://claude.ai (sign up, then access Claude Code)
   - Pro tier: $20/month for higher limits

2. **Install**:
   - Follow instructions at: https://docs.anthropic.com/en/docs/claude-code
   - Claude Code runs as a CLI tool in your terminal

3. **Verify**:
   - In terminal, run `claude` to start
   - Ask: "Create a file called test.py with a hello world function"
   - Verify the file is created

#### Gemini CLI Setup

**Time**: ~15 minutes

1. **Get Google AI API Key**:
   - Go to https://aistudio.google.com/apikey
   - Sign in with Google account
   - Click "Create API Key"
   - Copy the key (starts with `AIza...`)

2. **Install Gemini CLI**:
   ```bash
   # Using npm (Node.js required)
   npm install -g @anthropics/gemini-cli
   
   # Or download binary from:
   # https://github.com/google-gemini/gemini-cli/releases
   ```

3. **Configure**:
   ```bash
   # Set your API key
   export GEMINI_API_KEY="your-api-key-here"
   
   # Or add to your shell profile (.bashrc, .zshrc, etc.)
   echo 'export GEMINI_API_KEY="your-api-key-here"' >> ~/.bashrc
   ```

4. **Verify**:
   - Run `gemini` in terminal
   - Ask: "Create a file called test.py with a hello world function"
   - Verify the file is created

**Troubleshooting Gemini CLI**:
| Problem | Solution |
|---------|----------|
| "API key not found" | Ensure `GEMINI_API_KEY` is exported in current shell |
| "npm not found" | Install Node.js from https://nodejs.org |
| Rate limit errors | Wait a few minutes, free tier has generous limits |

---

### Local Docker Setup (Option B)

**Time**: ~20-30 minutes (depending on download speeds)

#### Step 1: Install Prerequisites

**Docker Desktop**:
1. Download from docker.com for your OS
2. Run installer, accept defaults
3. **Windows users**: Choose "Use WSL 2 instead of Hyper-V" when prompted
4. Restart computer if prompted
5. Launch Docker Desktop
6. Wait for "Docker Desktop is running" status

**VS Code**:
1. Download from code.visualstudio.com
2. Install with default options
3. Launch VS Code

**Dev Containers Extension**:
1. In VS Code, click Extensions icon (left sidebar)
2. Search "Dev Containers"
3. Install "Dev Containers" by Microsoft

**Git**:
1. Download from git-scm.com
2. Install with default options
3. Configure your identity:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

#### Step 2: Clone Template Repository

Open terminal (PowerShell on Windows, Terminal on Mac/Linux):

```bash
# Navigate to your preferred projects directory
cd ~/projects  # or wherever you keep code

# Clone the template (instructor will provide URL)
git clone <template-repo-url>
cd sdd-greenfield-template
```

#### Step 3: Open in Dev Container

```bash
# Open VS Code in current directory
code .
```

When VS Code opens:
1. You should see a notification: "Folder contains a Dev Container configuration"
2. Click **"Reopen in Container"**
3. Wait for container build (~3-5 minutes first time)

If no notification appears:
1. Press F1 (command palette)
2. Type "Dev Containers: Reopen in Container"
3. Press Enter

#### Step 4: Verify Environment

Same verification steps as Codespaces (Step 3 above).

#### Step 5: Verify AI Assistant

Same verification steps as Codespaces (Step 4 above).

---

## Troubleshooting

### Codespaces Issues

| Problem | Solution |
|---------|----------|
| **Codespace won't start** | Check quota at github.com/settings/billing â†’ Codespaces |
| **Container build fails** | Delete codespace, create new one |
| **"Out of storage" error** | Delete unused codespaces, or upgrade plan |
| **Slow performance** | Try different browser, or use local Docker |
| **Can't access terminal** | View â†’ Terminal, or press Ctrl+` |

### Docker Issues

| Problem | Solution |
|---------|----------|
| **"Docker daemon not running"** | Start Docker Desktop application |
| **WSL error (Windows)** | Run `wsl --update` in PowerShell as admin |
| **"Cannot connect" on Mac** | Increase memory: Docker Desktop â†’ Settings â†’ Resources â†’ 4GB+ |
| **Container build fails** | Run `docker system prune -a` to clear cache, retry |
| **Port already in use** | Stop conflicting service: `docker ps` then `docker stop <id>` |

### AI Assistant Issues

| Problem | Solution |
|---------|----------|
| **Copilot not suggesting code** | Check subscription active, sign out/in of GitHub |
| **Copilot Chat won't create files** | Use "Apply" button on suggested code, or try `/new` command |
| **Claude Code not responding** | Check API limits, restart terminal session |
| **Gemini CLI "API key not found"** | Run `export GEMINI_API_KEY="your-key"` in current terminal |
| **Gemini CLI "npm not found"** | Install Node.js from https://nodejs.org |
| **"Rate limit exceeded"** | Wait a few minutes, or upgrade to paid tier |
| **AI only suggests code, won't create files** | Some modes require you to "Apply" â€” check tool docs |

### Git/GitHub Issues

| Problem | Solution |
|---------|----------|
| **"Cannot fork" error** | Ensure you're signed into GitHub |
| **Fork already exists** | Use existing fork, or delete and re-fork |
| **Can't push to fork** | Codespaces handles this automatically â€” just commit |

### General Issues

| Problem | Solution |
|---------|----------|
| **Nothing works** | Contact instructor before course starts |
| **Works but slow** | Close other applications, check internet speed |
| **Different Python version** | Use the dev containerâ€”it has correct version |

---

## Environment Verification Script

Run this script to verify everything is working:

```bash
#!/bin/bash
echo "=== SDD Course Environment Check ==="

echo -n "Python 3.11+: "
python --version 2>&1 | grep -q "3.11" && echo "âœ…" || echo "âŒ"

echo -n "Docker: "
docker --version >/dev/null 2>&1 && echo "âœ…" || echo "âŒ"

echo -n "Docker Compose: "
docker-compose --version >/dev/null 2>&1 && echo "âœ…" || echo "âŒ"

echo -n "Git: "
git --version >/dev/null 2>&1 && echo "âœ…" || echo "âŒ"

echo -n "pytest: "
pytest --version >/dev/null 2>&1 && echo "âœ…" || echo "âŒ"

echo ""
echo "If all checks show âœ…, you're ready!"
echo "If any show âŒ, see troubleshooting guide."
```

Save as `check-env.sh`, run with `bash check-env.sh`.

---

## Pre-Course Skill Check (Optional)

Want to assess your starting point? Complete the optional pre-assessment:

1. Open [assessments/pre-course-assessment.md](assessments/pre-course-assessment.md)
2. Complete the assessment (15-20 minutes)
3. Submit via course portal (link provided by instructor)

This helps instructors tailor the course to participant needs.

---

## What to Bring on Day 1

- [ ] Laptop with browser access to GitHub
- [ ] Your forked repository with working Codespace (verified above)
- [ ] AI assistant configured and tested (Copilot, Claude Code, or Gemini CLI)
- [ ] Power adapter/charger
- [ ] Notebook for personal notes (optional)

---

## Questions Before the Course?

**Technical setup issues**: Post in course discussion forum or contact instructor

**Best to resolve issues at least 24 hours before Day 1** so we can start on time!

---

## See You at 09:00 on Day 1!

With your environment forked and AI assistant ready, you're prepared to experience specification-driven development firsthand. 

**First activity**: A 45-minute "contrast exercise" where you'll build something with your AI assistant â€” then discover why specs matter!
