---
name: nifemi-morning-brief
description: "Generates Nifemi's comprehensive daily morning brief and todo list as Head of Product at Pesa. ALWAYS trigger this skill when Nifemi says any of the following: \"give me my morning brief\", \"morning brief\", \"daily brief\", \"brief me\", \"what do I have today\", \"catch me up\", \"what happened overnight\", \"start my day\", or any similar phrase requesting a daily summary or briefing. This skill orchestrates a full cross-platform intelligence sweep across Intercom, Gmail, Google Drive, Notion, Slack (every channel + every DM), Jira, Fireflies, and Google Calendar, then delivers a structured brief + checkbox todo list as a Slack DM to the person running it."
---

# Nifemi Morning Brief Skill

## Purpose
Deliver a comprehensive morning intelligence brief to Nifemi as Head of Product at Pesa.
The brief is DM'd directly to the user running it via Slack every time it runs.

## Trigger Phrases
"give me my morning brief" / "morning brief" / "daily brief" / "brief me" / "catch me up" / "what do I have today"

---

## ⚠️ Pre-Deployment Checklist
Before this skill is used for the first time, complete these one-time steps:

- [ ] Copy `config.example.yaml` → `config.yaml` and fill in your personal values
- [ ] **At runtime:** Read `config.yaml` from the skill directory first. All values marked `[FROM CONFIG]` below are sourced from it.
- [ ] **Fireflies:** Confirm Fireflies MCP is connected and tool names match (`Fireflies:search_transcripts`, `Fireflies:get_transcript`). If not, Step 8 will gracefully degrade.
- [ ] Run once manually before enabling the autonomous routine.

---

## Execution Order

Run all 9 data pulls, then synthesize into one message and post to Slack.
Be thorough — do not skip any source. Nifemi explicitly wants everything.

⚠️ **Maintainability Note:** The active workstreams, team members, and partner names referenced throughout this skill reflect Pesa's state as of June 2026. Update them when workstreams close or new ones open. Review quarterly.

⚠️ **Silent Failure Rule:** If ANY step fails due to a tool error, auth issue, or connection problem — do NOT skip it silently. In the relevant section of the brief, write: "[Source] unavailable — data may be incomplete. Check manually." Then continue with the remaining steps. Never post a brief that appears complete but is missing data without flagging it.

---

## Step 1 — Intercom: Full Support Intelligence (Last 24 Hours)

**Goal:** Complete picture of support activity — volume, patterns, recurring phrases, provider failures, login issues, unresponded conversations, and anything that needs Nifemi's attention. Read the actual message content, not just metadata.

Run all queries in parallel.

### 1a. All conversations updated in the last 24hrs (not just created)
```
Intercom:search_conversations
  - updated_at: >= (NOW - 24hrs unix timestamp)
  - per_page: 150
```
Paginate until exhausted. This is the full dataset — all other queries below are filters on top of this.

### 1b. Priority conversations with no admin reply
```
Intercom:search_conversations
  - updated_at: >= (NOW - 24hrs unix timestamp)
  - priority: "priority"
  - open: true
  - per_page: 150
```
Cross-reference with `statistics_first_admin_reply_at` — flag any priority conversation where there has been NO admin reply at all, or where `statistics_time_to_admin_reply > 7200` (2 hours).

### 1c. Conversations waiting the longest without a reply
```
Intercom:search_conversations
  - open: true
  - statistics_last_admin_reply_at: <= (NOW - 24hrs unix timestamp)
  - per_page: 50
```
These are open conversations where the last admin reply was over 24hrs ago — users still waiting.

### 1d. Conversations with multiple reopens (unresolved, recurring users)
```
Intercom:search_conversations
  - updated_at: >= (NOW - 24hrs unix timestamp)
  - statistics_count_reopens: >= 2
  - per_page: 50
```

