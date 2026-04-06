---
layout: block
title: "MCP Servers — Connecting External Tools"
block_number: 9
description: "Extend Claude Code with the Model Context Protocol — connect to GitHub, filesystems, databases, and 50+ external tools without leaving your terminal."
time: "~35 min (10 min presentation + 25 min practical)"
features:
  - MCP config (.mcp.json)
  - claude mcp add
  - MCP tools in permissions
  - GitHub MCP server
  - Filesystem MCP server
objectives:
  - Understand MCP as Claude Code's plugin system for external tools
  - Configure the GitHub MCP server to interact with repos, issues, and PRs
  - Use Claude to create, list, and comment on GitHub issues
  - Configure the filesystem MCP server for cross-directory access
  - Manage MCP tool permissions via the /permissions command
  - Know where to find and evaluate community MCP servers
---

## Claude's Plugin System

Up to this point, Claude Code has been working with what's in front of it — your local files, your terminal, your git repo. That's powerful, but it's also a walled garden. What if you want Claude to check your GitHub issues? Comment on a PR? Query a database? Post to Slack?

That's what MCP does. The Model Context Protocol is how Claude connects to the outside world. Think of MCP servers as plugins — each one gives Claude a new set of abilities. The GitHub MCP server lets Claude read issues, create PRs, and comment on code reviews. The filesystem MCP server lets Claude access directories outside the current project. The Slack MCP server lets Claude post messages. And there are dozens more.

The beauty of MCP is that Claude treats these external tools exactly like its built-in tools. It doesn't "call an API" or "run a script" — it just has new capabilities that show up in its toolbox, as natural as reading a file or running a command.

## What We'll Cover

1. **What MCP is** — the Model Context Protocol and why it exists
2. **How MCP servers work** — processes that expose tools to Claude
3. **Configuration** — setting up `.mcp.json`
4. **GitHub MCP in practice** — issues, PRs, and comments
5. **Filesystem MCP** — accessing directories outside your project
6. **Permissions** — controlling which MCP tools Claude can use
7. **The MCP ecosystem** — where to find more servers

## Why This Block Matters

Here's the real power move: when Claude can interact with both your code _and_ your project management tools, it stops being just a coding assistant and becomes a development workflow assistant.

Imagine this: "Look at the open issues labeled `bug`, find the one about the dark theme toggle, read the relevant source files, implement a fix, create a branch, push it, and open a PR linking back to the issue." That's not science fiction — that's a Claude session with MCP configured.

MCP is what turns Claude from "a really smart pair programmer" into "an engineer who can navigate your entire development ecosystem."

## Prerequisites

- Completed Blocks 0-8
- A GitHub account with access to a repository (ai-coderrank or any test repo)
- The `gh` CLI installed and authenticated (`gh auth status` should show you logged in)
- Node.js installed (MCP servers run as Node processes)

---

## Part 1: Presentation {#presentation}

> **Duration**: ~10 minutes
> **Goal**: Students understand what MCP is, how servers work, and how to configure Claude Code to use external tools.

---

### Slide 1: The Integration Problem

Here's your daily workflow as a developer:

1. Read a GitHub issue
2. Open the codebase in your editor
3. Make changes
4. Run tests
5. Push a branch
6. Open a PR linking to the issue
7. Respond to review comments
8. Post an update in Slack

Steps 2-4? Claude Code handles those beautifully. But steps 1, 5-8? Claude can't see your GitHub issues. It can't open PRs. It doesn't know Slack exists. It's stuck inside your local terminal.

That's the integration problem. Claude is great at code, but code doesn't exist in a vacuum. It exists inside a workflow that spans multiple tools, platforms, and services. Without access to those tools, Claude is like a chef who can cook anything but can't read the menu or serve the food.

MCP solves this.

---

### Slide 2: What MCP Actually Is

**MCP = Model Context Protocol.** It's an open standard that defines how AI models connect to external tools and data sources.

Think of it like USB for AI. Before USB, every peripheral had its own connector — printers, keyboards, mice, cameras all used different cables. USB said "one standard plug, everything connects the same way." MCP does the same thing for AI integrations.

