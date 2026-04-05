---
layout: block
title: "GitHub Actions & CI/CD — Claude in the Pipeline"
block_number: 10
description: "Take Claude Code beyond your terminal and into GitHub Actions. Set up automated PR reviews, issue-to-PR workflows, and AI-powered CI/CD with the official Claude Code GitHub Action."
time: "~35 min (10 min presentation + 25 min practical)"
features:
  - anthropics/claude-code-action@v1
  - /install-github-app
  - "@claude trigger"
  - CLAUDE.md in CI
  - "--max-turns cost control"
objectives:
  - Understand how Claude Code runs inside GitHub Actions
  - Install the Claude GitHub App using /install-github-app
  - Create a workflow that triggers Claude on @claude mentions in PRs
  - See Claude automatically review a pull request
  - Watch Claude create a PR from a GitHub issue
  - Configure CLAUDE.md guidelines for CI context
  - Control costs with --max-turns and other safeguards
---

## From Your Terminal to Every Pull Request

Up to this point, Claude Code has been your personal co-pilot. You open a terminal, start a session, ask questions, make changes. It is powerful, but it is inherently local. It scales to one person at a time.

This block changes that. You are about to put Claude inside your CI/CD pipeline, where it can review every pull request, respond to issues, and act as a tireless team member who never takes a vacation day, never gets paged at 3 AM and forgets to check the error logs, and never says "I'll review that PR tomorrow" and then doesn't.

Here is a fun fact that might surprise you: some engineering teams have configured Claude as their _first_ reviewer on every PR. Not their only reviewer, but their first. By the time a human reviewer looks at the PR, Claude has already flagged missing tests, security issues, and style violations. The human reviewer can focus on architecture and business logic instead of nitpicking import order.

## What We'll Cover

1. **The official GitHub Action** -- `anthropics/claude-code-action@v1` and what it does
2. **The GitHub App** -- `/install-github-app` and one-click setup
3. **The `@claude` trigger** -- how to summon Claude in any PR or issue
4. **Automated PR review** -- Claude comments on PRs with structured feedback
5. **Issue-to-PR workflow** -- tag an issue with `@claude` and it generates a PR
6. **CLAUDE.md in CI** -- your project rules apply in the pipeline too
7. **Cost control** -- `--max-turns`, token limits, and keeping your bill predictable

## Why This Block Matters

There is an enormous gap between "we should review PRs more carefully" and "we actually review every PR carefully." Humans get tired, busy, and distracted. CI does not. Linters catch syntax issues but miss logic problems. Claude catches both.

This is not about replacing human reviewers. It is about giving every PR a thorough first pass so that human reviewers can do their best work on the things that actually need human judgment. Think of Claude in CI as the spell-checker for your code review process -- it catches the obvious stuff so you can focus on the subtle stuff.

By the end of this block, every PR to your ai-coderrank repo will get an automatic review, and you will be able to create implementation PRs just by writing a GitHub issue and tagging `@claude`.

## Prerequisites

- Completed Blocks 0-9 (codebase deployed, hooks and MCP configured)
- The ai-coderrank repository pushed to GitHub
- GitHub account with permissions to install apps and create workflows
- Anthropic API key (for the action's authentication)
