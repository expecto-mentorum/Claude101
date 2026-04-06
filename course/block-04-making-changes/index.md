---
layout: block
title: "Making Changes — The Dark Theme"
block_number: 4
description: "Implement the dark theme using Claude's Edit and Write tools, preview changes live, iterate on the design, run tests, and commit — all without leaving the terminal."
time: "~33 min (8 min presentation + 25 min practical)"
features:
  - Edit Tool (surgical replacements)
  - Write Tool (full file creation)
  - Git Integration (commit, diff)
  - Iterative Refinement
objectives:
  - Understand how the Edit tool makes targeted changes vs. the Write tool creating full files
  - Implement a complete dark theme across Tailwind config, CSS variables, and components
  - Use Claude's iterative refinement to adjust visual details like contrast and color
  - Run tests through Claude to catch regressions
  - Let Claude write a meaningful commit message and create a clean commit
---

## Time to Build

Block 3 was the blueprint. Block 4 is the construction site.

This is the block where Claude Code earns its name. You're going to watch an AI agent open files, find the exact lines that need to change, replace them surgically, and leave everything else untouched. Then you'll run the dev server, look at the result, say "make the sidebar darker," and watch it do it again. In seconds.

If you've ever done a theme change by hand — hunting through CSS files, tweaking hex codes, refreshing the browser 400 times — this block will feel like time travel.

## What We'll Cover

1. **The Edit tool** — how Claude performs targeted string replacements in files (think `sed` but it understands context)
2. **The Write tool** — when Claude creates or completely rewrites a file
3. **Live iteration** — tweaking the theme by talking to Claude in plain English
4. **Running tests** — making sure the theme change didn't break anything
5. **Git integration** — staging, committing, and reviewing the diff, all through Claude

## The Theme System in ai-coderrank

Before we start editing, it helps to know how the pieces fit together:

| Component | Role |
|-----------|------|
| `tailwind.config.ts` | Defines the color palette and dark mode strategy |
| Global CSS (variables) | Declares `--background`, `--foreground`, `--accent`, etc. |
| `ThemeProvider` | React context that manages light/dark state |
| `scripts/switch-theme.sh` | Shell script for toggling themes |
| Individual components | May have inline Tailwind classes that reference theme colors |

The architecture is already set up for theming — CSS variables and a ThemeProvider are in place. We just need to define the dark palette and wire it up.

## What Changes Look Like

Here's a preview of the kind of edit Claude will make. In your global CSS, you might have:

```css
:root {
  --background: #ffffff;
  --foreground: #1a1a1a;
}
```

Claude will add:

```css
[data-theme="dark"] {
  --background: #0a0a0a;
  --foreground: #e5e5e5;
}
```

It does this using the **Edit tool** — specifying the exact string to find and the exact string to replace it with. No regex guessing games. No accidental changes to other files. Surgical.

## Milestone

By the end of this block: **dark theme working locally.** You'll see it in the browser, the tests will pass, and the commit will be clean. This is the first tangible change you've made to the codebase with Claude Code — and it won't be the last.

## Part 1: Presentation {#presentation}

**Duration**: ~8 minutes
**Goal**: Explain how Claude edits files (Edit vs Write tools), how git integration works, and how iterative refinement lets you fine-tune changes conversationally.

---

### Opening Hook (1 min)

**Say something like:**

> "Have you ever watched someone use Find and Replace across a codebase and thought 'this is going to end badly'? Yeah. Bulk find-and-replace is a chainsaw. Claude's Edit tool is a scalpel.
>
> In the last block, we planned the dark theme. We have an ADR, we have diagrams, we have a clear list of files to touch. Now we execute — and you're going to see something that fundamentally changed how I work. You describe what you want in English, and Claude makes the precise edits, in the right files, without breaking anything around them."

**Key point:** This block is where the plan meets reality. Students will see their first multi-file code change driven entirely by conversation.

---

### How Claude Edits Files (3 min)

#### The Edit Tool — Surgical Replacements

- Claude's primary way of modifying existing files
- It works by specifying an **exact string to find** and an **exact string to replace it with**
- Think of it like `sed`, but Claude understands the context of what it's replacing — it's not pattern-matching blindly

**Example on screen:**

```
Edit tool call:
  file: src/styles/globals.css
  old_string: "--background: #ffffff;"
  new_string: "--background: #0a0a0a;"
```

- The string must be **unique** in the file — if it appears twice, Claude has to include more surrounding context to disambiguate
- This is intentional: it prevents accidental changes to the wrong location