```
Before MCP:                          With MCP:
  Claude ──custom code──> GitHub       Claude ──MCP──> GitHub
  Claude ──custom code──> Slack        Claude ──MCP──> Slack
  Claude ──custom code──> Jira         Claude ──MCP──> Linear
  Claude ──custom code──> Database     Claude ──MCP──> PostgreSQL
  (Each requires building & maintaining a custom integration)
  (One protocol. Plug any server in.)
```

An MCP **server** is a small process that:
1. Exposes a set of **tools** (functions Claude can call)
2. Communicates with Claude using the MCP protocol
3. Handles the actual API calls to the external service

From Claude's perspective, MCP tools look and work exactly like its built-in tools. It doesn't know (or care) that `create_issue` comes from an MCP server rather than being baked into the binary. It just has more tools available.

---

### Slide 3: Available MCP Servers

The MCP ecosystem is growing fast. Here are the servers you'll encounter most:

#### Official / Well-Maintained Servers

| Server | What It Provides | Use Case |
|--------|-----------------|----------|
| **GitHub** | Issues, PRs, reviews, comments, repo management | Full GitHub workflow from Claude |
| **Filesystem** | Read/write files in additional directories | Access files outside your project |
| **PostgreSQL** | Query and inspect databases | Debug data issues, explore schemas |
| **Slack** | Send messages, read channels | Team communication |
| **Linear** | Issues, projects, cycles | Project management |
| **Docker** | Container management | Build, run, inspect containers |
| **Puppeteer** | Browser automation | Testing, screenshots |

#### Community Servers

The community has built 50+ MCP servers for everything from Notion to Google Drive to AWS. Quality varies — check the GitHub stars, last commit date, and whether the maintainer is responsive to issues.

> **Where to find them**: https://github.com/modelcontextprotocol/servers — the official list. Also check https://mcp.so for a searchable directory.

---

### Slide 4: Configuration — `.mcp.json`

MCP servers are configured in a JSON file. Here's the structure:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ]
    }
  }
}
```

Each server entry has:
- **`command`** — what to run (usually `npx` for Node-based servers)
- **`args`** — arguments passed to the command (the package name, plus any server-specific config)
- **`env`** (optional) — environment variables the server needs (API tokens, etc.)

This file lives at `.mcp.json` for project-level config. User-level config is stored in `~/.claude.json` (managed automatically by `claude mcp add --scope user`).

When Claude Code starts (or when you restart a session), it launches each configured MCP server as a background process. The servers stay running for the duration of your session.

---

### Slide 5: How Claude Uses MCP Tools

Once an MCP server is configured, its tools appear in Claude's toolbox automatically. Here's what that looks like in practice:

**You say**: "List all open issues on the ai-coderrank repo labeled `bug`"

**Claude thinks**: "I have a `list_issues` tool from the GitHub MCP server. Let me use it."

**What happens**:
1. Claude calls the `list_issues` tool with parameters `{owner: "your-name", repo: "ai-coderrank", state: "open", labels: ["bug"]}`
2. The MCP server receives the call
3. The server makes the actual GitHub API request
4. Results come back to Claude
5. Claude presents the issues to you in a readable format

You didn't write any code. You didn't configure any API calls. You just asked a question, and Claude used the right tool to answer it.

The same flow works for creating issues, commenting on PRs, reading files from other directories, querying databases — any tool that any configured MCP server exposes.

---

### Slide 6: Permissions — Controlling MCP Access

Here's the thing about giving Claude access to your GitHub account: you might not want it to do _everything_. Maybe you're fine with it reading issues but not comfortable with it pushing code. Maybe it can list PRs but shouldn't merge them.

That's where permissions come in. The `/permissions` command in Claude Code shows and manages tool access:

```
> /permissions

Allowed:
  - mcp__github__list_issues
  - mcp__github__get_issue
  - mcp__github__create_issue
  - mcp__github__list_pull_requests

Denied:
  - mcp__github__merge_pull_request
  - mcp__github__delete_branch
