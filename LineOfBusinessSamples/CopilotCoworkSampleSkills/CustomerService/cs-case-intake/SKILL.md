---
name: cs-case-intake
description: |
  Normalizes inbound customer service events — emails, chats, forms, and call transcripts —
  into structured case records in the service case tracker.
  Use when user asks to "new service case", "log case for [customer]",
  "intake from [channel]", "new ticket from [customer]",
  "customer email case", "create case for [issue]",
  "open a service ticket", or "log customer inquiry".
  Do NOT use for assembling context (use cs-context-packet),
  classifying the issue (use cs-issue-classifier),
  assessing severity (use cs-severity-assessment),
  routing the case (use cs-case-routing),
  or drafting a response (use cs-response-drafter).
---

## Overview

Converts an inbound customer service event — email, chat message, web form submission, or call transcript — into a normalized case record. Checks for duplicate cases on the same email thread, validates required fields, calculates SLA deadline, and writes the case to the shared case tracker in SharePoint.

This is the entry point for the customer service triage workflow. Every downstream skill depends on a case record created here.

## When to Use

- A new customer email, chat, or form submission has arrived
- A call transcript needs to be converted into a service case
- A customer issue needs to be formally logged in the tracker

## When NOT to Use

- Assembling customer and account context — use cs-context-packet
- Classifying the issue type and intent — use cs-issue-classifier
- Assessing severity and SLA path — use cs-severity-assessment
- Routing the case to an agent or queue — use cs-case-routing
- Drafting a customer response or handoff — use cs-response-drafter
- Making the final triage disposition — this is a human-only step

## Core Instructions

### Progress Tracking

At the start, create tasks:

```
TaskCreate(subject="Gather case details from inbound signal", activeForm="Gathering case details")
TaskCreate(subject="Check for duplicate or related cases", activeForm="Checking for duplicates")
TaskCreate(subject="Create case record in tracker", activeForm="Creating case record")
```

Mark each task `in_progress` when starting and `completed` when done.

### Step 1: Gather Case Details

Collect the following from the user's message, email, or inbound signal:

| Field | Required | Source |
|-------|----------|--------|
| Customer name | Yes | User input, email sender, or SearchPeople |
| Customer email or account ID | Yes | User input or email header |
| Inbound channel (email, chat, form, call) | Yes | User input or detected from source |
| Issue summary | Yes | User input or email body |
| Product or service area | No | User input or inferred from content |
| Attachments (screenshots, logs, invoices) | No | Email attachments or uploaded files |

**If the case originated from an email**, search for it:
- `SearchM365(sources=["email"], query="<customer name or issue keywords>")` to find the inbound email
- `GetMessage(message_id=...)` to read the full email thread

**Resolve customer identity if known internally:**
- `SearchPeople(query="<customer name>")` to check if customer is a known contact
- `GetUserDetails(user_id="<email>")` to get profile data

**Check CRM if Graph Connector is available:**
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` to pull account record, entitlement tier, and active incidents

**Calculate SLA deadline:**
- Standard cases: 15 minutes from creation for routing completion
- If the user provides urgency context, note it for the severity assessment step

If critical fields are missing (customer name, issue summary), ask the user once, covering all gaps.

### Step 2: Check for Duplicates

Before creating a new case, check the case tracker:

- `SearchM365(sources=["files"], query="service case tracker")` to locate the tracker
- `ReadFileContent(...)` to read the tracker and check for matching cases (same customer email thread, same issue description within the last 7 days)

If a potential duplicate is found, present it to the user: "I found an existing case for this customer — [Case ID, issue summary, status]. Should I create a new case, or update the existing one?"

### Step 3: Create Case Record

Generate a case record with these fields:

| Column | Value |
|--------|-------|
| Case ID | Auto-generated (format: CS-YYYY-NNNNN) |
| Customer Name | From user input or email |
| Account ID | From CRM or "Unknown" |
| Channel | Email / Chat / Form / Call |
| Issue Summary | From user input or email body |
| Category | Pending (set by cs-issue-classifier) |
| Priority | Pending (set by cs-severity-assessment) |
| Status | New |
| Created Date | Now |
| SLA Deadline | 15 minutes from creation |
| Assigned To | Pending (set by cs-case-routing) |
| Resolution | Open |
| Last Updated | Now |
| Updated By | Current user |

**Present the case record to the user for confirmation before writing.** Show it as an Adaptive Card.

After confirmation, write the record to the tracker spreadsheet.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the inbound customer email |
| SearchM365 (files) | Locate the case tracker |
| SearchM365 (connectors) | Pull CRM account data if Graph Connector available |
| GetMessage | Read full email content |
| SearchPeople / GetUserDetails | Resolve customer identity |
| GetMyDetails | Current user for Updated By field |
| ReadFileContent | Read existing tracker for duplicate check |

## Guardrails

- **Never create duplicate cases** for the same customer email thread without explicit user confirmation
- **Validate required fields** before writing — customer name, issue summary, and channel are mandatory
- **Confirm details with user** before writing to the tracker
- **Calculate SLA deadline automatically** — 15 minutes from creation for standard cases
- **Log every action** — record case creation with timestamp, creating user, and source channel
- **Never fabricate case details** — if information is missing, mark as "Unknown" or "Pending"
- **Preserve original customer language** in the issue summary — do not editorialize or interpret
