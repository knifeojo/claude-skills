# 🌙 eod-desk

Nifemi's **end-of-day wrap** — the evening counterpart to the morning brief. Reconciles today's todos, summarizes what moved, and sets up tomorrow. Delivered as a Slack DM.

## Usage
- "eod" / "wrap up my day" / "what did I get done"

## Setup
```bash
cd skills/eod-desk
cp config.example.yaml config.yaml   # reuse your morning-brief personal values
```

## What it does (today only)
1. Reconciles the rolling Todos Canvas — done vs still-open (with evidence).
2. Captures what moved today across Slack, Jira PT, Notion, and PRs.
3. Lists what's still open / waiting on you.
4. Sets up tomorrow: first meeting + prep, and the top 3 to start with.

Read-only except the Slack DM + Todos Canvas carry-forward. Pairs with **morning-brief** (shares the Todos Canvas) and **ship-report**.
