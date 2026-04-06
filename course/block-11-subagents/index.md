---
layout: block
title: "Sub-agents — Specialized Workers"
block_number: 11
description: "Build a team of AI specialists that Claude can delegate to -- a security reviewer, a K8s validator, and custom agents with their own tools, models, and isolation boundaries."
time: "~35 min (10 min presentation + 25 min practical)"
features:
  - Custom sub-agents (.claude/agents/)
  - Sub-agent frontmatter (model, allowed-tools)
  - Explore and Plan built-in agents
  - "isolation: worktree"
  - /agents command
objectives:
  - Understand sub-agents as specialized AI workers within a Claude Code session
  - See the built-in Explore and Plan agents in action
  - Create a security reviewer agent restricted to read-only tools
  - Create a K8s validator agent that can run kubectl dry-run
  - Use model selection to balance speed and cost across agents
  - Understand worktree isolation for safe parallel work
  - List and manage agents with /agents
---

## From One Brain to a Team

In Block 6, you built skills -- reusable commands that tell Claude _what to do_. In Block 10, you put Claude into CI so it runs automatically. This block introduces something different: sub-agents, which are separate AI workers that Claude can _delegate_ to.

Think about how a real engineering team works. You do not ask the same person to do everything. The security engineer reviews for vulnerabilities. The SRE validates the Kubernetes manifests. The tech lead plans the architecture. Each person has their own expertise, their own tools, and their own scope of responsibility.

Sub-agents bring this model to Claude Code. Instead of one Claude session doing everything, you can have specialized workers -- each with its own system prompt, tool access, context window, and even model selection. The security reviewer only gets read access. The K8s validator can run `kubectl` but cannot edit source code. The fast-draft agent uses Haiku for speed, while the architecture planner uses Opus for depth.

## What We'll Cover

1. **What sub-agents are** -- specialized AI workers running inside your session
2. **Built-in agents** -- Explore (read-only research) and Plan (architecture design)
3. **Custom agents** -- `.claude/agents/` directory and agent frontmatter
4. **Tool restrictions** -- controlling what each agent can and cannot do
5. **Model selection** -- Haiku for fast/cheap, Opus for complex reasoning
6. **Worktree isolation** -- agents working on separate git branches safely
7. **The `/agents` command** -- listing and managing your agent roster

## Why This Block Matters

There is a scaling problem with a single AI session. The more context you pile into one conversation -- security concerns, architecture decisions, test strategies, Kubernetes configs -- the more the AI has to juggle. Sub-agents solve this by decomposition. Each agent gets a focused task, a focused prompt, and a focused tool set.

There is also a trust problem. You might trust Claude to read your code and suggest changes, but do you trust it to run arbitrary bash commands while reviewing for security issues? Probably not. Sub-agents let you give each worker exactly the permissions it needs and nothing more. Principle of least privilege, applied to AI.

And there is a cost problem. Not every task needs the most powerful model. A quick code search does not need Opus. A complex architecture decision does not work well with Haiku. Sub-agents let you match the model to the task, keeping costs predictable.

## Prerequisites

- Completed Blocks 0-10 (GitHub Actions CI configured)
- The ai-coderrank project with K8s manifests and source code
- Familiarity with `.claude/` directory structure from Blocks 5-6
- kubectl configured and pointing at your k3s cluster from Block 7

## Part 1: Presentation {#presentation}

> **Duration**: ~10 minutes
> **Goal**: Students understand sub-agents as delegated AI workers with their own prompts, tools, and models -- and why decomposition beats a single overloaded session.

---

### Slide 1: The Manager and the Team

Let me give you an analogy.

You are the engineering manager. Claude Code is your right hand -- your senior engineer who can do a bit of everything. But even the best senior engineer does not personally handle every task. They delegate.

"Hey, can someone from the security team eyeball this PR?"
"Can the SRE on call validate these manifests before we ship?"
"Can someone do a quick spike on whether this architecture will work?"

Sub-agents are those specialists. They run _inside_ your Claude Code session, spawned by Claude when it needs focused expertise. Each one gets:

- **Its own system prompt** -- what it knows and how it thinks
- **Its own tool access** -- what it can and cannot do
- **Its own context window** -- a fresh, focused context for the task
- **Its own model** -- the right brain for the job

When the sub-agent finishes, it reports back to your main Claude session with its findings. Claude synthesizes the results and continues your conversation. You stay in control -- the manager who delegates, reviews, and decides.

