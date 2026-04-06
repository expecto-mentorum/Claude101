---
layout: block
title: "Understanding Your Codebase"
block_number: 1
description: "Let Claude analyze the project, generate CLAUDE.md, and explore the architecture through conversation."
time: "~28 min (8 min presentation + 20 min practical)"
features:
  - /init
  - CLAUDE.md
  - Read Tool
  - Grep Tool
  - Glob Tool
objectives:
  - Generate a CLAUDE.md file using /init and understand its purpose
  - Explore the ai-coderrank architecture through conversational queries
  - Understand how Claude uses Read, Grep, and Glob tools under the hood
  - Navigate API routes, component structure, and theme switching through dialogue
  - Learn the difference between conversational exploration and direct commands
---

## From "What Is This?" to "I Know This Codebase"

You've joined a new team. There's a repo with 50 files, no documentation, and a Slack message that says "check out the code and get up to speed." Sound familiar?

This is where Claude Code transforms from "interesting tool" to "how did I ever work without this." Instead of spending hours clicking through files, reading imports, and mentally mapping the architecture, you have a conversation. You ask questions. Claude reads the code and gives you answers grounded in what's actually there — not what someone remembered to document six months ago.

## What We'll Cover

1. **CLAUDE.md** — what it is, why it matters, and how `/init` generates it automatically
2. **How Claude reads code** — the Read, Grep, and Glob tools working behind the scenes
3. **Conversational exploration** — asking Claude to explain architecture, data flow, and design decisions
4. **The ai-coderrank deep dive** — understanding every layer of the project through dialogue

## Why CLAUDE.md Matters

Every project has tribal knowledge — conventions, gotchas, "we do it this way because..." reasons that live in people's heads. CLAUDE.md captures that knowledge in a format Claude Code reads automatically at the start of every session.

Think of it this way:
- **README.md** is for humans joining the team
- **CLAUDE.md** is for Claude joining the team

When you run `/init`, Claude analyzes your entire codebase and generates a CLAUDE.md that captures the project structure, tech stack, conventions, and important patterns. You can (and should) edit it over time as you learn more about the project — but the auto-generated version is a remarkably good starting point.

## How Claude Reads Code

Claude Code doesn't just have access to your files — it has specialized tools for different kinds of exploration:

| Tool | What it does | Analogy |
|------|-------------|---------|
| **Read** | Opens and reads a specific file | `cat filename` |
| **Grep** | Searches for patterns across files | `rg "pattern"` |
| **Glob** | Finds files matching a pattern | `find . -name "*.ts"` |

When you ask Claude "how does the theme switching work?", it doesn't guess. It uses Grep to find theme-related code, Read to examine the relevant files, and Glob to check if there are related config files. You can watch these tool calls happen in real time during your session.

## Part 1: Presentation {#presentation}

**Target duration**: ~8 minutes
**Tone**: "Let me show you something that changed how I onboard to new projects."

---

### Slide 1: The Onboarding Problem (1.5 min)

**Talking points:**

> "How long does it take you to really understand a new codebase? Not just 'I can find the main file' — but truly understand the architecture, the patterns, the why behind the design decisions?
>
> For most of us, it's days. Sometimes weeks. You read files, follow imports, check git blame, ask teammates questions, read outdated documentation...
>
> What if you could have a 20-minute conversation instead?"

**Key point**: Claude Code turns codebase exploration from a solo scavenger hunt into a guided tour.

---

### Slide 2: How Claude Actually Reads Code (2.5 min)

**Talking points:**