### 1e. Keyword sweeps — read actual message content
Run these in parallel using `Intercom:search` with `source_body:contains`:
```
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"transfer" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"failed" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"login" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"can't log in" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"password" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"PIN" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"KYC" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"verification" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"fraud" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"refund" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"stuck" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"pending" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"not working" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"error" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"blocked" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"debit" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"card" state:open
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"limit" state:open
```

For any keyword returning >3 results, fetch a sample of the actual conversations via `Intercom:get_conversation` to read the full message content and understand the exact complaint.

### 1f. Funds / fraud / security — fetch full conversation content
```
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"stolen"
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"hacked"
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"unauthorized"
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"money missing"
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"didn't receive"
object_type:conversations updated_at:gte:YESTERDAY source_body:contains:"wrong account"
```
For ANY result here, immediately call `Intercom:get_conversation` on each one and read the full content. These are always critical.

---

**Synthesis — what to do with all of this:**

After running all queries, analyse the full dataset and produce:

**1. Volume summary**
- Total conversations updated in last 24hrs
- Open vs closed vs snoozed breakdown
- New conversations created vs re-opened vs ongoing

**2. Recurring phrases & themes (the most important section)**
Group conversations by the dominant complaint. Look across ALL message bodies from 1a and 1e and identify:
- What are the top 5 phrases or complaints that appear more than once?
- Is any single issue appearing in 3+ conversations? That's a pattern — name it explicitly.
- Example output: "Transfer stuck/pending — 12 conversations", "Login/access issues — 8 conversations", "KYC verification failing — 5 conversations"
- 🚨 Any theme with >5 conversations = flag as potential incident

**3. Provider / system failure signals**
Look for patterns that suggest a provider is down or degraded:
- Multiple users reporting the same failure at similar timestamps → likely a provider issue
- Keywords: "failed", "error", "not working", "pending", "stuck" in high volume
- 🚨 If 3+ users report the same failure type within a 2hr window → flag as probable provider incident. Name the suspected provider if identifiable (Payaza, Tribe, Bridge, Fincra, Keyless, Salt Edge)

**4. Login / access issues**
- Count of conversations mentioning login, password, can't access, locked out, biometric
- 🚨 If >3 login issues in 24hrs → flag (could indicate Keyless SDK issue, auth service degradation, or account lockout bug)

**5. Fraud & security**
- List every conversation from 1f with a one-line summary
- These are always 🔴 MUST DO in the todo list regardless of volume

**6. Unanswered & waiting**
- Conversations open with no admin reply >2hrs: count + list
- Conversations where the user has replied but admin hasn't responded since: list

**7. Conversations with multiple reopens**
- List any user who has reopened the same conversation 2+ times — signals an unresolved underlying issue

**Add to todo list:**
- 🔴 Any fraud/security conversation (read + escalate)
- 🔴 Any probable provider incident (investigate + notify eng)
- 🔴 Priority conversations with no reply >2hrs
- 🟡 Any theme with >5 conversations (investigate root cause)
- 🟡 Login issue spike (investigate with Keyless/auth team)
- 🟢 Recurring low-volume themes to monitor

---

## Step 2 — Gmail: Previous Day's Emails

**Goal:** Identify action items, flag unanswered emails older than 24hrs.

```
Gmail:search_threads
  - query: "after:YESTERDAY is:inbox -category:promotions -category:social"
  - pageSize: 50
```

Also run a second search for older unreplied threads:
```
Gmail:search_threads
  - query: "is:unread older_than:1d -from:noreply -from:no-reply -from:notification"
  - pageSize: 20
```

If either Gmail query returns an error or auth failure, note in the brief: "Gmail unavailable — emails not checked. Review manually." Do not halt the brief.

**Output format:**
- Emails requiring a response (group by sender/thread)
- ⚠️ OVERDUE: Emails >24hrs with no reply (flag these loudly)
- Documents/folders shared with Nifemi (e.g., Adeola sharing MFB docs)
- Calendar invites to accept/decline
- Any deadlines mentioned in email body

