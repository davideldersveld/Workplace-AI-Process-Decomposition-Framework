---
name: fin-match-context
description: |
  Assembles PO, goods receipt, vendor profile, policy extracts, and prior exception
  history into a reviewer context packet for an AP invoice exception case.
  Use when user asks to "build context packet for invoice", "gather matching data",
  "assemble AP case packet", "what context do we have for [invoice]",
  "pull matching data for exception", "evidence packet for AP case",
  "context for invoice [number]", or "case packet for [exception ID]".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  classifying exception type (use fin-exception-classify),
  assessing risk or control path (use fin-control-path),
  routing to action owner (use fin-exception-routing),
  drafting outreach or approval packets (use fin-ap-comms),
  or updating system status (use fin-status-update).
---

## Overview

Assembles all matching context needed for AP invoice exception review into a single Word document — the case context packet. Gathers invoice details, purchase order data, goods receipt status, vendor profile, relevant AP policy extracts, prior exception history for this vendor, approval thresholds, and key contacts from M365 and federated data sources.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources and assembles it without interpreting policy or making accounting judgments. It does not modify any source records.

## When to Use

- An exception case has been created and needs matching context assembled before triage
- The user wants to understand what supporting data exists for an invoice exception
- An AP analyst needs a complete case packet before classification or routing

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Classifying the exception type — use fin-exception-classify
- Assessing risk or determining the control path — use fin-control-path
- Routing to the action owner — use fin-exception-routing
- Drafting outreach or approval packets — use fin-ap-comms
- Updating case status — use fin-status-update

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read exception case and locate context sources", activeForm="Reading exception case data")
TaskCreate(subject="Assemble case context packet", activeForm="Assembling context packet")
```

### Step 1: Read Exception Case Record

Locate and read the exception case:

- `SearchM365(sources=["files"], query="AP exception tracker")` to find the tracker
- `ReadFileContent` to read the specific exception case row
- Extract: Exception Case ID, Invoice ID, Vendor ID, Vendor Name, PO ID, Amount, Currency, Exception Source, Business Unit

### Step 2: Gather Context from All Sources

Retrieve context from each source in parallel where possible:

**Invoice details:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` for full invoice header and line data from ERP
- `SearchM365(sources=["files"], query="invoice [Invoice ID]")` for invoice documents in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["invoice-capture-connector"])` for invoice capture platform data
- Collect: invoice header fields, line items, tax details, payment terms, submission date

**Purchase order data:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` for PO details
- `SearchM365(sources=["files"], query="purchase order [PO ID]")` for PO documents in SharePoint
- Collect: PO header, line items, amounts, delivery terms, receiving status, approval history

**Goods receipt status:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` for receipt postings
- `SearchM365(sources=["files"], query="goods receipt [PO ID]")` for receipt documents
- Collect: receipt date, quantities, quality inspection status, discrepancies

**Vendor profile:**
- `SearchM365(sources=["files"], query="vendor profile [Vendor ID]")` or `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` for vendor master data
- Collect: vendor name, payment terms, currency, tax status, compliance status, prior payment history summary

**AP policies and thresholds:**
- `SearchM365(sources=["files"], query="AP policy invoice exception")` for exception handling policies
- `SearchM365(sources=["files"], query="approval matrix AP")` or `SearchM365(sources=["files"], query="AP approval threshold")` for threshold tables
- `SearchM365(sources=["files"], query="exception taxonomy")` for classification rules
- Collect: applicable policy sections, materiality thresholds, approval requirements, exception taxonomy

**Prior exception history:**
- `SearchM365(sources=["files"], query="AP exception tracker")` and `ReadFileContent` to find prior cases for this vendor
- Collect: prior exception cases for this vendor with types, amounts, outcomes, and resolution times

**Key contacts:**
- `SearchPeople(query="<buyer name>")` to resolve the procurement buyer on the PO
- `SearchPeople(query="cost center owner [Business Unit]")` to resolve the cost center owner
- `SearchPeople(query="AP manager")` for the AP operations manager
- `GetManagerDetails(user_id="<analyst>")` for reporting chain
- Collect: buyer, cost center owner, AP manager, AP analyst, vendor contact (from email if available)

### Step 3: Produce Case Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Exception Case Summary** — Case ID, Invoice ID, Vendor ID, Vendor Name, PO ID, Amount, Currency, Exception Source, Business Unit, Created Date
2. **Invoice Details** — header fields, line items, tax, payment terms, submission date; flag any fields that could not be retrieved
3. **Purchase Order Data** — PO header, line items, amounts, delivery terms, approval history; flag if PO is missing or closed
4. **Goods Receipt Status** — receipt date, quantities, discrepancies; flag if receipt is missing or partial
5. **Vendor Profile** — payment terms, compliance status, prior payment history summary; flag any vendor master concerns
6. **AP Policy Context** — applicable policy sections with document reference and version date; materiality thresholds; exception taxonomy classification rules
7. **Prior Exception History** — prior cases for this vendor with types, amounts, and outcomes; note patterns or recurring issues
8. **Key Contacts** — buyer, cost center owner, AP manager, AP analyst
9. **Missing Context Flags** — any critical data that could not be retrieved, with source system and reason

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, AP policies, approval matrices, PO documents, vendor profiles, invoice documents |
| SearchM365 (connectors) | Pull invoice data, PO data, receipt data, vendor master from ERP or invoice capture platform |
| ReadFileContent | Read tracker, policies, threshold tables, supporting documents |
| GetDriveChildren | List files in the exception case evidence folder |
| SearchPeople / GetUserDetails | Resolve buyer, cost center owner, AP manager |
| GetManagerDetails | Reporting chain for escalation |

## Guardrails

- **Cite source system and retrieval date** for every data element — ERP extract date, document version, last sync timestamp
- **Flag missing critical context** — if PO, receipt, or vendor profile cannot be found, flag prominently in the packet
- **Never include bank account details** or sensitive vendor financial data in the context packet
- **Mark data freshness** — flag any context older than 5 business days as potentially stale
- **Never include draft or unposted data** without clearly labeling it as preliminary
- **Operate read-only** — never update the ERP, invoice capture platform, or exception tracker from this skill
- **Record assembly metadata** — note which sources were queried, what was found, and what was missing
