---
name: ops-exception-intake
description: |
  Normalizes inbound operational exception events into structured case
  records in the exception tracker.
  Use when user asks to "new exception case",
  "log operational exception", "exception from [system]",
  "intake exception for [process]", "queue exception for [team]",
  "create exception case for [transaction]",
  or "record exception from [source]".
  Do NOT use for gathering transaction context (use ops-context-packet),
  classifying exception type (use ops-classify-exception),
  assessing impact and priority (use ops-impact-assess),
  routing to an owner or queue (use ops-route-exception),
  or drafting follow-up communications (use ops-exception-comms).
---

## Overview

Normalizes inbound operational exception events — from transaction processing errors, service delivery issues, quality checks, or manual queue intake — into structured case records in the shared Excel exception tracker. Validates required fields, checks for duplicate exceptions on the same transaction reference and date, flags reopened exceptions, and sends a Teams notification to the assigned analyst and queue channel.

This skill operates in "Deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking without exercising judgment on exception severity, classification, or routing.

## When to Use

- An operational exception event has occurred and needs to be logged as a case
- An exception notification has arrived via email, Teams, or system alert and needs intake
- A manual exception needs to be entered from a phone call, inspection, or walk-up request
- A queue manager needs to confirm whether an exception has already been logged

## When NOT to Use

- Gathering process and transaction context — use ops-context-packet
- Classifying exception type and likely cause — use ops-classify-exception
- Assessing impact, priority, and aging risk — use ops-impact-assess
- Assigning an owner or routing to a queue — use ops-route-exception
- Drafting follow-up or handoff communications — use ops-exception-comms
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Validate exception inputs and check for duplicates", activeForm="Validating exception intake")
TaskCreate(subject="Create exception case record", activeForm="Creating exception case")
```

### Step 1: Gather Exception Inputs

Collect the following from the user or source artifact:

| Field | Required | Description |
|-------|----------|-------------|
| **Exception source** | Yes | Transaction processing, service delivery, quality check, manual intake, system alert |
| **Process type or queue** | Yes | The operational process or queue where the exception occurred |
| **Transaction or case reference** | Yes | The transaction ID, order number, case number, or other unique reference |
| **Initial description** | Yes | What happened — the error, discrepancy, or issue description |
| **Source system** | If available | The system that generated or detected the exception |
| **Requesting analyst** | If available | The person reporting or flagging the exception |

If the exception arrives via email or Teams, extract these fields from the message:
- `SearchM365(sources=["email"], query="[exception reference or keywords]")` — find the exception notification email
- `SearchM365(sources=["teams"], query="[exception reference or keywords]")` — find the exception report in Teams

If a Graph Connector is available for the workflow platform:
- `SearchM365(sources=["connectors"], connector_ids=["workflow-connector"])` — pull the source system record

### Step 2: Validate Required Fields

Check that all required fields are present:

- **Exception source** — must be one of the recognized source types
- **Process type or queue** — must match the organization's queue taxonomy
- **Transaction reference** — must be a valid reference format
- **Initial description** — must contain actionable detail (not just "error" or "issue")

If required fields are missing, present what was captured and identify the gaps for the user.

### Step 3: Check for Duplicates

Search the exception tracker for existing cases:
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent`
- Check for any open case with the same transaction reference and exception date
- If a duplicate is found, present the existing case details and ask whether this is a new exception or a duplicate

### Step 4: Check for Reopened Exceptions

Search for prior closed cases on the same transaction reference:
- If a closed case exists with the same transaction reference, flag this as a **reopened exception**
- Reopened exceptions receive elevated visibility: increment the Reopen Count and flag prominently in the case record

### Step 5: Generate Case ID and Record

Create the exception case record with the following fields:

| Column | Value |
|--------|-------|
| **Case ID** | Auto-generated (OPS-YYYYMMDD-NNN format) |
| **Source System** | From intake |
| **Process Type** | From intake |
| **Queue** | From intake |
| **Transaction Ref** | From intake |
| **Exception Type** | Pending classification (set by ops-classify-exception) |
| **Priority** | Pending assessment (set by ops-impact-assess) |
| **Status** | New |
| **Created Date** | Current timestamp |
| **Assigned Analyst** | Resolved from queue default or requesting analyst |
| **Aging (hours)** | 0 |
| **SLA Deadline** | Pending priority assignment |
| **Reopen Count** | 0 (or incremented if reopened) |
| **Safety Flag** | Pending classification |

### Step 6: Resolve Analyst Assignment

Determine the default analyst:
- `SearchPeople` — resolve the operations analyst assigned to this queue or process type
- If the requesting analyst is known and is a valid queue member, assign to them
- If no default analyst can be determined, assign to the queue manager for triage

### Step 7: Send Intake Notification

After case creation:
- `PostMessage` — send a Teams notification to the operations queue channel with:
  - Case ID and transaction reference
  - Process type and queue
  - Exception source and initial description
  - Assigned analyst
  - Reopened flag (if applicable)

### Step 8: Present Confirmation

Present the created case record via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Transaction Reference, Created Date
- **Exception details** — Source, Process Type, Queue, Initial Description
- **Assignment** — Assigned Analyst
- **Status flags** — New case or Reopened (with reopen count and prior case reference)
- **Next steps** — "Run ops-context-packet to gather transaction context" and "Run ops-classify-exception to classify the exception type"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find exception notification emails |
| SearchM365 (teams) | Find exception reports in Teams channels |
| SearchM365 (files) | Find and read the exception tracker for duplicate checking |
| SearchM365 (connectors) | Pull source system records via Graph Connector |
| ReadFileContent | Read the current exception tracker |
| SearchPeople | Resolve analyst assignment by queue or process type |
| PostMessage | Send Teams notification to queue channel |

## Guardrails

- **Never create duplicate cases** for the same transaction reference and exception date — always check the tracker first and present the existing case if found
- **Flag reopened exceptions** with elevated visibility — a reopened exception on the same transaction reference indicates a prior resolution may have failed
- **Validate process type against the queue taxonomy** — reject or flag unrecognized process types for queue manager review
- **Log case creation with actor and timestamp** in the tracker for audit trail
- **Never assign a priority or exception type at intake** — these are determined by ops-classify-exception and ops-impact-assess respectively
- **Never auto-route or auto-escalate at intake** — intake creates the case record; routing is a separate step
- **Include the transaction reference in all notifications** so analysts can cross-reference with source systems
- **Never include customer PII in queue channel notifications** — use transaction references and case IDs only
- **For exceptions that mention safety, quality defects, compliance violations, or equipment failures in the initial description**, add a preliminary safety flag to the case record for downstream skills to act on
