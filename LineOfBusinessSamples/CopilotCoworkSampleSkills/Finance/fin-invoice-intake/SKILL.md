---
name: fin-invoice-intake
description: |
  Normalizes inbound AP invoice exception events into structured case records
  in the exception tracker, validates invoice data, and checks for duplicate cases.
  Use when user asks to "new invoice exception", "AP exception for invoice [number]",
  "invoice failed matching", "set up exception case for",
  "log AP exception", "create exception case for [vendor]",
  "intake invoice exception", or "invoice exception request".
  Do NOT use for assembling matching context (use fin-match-context),
  classifying exception type (use fin-exception-classify),
  assessing risk or control path (use fin-control-path),
  routing to action owner (use fin-exception-routing),
  drafting outreach or approval packets (use fin-ap-comms),
  or updating system status (use fin-status-update).
---

## Overview

Normalizes inbound AP invoice exception events — from ERP exception queues, supplier email, invoice capture platforms, or direct intake — into structured case records in the shared Excel exception tracker. Validates that the invoice amount is positive, checks for duplicate cases by invoice ID, rejects entries missing required fields, and logs creation metadata for audit traceability.

This skill operates in "deterministic automation" mode — structured field extraction, data validation, duplicate checking, and case creation follow fixed rules with no AI judgment required.

## When to Use

- An invoice has failed three-way matching and entered the AP exception queue
- A supplier invoice needs to be logged as an exception case for triage
- The user wants to create a structured exception case from an incoming invoice event

## When NOT to Use

- Assembling PO, receipt, vendor, and policy context — use fin-match-context
- Classifying the exception type — use fin-exception-classify
- Assessing risk or determining the control path — use fin-control-path
- Routing to the action owner or approver — use fin-exception-routing
- Drafting outreach emails or approval packets — use fin-ap-comms
- Updating case status in the tracker — use fin-status-update
- Final payment release or vendor bank changes — these are always human actions outside of Cowork

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read inbound invoice event and extract exception fields", activeForm="Reading invoice event")
TaskCreate(subject="Validate, check for duplicates, and write case record", activeForm="Creating exception case")
```

### Step 1: Read Inbound Invoice Event

Identify the source of the invoice exception:

- **ERP exception queue**: `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` if Graph Connector is configured
- **Invoice capture platform**: `SearchM365(sources=["connectors"], connector_ids=["invoice-capture-connector"])` if available
- **Supplier email**: `SearchM365(sources=["email"])` with vendor name or invoice number, then `GetMessage(message_id=...)` to read the full email
- **SharePoint upload**: `SearchM365(sources=["files"])` to locate invoice documents or exception forms

Extract the following fields from the inbound event:

| Field | Description | Required |
|-------|-------------|----------|
| Invoice ID | Invoice number or document reference | Yes |
| Vendor ID | Vendor number or supplier identifier | Yes |
| Vendor Name | Supplier display name | Yes |
| PO ID | Purchase order reference (if available) | If available |
| Amount | Invoice amount | Yes |
| Currency | Currency code (USD, EUR, etc.) | Yes |
| Exception Source | How the exception was triggered (three-way match failure, missing PO, vendor mismatch, missing receipt, duplicate candidate) | Yes |
| Business Unit | Cost center, department, or business unit | Yes |
| Invoice Date | Date on the invoice document | If available |
| Due Date | Payment due date | If available |

**Resolve identities:**
- `SearchPeople(query="<submitter name>")` to resolve who submitted or flagged the exception
- `GetMyDetails` to record who created the exception case

### Step 2: Validate and Create Record

**Validation checks:**
- **Required fields**: Reject if invoice ID, vendor ID, amount, currency, exception source, or business unit is missing. Present the missing fields and ask the user to provide them.
- **Amount validation**: Verify that the invoice amount is a positive number. Flag zero or negative amounts for correction.
- **Currency validation**: Verify that the currency code is a recognized ISO 4217 code.

**Duplicate check:**
Search the exception tracker for existing cases matching the same invoice ID:
- `SearchM365(sources=["files"], query="AP exception tracker")` to locate the tracker
- `ReadFileContent` to read existing records
- If a case already exists for this invoice ID, present it to the user and ask whether to proceed with a new case or link to the existing one

**Generate Exception Case ID:**
Format: APX-YYYY-NNNNN (e.g., APX-2026-00142), incrementing from the last ID in the tracker.

**Write the exception case record** to the tracker with all extracted fields and:
- Status: "New"
- Created Date: current timestamp
- Priority: "Standard" (default; adjusted later by fin-control-path)
- Assigned To: blank (assigned later by fin-exception-routing)
- Classification: "Pending" (assigned later by fin-exception-classify)
- SLA target: 2 business days to resolution

**Present confirmation** via Adaptive Card (invoke `render-ui` skill first):
- Exception Case ID, Invoice ID, Vendor ID, Vendor Name
- PO ID (if available), Amount, Currency
- Exception Source, Business Unit
- Status: New
- SLA: 2 business days

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find supplier email thread related to the invoice |
| SearchM365 (files) | Find AP exception tracker, locate invoice documents in SharePoint |
| SearchM365 (connectors) | Pull invoice data from ERP or invoice capture platform if Graph Connector available |
| GetMessage | Read full supplier email content |
| ReadFileContent | Read exception tracker for duplicate check |
| SearchPeople / GetUserDetails | Resolve submitter identity |
| GetMyDetails | Current user for audit trail |

## Guardrails

- **Never create duplicate cases** for the same invoice ID — check the tracker first
- **Validate invoice amount** — reject zero or negative amounts
- **Reject entries missing required fields** — invoice ID, vendor ID, amount, currency, exception source, and business unit are mandatory
- **Confirm details with user** before writing to the exception tracker
- **Log creation metadata** — timestamp, created-by user, source channel, and invoice reference for audit traceability
- **Never release payments or modify vendor records** — this skill creates exception case records; payment actions are always human decisions
- **Preserve original exception source** — store the exception trigger exactly as reported by the source system
