---
name: fc-journal-context
description: |
  Assembles ledger context, accounting policy extracts, supporting document inventory,
  approval thresholds, and historical context into a reviewer evidence packet.
  Use when user asks to "build journal packet", "gather context for journal [ID]",
  "assemble support for close entry", "what do we need for this journal",
  "journal packet for [case]", "pull context for journal [ID]",
  "evidence packet for [entry]", or "reviewer packet for journal [ID]".
  Do NOT use for creating a new journal case (use fc-journal-intake),
  detecting missing evidence or risks (use fc-gap-risk-detection),
  determining the approval path (use fc-approval-routing),
  or drafting summaries and follow-ups (use fc-journal-comms).
---

## Overview

Assembles all context needed for journal entry review into a single Word document — the journal evidence packet. Gathers accounting policy extracts, ledger context, supporting document inventory, approval thresholds, historical comparisons, and key contacts from M365 and federated data sources.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources and assembles it without interpreting policy or making accounting judgments. It does not modify any source records.

## When to Use

- A journal case has been created and needs context assembled before review
- The user wants to understand what supporting evidence exists for a journal entry
- An accountant needs a complete evidence packet before routing for approval

## When NOT to Use

- Creating a new journal case — use fc-journal-intake
- Detecting missing evidence or control risks — use fc-gap-risk-detection
- Determining the approval path — use fc-approval-routing
- Drafting journal summaries or follow-up requests — use fc-journal-comms
- Posting a journal entry to the ERP — this is always a human action outside of Cowork

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read journal case and locate context sources", activeForm="Reading journal case data")
TaskCreate(subject="Assemble journal evidence packet", activeForm="Assembling evidence packet")
```

### Step 1: Read Journal Case Record

Locate and read the journal case:

- `SearchM365(sources=["files"], query="journal tracker")` to find the tracker
- `ReadFileContent` to read the specific journal case row
- Extract: Journal Case ID, Description, Requestor, Entity, Business Unit, Debit Account, Credit Account, Amount, Currency, Close Period

### Step 2: Gather Context from All Sources

Retrieve context from each source in parallel where possible:

**Accounting policies:**
- `SearchM365(sources=["files"], query="accounting policy journal entry")` for journal entry standards
- `SearchM365(sources=["files"], query="[entry type] accounting policy")` for type-specific policies (accruals, reclassifications, intercompany, etc.)
- `SearchM365(sources=["files"], query="materiality threshold")` for materiality and approval thresholds
- Collect: applicable policy sections, materiality thresholds, required evidence types, approval requirements

**Ledger context:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` for current account balances, prior period comparisons, and account metadata
- `SearchM365(sources=["files"], query="[account code] ledger")` for ledger extracts in SharePoint
- `SearchM365(sources=["files"], query="trial balance [entity] [period]")` for trial balance data
- Collect: current balances for affected accounts, prior period balances, account descriptions, normal balance direction

**Supporting document inventory:**
- `GetDriveChildren` to list files in the journal case folder
- `SearchM365(sources=["files"], query="[journal case ID] support")` for supporting schedules and memos
- Collect: list of uploaded documents, their types (schedule, reconciliation, memo, approval), and what is still missing

**Historical context:**
- `SearchM365(sources=["files"], query="journal tracker")` and `ReadFileContent` to find similar entries in prior periods
- Look for: same accounts, same entity, similar amounts, same entry type in prior close periods
- Collect: prior period entries with amounts, approval outcomes, and any notes

**Approval thresholds:**
- `SearchM365(sources=["files"], query="approval matrix")` or `SearchM365(sources=["files"], query="approval threshold")`
- `ReadFileContent` to read the threshold table
- Determine: required approval level based on amount, entry type, and entity

**Key contacts:**
- `SearchPeople(query="<requestor>")` and `GetUserDetails` for requestor profile
- `GetManagerDetails(user_id="<requestor>")` for reporting chain
- `SearchPeople(query="controller [entity]")` or `SearchPeople(query="accounting manager [business unit]")` for approver candidates
- Collect: requestor, preparer, accounting manager, controller, close coordinator

### Step 3: Produce Journal Evidence Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Journal Entry Details** — Case ID, description, requestor, entity, business unit, debit account, credit account, amount, currency, close period
2. **Accounting Policy Context** — applicable policy sections with document reference and version date; materiality thresholds; required evidence types for this entry type
3. **Ledger Context** — current balances for affected accounts, prior period comparison, normal balance direction, account descriptions
4. **Supporting Document Inventory** — list of uploaded evidence with document type and status; list of required but missing documents
5. **Approval Requirements** — required approval level per the threshold table; named approver candidates; segregation-of-duties note (preparer cannot approve)
6. **Historical Context** — similar entries from prior periods with amounts, approval outcomes, and any relevant notes
7. **Key Contacts** — requestor, preparer, accounting manager, controller, close coordinator
8. **Missing Context Flags** — any critical data that could not be retrieved, with source and reason

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find journal tracker, accounting policies, approval matrices, ledger extracts, supporting documents |
| SearchM365 (connectors) | Pull ledger balances and account metadata from ERP if Graph Connector available |
| ReadFileContent | Read tracker, policies, threshold tables, supporting documents |
| GetDriveChildren | List files in the journal case evidence folder |
| SearchPeople / GetUserDetails | Resolve requestor, accounting manager, controller |
| GetManagerDetails | Reporting chain for approval routing |

## Guardrails

- **Cite source and retrieval date** for every data element — policy version, ledger extract date, document upload date
- **Flag missing supporting documents** — if required evidence (schedule, reconciliation, memo) is not in the case folder, flag prominently
- **Never include draft or unposted ledger data** without clearly labeling it as preliminary
- **Mark every policy extract** with its version date and document reference
- **Operate read-only** — never update the ERP, close management system, or journal tracker from this skill
- **Protect sensitive financial data** — do not include account numbers or balances in Teams channel posts; limit to the Word packet and direct communications
- **Record assembly metadata** — note which sources were queried, what was found, and what was missing
