---
layout: default
title: "Cost Guide"
---

<section class="section">
<div class="container container--narrow">

# Cost Guide

One of the goals of this course is to prove that AI-assisted DevOps doesn't require a big budget. Here's exactly what you'll spend.

## Claude Code: $20/month

The **Pro subscription** at $20/month gives you full access to Claude Code across all platforms: CLI, VS Code, JetBrains, Desktop app, and web.

What you get:
- Unlimited Claude Code sessions (subject to rate limits)
- Access to Sonnet 4.6 (default) and Haiku 4.5
- Opus 4.6 available (uses more of your rate limit)
- All features: plans, skills, hooks, MCP, sub-agents, scheduled tasks

### Saving tokens

- **Use `/compact` often** — compresses your conversation, saving context tokens
- **Use Haiku for sub-agents** — `model: haiku` in agent frontmatter for simple lookups
- **Use `/clear` between unrelated tasks** — don't carry unnecessary context
- **Use `--model haiku`** for simple one-shot commands: `claude --model haiku -p "what's in this file?"`
- **Check `/cost`** periodically to see your session usage

## DigitalOcean: ~$24/month

We use a single droplet:

| Resource | Spec | Cost |
|----------|------|------|
| Droplet | s-2vcpu-4gb (2 vCPU, 4GB RAM) | $24/mo |
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

## GitHub Actions (Block 10)

The CI/CD block uses the Claude GitHub Action, which requires an **Anthropic API key** (separate from your Pro subscription). API pricing is pay-per-token:

| Model | Input | Output |
|-------|-------|--------|
| Haiku 4.5 | $0.80/MTok | $4/MTok |
| Sonnet 4.6 | $3/MTok | $15/MTok |
| Opus 4.6 | $15/MTok | $75/MTok |

For the course exercises (a few PR reviews and one issue implementation), expect to spend **$1-5 total** on API tokens. Use `max_turns: 3` in your workflow to limit costs.

You can skip Block 10 entirely if you want to avoid this cost — the rest of the course works without it.

## Total Cost Summary

| Item | Cost | Required? |
|------|------|-----------|
| Claude Pro subscription | $20/mo | Yes |
| DigitalOcean droplet | $24/mo (free with new account credit) | Yes (Blocks 7, 12) |
| Domain name | ~$10/yr | Optional |
| Anthropic API key | ~$1-5 total | Only for Block 10 |
| **Total for the course** | **~$25-45** | |

If you use DigitalOcean's new account credit, you can complete the entire course for just the $20 Claude Pro subscription.

</div>
</section>
