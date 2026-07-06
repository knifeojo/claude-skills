---
name: ship-report
description: "Generates a report of what shipped to production across Pesa's repos for a period (default: last week). ALWAYS trigger when Nifemi says any of: 'what shipped', 'ship report', 'shipped report', 'what did we ship', 'what went to production', 'weekly release report', 'release report', 'management update on what shipped', 'what did we ship this week/last week', or asks to summarize shipped/released/deployed work for a week, sprint, or date range. Scans merged PRs to the production branch across configured GitHub repos via the gh CLI, groups them by product workstream, and produces a management-ready summary (default) plus an optional detailed engineering digest — flagging reverts, mobile releases, and security items."
---

# Ship Report Skill

## Purpose
Produce a clear, audience-appropriate report of **what shipped to production** across Pesa's repos for a given period. Two audiences, one command:
- **Management report (default):** outcomes and business impact, grouped by product workstream, no PR numbers or eng jargon — paste-ready for a leadership call, Slack, or Notion.
- **Detailed engineering digest (on request):** per-repo PR list with authors, Jira refs, and the noise (deps/CI/chore) included.

## Trigger phrases
"what shipped" / "ship report" / "what did we ship" / "what went to production" / "weekly release report" / "management update on what shipped"

---

## ⚠️ Pre-flight
1. **Read `config.yaml`** from this skill directory first. Everything marked `[FROM CONFIG]` comes from it. If `config.yaml` is missing, fall back to `config.example.yaml` defaults and tell the user to create their own.
2. Confirm `gh` is authenticated: `gh auth status`. If not, stop and tell the user to run `gh auth login` (suggest they use `! gh auth login`). The gh CLI uses a personal token, so this works even where org-level Claude Code access is blocked.
3. **READ-ONLY.** This skill only reads Git/GitHub. Never push, commit, comment, or modify anything.

---

## Step 1 — Resolve the reporting window

Default window is **last calendar week: Monday 00:00 → Sunday 23:59** in the configured `timezone`.

Compute with the shell (do not hardcode dates):
```bash
# Last week's Monday and Sunday (macOS/BSD date)
MON=$(date -v-mon -v-7d +%Y-%m-%d)   # most recent Monday, then back 7 days
SUN=$(date -v-sun +%Y-%m-%d)          # if today is not Sunday, gives last Sunday
```
Accept explicit overrides from the user's request:
- "this week" → current Monday → today
- "last week" → previous Mon–Sun (default)
- "in June" / "since <date>" / "between X and Y" → parse to a `YYYY-MM-DD..YYYY-MM-DD` range
- "this sprint" → if the user gives sprint dates use them; otherwise ask.

State the resolved window at the top of the report so the reader knows the scope.

## Step 2 — Pull merged PRs per repo

For each repo in `[FROM CONFIG: repos]`, list PRs merged into `[FROM CONFIG: default_branch]` within the window:
```bash
gh pr list --repo <org>/<repo> --state merged --base <default_branch> \
  --search "merged:<MON>..<SUN>" --limit 100 \
  --json number,title,author,mergedAt,labels,url
```
Run repos in parallel. Skip repos that return zero merges. If a repo errors (access/deleted), note "<repo>: unavailable" and continue — never fail the whole report on one repo.

> **Production caveat:** merging to `main` deploys backend services to Cloud Run immediately, but **mobile** code only reaches users when an app-store build ships. If mobile has a version-bump PR (title matches `bump version` / `release notes` / `x.y.z+build`), treat that as "a release was cut" and add a watch item to confirm store availability — do not claim mobile changes are live to users.

## Step 3 — Enrich & classify

For each PR:
- Extract any **Jira ref** (`[PT-123]`, `PT-123`) from the title.
- Read the **conventional-commit scope** from the title (`feat(scope): ...`, `fix(scope): ...`).
- Map it to a **workstream** using `[FROM CONFIG: workstreams]` (scope/keyword → bucket). If nothing matches, bucket as "Other / misc".
- Mark as **noise** if the scope is in `[FROM CONFIG: exec_exclude_scopes]` (deps, ci, chore, test, format, renovate). Noise is excluded from the exec view, kept in the detailed digest.

## Step 4 — Detect watch items

Scan for things leadership should know:
- **Reverts / re-merges:** titles containing `Revert` — pair them with the original and the re-apply if present. A merge→revert→re-merge is a stability flag.
- **Mobile release cut:** version-bump PR (see Step 2 caveat).
- **Security / compliance:** scopes/keywords `security`, `sanctions`, `kyc`, `privacy`, `limits`, `compliance`.
- **Reliability fixes promoted to prod:** titles with `deadlock`, `retry`, `pool`, `perf`, `incident`, `promote`.
- **Heavy CS/admin tooling:** several `admin`/`cli` PRs in one week → possible product gap to productize.

## Step 5 — Compose the report

### Default: Management report
Use this structure. Lead with impact, drop PR numbers, translate scopes into plain outcomes. Group only the workstreams that actually had activity. 4–7 buckets max — merge thin ones.

```
🚀 *What we shipped — week of [Mon date]*
_[N] changes across [repos with activity]. [One-line theme of the week.]_

*TL;DR:* [1–2 sentences: the story of the week in business terms.]

*1. [Workstream]*
• [Outcome in plain language — what changed and why it matters to users/business]
• [Outcome …]

*2. [Workstream]*
• …

… (repeat for active workstreams)

🚩 *Watch items*
• [Reverts/stability, mobile release status, security/compliance, CS-tooling signal]
```

Rules for the exec view:
- Every bullet is an **outcome**, not a task. "We can now change sign-up rules without a new app release," not "feat(auth): serve registration-rules."
- No PR numbers, no author names, no scopes, no dep bumps/CI/formatting.
- Tie bullets to Pesa strategy where honest (multi-currency wallet, corridors, savings launch, compliance).
- Keep it scannable — a leader should absorb it in under a minute.

### On request: Detailed engineering digest
If the user asks for "detail", "full list", "engineering version", or "with PR numbers", append (or replace with) a per-repo breakdown:
```
📦 *[repo]* — [n] merged
  [date]  #[num]  [author]  [title]  [PT-ref]
  …
```
Include the noise (deps/CI/chore) here, grouped at the end as "Housekeeping (n): dependency bumps, CI, formatting."

## Step 6 — Deliver

- If `[FROM CONFIG: output]` is `print` (default): output the report in the chat as markdown.
- If `output` is `slack`: DM it to `[FROM CONFIG: slack_user_id]` via `Slack:slack_send_message` (channel_id = the user's own ID = a self-DM). If Slack isn't connected, fall back to printing and say so.
- Always offer: (a) switch to the detailed digest, (b) drop it into a Notion page, (c) narrow to a specific workstream.

---

## Guardrails
- **Read-only.** gh read commands only. Never push/comment/modify.
- **No secrets / PII** in output — you're only handling PR titles and metadata, but still never paste tokens, internal URLs, or personal data if they appear in a title.
- **Honest sourcing:** if a repo was unavailable, say so — don't imply full coverage. If zero PRs merged in the window, say "Nothing shipped in this window" rather than padding.
- **Don't overclaim mobile as live** — see the Step 2 caveat.

## Quality bar
Before finishing, verify:
- [ ] `config.yaml` read; window resolved and stated at the top
- [ ] All configured repos scanned (or unavailability noted)
- [ ] Exec view: outcomes not tasks, no PR numbers/jargon, ≤7 buckets
- [ ] Watch items surfaced (reverts, mobile release, security, CS-tooling signal)
- [ ] Offered the detailed digest / Notion / workstream drill-down
