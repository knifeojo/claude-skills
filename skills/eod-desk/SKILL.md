---
name: eod-desk
description: "Nifemi's end-of-day wrap: what moved today, what's still open, and what to prep for tomorrow — the evening counterpart to the morning brief. ALWAYS trigger when Nifemi says: 'eod', 'end of day', 'wrap up my day', 'close out the day', 'what did I get done', 'eod desk', or asks for an end-of-day summary. Delivered as a Slack DM (or printed), it reconciles today's todos, summarizes progress across tools, and sets up tomorrow."
---

# EOD Desk Skill

## Purpose
Close Nifemi's day: reconcile what got done against the morning todos, capture what moved across the tools **today**, surface what's still open or waiting on him, and tee up tomorrow. Lighter than the morning brief — it's a wrap + a launchpad, not a full intelligence sweep.

## Trigger phrases
"eod" / "end of day" / "wrap up my day" / "close out the day" / "what did I get done" / "eod desk"

---

## Fixed Pesa resources (baked in)
Reuses the morning-brief config where present (`slack_user_id`, Notion DB IDs, Jira `PT`). Read `config.yaml` first.

---

## Steps (today only — `since: start of today` in the configured timezone)

1. **Reconcile the day's todos.** Read the rolling "Today's Todos" Canvas (from morning-brief). Mark what's ✅ done vs ☐ still open (cross-check Slack/Jira/Notion for evidence it was actually handled). Carry unfinished items forward.
2. **What moved today** (concise, action-relevant only):
   - **Slack:** decisions, escalations, unanswered mentions/DMs from today.
   - **Jira PT:** issues moved to Done/In Review today; new blockers; anything assigned to or awaiting Nifemi.
   - **Notion:** PRDs/initiatives that changed status today.
   - **GitHub (optional):** PRs merged today (ties to ship-report) and PRs now awaiting his review.
   - **Email:** anything still needing a reply today (overdue flagged).
3. **Still open / waiting on you:** the short list of things that didn't close — unanswered DMs, pending decisions, PRs to review, overdue emails.
4. **Set up tomorrow:** tomorrow's calendar (first meetings + prep needed), and the top 3 things to start with, seeded from carry-overs + today's self-notes.

## Output
One Slack DM (to `slack_user_id`) — or printed if no Slack — structured:
```
🌙 EOD — [DAY, DATE]
✅ Done today: [n] — [highlights]
⏳ Still open / waiting on you: [list]
📊 What moved: [Jira / Notion / Slack / PRs — 1 line each]
🔜 Tomorrow: [first meeting + prep] · Top 3 to start with
```
Update the Todos Canvas with the carried-forward items so the morning brief picks them up.

## Guardrails
- **Today only** — don't re-summarize the whole week; the morning brief covers overnight.
- Source-honest: only report what a tool actually returned; flag unavailable sources.
- Don't invent completion — verify a todo is done before marking it ✅.
- Read-only except the Slack DM + Todos Canvas update.

## Quality bar
- [ ] Todos reconciled (done vs carried) with evidence
- [ ] Today's movements captured concisely, action-relevant only
- [ ] Open/waiting list is short and real
- [ ] Tomorrow's first meeting + top 3 set
- [ ] DM sent (or printed) + Canvas carried forward
