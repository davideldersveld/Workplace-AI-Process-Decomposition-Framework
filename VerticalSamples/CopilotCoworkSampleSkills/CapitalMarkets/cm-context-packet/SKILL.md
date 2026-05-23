---
name: cm-context-packet
description: |
  Assembles trade details, settlement instructions, counterparty context, prior break
  history, and applicable procedures into a trade break context packet.
  Use when user asks to "build break packet for [trade ID]",
  "assemble context for break [ID]", "what do we know about this trade exception",
  "pull settlement details for [counterparty]", "trade break context for [case]",
  "prepare break case materials", "evidence packet for [break ID]",
  or "SSI and settlement summary for [trade]".
  Do NOT use for creating a new break case (use cm-break-intake),
  classifying break type or root cause (use cm-break-classifier),
  assessing settlement risk or time criticality (use cm-risk-assessment),
  routing to desks or operations owners (use cm-break-routing),
  or drafting counterparty or internal communications (use cm-break-comms).
---

## Overview

Assembles a comprehensive trade break context packet for a case by gathering trade details, settlement instructions, affirmation and confirmation status, counterparty and SSI context, prior break history, applicable settlement procedures and cutoff schedules, and an evidence inventory. Produces a Word document for the auditable case file.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (SharePoint procedures, Graph Connectors for trade data, break tracker) without interpreting break patterns, making risk judgments, or recommending settlement actions.

## When to Use

- A break case has been created and needs trade and settlement context assembled
- An analyst needs the full picture for a break before classification or review
- Prior break history needs to be compiled for a repeat-pattern investigation
- A context packet needs to be refreshed with updated information

## When NOT to Use

- Creating a new break case — use cm-break-intake
- Classifying the break type or likely root cause — use cm-break-classifier
- Assessing settlement risk, fail exposure, or time criticality — use cm-risk-assessment
- Routing to the correct desk or owner — use cm-break-routing
- Drafting counterparty or internal handoff communications — use cm-break-comms
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read break case data and locate source materials", activeForm="Reading break data")
TaskCreate(subject="Assemble trade break context packet", activeForm="Assembling context packet")
```

### Step 1: Read Case Data and Locate Source Materials

**Read the break case record:**
- `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent` to find the case row by Break ID or Trade ID

**Resolve people:**
- `GetUserDetails` — assigned analyst (if any), operations contacts
- `SearchPeople` — desk owners, settlement contacts, counterparty relationship managers

**Retrieve trade and settlement data (when available):**
- `SearchM365(sources=["connectors"], connector_ids=["oms-connector"])` — trade record: trade date, settlement date, asset class, instrument, quantity, price, counterparty, allocation details
- `SearchM365(sources=["connectors"], connector_ids=["settlement-connector"])` — settlement status, SSI details, affirmation and confirmation status
- If Graph Connectors are not available, note the data gap and prompt the user for manual input of key fields

**Locate procedures and reference materials:**
- `SearchM365(sources=["files"], query="settlement cutoff schedules")` — cutoff times by market, custodian, and currency
- `SearchM365(sources=["files"], query="break taxonomy")` — break type definitions and classification criteria
- `SearchM365(sources=["files"], query="SSI reference")` — standing settlement instructions for the counterparty
- `SearchM365(sources=["files"], query="settlement procedures")` — settlement handling rules and fail escalation guidance
- `ReadFileContent` — read each located document

**Find related communications and evidence:**
- `SearchM365(sources=["email"], query="[Break ID] OR [Trade ID] OR [counterparty name]")` — related counterparty correspondence, internal desk emails, break notification threads
- `GetDriveChildren` — list documents in the break case evidence folder in SharePoint

**Log every document and data source accessed** — record source system, document name, retrieval timestamp, and Graph Connector index timestamp (if applicable) for evidence traceability.

### Step 2: Assemble Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Trade Summary**
   - Trade ID, trade date, settlement date
   - Asset class, instrument type, security identifier
   - Quantity, price, notional amount (if available from approved sources)
   - Counterparty short code (no client account numbers)
   - Originating desk and trader (if available)
   - Allocation references (if applicable)

2. **Settlement and SSI Details**
   - Expected settlement instructions (from SSI reference)
   - Received or matched settlement instructions
   - SSI comparison: expected vs. received — highlight discrepancies
   - Custodian and settlement agent details
   - Settlement method (DTC, Euroclear, manual, etc.)

3. **Affirmation and Confirmation Status**
   - Affirmation status (affirmed, unaffirmed, partially affirmed)
   - Confirmation status (confirmed, unconfirmed, mismatched)
   - Timestamp of last status update
   - Any outstanding confirmation discrepancies

4. **Prior Break History**
   - Prior breaks on the same counterparty (Break ID, date, type, disposition)
   - Prior breaks on the same instrument or asset class
   - Prior breaks from the same desk
   - Pattern indicators (same SSI issue, same counterparty, velocity)

5. **Counterparty and Channel Context**
   - Counterparty short code, relationship type
   - Known counterparty operational issues (if documented in procedures)
   - Channel of the break (OMS alert, settlement platform, counterparty email, desk escalation)
   - Related email thread excerpts with timestamps

6. **Applicable Procedures and Settlement Rules**
   - Settlement cutoff schedule for the relevant market and custodian
   - Fail escalation procedures
   - Break handling procedures for the break source type
   - Regulatory requirements (T+1 settlement timeline under SEC Rule 15c6-1)

7. **Evidence Inventory**
   - Documents received (from break case evidence folder)
   - Documents pending (trade confirmation, allocation file, SSI documentation)
   - Data sourced from Graph Connectors (with index timestamp and reliability note)
   - Data sourced from email (with reliability warning — email is not system of record)

8. **Cutoff Proximity**
   - Current time, settlement cutoff time, hours remaining
   - Settlement date and business day status

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find break tracker, settlement procedures, cutoff schedules, SSI reference, break taxonomy |
| SearchM365 (email) | Find related counterparty correspondence and break notification threads |
| SearchM365 (connectors) | Retrieve trade and settlement data from OMS and settlement platform via Graph Connector |
| ReadFileContent | Read procedures, cutoff schedules, tracker, SSI reference, and evidence documents |
| GetDriveChildren | List documents in the break case evidence folder |
| GetUserDetails / SearchPeople | Resolve analyst, desk owner, and operations contact identities |

## Guardrails

- **Never include client account numbers** — use counterparty short codes only; mask any account identifiers in all generated outputs
- **Cite the source system, retrieval timestamp, and Graph Connector index timestamp** for every data element — examiners need to trace every fact
- **Flag data sourced from email** with a reliability warning — email narrative and system settlement state may diverge; the system of record takes precedence
- **Never present stale settlement data without noting the retrieval time** — settlement states change rapidly; note retrieval time on every data point
- **Do not include MNPI** (material non-public information) from unrelated business lines — respect information barriers (Chinese walls)
- **Do not include position sizes or pricing data** beyond what is directly relevant to the break — restrict to trade-level details only
- **Read-only access to all trade and settlement data** — no write operations to OMS, settlement platform, or SSI repository
- **Never interpret break likelihood or recommend settlement actions** — present factual context only; the analyst makes all judgment calls
- **Preserve evidence traceability** — log every document and data source accessed with timestamp and source location
- **Surface cutoff proximity** in every context packet — operations teams must always see time remaining to settlement cutoff
- **Restrict packet access** — note that the generated packet should be stored in the break case folder with appropriate SharePoint permissions
