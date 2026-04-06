---
layout: block
title: "Memory & Project Intelligence"
block_number: 5
description: "Teach Claude Code how your team works — once. Memory files, conditional rules, and auto-learning turn a generic AI into your project's expert."
time: "~28 min (8 min presentation + 20 min practical)"
features:
  - Memory Hierarchy
  - ~/.claude/CLAUDE.md
  - Project CLAUDE.md
  - CLAUDE.local.md
  - .claude/rules/
  - Auto Memory
  - /memory
  - "@path imports"
objectives:
  - Understand the three-layer memory hierarchy (user, project, local)
  - Create a user-level CLAUDE.md with personal preferences
  - Write conditional rules that activate only for specific file paths
  - Configure local-only memory for environment-specific details
  - Use /memory to inspect everything Claude loads at session start
  - Trigger and verify auto memory from corrections
---

## The "New Hire" Problem

You've been there. A new developer joins the team. Smart, capable — but for the first two weeks, every pull request comes back with the same comments: "We don't use default exports here." "Our K8s labels follow this convention." "Tests go in `__tests__/`, not `test/`."

Claude Code has the same problem. Out of the box, it's a brilliant generalist who knows nothing about _your_ project's conventions. Every session starts fresh, and you end up repeating yourself.

Memory fixes that. Permanently.

## What We'll Cover