---

### Slide 2: Built-in Agents -- Explore and Plan

Claude Code ships with two built-in agents that you have probably already seen in action:

**Explore** -- The read-only researcher.

When you ask Claude a broad question like "how does authentication work in this app?" or "what's the data flow from the API to the UI?", Claude often delegates to the Explore agent behind the scenes. This agent can read files, search code, and traverse the codebase, but it cannot modify anything. It is a scout -- sent ahead to gather intelligence and report back.

```
You: "How does the scoring algorithm work?"

Claude: [delegates to Explore agent]
  Explore: [reads 8 files, traces the data flow, returns a summary]
Claude: "Here's how scoring works..."
```

**Plan** -- The architecture designer.

When you ask Claude to plan something complex -- "design a caching layer for the leaderboard" or "plan the migration from REST to GraphQL" -- it can delegate to the Plan agent. This agent thinks through the problem, considers trade-offs, and produces a structured plan. Also read-only -- it plans but does not implement.

You have been using these without knowing it. Every time Claude pauses to "think" before a big answer, there is a good chance it delegated to one of these agents internally.

---

### Slide 3: Custom Agents -- Your Specialists

The real power is building your own. Custom agents live in `.claude/agents/`:

```
.claude/
  agents/
    security-reviewer.md     <- Checks code for OWASP top 10
    k8s-validator.md         <- Validates K8s manifests
    quick-search.md          <- Fast Haiku-powered codebase search
```

Here is what an agent definition looks like:

```markdown
---
name: security-reviewer
description: Reviews code for OWASP top 10 vulnerabilities and security best practices
model: sonnet
allowed-tools:
  - Read
  - Grep
  - Glob
---

You are a senior application security engineer conducting a security review.

Focus on the OWASP Top 10:
1. Injection (SQL, NoSQL, command, LDAP)
2. Broken Authentication
3. Sensitive Data Exposure
...
```

The frontmatter is where the magic happens:

- **`model`**: Which Claude model to use. `haiku` for fast/cheap tasks, `sonnet` for balanced, `opus` for complex reasoning. If omitted, it uses the same model as the parent session.
- **`allowed-tools`**: Tool restrictions, just like skills. A security reviewer should not need to run bash commands.
- **`description`**: Helps Claude (and you) know when to use this agent.

The body is the system prompt -- the instructions that define the agent's expertise and behavior.

---

### Slide 4: Skills vs. Agents -- What's the Difference?

Students always ask this, so let us make it clear:

| | Skills | Sub-agents |
|---|--------|------------|
| **What they are** | Reusable instructions for Claude | Separate AI workers with their own context |
| **How they run** | In your current Claude session | In a new, isolated context |
| **Context** | Shares your session's full context | Gets a fresh context window |
| **Invocation** | You type `/skill-name` | Claude delegates automatically, or you ask explicitly |
| **Model** | Uses your session's model | Can use a different model |
| **Best for** | Standardized tasks (review this, audit that) | Complex tasks needing focused attention |

Think of it this way: a skill is a recipe card. An agent is a sous-chef.

A skill says "here's how to review K8s manifests." Claude follows the instructions in its current session, with all the context of your conversation so far.

An agent says "you are a K8s expert, here are your tools, go validate these manifests." It gets its own workspace, focuses entirely on that task, and reports back.

**When to use which?** If the task is straightforward and benefits from the current conversation context, use a skill. If the task is complex, benefits from a fresh context, or needs different tools/model, use an agent.

---

### Slide 5: Worktree Isolation -- Safe Parallel Work

Here is a scenario: you ask Claude to experiment with a new approach to the leaderboard component. You want it to try something bold -- restructure the code, move files around, change the data model. But you do not want those experimental changes in your working directory. You are in the middle of something else.

Worktree isolation solves this:

```markdown
---
name: experimenter
description: Tries new approaches in an isolated worktree
isolation: worktree
---
```

When `isolation: worktree` is set, the sub-agent:

1. Creates a new git worktree (a separate checkout of the same repo)
2. Creates a new branch for its work
3. Makes all changes in that worktree, not in your working directory
4. Reports back with the branch name and a summary of changes
5. The worktree is cleaned up after

Your working directory stays exactly as it was. The agent's experimental changes live on a separate branch that you can review, merge, or discard.

