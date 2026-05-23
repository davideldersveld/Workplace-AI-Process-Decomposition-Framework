---
name: sec-alert-intake
description: |
  Normalizes inbound security alerts into structured case records in the
  alert tracker, validates required fields, checks for duplicate or correlated
  alerts, and calculates SLA targets.
  Use when user asks to "new security alert from [source]",
  "log security case for [entity]", "intake this alert",
  "normalize this security event", "create security case",
  "new alert for [user/host]", or "SIEM alert intake".
  Do NOT use for enriching alert context (use sec-enrichment-packet),
  classifying alert type or risk (use sec-risk-classifier),
  assessing severity (use sec-severity-recommend),
  routing to analysts (use sec-analyst-router),
  or drafting investigation summaries (use sec-investigation-drafter).
---

## Overview

Normalizes inbound security alerts from SIEM, endpoint, identity, email security, or analyst intake into structured case records in the shared Excel alert tracker. Validates required fields, checks for duplicate or correlated alerts within a configurable time window, auto-generates Case IDs, calculates SLA targets, and confirms the normalized case with the analyst before writing.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking with no AI judgment on security decisions.

## When to Use

- A new security alert has been generated from a SIEM, endpoint platform, identity platform, or email security system
- An analyst has reported a security event that needs to be logged as a case
- A phishing report or user-reported security concern needs to be formalized into a case record
- An alert from a SOC Teams channel needs to be captured in the tracker

## When NOT to Use

- Enriching an existing alert with entity, asset, or threat context — use sec-enrichment-packet
- Classifying the alert type or likely risk — use sec-risk-classifier
- Assessing severity or investigation path — use sec-severity-recommend
- Routing to an analyst queue or SOC team — use sec-analyst-router
- Drafting investigation summaries or evidence requests — use sec-investigation-drafter
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read alert details and validate fields", activeForm="Reading alert details")
TaskCreate(subject="Create case record in alert tracker", activeForm="Creating case record")
```

### Step 1: Gather Alert Details

Collect required fields from the user's input or source system:

| Field | Required | Source |
|-------|----------|--------|
| Alert source | Yes | SIEM, endpoint, identity, email security, analyst report |
| Detection rule or signature name | Yes | Alert telemetry or analyst input |
| Affected entity | Yes | User, host, IP address, or application |
| Entity type | Yes | user, host, ip, application |
| Alert description | Yes | Raw alert summary or analyst description |
| Date and time of detection | Yes | Alert timestamp or current time |
| Reporting analyst | Yes | Current user or specified analyst |

If the alert involves a named user, resolve their identity:
- `SearchPeople` — find the affected user in the directory
- `GetUserDetails` — pull department, role, and location for the case record

### Step 2: Read Alert Tracker and Check for Duplicates

Locate and read the alert tracker:
- `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent`

Check for duplicate or correlated alerts:
- Search the tracker for the same detection rule and affected entity within a 1-hour window
- If a match is found, flag as **correlated** and recommend linking to the existing case rather than creating a new one
- Present the existing case details to the analyst for confirmation

### Step 3: Read Intake Template

- `SearchM365(sources=["files"], query="security alert intake template")` then `ReadFileContent` — field normalization rules and Case ID format

### Step 4: Generate Case Record

Auto-generate a Case ID using the format: `SEC-[YYYYMMDD]-[SOURCE]-[SEQ]` (for example, SEC-20260523-SIEM-001).

Build the normalized case record:

| Tracker Column | Value |
|----------------|-------|
| Case ID | Auto-generated |
| Alert Source | SIEM, endpoint, identity, email security, or analyst report |
| Detection Rule | Rule name or signature |
| Affected Entity | User, host, IP, or application identifier |
| Entity Type | user, host, ip, application |
| Alert Description | Normalized summary |
| Alert Category | Pending (set by sec-risk-classifier) |
| Severity | Pending (set by sec-severity-recommend) |
| Status | New |
| Created Date | Current timestamp |
| Assigned Analyst | Pending (set by sec-analyst-router) |
| Investigation Path | Pending (set by sec-severity-recommend) |
| SLA Target | Calculated based on default priority rules |
| Reporting Analyst | Name of the analyst who reported or initiated intake |
| Correlation Flag | Linked case ID if correlated, or none |

### Step 5: Confirm with Analyst

Present the normalized case via Adaptive Card (invoke `render-ui` skill first):

- **Case ID** and alert source
- **Affected entity** and entity type (never include raw IOCs such as IP addresses, hashes, or domain names in the card — reference by Case ID only)
- **Detection rule** name
- **Alert description** summary
- **Duplicate check result** — clean or correlated with existing case
- **SLA target** based on default priority
- **Confirmation request** — analyst confirms before the record is written to the tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find alert tracker, intake template |
| ReadFileContent | Read tracker, intake template |
| SearchPeople | Resolve affected user identity |
| GetUserDetails | Pull affected user profile, department, role |

## Guardrails

- **Never create duplicate cases** for the same detection rule and affected entity within a 1-hour window — flag as correlated and link to the existing case instead
- **Validate that all required fields are present** before writing to the tracker — alert source, affected entity, detection rule, and alert description are mandatory
- **Auto-generate Case ID** using the date and source prefix format — never accept user-specified Case IDs
- **Confirm with the analyst** via Adaptive Card before writing to the tracker — no silent writes
- **Never include raw IOCs** (IP addresses, hashes, domain names) in the Adaptive Card confirmation — reference them by Case ID and alert description only
- **Never set alert category or severity** during intake — these are pending fields set by downstream classification and severity skills
- **Never assign an analyst** during intake — assignment is handled by sec-analyst-router
- **Log the reporting analyst, timestamp, and alert source** for every case record as the creation audit trail
- **If the alert involves privileged accounts or executive users**, flag as elevated in the case record for downstream handling
- **Never post case details to general Teams channels** during intake — case confirmation is between the skill and the reporting analyst only
