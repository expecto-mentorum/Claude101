---
layout: block
title: "Custom Skills — Your Team's Playbook"
block_number: 6
description: "Build reusable slash commands that encode your team's best practices — Kubernetes reviews, Dockerfile audits, and one-command explanations."
time: "~33 min (8 min presentation + 25 min practical)"
features:
  - Custom Skills (.claude/skills/)
  - SKILL.md Frontmatter
  - "$ARGUMENTS"
  - "/simplify"
  - allowed-tools
objectives:
  - Understand skills as reusable, parameterized AI workflows
  - Create a K8s manifest review skill with allowed-tools restrictions
  - Create a Dockerfile audit skill that checks for best practices
  - Build a parameterized skill using $ARGUMENTS
  - Use the built-in /simplify skill on existing code
  - Know the difference between restricted and unrestricted skills
---

## From Knowledge to Action

Block 5 taught Claude what your team _knows_. This block teaches Claude what your team _does_.

Think about the playbooks that exist in every team — usually buried in a wiki page nobody reads or in the head of that one senior engineer who's been here since the beginning. "Before you deploy a new service, check these twelve things." "When reviewing Dockerfiles, look for these five antipatterns." "If a K8s pod keeps restarting, run these commands in this order."

Skills turn those playbooks into single commands. `/review-k8s` instead of a 15-point mental checklist. `/check-docker` instead of "ask Sarah, she knows what to look for." `/explain main.go` instead of reading code for 20 minutes before understanding what it does.

## What We'll Cover

1. **What skills are** — reusable slash commands defined in `.claude/skills/`
2. **SKILL.md anatomy** — frontmatter, instructions, and the `$ARGUMENTS` variable
3. **Built-in skills** — `/simplify`, and what ships out of the box
4. **Two custom skills from scratch** — a K8s reviewer and a Dockerfile auditor
5. **Parameterized skills** — passing arguments to make skills flexible
6. **`allowed-tools` restrictions** — controlling what a skill can and cannot do

## Why This Block Matters

Here's a question: how do you scale expertise?