**Why this matters for DevOps/SRE:**
> "When you're editing Kubernetes manifests or Terraform files, you really don't want an AI tool making approximate changes. You want it to change exactly the line it means to change. The Edit tool enforces that."

#### The Write Tool — Full File Creation

- Used when creating **new files** or doing a **complete rewrite** of an existing file
- Claude generates the entire file content from scratch
- Best for: new components, new config files, new scripts
- Not ideal for: tweaking one line in a 200-line file (use Edit for that)

**The rule of thumb:**
- Changing a few lines? **Edit tool.**
- Creating something new? **Write tool.**
- Rewriting most of a file? **Write tool.**

#### What it looks like in practice:

> "When Claude implements the dark theme, you'll see it use the Edit tool maybe 8-10 times — once for each targeted change. It'll update the CSS variables, modify the Tailwind config, adjust component classes. Each edit is a small, reviewable change. You can watch each one and say 'yes, that looks right' or 'no, let's try a different color.'"

---

### Git Integration (2 min)

#### Claude can drive your entire git workflow

- **Staging files:** Claude calls `git add` on the specific files it changed
- **Creating commits:** Claude writes a commit message based on what it actually did (not what you asked for — what it *did*)
- **Showing diffs:** You can ask "show me the diff" at any point and Claude will run `git diff`
- **Reviewing history:** "Show me the git log" works exactly as you'd expect

#### Commit messages that don't suck

> "Here's a hot take: Claude writes better commit messages than most developers. Not because developers are lazy — we are, but that's not the point — but because Claude has the full context of every change it made, and it summarizes them naturally.
>
> Instead of 'update styles' you get something like 'feat: add dark theme with CSS custom properties and ThemeProvider integration.' It includes what changed and why. Every time."

#### The workflow:

1. Claude makes edits
2. You review (live in the browser, or ask Claude to show the diff)
3. You say "commit this"
4. Claude stages the right files, writes a meaningful message, creates the commit
5. You check `git log` — clean history

**Key point:** Claude doesn't just `git add .` and slam everything in. It stages specifically the files it changed.

---

### Iterative Refinement (1.5 min)

#### The real power: conversation-driven design

> "Here's where Claude Code becomes something different from any other tool. You run `npm run dev`, look at the dark theme in your browser, and say 'the sidebar is too light.' Claude opens the right file, finds the sidebar background CSS variable, and darkens it. You say 'the charts need more contrast.' Claude adjusts the Recharts color palette. You say 'the accent color should be more blue, less purple.' Done.
>
> You're having a *design conversation* with an agent that can instantly implement what you describe. No switching between Figma and your editor. No googling hex codes. Just talking."

#### Tips for effective iteration:

- **Be specific:** "Make it darker" works. "Change the sidebar background from gray-800 to gray-900" works better.
- **Reference what you see:** "The text on the card components is hard to read" — Claude will figure out which CSS variable controls that.
- **Iterate in small steps:** Three small refinements beat one massive prompt. Each step, you verify the result.

---

### The Theme System Quick Reference (30 sec)

Quick recap for students who might not have worked with CSS variables + Tailwind before:

| Layer | What it does |
|-------|-------------|
| CSS Variables (`:root`) | Define the actual color values |
| `[data-theme="dark"]` | Override those values for dark mode |
| Tailwind Config | Maps CSS variables to Tailwind utility classes |
| ThemeProvider | React context that toggles the `data-theme` attribute |
| `switch-theme.sh` | CLI shortcut for toggling the theme |

> "The beauty of this architecture is that Claude only needs to define the dark color values and make sure the ThemeProvider toggles the right attribute. The components already use the CSS variables — they'll pick up the new colors automatically."

---

### Closing (30 sec)

> "This is the block where it gets real. You're going to go from a plan to a working dark theme in about 25 minutes. You'll make edits, preview them, iterate, run tests, and commit — all through conversation. By the end, you'll have your first real feature built with Claude Code. Let's do it."

---

### Common Student Questions

**Q: What if Claude makes an edit I don't like?**
A: You can always undo. Claude respects your git workflow — if something looks wrong, `git checkout -- <file>` or just tell Claude "undo that last change." You're always in control.

**Q: Does Claude run the dev server for me?**
A: Claude can start `npm run dev`, but you'll want to view the result in your own browser. Claude can't see your screen — you're the visual feedback loop. Tell it what you see, and it adjusts.

