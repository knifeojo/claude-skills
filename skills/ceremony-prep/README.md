# 🗓️ ceremony-prep

Prepares Nifemi for the three recurring agile ceremonies from live Jira PT + Notion: **backlog refinement**, **sprint planning**, and **retro**.

## Usage
- "backlog prep" / "prep my backlog refinement"
- "sprint planning prep" / "plan the sprint"
- "retro prep"

## Setup
```bash
cd skills/ceremony-prep
cp config.example.yaml config.yaml
```
Needs the Jira (Atlassian) + Notion connectors.

## What it does
- **Backlog:** open issues grouped by workstream + refinement flags (vague, no AC, stale, oversized, no PRD) + a ranked agenda.
- **Sprint planning:** carryover + ranked, ready candidates + proposed sprint goal (asks for capacity if unknown).
- **Retro:** committed-vs-shipped, stalls/incidents, and a data-seeded retro agenda + prior action-item follow-through.

Read-only on Jira/Notion. Estimation is new for the team, so it never flags "missing estimates" as a defect.
