---
layout: block
title: "Running & Testing Locally"
block_number: 2
description: "npm install, dev server, tests, Docker build — and watching Claude handle errors in real time."
time: "~33 min (8 min presentation + 25 min practical)"
features:
  - Bash Tool
  - Permission Prompts
  - /clear
  - /compact
objectives:
  - Understand how Claude Code executes shell commands via the Bash tool
  - Experience the permission model — Claude asks before running things
  - Set up the project with npm install and run the dev server
  - Run tests and the linter through Claude
  - Build a Docker image using Claude
  - Deliberately break something and watch Claude diagnose and fix it
  - Use /clear and /compact to manage conversation context
---

## From Reading to Doing

In Block 1, Claude read your code and explained it. That was impressive — but Claude Code is not just a reader. It's a doer.

This block is where things get real. Claude will run `npm install`, start your dev server, execute your test suite, lint your code, and build a Docker image. And when something breaks — because we're going to break something on purpose — Claude will diagnose the error, figure out the fix, and apply it.

This is the Bash tool in action: Claude's ability to execute shell commands in your terminal, with a safety-first permission model that makes sure you're always in control.

## What We'll Cover

1. **The Bash tool** — how Claude executes commands and reads their output
2. **The permission model** — why Claude asks before running things, and how to manage it
3. **Project setup** — `npm install` and `npm run dev` via Claude
4. **Testing and linting** — running the full quality pipeline
5. **Docker builds** — multi-stage image building
6. **Error handling** — the break-and-fix exercise
7. **Context management** — using `/clear` and `/compact`

## The Permission Model

Here's something important: Claude Code does not just run commands willy-nilly. When Claude wants to execute a shell command, it asks you first. You'll see a prompt like:

```
Claude wants to run: npm install
Allow? (y/n)
```

This is intentional. Claude is powerful, and power requires guardrails. You stay in control of what actually executes on your machine. Over time, you'll develop a rhythm — approve routine commands quickly, pause and think about unfamiliar ones.

You can also configure which commands are auto-approved and which always require confirmation. We'll cover that customization in later blocks.

## Context Management: /clear and /compact

As your conversation grows, Claude's context window fills up. Two commands help manage this:

- **`/clear`** — nuclear option. Wipes the entire conversation and starts fresh. Use this when you're switching to a completely different task.
- **`/compact`** — smart option. Claude summarizes the conversation so far into a compressed form, freeing up context space while retaining the key information. Use this when you're deep into a long session and Claude starts feeling slow.

Think of `/clear` as closing all your browser tabs and starting over. `/compact` is more like bookmarking the important tabs and closing the rest.

## Part 1: Presentation {#presentation}

**Target duration**: ~8 minutes
**Tone**: "Now Claude stops being read-only and starts doing things. This is where it gets exciting."

---

### Slide 1: Claude as an Operator (2 min)

**Talking points:**

> "Up until now, Claude has been reading. It looked at your files, searched for patterns, explained architecture. Impressive — but passive.
>
> Starting now, Claude becomes an operator. It can run commands. It can install dependencies, start servers, execute tests, build Docker images. And when something goes wrong — and it will — it reads the error output and tries to fix it."

- The Bash tool is what makes Claude Code an *agent* rather than just an *assistant*
- Under the hood, Claude sends a command to your shell, reads stdout and stderr, and reasons about the output
- It's like having a junior developer who can run anything you tell them, but — critically — they also understand the output

**Analogy**: "Think of it this way. Block 1 was like having a colleague who can read the codebase and explain it. Block 2 is like giving that colleague a terminal. Same intelligence, now with the ability to act."

---

### Slide 2: The Permission Model — Safety First (2 min)

**Talking points:**

> "Before you get nervous about an AI running commands on your machine — let's talk about the permission model. Because Anthropic clearly thought about this."

- Claude Code asks for permission before running any command
- You see exactly what it wants to execute before you approve
- There's a clear prompt: the command is shown, and you type `y` or `n`
- This applies to everything: npm install, starting servers, writing files, running scripts

**Why this matters:**

- You stay in the loop — no surprise `rm -rf` moments
- You build trust gradually — after a few sessions, you'll approve routine commands quickly
- It's a teaching tool — watching what Claude wants to run teaches you how it thinks
- You can configure trust levels later (auto-approve safe commands, always prompt for dangerous ones)

> "The permission model is not a limitation. It's a feature. It means you can hand Claude Code to a junior developer and know they won't accidentally nuke the database — because they have to approve every command."

**Quick demo note**: "In the practical, you'll see these permission prompts in action. You'll see Claude ask to run `npm install`, `npm test`, `docker build`. Each time, you decide. That's the deal."

---

### Slide 3: The Error Handling Loop (2 min)

**Talking points:**

