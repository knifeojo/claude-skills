# 📐 trd-creator

Generates a **Technical Requirements Document** for a Pesa feature by combining an approved/deployed **PRD with the real code**, following Pesa's TRD template, and writes it into the TRD database linked to the PRD.

Built to close the PRD→TRD governance gap: most production features have no TRD. This drafts one that's *grounded in what the code actually does* and honestly surfaces as-built gaps (missing idempotency, reconciler blind spots, latency risks) as owned action items.

## Usage

In Claude Code:
- "write a TRD for the Send Money PRD"
- "back-fill the TRD for card issuance"
- "generate a tech spec for the referral program" (paste the PRD link or name)

Works **retrospectively** (feature already in prod) or **forward** (approved PRD not yet built).

## Setup

```bash
cd skills/trd-creator
cp config.example.yaml config.yaml   # adjust local repo paths if needed
```

Needs the Notion connector (PRD + TRD databases) and read access to the code repos.

## What it does

1. Fetches the PRD (user stories, scope, status) from Notion.
2. Maps the real implementation across the repos (read-only grep/agents).
3. Writes a template-conformant TRD — architecture, data model, API, security, and a **gaps-and-tech-debt** section grounded in the code.
4. Creates it in the TRD database and links the PRD relation.
5. Reports the new TRD + the list of ⚠️ gaps it found, ready to become PT tickets.

**Read-only on code** — never pushes or modifies anything.

## Related
Pairs with the PRD→production→TRD gap analysis (find deployed features missing a TRD, then back-fill them one by one).
