---
name: trd-creator
description: "Generates a Technical Requirements Document (TRD) for a Pesa feature by combining an approved/deployed PRD with the real code, following Pesa's TRD template, and writes it into the TRD database linked to the PRD. ALWAYS trigger when Nifemi says: 'write a TRD', 'create a TRD', 'draft a TRD for X', 'back-fill the TRD', 'generate a tech spec for X', 'TRD this PRD', or asks to document a shipped feature's technical spec. Works retrospectively (for features already in production) or forward (for an approved PRD not yet built). Grounds every section in the actual code and honestly flags as-built gaps/tech-debt."
---

# TRD Creator Skill

## Purpose
Turn an approved/deployed **PRD + the real code** into a **template-conformant TRD** in Pesa's TRD database, linked to the PRD. The value is not a filled-in template — it's a spec **grounded in what the code actually does**, that honestly surfaces as-built gaps (missing idempotency, reconciler blind spots, latency risks) as owned action items. Built to close the PRD→TRD governance gap (most production features have no TRD).

## Trigger phrases
"write/create/draft a TRD for X" · "back-fill the TRD" · "generate a tech spec for X" · "TRD this PRD"

---

## Fixed Pesa resources (baked in)

- **PRD database:** `collection://21eaa57a-ba0b-8013-b503-000b84aa5aee` — Status values include `Approved` (signed off, not in prod) and `Deployed` (in prod).
- **TRD database (target):** `collection://2f7aa57a-ba0b-8075-85fb-000b2ca08f82`
  - Title property: **`Spec Title`** · Status: one of `Draft | In Review | Approved | Implemented`
  - PRD link is the relation property named **`All  Product Requirement Documents `** *(exact name — two spaces after "All", trailing space)*.
- **TRD template page:** `34aaa57a-ba0b-8072-bb48-f685b0348e9b` — the 13-section structure to follow. Fetch it if you need the exact section list.
- **Code repos (read-only):** `/Users/nifemi/pesapeer-backend` (Go), `/Users/nifemi/pesapeer-mobile` (Dart), `/Users/nifemi/pesa-card-nimbus` (Go), `/Users/nifemi/providersdks` (Go).

⚠️ **Notion plan note:** bulk SQL (`notion-query-data-sources`) is gated on this workspace. Use `notion-fetch` for specific pages and `notion-search` (with `data_source_url`) to find them.

---

## Inputs
The user gives a **PRD** (Notion URL, or a feature name to search for) — optionally a workstream. If only a feature name is given, `notion-search` the PRD data source and confirm the right PRD before proceeding.

---

## Steps

### 1. Load the PRD
`notion-fetch` the PRD. Extract: problem, user stories + acceptance criteria, scope / out-of-scope, success metrics, dependencies, and its **Status** (Deployed → write a *retrospective* TRD; Approved-not-deployed → write a *forward* TRD and say so). Note the PRD URL(s) — you'll link them.

### 2. Map the real code
This is what makes the TRD non-generic. Using Bash `git grep`/`find` in the repos, locate the feature's implementation and read the key files:
- entrypoints/handlers, core service, data model (tables/migrations), external providers/integrations, background workers/crons, webhooks, metrics/operations names.
- Identify the **as-built architecture** and, critically, **real edge cases and gaps** (missing idempotency, no rollback, reconciler blind spots, latency budgets, external calls inside DB tx, etc.). Spawn parallel `Explore`/`general-purpose` agents for a big surface; keep it read-only.
- If the code can't be found, say so and write the TRD from the PRD + stated architecture, marking unverified sections.

### 3. Write the TRD (follow the template's 13 sections)
Metadata (link the PRD) · 1 Context (engineering framing, not a PRD repeat) · 2 Goals/Non-Goals · 3 Requirements (trace each to a PRD user story; make verifiable) · 4 Proposed Solution (as-built architecture + key design decisions *with rationale and rejected alternatives*; include a mermaid diagram) · 5 Technical Design (real data model, API/integration surface, component breakdown, third-party deps, infra/config) · 6 Security & Compliance (auth, data classification, encryption, audit, fraud, regulatory — FCA/CBN/FINTRAC/PCI/PSD2 as relevant) · 7 **Edge Cases & Error Handling — including known gaps** (the differentiator: mark ✅ handled / ⚠️ gap / ❓ verify with the real behaviour) · 8 Performance & Scalability · 9 Testing Strategy (name the real test stack; call out missing coverage) · 10 Rollout (retrospective = already live; rollback path) · 11 Monitoring (use the **real metric names** from code) · 12 Open Questions (turn the §7 gaps into owned, prioritized actions) · 13 Appendix (PRD links, code paths, Tribe/provider docs).

**Quality rules:**
- Every §4/§5 claim traces to real code. Don't invent endpoints or tables — grep for them.
- §7 and §12 are where the value is: surface the actual technical debt, honestly. A hollow "we handle errors gracefully" TRD is a failure.
- Add a callout at the top noting it's a **retrospective** TRD (system already in prod) when applicable.
- Keep the template's **Approval checklist**, including "Nifemi (Product) has reviewed alignment with PRD".

### 4. Create it in the TRD database
`notion-create-pages` with `parent: {data_source_id: "2f7aa57a-ba0b-8075-85fb-000b2ca08f82"}`, properties `{"Spec Title": "<Feature> — Technical Requirements Document (Retrospective)", "Status": "Draft"}`, and the full content. Content uses Notion-flavored Markdown (tables as `<table>`, callouts, ```mermaid, code blocks). Do **not** put the title in the body.

### 5. Link the PRD relation
After creation, `notion-update-page` (command `update_properties`) set `"All  Product Requirement Documents "` to a JSON-array string of the PRD URL(s), e.g. `"[\"https://app.notion.com/p/<prd-id>\"]"`. This makes the link show in both databases.

### 6. Report
Give the user the new TRD URL, a one-paragraph summary, and — most useful — the **list of ⚠️ gaps** it surfaced, offering to turn the top ones into PT tickets.

---

## Guardrails
- **Read-only on code.** Never push/commit/modify repos.
- Creates exactly **one** Notion TRD page (+ the relation update). Don't modify the PRD or template.
- Be honest about coverage: if a section couldn't be grounded in code, mark it unverified rather than inventing detail.

## Quality bar
- [ ] PRD fetched; retrospective vs forward determined and stated
- [ ] Architecture, data model, API, components all traced to real code (grep'd, not guessed)
- [ ] §7 edge-cases and §12 open-questions capture the real as-built gaps
- [ ] Monitoring section uses actual metric/operation names
- [ ] TRD created in the TRD DB and PRD relation linked
- [ ] Gaps summarized to the user with an offer to file PT tickets