**Q: What if tests fail after the theme change?**
A: That's actually a great learning moment. Claude will read the test output, figure out what broke (probably a snapshot test or a color assertion), and fix it. This is exactly the kind of error-handling workflow you'll use constantly.

**Q: Can Claude handle merge conflicts?**
A: Yes, but that's not in scope for this block. We're working on a clean branch. Merge conflict resolution comes up naturally in later blocks.

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: A fully working dark theme implementation with a clean git commit
> **Prerequisites**: Block 3 completed (ADR and diagrams created, plan reviewed), Claude Code running in the ai-coderrank project directory

---

### Step 1: Kick Off the Implementation (3 min)

We have a plan from Block 3. Now we execute. Make sure you're in **Act mode** (not Plan mode — press Shift+Tab if needed).

**Type:**

```
We planned a dark theme in our last session. The ADR is at docs/adr/001-dark-theme.md.
Implement the dark theme now. Start with the CSS variables and Tailwind config, then
update the ThemeProvider to support toggling, and finally adjust any component styles
that need dark variants.

Use the existing CSS variable architecture — define dark values under
[data-theme="dark"] so components pick them up automatically.
```

**What to watch for:**
- Claude will read the current CSS file, `tailwind.config.ts`, and the ThemeProvider component
- It will use the **Edit tool** multiple times — one targeted replacement per change
- Watch the tool calls: each one shows `old_string` (what it found) and `new_string` (what it replaced it with)
- The first edits will likely be in the global CSS file, adding dark theme variable overrides

**You should see Claude making changes like:**

1. Adding `[data-theme="dark"]` CSS variable block with dark colors
2. Updating `tailwind.config.ts` to reference CSS variables (if not already)
3. Modifying the ThemeProvider to handle theme toggling and persistence
4. Adjusting the `switch-theme.sh` script if needed

Let Claude work through the full implementation. Don't interrupt unless you see something clearly wrong — let it finish its chain of edits first.

---

### Step 2: Preview the Changes (3 min)

Time to see what we've got. Ask Claude to start the dev server:

```
Run npm run dev so I can preview the changes.
```

Claude will run the command. Open your browser and go to **http://localhost:3000**.

**Toggle the theme** using whatever mechanism the ThemeProvider exposes (a button in the UI, or use the script):

```
Run scripts/switch-theme.sh to toggle to the dark theme, or tell me how to
toggle it in the browser.
```

**Look at the result in your browser.** You're the eyes here — Claude can't see your screen. Take note of:
- Is the background dark?
- Is the text readable?
- How do the charts look? (Recharts might need color adjustments)
- Does the sidebar look right?
- Are there any elements that are still "stuck" in light mode?

---

### Step 3: Iterate on the Design (8 min)

This is the fun part. You're going to refine the theme through conversation. Here are some prompts to try — pick the ones that match what you see in the browser:

#### If the sidebar needs work:

```
The sidebar background is too light for dark mode. Make it darker — something
like a very dark gray or near-black. The sidebar text should be light gray,
not white, so it's easy on the eyes.
```

#### If the charts need contrast:

```
The Recharts charts are hard to read in dark mode. The line colors and bar
colors need to be brighter and more saturated against the dark background.
Adjust the chart color palette for dark mode.
```

#### If the accent color isn't right:

```
The accent color looks washed out in dark mode. Change it to a brighter blue —
something like #3b82f6 — and make sure it has enough contrast against the
dark background.
```

#### If cards or panels blend into the background:

```
The card components are blending into the page background. Add a subtle
border or make the card background slightly lighter than the page background
so they stand out.
```

#### If text contrast is off:

```
Some of the secondary text is too dim in dark mode. Increase the contrast —
the secondary text color should be at least #9ca3af, not darker.
```

**After each change, refresh your browser and evaluate.** Tell Claude what you see. This back-and-forth is the core workflow — describe the problem in plain English, let Claude make the targeted edit, check the result, repeat.

**Pro tip:** You don't need to know the exact file or variable name. Saying "the sidebar text is too dark" is enough — Claude will find the right CSS variable and adjust it.

---

### Step 4: Check the Diff (2 min)

Before we run tests, let's see everything that changed. Ask Claude:

```
Show me the full git diff of all changes we've made.
```

**What to look for in the diff:**
- CSS variable additions under `[data-theme="dark"]`
- Tailwind config modifications (if any)
- ThemeProvider changes for toggle logic
- Any component-level style adjustments
- No unintended changes to unrelated files

