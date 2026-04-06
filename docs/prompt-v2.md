# Prompt v2 — Claude Code 101 Content Generation

## Rules for content generation

1. **Official source verification**: Every Claude Code feature, command, flag, or behavior claimed in course content MUST be verifiable against https://docs.anthropic.com/en/docs/claude-code/. If uncertain, mark as "needs verification."

2. **Date all facts**: All pricing, feature availability, and product claims must include "as of April 2026" or the relevant verification date.

3. **Single default path**: Always use the locked lab architecture:
   - DigitalOcean s-2vcpu-4gb, Ubuntu 22.04
   - k3s, Traefik (built-in), ArgoCD
   - GHCR for container images
   - GitHub Actions for CI/CD
   - Alternatives go in "Optional Sidebar" callouts only

4. **Separate course choices from product facts**:
   - "We use Sonnet for this course" = course choice
   - "Sonnet is the default model in Claude Code" = product fact
   - Never mix these. Label each clearly.

5. **Cost transparency**: Any step that costs money beyond the $20 Pro subscription must be flagged with a cost callout.

6. **Experimental features**: Anything marked experimental, research preview, or beta in official docs must be labeled as such in our content. Never present experimental features as stable.

7. **Deliverables per block**: Each block ships as three pages:
   - `index.md` — short overview, context, prerequisites, and links to the two formats
   - `presentation.md` — timed slides with talking points, fun facts, analogies
   - `hands-on.md` — numbered steps with exact commands/prompts, expected output, common failure modes

8. **Block contract**: Every Hands-On page must include:
   - Duration, Outcome, and Prerequisites in a blockquote header
   - Expected result: what success looks like
   - Common failure modes: top 3 things that can go wrong
   - Handoff: transition to the next block
   - Copy-pasteable code blocks for terminal commands and Claude prompts

9. **Tone**: Energetic educator. Analogies, hooks, fun facts. Not dry, not marketing. Like a senior engineer excited to share their toolkit.

10. **No placeholders**: Every command, file path, and config example must be complete and runnable. No "replace with your..." unless it's truly variable (like IP addresses).