**Add to todo list:** Every email that requires a response or action.
**Flag overdue emails persistently** — if they appear again in tomorrow's brief, escalate urgency.

---

## Step 3 — Google Drive: Meeting Notes

**Goal:** Refresh Nifemi's memory on yesterday's meetings and extract todos.

```
Google Drive:search_files
  - query: "fullText contains 'meeting OR standup OR notes OR recap' and mimeType='application/vnd.google-apps.document' and modifiedTime > 'YESTERDAY_ISO'"
  - orderBy: modifiedTime desc
```

For each meeting note doc found, fetch and read it:
```
Google Drive:read_file_content
  - file_id: <id from search results>
```

If `Google Drive:search_files` returns an error or zero results, note in the brief: "Google Drive unavailable or no meeting notes found — check manually." Do not halt the brief.

**Output format:**
- For each meeting: title, attendees, key decisions made, action items
- 🎯 Identify actions assigned to Nifemi specifically
- Flag any decisions that affect current in-flight work (Savings, Keyless, etc.)

**Add to todo list:** All action items assigned to Nifemi from meeting notes.

---

## Step 4 — Notion: Product Wiki Full Sweep

**Goal:** Every page and database entry touched in the last 24hrs across the entire Product Wiki — all movement, whether or not it needs attention.

⚠️ **Plan-tier note:** `notion-query-database-view` requires a Notion Business plan, which this workspace does NOT have. It will fail. Use `notion-search` with `data_source_url` + filters instead — this works on the current plan. Do not use view queries.

⚠️ Limitation: `notion-search` filters on `created_date_range` (creation), not `last_edited_time`. To catch status changes on existing entries, run an unfiltered search per database and check each result's `last_edited_time` client-side. This is best-effort — note it in the brief.

### 4a. PRD Database
```
# Newly created PRDs
Notion:notion-search
  - query: ""
  - data_source_url: "collection://21eaa57a-ba0b-8013-b503-000b84aa5aee"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25

# Recently edited PRDs (status changes) — unfiltered, sorted by last edited
Notion:notion-search
  - query: ""
  - data_source_url: "collection://21eaa57a-ba0b-8013-b503-000b84aa5aee"
  - sort: { direction: descending, timestamp: last_edited_time }
  - page_size: 25
```
From the second query, keep only entries with `last_edited_time >= yesterday`. Flag anything that moved to In Progress, In Review, Approved, or Blocked.

### 4b. Product Initiatives Database
```
Notion:notion-search
  - query: ""
  - data_source_url: "collection://2d9aa57a-ba0b-8015-ab0c-000b73af4171"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25

Notion:notion-search
  - query: ""
  - data_source_url: "collection://2d9aa57a-ba0b-8015-ab0c-000b73af4171"
  - sort: { direction: descending, timestamp: last_edited_time }
  - page_size: 25
```
Keep entries edited since yesterday.

### 4c. OKRs & Projects Tracker
```
Notion:notion-search
  - query: ""
  - data_source_url: "collection://29daa57a-ba0b-80d2-8b1e-000b6095bf6a"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25

Notion:notion-search
  - query: ""
  - data_source_url: "collection://29daa57a-ba0b-80d2-8b1e-000b6095bf6a"
  - sort: { direction: descending, timestamp: last_edited_time }
  - page_size: 25
```
Keep entries edited since yesterday. 🚨 Flag any item that is Blocked, At Risk, or On Hold.

### 4d. Product Requests Database
```
Notion:notion-search
  - query: ""
  - data_source_url: "collection://2c5aa57a-ba0b-805a-8dcd-000b0d450eff"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25
```

### 4e. Product Wiki sub-pages (docs, notes, etc.)

