---
name: ins-fnol-intake
description: |
  Normalizes first notice of loss events into structured claim case records
  in the claims tracker.
  Use when user asks to "new loss reported", "FNOL for [claimant name]",
  "set up claim for [policy number]", "loss notification from [broker/agent]",
  "new claim intake", "log FNOL for [claimant]",
  "intake loss report", "new property claim for [insured]",
  or "claim intake for [date of loss]".
  Do NOT use for assembling policy and claimant context (use ins-claim-context),
  classifying claim type or severity (use ins-claim-classifier),
  assessing coverage path or evidence gaps (use ins-coverage-gap-detection),
  routing to adjusters or SIU (use ins-claim-routing),
  or drafting claimant or adjuster communications (use ins-claim-comms).
---

## Overview

Converts inbound first notice of loss signals — phone transcripts, web portal submissions, email notifications, broker reports, and agent submissions — into normalized, validated claim case records in the shared Excel claims tracker. Validates required fields, checks for duplicate claims, resolves contact identities, and confirms details before writing.

This skill operates in "deterministic automation" mode — structured field extraction, date validation, and duplicate checking with no AI judgment. The user confirms all details via Adaptive Card before the tracker is updated.

## When to Use

- A new loss has been reported by phone, portal, email, broker, or agent
- A claimant or insured has filed a first notice of loss through any channel
- A broker or agent has submitted a loss notification on behalf of a policyholder
- An existing loss report needs to be logged as a new claim case

## When NOT to Use

- Assembling policy, claimant, and loss context — use ins-claim-context
- Classifying the claim type or severity — use ins-claim-classifier
- Assessing initial coverage path or missing evidence — use ins-coverage-gap-detection
- Routing to an adjuster, catastrophe desk, or SIU — use ins-claim-routing
- Drafting claimant, broker, or adjuster communications — use ins-claim-comms
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather FNOL event details", activeForm="Gathering loss details")
TaskCreate(subject="Validate and create claim case record", activeForm="Creating claim case")
```

### Step 1: Gather FNOL Event Details

Collect or extract the following from the user's request, loss notification, or broker report:

**Required fields:**
- **Claimant name** — the person reporting the loss or the named insured
- **Policy number** — the policy under which the loss is being reported
- **Date of loss** — when the loss event occurred
- **Loss type** — property damage, auto physical damage, bodily injury, liability, catastrophe, other
- **Loss description** — brief narrative of what happened
- **Reporting channel** — phone, portal, email, broker, agent

**Optional fields:**
- Loss location (address or geographic area)
- Jurisdiction (state)
- Product line (homeowners, commercial property, auto, general liability)
- Agent or broker name and contact
- Police report number
- Claimant phone number and email
- Photos or documents already submitted

**Resolve identities:**
- `SearchPeople` — resolve the reporting agent, broker, or intake specialist
- `GetUserDetails` — pull profile data for internal contacts

**Check for source signal:**
- `SearchM365(sources=["email"], query="loss notification [claimant name] OR [policy number]")` — locate the originating loss notification
- If a source email is found, extract key fields from it
- `ReadFileContent` — read attached FNOL forms or phone transcript summaries from SharePoint

### Step 2: Validate and Check Duplicates

**Read the claims tracker:**
- `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent`

**Validate required fields:**
- All required fields must be populated before writing
- Date of loss must not be in the future
- Policy number must be present

**Check for duplicates:**
- Search the tracker for existing claims with the same policy number and date of loss
- If a potential duplicate is found, present it to the user with details and ask whether to proceed, merge, or skip

**Calculate SLA deadline:**
- Standard FNOL cases: 4 business hours from intake timestamp
- Set the SLA Deadline field based on the intake time

### Step 3: Confirm and Write Case Record

Present the normalized claim case record via Adaptive Card (invoke `render-ui` skill first) for user confirmation:

- **Claim ID** — auto-generated (format: CLM-YYYY-NNNNN based on year and sequence)
- **Claimant Name** — as reported
- **Policy Number** — from the loss notification
- **Date of Loss** — as reported (validated not future-dated)
- **Loss Type** — initial categorization from intake
- **Severity** — "Pending Classification"
- **Reporting Channel** — source channel
- **Status** — "Intake Complete"
- **Created Date** — current timestamp
- **Assigned To** — blank (pending routing)
- **Jurisdiction** — state of loss if known
- **Product Line** — if identifiable from policy number
- **SLA Deadline** — calculated from intake time

After user confirmation, log the case creation with timestamp, creating user, and intake source for audit traceability.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find inbound loss notification or broker submission email |
| SearchM365 (files) | Find claims tracker, FNOL forms, phone transcripts |
| ReadFileContent | Read the tracker to check for duplicates; read attached forms |
| SearchPeople / GetUserDetails | Resolve agent, broker, and intake specialist identities |
| GetDriveChildren | Check existing tracker for duplicate entries |

## Guardrails

- **Never create duplicate claim cases** for the same policy number and date of loss without explicit user confirmation
- **Validate all mandatory fields** before writing — do not create incomplete case records
- **Date of loss must not be in the future** — flag and ask the user to verify if the date appears incorrect
- **Confirm details via Adaptive Card** before writing to the tracker
- **Never auto-populate coverage status or reserve amounts** — this skill creates claim records only; coverage and reserves are adjuster decisions
- **Never interpret or assess the loss event** — record factual intake information only; classification is handled by ins-claim-classifier
- **Log intake source and timestamp** for audit traceability — every claim record must have a complete creation audit entry
- **Include SLA deadline** in every claim record — the 4-business-hour triage window is operationally critical
- **Mask claimant SSN and financial data** if present in the source signal — these must not appear in the tracker
