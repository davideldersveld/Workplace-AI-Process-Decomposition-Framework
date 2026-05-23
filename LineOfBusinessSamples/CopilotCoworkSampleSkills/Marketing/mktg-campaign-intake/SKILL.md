---
name: mktg-campaign-intake
description: |
  Normalizes inbound campaign requests into structured case records
  in the campaign tracker.
  Use when user asks to "new campaign request", "log campaign intake for [name]",
  "set up campaign case", "campaign request from [stakeholder]",
  "create campaign for [product or event]",
  "intake for marketing campaign [name]",
  or "register new campaign request".
  Do NOT use for assembling brand and product context (use mktg-context-packet),
  extracting goals and constraints (use mktg-signal-extraction),
  drafting the campaign brief (use mktg-brief-draft),
  or routing for review (use mktg-review-routing).
---

## Overview

Normalizes inbound campaign requests — whether submitted via email, Teams, intake form, or direct conversation — into structured case records in the Excel campaign tracker on SharePoint. Validates required fields, checks for duplicate campaigns, flags campaigns requiring specialized review, and confirms the record with the analyst via Adaptive Card before writing.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, and duplicate checking without AI judgment on campaign strategy, messaging, or brief content.

## When to Use

- A new campaign request has been received via email, Teams, or direct submission
- A stakeholder has submitted a launch request, event request, or field marketing request
- Marketing operations needs to log a new campaign into the tracker

## When NOT to Use

- Assembling brand, product, and audience context — use mktg-context-packet
- Extracting goals, constraints, and dependencies — use mktg-signal-extraction
- Drafting the campaign brief — use mktg-brief-draft
- Routing for brand, product, or legal review — use mktg-review-routing
- Confirming brief baseline — this is always a human decision (MK-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read campaign request and validate inputs", activeForm="Reading campaign request")
TaskCreate(subject="Create campaign case in tracker", activeForm="Creating campaign case")
```

### Step 1: Capture Campaign Request

Identify the request source and extract case details:

**From email:**
- `SearchM365(sources=["email"], query="campaign request [campaign name or stakeholder]")` — find the original request thread
- Extract: campaign name, requesting stakeholder, campaign type, requested launch date, budget range, attached briefs or decks

**From Teams:**
- `SearchM365(sources=["teams"], query="campaign request [campaign name or stakeholder]")` — find the request message
- Extract the same fields from the Teams message context

**From direct intake (user provides details in the prompt):**
- Extract details from the user's message directly

**Resolve the requesting stakeholder:**
- `SearchPeople` — resolve stakeholder by name or email
- `GetUserDetails` — pull stakeholder profile, department, and business unit

### Step 2: Validate Required Fields

Every campaign case requires:

| Field | Source | Validation |
|-------|--------|------------|
| Campaign name | Request details | Must be a descriptive name identifying the campaign |
| Requesting stakeholder | Request source | Must resolve to a valid user in the directory |
| Department | Stakeholder profile or user input | Must match a recognized department or business unit |
| Campaign type | Request details | Must match a recognized type: Product Launch, Event, Demand Generation, Brand, Field Marketing, Content, Digital, or Other |
| Requested launch date | Request details | Must be a valid future date; flag if less than 3 business days from today (insufficient review time) |
| Budget range | Request details or default | Note if provided; flag if absent for campaigns above standard scope |

If any required field is missing, prompt the analyst for the missing information before proceeding.

### Step 3: Check for Duplicate Campaigns

Read the campaign tracker:
- `SearchM365(sources=["files"], query="campaign tracker")` then `ReadFileContent`
- Check for existing open campaigns with the same name or similar name and launch date
- If a potential duplicate is found, flag it for the analyst with the existing Campaign ID and creation date

### Step 4: Assess Campaign Priority

Assign initial priority based on request characteristics:

| Priority | Criteria |
|----------|----------|
| **High** | Executive-sponsored, revenue-critical launch, regulatory deadline, event with fixed date within 2 weeks |
| **Standard** | Normal business request with adequate lead time |
| **Low** | Internal communications, evergreen content, no fixed deadline |

### Step 5: Flag Specialized Review Requirements

Check whether the campaign type or content triggers specialized review:

- **Regulated industry claims** (healthcare, financial services) — flag for legal review
- **Executive communications** — flag for senior leadership review
- **New product or service launch** — flag for product marketing review
- **External-facing with customer data references** — flag for privacy review
- **Cross-regional or international** — flag for regional marketing review

### Step 6: Generate Campaign ID

Assign a sequential Campaign ID following the format: `MKT-YYYY-NNNN` where YYYY is the current year and NNNN is the next sequential number from the tracker.

### Step 7: Present Case Record for Confirmation

Present the proposed case record via Adaptive Card (invoke `render-ui` skill first):

- **Campaign ID** — generated identifier
- **Campaign name** — descriptive title
- **Requesting stakeholder** — name, email, department
- **Campaign type** — Product Launch, Event, Demand Gen, etc.
- **Requested launch date** — target date with lead time assessment
- **Budget range** — if provided; "Not specified" if absent
- **Priority** — High, Standard, or Low with rationale
- **Specialized review flags** — legal, product marketing, privacy, or regional reviews required
- **Status** — "Intake Complete" (initial status)
- **Duplicate check result** — "No duplicates found" or details of potential match
- **SLA target** — 3 business days for first review-ready brief

### Step 8: Write to Campaign Tracker (After Confirmation)

After analyst confirmation, write the case record to the Excel campaign tracker.

Record the following columns:
- Campaign ID, Campaign Name, Requester, Department, Campaign Type, Requested Launch Date, Budget Range, Status, Created Date, Assigned Campaign Manager (blank — assigned later), Priority, Review Status (blank — populated during routing)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the original campaign request email thread |
| SearchM365 (teams) | Find campaign request messages in Teams |
| SearchM365 (files) | Find the campaign tracker in SharePoint |
| ReadFileContent | Read the campaign tracker to check for duplicates and determine next Campaign ID |
| GetDriveChildren | Browse campaign workspace for attached request documents |
| SearchPeople | Resolve requesting stakeholder identity |
| GetUserDetails | Pull stakeholder profile, department, and business unit |

## Guardrails

- **Never create duplicate campaigns** for the same name and launch date — flag potential duplicates for analyst review
- **Validate all required fields** before writing to the tracker — prompt for missing information rather than writing incomplete records
- **Confirm the case record with the analyst via Adaptive Card** before writing to the tracker — intake writes are not auto-committed
- **Flag campaigns with insufficient lead time** — if the requested launch date is less than 3 business days away, warn that the standard SLA cannot be met
- **Flag campaigns requiring specialized review** — regulated claims, executive communications, new product launches, and cross-regional campaigns need additional reviewers
- **Never assess campaign strategy, messaging quality, or audience fit** during intake — those decisions belong to downstream skills
- **Never draft or generate brief content** during intake — intake captures metadata only
- **Include the SLA target** (3 business days for first review-ready brief) in every case record for tracking
- **Tag the intake source** (email, Teams, direct intake) in the case record for audit trail
