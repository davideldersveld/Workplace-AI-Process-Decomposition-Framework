---
name: ops-context-packet
description: |
  Gathers process context, transaction details, queue history, applicable
  SOPs, and prior similar exceptions for an operational exception case.
  Use when user asks to "build context for exception [ID]",
  "what happened with [transaction]",
  "assemble case context for [exception]",
  "gather exception details for [case]",
  "pull transaction history for [reference]",
  or "get background on exception [ID]".
  Do NOT use for creating a new exception case (use ops-exception-intake),
  classifying exception type (use ops-classify-exception),
  assessing impact and priority (use ops-impact-assess),
  routing to an owner or queue (use ops-route-exception),
  or drafting follow-up communications (use ops-exception-comms).
---

## Overview

Assembles process and transaction context for an active exception case — pulling transaction details, processing history, error state, queue and work item history, similar prior exceptions and their resolutions, applicable SOPs and resolution guides, and supporting documents. Presents findings via Adaptive Card for rapid analyst review.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (transaction systems, workflow platforms, SharePoint SOPs, prior case history) without exercising judgment on exception classification, priority, or resolution approach.

## When to Use

- An exception case has been created and needs transaction and process context before classification
- An analyst needs to understand what happened with a specific transaction or work item
- A case needs updated context after new information arrives (stakeholder email, system update)
- A specialist resolver needs background on an exception before starting resolution

## When NOT to Use

- Creating a new exception case — use ops-exception-intake
- Classifying exception type and likely cause — use ops-classify-exception
- Assessing impact, priority, and aging risk — use ops-impact-assess
- Assigning an owner or routing to a queue — use ops-route-exception
- Drafting follow-up or handoff communications — use ops-exception-comms
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read exception case and gather source data", activeForm="Gathering exception context")
TaskCreate(subject="Present context packet for analyst review", activeForm="Building context packet")
```

### Step 1: Read Exception Case Data

Locate and read the exception case:
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent`
- Identify: Case ID, Source System, Process Type, Queue, Transaction Reference, Status, Created Date, Assigned Analyst, Reopen Count, Safety Flag

### Step 2: Retrieve Transaction Details

Pull transaction and processing data:

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — transaction details, processing history, error logs, related transactions
- `SearchM365(sources=["connectors"], connector_ids=["workflow-connector"])` — queue state, prior assignments, work item history, status transitions

**Via SharePoint bridge (if Graph Connector unavailable):**
- `SearchM365(sources=["files"], query="[transaction reference] transaction")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[queue name] queue snapshot")` then `ReadFileContent`

Extract:
- Transaction type, parties, amounts, dates, and current error state
- Processing step where the exception occurred
- Related or dependent transactions
- Queue position and aging

### Step 3: Retrieve SOPs and Resolution Guides

Find applicable procedures:
- `SearchM365(sources=["files"], query="[process type] SOP")` then `ReadFileContent` — standard operating procedure for this process type
- `SearchM365(sources=["files"], query="[exception type] resolution guide")` then `ReadFileContent` — resolution guidance if an exception type has been preliminarily identified
- `SearchM365(sources=["files"], query="[queue name] queue rules")` then `ReadFileContent` — queue-specific procedures and escalation rules

### Step 4: Find Prior Similar Exceptions

Search for pattern context:
- `SearchM365(sources=["files"], query="[process type] [error description] exception")` — prior cases with similar characteristics
- Read the exception tracker for cases with the same process type, queue, or transaction type
- Identify resolutions that were applied to similar cases

Compile:
- Number of similar prior exceptions in the last 90 days
- Most common resolution for this type of exception
- Average time to resolution for similar cases
- Whether this is a recurring pattern suggesting a process defect

### Step 5: Find Related Communications

Search for stakeholder context:
- `SearchM365(sources=["email"], query="[transaction reference] [case ID]")` — related correspondence, stakeholder notes, handoff emails
- `SearchM365(sources=["teams"], query="[transaction reference] [case ID]")` — coordination threads, analyst notes, queue discussions

### Step 6: Check for Supporting Documents

Browse the case folder for evidence:
- `GetDriveChildren` — check for supporting documents uploaded to the case workspace (screenshots, error reports, forms, inspection results)
- Inventory what is available and what may be missing

### Step 7: Safety Exception Check

If the case has a preliminary safety flag or the transaction context reveals safety-related indicators:
- **Quality defects** — product or service quality issues
- **Compliance violations** — regulatory or policy breaches
- **Equipment failures** — operational equipment or system failures with safety implications

Flag immediately with elevated visibility regardless of other context. Safety exceptions must be surfaced prominently for the analyst.

### Step 8: Present Context Packet

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Transaction Reference, Process Type, Queue, Aging, Status
- **Transaction details** — type, parties, amounts, dates, error state, processing step where exception occurred
- **Queue and work item history** — prior assignments, status transitions, reopen count
- **Similar prior exceptions** — count, most common resolution, average resolution time, recurring pattern flag
- **Applicable SOP** — reference to the relevant standard operating procedure with section citation
- **Supporting documents** — inventory of available evidence documents
- **Related communications** — summary of stakeholder correspondence and coordination threads
- **Safety flag** — prominently displayed if safety indicators are present
- **Data freshness indicators** — source and timestamp for each data element
- **Missing context** — data that could not be retrieved with source and reason

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (connectors) | Pull transaction details and queue state via Graph Connector |
| SearchM365 (files) | Find exception tracker, SOPs, resolution guides, queue rules, prior cases |
| SearchM365 (email) | Find related correspondence and stakeholder notes |
| SearchM365 (teams) | Find coordination threads and analyst notes |
| ReadFileContent | Read SOPs, routing rules, tracker, queue snapshots |
| GetDriveChildren | Browse case workspace for supporting documents |

## Guardrails

- **Cite data source and timestamp** for every transaction detail and context element — the analyst must know when data was captured and from which system
- **Flag stale or unavailable data explicitly** — if source system data could not be retrieved or is older than expected, surface the gap rather than omitting it silently
- **Never modify transaction records, queue state, or the exception tracker** — this skill is read-only context assembly
- **For safety-related exceptions (quality defects, compliance violations, equipment failures), flag immediately** with elevated visibility regardless of other context — safety exceptions bypass standard presentation priority
- **Never interpret or classify the exception** — context assembly presents facts; classification belongs to ops-classify-exception
- **Never recommend a priority or resolution approach** — context informs the analyst's judgment; priority belongs to ops-impact-assess
- **Scope transaction detail visibility** to what the assigned analyst needs — do not surface customer PII or financial details beyond what is relevant to the exception
- **Include prior similar exception patterns** so the analyst can identify recurring issues, but do not assert root cause — pattern context supports judgment, not replaces it
- **Flag if the SOP or resolution guide has been updated** since the last similar exception was resolved — the analyst should review updated procedures
