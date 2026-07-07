---
name: ceremony-prep
description: "Prepares Nifemi for the three recurring agile ceremonies from live Jira + Notion: backlog refinement, sprint planning, and retro. ALWAYS trigger when Nifemi says: 'prep my backlog refinement', 'backlog prep', 'sprint planning prep', 'plan the sprint', 'retro prep', 'prep for retro', or 'get me ready for [ceremony]'. Pulls the PT board, groups by workstream, flags stale/ambiguous/unestimated items, and drafts a ready-to-run agenda for the chosen ceremony."
---

# Ceremony Prep Skill

## Purpose
One skill, three modes — backlog refinement, sprint planning, retro — each producing a **paste-ready agenda + working doc** from the live Jira PT board and Notion, so Nifemi walks in prepared. Note: the team's estimation practice is new (first sizing began ~mid-2026) — **do not flag "missing estimates" as a defect** unless explicitly doing sprint sizing.

## Trigger phrases
"backlog prep" / "prep my backlog refinement" · "sprint planning prep" / "plan the sprint" · "retro prep"

---

## Fixed Pesa resources (baked in)
- **Jira project `PT`**, board 101 (`pesapeer.atlassian.net`). Use `getVisibleJiraProjects` → cloudId, then `searchJiraIssuesUsingJql`.
- **Notion:** OKR & Projects tracker `collection://29daa57a-ba0b-80d2-8b1e-000b6095bf6a`, PRD DB `collection://21eaa57a-ba0b-8013-b503-000b84aa5aee`.
- **Workstreams** (group issues by these): group savings, personal savings, Nibss, onboarding, multicurrency wallets, Telna eSIM, cards, console, transfers/payouts, compliance.

---

## Mode 1 — Backlog refinement
```
JQL: project = PT AND status not in (Done, Released) ORDER BY created DESC
```
Produce:
- Open issues **grouped by workstream**, with a one-line summary each.
- 🚩 **Refinement flags:** vague/one-line descriptions, no acceptance criteria, no owner, duplicates, stale (>30d no update), oversized (should split), missing a linked PRD.
- A **ranked agenda**: which items to refine first (highest priority × least ready), with the specific question to resolve for each.
- Do **not** flag missing estimates as defects (first-time sizing).

## Mode 2 — Sprint planning
```
JQL (candidates): project = PT AND status = "To Do" AND (Sprint is EMPTY) ORDER BY priority DESC
JQL (carryover): project = PT AND sprint in openSprints() AND statusCategory != Done
```
Produce:
- **Carryover** from the current sprint (what didn't finish + why, if inferable).
- **Candidate list** ranked by priority and readiness (has AC, has owner, refined).
- **Capacity note** (ask for team size/velocity if unknown; don't invent).
- A proposed **sprint goal** tied to the top workstream, and the ordered pull-in list.
- Flags: anything unrefined being pulled in, cross-team dependencies, compliance items.

## Mode 3 — Retro
```
JQL: project = PT AND sprint in (closed sprints, most recent) ORDER BY updated DESC
```
Produce, from the just-closed sprint:
- **What shipped** vs **what was committed** (completed vs carried/dropped).
- Signals worth discussing: bugs/incidents in-sprint, items that stalled, scope added mid-sprint.
- A **retro agenda**: What went well / What didn't / Action items — pre-seeded with data-driven prompts (not blank).
- Pull the prior retro's action items (Notion) if available and check follow-through.

---

## Output
Paste-ready for Slack/Notion. Lead with the agenda; put the supporting detail below. Offer to (a) create it as a Notion page, (b) narrow to one workstream.

## Guardrails
- Read-only on Jira/Notion. Never transition issues or edit tickets.
- Don't invent velocity/capacity/estimates — ask or mark unknown.
- If the board is kanban (no sprints), fall back to recently-updated open issues and say so.

## Quality bar
- [ ] Correct mode selected; window/board stated
- [ ] Issues grouped by workstream
- [ ] Flags are specific and actionable (not "needs work")
- [ ] Agenda is ranked/ordered, ready to run
- [ ] No fabricated estimates/velocity