```

MCP tools follow the naming convention `mcp__<server>__<tool>`. You can allow or deny each one individually, or use patterns:

- Allow all GitHub read operations: `mcp__github__get_*`, `mcp__github__list_*`
- Block all destructive operations: deny `mcp__github__delete_*`, `mcp__github__merge_*`

This gives you fine-grained control. Claude can be a helpful assistant that reads your issues and opens PRs, without being able to merge to main or delete branches.

> **Rule of thumb**: Start with read-only access. Add write permissions as you build trust. Never auto-approve destructive operations.

---

### Key Takeaways

| Concept | What It Is | Why It Matters |
|---------|-----------|----------------|
| MCP | Model Context Protocol — AI's plugin standard | One protocol to connect any tool |
| MCP server | A process exposing tools via MCP | Each server adds capabilities to Claude |
| `.mcp.json` | MCP configuration file | Where you declare which servers to run |
| MCP tools | Functions exposed by MCP servers | Claude calls them like built-in tools |
| `/permissions` | Tool access management | Control exactly what Claude can do |
| `mcp__server__tool` | MCP tool naming convention | Used for fine-grained permission rules |

> **Transition**: Time to wire things up. You're about to connect Claude to GitHub and watch it create issues, comment on PRs, and navigate your project management workflow — all from the terminal.

---

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: GitHub MCP integration for managing issues and PRs, filesystem MCP for cross-directory access, and permissions management for MCP tools.
> **Prerequisites**: Completed Blocks 0-8, a GitHub account, `gh` CLI installed and authenticated, Node.js installed

---

### Step 1: Add the GitHub MCP Server (~5 min)

The preferred way to configure MCP servers is the `claude mcp add` command. It writes the config to `.mcp.json` (project-scoped with `--scope project`) or `~/.claude.json` (user-scoped with `--scope user`) automatically — no hand-editing JSON.

First, you need a GitHub personal access token. If you don't have one:

1. Go to https://github.com/settings/tokens
2. Click **Generate new token (classic)**
3. Name it `claude-code-mcp`
4. Select scopes: `repo` (full control of private repositories), `read:org`
5. Generate and copy the token

Set the environment variable (add this to your `~/.zshrc` or `~/.bashrc` to make it permanent):

```bash
export GITHUB_TOKEN="ghp_your_token_here"
```

Now add the GitHub MCP server using the CLI:

```bash
claude mcp add github --scope project -- npx -y @modelcontextprotocol/server-github
```

This creates `.mcp.json` in your project root with the server configuration. You can inspect it:

```bash
cat .mcp.json
```

You'll see something like:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

> **Security note**: The `${GITHUB_TOKEN}` syntax references your shell environment variable at runtime — the actual token is never stored in the file. If `.mcp.json` is committed to git, this is safe. For extra caution, add `.mcp.json` to `.gitignore`.
>
> **Why `claude mcp add` instead of hand-editing?** The CLI command handles JSON formatting, validates the server config, and supports `--scope project` vs `--scope user` to control where the config is stored. It's the officially recommended approach.

---

### Step 2: Restart Claude and Verify MCP (~2 min)

MCP servers load when a session starts. Exit your current Claude session and start a new one:

```bash
cd ~/ai-coderrank
claude
```

When Claude starts, it will launch the GitHub MCP server in the background. You might see a brief initialization message.

Verify the MCP tools are available by asking:

```
What MCP tools do you have available? List them.
```

Claude should report tools like:
- `create_or_update_file` — Create or update a file in a repository
- `create_issue` — Create a new issue
- `create_pull_request` — Create a new pull request
- `get_issue` — Get details of an issue
- `list_issues` — List issues in a repository
- `add_issue_comment` — Comment on an issue
- `search_repositories` — Search for repositories
- `get_file_contents` — Get file contents from a repo
- And more...

If Claude says it doesn't have MCP tools, check:
1. Is `GITHUB_TOKEN` exported in your current shell?
2. Is `.mcp.json` valid JSON? (ask Claude to validate it)
3. Try `npx -y @modelcontextprotocol/server-github` manually to see if it runs

---

### Step 3: List Issues with MCP (~3 min)

Now let's use the GitHub MCP tools. Ask Claude:

```
List all open issues on the ai-coderrank repository. 
My GitHub username is <your-github-username>.
```

Claude will use the `list_issues` tool from the GitHub MCP server. It makes the actual GitHub API call and returns the results.

If there are no issues yet (it's a fresh repo), Claude will tell you that. That's fine — we're about to create some.

Try a broader query:

```
Search for any repositories I own that have "coderrank" in the name.
```

This uses the `search_repositories` tool. Claude can navigate your entire GitHub presence through MCP.

---

### Step 4: Create a GitHub Issue via MCP (~3 min)

Here's where it gets fun. Ask Claude:

```
Create a new issue on the ai-coderrank repository:
- Title: "Add dark theme documentation"
- Body: "The dark theme was implemented in Block 4 but there's no user-facing 
  documentation. We need:
  - A section in the README explaining the theme toggle
  - Screenshot showing both light and dark modes
  - Any relevant accessibility notes (contrast ratios, etc.)
  
  Priority: low
  Relates to: dark theme implementation"