- Claude Code isn't magic — it uses concrete tools to interact with your filesystem
- Three core exploration tools:
  - **Read** — opens a file and reads its contents (like `cat` but smarter — it can read specific line ranges)
  - **Grep** — searches for patterns across your codebase (powered by ripgrep, so it's fast)
  - **Glob** — finds files matching patterns (like `find` but more ergonomic)
- When you ask a question, Claude decides which tools to use and in what order
- It chains them together: "Find all TypeScript files" (Glob) -> "Search for 'theme' in those files" (Grep) -> "Read the most relevant file" (Read)

**Analogy**: "It's like watching a senior developer orient themselves in a new codebase. They don't read every file top to bottom. They search for patterns, follow the trail, and build a mental model. Claude does exactly that — except it can do it in seconds."

> "Watch the tool calls in your terminal. You'll see lines like 'Reading src/app/layout.tsx' or 'Searching for pattern: theme'. That's not decoration — that's Claude actively exploring your code to answer your question."

---

### Slide 3: CLAUDE.md — Project Memory (2.5 min)

**Talking points:**

- CLAUDE.md is a special file that Claude Code reads automatically at the start of every session
- Think of it as a README, but for your AI pair programmer
- It contains: project overview, tech stack, file structure, conventions, common commands, gotchas
- The `/init` command auto-generates one by analyzing your codebase

> "Here's the mental model I want you to have. README.md is the onboarding doc for a new human teammate. CLAUDE.md is the onboarding doc for Claude. And just like a good README, a good CLAUDE.md gets better over time as you add to it."

**What goes in CLAUDE.md:**

- Project description and purpose
- Tech stack and key dependencies
- Directory structure overview
- Build/test/deploy commands
- Coding conventions and patterns
- Known quirks and gotchas

**Important nuance**: `/init` gives you a great starting point, but the real power comes from editing it yourself over time. When you discover that "oh, we always use named exports in this project" or "the API routes follow this specific pattern" — add it to CLAUDE.md. Every future session benefits.

---

### Slide 4: Conversational Exploration vs. Direct Commands (1 min)

**Talking points:**

Two modes of working with Claude Code:

1. **Conversational**: Ask open-ended questions
   - "Explain the architecture of this project"
   - "How does data flow from the API to the charts?"
   - "What testing strategy does this project use?"

2. **Direct**: Give specific commands
   - "Read the file src/app/api/models/route.ts and explain what it does"
   - "Find all files that import the ModelCard component"
   - "Search for TODO comments in the codebase"

> "Both are valid. Conversational is great for exploration — when you don't know what you don't know. Direct is great when you have a specific question. You'll develop an instinct for which to use."

---

### Slide 5: Transition to Practical (0.5 min)

**Talking points:**

> "Enough theory. Let's run `/init` on ai-coderrank and see what Claude discovers about our project. Then we'll have a real conversation about the architecture — API routes, component structure, theme switching, all of it. Switch to your terminal."

## Part 2: Hands-On {#practical}

> **Duration**: ~20 minutes
> **Outcome**: A generated CLAUDE.md file and a thorough understanding of the ai-coderrank architecture gained through conversational exploration
> **Prerequisites**: Block 0 completed, Claude Code installed and authenticated

---

### Step 1: Launch Claude Code in the Project

Make sure you're in the ai-coderrank directory and start a session:

```bash
cd ai-coderrank
claude
```

You should see the Claude Code prompt. If you completed Block 0, this should feel familiar.

---

### Step 2: Generate CLAUDE.md with /init

This is the first "wow" moment. Type:

```
/init
```

Watch what happens. Claude will:

1. Scan your entire project structure
2. Read key files like `package.json`, `tsconfig.json`, `Dockerfile`, etc.
3. Analyze the directory layout
4. Generate a `CLAUDE.md` file in the project root

This takes 15-30 seconds depending on project size. For ai-coderrank, it should be quick.

> **What to say on video**: "Watch the tool calls scrolling by. Claude is reading package.json to understand dependencies, looking at tsconfig for TypeScript config, checking the Dockerfile, scanning the K8s manifests. It's doing in 20 seconds what would take you 10 minutes of clicking around."

---

### Step 3: Review the Generated CLAUDE.md

Once `/init` completes, ask Claude to show you what it generated:

```
Show me the CLAUDE.md file you just created
```

Alternatively, you can read it yourself in another terminal:

```bash
cat CLAUDE.md
```

**Things to look for in the generated CLAUDE.md:**

- Does it correctly identify the tech stack (Next.js 14, React 18, TypeScript, Tailwind)?
- Does it mention the charting library (Recharts)?
- Does it document the project structure?
- Does it list useful commands (dev, build, test, lint)?
- Does it mention Docker and Kubernetes configs?

> **What to say on video**: "The auto-generated CLAUDE.md is a solid starting point. It's maybe 80% of the way there. Over time, you'll add your own notes — team conventions, deployment quirks, things Claude should always remember. We'll cover that in Block 5 when we talk about memory."

---

### Step 4: Ask Claude to Explain the Architecture

Now let's use Claude as our personal architecture guide. Start broad:

```
Explain the overall architecture of this project. What are the main layers and how do they connect?
```

Claude will read through the source files and give you a structured breakdown. Watch the tool calls — you'll see it reading files across `src/app/`, `src/components/`, and the API routes.

**Follow up with more specific questions:**

```
What API routes does this project have and what does each one do?
```

Watch Claude use Glob to find the route files, then Read to examine each one. It should identify the API endpoints and explain the data they serve.

> **Note for the video**: Pause here and point out the tool calls in the terminal output. Show viewers how Claude chains Glob -> Read to answer the question. This is a great teaching moment about how the agent works under the hood.

---

### Step 5: Explore the Component Structure

Dive into the frontend:

```
Walk me through the main React components. What does each one render and how do they compose together?
```

Then get specific about a visual component:

```
How do the charts work? What data do they receive and how is it visualized?
```

Claude should identify the Recharts usage and explain the data flow from API routes to chart components.

---

### Step 6: Investigate the Theme Switching

This is a great example of tracing a feature through multiple files:

```
How does the theme switching mechanism work? Trace the flow from the UI toggle to the CSS changes.
```

Claude will typically:

1. Search for theme-related code across the codebase (Grep)
2. Find the theme toggle component (Read)
3. Trace the state management or context provider
4. Show how CSS variables or Tailwind classes change based on theme

> **What to say on video**: "This is where Claude Code really shines. You asked one question and it traced a feature across multiple files — the toggle component, the context provider, the CSS variables. To do this manually, you'd be cmd-clicking through imports for five minutes."

---

### Step 7: Try a Direct Command

Now let's switch from conversational to direct mode. Instead of asking open-ended questions, give Claude a specific task:

```
Find all files that contain "TODO" or "FIXME" comments
```

Then try:

```
List every external npm dependency and its version from package.json
```

And:

```
Show me the Dockerfile and explain each stage of the multi-stage build
```

Notice the difference in response style. Direct questions get focused, precise answers. Open-ended questions get broader, exploratory answers. Both are useful.

---

### Step 8: Test Claude's Understanding

Here's a fun exercise — quiz Claude on what it's learned:

```
If I wanted to add a new AI model to the comparison dashboard, which files would I need to modify? Walk me through the steps.
```

This forces Claude to synthesize everything it knows about the data model, API routes, and frontend components into a practical answer. It should give you a clear, ordered list of files to touch and changes to make.

> **What to say on video**: "This is the kind of question that separates Claude Code from a simple file reader. It's not just finding files — it's reasoning about how pieces fit together and giving you a practical action plan."

---

### Step 9: Check Context Usage and Exit

Let's see how much context this exploration used:

```
/cost
```

Note the token count. Codebase exploration is one of the heavier token operations because Claude reads many files. This is normal and expected.

Now exit:

```
/exit
```

---

### What Just Happened?

In about 20 minutes, you went from "I've never seen this codebase" to understanding:

1. **The project structure** — where everything lives and why
2. **The architecture** — how frontend components connect to API routes
3. **The data flow** — how model data moves from backend to charts
4. **The theme system** — a full feature traced across multiple files
5. **The infrastructure** — Docker builds, K8s manifests, CI config

You also generated a **CLAUDE.md** that will make every future Claude Code session smarter about this specific project.

This is the superpower. Not writing code (that comes next) — but *understanding* code. Every senior engineer will tell you: the hardest part isn't writing the fix, it's knowing where to look. Claude Code just shortcut that process from days to minutes.

---

### Going Further

If you want extra practice before Block 2, try these exploration exercises:

- Ask Claude to compare the development and production Docker configurations
- Ask it to explain the GitHub Actions workflow step by step
- Ask it to find any potential security concerns in the codebase
- Ask it to describe the testing setup and what kinds of tests exist
