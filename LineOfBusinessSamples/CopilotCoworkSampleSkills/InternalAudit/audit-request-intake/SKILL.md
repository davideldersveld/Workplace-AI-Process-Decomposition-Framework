---
name: audit-request-intake
description: |
  Normalizes audit request events into structured case records in the audit case tracker.
  Use when user asks to "new audit request", "audit intake for [process area]",
  "log audit case for [control area]", "walkthrough scheduled for [control area]",
  "open evidence request for [engagement]", "new audit engagement",
  "create audit case", or "intake for [audit type]".
  Do NOT use for assembling scope and evidence context (use audit-evidence-packet),
  detecting evidence gaps (use audit-gap-detection),
  routing evidence requests (use audit-request-routing),
  or drafting audit communications (use audit-case-comms).
---

## Overview

Converts audit request signals — engagement scheduling emails, ad hoc requests, walkthrough notifications, or follow-up case openings — into normalized, validated case records in the shared Excel audit case tracker. Validates required fields, checks for duplicate or related engagements, resolves auditor identities, and confirms details before writing.

This skill operates in "deterministic automation" mode — structured field extraction, date validation, and duplicate checking with no AI judgment. The user confirms all details via Adaptive Card before the tracker is updated.

## When to Use

- A new audit engagement has been scheduled and needs a case record
- An ad hoc audit request has been received and needs to be logged
- A control walkthrough has been scheduled and needs tracking
- A follow-up engagement on prior findings needs to be opened
- An evidence request case needs to be created

## When NOT to Use

- Assembling scope, control, and prior-evidence context — use audit-evidence-packet
- Detecting missing evidence or scope gaps — use audit-gap-detection
- Routing evidence requests to control owners — use audit-request-routing
- Drafting audit summaries or evidence request communications — use audit-case-comms
- Confirming audit packet disposition — this is always a human decision (IA-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather audit request details", activeForm="Gathering audit request details")
TaskCreate(subject="Validate and create case record", activeForm="Creating case record")
```

### Step 1: Gather Audit Request Details

Collect or extract the following from the user's request, email, or system signal:

**Required fields:**
- **Engagement type** — scheduled audit, ad hoc request, walkthrough, follow-up
- **Process area or control area** under audit
- **Business unit** and **legal entity**
- **Audit period** — start date and end date
- **Lead auditor** assignment

**Optional fields:**
- Related prior audit findings (if known)
- Audit committee or management request reference
- Risk rating from the annual audit plan
- Engagement priority (standard, expedited)

**Resolve identities:**
- `SearchPeople` — resolve lead auditor, audit manager, and any referenced control owners
- `GetUserDetails` — confirm auditor profiles and reporting relationships

**Check for source email:**
- `SearchM365(sources=["email"], query="audit request [process area]")` — find the original audit request or scheduling email
- `GetMessage` — read full content if a source email is identified

### Step 2: Validate and Check Duplicates

**Read the audit case tracker:**
- `SearchM365(sources=["files"], query="audit case tracker")` then `ReadFileContent`

**Validate required fields:**
- All required fields (engagement type, process area, business unit, audit period, lead auditor) must be populated
- Audit period start date must be a valid date
- Audit period end date must be after start date

**Check for duplicates:**
- Search the tracker for existing cases with the same process area AND overlapping audit period
- If a potential duplicate is found, present it to the user with details and ask whether to proceed or link to the existing case
- Search for related prior engagements in the same process area for cross-reference

### Step 3: Confirm and Write Case Record

Present the normalized case record via Adaptive Card (invoke `render-ui` skill first) for user confirmation:

- **Case ID** — auto-generated (format: IA-YYYY-NNN based on year and sequence)
- **Engagement Type** — scheduled audit / ad hoc request / walkthrough / follow-up
- **Process Area** — the control area or process under audit
- **Business Unit** — organizational unit
- **Legal Entity** — legal entity scope
- **Audit Period** — start and end dates
- **Lead Auditor** — resolved name and email
- **Status** — "Intake Complete"
- **Created Date** — today's date
- **Evidence Status** — "Not Started"
- **Prior Findings Count** — number of related prior findings (0 if none)
- **Disposition** — "Pending"

After user confirmation, log the case creation with timestamp and creating user in the Audit Log column.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchPeople | Resolve auditor assignments and control owner identities |
| GetUserDetails | Pull auditor and control owner profiles |
| SearchM365 (email) | Find the original audit request or scheduling email |
| GetMessage | Read full request email content |
| SearchM365 (files) | Find audit case tracker in SharePoint |
| ReadFileContent | Read the tracker to check for duplicates and related engagements |

## Guardrails

- **Never create duplicate cases** for the same process area and overlapping audit period without explicit user confirmation
- **Validate all required fields** before writing — do not create incomplete case records
- **Confirm details via Adaptive Card** before writing to the tracker — present all fields for review
- **Log case creation** with timestamp, creating user, and source reference in the Audit Log column
- **Never assign audit engagements** — only record assignments made by the audit manager; this skill captures, it does not decide
- **Never interpret audit scope or risk** — record factual intake information only
- **Preserve audit independence** — do not include commentary, risk opinions, or scope recommendations in case records
- **Include source reference** — if the case originated from an email, meeting, or management request, record the reference for traceability
