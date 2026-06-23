# 🌅 Morning Brief Skill

A comprehensive daily intelligence sweep across every tool in the Pesa stack, delivered as a Slack DM every weekday morning.

## What it checks

| Source | What it looks for |
|---|---|
| **Intercom** | Support volume, recurring themes, provider failures, fraud signals, unanswered conversations |
| **Gmail** | Action items, overdue emails, shared docs, deadlines |
| **Google Drive** | Meeting notes from yesterday, action items assigned to you |
| **Notion** | All movement in PRDs, Initiatives, OKRs, Product Requests, and Wiki pages |
| **Slack** | Every channel + every DM — decisions, blockers, partner updates, mentions |
| **Google Calendar** | Today's meetings with prep context |
| **Jira** | Open tickets, blockers, active epics |
| **Fireflies** | Meeting transcripts and action items |

## Output

A structured Slack DM to yourself with:
- 🚨 Critical items that need you today
- Section-by-section intelligence summary
- ✅ Prioritised todo list (🔴 Must Do / 🟡 Important / 🟢 Watch)
- ♻️ Carry-overs from previous briefs

## Setup

```bash
cp config.example.yaml config.yaml
```

Fill in `config.yaml`:
- `slack_user_id` — your Slack member ID (Slack → your profile → `...` → Copy member ID)
- `jira_assignee_id` — your Atlassian account ID (visible in your Jira profile URL)
- `timezone` — your local timezone
- Toggle any sources you don't have access to

## Running autonomously

In Claude Code → Routines → New routine:
> "Read SKILL.md from the morning-brief skill and execute it"

Set schedule: weekdays at 07:00 in your timezone.

## Pre-flight checklist (first run)
- [ ] `config.yaml` filled in
- [ ] All MCP connections active in Claude (Slack, Gmail, Notion, Jira, Intercom, Fireflies, Google)
- [ ] Run once manually to verify delivery before enabling the routine
