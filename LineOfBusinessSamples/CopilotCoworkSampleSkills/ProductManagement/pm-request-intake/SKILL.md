---
name: pm-request-intake
description: |
  Normalizes inbound feature requests into structured opportunity cases
  in the product opportunity tracker.
  Use when user asks to "new feature request from [source]",
  "log opportunity for [feature]",
  "intake this request from [customer]",
  "customer asked for [feature]",
  "create opportunity case for [request]",
  or "set up opportunity for [feature request]".
  Do NOT use for gathering product and customer context (use pm-context-packet),
  detecting duplicates or clustering demand (use pm-demand-cluster),
  drafting the opportunity brief (use pm-opportunity-brief),
  preparing the stakeholder review packet (use pm-review-packet),
  or routing to reviewers (use pm-reviewer-router).
---

## Overview

Normalizes inbound feature requests from email, Teams, customer call notes, intake forms, and internal stakeholder submissions into a structured opportunity case record in the shared Excel opportunity tracker. Validates required fields, flags potential duplicates against existing tracker entries, generates a unique Opportunity ID, and presents the normalized case for confirmation via Adaptive Card before writing.

This skill operates in "deterministic automation" mode — it applies stable field extraction and validation rules without exercising judgment on opportunity merit, priority, or roadmap impact.

## When to Use

- A new feature request has arrived via email, Teams, customer call, or intake form and needs to be captured as a structured opportunity case
- A product manager or product operations analyst wants to log a new demand signal into the opportunity tracker
- A batch of customer feedback needs to be broken into individual opportunity cases
- A sales escalation or support case includes a feature request that should enter the product intake pipeline

## When NOT to Use

- Gathering product, customer, and telemetry context — use pm-context-packet
- Detecting duplicates or clustering related demand signals — use pm-demand-cluster
- Drafting the opportunity statement and problem framing — use pm-opportunity-brief
- Preparing the stakeholder review packet — use pm-review-packet
- Routing the review packet to reviewers — use pm-reviewer-router
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Extract and normalize request fields", activeForm="Processing feature request")
TaskCreate(subject="Write opportunity case to tracker", activeForm="Creating opportunity case")
```

### Step 1: Identify the Request Source

Determine where the request originated:

| Source Channel | How to Find It |
|---------------|---------------|
| **Email** | `SearchM365(sources=["email"])` — find the original request thread by sender, subject, or keywords |
| **Teams** | `SearchM365(sources=["teams"])` — find the request message in product intake or escalation channels |
| **Customer call notes** | `SearchM365(sources=["files"])` — find call notes or meeting summaries in SharePoint |
| **Intake form** | `SearchM365(sources=["files"])` — find the submitted intake form in SharePoint |
| **Direct submission** | User provides request details directly in the conversation |

### Step 2: Extract and Normalize Fields

Extract the following fields from the request source:

| Field | Required | Source |
|-------|----------|--------|
| **Request description** | Yes | Original request text — preserve the requester's exact words |
| **Source channel** | Yes | Where the request originated (sales escalation, support case, customer advisory board, internal stakeholder, analytics signal) |
| **Requester** | Yes | Person who submitted or surfaced the request |
| **Customer/Account** | If external | Customer name and account identifier |
| **Product area** | If identifiable | Which product area the request relates to |
| **Urgency indicators** | If present | Any stated urgency, deadlines, or escalation context |

**Resolve requester identity:**
- `SearchPeople` — resolve the requesting party by name or email
- `GetUserDetails` — pull requester profile and organizational context

### Step 3: Generate Opportunity ID

Generate a unique Opportunity ID using the format: `PM-[YYYYMMDD]-[SOURCE]-[SEQ]`

- Date: current date
- Source prefix: SALES, SUPPORT, CAB (customer advisory board), INTERNAL, ANALYTICS, INTAKE
- Sequence: three-digit sequential number (001, 002, etc.)

### Step 4: Check for Potential Duplicates

Before creating the case, check for existing entries:

- `SearchM365(sources=["files"], query="opportunity tracker")` then `ReadFileContent` — scan the tracker for entries with similar product area, keywords, or customer
- Flag any entries where the product area AND key feature terms overlap with the new request

For each potential duplicate found, record:
- Existing Opportunity ID
- Summary of the existing entry
- Similarity basis (same feature, same customer, overlapping keywords)

### Step 5: Prepare the Opportunity Case Record

Assemble the normalized case record with these fields:

| Tracker Column | Value |
|---------------|-------|
| Opportunity ID | Generated ID |
| Request Source | Source channel |
| Requester | Resolved name and email |
| Customer/Account | Customer name (or "Internal" for internal requests) |
| Product Area | Identified product area (or "Unassigned" if not identifiable) |
| Summary | Normalized one-paragraph summary of the request |
| Status | "Intake — Not Committed" |
| Created Date | Current date |
| Assigned PM | Pending (to be assigned by product operations) |
| Theme | Pending (to be assigned after demand clustering) |
| Duplicate Flag | "Possible duplicate — see [ID]" or "No duplicates found" |
| Evidence Links | Link to the original request source |

### Step 6: Present for Confirmation

Present the normalized case via Adaptive Card (invoke `render-ui` skill first):

- **Opportunity ID** — generated ID
- **Source** — channel and requester
- **Customer/Account** — if applicable
- **Product Area** — identified area
- **Summary** — normalized description
- **Original request** — preserved verbatim
- **Duplicate check** — results of the duplicate scan
- **Status** — "Intake — Not Committed"
- **Confirmation prompt** — "Confirm to write this opportunity case to the tracker"

### Step 7: Write to Tracker (After Confirmation)

After the user confirms:
- Write the case record as a new row in the Excel opportunity tracker
- Create a per-opportunity folder in the SharePoint product opportunities library (for future evidence documents)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the original request thread |
| SearchM365 (teams) | Find request messages in product channels |
| SearchM365 (files) | Find intake forms, call notes, opportunity tracker |
| ReadFileContent | Read the opportunity tracker and intake form template |
| SearchPeople | Resolve requester identity |
| GetUserDetails | Pull requester profile and org context |

## Guardrails

- **Flag potential duplicates but never auto-merge** — duplicate detection is a recommendation; the PM decides whether to merge, link, or keep separate
- **Validate that required fields are present** before writing — requester, request description, and source channel must all be captured
- **Preserve the requester's exact words** in the Evidence Links or an attached note — normalization summarizes but does not replace the original request text
- **Never interpret a feature request as a commitment or promise** — every intake record explicitly sets Status to "Intake — Not Committed"
- **Never assign priority, impact, or roadmap position** at intake — these are determined in later steps by the product manager
- **Never auto-assign a product manager** — PM assignment is a product operations decision
- **Present the normalized case for confirmation** before writing to the tracker — the user reviews field accuracy before the record is created
- **Auto-generate the Opportunity ID** using the defined format — do not reuse or recycle IDs from closed cases
- **If the request is ambiguous or contains multiple distinct feature asks**, flag for the user to decide whether to create one case or multiple cases — do not silently split or merge
- **Never include customer financial data** (contract value, ARR, renewal dates) in the intake record — customer context is gathered in pm-context-packet