This is incredibly powerful for:
- **Experimental implementations** -- "try this two different ways and let me compare"
- **Risky refactors** -- "restructure the auth module without touching my current work"
- **Parallel tasks** -- "fix this bug on a branch while I keep working here"

---

### Slide 6: Model Selection -- The Right Brain for the Job

Not every task needs the most powerful model. Sub-agents let you match model to task:

```markdown
# Quick codebase search -- fast and cheap
---
model: haiku
allowed-tools: [Read, Grep, Glob]
---

# Security review -- needs careful reasoning
---
model: sonnet
allowed-tools: [Read, Grep, Glob]
---

# Architecture planning -- complex trade-offs
---
model: opus
allowed-tools: [Read, Grep, Glob]
---
```

**Haiku** -- Fast, cheap, good for rote tasks. Searching code, formatting output, simple analysis. Runs in seconds, costs fractions of a cent.

**Sonnet** -- Balanced. Good for most review and analysis tasks. The sweet spot for code review, security scanning, and validation.

**Opus** -- Deep reasoning. Architecture decisions, complex debugging, nuanced trade-off analysis. Slower and more expensive, but sees things other models miss.

> **Cost tip**: If you have a sub-agent that runs frequently (like on every PR), make it Haiku. Reserve Opus for the tasks where the extra reasoning depth actually matters. Your wallet will thank you.

---

### Key Takeaways

| Concept | What It Is | When to Use |
|---------|-----------|-------------|
| Sub-agent | Separate AI worker with own context and tools | Complex tasks needing focused attention |
| Explore (built-in) | Read-only codebase researcher | Broad "how does X work" questions |
| Plan (built-in) | Architecture design agent | Planning complex changes |
| `.claude/agents/` | Custom agent definitions | Team-specific specialists |
| `allowed-tools` | Tool restrictions per agent | Security: limit what agents can do |
| `model` | Per-agent model selection | Balance speed, cost, and capability |
| `isolation: worktree` | Git worktree isolation | Safe experimentation without affecting your work |
| `/agents` | List all configured agents | See your full agent roster |

## Part 2: Hands-On {#practical}

> **Duration**: ~25 minutes
> **Outcome**: Two custom sub-agents (security reviewer and K8s validator), tested on real code, with model selection and worktree isolation explored, plus discovery of the built-in agents you have been using all along.
> **Prerequisites**: Completed Blocks 0-10 (GitHub Actions CI configured), the ai-coderrank project with K8s manifests and source code, kubectl configured

---

### Step 1: Watch the Explore Agent in Action (~3 min)

Before building custom agents, let us see the built-in ones at work. Start Claude Code in the ai-coderrank project:

```bash
cd ~/ai-coderrank
claude
```

Now ask a broad, research-style question:

```
How does the API layer in this project work? Trace a request from the route handler to the database and back. Show me the data flow.
```

Watch what happens. Claude will delegate to its built-in Explore agent to research the codebase. You may see it reading multiple files, searching for patterns, and tracing imports -- all in a focused research pass. The Explore agent gathers information and reports back, then Claude synthesizes it into a coherent answer for you.

The key observation: the research happened in a _separate_ context. The Explore agent got a fresh window focused entirely on answering your question, without the baggage of your previous conversation history.

> **Tip**: You can explicitly request delegation by saying "use the Explore agent to research how authentication works in this project." But Claude often delegates on its own when it recognizes a research task.

---

### Step 2: Create the Security Reviewer Agent (~5 min)

Now let us build our first custom agent. This one reviews code for OWASP Top 10 vulnerabilities and common security issues. Critically, it is read-only -- a security reviewer should observe and report, not modify code.

Create the agent file:

```bash
mkdir -p ~/ai-coderrank/.claude/agents
```

Create `.claude/agents/security-reviewer.md` with the following content:

