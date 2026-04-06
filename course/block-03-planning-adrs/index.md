---
layout: block
title: "Planning with ADRs & Diagrams"
block_number: 3
description: "Use Plan mode to think before acting, create Architecture Decision Records, and generate Mermaid infrastructure diagrams — all before writing a single line of code."
time: "~35 min (10 min presentation + 25 min practical)"
features:
  - Plan Mode (/plan, Shift+Tab)
  - Edit/Write Tools
  - Structured Output
  - Mermaid Diagrams
objectives:
  - Understand why planning matters even when working with AI
  - Use Plan mode to outline a multi-file theme change before touching code
  - Create an Architecture Decision Record (ADR) that captures the *why* behind a decision
  - Generate Mermaid diagrams for infrastructure topology, traffic flow, and deployment pipeline
  - Review and iterate on a plan before committing to implementation
---

## Measure Twice, Cut Once

Here's a pattern that kills projects: someone opens a file, starts hacking, and three hours later they've changed 47 files and can't remember why they started. AI makes this worse — it's so fast at making changes that you can dig yourself into a hole at superhuman speed.

Plan mode is the antidote. It's Claude Code's way of saying: "Let me think about this first, show you what I'd do, and get your approval before I touch anything."

This block is about discipline. Not the boring kind — the kind that makes you faster. We're going to plan the dark theme change we'll implement in Block 4, and we'll leave behind documentation that future-you will actually thank present-you for writing.

## What We'll Cover

1. **Plan mode** — how Shift+Tab and `/plan` switch Claude from "doer" to "thinker"
2. **Architecture Decision Records** — lightweight docs that capture decisions and their reasoning
3. **Mermaid diagrams** — text-based diagrams that live in your repo and render on GitHub
4. **The dark theme plan** — mapping out every file we'll touch, every variable we'll change, before writing a single line of code

## What You'll Create

By the end of this block, your repo will have four new files:

| File | Purpose |
|------|---------|
| `docs/adr/001-dark-theme.md` | Decision record for the theme change |
| `docs/diagrams/infrastructure.md` | Mermaid diagram of the DO + k3s topology |
| `docs/diagrams/traffic-flow.md` | Mermaid diagram of request routing |
| `docs/diagrams/deployment-pipeline.md` | Mermaid diagram of the CI/CD pipeline |

These aren't homework assignments. They're the kind of artifacts that senior engineers produce before making infrastructure-level changes — and Claude can generate them in minutes.

## Why This Matters for DevOps

If you've ever inherited a project and asked "why is this configured this way?" — only to find zero documentation — you know the pain. ADRs solve that. They're not design docs (those are long and nobody reads them). ADRs are short, opinionated, and answer one question: **why did we make this choice?**

And Mermaid diagrams? They're version-controlled, diff-friendly, and GitHub renders them natively. No more Lucidchart links that expire when someone leaves the company.

## Part 1: Presentation {#presentation}

**Duration**: ~10 minutes
**Goal**: Convince students that planning with AI is a superpower, not a bottleneck. Introduce Plan mode, ADRs, and Mermaid diagrams.

---

### Opening Hook (1 min)

**Say something like:**

> "Raise your hand if you've ever started a 'quick change' that turned into a two-day refactor. Yeah — me too. Now imagine that, but with an AI that can edit 30 files per minute. Without a plan, AI doesn't just help you dig a hole faster — it helps you dig the *wrong* hole faster.
>
> This block is about the single habit that separates people who are productive with AI from people who are just busy with AI: **planning before doing**."

**Key point:** Speed without direction is just chaos. Plan mode turns Claude from a code-generating machine gun into a precision instrument.

---

### Plan Mode — Think Before You Act (3 min)

#### What is Plan mode?

- Two ways to enter it:
  - Type `/plan` in the conversation
  - Press **Shift+Tab** to toggle between Plan and Act modes
- In Plan mode, Claude will **analyze, reason, and outline** — but it will NOT edit files, run commands, or make changes
- Think of it as the difference between a surgeon studying the MRI before picking up the scalpel

#### What you get from Plan mode:

- A list of files that need to change
- The order of operations
- Potential risks or breaking changes
- An opportunity to say "no, actually, let's do it differently"

#### Why it matters:

- **Context is expensive.** Every wrong turn burns tokens and time. A 2-minute plan can save 20 minutes of backtracking.
- **It's a communication tool.** Plan mode output is something you can paste into a PR description, share with your team, or use as a checklist.
- **It catches misunderstandings early.** If Claude's plan doesn't match what you had in mind, you find out *before* it edits 15 files.

**Show on screen:** Toggle Shift+Tab in Claude Code and show how the mode indicator changes.

