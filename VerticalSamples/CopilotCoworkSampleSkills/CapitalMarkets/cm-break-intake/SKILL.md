---
name: cm-break-intake
description: |
  Normalizes trade exception and settlement break events into structured case records
  in the break tracker.
  Use when user asks to "new trade break", "settlement exception for [trade ID]",
  "log break case", "trade fail alert", "SSI mismatch on [counterparty]",
  "intake trade exception", "new break case for [trade ID]",
  "normalize this break event", or "log settlement break".
  Do NOT use for assembling trade and settlement context (use cm-context-packet),
  classifying break type or root cause (use cm-break-classifier),
  assessing settlement risk or time criticality (use cm-risk-assessment),
  routing to desks or operations owners (use cm-break-routing),
  or drafting counterparty or internal communications (use cm-break-comms).
---

## Overview

Converts inbound trade exception and settlement break signals — OMS break notifications, counterparty notices, settlement platform alerts, operations desk escalations, and confirmations system mismatches — into normalized, validated case records in the shared Excel break tracker. Validates required fields, checks for duplicate cases, resolves operations contacts, and confirms details before writing.

This skill operates in "deterministic automation" mode — structured field extraction, date validation, and duplicate checking with no AI judgment. The user confirms all details via Adaptive Card before the tracker is updated.

## When to Use

- A trade exception has been flagged by the OMS or trade management system
- A settlement break notification has arrived from the settlement platform
- An SSI mismatch has been identified on a counterparty trade
- An affirmation or confirmation break needs logging
- A counterparty has sent a break notice via email
- An operations desk has escalated a settlement exception

## When NOT to Use

- Assembling trade, settlement, and counterparty context — use cm-context-packet
- Classifying the break type or likely root cause — use cm-break-classifier
- Assessing settlement risk, fail exposure, or time criticality — use cm-risk-assessment
- Routing the break to the correct desk or owner — use cm-break-routing
- Drafting counterparty or internal handoff communications — use cm-break-comms
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather trade break event details", activeForm="Gathering break details")
TaskCreate(subject="Validate and create break case record", activeForm="Creating break case")
```

### Step 1: Gather Break Event Details

Collect or extract the following from the user's request, break notification, or counterparty notice:

**Required fields:**
- **Trade ID** — from the OMS, trade management system, or break notification
- **Break source** — OMS alert, settlement platform, counterparty notice, operations desk, confirmations system
- **Asset class** — equities, fixed income, FX, derivatives, or other
- **Instrument type** — specific security or instrument description
- **Settlement date** — the expected or scheduled settlement date
- **Counterparty name** — the external counterparty on the trade

**Optional fields:**
- Trade date
- Quantity and notional amount
- Custodian or settlement agent
- SSI reference
- Allocation reference
- Desk or business unit originating the trade
- Operations analyst who identified the break

**Resolve identities:**
- `SearchPeople` — resolve the reporting operations analyst, desk owner, or settlement contact
- `SearchM365(sources=["email"])` — find the break notification email or counterparty notice

**Check for source signal:**
- `SearchM365(sources=["email"], query="trade break [Trade ID]")` — locate the originating break notification
- `SearchM365(sources=["email"], query="settlement exception [counterparty]")` — locate counterparty notices
- If a source email is found, extract key fields from it

### Step 2: Validate and Check Duplicates

**Read the break tracker:**
- `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent`

**Validate required fields:**
- All required fields must be populated before writing
- Settlement date must be present and parseable
- Trade ID must be present

**Check for duplicates:**
- Search the tracker for existing cases with the same Trade ID and settlement date
- If a potential duplicate is found, present it to the user with details and ask whether to proceed, merge, or skip

**Calculate SLA deadline:**
- Standard breaks: 30 minutes from intake timestamp
- Same-day settlement breaks: flag as critical immediately
- Set the SLA Deadline field based on the intake time

**Surface cutoff proximity:**
- Calculate hours remaining until settlement cutoff based on settlement date and current time
- Flag if settlement date is today or next business day

### Step 3: Confirm and Write Case Record

Present the normalized break case record via Adaptive Card (invoke `render-ui` skill first) for user confirmation:

- **Break ID** — auto-generated (format: BRK-YYYY-NNNNN based on year and sequence)
- **Trade ID** — from source system
- **Asset Class** — as identified
- **Instrument** — security or instrument description
- **Counterparty** — counterparty short code (no client account numbers)
- **Settlement Date** — expected settlement date
- **Break Source** — originating system or channel
- **Break Type** — "Pending Classification"
- **Priority** — "Pending Risk Assessment" (or "Critical" if same-day settlement)
- **Fail Risk** — "Pending"
- **Status** — "Intake Complete"
- **Created Timestamp** — current timestamp
- **Assigned To** — blank (pending routing)
- **Desk** — originating desk if known
- **SLA Deadline** — calculated from intake time
- **Cutoff Proximity** — hours to settlement cutoff

After user confirmation, log the case creation with timestamp, creating user, and intake source for audit traceability.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find break notification or counterparty notice email |
| SearchM365 (files) | Find break tracker and break taxonomy reference |
| SearchM365 (connectors) | Pull trade record from OMS if Graph Connector available |
| ReadFileContent | Read the tracker to check for duplicates |
| SearchPeople | Resolve operations analyst and desk ownership contacts |

## Guardrails

- **Never create duplicate cases** for the same Trade ID and settlement date without explicit user confirmation
- **Validate all mandatory fields** before writing — do not create incomplete case records
- **Confirm details via Adaptive Card** before writing to the tracker
- **Never amend bookings, change SSIs, or authorize settlement actions** — this skill creates break case records only
- **Mask client account numbers** in all displayed and generated outputs; use counterparty short codes only
- **Log intake source and timestamp** for audit traceability — every case record must have a complete creation audit entry (SEC Rule 17a-4, FINRA Rule 4511)
- **Never infer trade details** not present in the source signal — flag missing fields explicitly rather than guessing
- **Include settlement date and cutoff proximity** in every case record — T+1 settlement deadlines are operationally critical (SEC Rule 15c6-1)
- **Flag same-day settlement breaks as critical immediately** — these cannot wait for the standard risk assessment step
- **Surface Graph Connector index timestamp** when trade data is sourced from connectors — trade data may be minutes behind the system of record