```markdown
---
name: security-reviewer
description: Reviews code for OWASP Top 10 vulnerabilities, secret leaks, and injection risks
model: sonnet
allowed-tools:
  - Read
  - Grep
  - Glob
---

You are a senior application security engineer performing a focused security review.

## Your Mission

Systematically audit the target code for security vulnerabilities. You are thorough, specific, and practical -- you flag real risks, not theoretical possibilities with no exploitation path.

## What to Check

### 1. Injection Vulnerabilities
- SQL injection: raw queries, string concatenation in queries, unsanitized user input
- NoSQL injection: dynamic query construction with user input
- Command injection: user input passed to exec, spawn, or system calls
- LDAP injection: user input in LDAP filters

### 2. Broken Authentication
- Hardcoded credentials, API keys, or tokens in source code
- Weak password validation rules
- Missing rate limiting on auth endpoints
- Session tokens in URLs or logs

### 3. Sensitive Data Exposure
- Secrets in code (.env values, API keys, connection strings)
- PII logged to console or files
- Sensitive data in error messages returned to users
- Missing encryption for sensitive fields

### 4. Broken Access Control
- Missing authorization checks on API routes
- Direct object reference without ownership validation
- Admin endpoints accessible without role checks
- CORS misconfiguration allowing unauthorized origins

### 5. Security Misconfiguration
- Debug mode enabled in production configs
- Default credentials or settings
- Unnecessary ports or services exposed
- Missing security headers (CSP, HSTS, X-Frame-Options)

### 6. Cross-Site Scripting (XSS)
- User input rendered without sanitization
- dangerouslySetInnerHTML usage
- Unescaped template variables
- DOM manipulation with user-controlled data

### 7. Dependencies and Supply Chain
- Check package.json for known vulnerable patterns
- Look for wildcard version ranges (*)
- Check for deprecated or unmaintained packages

## Output Format

### Security Review Report

**Scope**: [files/directories reviewed]
**Risk Level**: Critical / High / Medium / Low

#### Critical Findings
| # | Vulnerability | File:Line | Description | OWASP Category | Remediation |
|---|--------------|-----------|-------------|---------------|-------------|

#### High-Risk Findings
(same table format)

#### Medium-Risk Findings
(same table format)

#### Low-Risk / Informational
(same table format)

#### Summary
- Total files scanned: N
- Critical: N | High: N | Medium: N | Low: N
- Top 3 priorities for remediation
```

> **Notice three things about this agent**: (1) It uses `model: sonnet` -- good enough for security analysis without the cost of Opus. (2) It is restricted to `Read`, `Grep`, and `Glob` -- it cannot modify files, run commands, or access the network. (3) The system prompt is long and specific -- this is where expertise lives.

---

### Step 3: Create the K8s Validator Agent (~5 min)

This agent validates Kubernetes manifests against best practices. Unlike the security reviewer, this one _can_ run bash commands -- specifically `kubectl` for dry-run validation.

Create `.claude/agents/k8s-validator.md`:

```markdown
---
name: k8s-validator
description: Validates Kubernetes manifests against best practices using kubectl dry-run and static analysis
model: sonnet
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---

You are a senior SRE validating Kubernetes manifests before they are applied to a production cluster.

## Your Mission

Perform both static analysis (reading the YAML) and dynamic validation (kubectl dry-run) on all Kubernetes manifests in the project.

## Step 1: Static Analysis

Use Glob to find all YAML files in the `k8s/` directory. Read each file and check:

### Resource Definitions
- CPU and memory requests AND limits are set on all containers
- Requests <= limits (requests should not exceed limits)
- Values are reasonable for the workload type (a web server does not need 4Gi memory)

### Security Context
- `runAsNonRoot: true` is set
- `readOnlyRootFilesystem: true` where possible
- `allowPrivilegeEscalation: false` is set
- Capabilities are dropped (`drop: ["ALL"]`), only necessary ones added back

### High Availability
- Deployment replicas > 1 for production services
- Pod Disruption Budget exists for critical services
- Anti-affinity rules prevent all replicas on the same node

### Health Checks
- Liveness probe is configured (with appropriate failure threshold)
- Readiness probe is configured (separate from liveness)
- Startup probe for containers with slow initialization

### Image Policy
- No `latest` tag -- images must be pinned to a specific version or SHA
- `imagePullPolicy` is set appropriately
- Images come from an expected registry

### Labels and Metadata
- Standard labels present: `app.kubernetes.io/name`, `app.kubernetes.io/version`, `app.kubernetes.io/component`
- Labels are consistent across related resources (Deployment, Service, HPA)
- Namespace is explicitly set

## Step 2: Dynamic Validation

Run kubectl dry-run against each manifest to catch schema and structural errors:

```bash
kubectl apply --dry-run=client -f <manifest-file> 2>&1
```

If kubeconfig is available and the cluster is reachable, also run server-side dry-run:

```bash
kubectl apply --dry-run=server -f <manifest-file> 2>&1
```

Report any errors or warnings from kubectl.

## Step 3: Cross-Resource Validation

- Service selectors match Deployment labels
- Ingress/Route backends reference existing Services
- ConfigMap and Secret references in Deployments actually exist
- PVC names in volume mounts match existing PVC definitions
- HPA targets reference existing Deployments

## Output Format

### Kubernetes Validation Report

**Manifests Found**: N files in k8s/
**kubectl Dry-Run**: PASS / FAIL (with errors)

#### Per-Manifest Results

##### `<filename>`
| Check | Status | Details |
|-------|--------|---------|
| Resource Limits | PASS/WARN/FAIL | ... |
| Security Context | PASS/WARN/FAIL | ... |
| Health Checks | PASS/WARN/FAIL | ... |
| Image Policy | PASS/WARN/FAIL | ... |
| Labels | PASS/WARN/FAIL | ... |
| kubectl dry-run | PASS/FAIL | ... |

#### Cross-Resource Issues
| Issue | Resources Involved | Impact |
|-------|-------------------|--------|
| ... | ... | ... |

#### Summary
- Total manifests: N
- Passed all checks: N
- Has warnings: N
- Has failures: N
- **Recommended fix order** (most impactful first):
  1. ...
  2. ...
  3. ...
```

