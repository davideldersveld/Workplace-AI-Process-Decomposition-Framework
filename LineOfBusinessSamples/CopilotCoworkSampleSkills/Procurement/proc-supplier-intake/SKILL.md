---
name: proc-supplier-intake
description: |
  Normalizes inbound supplier onboarding requests into structured case
  records in the onboarding tracker.
  Use when user asks to "new supplier request",
  "onboard supplier [name]", "set up supplier case for [company]",
  "supplier onboarding for [company]",
  "create supplier case for [name]",
  "log supplier request from [requester]",
  or "intake new supplier [name]".
  Do NOT use for gathering supplier and policy context (use proc-context-packet),
  detecting missing items or risk indicators (use proc-gap-risk-detect),
  determining review path (use proc-review-routing),
  drafting outreach communications (use proc-supplier-comms),
  or summarizing the reviewer packet (use proc-reviewer-packet).
---

## Overview

Normalizes inbound supplier onboarding requests — from email submissions, Teams messages, intake forms, or direct requests — into structured case records in the shared Excel onboarding tracker. Validates required fields, checks for duplicate supplier records, resolves the assigned procurement analyst, and confirms the case details before writing to the tracker.

This skill operates in "Deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking without exercising judgment on supplier risk, review requirements, or approval paths.

## When to Use

- A new supplier request has been submitted and needs to be logged as an onboarding case
- A requester has sent a supplier onboarding request via email or Teams
- A procurement analyst needs to confirm whether a supplier already has an open onboarding case
- A category manager wants to initiate onboarding for a supplier identified during sourcing

## When NOT to Use

- Gathering supplier and policy context — use proc-context-packet
- Detecting missing documents or risk indicators — use proc-gap-risk-detect
- Determining the review path and assigning reviewers — use proc-review-routing
- Drafting outreach or follow-up communications — use proc-supplier-comms
- Summarizing the case for reviewer approval — use proc-reviewer-packet
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Validate supplier request and check for duplicates", activeForm="Validating supplier intake")
TaskCreate(subject="Create onboarding case record", activeForm="Creating supplier case")
```

### Step 1: Gather Supplier Request Inputs

Collect the following from the user or source artifact:

| Field | Required | Description |
|-------|----------|-------------|
| **Supplier company name** | Yes | Legal or trading name of the supplier |
| **Requester name or email** | Yes | The person requesting the supplier be onboarded |
| **Spend category** | Yes | The procurement category (IT services, professional services, manufacturing, facilities, etc.) |
| **Geography or region** | Yes | Supplier's primary operating geography |
| **Priority or urgency** | If available | Standard or expedited onboarding |
| **Estimated annual spend** | If available | Helps determine spend tier and approval path |
| **Requested services** | If available | What the supplier will provide |

If the request arrives via email or Teams, extract these fields from the message:
- `SearchM365(sources=["email"], query="[supplier name] supplier onboarding request")` — find the original request email
- `SearchM365(sources=["teams"], query="[supplier name] supplier onboarding")` — find the request in Teams

### Step 2: Resolve Requester and Analyst

- `SearchPeople` — resolve the requester's identity and department
- `GetUserDetails` — pull requester profile for department and manager context
- `SearchPeople` — resolve the procurement analyst assigned to this spend category or geography

### Step 3: Validate Required Fields

Check that all required fields are present and valid:

- **Supplier company name** — must be a recognizable business entity name
- **Requester** — must be a valid employee in the directory
- **Spend category** — must match the organization's approved category taxonomy
- **Geography** — must be a recognized operating region

If required fields are missing, present what was captured and identify the gaps for the user.

### Step 4: Check for Duplicate Suppliers

Search for existing onboarding cases and vendor master records:
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` — check for any open or recently completed case with the same supplier name
- `SearchM365(sources=["files"], query="[supplier name] supplier")` — search for existing vendor records or prior onboarding attempts
- If a Graph Connector is available: `SearchM365(sources=["connectors"], connector_ids=["vendor-master-connector"])` — check the vendor master for an existing supplier record

If a duplicate is found:
- Present the existing case or vendor record details
- Ask whether this is a new request, a reactivation, or a duplicate

### Step 5: Generate Case ID and Record

Create the onboarding case record with the following fields:

| Column | Value |
|--------|-------|
| **Case ID** | Auto-generated (PROC-YYYYMMDD-NNN format) |
| **Supplier Name** | From intake |
| **Requester** | From intake (resolved name and email) |
| **Spend Category** | From intake (validated against taxonomy) |
| **Geography** | From intake |
| **Priority** | Standard or Expedited |
| **Status** | New |
| **Created Date** | Current timestamp |
| **Assigned Analyst** | Resolved from category or geography assignment |
| **Risk Tier** | Pending assessment (set by proc-gap-risk-detect) |
| **Target Completion Date** | Created Date + 3 business days (SLA) |
| **Checklist Completion %** | 0% |

### Step 6: Present Confirmation

Present the case record via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Supplier Name, Requester, Created Date
- **Supplier details** — Spend Category, Geography, Estimated Spend (if available), Requested Services
- **Assignment** — Assigned Procurement Analyst
- **Duplicate check result** — No duplicate found, or existing record details
- **SLA target** — Target Completion Date (3 business days)
- **Next steps** — "Run proc-context-packet to gather supplier and policy context"

Confirm with the user before writing to the tracker.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find original supplier request emails |
| SearchM365 (teams) | Find supplier requests in Teams channels |
| SearchM365 (files) | Find and read the onboarding tracker for duplicate checking |
| SearchM365 (connectors) | Check vendor master for existing supplier records |
| ReadFileContent | Read the current onboarding tracker |
| SearchPeople | Resolve requester identity and analyst assignment |
| GetUserDetails | Pull requester profile and department context |

## Guardrails

- **Never create duplicate cases** for the same supplier name and requester combination — always check the tracker and vendor master first
- **Validate spend category against the approved taxonomy** — reject or flag unrecognized categories for procurement operations review
- **Confirm case details with the user** before writing to the onboarding tracker — intake is deterministic but the user must verify captured fields
- **Log case creation with actor and timestamp** in the tracker for audit trail
- **Never assign a risk tier or review path at intake** — these are determined by proc-gap-risk-detect and proc-review-routing respectively
- **Never auto-route or auto-assign reviewers at intake** — intake creates the case record; routing is a separate step
- **Include the estimated annual spend** if available — this drives spend tier determination downstream but is never fabricated
- **Never include bank account details or tax identifiers** in the case record — these are collected through secure channels during onboarding
- **Flag if the supplier geography is in a sanctions-sensitive region** — add a preliminary note for downstream risk detection but do not make a risk determination at intake