Run two searches to maximise recall:
```
Notion:notion-search
  - query: "update"
  - page_url: "10eaa57a-ba0b-803d-9f98-dba7f14ebcd2"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25

Notion:notion-search
  - query: "review"
  - page_url: "10eaa57a-ba0b-803d-9f98-dba7f14ebcd2"
  - filters: { created_date_range: { start_date: YESTERDAY } }
  - page_size: 25
```
Deduplicate results across both searches. This catches standalone pages edited under the wiki (meeting notes, incident logs, weekly updates, etc.) that aren't in a structured database.

⚠️ Note: this search relies on keyword matching and will miss edited pages that don't contain the search terms. It is best-effort — for complete wiki coverage, manually check the Product Wiki root page periodically.

---

**Output format — report ALL movement, not just flagged items:**

*PRDs*
- New PRDs created: [title, owner, status]
- PRDs with status changes: [title, old status → new status, owner]

*Product Initiatives*
- New initiatives: [title, category, assigned to, status]
- Status changes: [title, old → new, owner]
- 🚨 Any initiative marked Blocked or moved to Backlog unexpectedly

*OKRs & Projects*
- New projects/tasks: [title, priority, due date, assignee]
- Status changes: [title, old → new]
- 🚨 Blocked, At Risk, or On Hold items — flag these loudly
- Progress % changes on active projects

*Product Requests*
- New requests submitted: [title, submitted by]

*Wiki docs & pages*
- Pages created or edited: [title, editor, brief summary if fetchable]

**Add to todo list:** PRDs needing review, Blocked/At-Risk items needing unblocking, new requests needing triage, initiatives assigned to Nifemi with no recent update.

---

## Step 5 — Slack: Full Channel + DM Sweep

**Goal:** Read every single channel and DM Nifemi is in. No keyword filtering. No hardcoded channel list. Everything.

### 5a. Discover and read every channel

Step 1 — Get all channels Nifemi is a member of:
```
Slack:slack_search_channels
  - query: "" (empty — returns all)
  - limit: 200
```
If results are paginated, fetch all pages until exhausted.

If the empty query returns 0 results, retry with:
```
Slack:slack_search_channels
  - query: "a"
  - limit: 200
```
If still 0 results, note "Slack channel list unavailable — sweep incomplete" in the brief and proceed with Step 5c mentions search only.

Step 2 — For each channel returned, read the last 24hrs:
```
Slack:slack_read_channel
  - channel_id: [each channel_id from results]
  - limit: 50
  - oldest: YESTERDAY_UNIX_TIMESTAMP
```

Do this for **every channel** — internal, external, partner, announcement, random. Do not skip any.

### 5b. Discover and read every DM

Step 1 — Find all DM conversations Nifemi is in by searching for direct messages:
```
Slack:slack_search_public_and_private
  - query: "is:dm after:YESTERDAY_DATE"
  - limit: 50
```

Also search for group DMs (MPIMs):
```
Slack:slack_search_public_and_private
  - query: "is:mpim after:YESTERDAY_DATE"
  - limit: 50
```

Step 2 — For each unique DM/MPIM channel_id returned in results, read the full thread:
```
Slack:slack_read_channel
  - channel_id: [each DM channel_id]
  - limit: 20
  - oldest: YESTERDAY_UNIX_TIMESTAMP
```

If DM enumeration returns 0 results, note: "DM sweep returned no results — Slack search may not support is:dm filtering. Check DMs manually."

Flag any DM with an unanswered message older than 12 hours.

### 5c. Also check Nifemi's direct mentions across all of Slack
```
Slack:slack_search_public_and_private
  - query: "<@[FROM CONFIG: slack_user_id]> after:YESTERDAY_DATE"
  - limit: 50
```
This catches any mention of Nifemi in channels that may have been missed.

**Output format:**
- Decisions made without Nifemi that need his awareness
- Escalations from ops/compliance team
- External partner requests or updates (flag partner channel messages specifically)
- 🚨 Unanswered DMs or mentions older than 12 hours
- Engineering blockers on active features (Ouribank, USD Wallet, Country of Origin)
- Fraud/security incidents in any channel