---

### Architecture Decision Records (3 min)

#### What are ADRs?

> "ADRs are like commit messages for decisions. A commit message tells you *what* changed. An ADR tells you *why*."

- Lightweight markdown documents
- Typically follow a standard template: Title, Status, Context, Decision, Consequences
- Numbered sequentially: `001-dark-theme.md`, `002-add-caching.md`, etc.
- Live in the repo — right next to the code they describe

#### Why ADRs and not a wiki/Confluence/Notion?

- **They're version-controlled.** The decision record evolves with the code.
- **They're discoverable.** `ls docs/adr/` — done. No searching through a wiki.
- **They survive team turnover.** People leave. Repos stay.
- **They're PR-reviewable.** Someone can comment "I disagree with this decision" right in the code review.

#### The template we'll use:

```
# ADR-001: [Title]
- Status: [Proposed / Accepted / Deprecated / Superseded]
- Date: [Date]
- Context: [Why are we making this decision?]
- Decision: [What did we decide?]
- Consequences: [What happens as a result?]
- Alternatives Considered: [What else did we think about?]
```

**Fun fact:** ADRs were popularized by Michael Nygard in 2011. They've since been adopted by teams at GitHub, Spotify, and Shopify. The format is intentionally short — if your ADR is longer than one screen, it's probably a design doc, not a decision record.

---

### Mermaid Diagrams — Diagrams as Code (2 min)

#### What is Mermaid?

- A text-based diagramming language
- You write something like:
  ```
  graph LR
    A[User :30080] --> B[NodePort]
    B --> C[Service]
    C --> D[Pod :3000]
  ```
  ...and it renders as an actual diagram
- GitHub, GitLab, and Notion all render Mermaid natively in markdown files

#### Why Mermaid for infrastructure docs?

- **Diffable.** When you add a new service, the diagram change shows up in the PR diff.
- **No external tools.** No Lucidchart, no draw.io, no "where's the Figma link?"
- **Claude is great at generating them.** Describe your architecture in plain English, get a diagram back. Iterate until it's right.

#### The three diagrams we'll create:

1. **Infrastructure topology** — DigitalOcean Droplet > k3s cluster > pods and services
2. **Traffic flow** — User > NodePort 30080 > Service > Pod :3000 (how a request reaches the app)
3. **Deployment pipeline** — git push > GitHub Actions > Docker build > registry > ArgoCD > k3s

**Show on screen:** A quick example of a Mermaid diagram rendered on GitHub.

---

### The Dark Theme Plan (1 min)

#### Setting up the practical:

> "In the practical, we're going to plan the dark theme change for ai-coderrank. Not implement it — that's Block 4. Just plan it.
>
> Here's what we'll ask Claude to do:
> 1. Enter Plan mode and analyze the current theme system
> 2. Create an ADR explaining *why* we're adding a dark theme
> 3. Generate three Mermaid diagrams documenting our infrastructure
> 4. Review everything and make sure we're happy before we write a line of code
>
> By the end, we'll have a plan we're confident in, documentation that would impress any code reviewer, and a clear roadmap for Block 4."

---

### Closing (30 sec)

> "Planning isn't the opposite of moving fast. Planning *is* moving fast — you just front-load the thinking. Let's go do it."

---

### Common Student Questions

**Q: Can I use Plan mode for everything?**
A: You *can*, but it's most valuable for multi-file changes, architecture decisions, or anything where you'd normally sketch on a whiteboard first. For a quick bug fix? Just ask Claude directly.

**Q: Do ADRs replace design docs?**
A: No. ADRs are for *decisions*. Design docs are for *designs*. An ADR might say "We chose PostgreSQL over MongoDB." A design doc would explain the schema, indexing strategy, and migration plan. ADRs are minutes; design docs are hours.

**Q: Does Mermaid support every kind of diagram?**
A: Most of them — flowcharts, sequence diagrams, class diagrams, state diagrams, Gantt charts, ER diagrams, and more. It doesn't replace specialized tools for complex UML, but for infrastructure and flow diagrams it's excellent.

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: An ADR, three Mermaid diagrams, and a solid implementation plan for the dark theme
> **Prerequisites**: Block 2 completed, Claude Code running inside the ai-coderrank project directory

---

### Step 1: Enter Plan Mode (2 min)

We're going to plan the dark theme change before writing any code. First, let's switch Claude into planning mode.

**In your Claude Code session, press Shift+Tab.**

You'll see the mode indicator change. Claude is now in **Plan mode** — it will analyze and reason but won't edit files or run commands.

> **Alternative:** You can also type `/plan` to enter plan mode.

Now give Claude the big picture:

