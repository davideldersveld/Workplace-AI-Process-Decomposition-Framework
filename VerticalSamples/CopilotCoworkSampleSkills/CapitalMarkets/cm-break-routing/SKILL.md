---
name: cm-break-routing
description: |
  Routes trade break cases to the correct desk, operations owner, or escalation lane
  based on the routing matrix and case classification.
  Use when user asks to "route this break", "who handles [break type]",
  "assign break [ID] to the right desk", "send to settlements",
  "escalate to operations control", "route trade exception [ID]",
  "desk assignment for [break]", or "break routing for [case]".
  Do NOT use for creating a new break case (use cm-break-intake),
  assembling trade and settlement context (use cm-context-packet),
  classifying break type or root cause (use cm-break-classifier),
  assessing settlement risk or time criticality (use cm-risk-assessment),
  or drafting counterparty or internal communications (use cm-break-comms).
---

## Overview

Routes trade break cases to the correct operations desk, individual analyst, or escalation lane based on the desk routing matrix, break classification, risk assessment, and asset class. Resolves desk owners via org structure, enforces routing matrix compliance, and sends Teams notifications after supervisor review. Creates calendar holds for settlement cutoff deadlines when urgency warrants it.

This skill operates in "AI draft plus approve" mode — every routing recommendation is presented for operations supervisor review and confirmation before any message is sent or tracker is updated.

## When to Use

- A break has been classified and risk-assessed and needs desk assignment
- An analyst needs to determine which desk or owner handles a specific break type
- A break needs escalation to operations control or a supervisor
- A break needs re-routing after reclassification or new evidence

## When NOT to Use

- Creating a new break case — use cm-break-intake
- Assembling trade and settlement context — use cm-context-packet
- Classifying the break type or likely root cause — use cm-break-classifier
- Assessing settlement risk, fail exposure, or time criticality — use cm-risk-assessment
- Drafting counterparty or internal handoff communications — use cm-break-comms
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read break data and routing matrix", activeForm="Reading routing inputs")
TaskCreate(subject="Recommend routing and send notifications", activeForm="Routing break case")
```

### Step 1: Read Routing Inputs

**Read the break case, classification, and risk assessment:**
- `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent` — current case data

**Read the routing matrix:**
- `SearchM365(sources=["files"], query="desk routing matrix")` then `ReadFileContent` — who handles what by break type, asset class, and priority level

**Resolve desk owners and analysts:**
- `SearchPeople` — resolve desk owners by function, role, or name
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `GetUserDetails` — resolve specific analyst or supervisor identities

**Check analyst availability (for urgent cases):**
- `ListCalendarView` — check assigned analyst or desk owner availability for same-day or critical breaks

### Step 2: Determine Routing

Match the break to the routing matrix based on:

| Routing Factor | Source |
|----------------|--------|
| Break type | Classification from cm-break-classifier |
| Asset class | Break case record |
| Priority level | Risk assessment from cm-risk-assessment |
| Counterparty complexity | Context packet |
| Market or custodian | Settlement details from context packet |

### Handling Lanes

| Lane | Handles | Typical Break Types |
|------|---------|---------------------|
| **Settlements desk** | SSI mismatches, settlement timing issues, custodian discrepancies | SSI mismatch, settlement timing issue |
| **Confirmations desk** | Affirmation failures, confirmation mismatches | Affirmation failure |
| **Middle office** | Allocation discrepancies, booking inconsistencies | Allocation discrepancy |
| **Reference data team** | Security, instrument, or counterparty data errors | Counterparty data error, reference data break |
| **Trade support** | Booking corrections, trade amendment needs | Booking correction needed |
| **Operations control** | Critical breaks, escalations, same-day fail risks, repeat patterns | Any break type at critical priority |

### Step 3: Check Information Barriers

Before routing, verify that the recommended recipient is authorized to receive break details for this asset class and business line:
- Do not route equity desk break details to fixed income personnel
- Do not route proprietary trading break details to client-facing desk personnel
- If the routing matrix suggests a recipient who may be on the wrong side of a Chinese wall, flag for supervisor review

### Step 4: Present Routing Recommendation

Present the routing recommendation via Adaptive Card (invoke `render-ui` skill first) for supervisor review:

- **Case header** — Break ID, Trade ID, Counterparty (short code), Settlement Date, Cutoff Proximity
- **Classification** — break type and confidence level
- **Priority** — from risk assessment
- **Recommended desk** — target handling lane from routing matrix
- **Recommended owner** — specific analyst or desk owner (with availability status for urgent cases)
- **Routing rationale** — why this desk and owner, based on which routing matrix criteria matched
- **Escalation flags** — if the break requires supervisor copy, operations control involvement, or elevated review
- **Information barrier check** — confirmation that the routing does not cross Chinese wall boundaries

### Step 5: Execute Routing (After Confirmation)

After supervisor confirmation:

**Send Teams notification:**
- `PostMessage` — Teams message to the assigned desk owner or analyst with:
  - Break ID, Trade ID, Counterparty (short code), Settlement Date
  - Break type and priority
  - Cutoff proximity
  - Link to the SharePoint break case folder (do not include full trade details in the Teams message)
  - Required action and deadline

**Create cutoff deadline calendar hold (for critical and high-priority breaks):**
- `CreateEvent` — calendar hold for the settlement cutoff deadline so the assigned owner has a visible reminder

**Update tracker:**
- Record assigned owner, desk, routing timestamp, routing rationale, and confirming supervisor in the break tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find break tracker, desk routing matrix, escalation procedures |
| ReadFileContent | Read routing matrix, tracker, escalation rules |
| SearchPeople | Resolve desk owners and analyst identities |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths |
| GetUserDetails | Resolve specific analyst or supervisor contacts |
| ListCalendarView | Check analyst availability for urgent break assignments |
| PostMessage | Teams notifications to assigned desk or analyst |
| CreateEvent | Calendar hold for settlement cutoff deadlines |

## Guardrails

- **Present routing recommendation for supervisor review** before sending any messages or updating the tracker
- **Never route to someone outside the desk routing matrix** — ad hoc analyst assignments are not permitted
- **Critical and high-priority breaks must include the operations supervisor** on all routing notifications
- **Escalations to operations control require explicit user confirmation** — never auto-escalate
- **Never include full trade booking details or settlement instructions in Teams messages** — link to the SharePoint break case folder instead
- **Information barrier compliance** — do not route break details across Chinese wall boundaries; if uncertain, flag for compliance review rather than proceeding
- **Never amend bookings, change SSIs, or authorize settlement actions** as part of routing — routing assigns ownership only; corrective actions are human decisions
- **Log routing decisions** — record actor, rationale, assigned desk, assigned owner, confirming supervisor, and timestamp for audit trail (SEC Rule 17a-4, FINRA Rule 3110)
- **Surface cutoff proximity** in every routing notification — the assigned owner must immediately see time remaining
- **Verify the intake analyst is not the sole reviewer** for critical breaks — segregation of duties requires independent review for high-risk cases