> **Key difference from the security reviewer**: This agent has `Bash` in its `allowed-tools`. It needs to run `kubectl` commands for dynamic validation. The security reviewer intentionally does not have bash access -- it should never execute anything, only read and report.

---

### Step 4: Test the Security Reviewer (~3 min)

Time to put your security reviewer to work. In your Claude Code session:

```
Run the security reviewer agent on the src/ directory. Focus on the API routes and any authentication-related code.
```

Watch Claude delegate to the security-reviewer agent. The agent will:
1. Glob for files in `src/`
2. Read API route handlers
3. Search for common vulnerability patterns (raw SQL, unsanitized input, hardcoded secrets)
4. Produce a structured report with findings categorized by OWASP category

Read the report carefully. Common findings in a typical Next.js project:
- Missing input validation on API routes
- CORS not explicitly configured
- Error messages leaking implementation details
- No rate limiting on public endpoints

> **Insight**: This agent found issues by _reasoning about security patterns_, not by running a linter. It understands intent, not just syntax. It can tell you "this API route accepts user input and passes it to a database query without validation" -- something no static analysis tool would phrase that clearly.

---

### Step 5: Test the K8s Validator (~3 min)

Now test the K8s validator:

```
Run the k8s-validator agent to validate all Kubernetes manifests in k8s/
```

This agent does two things the security reviewer cannot:
1. **Reads the manifests** (static analysis -- same as the security reviewer)
2. **Runs `kubectl apply --dry-run`** (dynamic validation -- only possible because it has Bash access)

The dynamic validation catches errors that reading YAML alone cannot -- invalid API versions, malformed selectors, schema violations, resource types that do not exist in your cluster version.

Compare the output to what you got from `/review-k8s` in Block 6. The skill gave you a static review. The agent gives you static review _plus_ dynamic validation. Same task, deeper analysis.

---

### Step 6: Create a Fast Agent with Haiku (~3 min)

Not every agent needs heavy reasoning. Let us create a fast, cheap agent for quick codebase searches.

Create `.claude/agents/quick-search.md`:

```markdown
---
name: quick-search
description: Fast codebase search using Haiku for speed and low cost
model: haiku
allowed-tools:
  - Read
  - Grep
  - Glob
---

You are a fast codebase search assistant. Your job is to find things quickly and report back concisely.

When asked to find something:
1. Use Grep to search for the pattern across the codebase
2. Use Glob if you need to find files by name
3. Use Read only if you need to check specific context around a match
4. Report your findings in a brief, structured format

Keep your responses short. List the file paths and line numbers where you found matches. Include a one-line summary of each match for context. Do not explain the code -- just locate it.

## Output Format

**Query**: [what was searched for]
**Matches**: N results

| File | Line | Context |
|------|------|---------|
| path/to/file.ts | 42 | `const score = calculateScore(...)` |
| ... | ... | ... |
```

Test it:

```
Use the quick-search agent to find all places where environment variables are read in the codebase.
```

