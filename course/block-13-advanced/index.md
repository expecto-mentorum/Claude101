---
layout: block
title: "Advanced Patterns & What's Next"
block_number: 13
description: "The ecosystem beyond the CLI — agent teams, headless mode, batch operations, the Desktop app, and where to go from here."
time: "~25 min (10 min presentation + 15 min practical)"
features:
  - Agent teams (experimental)
  - /batch for parallel changes
  - Headless mode (claude -p)
  - /schedule for cloud automation
  - Desktop app and IDE extensions
  - Plugins for distributing skills
objectives:
  - Understand agent teams as coordinated multi-instance workflows
  - Use /batch to make parallel changes across multiple files
  - Run Claude Code in headless mode for scripting and automation
  - Set up a scheduled cloud task for recurring operations
  - Know the full Claude Code ecosystem — Desktop app, IDE extensions, web interface
  - Have a clear roadmap for continued learning after the course
---

## The View From the Summit

You made it. Block 12 was the finish line — your app is live on the internet via GitOps. This block is the view from the summit. We look around, see what else is out there, and map the trails you might want to explore next.

Think of blocks 0-12 as learning to drive. You can navigate city streets, handle highway merges, and parallel park. This block is the car manual — the features you didn't need yet but will appreciate as you take on longer journeys. Sport mode. Adaptive cruise control. The stuff that separates "I can drive" from "I drive well."

Some of these features are stable and production-ready. Some are experimental and evolving fast. All of them are worth knowing about because they'll show up in your work sooner than you think.

## What We'll Cover

1. **Agent teams** — multiple Claude instances working together on complex tasks
2. **`/batch`** — making the same change across many files in parallel
3. **Headless mode** — `claude -p` for scripting and CI pipelines
4. **Scheduled cloud tasks** — deeper dive into `/schedule`
5. **The ecosystem** — Desktop app, IDE extensions, web-based Claude Code, plugins
6. **Course wrap-up** — what you've built, where to go next

## Why This Block Matters

Every tool has a "90% use case" and a "10% power-user case." Blocks 0-12 covered the 90%. This block covers the 10% that makes the difference between using Claude Code and mastering it.

Agent teams turn single-threaded work into parallel work. `/batch` turns repetitive changes into one-shot operations. Headless mode turns interactive conversations into automated scripts. These aren't nice-to-haves — they're the features that, once you use them, you wonder how you ever worked without them.

## Prerequisites

- Completed Blocks 0-12 (everything deployed, GitOps working)
- A working Claude Code installation with authenticated sessions
- Curiosity about what else is possible