**Add to todo list:** Unanswered DMs, pending decisions, partner follow-ups, anything requiring Nifemi's input.

---

## Step 6 — Google Calendar: Today's Meetings

**Goal:** Brief Nifemi on what's coming today with context for each meeting.

```
Google Calendar:list_events
  - calendarId: primary
  - timeMin: today at 00:00 WAT
  - timeMax: today at 23:59 WAT
  - singleEvents: true
  - orderBy: startTime
```

If the tool name differs in your connected MCP, check available Google Calendar tools and use the equivalent "list events" call. If Calendar returns an error or auth failure, note in the brief: "Calendar unavailable — check your calendar manually." Do not leave the TODAY'S MEETINGS section blank.

**Output format for each meeting:**
- Time (WAT), duration, title, attendees
- Context: What was discussed last time / what decisions are pending
- Prep needed: Documents to review, questions to answer, decisions to make

If no events are returned, write "• No meetings scheduled today" in the TODAY'S MEETINGS section. Do not leave the section blank.

Cross-reference meeting context with:
- Recent Notion notes for that recurring meeting
- Recent Slack threads with the attendees
- Pending todos relevant to that meeting's topic

---

## Step 7 — Jira: Active Sprint State + Blockers

**Goal:** Give Nifemi (Head of Product) a complete picture of where the current sprint stands — not just his own tickets. Sprint health, blockers, stalling work, and anything he's mentioned on or watching.

Step 1 — Verify cloud ID and project key before running any queries:
```
Atlassian:getAccessibleAtlassianResources
Atlassian:getVisibleJiraProjects
```
Use the returned `cloudId` in all subsequent calls. Confirm the Pesa project key (expected: `PT`). If 0 resources or projects return, note "Jira unavailable" in the brief and skip this step.

Step 2 — Run sprint and blocker queries:
```
# Full active sprint — all issues, all statuses
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND sprint in openSprints() ORDER BY status ASC, updated DESC"
  - maxResults: 100

# Blocked tickets anywhere in the project (not just sprint)
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND (status = 'Blocked' OR flagged = Impediment) ORDER BY updated DESC"
  - maxResults: 30

# Stalling — sprint tickets not updated in 48hrs+ and not done
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND sprint in openSprints() AND statusCategory != Done AND updated <= -2d ORDER BY updated ASC"
  - maxResults: 30

# Bugs / incidents in the sprint
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND sprint in openSprints() AND issuetype in (Bug, Incident) ORDER BY priority DESC"
  - maxResults: 30

# Tickets Nifemi is mentioned on or watching (uses currentUser — auth identity, not config)
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND (watcher = currentUser() OR comment ~ currentUser()) AND statusCategory != Done ORDER BY updated DESC"
  - maxResults: 20
```

If `openSprints()` returns nothing (board may use kanban, not sprints), fall back to:
```
Atlassian:searchJiraIssuesUsingJql
  - jql: "project = PT AND statusCategory != Done AND updated >= -1d ORDER BY updated DESC"
  - maxResults: 50
```
and note "No active sprint found — showing recently updated open tickets instead."

**Output format:**

*Sprint health*
- Sprint name + end date if available
- Status breakdown: To Do: X | In Progress: X | In Review: X | Done: X | Blocked: X
- Rough completion: X of Y issues done

*🚨 Blockers* (always flag)
- [Ticket key] — [summary] — what's blocking, who owns it

*⏳ Stalling*
- Sprint tickets with no update in 48hrs+ — [key, summary, assignee, days stalled]

*🐞 Bugs / incidents in sprint*
- [key, summary, priority, assignee]

*👀 You're mentioned / watching*
- [key, summary, latest activity]

*Unowned work*
- Any sprint ticket with no assignee

