---
name: data-analyst
description: "Turns a plain-English data question into a correct, paste-ready SQL query against Pesa's schema, following Nifemi's SQL conventions, and explains the result. ALWAYS trigger when Nifemi says: 'write a query for X', 'how many users/transactions...', 'pull the numbers on X', 'SQL for X', 'analyze X', 'CAD corridor volume', or asks any question answerable from the database. Uses the authoritative schema and the correct join paths; outputs SQL ready for a SQL editor plus a short read of what it answers."
---

# Data Analyst Skill

## Purpose
Convert Nifemi's data questions into **correct, paste-ready SQL** using Pesa's real schema and conventions, so he can drop it straight into a SQL editor. Optionally run it if a DB connection is configured. Nifemi is Head of Product — lead with the answer/query, keep it concise.

## Trigger phrases
"write a query for X" · "how many users/transactions…" · "pull the numbers on X" · "SQL for X" · "CAD corridor volume" · "analyze X"

---

## SQL conventions (authoritative — follow exactly)
- Field is **`bvn`** (Bank Verification Number), never `ben`.
- Authoritative schema: **"PESA Database Tables.xlsx"** (sheet *All Tables details*, from row 2). If a column/table is uncertain, say so rather than guessing.
- Preferred join path: **`ps_money_transfers → v_ps_wallets → v_users`** via `pmt.wallet_id = w.id` and `w.user_id = u.id`.
- `v_users.country_of_residence` stores **UUID-like strings** → join to `sys_countries.id`.
- `v_users.nationality` stores **2-letter ISO codes** → `sys_countries.two_letter_code`.
- Primary phone: prefer `primary_phone_number = true`, then most recent `verified_at`.

### Key tables & columns
- **ps_money_transfers:** id, user_id, wallet_id, status, beneficiary_payout_currency_code, beneficiary_payout_amount, wallet_deduction_amount, transaction_fee, created_at, beneficiary_id, beneficiary_type, provider, system_transaction_reference
- **v_ps_wallets:** id, user_id, currency_code, available_balance, ledger_balance, status, default_wallet
- **v_users:** id, email, phone_number, firstname, lastname, kyc_verified, kyc_status, country_of_residence, bvn, status, created_at
- **user_phone_numbers:** id, dialing_code, phone_number, primary_phone_number, verified_at, user_id, status, created_at, risk_level, risk_score, line_type, carrier, country
- **sys_countries:** id, name, two_letter_code, three_letter_code, dialing_code, status, currency_id, registration_allowed, wallet_creation_allowed, virtual_account_supported, supported_beneficiary_types, payment_provider

---

## Steps
1. **Restate** the question precisely (assumptions explicit): time window, statuses to include (e.g. successful only?), currency handling, dedup.
2. **Write the SQL** — Postgres dialect, using the conventions and join paths above. Prefer CTEs for readability. Comment non-obvious logic. For money, be explicit about `wallet_deduction_amount` vs `beneficiary_payout_amount`.
3. **Explain** — 2–3 lines: what the query returns and any caveat (e.g. "successful transfers only", "excludes reversed").
4. **Deliver** paste-ready (fenced SQL block). If `db.connection` is configured, offer to run it and summarize; otherwise output the query for his SQL editor.

### Recurring analysis: CAD corridor volume (an active work area)
Week-on-week **CAD-equivalent volume**: sum outbound transfers by ISO week, converting to CAD, filtered to successful transfers. Provide the query grouped by `date_trunc('week', created_at)` with WoW delta. Confirm the CAD-equivalent conversion basis (rate table vs stored amount) before finalizing.

---

## Guardrails
- **Read-only analytics only** — SELECTs. Never write/UPDATE/DELETE. If a connection is configured, enforce read-only.
- Don't invent columns/tables — if unsure, flag against the authoritative schema and ask.
- State assumptions (window, status filter, currency) up front so the number is trustworthy.
- Be careful with money: name the amount column used and the currency basis.

## Quality bar
- [ ] Question restated with explicit assumptions
- [ ] Uses `bvn`, correct join path, country/nationality handling
- [ ] Paste-ready Postgres SQL, commented
- [ ] 2–3 line read of what it answers + caveats
- [ ] No fabricated schema; no write statements
