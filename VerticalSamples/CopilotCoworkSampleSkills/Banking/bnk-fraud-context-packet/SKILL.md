---
name: bnk-fraud-context-packet
description: |
  Assembles account profile, transaction details, prior case history, merchant context,
  and applicable procedures into a fraud or dispute context packet.
  Use when user asks to "build fraud context packet", "assemble case context for [case ID]",
  "gather transaction history for this dispute", "what do we know about this fraud case",
  "pull account context for [case]", "prepare fraud case materials",
  "evidence context for [case ID]", or "account and transaction summary for [dispute]".
  Do NOT use for creating a new case (use bnk-case-intake),
  classifying fraud scenario or dispute type (use bnk-scenario-classifier),
  assessing urgency or evidence gaps (use bnk-gap-risk-detection),
  routing to analyst queues (use bnk-case-routing),
  or drafting communications (use bnk-case-comms).
---

## Overview

Assembles a comprehensive fraud or dispute context packet for a case by gathering customer and account profile, transaction details for the flagged activity, prior fraud alert and dispute history, merchant and channel context, applicable fraud procedures and dispute handling rules, and an evidence inventory. Produces a Word document for the auditable case file.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (SharePoint procedures, Graph Connectors for banking data, case tracker) without interpreting fraud patterns, making risk judgments, or recommending account actions.

## When to Use

- A fraud or dispute case has been created and needs account and transaction context assembled
- An analyst needs the full picture for a case before classification or review
- Prior case history needs to be compiled for a repeat-pattern investigation
- A context packet needs to be refreshed with updated information

## When NOT to Use

- Creating a new case — use bnk-case-intake
- Classifying the fraud scenario or dispute type — use bnk-scenario-classifier
- Assessing urgency, evidence gaps, or risk — use bnk-gap-risk-detection
- Routing to an analyst queue — use bnk-case-routing
- Drafting customer or analyst communications — use bnk-case-comms
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and locate source materials", activeForm="Reading case data")
TaskCreate(subject="Assemble fraud context packet", activeForm="Assembling context packet")
```

### Step 1: Read Case Data and Locate Source Materials

**Read the case record:**
- `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent` to find the case row by Case ID or Alert ID

**Resolve people:**
- `GetUserDetails` — assigned analyst (if any), reporting representative
- `SearchPeople` — fraud operations contacts referenced in the case

**Retrieve banking data (when available):**
- `SearchM365(sources=["connectors"], connector_ids=["core-banking-connector"])` — customer account profile, account type, customer tier, account status
- `SearchM365(sources=["connectors"], connector_ids=["core-banking-connector"])` — transaction details for the disputed or flagged transactions
- If Graph Connectors are not available, note the data gap and prompt the user for manual input of key fields

**Locate procedures and reference materials:**
- `SearchM365(sources=["files"], query="fraud procedures")` — fraud handling procedures, dispute procedures
- `SearchM365(sources=["files"], query="dispute evidence checklist")` — evidence requirements for the dispute type
- `SearchM365(sources=["files"], query="Reg E guidance")` — Regulation E handling guidance if applicable
- `ReadFileContent` — read each located document

**Find related communications and evidence:**
- `SearchM365(sources=["email"], query="[Case ID] OR [Alert ID] OR [customer name]")` — related customer correspondence, alert threads
- `GetDriveChildren` — list documents in the case evidence folder in SharePoint

**Log every document and data source accessed** — record source system, document name, retrieval timestamp for evidence traceability.

### Step 2: Assemble Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Customer and Account Profile**
   - Customer name, customer tier, account type, product type
   - Account status (active, restricted, closed)
   - Account opening date and relationship tenure
   - Account number masked to last four digits

2. **Transaction Details**
   - Flagged transaction(s): date, amount, merchant name, merchant category, channel, authorization status
   - Transaction reference numbers
   - Related transactions in the same timeframe (if available)
   - Card number masked to last four digits (if card-related)

3. **Prior Fraud Alert and Dispute History**
   - Prior fraud alerts on the same account (alert ID, date, type, disposition)
   - Prior disputes on the same account (case ID, date, type, outcome)
   - Prior account restrictions or card replacements
   - Pattern indicators (same merchant, same channel, velocity)

4. **Merchant and Channel Context**
   - Merchant name, category code, location
   - Known merchant risk indicators (if available from fraud procedures)
   - Channel of the disputed transaction (POS, online, ATM, mobile, branch)
   - Channel of the customer report (digital, contact center, branch)

5. **Applicable Procedures and Handling Rules**
   - Fraud handling procedures for the alert type
   - Dispute handling procedures for the dispute type
   - Reg E applicability and provisional credit timeline (if applicable)
   - Card network chargeback rules and deadlines (if card-related)
   - Evidence requirements from the dispute checklist

6. **Evidence Inventory**
   - Documents received (from case evidence folder)
   - Documents pending (from evidence checklist comparison)
   - Customer narrative or statement (if available from call notes or email)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, fraud procedures, dispute checklists, Reg E guidance |
| SearchM365 (email) | Find related customer correspondence and alert notification threads |
| SearchM365 (connectors) | Retrieve account and transaction data from core banking via Graph Connector |
| ReadFileContent | Read procedures, checklists, tracker, and evidence documents |
| GetDriveChildren | List documents in the case evidence folder |
| GetUserDetails / SearchPeople | Resolve analyst and operations contact identities |

## Guardrails

- **Never include full account numbers or card numbers** — mask to last four digits in all generated outputs
- **Cite the source system and retrieval timestamp** for every data element — examiners need to trace every fact
- **Flag if any required data source is unavailable** or returns stale results — note explicitly as "data not available from [source]" with search terms used
- **Do not surface SAR or suspicious activity investigation status** — these are restricted to AML teams and must never appear in a general fraud context packet
- **Do not surface internal fraud scores or risk ratings** in the context packet — these are for analyst-only tools, not case documentation
- **Read-only access to all banking data** — no write operations to core systems, no account modifications, no transaction state changes
- **Never interpret fraud likelihood or dispute validity** — present factual context only; the analyst makes all judgment calls
- **Preserve evidence traceability** — log every document and data source accessed with timestamp and source location
- **Restrict packet access** — note that the generated packet should be stored in the case folder with appropriate SharePoint permissions