**Add to todo list:** Blockers needing Nifemi's decision, stalling tickets on critical workstreams, bugs flagged high priority, tickets where he's been asked something in a comment.

⚠️ Note: this step uses `currentUser()` (resolved from the authenticated Atlassian session), NOT a hardcoded assignee ID. No personal Jira ID needed in config.

---

## Step 8 — Fireflies: Recent Meeting Transcripts

**Goal:** Extract decisions and action items from any meetings recorded in Fireflies in the last 24hrs. This supplements Google Drive notes which are often incomplete or missing.

```
Fireflies:search_transcripts
  - date_range: last 24 hours
  - limit: 10
```

For each transcript found, fetch and read it:
```
Fireflies:get_transcript
  - id: [transcript id]
```

⚠️ If Fireflies tools are unavailable, return an error, or return 0 results due to a connection issue (not genuinely no meetings), note in the brief: "🎙️ Fireflies tool unavailable." Then **fall back to email recaps**: Fireflies and Fathom send meeting recap emails. Check Gmail (from Step 2) for senders like `fireflies.ai`, `fathom.video`, or subjects containing "recap" / "meeting notes" / "summary" — extract decisions and action items from those instead. Do not halt the brief.

To add Fireflies as a proper tool: connect it as a connector in the Claude Code routine settings.

**Output format:**
- For each meeting: title, date, attendees, key decisions, action items
- 🎯 Flag any action item explicitly assigned to Nifemi
- Flag any decisions that contradict or advance current PRDs or epics
- Cross-reference with Google Drive step — if same meeting appears in both, merge, don't duplicate

**Add to todo list:** Action items assigned to Nifemi from transcripts.

---

## Step 9 — Previous Todo List Check

**Goal:** Surface items from previous briefs that haven't been completed.

Read previous briefs from your DM-with-yourself (where all briefs are delivered):
```
Slack:slack_read_channel
  - channel_id: [FROM CONFIG: slack_user_id]  ← your own user ID doubles as your DM channel
  - limit: 10 (last 10 messages)
```

If this returns an error, note: "Previous todos unavailable — carry-over check skipped." Do not halt the brief.

Scan for checkbox items (☐) that have not been checked off (✅).
These become **Carry-Over items** in today's brief — flagged as overdue if >1 day old.

---

## Output Format

Compile everything into one Slack message with this exact structure:

```
🌅 *NIFEMI'S MORNING BRIEF — [DAY, DATE]*
_Compiled from: Intercom · Gmail · Google Drive · Notion · Slack (all channels + DMs) · Jira · Fireflies · Calendar_

---

🚨 *CRITICAL — NEEDS YOU TODAY*
[Items that are urgent, flagged, or overdue — max 5, ordered by severity]

---

📊 *INTERCOM SUMMARY (Last 24hrs)*
• Total conversations updated: X | Open: X | Closed: X | Snoozed: X

🔁 *Recurring themes:*
• [Theme 1] — X conversations (e.g. "Transfer stuck/pending — 12")
• [Theme 2] — X conversations
• [Theme 3] — X conversations

⚡ *Provider signals:*
• [Provider name or "None detected"]

🔐 *Login/access issues:* X conversations [flag if >3]

🚨 *Fraud & security:* [list each, or "None"]

⏳ *Unanswered >2hrs:* X conversations — [list]

🔄 *Reopened 2+ times:* [list users or "None"]

---

📧 *EMAIL ACTION ITEMS*
• [Email action 1] — [sender] — [age]
• ⚠️ OVERDUE (Xhrs): [email description]

---

📅 *TODAY'S MEETINGS*
• [TIME] — [Meeting name] — [attendees]
  → Prep: [what you need to know/bring]

---

📋 *PRODUCT / NOTION UPDATES*
• [PRD status changes]
• [New initiatives]
• [Key decisions made without you]

---

🎫 *JIRA: ACTIVE SPRINT STATE*
• Sprint: [name] (ends [date]) — [X of Y done]
• Status: To Do X · In Progress X · In Review X · Blocked X · Done X
• 🚨 Blockers: [key — summary — owner], or "None"
• ⏳ Stalling (>48hrs): [key — summary — assignee]
• 🐞 Bugs in sprint: [key — summary — priority]
• 👀 You're mentioned/watching: [key — summary]
• Unowned: [any sprint ticket with no assignee]

---

🎙️ *MEETING TRANSCRIPTS (Fireflies)*
• [Meeting name] — [key decision or action item assigned to Nifemi]

---

💬 *SLACK INTELLIGENCE*
• [Key decisions, escalations, partner updates]
• [Engineering blockers]
• [Unanswered DMs]

---

♻️ *CARRY-OVER FROM PREVIOUS BRIEFS*
• [Items not yet actioned — marked ⚠️ if >1 day]

---

✅ *TODAY'S TODO LIST*

🔴 MUST DO TODAY
☐ [Task] — [source/context]
☐ [Task] — [source/context]

🟡 IMPORTANT
☐ [Task] — [source/context]
☐ [Task] — [source/context]

🟢 WATCH / MONITOR
☐ [Task] — [source/context]

---
_Brief generated: [timestamp WAT]_
```