You can write documentation (nobody reads it). You can do pair programming (doesn't scale). You can set up linters (catches syntax, not architecture). You can hope people learn from code reviews (takes months).

Or you can encode your expertise into a skill that anyone on the team can run with a single slash command. A junior engineer runs `/review-k8s` and gets the same review quality as your most experienced SRE. A contractor runs `/check-docker` and catches the same issues your lead DevOps engineer would.

Skills are how tribal knowledge becomes institutional knowledge — and they're shockingly easy to create.

## Prerequisites

- Completed Blocks 0-5 (memory system configured)
- The ai-coderrank project with K8s manifests and Dockerfile
- Familiarity with the `.claude/` directory structure from Block 5

---

## Part 1: Presentation {#presentation}

> **Duration**: ~8 minutes
> **Goal**: Students understand what skills are, how they're structured, and why they're a game-changer for team consistency.

---

### Slide 1: Shell Aliases for AI

You know shell aliases, right?

```bash
alias k="kubectl"
alias gs="git status"
alias deploy="./scripts/deploy.sh --env staging --notify slack"
```

One short command that expands into something longer and more complex. Skills are the same idea, but for AI workflows.

```
/review-k8s     →  Read all K8s manifests, check against best practices,
                    report issues with severity and fix suggestions

/check-docker   →  Audit the Dockerfile for security, performance,
                    and build optimization issues

/explain app.ts →  Read the file, explain what it does in plain English,
                    note any non-obvious patterns
```

The difference? Shell aliases run commands. Skills run _thinking_. They give Claude a structured set of instructions, optional constraints, and let it apply its reasoning to your specific codebase.

---

### Slide 2: Anatomy of a Skill

Every skill lives in `.claude/skills/<skill-name>/SKILL.md`. Here's the structure:

```
.claude/
  skills/
    review-k8s/
      SKILL.md          ← This is the skill definition
    check-docker/
      SKILL.md
    explain/
      SKILL.md
```

And here's what a SKILL.md looks like inside:

```markdown
---
name: review-k8s
description: Reviews Kubernetes manifests for best practices and security issues
allowed-tools:
  - Read
  - Glob
  - Grep
---

You are a Kubernetes security and best-practices reviewer.

Read all YAML files in the k8s/ directory. For each manifest, check:
1. Resource limits are set (cpu and memory)
2. Security context is configured (non-root, read-only root filesystem)
3. Health checks are present (liveness, readiness)
...

Output a report with: file name, issue, severity, suggested fix.
```

Three parts:
1. **Frontmatter** — name, description, and optional tool restrictions
2. **Instructions** — what Claude should do when the skill is invoked
3. **That's it.** No build step, no compilation, no registration. Drop the file in and it works.

---

### Slide 3: The `$ARGUMENTS` Variable

Some skills need input. "Explain _this_ file." "Review _this_ PR." "Generate tests for _this_ component."

That's what `$ARGUMENTS` does. It's a placeholder in your skill instructions that gets replaced with whatever the user types after the slash command.

```markdown
---
name: explain
description: Explains any file in plain English
---

Read the file at $ARGUMENTS.

Explain what this file does in plain, jargon-free English. Structure:
1. **Purpose**: One sentence — what problem does this file solve?
2. **How it works**: Walk through the logic step by step
3. **Key patterns**: Note any design patterns or non-obvious techniques
4. **Dependencies**: What does this file rely on?
```

Usage:

```
/explain src/app/api/models/route.ts
/explain k8s/deployment.yaml
/explain Dockerfile
```

The `$ARGUMENTS` gets replaced with whatever follows the command. Simple, flexible, powerful.

> **Tip**: You can reference `$ARGUMENTS` multiple times in the same skill. For example, a skill that reads a file AND its tests: "Read `$ARGUMENTS` and look for a corresponding test file."

---

### Slide 4: Built-in Skills

Claude Code ships with some skills out of the box. The one we'll use today:

**`/simplify`** — Reviews your changed code and looks for:
- Opportunities to reuse existing utilities instead of reimplementing
- Code quality improvements
- Performance optimizations
- Then automatically fixes what it finds

This is perfect for running after a big feature implementation (like our dark theme from Block 4). You write the code to make it work, then `/simplify` makes it clean.

Other useful built-in skills:
- **`/batch`** — Makes the same change across many files (rename a variable everywhere, update an import path, change a pattern)
- **`/debug`** — Activates a troubleshooting mode with enhanced diagnostic output

---

### Slide 5: `allowed-tools` — The Safety Net

Here's where skills get interesting from a safety perspective.

By default, a skill can use any tool Claude has access to — read files, write files, run commands, the works. But sometimes you want a skill to be **read-only**. A review skill shouldn't modify anything. An explanation skill shouldn't create files.

That's what `allowed-tools` does:

```yaml
---
name: review-k8s
allowed-tools:
  - Read
  - Glob
  - Grep
---
```

This skill can read files, search for patterns, and list files — but it **cannot** write, edit, or execute commands. It's physically unable to make changes. This is great for:

- **Review skills** — look but don't touch
- **Audit skills** — report problems but let the human decide the fix
- **Explanation skills** — read and explain, nothing else

Compare that to an unrestricted skill:

```yaml
---
name: fix-k8s
# No allowed-tools = unrestricted
---
```

This skill can make changes, run kubectl, modify files — the full toolkit. Use this for skills that need to _do_ things, not just report.

> **Rule of thumb**: If a skill's purpose is to _tell you something_, restrict its tools. If its purpose is to _do something_, leave it unrestricted.

---

### Slide 6: Project vs. User Skills

Just like memory, skills come in two flavors:

**Project skills** (`.claude/skills/` inside your repo):
- Committed to git, shared with the whole team
- Anyone who clones the repo gets the skills
- Great for team playbooks and project-specific workflows

**User skills** (`~/.claude/skills/`):
- Personal skills that follow you across projects
- Your custom productivity tools
- Things like "summarize this PR" or "write a commit message in my style"

```
~/.claude/
  skills/
    my-commit/SKILL.md    ← Personal, everywhere
    daily-summary/SKILL.md

ai-coderrank/
  .claude/
    skills/
      review-k8s/SKILL.md  ← Team, this project only
      check-docker/SKILL.md
```

When you type `/` in Claude Code, you'll see both project and personal skills in the autocomplete list.

---

### Key Takeaways

| Concept | What It Is | When to Use |
|---------|-----------|-------------|
| SKILL.md | Skill definition file | Always — it's the only required file |
| `$ARGUMENTS` | Placeholder for user input | When the skill needs a target (file, path, name) |
| `allowed-tools` | Restrict available tools | Review/audit skills that shouldn't modify anything |
| Project skills | `.claude/skills/` in repo | Team playbooks, shared workflows |
| User skills | `~/.claude/skills/` | Personal productivity tools |
| `/simplify` | Built-in cleanup skill | After implementing a feature |

> **Transition**: Time to build. You're about to create three custom skills, test them on real infrastructure code, and run `/simplify` on the dark theme you built in Block 4. Let's go.

---

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: Three custom skills (K8s reviewer, Dockerfile auditor, file explainer) plus a test drive of the built-in `/simplify` on your dark theme code.
> **Prerequisites**: Completed Blocks 0-5 (memory system configured)

---

### Step 1: Create the K8s Review Skill (~5 min)

This is our first custom skill — a read-only reviewer that checks Kubernetes manifests against best practices. It can look at your code but cannot modify anything.

Create the skill directory and file:

```bash
mkdir -p ~/ai-coderrank/.claude/skills/review-k8s
```

Create `.claude/skills/review-k8s/SKILL.md` with the following content:

```markdown
---
name: review-k8s
description: Reviews Kubernetes manifests for best practices, security, and operational readiness
allowed-tools:
  - Read
  - Glob
  - Grep
---

You are a senior SRE reviewing Kubernetes manifests before they go to production. Be thorough but practical — flag real issues, not theoretical nitpicks.

## Instructions

1. Use Glob to find all YAML files in the `k8s/` directory
2. Read each manifest file
3. Check every resource against the following criteria

## Checklist

### Resource Management
- [ ] CPU and memory requests are set
- [ ] CPU and memory limits are set
- [ ] Requests are lower than or equal to limits
- [ ] Resource values are reasonable (not 10 CPU cores for a web server)

### Security
- [ ] `runAsNonRoot: true` is set in the security context
- [ ] `readOnlyRootFilesystem: true` where applicable
- [ ] Capabilities are dropped (`drop: ["ALL"]`)
- [ ] No privileged containers
- [ ] No host networking or host PID

### Health & Reliability
- [ ] Liveness probe is configured
- [ ] Readiness probe is configured
- [ ] Startup probe for slow-starting containers
- [ ] Pod disruption budget exists for critical services
- [ ] Replica count is greater than 1 for production services

### Image Hygiene
- [ ] No `latest` tag — images are pinned to a specific version
- [ ] Image pull policy is appropriate (IfNotPresent for tagged, Always for mutable)
- [ ] Images come from an expected registry

### Labels & Metadata
- [ ] Standard Kubernetes labels are present (app.kubernetes.io/name, etc.)
- [ ] Labels are consistent across related resources (Deployment, Service, etc.)
- [ ] Namespace is explicitly set (not relying on `default`)

### Networking
- [ ] Services use appropriate type (ClusterIP for internal, LoadBalancer/Ingress for external)
- [ ] Port names follow convention (http, https, grpc, metrics)
- [ ] Network policies exist if the cluster uses them

## Output Format

For each file reviewed, produce:

### `<filename>`
**Overall**: PASS / WARN / FAIL

| Issue | Severity | Details | Suggested Fix |
|-------|----------|---------|---------------|
| ... | Critical/Warning/Info | What's wrong | How to fix it |

End with a **Summary** section: total files reviewed, critical issues, warnings, and a recommended priority order for fixes.
```

> **Notice the `allowed-tools` section.** This skill can only Read, Glob, and Grep. It cannot edit files, run commands, or make any changes. This is a reviewer, not a fixer — it tells you what's wrong and lets you decide what to do about it.

---

### Step 2: Create the Docker Audit Skill (~5 min)

Same pattern — a read-only skill that audits Dockerfiles for common issues.

```bash
mkdir -p ~/ai-coderrank/.claude/skills/check-docker
```

Create `.claude/skills/check-docker/SKILL.md`:

```markdown
---
name: check-docker
description: Audits Dockerfiles for security, performance, and build optimization
allowed-tools:
  - Read
  - Glob
  - Grep
---

You are a Docker build optimization expert reviewing a Dockerfile before it ships.

## Instructions

1. Read the `Dockerfile` in the project root
2. Check for a `.dockerignore` file — read it if it exists
3. Evaluate the Dockerfile against every criterion below
4. Check that `.dockerignore` excludes common unnecessary files

## Dockerfile Checklist

### Build Stages
- [ ] Uses multi-stage builds (separate build and runtime stages)
- [ ] Build dependencies don't leak into the final image
- [ ] Final stage uses a minimal base image (alpine, distroless, or slim)
- [ ] Base image is pinned to a specific version (not just `node:20`, but `node:20.11-alpine`)

### Layer Optimization
- [ ] Package manager install + cleanup happen in the same RUN instruction
- [ ] `COPY package*.json` (or equivalent) happens BEFORE `COPY . .` for layer caching
- [ ] Dependencies are installed before application code is copied
- [ ] Unnecessary files are not copied into the image

### Security
- [ ] Final image runs as non-root user (`USER node` or custom user)
- [ ] No secrets, API keys, or credentials in the Dockerfile or build args
- [ ] `apt-get update && apt-get install` are combined and cleaned up
- [ ] No `sudo` or `chmod 777` in the Dockerfile

### Runtime
- [ ] `EXPOSE` matches the actual port the application listens on
- [ ] `HEALTHCHECK` instruction is present (or documented why it's omitted)
- [ ] Entrypoint uses exec form (`["node", "server.js"]`) not shell form
- [ ] Signal handling is correct (PID 1 problem addressed)

### .dockerignore
- [ ] `.dockerignore` exists
- [ ] Excludes: `node_modules/`, `.git/`, `*.md`, `.env*`, `.vscode/`, `coverage/`
- [ ] Excludes K8s and ArgoCD config (not needed in the image)

## Output Format

### Dockerfile Audit Report

**Image**: `<base image detected>`
**Stages**: `<number of stages>`
**Final image size estimate**: `<estimate based on base image + dependencies>`

| Category | Status | Finding | Recommendation |
|----------|--------|---------|----------------|
| Build Stages | PASS/WARN/FAIL | ... | ... |
| Layer Caching | PASS/WARN/FAIL | ... | ... |
| Security | PASS/WARN/FAIL | ... | ... |
| Runtime | PASS/WARN/FAIL | ... | ... |
| .dockerignore | PASS/WARN/FAIL | ... | ... |

**Top 3 Changes That Would Help Most:**
1. ...
2. ...
3. ...
```

---

### Step 3: Test `/review-k8s` on the Project (~3 min)

Time to see your first custom skill in action. Start Claude Code in the ai-coderrank project:

```bash
cd ~/ai-coderrank
claude
```

Now run your skill:

```
/review-k8s
```

Claude will:
1. Discover all YAML files in `k8s/`
2. Read each one
3. Evaluate them against your checklist
4. Produce a structured report with findings and severity ratings

Pay attention to what it finds. Common issues in the ai-coderrank manifests:
- Missing resource limits
- No security context
- Missing health check probes
- `latest` image tags

> **Key insight**: You just ran a comprehensive K8s review with a single command. A junior engineer who has never written a Kubernetes manifest can run this and get senior-level feedback. That's the power of skills.

---

### Step 4: Test `/check-docker` on the Dockerfile (~3 min)

Still in your Claude Code session:

```
/check-docker
```

Watch Claude audit the Dockerfile systematically. It should produce a report covering:
- Whether multi-stage builds are used
- Layer caching effectiveness
- Security posture (non-root user, no secrets)
- The state of `.dockerignore`

Compare the output to what you'd check manually. Anything it caught that you might have missed?

---

### Step 5: Create a Parameterized Skill with `$ARGUMENTS` (~5 min)

Now let's build a flexible skill that takes input — an explainer that works on any file you point it at.

```bash
mkdir -p ~/ai-coderrank/.claude/skills/explain
```

Create `.claude/skills/explain/SKILL.md`:

```markdown
---
name: explain
description: Explains any file in plain, jargon-free English
allowed-tools:
  - Read
  - Glob
  - Grep
---

Read the file at `$ARGUMENTS`.

If the path doesn't exist, use Glob to search for likely matches and ask the user to confirm.

## Your Task

Explain this file as if you're talking to a smart developer who just joined the team and has never seen this codebase before.

## Structure Your Explanation As:

### What This File Does
One sentence. No jargon. What problem does it solve?

### How It Works — Step by Step
Walk through the file's logic from top to bottom. For each significant section:
- What it does
- Why it does it that way (if non-obvious)
- What would break if you removed it

### Key Patterns to Know
- Design patterns used (and why)
- Non-obvious conventions
- Anything a new developer might find surprising

### Connections
- What files import or depend on this file?
- What does this file import or depend on?
- If you changed this file, what else might need updating?

### Watch Out For
- Common mistakes when editing this file
- Edge cases it handles (or doesn't)
- Performance considerations

Keep the language conversational. Use analogies where they help. If something is genuinely complex, say so — but explain it anyway.
```

Now test it in your Claude Code session:

```
/explain src/app/page.tsx
/explain k8s/deployment.yaml
/explain Dockerfile
```

Each invocation replaces `$ARGUMENTS` with the path you provided. Same skill, different targets. Notice how the explanations adapt to the file type — a React component gets explained differently than a K8s manifest.

---

### Step 6: Run `/simplify` on the Dark Theme Changes (~3 min)

Remember the dark theme you implemented in Block 4? Let's see if `/simplify` can clean it up.

In your Claude Code session:

```
/simplify
```

`/simplify` is a built-in skill that:
1. Looks at your recent code changes
2. Identifies opportunities for reuse (are you reimplementing something that already exists in the codebase?)
3. Spots code quality improvements
4. Finds performance optimizations
5. Applies the fixes automatically

Watch what it finds in your dark theme implementation. Common things `/simplify` catches:
- Duplicate color values that should be CSS variables or Tailwind theme extensions
- Conditional class logic that could use `clsx` or `cn()` more cleanly
- Components that could share a common base style
- Theme toggle logic that could be simplified with a custom hook

Review each change it proposes. Accept the ones you agree with.

> **When to use `/simplify`**: After finishing any feature implementation. Write code to make it work, then `/simplify` to make it clean. It's like having a code reviewer who only cares about craft — no ego, no style wars, just "here's how this code could be better."

---

### Step 7: Understand `allowed-tools` Restrictions (~1 min)

Let's make the difference crystal clear.

**Restricted skill** (our review-k8s):
```yaml
allowed-tools:
  - Read
  - Glob
  - Grep
```
- Can read files, search patterns, list files
- CANNOT edit files, write files, or run bash commands
- Perfect for: reviews, audits, explanations, reports

**Unrestricted skill** (no `allowed-tools` in frontmatter):
```yaml
---
name: fix-k8s-issues
description: Fixes common K8s manifest issues automatically
---
```
- Can use ALL tools — read, write, edit, run commands
- Can make changes directly to your files
- Perfect for: automated fixes, code generation, refactoring

**The decision is simple**: If a skill only needs to _observe and report_, restrict it. If it needs to _make changes_, leave it unrestricted. When in doubt, start restricted — you can always loosen it later.

Try running `/review-k8s` and then asking Claude to "fix issue #1 from the report" in the same session. The skill will have finished, and Claude (now back in regular mode, not skill mode) can make the fix with full tool access.

---

### Checkpoint

Your `.claude/skills/` directory should now look like this:

```
.claude/
  skills/
    review-k8s/
      SKILL.md          ← K8s best practices reviewer (restricted)
    check-docker/
      SKILL.md          ← Dockerfile auditor (restricted)
    explain/
      SKILL.md          ← File explainer with $ARGUMENTS (restricted)
```

You've also used the built-in `/simplify` skill on real code. These four commands now live in your project — any teammate who pulls the repo gets them for free.

---

### Bonus Challenges

**Challenge 1: Create a `/generate-test` skill**
Build an unrestricted skill that takes a file path via `$ARGUMENTS`, reads the file, and generates a test file following the testing conventions from your Block 5 `.claude/rules/testing.md`. It should create the test file in the right location and use the right framework.

**Challenge 2: Create a personal skill**
Create a skill in `~/.claude/skills/` (not in the project) that does something you personally find useful. Ideas:
- `/tldr` — summarizes any file in 3 bullet points
- `/review-pr` — reviews the current branch's diff against main
- `/standup` — generates a standup update from your recent git commits

**Challenge 3: Chain skills**
Run `/review-k8s`, then use the findings to manually ask Claude to fix the critical issues. Compare the before and after by running `/review-k8s` again. The second report should show improvements.

---

> **Next up**: In Block 7, we take Claude Code beyond the codebase and into real infrastructure — Docker builds, Kubernetes clusters, and SSH into live servers. Claude doesn't just write code; it operates systems.