This is arguably the most valuable pattern in Claude Code:

1. Claude runs a command
2. The command fails (error in stdout/stderr)
3. Claude reads the error output
4. Claude figures out what went wrong
5. Claude proposes a fix
6. You approve the fix
7. Claude runs the command again

> "This is the loop that saves you the most time in practice. Not the code generation. Not the file reading. The error handling. Because how many hours of your career have you spent staring at a stack trace, googling the error message, trying solutions from Stack Overflow?
>
> Claude does that loop in seconds. It reads the error, has the context of your entire codebase, and usually gets it right on the first try."

**Real example**: "In the practical, we're going to deliberately break something — introduce a type error, mess up an import — and watch Claude detect it, diagnose it, and fix it. All in one fluid sequence."

---

### Slide 4: Context Management — /clear and /compact (1.5 min)

**Talking points:**

- Claude has a context window — a maximum amount of conversation it can hold at once
- Long sessions eventually fill it up, which can slow things down
- Two tools to manage this:

**`/clear`** — complete reset
- Wipes the entire conversation
- Use when switching tasks: "I'm done debugging, now I want to work on a new feature"
- Like closing all browser tabs

**`/compact`** — smart compression
- Claude summarizes the conversation into a compressed form
- Keeps the key facts, drops the verbose back-and-forth
- Use when you're deep into a task and want to keep going
- Like bookmarking important tabs and closing the rest

> "Here's my rule of thumb: if I'm changing topics, I `/clear`. If I'm still working on the same thing but the conversation is getting long, I `/compact`. Simple as that."

---

### Slide 5: Transition (0.5 min)

**Talking points:**

> "Time to get hands-on. We're going to set up the project, run the full test suite, build a Docker image, and then — my favorite part — deliberately break something and watch Claude fix it. Let's go."

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: Project fully set up locally with dependencies installed, tests passing, Docker image built, and first-hand experience with Claude's error-handling loop
> **Prerequisites**: Block 1 completed, Claude Code running inside the ai-coderrank project

---

### Step 1: Start a Fresh Session

Navigate to the project and launch Claude Code:

```bash
cd ai-coderrank
claude
```

Since we had a conversation in Block 1, let's start clean:

```
/clear
```

This wipes the previous conversation. Claude starts with a blank slate (but still reads CLAUDE.md automatically, so it knows about the project).

---

### Step 2: Install Dependencies

Ask Claude to set up the project:

```
Install the project dependencies
```

**Watch what happens.** Claude will propose running `npm install`. You'll see a permission prompt:

```
Claude wants to run: npm install
Allow? (y/n)
```

Type `y` to approve. This is the permission model in action — Claude never runs a command without your explicit approval.

Watch the npm install output scroll by. Claude reads the output too, and if there are any warnings or errors, it will flag them.

> **What to say on video**: "See that permission prompt? Every single time Claude wants to run a command, you see exactly what it's about to do. You're always in the driver's seat. The AI proposes, you approve."

---

### Step 3: Start the Dev Server

Now let's run the application:

```
Start the development server
```

Claude will propose running `npm run dev`. Approve it.

You should see the familiar Next.js dev server output — something like:

```
  ▲ Next.js 14.x.x
  - Local:        http://localhost:3000
```

> **Note for the video**: Open `http://localhost:3000` in a browser to show the running dashboard. Point out the model comparison charts, the pricing table, the theme toggle. This is the app we'll be working with throughout the course.

**Important**: The dev server is now running in Claude's terminal. To continue working with Claude, you'll need to stop the server. Ask Claude:

```
Stop the dev server
```

Claude will send a Ctrl+C signal to terminate the process.

---

### Step 4: Run the Test Suite

Now let's verify that everything passes:

```
Run the tests
```

