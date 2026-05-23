---
name: legal-contract-intake
description: |
  Normalizes inbound contract requests into structured legal case records
  in the contract case tracker.
  Use when user asks to "new contract request", "contract intake for [counterparty]",
  "set up legal case for", "log contract request from [requestor]",
  "incoming contract from [counterparty]",
  "create legal case for [contract type]",
  or "ticket for contract review".
  Do NOT use for assembling review context (use legal-review-packet),
  detecting clause deviations (use legal-deviation-detection),
  determining review path (use legal-review-routing),
  or drafting summaries and follow-ups (use legal-triage-comms).
---

## Overview

Normalizes inbound contract requests — whether submitted via email, Teams, or direct intake — into structured legal case records in the Excel contract case tracker on SharePoint. Validates required fields, checks for duplicate cases, and confirms the record with the analyst via Adaptive Card before writing.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking without AI judgment on contract substance, clause deviations, or review routing.

## When to Use

- A new contract request has been received via email, Teams, or direct submission
- A third-party paper or redlined agreement has been uploaded and needs a case record
- A business requestor has asked legal operations to set up a case for a contract

## When NOT to Use

- Assembling contract context or playbook materials — use legal-review-packet
- Detecting clause deviations against the playbook — use legal-deviation-detection
- Determining the review path or required approvals — use legal-review-routing
- Drafting summaries, status updates, or follow-up requests — use legal-triage-comms
- Confirming final triage disposition — this is always a human decision (LG-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read contract request and validate inputs", activeForm="Reading contract request")
TaskCreate(subject="Create case record in tracker", activeForm="Creating case record")
```

### Step 1: Capture Contract Request

Identify the request source and extract case details:

**From email:**
- `SearchM365(sources=["email"], query="contract request [counterparty or requestor]")` — find the original request thread
- Extract: requestor name, counterparty, contract type, business unit, urgency, any attached documents

**From Teams:**
- `SearchM365(sources=["teams"], query="contract request [counterparty or requestor]")` — find the request message
- Extract the same fields from the Teams message context

**From direct intake (user provides details in the prompt):**
- Extract details from the user's message directly

**Resolve the requestor:**
- `SearchPeople` — resolve requestor by name or email
- `GetUserDetails` — pull requestor profile, department, and business unit

### Step 2: Validate Required Fields

Every case record requires:

| Field | Source | Validation |
|-------|--------|------------|
| Counterparty | Request email, Teams message, or user input | Must be a named legal entity or individual |
| Contract type | Request details | Must match a recognized type: NDA, MSA, SOW, Amendment, License Agreement, Services Agreement, Data Processing Agreement, or Other |
| Business unit | Requestor profile or user input | Must match a recognized department or business unit |
| Requestor | Request source | Must resolve to a valid user in the directory |
| Urgency | Request details or default | Standard (default), Expedited, or Urgent |
| Attached document | Request email, Teams, or SharePoint upload | Note if present; flag if absent — request may need follow-up |

If any required field is missing, prompt the analyst for the missing information before proceeding.

### Step 3: Check for Duplicate Cases

Read the contract case tracker:
- `SearchM365(sources=["files"], query="contract case tracker")` then `ReadFileContent`
- Check for existing open cases with the same counterparty and contract type created within the last 30 days
- If a potential duplicate is found, flag it for the analyst with the existing Case ID and creation date

### Step 4: Generate Case ID

Assign a sequential Case ID following the format: `LGL-YYYY-NNNN` where YYYY is the current year and NNNN is the next sequential number from the tracker.

### Step 5: Present Case Record for Confirmation

Present the proposed case record via Adaptive Card (invoke `render-ui` skill first):

- **Case ID** — generated identifier
- **Counterparty** — legal entity name
- **Contract type** — NDA, MSA, SOW, Amendment, etc.
- **Business unit** — requestor's department
- **Requestor** — name, email, title
- **Urgency** — Standard, Expedited, or Urgent
- **Received date** — date the request was received
- **Status** — "Intake Complete" (initial status)
- **Attached document** — filename if present; "No document attached — follow-up required" if absent
- **Duplicate check result** — "No duplicates found" or details of potential match
- **SLA target** — 1 business day for standard triage completion

### Step 6: Write to Case Tracker (After Confirmation)

After analyst confirmation, write the case record to the Excel contract case tracker.

Record the following columns:
- Case ID, Counterparty, Contract Type, Business Unit, Requestor, Urgency, Status, Received Date, Assigned Counsel (blank — assigned in routing step), Deviation Count (blank — populated after deviation detection), Disposition (blank — set in final triage)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the original contract request email thread |
| SearchM365 (teams) | Find contract request messages in Teams |
| SearchM365 (files) | Find the contract case tracker in SharePoint |
| ReadFileContent | Read the case tracker to check for duplicates and determine next Case ID |
| SearchPeople | Resolve requestor identity |
| GetUserDetails | Pull requestor profile, department, and business unit |

## Guardrails

- **Never create duplicate cases** for the same counterparty and contract type within 30 days — flag potential duplicates for analyst review
- **Validate all required fields** before writing to the tracker — prompt for missing information rather than writing incomplete records
- **Confirm the case record with the analyst via Adaptive Card** before writing to the tracker — intake writes are not auto-committed
- **Never extract, display, or summarize terms from attached contract documents** at this stage — intake captures metadata only; contract substance is analyzed in downstream skills
- **Never assign counsel or determine review path** during intake — those decisions belong to legal-review-routing
- **Never assess clause deviations or risk** during intake — that analysis belongs to legal-deviation-detection
- **Include the SLA target** (1 business day for standard triage) in every case record for tracking
- **Tag the intake source** (email, Teams, direct intake) in the case record for audit trail
- **Flag missing contract documents** — if no document is attached, note this in the case record so legal-triage-comms can draft a follow-up request
