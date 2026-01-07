---
title: "Lab 2.0: The Inherited Codebase"
layout: default
parent: "Part 2: Transforming Legacy Code"
nav_order: 1
---

# Lab 2.0: The Inherited Codebase
{: .fs-9 }

Monday Morning Surprise
{: .fs-6 .fw-300 }

[~45 min]{: .label .label-blue }
[Exploration]{: .label .label-green }

---

## Learning Objective

Experience the reality of inheriting undocumented legacy code. Develop systematic exploration techniques before attempting any changes.

> **Before you can fix it, you must understand it.**
{: .highlight }

---

## The Scenario

You shipped your demo on Friday. Investors loved it. Monday morning, your PM has news:

> "Great news! We've acquired OrderFlow Inc. Their system handles 200 orders/day and 'just works.'
> 
> Bad news: The previous developer left 6 months ago. There's no documentation.
> 
> Your task: Figure out what this thing does, then integrate it with our payment system.
> 
> First step: Tell us what we actually bought."

**This is every legacy codebase story.** Code that works, nobody knows why, and now it's your problem.

---

## Step 0: Multi-Repo Setup (10 min)

Part 2 requires working across **two repositories**:

1. **Your greenfield repo** from Part 1 (payment service) — where you'll write integration code
2. **legacy-orderflow** — the acquired codebase you need to understand

### Clone the Legacy Repo

```bash
# Navigate to your labs directory (same parent as your Part 1 repo)
cd ~/labs  # or wherever your Part 1 repo lives

# Clone the acquired company's codebase
git clone https://github.com/DataStoics/legacy-orderflow
```

Your directory structure should now look like:

```
~/labs/
├── sdd-greenfield-starter/     # Your Part 1 repo (payment service)
│   ├── .specify/
│   ├── specs/
│   └── src/
└── legacy-orderflow/           # The acquired monolith (just cloned)
    └── app.py                  # ~2,500 lines of "it just works"
```

### Set Up Your Workspace

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

Download and open the multi-root workspace:

```bash
curl -O https://raw.githubusercontent.com/DataStoics/sdd-course-guide/main/workspaces/part2-brownfield.code-workspace
code part2-brownfield.code-workspace
```

**What this does**: Opens both repos in a single VS Code window. Copilot can reference files across repos using `@legacy-orderflow/app.py` syntax.

{: .tip }
> Edit the workspace file to point to your actual Part 1 repo path if it's named differently than `sdd-greenfield-starter`.

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

You'll work with two terminal sessions:

```bash
# Terminal 1 - For analyzing legacy code:
cd ~/labs/legacy-orderflow

# Terminal 2 - For writing integration code:
cd ~/labs/sdd-greenfield-starter
```

Reference the legacy repo from your greenfield repo using relative paths:

```
> "Analyze ../legacy-orderflow/app.py and..."
```

</details>

---

## Step 1: First Contact (10 min)

**Run the legacy application** without reading any code first.

```bash
cd legacy-orderflow

# Option A: Docker (recommended)
docker-compose up -d

# Option B: Direct Python
pip install -r requirements.txt
python app.py
```

The app runs on **port 5001** (your Part 1 service is on 8000).

### Explore Like a User

Try these endpoints and document what happens:

| Endpoint | Method | What to Try | What Happened? |
|----------|--------|-------------|----------------|
| `/health` | GET | Basic health check | |
| `/products` | GET | Browse catalog | |
| `/register` | POST | Create user `{"username":"test","email":"test@test.com","password":"test123"}` | |
| `/login` | POST | Login `{"email":"test@test.com","password":"test123"}` | |
| `/cart/add` | POST | Add to cart (use token from login) | |
| `/orders` | POST | Create order | |

{: .note }
> Don't read the code yet! Explore as a user would. Your first observations will inform what to look for in the code.

### Document Your Findings

Create a simple exploration log:

```markdown
## First Contact Notes

**What Works:**
- ...

**What's Confusing:**
- ...

**What Looks Suspicious:**
- ...

**Questions I Need Answered:**
- ...
```

---

## Step 2: Code Archaeology (20 min)

Now open `app.py` and systematically explore. **Don't try to understand everything** — map the territory first.

### Ask Your AI to Map the Codebase

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Analyze @legacy-orderflow/app.py and create a system map:
1. List all route handlers (endpoints)
2. Identify global state (dictionaries, variables)
3. Find all helper/utility functions
4. Note any constants or configuration

Format as a markdown table for endpoints and a list for helpers/state.
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

From your greenfield repo directory:

```text
Analyze ../legacy-orderflow/app.py and create a system map:
1. List all route handlers (endpoints)
2. Identify global state (dictionaries, variables)  
3. Find all helper/utility functions
4. Note any constants or configuration

Format as a markdown table for endpoints and a list for helpers/state.
```

</details>

