---
layout: default
title: "Cost Guide"
---

<section class="section">
<div class="container container--narrow">

# Cost Guide

One of the goals of this course is to prove that AI-assisted DevOps doesn't require a big budget. Here's exactly what you'll spend.

## Claude Code: $20/month (recommended for this course)

The **Pro subscription** at $20/month is the recommended way to follow this course. It gives you full access to Claude Code across all platforms: CLI, VS Code, JetBrains, Desktop app, and web.

> **Note**: Higher-tier plans (Max at $100/mo, Team at $30/seat/mo) also include Claude Code with higher rate limits. Pro is sufficient for everything in this course.

What you get:
- Unlimited Claude Code sessions (subject to rate limits)
- Access to Sonnet 4.6 (default) and Haiku 4.5
- Opus 4.6 available (uses more of your rate limit)
- All features: plans, skills, hooks, MCP, sub-agents, scheduled tasks

> **Terminal vs. API**: Your Pro subscription covers all interactive Claude Code usage (terminal, IDE, desktop app). It does **not** cover API usage. Block 10 (GitHub Actions) requires a separate Anthropic API key with pay-per-token billing -- see the GitHub Actions section below.

### Saving tokens

- **Use `/compact` often** — compresses your conversation, saving context tokens
- **Use Haiku for sub-agents** — `model: haiku` in agent frontmatter for simple lookups
- **Use `/clear` between unrelated tasks** — don't carry unnecessary context
- **Use `--model haiku`** for simple one-shot commands: `claude --model haiku -p "what's in this file?"`
- **Check `/cost`** periodically to see your session usage

## DigitalOcean: ~$24/month

We use a single **s-2vcpu-4gb** droplet (2 vCPU, 4 GB RAM). This is the locked recommendation for the course -- do not go smaller.

| Resource | Spec | Cost |
|----------|------|------|
| Droplet | **s-2vcpu-4gb** (2 vCPU, 4GB RAM) | $24/mo |
| Domain (optional) | via DigitalOcean or external | $10-15/yr |
| Load Balancer | Not needed (k3s Traefik handles it) | $0 |

### New account credit

DigitalOcean offers **$200 in free credit** for new accounts (valid for 60 days). This is more than enough to complete the entire course for free.

### After the course

**Tear down your droplet** when you're done to stop charges immediately. Your k3s cluster, ArgoCD, and app are all disposable — the real value is in the knowledge and the code in your git repo.

```bash
# Delete the droplet via doctl
doctl compute droplet delete <droplet-id> --force

# Or just use the DO console: Droplets > your-droplet > Destroy
```

### Could you go cheaper?

Yes, but we don't recommend it for this course:

| Droplet | RAM | Cost | Works? |
|---------|-----|------|--------|
| s-1vcpu-1gb | 1GB | $6/mo | Too small for k3s + ArgoCD + app |
| s-1vcpu-2gb | 2GB | $12/mo | Tight but possible without ArgoCD |
| **s-2vcpu-4gb** | **4GB** | **$24/mo** | **Recommended. Comfortable for everything.** |
| s-2vcpu-8gb | 8GB | $48/mo | Overkill for this course |

## GitHub Actions (Block 10) -- Extra Cost

> **This block requires a separate Anthropic API key** with pay-per-token billing. This is separate from your Pro subscription.

The CI/CD block uses the Claude GitHub Action, which calls the Anthropic API directly. API pricing as of April 2026:

| Model | Input | Output |
|-------|-------|--------|
| Haiku 4.5 | $0.80/MTok | $4/MTok |
| Sonnet 4.6 | $3/MTok | $15/MTok |
| Opus 4.6 | $15/MTok | $75/MTok |

*API pricing may change -- check [anthropic.com/pricing](https://anthropic.com/pricing) for current rates.*

For the course exercises (a few PR reviews and one issue implementation), expect to spend **$1-5 total** on API tokens. Use `max_turns: 3` in your workflow to limit costs.

**Block 10 is optional.** You can skip it entirely if you want to avoid this cost -- the rest of the course works without it.

## Total Cost Summary

| Item | Cost | Required? |
|------|------|-----------|
| Claude Pro subscription | $20/mo | Yes (all blocks) |
| DigitalOcean droplet (s-2vcpu-4gb) | $24/mo (free with new account credit) | Yes (Blocks 7, 12) |
| Domain name | ~$10/yr | Optional |
| Anthropic API key (Block 10) | ~$1-5 total | **Extra cost** -- only for Block 10 |
| **Total for the course** | **~$25-45** | |

If you use DigitalOcean's new account credit, you can complete Blocks 0-9 and 11-12 for just the $20 Claude Pro subscription. Block 10 adds $1-5 in API costs. Block 13 uses only your Pro subscription (no extra cost).

</div>
</section>
