---
name: itsm-incident-intake
description: |
  Normalizes inbound incident signals into structured incident records in
  the incident tracker, validates required fields, checks for duplicate
  or related incidents, and calculates SLA targets.
  Use when user asks to "new incident from [source]",
  "log incident for [user]", "ticket came in for [service]",
  "create incident case", "intake this incident",
  "new ticket from [channel]", or "incident intake for [issue]".
  Do NOT use for enriching incident context (use itsm-context-packet),
  classifying incident type (use itsm-classification-assist),
  assessing severity (use itsm-severity-recommend),
  routing to resolver groups (use itsm-assignment-router),
  or drafting user updates (use itsm-comms-drafter).
---

## Overview

Normalizes inbound incident signals from email, Teams chat, phone, monitoring alerts, or analyst intake into structured incident records in the shared Excel incident tracker. Validates required fields, checks for duplicate or related incidents within a configurable time window, auto-generates Incident IDs, calculates SLA targets, and confirms the normalized incident with the analyst before writing.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking with no AI judgment on incident severity or classification.

## When to Use

- A new incident has been reported via email, Teams message, phone, or monitoring alert
- An analyst needs to formalize a user-reported issue into a tracked incident record
- A monitoring system has generated a ticket that needs to be captured in the tracker
- An escalation from the service desk Teams channel needs to be logged

## When NOT to Use

- Enriching an existing incident with service, asset, or change context — use itsm-context-packet
- Classifying the incident type or affected service — use itsm-classification-assist
- Assessing severity or escalation path — use itsm-severity-recommend
- Routing to a resolver group — use itsm-assignment-router
- Drafting user updates or handoff summaries — use itsm-comms-drafter
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident details and validate fields", activeForm="Reading incident details")
TaskCreate(subject="Create incident record in tracker", activeForm="Creating incident record")
```

### Step 1: Gather Incident Details

Collect required fields from the user's input or source system:

| Field | Required | Source |
|-------|----------|--------|
| Reported by | Yes | User name or email from the report |
| Report channel | Yes | Email, chat, phone, monitoring, or walk-up |
| Symptom description | Yes | User-reported symptoms or monitoring alert summary |
| Affected service area | Yes | Service or application area experiencing the issue |
| Date and time reported | Yes | Timestamp of the report or current time |
| Intake analyst | Yes | Current user or specified analyst |

If the reporter is a named user, resolve their identity:
- `SearchPeople` — find the affected user in the directory
- `GetUserDetails` — pull department, location, role, and VIP status for the incident record

If the incident was reported via email:
- `SearchM365(sources=["email"], query="[symptom keywords] from:[reporter]")` — find the original report thread for reference

If the incident was reported via Teams:
- `SearchM365(sources=["teams"], query="[symptom keywords]")` — find the escalation message in the service desk channel

### Step 2: Read Incident Tracker and Check for Duplicates

Locate and read the incident tracker:
- `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent`

Check for duplicate or related incidents:
- Search the tracker for the same reporter and similar symptom description within a 4-hour window
- Search for open incidents affecting the same service area
- If a match is found, flag as **potentially related** and recommend linking to the existing incident rather than creating a new one
- Present the existing incident details to the analyst for confirmation

### Step 3: Read Intake Template

- `SearchM365(sources=["files"], query="incident intake template")` then `ReadFileContent` — field normalization rules and Incident ID format

### Step 4: Generate Incident Record

Auto-generate an Incident ID using the format: `INC-[YYYYMMDD]-[SEQ]` (for example, INC-20260523-001).

Build the normalized incident record:

| Tracker Column | Value |
|----------------|-------|
| Incident ID | Auto-generated |
| Reported By | User name and email |
| Report Channel | Email, chat, phone, monitoring, or walk-up |
| Summary | Normalized symptom description |
| Affected Service | Service area from report (pending classification confirmation) |
| Category | Pending (set by itsm-classification-assist) |
| Priority | Pending (set by itsm-severity-recommend) |
| Status | New |
| Created Date | Current timestamp |
| Assigned To | Pending (set by itsm-assignment-router) |
| SLA Target | Default based on standard priority until severity is assessed |
| Intake Analyst | Name of the analyst who performed intake |
| Duplicate Flag | Linked incident ID if related, or none |
| Source Tag | System-generated, manual entry, or email-sourced |

### Step 5: Confirm with Analyst

Present the normalized incident via Adaptive Card (invoke `render-ui` skill first):

- **Incident ID** and report channel
- **Reported by** — user name, department, location
- **Symptom summary** — normalized description
- **Affected service area** — as reported (pending classification)
- **Duplicate check result** — clean or potentially related to existing incident
- **SLA target** — default target based on standard priority
- **Confirmation request** — analyst confirms before the record is written to the tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find incident tracker, intake template |
| SearchM365 (email) | Find original report thread if submitted via email |
| SearchM365 (teams) | Find escalation messages in service desk channel |
| ReadFileContent | Read tracker, intake template |
| SearchPeople | Resolve affected user identity |
| GetUserDetails | Pull user profile, department, location, VIP status |

## Guardrails

- **Never create duplicate incidents** for the same reporter and similar symptoms within a 4-hour window — flag as potentially related and recommend linking instead
- **Validate that all required fields are present** before writing to the tracker — reporter, symptom description, and affected service area are mandatory
- **Auto-generate Incident ID** using the date-based sequence format — never accept user-specified Incident IDs
- **Confirm with the analyst** via Adaptive Card before writing to the tracker — no silent writes
- **Never set incident category or priority** during intake — these are pending fields set by downstream classification and severity skills
- **Never assign a resolver group** during intake — assignment is handled by itsm-assignment-router
- **Log the intake analyst, timestamp, report channel, and source tag** for every incident record as the creation audit trail
- **If the reporter is flagged as VIP or executive**, note this in the incident record for downstream severity assessment
- **The ITSM platform is the source of truth** — the Excel tracker is a working copy; note this in any output where state consistency matters
- **Never include raw infrastructure details** (IP addresses, hostnames, internal system names) in the Adaptive Card confirmation — use service area names only