### Expected Discovery: It's Bigger Than It Looks

You'll find this is no simple app:

- **40+ endpoints** — way more than the PM described
- **In-memory storage** — no database, just Python dicts
- **Global state everywhere** — users, orders, products, sessions...
- **Mixed concerns** — auth, orders, payments, admin, reports all in one file

{: .warning }
> This is a ~2,500 line Flask monolith. It handles: user registration, authentication, products, shopping cart, orders, payments, refunds, promos, wishlists, reviews, admin panel, reports, and more.

---

## Step 3: Identify Red Flags (10 min)

Hunt for concerning patterns. These are the "integration risks" you'll report to your PM.

### Ask AI to Find Problems

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Analyze @legacy-orderflow/app.py for security and reliability red flags:
1. Authentication/authorization issues
2. Password handling problems
3. Hardcoded secrets or credentials
4. Missing input validation
5. Logging/audit issues
6. Error handling problems

For each finding: what line, what's wrong, how severe (critical/high/medium/low).
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Analyze ../legacy-orderflow/app.py for security and reliability red flags:
1. Authentication/authorization issues
2. Password handling problems
3. Hardcoded secrets or credentials
4. Missing input validation
5. Logging/audit issues
6. Error handling problems

For each finding: what line, what's wrong, how severe (critical/high/medium/low).
```

</details>

### Expected Red Flags (There Are Many)

Your AI should find issues like:

| Category | Finding | Severity |
|----------|---------|----------|
| **Auth** | MD5 password hashing | Critical |
| **Auth** | Admin endpoints without auth | Critical |
| **Secrets** | Hardcoded API keys in source | Critical |
| **Audit** | Only 90% of actions logged | High |
| **Debug** | `/debug/env` leaks all secrets | Critical |
| **Debug** | `/debug/users` exposes password hashes | Critical |

{: .important }
> **Don't panic.** Every legacy system has problems. Your job is to document them, not fix them all today.

---

## Step 4: Save Your Findings (5 min)

Create a legacy analysis document in your greenfield repo:

<details open markdown="block">
<summary><strong>VS Code + GitHub Copilot</strong></summary>

```text
Based on the analysis of @legacy-orderflow/app.py, create a document at 
.specify/legacy-integration/system-analysis.md that includes:
1. System overview (what the app does)
2. Endpoint inventory (table of all routes)
3. Data model (what data structures exist)
4. Red flags (security/reliability issues found)
5. Integration risks (what could break during integration)
```

</details>

<details markdown="block">
<summary><strong>Claude Code / Gemini CLI</strong></summary>

```text
Based on the analysis of ../legacy-orderflow/app.py, create 
.specify/legacy-integration/system-analysis.md that includes:
1. System overview (what the app does)
2. Endpoint inventory (table of all routes)
3. Data model (what data structures exist)
4. Red flags (security/reliability issues found)
5. Integration risks (what could break during integration)
```

</details>

### Commit Your Analysis

```text
Commit the system-analysis.md with message: docs: initial legacy system analysis
```

---

## Success Criteria

- [ ] Legacy app running locally (port 5001)
- [ ] Both repos accessible (your greenfield + legacy-orderflow)
- [ ] Endpoint inventory complete (40+ endpoints documented)
- [ ] At least 5 red flags identified with severity
- [ ] `.specify/legacy-integration/system-analysis.md` created
- [ ] **No code changes made to legacy repo** (exploration only!)

---

## The Part 1 → Part 2 Mindset Shift

| Part 1: Greenfield | Part 2: Brownfield |
|-------------------|-------------------|
| You write the spec | Spec is hidden in the code |
| Full technology control | Inherited tech stack |
| "What should it do?" | "What does it actually do?" |
| Build from scratch | Understand before changing |
| Monday spec → Friday ship | Monday archaeology → careful integration |

**Same spec-first discipline, different starting point.**

---

## Reflection Questions

1. **Scale surprise**: How close was your estimate of "how big is this app?" to reality (40+ endpoints)? What does this tell you about legacy systems?

2. **Red flag severity**: Which red flag concerns you most? How would you prioritize fixing it during integration vs. accepting it temporarily?

3. **Part 1 contrast**: In Part 1, you started with an empty repo. How does starting with 2,500 lines of unknown code change your approach to the first hour?

4. **Stakeholder communication**: If you had to brief your PM on this legacy system in 5 minutes, what would you say?

---

## Key Takeaways

1. **Explore before you read** — Run the app as a user first
2. **Map before you modify** — Understand the structure before changing anything
3. **Document the dangers** — Red flags inform integration strategy
4. **Two repos, one project** — Your code and their code stay separate (for now)

---

## What's Next?

In **Lab 2.1**, you'll write characterization tests that capture the legacy system's actual behavior — creating executable documentation of "how it works today."

**The first rule of legacy code: Don't change behavior you haven't documented.**
