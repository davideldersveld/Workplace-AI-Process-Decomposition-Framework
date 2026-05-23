---
name: ps-escalation-intake
description: |
  Normalizes escalated support cases into structured escalation records
  in the product support escalation tracker.
  Use when user asks to "new escalation for [case]",
  "escalated case from [agent]",
  "product issue for [customer]",
  "log escalation for case [ID]",
  "suspected defect from support for [issue]",
  or "create escalation for [case ID]".
  Do NOT use for gathering evidence and context (use ps-evidence-packet),
  classifying the issue or defect path (use ps-defect-classifier),
  assessing customer impact or severity (use ps-impact-assessment),
  routing to engineering (use ps-engineering-routing),
  or drafting handoff communications (use ps-handoff-drafter).
---

## Overview

Normalizes escalated support cases from email, Teams, support platform submissions, and direct escalation into a structured escalation record in the shared Excel escalation tracker. Validates required fields, checks for duplicate escalations against the same originating case, generates a unique Escalation ID, calculates the 4-hour SLA deadline, and presents the normalized record for confirmation via Adaptive Card before writing.

This skill operates in "deterministic automation" mode — it applies stable field extraction and validation rules without exercising judgment on severity, defect classification, or engineering routing.

## When to Use

- A support case has been escalated beyond frontline resolution and needs to be captured as a structured escalation record
- An escalation engineer or support analyst wants to log a new suspected product defect into the escalation tracker
- A support case escalation email has arrived and needs to be processed into the tracking system
- A batch of escalated cases needs intake processing

## When NOT to Use

- Gathering case evidence, logs, and telemetry context — use ps-evidence-packet
- Classifying the issue type or suspected defect path — use ps-defect-classifier
- Assessing customer impact or recommending severity — use ps-impact-assessment
- Routing the escalation to an engineering owner or queue — use ps-engineering-routing
- Drafting the engineering handoff or customer update — use ps-handoff-drafter
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Extract and normalize escalation fields", activeForm="Processing escalation")
TaskCreate(subject="Write escalation record to tracker", activeForm="Creating escalation record")
```

### Step 1: Identify the Escalation Source

Determine where the escalation originated:

| Source Channel | How to Find It |
|---------------|---------------|
| **Escalation email** | `SearchM365(sources=["email"])` — find the escalation email thread by sender, case ID, or customer name |
| **Teams message** | `SearchM365(sources=["teams"])` — find the escalation message in support or triage channels |
| **Support platform** | `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — pull the originating support case record via Graph Connector |
| **Direct submission** | User provides escalation details directly in the conversation |

### Step 2: Extract and Normalize Fields

Extract the following fields from the escalation source:

| Field | Required | Source |
|-------|----------|--------|
| **Originating Case ID** | Yes | Support case reference number |
| **Customer Name** | Yes | Customer or account name from the case |
| **Account ID** | If available | Account identifier from CRM or support platform |
| **Escalating Agent** | Yes | Support agent who escalated the case |
| **Issue Summary** | Yes | Description of the problem — preserve the escalating agent's words |
| **Product Area** | If identifiable | Which product area the issue relates to |
| **Suspected Defect Path** | If identifiable | Initial hypothesis about the defect area |
| **Attached Evidence** | If available | Logs, screenshots, HAR files, reproduction notes |

**Resolve identities:**
- `SearchPeople` — resolve the escalating agent and account owner
- `GetUserDetails` — pull profile data for the escalating agent and customer contact

### Step 3: Generate Escalation ID

Generate a unique Escalation ID using the format: `ESC-[YYYYMMDD]-[SEQ]`

- Date: current date
- Sequence: three-digit sequential number (001, 002, etc.)

### Step 4: Check for Duplicate Escalations

Before creating the record, check for existing entries:

- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — scan the tracker for entries with the same Originating Case ID
- Flag if the originating case already has an open escalation
- Flag if the originating case has an active incident link (may be part of a broader issue)

### Step 5: Calculate SLA Deadline

Calculate the 4-business-hour SLA deadline from creation time:

- Standard escalations: 4 business hours to engineering-ready packet
- If the escalation is created outside business hours, the SLA clock starts at the next business day open

### Step 6: Prepare the Escalation Record

Assemble the normalized record with these fields:

| Tracker Column | Value |
|---------------|-------|
| Escalation ID | Generated ID |
| Originating Case ID | Support case reference |
| Customer Name | Resolved customer name |
| Account ID | Account identifier (or blank) |
| Product Area | Identified area (or "Unassigned") |
| Issue Summary | Normalized summary |
| Suspected Defect Path | Initial hypothesis (or "Pending Classification") |
| Severity | Pending (set by ps-impact-assessment) |
| Status | "Intake — Pending Evidence" |
| Created Date | Current timestamp |
| SLA Deadline | Calculated 4-hour deadline |
| Assigned To | Pending (set by ps-engineering-routing) |
| Engineering Queue | Pending (set by ps-engineering-routing) |
| Evidence Completeness | Pending (set by ps-evidence-packet) |
| Resolution | Open |

### Step 7: Present for Confirmation

Present the normalized record via Adaptive Card (invoke `render-ui` skill first):

- **Escalation ID** — generated ID
- **Originating Case** — case ID and source
- **Customer** — name and account
- **Product Area** — identified area
- **Issue Summary** — normalized description
- **SLA Deadline** — calculated deadline with countdown
- **Duplicate check** — results of the duplicate scan
- **Status** — "Intake — Pending Evidence"
- **Confirmation prompt** — "Confirm to write this escalation to the tracker"

### Step 8: Write to Tracker (After Confirmation)

After the user confirms:
- Write the record as a new row in the Excel escalation tracker
- Create a per-escalation evidence folder in the SharePoint escalation library (for logs, screenshots, and reproduction notes)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the escalation email thread |
| SearchM365 (teams) | Find escalation messages in support channels |
| SearchM365 (connectors) | Pull originating support case record via Graph Connector |
| SearchM365 (files) | Find the escalation tracker |
| ReadFileContent | Read the escalation tracker for duplicate checking |
| SearchPeople | Resolve escalating agent and account owner |
| GetUserDetails | Pull profile data for support contacts |

## Guardrails

- **Never create duplicate escalations** for the same originating case ID — flag existing escalations and present the conflict for the user to resolve
- **Validate that all required fields are populated** before writing — Originating Case ID, Customer Name, Escalating Agent, and Issue Summary are mandatory
- **Preserve the escalating agent's original description** — normalization summarizes but does not replace the original issue text
- **Never assign severity at intake** — severity is determined by ps-impact-assessment after evidence gathering and classification
- **Never assign an engineering owner at intake** — routing is handled by ps-engineering-routing after classification and impact assessment
- **Present the normalized record for confirmation** before writing to the tracker — the user reviews field accuracy before the record is created
- **Auto-calculate the SLA deadline** based on 4 business hours — display the countdown prominently
- **Flag if the originating case has an active incident link** — the escalation may be part of a broader outage
- **Never interpret an escalation as confirming a defect** — the intake record captures a suspected issue, not a confirmed bug
- **If the escalation contains multiple distinct issues**, flag for the user to decide whether to create one record or separate records — do not silently split or merge
