---
layout: default
title: "Troubleshooting"
---

<section class="section">
<div class="container container--narrow">

# Troubleshooting

Common issues you might hit during the course, and how to fix them.

## Claude Code

### "Claude is not responding" / Session hangs

```bash
# Kill and restart
Ctrl+C  # Interrupt current turn
/clear  # Start fresh if needed
```

If Claude Code itself is unresponsive, close the terminal and start a new session.

### "Permission denied" when Claude tries to run a command

This is by design. Claude asks permission before running commands. You can:
- **Allow once**: press Enter or type `y`
- **Allow for session**: type `a` (allows this tool for the rest of the session)
- **Change permission mode**: `Shift+Tab` to cycle modes, or start with `--permission-mode acceptEdits`

### CLAUDE.md not loading

```bash
# Check what's loaded
/memory

# Common issues:
# - File is in wrong location (should be ./CLAUDE.md or ./.claude/CLAUDE.md)
# - File is excluded in _config.yml or settings
# - Syntax error in @path imports
```

### Auto memory not working

Auto memory requires Claude Code v2.1.59+. Check your version:
```bash
claude --version
claude update  # Update if needed
```

Also verify it's enabled:
```bash
# In ~/.claude/settings.json
{
  "autoMemoryEnabled": true
}
```

### Context getting too long / Claude losing track

```bash
/compact  # Compress conversation, keep key context
```

Use `/compact` proactively — don't wait until Claude starts forgetting things.

### Skills not showing up

```bash
# Skills must be in the right location:
# Project: .claude/skills/skill-name/SKILL.md
# User:    ~/.claude/skills/skill-name/SKILL.md

# Check with:
ls .claude/skills/*/SKILL.md

# Reload if needed — start a new session
```

### Hooks not firing

```bash
# Enable verbose mode to see hook execution:
Ctrl+O

# Check settings.json syntax:
claude -p "validate the JSON in .claude/settings.json"

# Common issues:
# - Incorrect event name (PreToolUse, not preToolUse)
# - Matcher not matching (check tool names exactly)
# - Hook script not executable (chmod +x)
# - Exit code wrong (0 = allow, 2 = block)
```

### MCP server not connecting

```bash
# Check config location: .claude/mcp.json
# Restart session after changing mcp.json
# Verify the server binary/command exists

# Common issues:
# - Missing environment variables (GITHUB_TOKEN, etc.)
# - Wrong server command path
# - Server not installed globally
```

## DigitalOcean & k3s

### Can't SSH into droplet

```bash
# Check your SSH key
ssh -v root@<droplet-ip>

# Common fixes:
# - Wrong IP address (check DO console)
# - SSH key not added during droplet creation
# - Firewall blocking port 22
```

### k3s not starting

```bash
# Check status
sudo systemctl status k3s

# Check logs
sudo journalctl -u k3s -f

# Common issues:
# - Not enough RAM (need at least 2GB)
# - Port 6443 already in use
# - Firewall blocking required ports
```

### kubectl connection refused

```bash
# On the droplet, k3s uses its own kubeconfig:
sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml get nodes

# For local access, fix the server address:
# In your local kubeconfig, change 127.0.0.1 to your droplet's public IP
```

### Pods stuck in ImagePullBackOff

```bash
kubectl describe pod <pod-name> -n ai-coderrank

# Common causes:
# - Image doesn't exist in registry
# - Private registry without credentials
# - Image tag typo
# - Docker Hub rate limiting (use ghcr.io instead)
```

### Pods stuck in CrashLoopBackOff

```bash
kubectl logs <pod-name> -n ai-coderrank --previous

# Common causes:
# - App crashing on startup (check env vars, config)
# - Missing ConfigMap or Secret
# - Port mismatch between container and service
```

### App not accessible on public IP

```bash
# Check service type (should be NodePort or through Ingress):
kubectl get svc -n ai-coderrank

# Check if the port is open on the droplet:
curl http://localhost:<nodeport>

# Check DigitalOcean firewall:
# DO Console > Networking > Firewalls > ensure NodePort range is allowed
```

## ArgoCD

### ArgoCD UI not accessible

```bash
# Port-forward:
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### ArgoCD not syncing

```bash
# Check application status:
kubectl get app -n argocd

# Force sync:
kubectl patch app ai-coderrank -n argocd --type merge -p '{"operation":{"sync":{}}}'

# Common issues:
# - Repo URL typo in application.yaml
# - Private repo needs credentials
# - Branch name mismatch
# - Path doesn't match actual directory structure
```

### ArgoCD showing "Degraded" health

```bash
# Check what's unhealthy:
kubectl get app ai-coderrank -n argocd -o yaml | grep -A 20 health

# Usually means a pod is failing — check pod logs
kubectl logs -n ai-coderrank -l app=ai-coderrank --tail=50
```

## GitHub Actions

### Claude GitHub Action not triggering

```bash
# Check:
# 1. Workflow file is in .github/workflows/
# 2. The trigger matches (issue_comment, pull_request_review_comment)
# 3. The if condition matches (contains 'claude' or '@claude')
# 4. ANTHROPIC_API_KEY secret is set in repo settings
# 5. GitHub App permissions are correct
```

### Claude Action running but producing no output

```bash
# Check the Actions log in GitHub UI
# Common issues:
# - API key invalid or expired
# - Rate limited (check Anthropic dashboard)
# - max_turns set too low
# - CLAUDE.md has restrictive instructions
```

## General Tips

1. **When in doubt, ask Claude**: `"I'm getting this error: <paste error>. What's wrong?"`
2. **Check the official docs**: [docs.anthropic.com/en/docs/claude-code/overview](https://docs.anthropic.com/en/docs/claude-code/overview)
3. **Start fresh**: `/clear` fixes most session-level issues
4. **Update**: `claude update` ensures you have the latest fixes
5. **Verbose mode**: `Ctrl+O` shows what's happening under the hood

</div>
</section>
