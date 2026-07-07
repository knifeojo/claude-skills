# 📊 data-analyst

Turns a plain-English data question into correct, **paste-ready SQL** against Pesa's schema, following Nifemi's SQL conventions — plus a short read of what it answers.

## Usage
- "how many users onboarded in NG last month?"
- "SQL for CAD corridor volume week-on-week"
- "pull successful transfer volume by currency"

## Setup
```bash
cd skills/data-analyst
cp config.example.yaml config.yaml
# optionally add a read-only DB connection
```

## What it does
1. Restates the question with explicit assumptions (window, status filter, currency).
2. Writes Postgres SQL using the authoritative schema, `bvn`, and the correct join path (`ps_money_transfers → v_ps_wallets → v_users`), with country/nationality handling.
3. Explains what it returns + caveats.
4. Outputs paste-ready SQL (or runs it if a read-only connection is configured).

**Read-only** — SELECTs only, never writes. Doesn't invent schema; flags uncertainty against the authoritative table doc.
