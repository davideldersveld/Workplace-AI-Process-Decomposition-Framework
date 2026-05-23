---
name: cm-risk-assessment
description: |
  Assesses settlement risk, fail exposure, time criticality, and evidence gaps
  for trade break cases.
  Use when user asks to "assess fail risk for [trade ID]",
  "how urgent is this break", "settlement risk check", "is this break near cutoff",
  "check aging on break [ID]", "fail risk for [break]",
  "time criticality for [trade exception]", "evidence gaps for [break ID]",
  or "escalation check for [case]".
  Do NOT use for creating a new break case (use cm-break-intake),
  assembling trade and settlement context (use cm-context-packet),
  classifying break type or root cause (use cm-break-classifier),
  routing to desks or operations owners (use cm-break-routing),
  or drafting counterparty or internal communications (use cm-break-comms).
---

## Overview

Assesses settlement risk, fail exposure, time criticality, evidence gaps, and escalation needs for trade break cases. Evaluates break aging, cutoff proximity, counterparty patterns, and missing evidence against settlement rules and fail escalation procedures. Presents risk assessment with priority recommendation and escalation flags for analyst review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents risk assessment as a recommendation via Adaptive Card. The analyst confirms or overrides the assessment before it is recorded.

## When to Use

- A break has been classified and needs settlement risk and urgency assessment
- An analyst wants to check fail exposure before routing or escalation
- A break is approaching settlement cutoff and needs time criticality evaluation
- Evidence gaps need to be identified before a break can be fully triaged
- Break aging needs to be assessed for SLA compliance

## When NOT to Use

- Creating a new break case — use cm-break-intake
- Assembling trade and settlement context — use cm-context-packet
- Classifying the break type or likely root cause — use cm-break-classifier
- Routing to the correct desk or owner — use cm-break-routing
- Drafting counterparty or internal handoff communications — use cm-break-comms
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read break data and settlement rules", activeForm="Reading risk inputs")
TaskCreate(subject="Assess settlement risk and time criticality", activeForm="Assessing settlement risk")
```

### Step 1: Read Risk Assessment Inputs

**Read the break case and classification:**
- `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent` — current case data including classification

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Break ID] OR [Trade ID]")` then `ReadFileContent`

**Read settlement rules and cutoff schedules:**
- `SearchM365(sources=["files"], query="settlement cutoff schedules")` then `ReadFileContent` — cutoff times by market, custodian, and currency
- `SearchM365(sources=["files"], query="fail escalation procedures")` then `ReadFileContent` — fail escalation rules and thresholds

**Read prior fail history:**
- `SearchM365(sources=["files"], query="break report [counterparty]")` — prior fail history for this counterparty or instrument
- `SearchM365(sources=["connectors"], connector_ids=["settlement-connector"])` — current settlement status if indexed

### Step 2: Assess Settlement Risk

Evaluate risk across multiple dimensions:

**Time criticality:**
- Settlement date vs. current date — calculate hours to settlement cutoff
- Market-specific cutoff times (from cutoff schedule)
- Custodian-specific cutoff times
- Whether the break is same-day, next-day, or future-dated

**Fail risk indicators:**
- Same-day settlement with unresolved SSI mismatch — high fail risk
- Approaching cutoff with missing counterparty response — elevated fail risk
- Prior fails on the same counterparty or instrument — pattern-based risk
- Unaffirmed trade near settlement — affirmation deadline risk
- Complex or cross-market settlement with multiple dependencies — structural risk

**Break aging:**
- Hours since break was created
- Hours since last status update
- Whether the break has exceeded the 30-minute SLA target
- Whether the break is trending toward fail based on aging velocity

**Evidence gaps:**
- Missing trade confirmation documentation
- Missing or unmatched SSI documentation
- Missing counterparty response to break notice
- Missing allocation file or allocation discrepancy not resolved
- Missing affirmation or confirmation
- Unverified data points sourced from email rather than system of record

### Step 3: Determine Priority

Assign priority recommendation:

| Priority | Criteria |
|----------|----------|
| **Critical** | Same-day settlement break; approaching cutoff with unresolved issue; prior fail on same counterparty within 30 days; high-value trade with fail exposure |
| **High** | Next-business-day settlement; SSI mismatch not yet responded to; break aging exceeds 30-minute SLA; multiple evidence gaps |
| **Standard** | Future-dated settlement with adequate time; break classified and evidence substantially complete; no fail-risk indicators |
| **Low** | Future-dated settlement with complete evidence; minor discrepancy with clear resolution path; no counterparty or timing risk |

### Step 4: Identify Escalation Flags

Check for escalation triggers:

| Flag | Description | Required Action |
|------|-------------|-----------------|
| **Same-day settlement fail risk** | Break will fail if not resolved before today's cutoff | Immediate supervisor notification |
| **Cutoff approaching** | Less than 2 hours to settlement cutoff | Urgent desk escalation |
| **SLA breach** | Break exceeds 30-minute triage SLA | Supervisor notification |
| **Repeat counterparty pattern** | Same counterparty with 3+ breaks in 30 days | Pattern review escalation |
| **Missing critical evidence** | Trade confirmation or SSI documentation not available | Evidence collection urgency |
| **Cross-market complexity** | Break involves multiple markets, custodians, or currencies | Operations control review |

### Step 5: Present Risk Assessment

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Break ID, Trade ID, Counterparty (short code), Settlement Date
- **Cutoff proximity** — hours and minutes to settlement cutoff, market and custodian cutoff times
- **Priority recommendation** — Critical / High / Standard / Low with rationale
- **Fail risk flag** — Yes / No with specific risk factors
- **Break aging** — hours since creation, hours since last update, SLA status
- **Escalation flags** — any triggered escalation indicators with required actions
- **Evidence gaps** — missing items that increase risk uncertainty, with severity rating
- **Prior fail history** — relevant counterparty or instrument fail patterns
- **Recommended next steps** — route to desk, escalate to supervisor, request missing evidence, or await counterparty response

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find break tracker, settlement cutoff schedules, fail escalation procedures, prior break reports |
| SearchM365 (connectors) | Retrieve current settlement status from settlement platform via Graph Connector |
| ReadFileContent | Read cutoff schedules, escalation rules, tracker, context packet |

## Guardrails

- **Same-day settlement breaks must always be flagged as critical** regardless of other factors — T+1 settlement deadlines are non-negotiable (SEC Rule 15c6-1)
- **Never downgrade a fail-risk flag once set** — only a human supervisor may reduce priority; skills may escalate but never de-escalate
- **Surface cutoff proximity in every output** — operations teams must always see time remaining; settlement states change rapidly
- **Do not calculate financial exposure amounts** — that requires authorized position and pricing data that skills must not access or display
- **Present risk assessment for user review** — do not auto-update tracker for critical or high-priority breaks; these require explicit analyst confirmation
- **Never recommend settlement actions** — do not suggest SSI amendments, booking corrections, or settlement instruction changes; these are human decisions
- **Flag evidence gaps by severity** — distinguish between gaps that block resolution (critical) and gaps that reduce confidence (informational)
- **Note Graph Connector index timestamp** when settlement data is sourced from connectors — the data may be minutes behind the system of record
- **Respect information barriers** — do not reference trading activity, positions, or break details from other business lines separated by Chinese walls
- **Do not suppress or alter break records** — all break cases must be preserved with complete audit trail (FINRA Rule 3110, SEC Rule 17a-4)
- **Every risk assessment must include the current timestamp** — risk assessments are point-in-time and may become stale as settlement status changes
