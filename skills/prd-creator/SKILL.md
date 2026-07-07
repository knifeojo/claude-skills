---
name: prd-creator
description: "Drafts a Pesa Product Requirements Document from a feature idea or product request, following Pesa's PRD template, and creates it in the PRD database. ALWAYS trigger when Nifemi says: 'create a PRD', 'write a PRD for X', 'draft a PRD', 'new PRD', 'turn this into a PRD', 'product request' (his shorthand), or describes a feature he wants specced. Auto-surfaces user stories with Given/When/Then acceptance criteria, failure scenarios, edge cases, notifications, compliance needs, and 30/60/90 success metrics — and pulls context from the real code/related PRDs where useful."
---

# PRD Creator Skill

## Purpose
Turn a feature idea into a **complete, review-ready PRD** using Pesa's actual template — with the parts PMs skip (acceptance criteria, failure scenarios, edge cases, guardrail metrics, notification list, compliance) proactively surfaced. Nifemi is Head of Product; outputs must include **acceptance criteria and explicit scope boundaries**.

## Trigger phrases
"create/write/draft a PRD for X" · "new PRD" · "turn this into a PRD" · "product request"

---

## Fixed Pesa resources (baked in)
- **PRD database (target):** `collection://21eaa57a-ba0b-8013-b503-000b84aa5aee` — title property `Task name`, Status `Draft|In review|Approved|Deployed`.
- **PRD templates** (pick by type):
  - New feature → `328aa57aba0b80559e1bed55278be501`
  - New feature × integration → `328aa57aba0b807b8fedca47fb1b6255`
  - Pure integration → `328aa57aba0b80d1bd7cf84ac4adb142`
- **Personas** (use these names in user stories): **Tunde, Amara, Sofia, Chidi, Kwame**.
- **Code repos** (read-only, for grounding): `/Users/nifemi/pesapeer-backend`, `pesapeer-mobile`, `pesa-card-nimbus`, `providersdks`.

---

## Steps

### 1. Understand the request
Gather: the problem, who's affected, the desired outcome, and any constraints. If the ask is thin, ask **2–3 sharp clarifying questions** (target user, the corridor/currency/market, whether a third-party integration is involved) — then proceed. Decide the PRD **type** (new feature / feature×integration / integration).

### 2. Ground it (optional but preferred)
Quickly check for context: search the PRD database for related/overlapping PRDs (avoid duplicates), and `git grep` the code for existing patterns this would touch (existing endpoints, wallet/transfer/card logic, notification infra). Fold real specifics in — it makes the PRD credible.

### 3. Draft the PRD (Pesa New Feature template)
Produce these sections:
- **Problem** — what's broken/missing today, *who* is affected, how often, consequence. Add **Supporting evidence** (data/tickets/feedback — mark assumptions if none).
- **Solution** — one paragraph on what's built and why it solves it. **Out of scope** — explicit (anything not listed is assumed in scope by eng/design).
- **User stories** — for each: *As a [Tunde/Amara/Sofia/Chidi/Kwame], I want [action] so that [outcome].* Each with **acceptance criteria in Given / When / Then**.
- **Design** — placeholder link to the design request.
- **Requirements by team** — **Engineering** (technical requirements, state machines, audit trail), **Compliance & Risk** (regulatory, data handling, consent), **Notifications** (every template: trigger · channel push/email/in-app · content outline), **Customer Success** (edge-case handling/support).
- **Failure scenarios & edge cases** — name each failure condition + how the product handles it; list unusual-but-possible edge cases so eng/design don't miss them.
- **Success metrics** — 30 / 60 / 90 days: **Primary**, **Secondary**, **Guardrail** (what must not get worse).
- **Rollout plan** — Internal → 10% → 50% → 100%.
- **Open issues & key decisions** — table (Decision · Options considered · Decision taken · Owner).
- Stakeholder-validation note.

**Quality rules:** every feature needs acceptance criteria + explicit out-of-scope. Proactively surface edge cases, integrations, and a guardrail metric even if the user didn't ask. Be concise-first; expand on request. Tie to Pesa strategy (multi-currency wallet, corridors, zero-fee) where honest.

### 4. Deliver
Default: **create the PRD in the PRD database** (`notion-create-pages`, parent `{data_source_id: "21eaa57a-ba0b-8013-b503-000b84aa5aee"}`, `{"Task name": "<Feature> PRD", "Status": "Draft"}`), content in Notion-flavored Markdown (tables as `<table>`, callouts, buttons omitted). Then report the URL + a 2-line summary + the open decisions that need Nifemi's call.
- If the user says "just draft it" / "here", output paste-ready Markdown instead of creating the page.
- **`product request` shorthand:** if Nifemi says "product request", also honour his personal quick format — `## Goal / ## Context / ## Documents / ### Product Requirements / ### Scope / ### Out of scope` — as the concise version, then offer to expand to the full PRD.

---

## Guardrails
- Read-only on code. Creates one PRD page (or outputs markdown). Never modify other pages.
- Don't invent evidence/metrics — mark assumptions as assumptions.
- Don't duplicate an existing PRD — if a near-match exists, surface it and ask whether to extend it.

## Quality bar
- [ ] Problem names who/how-often/consequence, with evidence (or flagged assumptions)
- [ ] Explicit **out of scope**
- [ ] Every user story has **Given/When/Then** acceptance criteria, using the Pesa personas
- [ ] Requirements-by-team incl. **Notifications** list and **Compliance**
- [ ] Failure scenarios + edge cases named
- [ ] 30/60/90 metrics incl. a **guardrail**
- [ ] Created in PRD DB (or markdown) + open decisions surfaced to Nifemi