Notice how much faster this responds compared to a full Sonnet or Opus query. Haiku is ideal for tasks where you need speed and the reasoning is straightforward -- finding, listing, simple pattern matching.

> **Cost comparison**: A Haiku agent call for a simple search costs a fraction of a cent. The same task on Opus might cost 10-20x more. For repetitive, simple tasks, the savings add up fast.

---

### Step 7: Demo Worktree Isolation (~5 min)

Worktree isolation is one of the most powerful sub-agent features. It lets an agent work on a separate git branch without affecting your working directory.

Try this in your Claude Code session:

```
Create a new sub-agent with worktree isolation. Have it experiment with refactoring the score formatting utilities in src/utils/format-score.ts into a class-based approach. I want to compare the result with the current functional approach without changing my working directory.
```

Claude will:
1. Create a git worktree (a separate checkout of your repo on a new branch)
2. Delegate to a sub-agent running in that worktree
3. The agent makes changes in the isolated worktree
4. When it finishes, it reports back with the branch name and a summary
5. Your working directory is untouched

You can then compare the two approaches:

```bash
# See what the agent changed
git diff main...<branch-name-from-agent>

# If you like it, merge it
git merge <branch-name-from-agent>

# If you don't, just delete the branch
git branch -D <branch-name-from-agent>
```

This is incredibly useful for:
- "Try it both ways and let me compare"
- "Refactor this module without breaking what I'm working on"
- "Explore a risky change without committing to it"

> **How worktree isolation works under the hood**: Git worktrees are a native git feature (`git worktree add`). They create a second checkout of the same repo in a different directory, sharing the same `.git` history. The sub-agent works in that second directory, so changes happen on a different branch in a different folder. Your working directory never sees the changes unless you explicitly merge.

---

### Step 8: List Your Agents with `/agents` (~1 min)

Now that you have built several agents, let us see the full roster. In Claude Code:

```
/agents
```

This lists all configured agents -- both built-in and custom. You should see:

```
Built-in Agents:
  - Explore    Read-only codebase research
  - Plan       Architecture design and planning

Custom Agents (.claude/agents/):
  - security-reviewer   Reviews code for OWASP top 10 vulnerabilities
  - k8s-validator       Validates K8s manifests with kubectl dry-run
  - quick-search        Fast Haiku-powered codebase search
```

Each entry shows the agent name, its description, and its model/tool configuration. This is your team roster -- the specialists that Claude can delegate to at any time.

---

### Checkpoint

Your `.claude/agents/` directory should now contain:

```
.claude/
  agents/
    security-reviewer.md    <- OWASP security review (Sonnet, read-only)
    k8s-validator.md        <- K8s manifest validation (Sonnet, with Bash)
    quick-search.md         <- Fast codebase search (Haiku, read-only)
```

You have also experienced:
- The built-in Explore agent researching your codebase
- Sub-agent delegation happening automatically
- Model selection matching brain-power to task complexity
- Worktree isolation for safe experimentation

Your Claude Code setup now has skills (Block 6) for standardized workflows and agents (this block) for specialized delegation. Together, they cover both "do this task" and "delegate this task to a specialist."

---

### Bonus Challenges

**Challenge 1: Create a test-writer agent**
Build an agent in `.claude/agents/test-writer.md` that generates tests for any given file. Give it access to `Read`, `Grep`, `Glob`, `Edit`, and `Write` (it needs to create test files). Have it follow the testing conventions from your CLAUDE.md and the patterns in your existing tests.

**Challenge 2: Agent pipeline**
Chain two agents: first run the security-reviewer on `src/`, then pass its findings to Claude and ask it to fix the critical issues. Compare this to doing both tasks in a single Claude session -- does the separation lead to a better result?

**Challenge 3: Create a documentation agent**
Build a read-only agent (`model: haiku`) that generates JSDoc comments for functions that lack them. Test it on a file with undocumented functions. This is a great example of a Haiku-appropriate task -- simple, repetitive, and pattern-based.

**Challenge 4: Explore model differences**
Run the same security review with `model: haiku`, `model: sonnet`, and `model: opus`. Compare the depth and quality of findings. This gives you a practical feel for when each model is worth the cost.

---

> **Next up**: In Block 12, we bring everything together with GitOps -- ArgoCD, automated deployments, and the full loop from code change to production. Your CI reviews PRs with Claude, your agents validate the manifests, and ArgoCD syncs the result to your cluster.