**If something looks wrong:**

```
I see you changed [file] but I don't think that was necessary. Can you
revert that specific change?
```

Claude can selectively undo changes using the Edit tool — putting the original string back.

---

### Step 5: Run the Tests (3 min)

Let's make sure we didn't break anything:

```
Run npm test and show me the results. If any tests fail, explain why and
fix them.
```

**What to watch for:**
- Claude runs the test command
- If tests pass: great, move on
- If tests fail: Claude will read the error output, identify the cause, and propose fixes

**Common test failures after a theme change:**

1. **Snapshot tests** — Components render differently with new CSS. Claude will update the snapshots.
2. **Color assertions** — If tests check for specific color values, they'll need updating.
3. **Accessibility tests** — Contrast ratio checks might flag colors that are too similar.

**If tests fail, let Claude fix them:**

```
Fix the failing tests. If they're snapshot tests, update the snapshots.
If they're asserting specific colors, update them to match the dark theme values.
```

Run the tests again after fixes:

```
Run npm test again to confirm everything passes.
```

---

### Step 6: Commit with Claude (4 min)

Everything works. Time to commit. This is where Claude's git integration shines:

```
Commit all the dark theme changes. Write a descriptive commit message that
explains what was changed and why.
```

**What to watch for:**
- Claude runs `git add` on the specific files it changed (not `git add .`)
- It writes a commit message — watch the quality. It should mention:
  - What: dark theme implementation
  - How: CSS variables, ThemeProvider updates
  - Why: references the ADR or mentions the planning decision
- The commit is created cleanly

**The commit message should look something like:**

```
feat: implement dark theme using CSS custom properties

Add dark theme color palette under [data-theme="dark"] selector,
update ThemeProvider to support theme toggling with localStorage
persistence, and adjust chart colors for dark mode contrast.

Implements the decision documented in docs/adr/001-dark-theme.md.
```

Not `"update styles"`. Not `"dark mode"`. A real, useful commit message.

---

### Step 7: Review the Git Log (2 min)

Let's verify everything is clean:

```
Show me the git log with the last 5 commits, and then show me a summary
of the files changed in the most recent commit.
```

**You should see:**
- Your dark theme commit at the top
- A clean list of changed files (CSS, config, ThemeProvider, possibly components)
- No unexpected files in the commit

**Optional — verify the commit content:**

```
Show me git show --stat for the latest commit.
```

This gives you a summary of files changed with lines added/removed — a nice final verification.

---

### Checkpoint

Before moving on, verify:

- [ ] The dark theme displays correctly at http://localhost:3000
- [ ] Text is readable in both light and dark modes
- [ ] Charts have adequate contrast in dark mode
- [ ] The theme toggle works (switching between light and dark)
- [ ] `npm test` passes with no failures
- [ ] A clean commit exists with a descriptive message
- [ ] `git log` shows the commit and `git diff` is clean (no unstaged changes)

**Milestone reached: Dark theme working locally!**

---

### Key Takeaways

1. **The Edit tool is precision, not brute force.** Each edit specifies an exact string to find and replace. This is why Claude can modify a single CSS variable in a 200-line file without touching anything else. It's also why you can trust it with config files and manifests.

2. **You are the visual feedback loop.** Claude can't see your browser. Your job is to describe what you see — "the text is hard to read," "the sidebar blends in," "the charts look great." Claude translates your feedback into precise code changes. The better you describe what you see, the fewer iterations you need.

3. **Iteration is the workflow, not a fallback.** Getting it right on the first prompt is nice. Getting it right after three refinements is *normal*. Professional designers iterate. Professional engineers iterate. You're iterating with an AI partner — that's not a limitation, it's the feature.

4. **Git integration closes the loop.** You don't need to context-switch to the terminal to commit. Claude stages the right files, writes a meaningful message, and keeps your git history clean. This matters more than it sounds — when you're in flow, staying in the conversation keeps you there.

5. **Tests catch what your eyes miss.** Always run the test suite after visual changes. A theme change can break snapshot tests, accessibility checks, and color-based assertions. Claude handles these fixes quickly, but you have to ask it to run the tests first.

---

### What's Next

In Block 5, we'll teach Claude to *remember* your preferences. Like the fact that you prefer dark mode. Or that your team uses conventional commits. Or that your Kubernetes manifests should always include resource limits. This is where Claude stops being a tool you use and starts being a collaborator that knows your codebase.
