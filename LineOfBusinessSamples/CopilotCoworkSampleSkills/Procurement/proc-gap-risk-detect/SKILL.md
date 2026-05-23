---
name: proc-gap-risk-detect
description: |
  Detects missing onboarding documents, checklist gaps, and risk
  indicators for a supplier onboarding case.
  Use when user asks to "check supplier gaps for [supplier]",
  "what's missing for [supplier] onboarding",
  "supplier risk check for [case]",
  "audit supplier case [ID]",
  "onboarding completeness check for [supplier]",
  "detect risk flags for [supplier]",
  or "review supplier checklist for [case]".
  Do NOT use for creating a new supplier case (use proc-supplier-intake),
  gathering supplier and policy context (use proc-context-packet),
  determining review path (use proc-review-routing),
  drafting outreach communications (use proc-supplier-comms),
  or summarizing the reviewer packet (use proc-reviewer-packet).
---

## Overview

Compares the required onboarding checklist against uploaded documents and available screening data to identify documentation gaps, approaching deadlines, and risk indicators. Flags sanctions or compliance risk with elevated visibility, detects duplicate supplier patterns, and identifies expired or insufficient insurance coverage. Presents findings for analyst review via Adaptive Card.

This skill operates in "AI assist" mode — it reads and analyzes onboarding data but only presents findings as recommendations. The procurement analyst reviews and confirms before any tracker updates are made.

## When to Use

- A supplier onboarding case has a context packet and needs completeness and risk review
- A procurement analyst wants to identify all documentation gaps before routing for review
- New documents have been uploaded and the checklist needs re-evaluation
- A case needs risk re-assessment after a category or geography change

## When NOT to Use

- Creating a new supplier case — use proc-supplier-intake
- Gathering supplier and policy context — use proc-context-packet
- Determining the review path and assigning reviewers — use proc-review-routing
- Drafting outreach or follow-up communications — use proc-supplier-comms
- Summarizing the case for reviewer approval — use proc-reviewer-packet
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read checklist and uploaded documents", activeForm="Checking supplier completeness")
TaskCreate(subject="Present gap and risk findings", activeForm="Detecting gaps and risk flags")
```

### Step 1: Read Onboarding Inputs

**Read the case data:**
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` — case status, spend category, geography, assigned analyst

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [supplier name]")` then `ReadFileContent` — required documents checklist, screening requirements, policy references

**Read the onboarding checklist:**
- `SearchM365(sources=["files"], query="supplier onboarding checklist")` then `ReadFileContent` — master checklist with required items by category and geography

### Step 2: Inventory Uploaded Documents

Check what has been provided:
- `GetDriveChildren` — list all files in the supplier's onboarding folder in SharePoint
- Match uploaded documents against the required checklist items
- Note document types, upload dates, and file names

For each required checklist item, determine status:

| Status | Definition |
|--------|------------|
| **Received** | Document is present in the onboarding folder and matches the requirement |
| **Pending** | Document has not been uploaded |
| **Expired** | Document is present but has passed its expiration date (insurance certs, certifications) |
| **Insufficient** | Document is present but does not meet minimum requirements (insurance below minimums) |
| **Not required** | Checklist item does not apply to this category or geography |

### Step 3: Check Screening Status

Evaluate screening and due diligence:

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["risk-screening-connector"])` — pull sanctions screening, debarment check, and financial health results

**Via SharePoint (if manual):**
- `SearchM365(sources=["files"], query="[supplier name] screening results")` then `ReadFileContent`

Assess screening status:
- Sanctions and debarment screening: completed / pending / flagged
- Conflict of interest: cleared / pending / flagged
- Financial health (if required): satisfactory / concerns / pending
- Cybersecurity assessment (if required): passed / pending / issues identified

### Step 4: Detect Risk Indicators

Flag risk indicators that require human review:

| Risk Category | Indicators | Severity |
|--------------|-----------|----------|
| **Sanctions or debarment** | Name match on OFAC, EU sanctions, or debarment lists | Critical — requires immediate human review |
| **Duplicate supplier** | Matching name, address, or tax ID with existing vendor master record | High — potential duplicate payment or fraud risk |
| **Expired insurance** | Insurance certificate past expiration date | High — supplier not covered for current work |
| **Insufficient coverage** | Insurance amounts below category minimums | Medium — requires waiver or updated certificate |
| **Missing critical documents** | W-9/W-8, banking form, or required contract not received | Medium — blocks onboarding completion |
| **Sanctions-sensitive geography** | Supplier operates in a country subject to comprehensive sanctions | High — requires enhanced due diligence |
| **High-spend tier** | Estimated annual spend exceeds elevated approval threshold | Medium — triggers additional review requirements |
| **Missing screening** | Required screening not yet completed | Medium — blocks review routing |

### Step 5: Calculate Completeness

Compute the checklist completion percentage:
- Count required items with "Received" status
- Divide by total required items (excluding "Not required")
- Calculate percentage

Assess SLA risk:
- Days elapsed since case creation versus 3 business day SLA target
- Flag if remaining time is insufficient to complete pending items

### Step 6: Present Gap and Risk Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Supplier Name, Spend Category, Geography, SLA Target Date
- **Checklist completion** — percentage complete with count (e.g., "7 of 10 items received")
- **Missing documents** — list of pending items with criticality level
- **Expired or insufficient items** — documents that need renewal or upgrade
- **Risk flags** — each flag with severity, evidence, and "requires human review" label
- **Screening status** — status of each required screening with completion date or pending flag
- **Duplicate supplier alert** — if a potential duplicate was detected, with existing record details
- **SLA risk** — days remaining versus items pending, with risk level
- **Recommended risk tier** — suggested tier based on findings (for analyst confirmation)
- **Draft label** — "GAP AND RISK REPORT — analyst review required before tracker update"

### Step 7: Record Findings (After Confirmation)

After the analyst confirms the findings:
- Update the onboarding tracker with: Checklist Completion %, Risk Tier, Risk Flags summary, Screening Status
- Do not update until the analyst has reviewed and confirmed

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, context packet, checklist, screening results, supplier documents |
| SearchM365 (connectors) | Pull screening results via Graph Connector |
| ReadFileContent | Read checklist, context packet, tracker, screening data |
| GetDriveChildren | Inventory uploaded documents in supplier onboarding folder |

## Guardrails

- **Never mark a checklist item as complete without document evidence** in the SharePoint folder — completeness is evidence-based, not self-reported
- **Flag sanctions, debarment, or compliance risk separately** with elevated visibility and explicit "requires human review" label — these are never auto-cleared
- **Present findings for analyst review** before updating the tracker — gap detection is a recommendation, not an automated update
- **Never suppress or downgrade a risk flag** — all flags must be visible to the analyst; the analyst decides disposition
- **Include confidence level** for risk indicators derived from document analysis versus screening system results — system-sourced flags are higher confidence than document-inferred flags
- **Never auto-clear a risk flag** — all sanctions, debarment, and compliance alerts require human review and documented disposition
- **Flag expired insurance and insufficient coverage separately** from missing documents — these require different remediation (renewal versus first submission)
- **Never fabricate screening results** — if screening has not been completed, report the status as "pending" rather than assuming clearance
- **Include the SLA target date** in every report — the analyst needs to prioritize based on remaining time
- **Never include bank account details or tax identifiers** in the gap report — reference the document status ("W-9 received" or "W-9 pending") without reproducing sensitive content
