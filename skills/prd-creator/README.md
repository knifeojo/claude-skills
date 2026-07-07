# 📋 prd-creator

Drafts a Pesa **Product Requirements Document** from a feature idea, following Pesa's PRD template, and creates it in the PRD database.

Auto-surfaces the parts PMs skip: Given/When/Then acceptance criteria, failure scenarios, edge cases, the full notification list, compliance needs, and 30/60/90 success metrics with a guardrail.

## Usage
- "create a PRD for scheduled transfers"
- "turn this into a PRD: [idea]"
- "product request" (Nifemi's shorthand → concise Goal/Context/Requirements/Scope format)

## Setup
```bash
cd skills/prd-creator
cp config.example.yaml config.yaml
```
Needs the Notion connector (PRD database) and read access to the code repos for grounding.

## What it does
1. Understands the request (asks 2–3 sharp questions if thin), picks the PRD type.
2. Grounds it in existing PRDs + real code.
3. Drafts the full PRD (problem/evidence, solution/out-of-scope, user stories with AC, requirements-by-team, failure & edge cases, 30/60/90 metrics, staged rollout, open decisions).
4. Creates it in the PRD database (Status: Draft) and surfaces the open decisions that need Nifemi's call.

Read-only on code. Pairs with **trd-creator** (PRD → TRD) and **prd-reviewer**.