- Labels: documentation, enhancement
```

Claude will use the `create_issue` tool. You should get back the issue number and URL.

Verify it worked:

```
Show me the issue you just created. Include the full body and any labels.
```

Claude reads it back using `get_issue`. Then go to your GitHub repo in the browser to confirm it's there.

Now create another issue:

```
Create an issue titled "Add health check endpoints" with a body describing 
the need for /health and /ready endpoints for Kubernetes liveness and 
readiness probes. Label it with "enhancement" and "infrastructure".
```

---

### Step 5: Comment on an Issue or PR via MCP (~3 min)

Let's interact with existing issues. Ask Claude:

```
Add a comment to issue #1 on ai-coderrank that says:
"Investigated this — the dark theme implementation is in src/app/providers.tsx 
and uses next-themes. Documentation should cover:
1. How the ThemeProvider works
2. The useTheme() hook for components that need theme-aware styling
3. How to add new theme-sensitive components

Will pick this up in a future block."
```

Claude uses the `add_issue_comment` tool. The comment appears on the issue as if you typed it in the GitHub UI.

If you have any open PRs, try:

```
List all open pull requests on ai-coderrank. For each one, show the title, 
author, and number of changed files.
```

And then:

```
Add a review comment to PR #<number> that says "Looks good! Just one note — 
make sure the resource limits are set in the K8s deployment."
```

> **What you're seeing**: Claude is navigating your GitHub workflow entirely from the terminal. No browser tabs, no context switching. Issue tracking, code review, and development all in one place.

---

### Step 6: Configure Filesystem MCP (~3 min)

The filesystem MCP server lets Claude access directories outside your current project. This is useful when you need Claude to reference configs, scripts, or data in another location.

Ask Claude:

Use the CLI to add it:

```bash
claude mcp add filesystem --scope project -- npx -y @modelcontextprotocol/server-filesystem ~/.kube ~/scripts
```

This adds the filesystem server alongside the existing GitHub server in `.mcp.json`. The filesystem MCP server takes directory paths as arguments — Claude can only access files within those directories, not anywhere else.

Restart Claude and test:

```
Using the filesystem MCP, read my kubeconfig at ~/.kube/config-do and tell me 
what cluster it points to, what user credentials it uses, and whether the 
certificate is Base64-encoded or a file reference.
```

Claude uses the filesystem MCP to read a file outside the project directory — something it normally can't do.

---

### Step 7: Manage MCP Permissions (~3 min)

Now that you have MCP tools, let's control what Claude can do with them.

In your Claude session, run:

```
/permissions
```

You'll see the standard permissions list, plus MCP tools. They follow the naming pattern `mcp__<server>__<tool>`:

```
mcp__github__create_issue
mcp__github__list_issues
mcp__github__create_pull_request
mcp__github__merge_pull_request
mcp__filesystem__read_file
mcp__filesystem__write_file
```

You can allow or deny specific tools. For safety, let's block the dangerous ones:

```
Update permissions to deny these MCP tools:
- mcp__github__merge_pull_request (don't want accidental merges)
- mcp__github__delete_branch (protect branches)
- mcp__filesystem__write_file (read-only filesystem access)
```

This goes into your `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Write", "Edit", "Bash",
      "mcp__github__list_issues",
      "mcp__github__get_issue",
      "mcp__github__create_issue",
      "mcp__github__add_issue_comment",
      "mcp__github__list_pull_requests",
      "mcp__github__create_pull_request",
      "mcp__filesystem__read_file",
      "mcp__filesystem__list_directory"
    ],
    "deny": [
      "mcp__github__merge_pull_request",
      "mcp__github__delete_branch",
      "mcp__filesystem__write_file"
    ]
  }
}
```

Now Claude can create issues and PRs but can't merge or delete. It can read your kubeconfig but can't modify it. This is the principle of least privilege applied to AI tools.

#### Test the Permissions

```
Try to merge pull request #1 on ai-coderrank.
```

Claude should report that it doesn't have permission to use that tool.

---

### Step 8: Explore the MCP Ecosystem (~3 min)

The GitHub and filesystem servers are just the beginning. Here's a quick tour of what else is out there:

#### Where to Find MCP Servers

- **Official registry**: https://github.com/modelcontextprotocol/servers
- **MCP directory**: https://mcp.so
- **Anthropic's list**: Check the Claude Code documentation for officially supported servers

#### Notable Servers Worth Exploring

| Server | What It Does | Install |
|--------|-------------|---------|
| **Slack** | Send/read messages, manage channels | `@anthropic/mcp-slack` |
| **Linear** | Issues, projects, cycles | `@linear/mcp-server` |
| **PostgreSQL** | Query databases, inspect schemas | `@modelcontextprotocol/server-postgres` |
| **Docker** | Manage containers, images, volumes | `@modelcontextprotocol/server-docker` |
| **Puppeteer** | Browser automation, screenshots | `@modelcontextprotocol/server-puppeteer` |
| **Memory** | Persistent knowledge graph | `@modelcontextprotocol/server-memory` |

#### How to Evaluate a Community MCP Server

Before adding any MCP server to your configuration, check:

1. **Source code** — Is it open source? Can you read what it does?
2. **Permissions** — What API scopes does it need? (Minimal is better)
3. **Maintenance** — When was the last commit? Are issues being addressed?
4. **Stars/forks** — Community validation (not definitive, but a signal)
5. **Token handling** — Does it handle credentials securely?

> **Rule of thumb**: Official servers from Anthropic and the MCP organization are safe. Community servers should be reviewed like any open-source dependency — check the code, check the maintainer, check the permissions.

---

### Checkpoint

Your MCP setup:

```
.mcp.json
├── github (MCP server)
│   ├── list_issues         ✓ allowed
│   ├── create_issue        ✓ allowed
│   ├── add_issue_comment   ✓ allowed
│   ├── create_pull_request ✓ allowed
│   ├── merge_pull_request  ✗ denied
│   └── delete_branch       ✗ denied
└── filesystem (MCP server)
    ├── read_file           ✓ allowed
    ├── list_directory      ✓ allowed
    └── write_file          ✗ denied
