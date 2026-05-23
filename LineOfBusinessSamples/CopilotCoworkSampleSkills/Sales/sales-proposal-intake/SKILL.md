---
name: sales-proposal-intake
description: |
  Normalizes an RFP receipt, proposal request, or deal support request
  into a structured proposal case record in the shared tracker.
  Use when user asks to "new proposal request for [account]",
  "RFP received for [account]",
  "set up proposal case for [opportunity]",
  "log deal support request for [account]",
  "create proposal record for [RFP]",
  or "intake proposal for [deal]".
  Do NOT use for gathering account and content context (use sales-context-packet),
  extracting RFP requirements (use sales-rfp-extraction),
  mapping requirements to approved content (use sales-content-mapping),
  drafting the proposal response (use sales-proposal-draft),
  or routing for approval (use sales-approval-routing).
---

## Overview

Converts an incoming RFP receipt, proposal request email, or deal support request into a structured proposal case record in the shared Excel proposal tracker. Validates required fields, checks for duplicate proposals on the same opportunity, generates a unique Proposal ID, calculates the SLA deadline (3 business days to first draft), and flags deals requiring executive sponsor involvement based on deal size thresholds.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, duplicate checking, and SLA calculation without exercising AI judgment on deal strategy or pricing.

## When to Use

- An RFP has been received and needs to be logged as a proposal case
- A proposal request has been submitted by an account executive or sales leader
- A deal support request needs to be formalized with a proposal case record
- An existing opportunity needs a proposal case linked to it

## When NOT to Use

- Gathering account history, product context, or approved content — use sales-context-packet
- Extracting requirements from the RFP document — use sales-rfp-extraction
- Mapping requirements to approved content or identifying gaps — use sales-content-mapping
- Drafting the proposal response document or deck — use sales-proposal-draft
- Routing the proposal package for pricing, legal, or product approval — use sales-approval-routing
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read intake data and check for duplicates", activeForm="Processing proposal request")
TaskCreate(subject="Create proposal case record", activeForm="Creating proposal case")
```

### Step 1: Identify the Intake Signal

Determine the source of the proposal request:

| Source | Detection Method | Key Fields |
|--------|-----------------|------------|
| **RFP email** | `SearchM365(sources=["email"], query="RFP [account name]")` | Sender, account name, RFP title, due date, attachments |
| **Deal support request** | `SearchM365(sources=["email"], query="deal support request [account]")` or `SearchM365(sources=["teams"], query="proposal request [account]")` | Account executive, opportunity ID, deal context |
| **Direct request** | User provides details in the conversation | Account name, opportunity ID, due date, deal context |
| **CRM trigger** | `SearchM365(sources=["connectors"], connector_ids=["dynamics-connector"])` | Opportunity ID, stage, deal size, account tier |

### Step 2: Extract and Validate Required Fields

Extract these fields from the intake signal:

| Field | Source | Validation |
|-------|--------|-----------|
| **Account Name** | Email, CRM, or user input | Must match a known account — resolve via `SearchM365(sources=["connectors"])` or SharePoint account list |
| **Opportunity ID** | CRM or user input | Must be a valid opportunity ID if provided |
| **RFP Title** | RFP document filename or email subject | Required — prompt if missing |
| **Due Date** | RFP document, email body, or user input | Must be in the future; flag if less than 5 business days |
| **Account Executive** | `SearchPeople` — resolve from email sender or user input | Must resolve to a valid user in the directory |
| **Proposal Manager** | Assigned based on deal size and product area, or user input | Resolve via `SearchPeople` and `GetUserDetails` |
| **Deal Size** | CRM opportunity data or user input | Used for complexity rating and executive sponsor threshold |
| **Product Areas** | RFP document or user input | Used for downstream content mapping and approval routing |

### Step 3: Check for Duplicate Proposals

Read the proposal tracker to check for existing proposals on the same opportunity:

- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent`
- Check for matching Opportunity ID or matching Account Name plus RFP Title combination
- If a duplicate exists:
  - Surface the existing Proposal ID, status, and due date
  - Ask the user whether to link to the existing case, create a new case, or cancel

### Step 4: Generate Proposal ID

Format: `PROP-YYYYMMDD-SEQ` where:
- `YYYYMMDD` is the current date
- `SEQ` is a three-digit sequence number based on the count of proposals created on the same date in the tracker

### Step 5: Calculate SLA Deadline

- SLA target: 3 business days from intake to first draft response package
- Calculate the SLA deadline excluding weekends and standard US holidays
- Flag if the RFP due date is less than 5 business days away — this creates urgency for the entire proposal cycle

### Step 6: Assess Complexity Rating

| Rating | Criteria |
|--------|---------|
| **High** | Deal size above executive threshold, multi-product solution, custom terms required, competitive displacement, or RFP exceeds 50 requirements |
| **Medium** | Standard deal size, single product area, some custom requirements, or RFP with 20–50 requirements |
| **Low** | Standard deal, standard product, template-friendly requirements, or RFP with fewer than 20 requirements |

### Step 7: Present Case Record for Confirmation

Present the proposed case record via Adaptive Card (invoke `render-ui` skill first):

- **Proposal ID** — generated ID
- **Account Name** and **Opportunity ID**
- **RFP Title**
- **Due Date** — with urgency flag if tight timeline
- **Account Executive** and **Proposal Manager**
- **Deal Size** and **Complexity Rating**
- **SLA Deadline** — 3 business day target date
- **Executive sponsor required** — yes/no based on deal size threshold
- **Duplicate check result** — no duplicates found, or link to existing case

### Step 8: Write to Proposal Tracker (After Confirmation)

After the user confirms, write the case record to the Excel proposal tracker:

- Proposal ID, Account Name, Opportunity ID, RFP Title, Due Date
- Account Executive, Proposal Manager, Status ("Intake Complete")
- Created Date, Deal Size, Complexity Rating, SLA Deadline

### Step 9: Locate and Store RFP Document

- If an RFP document was attached to the intake email or is in SharePoint, note its location
- If no RFP document is found, flag as "RFP document pending — required for requirement extraction"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the RFP receipt email or deal support request thread |
| SearchM365 (teams) | Find deal support discussions in Teams channels or chats |
| SearchM365 (files) | Find the proposal tracker, RFP documents, account data |
| SearchM365 (connectors) | Pull CRM opportunity data via Graph Connector |
| ReadFileContent | Read the proposal tracker, intake forms, RFP documents |
| GetDriveChildren | Check for existing proposal folders and RFP attachments |
| SearchPeople | Resolve account executive, proposal manager, deal team members |
| GetUserDetails | Verify team member profiles and roles |
| render_ui (Adaptive Card) | Present the case record for confirmation |

## Guardrails

- **Never create duplicate cases** for the same opportunity and due date — always check the tracker first
- **Validate that the due date is in the future** — reject intake for past-due RFPs and surface the issue
- **Confirm all case details with the user** before writing to the proposal tracker — intake normalization is deterministic but the source data may be ambiguous
- **Flag deals exceeding the executive sponsor threshold** — these require additional oversight in downstream approval routing
- **Flag tight timelines** — if the RFP due date is less than 5 business days away, surface the urgency prominently
- **Never populate pricing or deal terms** in the intake record — deal size is captured for routing purposes only
- **Include the Proposal ID in every output** for traceability across the proposal lifecycle
- **Preserve the original intake signal** — note the source email, Teams message, or request that triggered the case
