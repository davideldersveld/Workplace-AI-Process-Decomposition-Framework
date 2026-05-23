---
name: fin-exception-classify
description: |
  Classifies AP invoice exceptions by comparing the context packet against the
  exception taxonomy, assigning a dominant exception type with confidence score.
  Use when user asks to "classify this exception", "what type of exception is this",
  "triage invoice exception", "categorize AP case",
  "exception classification for [invoice]", "what's wrong with this invoice",
  "triage this AP exception", or "classify AP case [ID]".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  assembling matching context (use fin-match-context),
  assessing risk or control path (use fin-control-path),
  routing to action owner (use fin-exception-routing),
  drafting outreach or approval packets (use fin-ap-comms),
  or updating system status (use fin-status-update).
---

## Overview

Compares the invoice data in the context packet against PO, receipt, and vendor data to identify the dominant exception type. Applies the exception taxonomy rules from SharePoint, assigns a confidence score, flags missing information that prevents confident classification, and identifies secondary exception types for multi-issue cases.

This skill operates in "AI assist" mode — it presents the classification for AP analyst review via Adaptive Card. It does not update the tracker or take any action without user confirmation.

## When to Use

- A context packet has been assembled and the exception needs to be classified before routing
- The user wants to understand what type of exception an invoice has
- An AP analyst needs to triage an exception case

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Assembling matching context — use fin-match-context
- Assessing risk or determining the control path — use fin-control-path
- Routing to the action owner — use fin-exception-routing
- Drafting outreach or approval packets — use fin-ap-comms
- Updating case status — use fin-status-update

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read context packet and exception taxonomy", activeForm="Reading classification inputs")
TaskCreate(subject="Classify exception type and present findings", activeForm="Classifying exception")
```

### Step 1: Read Classification Inputs

Locate and read required inputs:

- **Exception case data** — from the tracker: `SearchM365(sources=["files"], query="AP exception tracker")` then `ReadFileContent`
- **Context packet** — from fin-match-context output: `SearchM365(sources=["files"], query="context packet [Case ID]")` or `SearchM365(sources=["files"], query="case packet [Invoice ID]")` then `ReadFileContent`
- **Exception taxonomy** — `SearchM365(sources=["files"], query="exception taxonomy")` or `SearchM365(sources=["files"], query="AP exception classification rules")` then `ReadFileContent`
- **Prior classified exceptions** — from the tracker, filtered for similar vendor, amount, or exception source patterns

### Step 2: Classify Exception Type

Compare invoice data against PO, receipt, and vendor data from the context packet. Apply the exception taxonomy rules to determine the dominant exception type.

**Exception categories:**

| Category | Description | Typical Evidence |
|----------|-------------|-----------------|
| Amount mismatch | Invoice amount differs from PO amount | PO line total vs. invoice line total; variance outside tolerance |
| Missing goods receipt | Goods receipt not posted or partial | PO shows open delivery; no receipt document in system |
| Missing or invalid PO | Invoice references no PO or an invalid PO number | PO not found in system; PO is closed or cancelled |
| Duplicate invoice candidate | Invoice matches an existing invoice by amount, vendor, and date | Prior invoice with same vendor, amount, and similar date in tracker or ERP |
| Vendor master mismatch | Invoice vendor details do not match vendor master record | Name, address, tax ID, or bank details differ between invoice and master |
| Incomplete supporting documentation | Required supporting documents are missing | Contract, delivery note, or service acceptance not attached |
| Tax or withholding discrepancy | Tax calculation or withholding amount does not match expected values | Tax rate, jurisdiction, or withholding code mismatch |

**Classification logic:**
- Compare each invoice field against its expected value from PO, receipt, and vendor data
- Identify all discrepancies and map each to an exception category
- Assign the **dominant exception type** — the primary issue blocking invoice processing
- Assign **secondary exception types** if multiple issues exist (flag as multi-issue)
- Calculate a **confidence score** based on data completeness and match clarity:
  - **High confidence (90%+)**: Clear evidence of the exception type with no ambiguity
  - **Medium confidence (70-89%)**: Reasonable classification but some data gaps or ambiguity
  - **Low confidence (below 70%)**: Insufficient data or conflicting signals; manual triage recommended

**Missing information flags:**
- Note any fields that were expected but not available in the context packet
- Flag cases where classification depends on data that could not be retrieved
- Identify what additional information would increase confidence

### Step 3: Present Classification Report

Present via Adaptive Card (invoke `render-ui` skill first):

**Exception Classification:**
- Dominant exception type with description
- Confidence score with rationale
- Secondary exception types (if multi-issue)

**Evidence Summary:**
- Key data points supporting the classification
- Specific discrepancies identified (e.g., "PO amount: $12,500; Invoice amount: $13,750 — variance of $1,250")

**Missing Information:**
- Data gaps that affect classification confidence
- Recommended actions to close gaps (e.g., "Request goods receipt confirmation from warehouse")

**Prior Pattern:**
- Similar exceptions for this vendor from prior periods
- Whether this appears to be a recurring issue

**Recommended Next Step:**
- Proceed to control path assessment (fin-control-path)
- Gather additional context first (fin-match-context)
- Flag for immediate manual review (multi-issue or low confidence)

After user confirms the classification, update the exception tracker with: Classification, Confidence Score, and Secondary Types (if any).

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, context packet, exception taxonomy, prior exceptions |
| ReadFileContent | Read taxonomy rules, context packet, tracker |
| SearchM365 (connectors) | Pull additional ERP data if needed for classification validation |

## Guardrails

- **Present classification for AP analyst review** before updating tracker — this is AI assist mode
- **Include confidence score and rationale** with every classification — analysts need to understand why a type was assigned
- **Flag multi-issue cases** for manual triage when multiple conflicting exception types are present
- **Never auto-classify cases above the materiality threshold** without analyst confirmation
- **Flag duplicate invoice candidates with high severity** — duplicate payments are a critical financial risk
- **Note missing data** that prevented confident classification rather than guessing
- **Cross-reference with prior exceptions** for this vendor — recurring patterns are important context for triage decisions
- **Log the classification** — record exception type, confidence score, rationale, and timestamp in the tracker after user confirmation