```

You've:
- Configured two MCP servers (GitHub and filesystem)
- Created issues and comments directly from Claude
- Set up fine-grained permissions for MCP tools
- Explored the MCP ecosystem

Claude is no longer confined to your local codebase. It can interact with your GitHub projects, read configs from other directories, and — with additional MCP servers — connect to almost any tool in your development workflow.

---

### Bonus Challenges

**Challenge 1: Full issue workflow**
Create an issue, make the code change to fix it, commit the fix, and ask Claude to open a PR that references the issue — all in one session, all using MCP tools. This is the dream workflow: issue to PR without leaving the terminal.

```
Create an issue titled "Add /health endpoint for K8s readiness probe". Then 
implement the fix, commit it to a new branch, push it, and create a PR that 
closes the issue. Do the whole workflow.
```

**Challenge 2: Add Slack MCP**
If your team uses Slack, configure the Slack MCP server and have Claude post a message when a deployment completes. Combine this with the Stop hook from Block 8 for automatic notifications.

**Challenge 3: Database MCP**
If ai-coderrank uses a database, configure the PostgreSQL MCP server and ask Claude to explore the schema, describe the tables, and suggest index improvements.

---

> **Next up**: In Block 10, we take Claude beyond your terminal and into your CI/CD pipeline. GitHub Actions with Claude Code — automated PR reviews, issue implementation, and AI-powered workflows that run on every push.