Claude will propose `npm test` (or whatever the project's test command is). Approve it.

Watch the test output. Claude reads every line — if any tests fail, it will tell you which ones and why.

**Follow up:**

```
What test framework is this project using and how are the tests organized?
```

This combines Claude's Bash tool (it just ran the tests) with its Read tool (it will look at test files to explain the organization). It's a natural back-and-forth.

> **What to say on video**: "Notice how Claude seamlessly switches between running commands and reading files. It ran the tests, saw the output, and now it's reading the test files to explain the organization. That's the agent model — it uses whatever tool is appropriate for the question."

---

### Step 5: Run the Linter

Code quality check:

```
Run the linter and show me if there are any issues
```

Claude will propose running the lint command (usually `npm run lint` for Next.js projects). Approve it.

If the linter finds issues, Claude will list them with file locations and explanations. If it's clean, Claude will confirm.

**Optional follow-up if there are lint warnings:**

```
Fix the lint issues you found
```

This is where you'll see Claude shift from reading/running to editing. It will propose changes to files to fix the lint warnings, and you'll see the permission prompt for each file edit.

---

### Step 6: Build the Docker Image

Let's test the containerized build:

```
Build the Docker image for this project
```

Claude will propose a `docker build` command. Something like:

```bash
docker build -t ai-coderrank .
```

Approve it and watch the multi-stage build execute. This takes a minute or two. Claude reads the build output and can explain each stage.

**Follow up:**

```
Explain what just happened during the Docker build. What did each stage do?
```

Claude will reference both the Dockerfile it read in Block 1 and the build output it just observed to give you a clear explanation of the multi-stage build process.

> **What to say on video**: "We just went from source code to a Docker image in one command. And Claude can explain every layer of that build because it's read the Dockerfile AND seen the build output. That's contextual understanding you don't get from just reading docs."

---

### Step 7: The Break-and-Fix Exercise

This is the highlight of Block 2. We're going to deliberately break the application and watch Claude diagnose and fix it.

**Step 7a: Introduce a bug**

Open a source file in your regular editor (VS Code, vim, whatever) and introduce a deliberate error. Some ideas:

- **TypeScript error**: Remove a required prop from a component call
- **Import error**: Rename an import to something that doesn't exist
- **Syntax error**: Delete a closing bracket or parenthesis

For example, open `src/app/page.tsx` and break an import:

```typescript
// Change this:
import { ModelCard } from '@/components/ModelCard'
// To this:
import { ModelCard } from '@/components/ModelCardBROKEN'
```

Save the file.

**Step 7b: Ask Claude to check**

Go back to your Claude Code session:

```
Run the TypeScript compiler to check for errors
```

Claude will propose running `npx tsc --noEmit` or a similar type-check command. Approve it.

The compiler will fail. Now watch the magic:

```
Fix the error
```

Claude will:

1. Read the error output from the TypeScript compiler
2. Identify the file and line number
3. Read the broken file
4. Understand what went wrong (the import path is incorrect)
5. Propose the fix (change the import back to the correct path)
6. Ask permission to edit the file

Approve the edit. Then verify:

```
Run the type check again to make sure it's fixed
```

It should pass now.

> **What to say on video**: "This is my favorite demo in the entire course. We broke something on purpose, and Claude: read the error, found the file, understood the problem, proposed the fix, and applied it — all in about 10 seconds. This is the error handling loop I was talking about in the presentation. This loop alone will save you hours every week."

---

### Step 8: Use /compact

By now, we've had a long conversation — installation, dev server, tests, linting, Docker build, the break-and-fix exercise. Let's compress it:

```
/compact
```

Claude will summarize the conversation into a compressed form. You'll see it condense everything into key points while preserving the important context (like the fact that we're working on ai-coderrank, that tests pass, that the Docker build succeeds).

After compacting, verify that Claude still has context:

```
What did we accomplish in this session so far?
```

Claude should give you a summary of everything you've done, even though the detailed back-and-forth has been compressed.

> **What to say on video**: "Compacting is like writing meeting notes. You keep the decisions and action items, but drop the lengthy discussion. Claude's context window stays fresh, and you can keep working without starting over."

---

### Step 9: Exit

```
/exit
```

---

### What Just Happened?

This was a big block. Let's recap:

1. **npm install** — Claude set up the project dependencies (permission model: first contact)
2. **Dev server** — Claude started and stopped the Next.js development server
3. **Tests** — Claude ran the test suite and explained the testing setup
4. **Linter** — Claude checked code quality and (optionally) fixed issues
5. **Docker build** — Claude built the multi-stage Docker image and explained the process
6. **Break-and-fix** — You deliberately broke the code, Claude diagnosed it and applied the fix
7. **Context management** — You used `/clear` to start fresh and `/compact` to compress a long session

You've now experienced all three of Claude Code's core capabilities:
- **Reading** code (Block 1)
- **Running** commands (Block 2)
- **Editing** files (the break-and-fix exercise)

That's the complete agent trifecta. Everything from here on builds on these three capabilities.

---

### Troubleshooting

**"Docker not found" or docker build fails**
Make sure Docker Desktop is installed and running. On macOS: open Docker Desktop from Applications. On Linux: ensure the Docker daemon is running (`sudo systemctl start docker`).

**npm install fails with permission errors**
Try clearing the npm cache: `npm cache clean --force`. If you're on macOS and see EACCES errors, you may need to fix npm permissions — see the [npm docs](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally).

**Dev server port already in use**
If port 3000 is occupied, you'll see an EADDRINUSE error. Either stop whatever's running on 3000, or ask Claude: "Start the dev server on port 3001".

**Claude seems slow or confused after a long session**
Use `/compact` to compress the conversation, or `/clear` for a full reset. Long conversations with many tool calls can fill the context window.
