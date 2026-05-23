---
name: fc-journal-intake
description: |
  Normalizes inbound manual journal entry requests into structured case records
  in the journal tracker, validates debit-credit balance, and checks for duplicates.
  Use when user asks to "new journal entry request", "journal case for [description]",
  "set up journal review for", "close adjustment request",
  "manual JE for [description]", "log journal entry",
  "create journal case for [entity]", or "intake journal request".
  Do NOT use for assembling ledger and policy context (use fc-journal-context),
  detecting missing evidence or risks (use fc-gap-risk-detection),
  determining the approval path (use fc-approval-routing),
  or drafting summaries and follow-ups (use fc-journal-comms).
---

## Overview

Normalizes inbound manual journal entry requests — from email, chat, close management events, or direct intake — into structured case records in the shared Excel journal tracker. Validates that debit and credit amounts balance, checks for duplicate cases, rejects entries missing required fields, and logs creation metadata for audit traceability.

This skill operates in "deterministic automation" mode — structured field extraction, balance validation, duplicate checking, and case creation follow fixed rules with no AI judgment required.

## When to Use

- A manual journal entry request has been submitted via email, chat, or close management workflow
- A close-period adjustment needs to be logged and tracked
- The user wants to create a structured journal case from a request

## When NOT to Use

- Assembling ledger, policy, and support context — use fc-journal-context
- Detecting missing evidence or control risks — use fc-gap-risk-detection
- Determining the approval path — use fc-approval-routing
- Drafting journal summaries or follow-up requests — use fc-journal-comms
- Posting a journal entry to the ERP — this is always a human action outside of Cowork

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read inbound request and extract journal fields", activeForm="Reading journal request")
TaskCreate(subject="Validate, check for duplicates, and write case record", activeForm="Creating journal case")
```

### Step 1: Read Inbound Request

Identify the source of the journal entry request:

- **Email**: `SearchM365(sources=["email"])` with requestor name or journal description, then `GetMessage(message_id=...)` to read the full request
- **Chat**: `ListChatMessages(person=...)` or `SearchM365(sources=["teams"])` to find the request message
- **File attachment**: `SearchM365(sources=["files"])` to locate journal request forms or supporting schedules in SharePoint
- **Close management system**: `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` if Graph Connector is configured

Extract the following fields from the inbound request:

| Field | Description | Required |
|-------|-------------|----------|
| Description | Journal entry rationale or purpose | Yes |
| Requestor | Name or email of the requesting accountant | Yes |
| Entity | Legal entity or company code | Yes |
| Business Unit | Cost center, department, or business unit | Yes |
| Debit Account | Account code(s) for debit side | Yes |
| Credit Account | Account code(s) for credit side | Yes |
| Amount | Entry amount | Yes |
| Currency | Currency code (USD, EUR, etc.) | Yes |
| Close Period | Accounting period (e.g., "May 2026", "Q2 2026") | Yes |
| Supporting Documents | References to schedules, reconciliations, or memos | If available |

**Resolve identities:**
- `SearchPeople(query="<requestor name>")` to resolve the requesting accountant
- `GetUserDetails(user_id="<requestor>")` to pull the requestor's profile and reporting chain
- `GetMyDetails` to record who created the journal case

### Step 2: Validate and Create Record

**Validation checks:**
- **Required fields**: Reject if entity, debit account, credit account, amount, currency, or close period is missing. Present the missing fields and ask the user to provide them.
- **Debit-credit balance**: Verify that total debits equal total credits. Flag any imbalance for correction before proceeding.

**Duplicate check:**
Search the journal tracker for existing cases matching the same description, amount, accounts, and close period:
- `SearchM365(sources=["files"], query="journal tracker")` to locate the tracker
- `ReadFileContent` to read existing records
- If a potential duplicate is found, present it to the user and ask whether to proceed or link to the existing case

**Generate Journal Case ID:**
Format: JE-YYYY-NNNNN (e.g., JE-2026-00087), incrementing from the last ID in the tracker.

**Write the journal case record** to the tracker with all extracted fields and:
- Status: "New"
- Created Date: current timestamp
- Support Status: "Pending" (until evidence is assembled)
- Approval Path: "Pending" (until routing is determined)
- SLA target: 1 business day to review-ready state

**Present confirmation** via Adaptive Card (invoke `render-ui` skill first):
- Journal Case ID, Description, Requestor, Entity
- Debit Account → Credit Account, Amount, Currency
- Close Period
- Status: New
- Support Status: Pending

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find inbound journal entry request email |
| SearchM365 (files) | Find journal tracker in SharePoint, locate supporting documents |
| SearchM365 (connectors) | Pull journal request from close management system if available |
| GetMessage | Read full request email content |
| ReadFileContent | Read journal tracker for duplicate check |
| SearchPeople / GetUserDetails | Resolve requestor identity and reporting chain |
| GetMyDetails | Current user for audit trail |

## Guardrails

- **Never create duplicate cases** for the same journal description, amount, accounts, and period — check the tracker first
- **Validate debit-credit balance** — reject entries where debits do not equal credits
- **Reject entries missing required fields** — entity, accounts, amount, currency, and period are mandatory
- **Confirm details with user** before writing to the journal tracker
- **Log creation metadata** — timestamp, created-by user, source channel, and request reference for audit traceability
- **Never post journal entries** — this skill creates case records for review; ERP posting is always a human action
- **Preserve original request language** — store the requestor's rationale verbatim as the description
