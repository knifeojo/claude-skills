---
name: prd-reviewer
description: "Reviews a Pesa PRD three ways: (1) quality/completeness against the PRD template, (2) discrepancy vs what the code actually does, and (3) whether it has a linked TRD. ALWAYS trigger when Nifemi says: 'review this PRD', 'is this PRD ready', 'check this PRD', 'PRD gaps', 'does this PRD match the code', 'spec vs implementation', or 'which PRDs are missing a TRD'. Returns a scored review with concrete gaps and fixes, grounded in the real code — not a generic checklist."
---

# PRD Reviewer Skill

## Purpose
Give a PRD a rigorous, Pesa-specific review across three lenses so Nifemi (Head of Product) can approve, send back, or spot drift between spec and reality:
1. **Quality** — is it complete, unambiguous, and reviewable against the PRD template?
2. **Discrepancy** — does what's described match what the **code** actually does? (spec-vs-implementation)
3. **Governance** — does it have a **linked TRD**? Is it approved/deployed without one?

## Trigger phrases
"review this PRD" · "is this PRD ready" · "PRD gaps" · "does this PRD match the code" · "spec vs implementation" · "which PRDs are missing a TRD"

---

## Fixed Pesa resources (baked in)
- **PRD database:** `collection://21eaa57a-ba0b-8013-b503-000b84aa5aee` (Status incl. `Approved`, `Deployed`; TRD link = relation property `PRD`).
- **TRD database:** `collection://2f7aa57a-ba0b-8075-85fb-000b2ca08f82`.
- **Code repos (read-only):** `/Users/nifemi/pesapeer-backend`, `pesapeer-mobile`, `pesa-card-nimbus`, `providersdks`.
- **Template sections a good PRD has:** Problem+evidence · Solution+out-of-scope · User stories w/ Given/When/Then AC · Requirements by team (Eng/Compliance/Notifications/CS) · Failure scenarios & edge cases · 30/60/90 metrics w/ guardrail · Rollout · Open decisions.

⚠️ Notion bulk SQL is plan-gated — use `notion-fetch` + `notion-search(data_source_url)`.

---

## Modes (pick based on the ask; default = all three on a single PRD)

### Mode A — Quality review (single PRD)
`notion-fetch` the PRD. Score each template section present/absent/weak. Flag specifically:
- Missing or vague **acceptance criteria** (must be Given/When/Then, testable).
- No explicit **out of scope** (the #1 cause of eng scope-creep).
- Missing **failure scenarios / edge cases**, **notification list**, **compliance** section.
- No **guardrail metric**; unmeasurable success metrics.
- Ambiguous ownership / unresolved open decisions.
Output: a section-by-section table (✅ / ⚠️ weak / ❌ missing) + the top 3–5 fixes, ranked.

### Mode B — Discrepancy: PRD vs code (spec-vs-implementation)
For a Deployed/Approved PRD, map its claims to the real implementation via `git grep`/agents, then report where **the code diverges from the spec** — features described but not built, built differently, or extra behaviour not in the PRD. This is the AI-resistant, high-value lens (we've used it for savings and cards). Cite `file:line`.

### Mode C — Governance: PRD ↔ TRD gap
- For **one** PRD: does it have a linked TRD (relation `PRD` populated)? If Deployed with no TRD → flag; offer to run **`trd-creator`**.
- For the **whole set**: enumerate PRDs (search by workstream, read Status + TRD relation) and list those that are **Deployed with no TRD** and **Approved-not-deployed**. (This is the PRD→TRD gap audit.)

---

## Output
- **Verdict** first: Ready to approve / Send back / Needs code reconciliation.
- Scored section table (Mode A), discrepancy list with `file:line` (Mode B), TRD status (Mode C).
- **Top fixes ranked**, each actionable.
- Offer next actions: create the missing TRD (`trd-creator`), or file discrepancies as PT tickets.

## Guardrails
- Read-only everywhere — never modify the PRD, code, or TRD.
- Be specific, not generic: "AC missing on story 2" not "improve acceptance criteria". Ground discrepancy claims in real code.
- Don't rubber-stamp: if it's thin, say send-back and why.

## Quality bar
- [ ] Verdict stated up front
- [ ] Every template section assessed (Mode A)
- [ ] Discrepancies cite real `file:line` (Mode B)
- [ ] TRD-link status reported; back-fill offered if missing (Mode C)
- [ ] Top fixes ranked and actionable