---

## Delivery

**Read config.yaml first.** Load `slack_user_id` from config — this is both who the brief is for and where it's delivered.

Post the brief as a DM to the user running it:
```
Slack:slack_send_message
  - channel_id: [FROM CONFIG: slack_user_id]  ← sending to your own user ID = DM to yourself
  - message: [formatted brief]
```

**Do NOT send as a draft.** Send directly so it lands immediately.

**If send fails:** Stop. Inform the user that Slack delivery failed. Do not retry silently.

---

## Key Context for Intelligence Synthesis

### Pesa Product Context
- **Active workstreams:** Group Savings (go-live imminent), Keyless Biometric Auth, Trice USD integration, MFB application
- **Key team:** Wale (eng lead), Clement (backend), Harrison (Flutter), Sola/Dami (design), Tolu (ops/CEO), Oyekanmi (finance/compliance)
- **Critical monitoring:** PIN change incident (48hr review flow), Intercom fraud spike, Payaza recalls
- **External partners:** Tribe (cards), Bridge (USD/FedNow), Fincra (GHS/UGX), Payaza (NGN), Salt Edge, Keyless, Griffin

⚠️ **Review this section quarterly or when a major workstream closes/opens.** Nifemi owns this — update directly in the SKILL.md file. Last reviewed: June 2026.

### Todo Prioritisation Logic
- 🔴 MUST DO: Deadlines today, unblocking engineering, security/fraud incidents, unanswered external partners
- 🟡 IMPORTANT: Follow-ups >24hrs old, PRD reviews, people decisions, meetings prep
- 🟢 WATCH: Items to monitor, no direct action needed today

### Overdue Email Flagging
- 0–24hrs: Normal
- 24–48hrs: ⚠️ Flag in brief
- 48hrs+: 🔴 Escalate in CRITICAL section, repeat every brief until actioned

---

## Quality Bar

Before posting, verify:
- [ ] All 9 sources were checked (Intercom, Gmail, Drive, Notion, Slack, Jira, Fireflies, Calendar, Previous todos)
- [ ] Every Slack channel Nifemi is in was read — not just named ones
- [ ] Every DM with recent activity was checked
- [ ] Todo items have a source reference (where it came from)
- [ ] No duplicate tasks in the list
- [ ] Critical items are at the top
- [ ] Calendar items have prep context, not just meeting names
- [ ] Carry-over items from previous brief are included
- [ ] Jira blockers are flagged if any exist
- [ ] Fireflies and Drive meeting notes are merged — no duplicates
- [ ] Message is posted to channel (not drafted)