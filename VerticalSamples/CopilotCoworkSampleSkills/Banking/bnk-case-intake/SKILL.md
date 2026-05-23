---
name: bnk-case-intake
description: |
  Normalizes fraud alert and dispute submission events into structured case records
  in the fraud and dispute case tracker.
  Use when user asks to "new fraud alert", "dispute case for [customer]",
  "normalize this fraud event", "intake fraud report", "new dispute submission",
  "log fraud case", "create dispute case for [account]",
  "intake for [alert ID]", or "new card fraud case".
  Do NOT use for assembling account and transaction context (use bnk-fraud-context-packet),
  classifying fraud scenario or dispute type (use bnk-scenario-classifier),
  assessing urgency or evidence gaps (use bnk-gap-risk-detection),
  routing to analyst queues (use bnk-case-routing),
  or drafting customer or analyst communications (use bnk-case-comms).
---

## Overview

Converts inbound fraud and dispute signals — fraud monitoring alerts, customer dispute submissions, contact center reports, branch escalations, and digital channel notifications — into normalized, validated case records in the shared Excel fraud and dispute case tracker. Validates required fields, checks for duplicate alerts, resolves contact identities, and confirms details before writing.

This skill operates in "deterministic automation" mode — structured field extraction, date validation, and duplicate checking with no AI judgment. The user confirms all details via Adaptive Card before the tracker is updated.

## When to Use

- A fraud monitoring alert has fired and needs a case record
- A customer has submitted a dispute through any channel (digital, branch, contact center)
- A contact center representative has taken a fraud or dispute report
- A branch has escalated a suspected fraud event
- An existing alert needs to be logged as a new case

## When NOT to Use

- Assembling account, transaction, and customer context — use bnk-fraud-context-packet
- Classifying the fraud scenario or dispute type — use bnk-scenario-classifier
- Assessing urgency, evidence gaps, or risk path — use bnk-gap-risk-detection
- Routing the case to an analyst queue — use bnk-case-routing
- Drafting customer or analyst communications — use bnk-case-comms
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather fraud or dispute event details", activeForm="Gathering case details")
TaskCreate(subject="Validate and create case record", activeForm="Creating case record")
```

### Step 1: Gather Event Details

Collect or extract the following from the user's request, alert notification, or customer report:

**Required fields:**
- **Alert ID** — from the fraud monitoring system (if alert-originated)
- **Customer name** — as identified in the alert or report
- **Account number** — the affected account (will be masked in all outputs)
- **Transaction reference** — the specific transaction under dispute or alert
- **Dispute type or fraud indicator** — initial categorization from the source
- **Source channel** — digital, branch, contact center, monitoring system

**Optional fields:**
- Card number (will be masked to last four digits)
- Merchant name and category
- Transaction amount and date
- Customer narrative summary
- Contact center representative or branch contact

**Resolve identities:**
- `SearchPeople` — resolve the reporting contact center representative or branch contact
- `SearchM365(sources=["email"])` — find the alert notification email or customer dispute submission

**Check for source signal:**
- `SearchM365(sources=["email"], query="fraud alert [alert ID]")` — locate the originating alert
- If a source email is found, extract key fields from it

### Step 2: Validate and Check Duplicates

**Read the case tracker:**
- `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent`

**Validate required fields:**
- All required fields must be populated before writing
- Transaction reference must be present
- Account number must be present (will be masked in outputs)

**Check for duplicates:**
- Search the tracker for existing cases with the same Alert ID
- Search for cases with the same transaction reference
- If a potential duplicate is found, present it to the user with details and ask whether to proceed, merge, or skip

**Calculate SLA deadline:**
- Standard cases: 30 minutes from intake timestamp
- Set the SLA Deadline field based on the intake time

### Step 3: Confirm and Write Case Record

Present the normalized case record via Adaptive Card (invoke `render-ui` skill first) for user confirmation:

- **Case ID** — auto-generated (format: FRD-YYYY-NNNNN based on year and sequence)
- **Alert ID** — from source system
- **Customer Name** — as identified
- **Account Number** — masked to last four digits in display
- **Transaction Reference** — the flagged transaction
- **Dispute Type / Fraud Indicator** — initial categorization
- **Channel** — source channel
- **Status** — "Intake Complete"
- **Created Date** — current timestamp
- **Assigned To** — blank (pending routing)
- **SLA Deadline** — calculated from intake time
- **Priority** — "Standard" (pending risk assessment)

After user confirmation, log the case creation with timestamp, creating user, and intake source for audit traceability.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find alert notification or customer dispute submission email |
| SearchM365 (files) | Find case tracker and intake normalization procedures |
| ReadFileContent | Read the tracker to check for duplicates |
| SearchPeople | Resolve contact center representative or branch contact identities |

## Guardrails

- **Never create duplicate cases** for the same Alert ID or transaction reference without explicit user confirmation
- **Validate all mandatory fields** before writing — do not create incomplete case records
- **Confirm details via Adaptive Card** before writing to the tracker
- **Never modify account status, card status, or transaction state** — this skill creates case records only
- **Mask account and card numbers** to last four digits in all displayed and generated outputs
- **Log intake source and timestamp** for audit traceability — every case record must have a complete creation audit entry
- **Never interpret the fraud or dispute event** — record factual intake information only; classification is handled by bnk-scenario-classifier
- **Include SLA deadline** in every case record — the 30-minute triage window is operationally critical
