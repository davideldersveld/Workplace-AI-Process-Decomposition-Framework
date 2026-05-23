---
name: ins-coverage-gap-detection
description: |
  Assesses initial coverage path considerations, detects missing evidence,
  identifies fraud indicator flags, and tracks regulatory deadlines for claims.
  Use when user asks to "check evidence gaps for [claim ID]",
  "what's missing on this claim", "coverage prep check",
  "evidence audit for claim [number]", "assess coverage path",
  "missing documents for [claim]", "fraud indicator check for [case]",
  or "regulatory deadline status for [claim ID]".
  Do NOT use for creating a new claim case (use ins-fnol-intake),
  assembling policy and claimant context (use ins-claim-context),
  classifying claim type or severity (use ins-claim-classifier),
  routing to adjusters or SIU (use ins-claim-routing),
  or drafting claimant or adjuster communications (use ins-claim-comms).
---

## Overview

Assesses initial coverage path considerations, detects missing evidence against the product-specific checklist, identifies fraud indicator flags from prior claim patterns and loss characteristics, and tracks jurisdiction-specific regulatory deadlines. Presents findings with evidence gap severity and coverage path considerations for adjuster review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents findings as recommendations via Adaptive Card. The adjuster confirms or overrides before any tracker update.

## When to Use

- A claim has been classified and needs evidence completeness assessment
- An adjuster wants to check what documents are missing before review
- A claim needs fraud indicator screening before routing
- Regulatory deadlines need to be tracked for a jurisdiction
- A claim needs coverage path preparation before adjuster assignment

## When NOT to Use

- Creating a new claim case — use ins-fnol-intake
- Assembling policy and claimant context — use ins-claim-context
- Classifying the claim type or severity — use ins-claim-classifier
- Routing to an adjuster, catastrophe desk, or SIU — use ins-claim-routing
- Drafting claimant, broker, or adjuster communications — use ins-claim-comms
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read claim data and evidence checklist", activeForm="Reading evidence inputs")
TaskCreate(subject="Assess coverage path and detect evidence gaps", activeForm="Detecting evidence gaps")
```

### Step 1: Read Assessment Inputs

**Read the claim case and classification:**
- `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent` — current case data including classification

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Claim ID]")` then `ReadFileContent`

**Read evidence checklist and handling standards:**
- `SearchM365(sources=["files"], query="evidence checklist [product line]")` then `ReadFileContent` — required documents by claim type and product line
- `SearchM365(sources=["files"], query="claims handling standards")` then `ReadFileContent` — handling procedures and quality standards

**Check evidence folder:**
- `GetDriveChildren` — list documents in the claim evidence folder to check what has been uploaded
- `SearchM365(sources=["files"], query="[Claim ID] photos OR police report OR estimate OR proof of loss")` — find uploaded evidence documents

**Read fraud indicators and regulatory requirements:**
- `SearchM365(sources=["files"], query="fraud trigger checklist")` then `ReadFileContent` — SIU referral trigger criteria
- `SearchM365(sources=["files"], query="[jurisdiction] claims requirements")` then `ReadFileContent` — state-specific evidence and timeline requirements
- `SearchM365(sources=["connectors"], connector_ids=["claims-platform-connector"])` — fraud indicator flags from claims platform if indexed

### Step 2: Assess Evidence Completeness

Compare required checklist items against uploaded documents in the claim's evidence folder:

**Evidence Gap Categories:**

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing proof of loss | Signed proof of loss form not submitted | High |
| Missing photos or documentation | Loss scene photos, damage photos, or supporting documentation not provided | High |
| Missing police report | Police report requested but not received (theft, vandalism, auto accident) | Medium |
| Missing repair estimate | Contractor or body shop estimate not submitted | Medium |
| Missing medical documentation | Medical records or bills not provided (bodily injury claims) | High |
| Missing witness statement | Witness statements referenced but not collected | Medium |
| Missing subrogation documentation | Third-party liability evidence not assembled | Low |

### Step 3: Assess Coverage Path Considerations

Identify coverage-relevant conditions for adjuster awareness — frame as considerations, never as determinations:

**Coverage path factors:**
- Policy status on date of loss (active, lapsed, canceled)
- Endorsement exclusions that may apply to the loss type
- Deductible amount relative to the claimed loss
- Named insured vs. additional insured vs. third-party claimant
- Jurisdiction-specific coverage requirements

**Important:** Frame all findings as "potential coverage path considerations for adjuster review" — never state that coverage exists or does not exist.

### Step 4: Screen for Fraud Indicators

Check for SIU trigger criteria from the fraud trigger checklist:

| Indicator | Description | Visibility |
|-----------|-------------|------------|
| Repeated claims | 3+ claims on the same policy within 24 months | Elevated |
| Same-provider pattern | Same repair vendor, medical provider, or attorney across multiple claims | Elevated |
| Conflicting narratives | Claimant statement conflicts with evidence or system records | Elevated |
| Recent policy change | Coverage increase or endorsement change within 90 days before loss | Flagged |
| Prior SIU referral | Claimant or policy has prior SIU investigation history | Elevated |
| Suspicious timing | Loss reported close to policy cancellation, renewal, or premium increase | Flagged |

### Step 5: Track Regulatory Deadlines

Identify jurisdiction-specific deadlines:

- State-mandated proof of loss submission timeline
- State-mandated acknowledgment deadline (time to acknowledge receipt of claim)
- State-mandated coverage determination deadline
- Mandatory disclosure language requirements
- Unfair claims practices act timeline requirements

### Step 6: Present Findings

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Claim ID, Claimant Name, Policy Number, Date of Loss, Classification
- **Evidence completeness** — percentage complete, list of received and missing items with severity
- **Coverage path considerations** — flagged conditions for adjuster awareness (with explicit "for adjuster review" framing)
- **Fraud indicator flags** — any triggered SIU criteria with elevated visibility (never labeled as fraudulent)
- **Regulatory deadlines** — jurisdiction-specific timelines and days remaining
- **SLA status** — time elapsed since intake, hours remaining in triage window
- **Recommended next steps** — request missing evidence, proceed to routing, escalate for review
- **Draft label** — "DRAFT ANALYSIS — coverage determination requires adjuster review"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find claims tracker, evidence checklists, fraud trigger checklist, regulatory requirements, handling standards |
| SearchM365 (connectors) | Retrieve fraud indicator flags from claims platform via Graph Connector |
| ReadFileContent | Read checklists, fraud triggers, regulatory requirements, context packet, tracker |
| GetDriveChildren | List documents in the claim evidence folder |

## Guardrails

- **Never mark an evidence item as complete** without document evidence confirmed in the SharePoint claim folder
- **Never state that coverage exists or does not exist** — frame all findings as "potential coverage path considerations for adjuster review"
- **All gap findings must include the statement** "DRAFT ANALYSIS — coverage determination requires adjuster review"
- **Flag fraud indicators separately with elevated visibility** but never label a claim as fraudulent — SIU determination is exclusively human-owned
- **Never auto-clear fraud indicators** — once flagged, only a human reviewer may dismiss a fraud indicator
- **Flag jurisdiction-specific evidence requirements** — state-mandated timelines are regulatory obligations, not operational preferences
- **Present findings for user review** before updating the Excel tracker — do not auto-update evidence status
- **Do not recommend reserve amounts or coverage decisions** — these are adjuster and supervisor responsibilities
- **Respect litigation hold flags** — if a claim is flagged with a litigation hold in the tracker, do not recommend deletion or modification of any evidence
- **Do not surface SIU investigation details** in outputs that may be visible to claimants or external parties — fraud screening results are internal only