1. **The memory hierarchy** — three layers that stack like CSS specificity: user, project, and local
2. **User-level CLAUDE.md** — your personal preferences that follow you everywhere
3. **Project CLAUDE.md** — team standards committed to version control (you've already seen this one)
4. **CLAUDE.local.md** — your machine-specific secrets and paths, safely gitignored
5. **Conditional rules** — `.claude/rules/` files that activate only when relevant files are touched
6. **Auto memory** — how Claude learns from your corrections without you writing a single config line
7. **The `/memory` command** — X-ray vision into what Claude knows about your project

## Why This Block Matters

Here's the thing about AI tools: the difference between "occasionally useful" and "indispensable" is almost never about the model's intelligence. It's about context. A doctor who doesn't know your medical history can still give generic advice. A doctor with your full chart? That's a different conversation entirely.

After this block, Claude Code will know your team's commit conventions, your testing patterns, your K8s labeling strategy, and your personal coding preferences. Every session. Automatically. Without you saying a word.

## Prerequisites

- Completed Blocks 0-4 (Claude Code installed, ai-coderrank running locally, dark theme implemented)
- The ai-coderrank project open in your terminal
- A willingness to tell Claude how you actually like things done (be honest — it won't judge)

---

## Part 1: Presentation {#presentation}

> **Duration**: ~8 minutes
> **Goal**: Students understand the memory hierarchy, know when to use each layer, and grasp how auto memory works.

---

### Slide 1: The First Thing Claude Reads

Here's a fun fact that changes how you think about Claude Code:

**CLAUDE.md is loaded every single session. It's literally the first thing Claude reads about your project.**

Before it looks at your code. Before it reads your prompt. Before anything — it reads its memory files. Think of it like walking into the office and checking the team wiki before opening Slack.

And here's the kicker: you control what's on that wiki.

---

### Slide 2: The Memory Hierarchy

Claude's memory works in layers, like a cake — each layer adds flavor, and they all combine into the final result.

```
┌─────────────────────────────────────────────┐
│  Layer 3: CLAUDE.local.md                   │
│  Your machine only. Gitignored. Secrets,    │
│  local paths, personal overrides.           │
├─────────────────────────────────────────────┤
│  Layer 2: Project CLAUDE.md + .claude/rules │
│  Committed to git. Shared with team.        │
│  The "team agreement" layer.                │
├─────────────────────────────────────────────┤
│  Layer 1: ~/.claude/CLAUDE.md               │
│  Your personal preferences. Follows you     │
│  across every project on your machine.      │
└─────────────────────────────────────────────┘
```

**Layer 1 — User-level** (`~/.claude/CLAUDE.md`):
Your personal coding DNA. Prefer functional style over classes? Always want commit messages in conventional format? Hate semicolons? This is where that goes. It applies to _every_ project you open.

**Layer 2 — Project-level** (`CLAUDE.md` at the project root + `.claude/rules/`):
Team standards. "We use Tailwind, not CSS modules." "All API routes return a standard error shape." "K8s manifests use label `app.kubernetes.io/name`." This lives in version control — when a teammate pulls, they get the same instructions.

**Layer 3 — Local** (`CLAUDE.local.md`):
Your machine-specific details. The path to your local Kubernetes config. Your staging environment URL. That API key you use for local testing. This file is gitignored by default — it never leaves your machine.

> **Analogy**: Think of it like CSS specificity. User-level is the browser default. Project-level is the stylesheet. Local is the inline style — highest priority, most specific.

---

### Slide 3: The `.claude/rules/` Directory — Conditional Intelligence

This is where things get really clever.

Instead of dumping everything into one giant CLAUDE.md, you can create focused rule files that activate **only when relevant files are being edited**.

```
.claude/
  rules/
    testing.md      # Loads when you edit tests/**
    k8s.md          # Loads when you edit k8s/**
    docker.md       # Loads when you edit Dockerfile*
    api.md          # Loads when you edit src/app/api/**
```

Each file has a path filter at the top:

```markdown
---
path: "k8s/**"
---

When working with Kubernetes manifests:
- Always include resource limits (cpu and memory)
- Use label convention: app.kubernetes.io/name, app.kubernetes.io/component
- Never use `latest` tag for images...
```

**Why this matters**: Claude doesn't need to think about your K8s conventions when you're editing a React component. Conditional rules keep context focused and relevant — less noise, better results.

It's like how a surgeon doesn't review the cafeteria menu before an operation. Right information, right time.

---

### Slide 4: Auto Memory — Claude Learns From You

This one's almost magical.

When you correct Claude during a session — "No, we use `pnpm` here, not `npm`" or "Our tests use `vitest`, not `jest`" — Claude can remember that correction for next time.

**How it works**:
1. You correct Claude on something project-specific
2. Claude recognizes this as a learnable convention
3. It offers to save the insight to your project's memory
4. Next session, it already knows

You can also use the **`/memory`** command at any time to:
- See everything Claude currently has loaded
- Edit memory entries directly
- Add new conventions on the fly

**Pro tip**: After a code review where you gave a lot of feedback, start a Claude session and just say "I want to add some conventions to our project memory." Then list what you told the PR author. Now Claude enforces those conventions automatically.

---

### Slide 5: `@path` Imports — Pointing Claude at References

Sometimes you want Claude to read a specific file as context, even if it wouldn't naturally look there. That's what `@path` does.

In any CLAUDE.md or rules file, you can reference other files:

```markdown
For our API response format, follow the pattern in @src/lib/api-response.ts
For deployment conventions, see @docs/deployment-guide.md
```

Claude will load those referenced files as additional context. It's like saying "before you start, read this."

---

### Slide 6: Putting It All Together

Here's what a well-configured project looks like:

```
~/.claude/
  CLAUDE.md              # "I prefer TypeScript strict mode, conventional commits"

ai-coderrank/
  CLAUDE.md              # "Next.js 14 app, pnpm, Tailwind CSS, team conventions"
  CLAUDE.local.md        # "My staging URL is..., my API key is..."
  .claude/
    rules/
      testing.md         # Testing patterns (path: tests/**)
      k8s.md             # K8s conventions (path: k8s/**)
      docker.md          # Docker best practices (path: Dockerfile*)
```

When you open a session and edit a K8s manifest, Claude loads:
1. Your personal `~/.claude/CLAUDE.md`
2. The project `CLAUDE.md`
3. Your `CLAUDE.local.md`
4. The `k8s.md` rule (because you're touching K8s files)

That's four layers of context, loaded automatically, before you type a single word.

**This is how you turn a general-purpose AI into your team's expert.**

---

### Key Takeaways

| Layer | File | Shared? | Use Case |
|-------|------|---------|----------|
| User | `~/.claude/CLAUDE.md` | No (personal) | Coding style, preferred tools, commit format |
| Project | `CLAUDE.md` | Yes (git) | Team standards, tech stack, conventions |
| Local | `CLAUDE.local.md` | No (gitignored) | Env vars, local paths, secrets |
| Rules | `.claude/rules/*.md` | Yes (git) | Conditional, path-specific conventions |
| Auto | Learned from corrections | Per project | Accumulated knowledge from usage |

> **Transition**: Alright — enough theory. Let's build out the full memory system for ai-coderrank. By the end of the practical, Claude will know this project like a senior engineer who's been on the team for a year.

---

## Part 2: Hands-On {#practical}

> **Duration**: ~20 minutes
> **Outcome**: A complete memory system for ai-coderrank -- user preferences, conditional rules for K8s and testing, local environment config, and auto-learned conventions.
> **Prerequisites**: Completed Blocks 0-4 (Claude Code installed, ai-coderrank running locally, dark theme implemented)

---

### Step 1: Create Your User-Level CLAUDE.md (~3 min)

This file lives outside any project — it's YOUR preferences, applied everywhere.

Create the directory if it doesn't exist, then create the file:

```bash
mkdir -p ~/.claude
```

Now create `~/.claude/CLAUDE.md` with your personal preferences. Here's a template — **customize it to match how you actually work**:

```markdown
# Personal Coding Preferences

## Style
- I prefer functional programming patterns over class-based OOP
- Use TypeScript strict mode whenever possible
- Prefer named exports over default exports
- Use `const` by default, `let` only when reassignment is needed

## Commits
- Follow Conventional Commits format: type(scope): description
- Types: feat, fix, docs, style, refactor, test, chore, ci
- Keep subject line under 72 characters
- Write commit messages in imperative mood ("add feature" not "added feature")

## Communication
- Be concise in explanations — I prefer bullet points over paragraphs
- When showing code changes, always explain WHY not just WHAT
- If something could break production, flag it loudly

## Tools
- I use pnpm as my package manager
- My terminal is zsh with oh-my-zsh
- I use vim keybindings
```

> **Important**: This is YOUR file. Change it to reflect your real preferences. Use npm instead of pnpm? Prefer classes? Like verbose explanations? Make it yours. The whole point is that Claude adapts to you, not the other way around.

---

### Step 2: Review the Project CLAUDE.md (~2 min)

Back in Block 1, when we first explored ai-coderrank, a project-level `CLAUDE.md` was already there. Let's look at it again with fresh eyes, now that we understand what it does.

```bash
cd ~/ai-coderrank
cat CLAUDE.md
```

You should see something like:

```markdown
# CLAUDE.md

This is a Next.js 14 application that compares AI coding models.

## Tech Stack
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS
- Recharts for data visualization
- pnpm as package manager

## Commands
- `pnpm dev` — start development server
- `pnpm build` — production build
- `pnpm test` — run tests
- `pnpm lint` — run ESLint

## Project Structure
- `src/app/` — Next.js app router pages and API routes
- `src/components/` — React components
- `src/lib/` — Utility functions and data
- `k8s/` — Kubernetes manifests
- `argocd/` — ArgoCD application config
```

This file is committed to git. Every developer on the team (and every Claude session) sees it. Think of it as the project's "hello, here's who I am" introduction.

> **Discussion point**: What's missing from this CLAUDE.md? What conventions would you add for your team? Keep this in mind — we'll come back to it.

---

### Step 3: Create Conditional Rules for Testing (~4 min)

Now for the good stuff. Let's create rules that activate only when Claude is working with test files.

Create the rules directory:

```bash
mkdir -p ~/ai-coderrank/.claude/rules
```

Create `.claude/rules/testing.md`:

```markdown
---
path: "**/*.test.*"
---

# Testing Conventions

## Framework
- We use Vitest as our test runner
- React components are tested with @testing-library/react
- API routes are tested with supertest

## Structure
- Test files live next to the code they test: `Component.tsx` -> `Component.test.tsx`
- Use `describe` blocks to group related tests
- Each `it` block should test exactly ONE behavior
- Name tests as sentences: `it('renders the model comparison chart with data')`

## Patterns
- Prefer `screen.getByRole()` over `getByTestId()` for accessibility
- Use `userEvent` over `fireEvent` for realistic user interactions
- Mock external APIs at the fetch level, not at the module level
- Always clean up: if you add to the DOM, make sure it's removed

## Coverage
- New features require tests — no exceptions
- Bug fixes require a regression test that would have caught the bug
- Minimum meaningful coverage, not chasing percentages

## Example Structure
```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { ModelCard } from './ModelCard';

describe('ModelCard', () => {
  it('displays the model name and provider', () => {
    render(<ModelCard model={mockModel} />);
    expect(screen.getByRole('heading')).toHaveTextContent('GPT-4');
    expect(screen.getByText('OpenAI')).toBeInTheDocument();
  });

  it('highlights the top-ranked model with a badge', () => {
    render(<ModelCard model={mockModel} rank={1} />);
    expect(screen.getByRole('status')).toHaveTextContent('#1');
  });
});
```
```

> **What just happened?** The `path: "**/*.test.*"` frontmatter tells Claude: "Only load these instructions when test files are involved." When you're editing a Kubernetes manifest, this file stays out of the way. When you touch a test? It's right there.

---

### Step 4: Create Conditional Rules for Kubernetes (~4 min)

Same concept, but for K8s manifests. This is where domain-specific knowledge really shines.

Create `.claude/rules/k8s.md`:

```markdown
---
path: "k8s/**"
---

# Kubernetes Manifest Conventions

## Labels (Required on All Resources)
Every resource MUST include these labels:
```yaml
metadata:
  labels:
    app.kubernetes.io/name: ai-coderrank
    app.kubernetes.io/component: <component>  # e.g., frontend, api, worker
    app.kubernetes.io/part-of: ai-coderrank
    app.kubernetes.io/managed-by: argocd
```

## Resource Limits (Required on All Containers)
Never deploy a container without resource limits. Use these as starting points:
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

## Security
- Never run containers as root — always set `runAsNonRoot: true`
- Always set `readOnlyRootFilesystem: true` where possible
- Drop all capabilities, add back only what's needed:
```yaml
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

## Images
- NEVER use the `latest` tag — always pin to a specific version
- Use the project's container registry: `ghcr.io/your-org/ai-coderrank`
- Multi-arch images preferred (amd64 + arm64)

## Naming
- Use kebab-case for resource names
- Prefix with `ai-coderrank-`: e.g., `ai-coderrank-deployment`, `ai-coderrank-service`
- Namespace: `ai-coderrank` (don't use `default`)

## Health Checks
Every Deployment must have:
- `livenessProbe` — is the process alive?
- `readinessProbe` — is it ready to receive traffic?
- `startupProbe` for containers with slow initialization
```

> **This is powerful.** Next time you ask Claude to "add a new Kubernetes deployment for a Redis cache," it will automatically include resource limits, security contexts, proper labels, and health checks — because it knows your team's rules.

---

### Step 5: Create CLAUDE.local.md (~3 min)

This file is for things that are true on YOUR machine but shouldn't be shared. Local paths, personal environment details, staging URLs.

Create `CLAUDE.local.md` in the project root:

```markdown
# Local Environment — DO NOT COMMIT

## Environment
- Local K8s cluster: minikube (profile: ai-coderrank)
- Kubeconfig path: ~/.kube/config
- Docker daemon: Docker Desktop on macOS

## Local URLs
- App: http://localhost:3000
- K8s Dashboard: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

## Credentials (for local development only)
- GitHub Container Registry: Use `gh auth token` for authentication
- ArgoCD admin password: see `argocd admin initial-password -n argocd`

## Personal Preferences for This Project
- I'm currently focused on the dark theme feature (Block 4)
- My preferred branch naming: feature/<block-number>-<short-description>
- I review diffs before committing — always show me the diff first
```

Now verify it's gitignored:

```bash
cd ~/ai-coderrank
cat .gitignore | grep -i claude
```

You should see `CLAUDE.local.md` in the gitignore. If it's not there, add it:

```bash
echo "CLAUDE.local.md" >> .gitignore
```

> **Why this matters**: CLAUDE.local.md is where you put things that would be a security incident if they hit GitHub. Local credentials, internal URLs, paths that reveal your username. It's gitignored by default in Claude Code projects, but always double-check.

---

### Step 6: Run `/memory` to See the Full Picture (~2 min)

Now let's see everything Claude has loaded. Start a Claude Code session in the ai-coderrank project:

```bash
cd ~/ai-coderrank
claude
```

Once inside, type:

```
/memory
```

Claude will show you a summary of all loaded memory:
- Your user-level `~/.claude/CLAUDE.md`
- The project-level `CLAUDE.md`
- Your `CLAUDE.local.md`
- Any `.claude/rules/` files and their path filters

You should see both your testing rules and K8s rules listed, with their activation conditions.

> **Try this**: Ask Claude "What do you know about how we write tests in this project?" It should reference your testing conventions from the rules file — even though you never mentioned them in the conversation.

---

### Step 7: Auto Memory — Watch Claude Learn (~2 min)

This is the "whoa" moment. Let's teach Claude something by correcting it.

In your Claude Code session, try this:

```
Show me how to run the linter for this project
```

If Claude suggests `npm run lint` or `yarn lint`, correct it:

```
Actually, we use pnpm in this project. The command is pnpm lint.
```

Claude will recognize this as a learnable convention and may offer to remember it. Accept the suggestion.

Now, **exit the session and start a new one**:

```
/exit
claude
```

In the new session, ask:

```
How do I run the linter?
```

Claude should now respond with `pnpm lint` without being told — it learned from your correction.

> **Behind the scenes**: Auto memory persists corrections to the project memory. Over time, these small corrections add up. After a few weeks of real usage, Claude knows your project as well as someone who's been on the team for months.

---

### Checkpoint

At this point, you should have:

| File | Location | Purpose |
|------|----------|---------|
| `~/.claude/CLAUDE.md` | Home directory | Personal preferences across all projects |
| `CLAUDE.md` | Project root | Team standards (already existed) |
| `CLAUDE.local.md` | Project root | Local env, gitignored |
| `.claude/rules/testing.md` | `.claude/rules/` | Test conventions (activates for test files) |
| `.claude/rules/k8s.md` | `.claude/rules/` | K8s conventions (activates for k8s/ files) |

**Five files. Zero repetition. Every session starts with full context.**

---

### Bonus Challenge

Create one more rule file: `.claude/rules/docker.md` with a path filter for `Dockerfile*`. Include conventions for:
- Multi-stage builds (separate builder and runtime stages)
- Non-root user in the final image
- Ordering `COPY` instructions for optimal layer caching (dependencies first, code second)
- Using `.dockerignore` to keep images small

Then ask Claude to review your existing Dockerfile against these new rules. See what it flags.

---

> **Next up**: In Block 6, we'll take this concept even further. If memory is what Claude _knows_, skills are what Claude can _do_. You'll create custom slash commands that turn complex workflows into one-liners.
