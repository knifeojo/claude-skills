# 🚀 ship-report

Reports **what shipped to production** across Pesa's repos for a period (default: last week), grouped by product workstream.

Two audiences, one command:
- **Management report (default)** — outcomes and business impact, no PR numbers or jargon. Paste-ready for a leadership call.
- **Detailed engineering digest (on request)** — per-repo PR list with authors, Jira refs, and housekeeping.

## Usage

Just ask, in Claude Code:
- "what shipped last week"
- "ship report"
- "management update on what we shipped"
- "what did we ship in June" / "…this sprint"
- add "with detail" / "full list" for the engineering digest

## Setup

```bash
cd skills/ship-report
cp config.example.yaml config.yaml   # fill in the personal section
```

Requires the `gh` CLI authenticated to an account with read access to the org repos:
```bash
gh auth status   # or: gh auth login
```
`gh` uses a personal token, so this works even where org-level Claude Code access is blocked.

## How it works

1. Resolves the reporting window (default: last Mon–Sun in your timezone).
2. Lists PRs merged to `main` across the configured repos via `gh` (read-only).
3. Classifies each PR into a workstream and filters dependency/CI/formatting noise out of the exec view.
4. Flags watch items: reverts/re-merges, mobile releases (code ≠ live until the store build ships), security/compliance, reliability fixes, and heavy CS-tooling weeks.

**Read-only** — never pushes, comments, or modifies anything.

## Autonomous

Once org Claude Code access is enabled, schedule it as a Monday-morning routine to auto-post the weekly ship report.
