# 🔍 prd-reviewer

Reviews a Pesa PRD three ways: **quality** (against the template), **discrepancy** (vs what the code actually does), and **governance** (does it have a linked TRD).

## Usage
- "review this PRD" (paste link) → quality review + verdict
- "does this PRD match the code?" → spec-vs-implementation discrepancy report
- "which PRDs are missing a TRD?" → PRD↔TRD gap audit

## Setup
```bash
cd skills/prd-reviewer
cp config.example.yaml config.yaml
```
Needs the Notion connector (PRD + TRD databases) and read access to the code repos.

## What it does
- **Mode A — Quality:** section-by-section scoring (✅/⚠️/❌), flags missing acceptance criteria, out-of-scope, edge cases, notifications, guardrail metrics.
- **Mode B — Discrepancy:** maps PRD claims to the real code and reports where they diverge, citing `file:line`.
- **Mode C — Governance:** checks the TRD link; for the whole set, lists Deployed-without-TRD and Approved-not-deployed PRDs.

Read-only everywhere. Pairs with **prd-creator** and **trd-creator** (offers to back-fill a missing TRD).