```
We need to add a dark theme to ai-coderrank. The app currently uses a light theme
with CSS variables and a ThemeProvider component. There's also a scripts/switch-theme.sh
script for theme switching.

Before we implement anything, I want to plan this change. Analyze the current theme
system — look at the Tailwind config, CSS variables, ThemeProvider, and the switch-theme
script. Tell me every file we'll need to touch and what changes each one needs.
```

**What to watch for:**
- Claude will read files like `tailwind.config.ts`, the global CSS file, the ThemeProvider component, and `scripts/switch-theme.sh`
- It will produce a structured plan listing files, changes, and the order of operations
- It should identify CSS variable definitions, Tailwind's dark mode configuration, and component-level styles

**Take a moment to read the plan.** Does it make sense? Does it miss anything? This is your chance to course-correct before any code changes happen.

---

### Step 2: Create the ADR Directory and Document (8 min)

Now we'll ask Claude to create an Architecture Decision Record. Stay in Plan mode for the initial discussion, then switch to Act mode for the file creation.

**Still in Plan mode, type:**

```
I want to create an ADR (Architecture Decision Record) for this dark theme decision.
The ADR should follow this structure:
- Title, Status, Date
- Context: why are we adding dark themes?
- Decision: what approach are we taking?
- Consequences: what are the trade-offs?
- Alternatives Considered: what else did we evaluate?

The context should mention: developer preference, reduced eye strain, industry standard
for dev tools, and that our target audience (developers comparing AI models) likely
prefers dark themes. The decision should reference the existing ThemeProvider and CSS
variable architecture.
```

Claude will draft the ADR content in Plan mode. Review it — suggest edits if needed.

**When you're satisfied with the plan, press Shift+Tab to switch back to Act mode.**

Now tell Claude to create the file:

```
Create the ADR at docs/adr/001-dark-theme.md with the content we just planned.
```

