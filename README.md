# AI Coding Agent Review Kit

Lightweight review materials for catching risky AI-generated code before it reaches production.

**Start here:** [Worked example — reviewing a synthetic agent PR](example_agent_pr_walkthrough.md)

This repository contains:

- A free [15-minute checklist](15_minute_ai_pr_review_checklist.md) for reviewing pull requests from AI coding agents
- A [worked walkthrough](example_agent_pr_walkthrough.md) showing scope drift, weak tests, and happy-path-only handlers on a plausible diff

**Landing page:** https://skinahan.github.io/ai-coding-agent-review-kit/

## What this helps with

- Scope drift buried in "refactor" commits
- Edge cases hidden behind plausible code
- Security and permission changes without tests
- Tests that pass but do not prove behavior

## Files

| File | Purpose |
| --- | --- |
| `example_agent_pr_walkthrough.md` | Teaching walkthrough (start here) |
| `15_minute_ai_pr_review_checklist.md` | Reusable checklist |
| `docs/index.html` | Landing page |
| `example_agent_pr_walkthrough.html` | Walkthrough (GitHub Pages) |

## Feedback

If you review AI-generated PRs, share what makes them hardest to trust:

https://github.com/skinahan/ai-coding-agent-review-kit/issues/new?template=ai-review-kit-interest.yml
