# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Claude Code 101 — a hands-on course site teaching DevOps/SRE/QA/developers to use Claude Code. Built with Jekyll, hosted on GitHub Pages. Licensed under MIT.

## Build & Serve

```bash
bundle install            # Install Jekyll dependencies
bundle exec jekyll serve  # Serve locally at localhost:4000/Claude101/
```

## Architecture

- **Jekyll** static site with GitHub Pages deployment
- **`_layouts/`**: `default.html` (base) and `block.html` (course block pages)
- **`_includes/`**: `head.html`, `header.html`, `footer.html`, `block-nav.html`
- **`course/block-NN-slug/`**: Each block has an `index.md` overview plus dedicated `presentation.md` and `hands-on.md` pages
- **`resources/`**: Cheatsheet, cost guide, troubleshooting
- **`assets/css/style.css`**: Full design system (Anthropic-inspired palette)

## Content Conventions

- Course content is markdown with Jekyll front matter
- Block overview pages use `layout: block`; presentation and hands-on pages use `layout: block-part`
- Tone: energetic educator — analogies, fun facts, not dry
- Each block has three learner entry points: overview, presentation, and hands-on
- Complexity increases linearly from Block 0 (basics) to Block 13 (advanced)

## Design System

Colors: `#faf9f5` (cream bg), `#f0eee6` (card bg), `#2c2b25` (dark), `#c96442` (accent terracotta), `#3b6b5e` (success sage)
Fonts: DM Serif Display (headings), Inter (body), JetBrains Mono (code)