**What to watch for:**
- Claude uses the **Write** tool to create the new file (since it doesn't exist yet)
- It creates the `docs/adr/` directory structure automatically
- The ADR should be clean, well-formatted markdown

**Verify the file was created:**

```
Show me the contents of docs/adr/001-dark-theme.md
```

Your ADR should look something like this (Claude's version will vary):

```markdown
# ADR-001: Add Dark Theme to ai-coderrank

**Status:** Accepted
**Date:** 2026-04-05

## Context

ai-coderrank is a developer-facing dashboard for comparing AI coding models.
Our target audience — developers, DevOps engineers, and engineering managers —
overwhelmingly prefers dark themes in their tooling...

## Decision

We will implement a dark theme using the existing CSS variable architecture
and ThemeProvider component...

## Consequences

### Positive
- Better developer experience, especially for extended use
- Aligns with industry standards for developer tools
...

### Negative
- Increases CSS surface area (two complete color palettes to maintain)
- Chart colors in Recharts need separate dark-optimized palettes
...

## Alternatives Considered

1. **System-preference-only** — Follow OS dark/light setting automatically...
2. **CSS-only toggle without ThemeProvider** — Simpler but less flexible...
3. **Keep light theme only** — Rejected because...
```

---

### Step 3: Generate the Infrastructure Topology Diagram (5 min)

Now let's create Mermaid diagrams that document our infrastructure. These are useful on their own, and they give Claude (and future team members) a visual map of the system.

**Type:**

```
Create a Mermaid diagram at docs/diagrams/infrastructure.md that shows our
infrastructure topology:

- DigitalOcean Droplet (the host machine, public IP)
  - k3s cluster running on the droplet
    - ai-coderrank app (pod with the Next.js container, exposed via NodePort 30080)
    - ArgoCD (pod for GitOps deployment)
    - System components (CoreDNS, Traefik, metrics-server)
- External: GitHub (source repo), GHCR (container registry)

Use a top-down graph. Include a brief explanation above the diagram of what
it shows. Wrap the Mermaid code in a ```mermaid fenced code block so GitHub
renders it automatically.
```

**What to watch for:**
- Claude creates the `docs/diagrams/` directory
- The Mermaid syntax should use `graph TD` (top-down) or `graph TB` (top-bottom)
- Nodes should have descriptive labels
- GitHub will render the diagram automatically when you view the file

**If the diagram isn't quite right, iterate:**

```
Can you add the DigitalOcean floating IP as the entry point, and group the k3s
components inside a subgraph?
```

This is one of Claude's strengths — iterating on diagrams is much faster than dragging boxes around in a GUI tool.

---

### Step 4: Generate the Traffic Flow Diagram (4 min)

**Type:**

```
Create a Mermaid diagram at docs/diagrams/traffic-flow.md that shows how a user
request reaches the ai-coderrank app:

1. User's browser hits http://DROPLET_IP:30080
2. DigitalOcean droplet receives traffic on NodePort 30080
3. k3s kube-proxy routes to the Kubernetes Service
4. Service forwards to ai-coderrank Pod on port 3000
5. Next.js serves the response back through the same chain

Use a left-to-right flow diagram. Include port numbers at each hop.
Add a brief description above the diagram.
```

**What to watch for:**
- The diagram should use `graph LR` for left-to-right flow
- It should clearly show the request path with port numbers at each hop
- NodePort 30080 is the external entry point, container port 3000 is internal

**Iterate if needed:**

```
Add a note showing that this is the simplest exposure method — no DNS, no TLS,
no Ingress needed. Just the droplet's public IP and the NodePort.
```

---

### Step 5: Generate the Deployment Pipeline Diagram (4 min)

**Type:**

```
Create a Mermaid diagram at docs/diagrams/deployment-pipeline.md that shows
the deployment pipeline:

1. Developer pushes to GitHub (main branch)
2. GitHub Actions triggers CI workflow
3. CI runs: lint, test, build
4. Docker image is built and pushed to container registry
5. ArgoCD detects the change (watches the repo or registry)
6. ArgoCD syncs the new image to the k3s cluster
7. k3s performs a rolling update of the ai-coderrank pod

Use a top-down flow. Color-code or use different shapes for: developer actions,
CI/CD automation, and Kubernetes operations. Add a description above the diagram.
```

**What to watch for:**
- Different node shapes for different concerns (rectangles for steps, diamonds for decisions, rounded for start/end)
- Clear separation between the CI phase and the CD phase
- ArgoCD's role as the bridge between "code pushed" and "app deployed"

**Iterate if needed:**

```
Can you add a decision diamond after the CI step that shows "Tests pass?" with
a Yes path to Docker build and a No path to "Notify developer, stop pipeline"?
```

---

### Step 6: Review Everything (2 min)

Let's take stock of what we've created. Ask Claude to give you a summary:

```
List all the files we created in this session with a one-line description of each.
Then give me a quick summary of the dark theme implementation plan — which files
will we change in Block 4 and in what order?
```

**You should see four files:**

| File | Description |
|------|-------------|
| `docs/adr/001-dark-theme.md` | Architecture Decision Record for the dark theme |
| `docs/diagrams/infrastructure.md` | Mermaid diagram of the DO/k3s infrastructure |
| `docs/diagrams/traffic-flow.md` | Mermaid diagram of request routing |
| `docs/diagrams/deployment-pipeline.md` | Mermaid diagram of the CI/CD pipeline |

**Optional — preview the diagrams:**

If you want to see the diagrams rendered, you can:
- Push the files to GitHub and view them in the browser (GitHub renders Mermaid natively)
- Use a VS Code extension like "Markdown Preview Mermaid Support"
- Paste the Mermaid code into [mermaid.live](https://mermaid.live) for a live preview

---

### Checkpoint

Before moving to Block 4, verify:

- [ ] `docs/adr/001-dark-theme.md` exists and has a complete ADR with Context, Decision, Consequences, and Alternatives
- [ ] `docs/diagrams/infrastructure.md` has a Mermaid diagram showing the DO droplet, k3s, and all pods
- [ ] `docs/diagrams/traffic-flow.md` has a Mermaid diagram showing the request path from user to pod
- [ ] `docs/diagrams/deployment-pipeline.md` has a Mermaid diagram showing the full CI/CD pipeline
- [ ] You have a clear mental model of which files will change for the dark theme (from the Plan mode output)
- [ ] You understand the difference between Plan mode and Act mode

---

### Key Takeaways

1. **Plan mode is your pre-flight checklist.** Use it for any change that touches more than 2-3 files. The few minutes you spend planning will save you from wrong turns.

2. **ADRs are surprisingly easy with Claude.** What normally takes 30 minutes of writing takes 2 minutes of prompting and reviewing. The hard part — capturing the *reasoning* — is exactly what Claude is good at.

3. **Mermaid diagrams are living documentation.** They live in your repo, they're diffable, and GitHub renders them. No more broken Lucidchart links.

4. **Iteration is the secret.** Your first prompt won't produce the perfect output. The second or third refinement will. This is normal and expected — it's how working with AI is supposed to feel.

---

### What's Next

In Block 4, we'll take this plan and *execute* it. Claude will edit Tailwind config, CSS variables, and component styles to implement the dark theme. You'll see the Edit tool in action — surgical, precise file modifications — and you'll learn how to iterate on visual changes with Claude until the result looks exactly right.

The plan is done. Time to build.
