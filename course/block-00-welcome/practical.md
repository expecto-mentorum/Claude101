---
layout: block
title: "Block 0 — Practical"
---

# Block 0: Welcome & Setup — Practical Guide

**Target duration**: ~15 minutes
**What you'll do**: Install Claude Code, authenticate, explore the `/help` system, fork the course project, and run Claude inside a real codebase for the first time.

---

## Step 1: Install Claude Code

Open your terminal and run the official installer:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

That's it. One line. The installer handles everything — it downloads the binary, puts it in the right place, and sets up your PATH.

**Verify the installation:**

```bash
claude --version
```

You should see a version number printed. If you see `command not found`, restart your terminal (or run `source ~/.zshrc` / `source ~/.bashrc`) so your PATH picks up the new binary.

> **Note for the video**: Show the full install process on screen. It usually takes about 10-15 seconds. Point out that there's no `npm install -g`, no Homebrew tap, no downloading a .dmg — just curl and done.

---

## Step 2: Authenticate

Now launch Claude Code for the first time:

```bash
claude
```

On first run, Claude Code will prompt you to authenticate. Follow the on-screen instructions — it will open a browser window where you sign into your Anthropic account.

**Requirements**: You need an active Anthropic Pro subscription ($20/month). Claude Code is included with Pro — no separate purchase.

After authentication succeeds, you'll land in an interactive session. You should see a prompt that looks something like:

```
claude >
```

Congratulations — you're in. But before we start chatting, let's learn the controls.

> **Note for the video**: Show the auth flow on screen. If you already authenticated before recording, you can mention that Claude Code remembers your session — you don't need to log in every time.

---

## Step 3: Explore /help

While still in the Claude Code session, type:

```
/help
```

This shows you all available slash commands. Take a moment to scan through them. The ones we'll use most in this course:

| Command | What it does |
|---------|-------------|
| `/help` | Shows all available commands |
| `/init` | Generates a CLAUDE.md file for your project |
| `/clear` | Clears conversation history and starts fresh |
| `/compact` | Summarizes the conversation to save context window space |
| `/cost` | Shows how many tokens you've used in this session |

> **What to say on video**: "Think of `/help` as your cheat sheet. When you forget a command — and you will, because there are a lot of them — just type `/help` and it's all right there."

**Try a quick conversation:**

Type something simple to verify everything works:

```
What can you help me with?
```

Claude will respond with an overview of its capabilities. Notice how it mentions reading files, running commands, editing code — these aren't just claims, these are actual tools it has access to. We'll see them in action very soon.

Now exit the session:

```
/exit
```

---

## Step 4: Fork and Clone ai-coderrank

Head to the course project repository on GitHub:

```
https://github.com/exitStatus0/ai-coderrank
```

**Fork the repo** using the GitHub UI (click the "Fork" button in the top right).

Then clone your fork locally:

```bash
git clone https://github.com/YOUR_USERNAME/ai-coderrank.git
cd ai-coderrank
```

Replace `YOUR_USERNAME` with your actual GitHub username.

**Quick sanity check** — make sure the project files are there:

```bash
ls
```

You should see the Next.js project structure: `package.json`, `src/`, `public/`, `Dockerfile`, `k8s/`, `.github/`, and more.

> **Note for the video**: Show the fork process on GitHub. Then show the clone command and the file listing. Point out the key directories — src for application code, k8s for Kubernetes manifests, .github for CI workflows.

---

## Step 5: Run Claude Code Inside the Project

This is the moment. Navigate into the project directory (if you aren't already there) and launch Claude Code:

```bash
cd ai-coderrank
claude
```

Now ask Claude something about the project:

```
What is this project? Give me a quick summary.
```

**Watch what happens.** Claude doesn't just guess based on the directory name. It actively reads files — you'll see it access `package.json`, `README.md`, maybe peek at `src/` — to build a real understanding of what this project does.

**Try a few more questions:**

```
What tech stack does this project use?
```

```
What does the file structure look like?
```

```
Are there any Docker or Kubernetes configs in this project?
```

Notice how Claude cites specific files and paths in its answers. It's not hallucinating — it's reading your actual codebase. This is the fundamental difference between Claude Code and a generic chatbot.

> **What to say on video**: "See those file reads happening? Claude is literally opening files on your machine and reading them. It's not guessing. It's not pulling from some training data about generic Next.js projects. It's reading YOUR code, right now, in real time."

---

## Step 6: Check Your Token Usage

Before wrapping up, let's see what that conversation cost:

```
/cost
```

This shows you the token count for the current session. In a typical first conversation, you'll use a relatively small number of tokens — the real usage comes when Claude starts reading many files and making edits (which we'll do starting in Block 1).

> **Note for the video**: Show the /cost output. Emphasize that the Pro plan includes generous usage, and for most development tasks you won't come close to hitting limits.

---

## Step 7: Exit

Clean exit:

```
/exit
```

---

## What Just Happened?

Let's recap what we accomplished:

1. **Installed** Claude Code with a single curl command
2. **Authenticated** with our Anthropic Pro subscription
3. **Explored** the `/help` system to see available commands
4. **Forked and cloned** the ai-coderrank project — our course companion
5. **Had our first conversation** with Claude Code inside a real codebase
6. **Watched Claude read actual files** to answer questions about our project

You now have a working Claude Code setup pointed at a real Next.js application. In the next block, we'll go much deeper — using `/init` to generate a CLAUDE.md file and having Claude explain the architecture, API routes, and component structure in detail.

---

## Troubleshooting

**"command not found: claude"**
Restart your terminal or source your shell config: `source ~/.zshrc` or `source ~/.bashrc`.

**Authentication fails or times out**
Make sure you have an active Pro subscription at [claude.ai](https://claude.ai). Free tier does not include Claude Code access.

**"Permission denied" during install**
The installer may need write access to `/usr/local/bin` or similar. Check the error message — you may need to adjust permissions or install to a user-local directory.
