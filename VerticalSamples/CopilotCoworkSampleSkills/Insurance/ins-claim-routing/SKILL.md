---
name: ins-claim-routing
description: |
  Routes claims to the correct adjuster, catastrophe desk, bodily injury team,
  or SIU based on the routing matrix, classification, and risk assessment.
  Use when user asks to "route this claim", "assign adjuster for [claim ID]",
  "where should this claim go", "recommend handling lane",
  "claim routing recommendation", "who handles this claim type",
  "assign claim [ID] to the right team", or "escalate claim to SIU".
  Do NOT use for creating a new claim case (use ins-fnol-intake),
  assembling policy and claimant context (use ins-claim-context),
  classifying claim type or severity (use ins-claim-classifier),
  assessing coverage path or evidence gaps (use ins-coverage-gap-detection),
  or drafting claimant or adjuster communications (use ins-claim-comms).
---

## Overview

Routes claims to the correct handling lane — standard adjuster, catastrophe desk, bodily injury team, commercial claims unit, or SIU — based on the routing rules matrix, claim classification, severity, evidence status, and fraud indicator flags. Resolves adjuster assignments by territory and specialization, enforces routing matrix compliance, and sends Teams notifications after claims operations review. Creates calendar holds for SLA-driven review deadlines.

This skill operates in "AI draft plus approve" mode — every routing recommendation is presented for claims operations review and confirmation before any message is sent or tracker is updated.

## When to Use

- A claim has been classified and assessed and needs adjuster or team assignment
- Claims operations needs to determine which handling lane fits a specific claim
- A claim needs escalation to the catastrophe desk after a catastrophe declaration
- A claim needs SIU referral based on fraud indicator flags
- A claim needs re-routing after reclassification or new evidence

## When NOT to Use

- Creating a new claim case — use ins-fnol-intake
- Assembling policy and claimant context — use ins-claim-context
- Classifying the claim type or severity — use ins-claim-classifier
- Assessing initial coverage path or missing evidence — use ins-coverage-gap-detection
- Drafting claimant, broker, or adjuster communications — use ins-claim-comms
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read claim data and routing matrix", activeForm="Reading routing inputs")
TaskCreate(subject="Recommend routing and send notifications", activeForm="Routing claim")
```

### Step 1: Read Routing Inputs

**Read the claim case, classification, and evidence assessment:**
- `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent` — current case data

**Read the routing matrix:**
- `SearchM365(sources=["files"], query="claims routing matrix")` then `ReadFileContent` — who handles what by claim type, severity, product line, and jurisdiction

**Read the fraud trigger checklist:**
- `SearchM365(sources=["files"], query="fraud trigger checklist")` then `ReadFileContent` — SIU referral criteria and thresholds

**Resolve adjusters and teams:**
- `SearchPeople` — resolve adjuster names by territory and specialization
- `GetManagerDetails` / `GetDirectReportsDetails` — claims operations org structure for escalation paths
- `GetUserDetails` — resolve specific adjuster or supervisor identities

### Step 2: Determine Routing

Match the claim to the routing matrix based on:

| Routing Factor | Source |
|----------------|--------|
| Claim type | Classification from ins-claim-classifier |
| Severity tier | Classification from ins-claim-classifier |
| Product line | Claim case record |
| Jurisdiction | Claim case record |
| Fraud indicators | Evidence assessment from ins-coverage-gap-detection |
| Catastrophe flag | Classification or catastrophe bulletin |

### Handling Lanes

| Lane | Handles | Typical Claim Types |
|------|---------|---------------------|
| **Standard adjuster** | Routine property and auto claims within normal parameters | Property damage (non-catastrophe), auto physical damage, minor water damage |
| **Catastrophe desk** | Claims linked to declared catastrophe events | Property damage — catastrophe, storm, wildfire, flood |
| **Bodily injury team** | Claims involving personal injury | Bodily injury, medical payments, personal injury protection |
| **Liability adjuster** | Third-party liability claims | General liability, premises liability |
| **Commercial claims unit** | Business property and commercial line claims | Commercial property, business interruption, equipment damage |
| **Complex claims unit** | High-value, multi-peril, or litigation-involved claims | Any claim type at complex severity |
| **SIU referral** | Claims with fraud indicator flags | Any claim type with elevated fraud indicators (requires human confirmation) |

### Step 3: Present Routing Recommendation

Present the routing recommendation via Adaptive Card (invoke `render-ui` skill first) for claims operations review:

- **Case header** — Claim ID, Claimant Name, Policy Number, Date of Loss
- **Classification** — claim type, severity, and confidence level
- **Evidence status** — evidence completeness percentage and key gaps
- **Recommended handling lane** — target lane from routing matrix
- **Recommended adjuster or team** — specific assignment based on territory, specialization, and availability
- **Routing rationale** — why this lane and adjuster, based on which routing matrix criteria matched
- **Fraud indicator status** — if SIU referral is recommended, display the triggering indicators prominently
- **SLA status** — time elapsed since intake, deadline for triage completion
- **Escalation flags** — if the claim requires supervisor review, complex claims involvement, or catastrophe desk routing

### Step 4: Execute Routing (After Confirmation)

After claims operations confirmation:

**Send Teams notification:**
- `PostMessage` — Teams message to the assigned adjuster or team with:
  - Claim ID, Claimant Name, Policy Number, Date of Loss
  - Claim type, severity, and handling lane
  - SLA deadline
  - Link to the SharePoint claim folder (do not include full policy or claimant details in the Teams message)
  - Required action and review deadline

**Create SLA deadline calendar hold:**
- `CreateEvent` — calendar hold for the triage SLA deadline so the assigned adjuster has a visible reminder

**Update tracker:**
- Record assigned adjuster, handling lane, routing timestamp, routing rationale, and confirming reviewer in the claims tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find claims tracker, routing matrix, fraud trigger checklist |
| ReadFileContent | Read routing matrix, tracker, fraud triggers |
| SearchPeople | Resolve adjuster names by territory and specialization |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths |
| GetUserDetails | Resolve specific adjuster or supervisor contacts |
| PostMessage | Teams notifications to assigned adjuster or team |
| CreateDraftMessage | Outlook draft for formal routing notification |
| CreateEvent | Calendar hold for SLA review deadlines |

## Guardrails

- **Present routing recommendation for claims operations review** before sending any messages or updating the tracker
- **Never auto-route to SIU without explicit human confirmation** — SIU referrals carry regulatory and legal implications; the skill recommends but the human decides
- **Never auto-assign claims above the configurable severity threshold** without supervisor review — complex and high-severity claims require claims supervisor sign-off
- **Never route to someone outside the routing matrix** — ad hoc adjuster assignments are not permitted
- **Escalate to the claims operations manager** if no clear routing path is found in the matrix
- **Include SLA deadline** in all routing notifications — the 4-business-hour triage window is operationally critical
- **Routing rationale must be captured** in the claim tracker for audit purposes — every routing decision records actor, rationale, and confirming reviewer
- **Never include full policy details, claimant financial data, or medical information in Teams messages** — link to the SharePoint claim folder instead
- **SIU referral notifications must be restricted** to authorized SIU personnel — never surface SIU referral details in general claims channels or claimant-facing outputs
- **Never make coverage, liability, or payment decisions** as part of routing — routing assigns handling ownership only; all claim authority decisions are human-owned
