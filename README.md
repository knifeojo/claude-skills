# Pesa Claude Skills

Claude skills for the Pesa product team — built to run autonomously via Claude Code routines.

Clone, configure, and run. Each skill is self-contained with its own instructions and config.

---

## Skills

| Skill | What it does | Autonomous? |
|---|---|---|
| [🌅 morning-brief](skills/nifemi-morning-brief/) | Daily intelligence sweep across all tools — delivered as a Slack DM | ✅ Runs on schedule |
| [🚀 ship-report](skills/ship-report/) | What shipped to production across all repos for a period — management report + optional eng digest | ✅ Weekly-ready |

> More skills coming — prd-creation, prd-review, skill-reviewer are in progress.

---

## Setup

### 1. Clone the repo
```bash
git clone https://github.com/pesapeer/claude-skills.git
```

### 2. Configure personal skills
Skills that need personal config have a `config.example.yaml`:
```bash
cd skills/<skill-name>
cp config.example.yaml config.yaml
# Fill in your values
```

### 3. Install in Claude Code
Point Claude Code at the relevant `SKILL.md`. For autonomous runs, set up a Claude Code routine.

---

## Shared Pesa Config

Same for everyone on the team — baked into each skill:

| Resource | ID |
|---|---|
| Notion PRD Database | `21eaa57a-ba0b-8013-b503-000b84aa5aee` |
| Notion Initiatives Database | `2d9aa57a-ba0b-8015-ab0c-000b73af4171` |
| Notion OKR & Projects Tracker | `29daa57a-ba0b-80d2-8b1e-000b6095bf6a` |
| Notion Product Requests | `2c5aa57a-ba0b-805a-8dcd-000b0d450eff` |
| Notion Product Wiki | `10eaa57a-ba0b-803d-9f98-dba7f14ebcd2` |
| Jira Project | `PT` |
| Jira Board | [PT Board](https://pesapeer.atlassian.net/jira/software/projects/PT/boards/101) |

---

## Adding a New Skill

1. Create `skills/your-skill-name/`
2. Add `SKILL.md`, `README.md`, and `config.example.yaml` if needed
3. Update the skills table above
