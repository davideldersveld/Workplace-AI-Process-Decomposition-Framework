---
name: cm-break-classifier
description: |
  Classifies trade break type and likely root cause, and recommends the handling lane
  based on the break taxonomy and case evidence.
  Use when user asks to "classify this break", "what type of break is [ID]",
  "root cause analysis for trade exception", "categorize the settlement mismatch",
  "break classification for [trade ID]", "what kind of break is this",
  "handling lane for [break ID]", or "triage classification for this exception".
  Do NOT use for creating a new break case (use cm-break-intake),
  assembling trade and settlement context (use cm-context-packet),
  assessing settlement risk or time criticality (use cm-risk-assessment),
  routing to desks or operations owners (use cm-break-routing),
  or drafting counterparty or internal communications (use cm-break-comms).
---

## Overview

Compares break case attributes against the break taxonomy and classification playbook to recommend a break type, likely root cause, and handling lane. Matches trade characteristics, settlement instruction discrepancies, counterparty patterns, and prior break history against known break profiles. Presents classification with confidence level and supporting evidence for analyst review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents classification as a recommendation via Adaptive Card. The analyst confirms or overrides the classification before it is recorded.

## When to Use

- A break context packet has been assembled and the case needs break type classification
- An analyst wants a recommended break category and handling lane before review
- A case needs reclassification after new evidence or updated context
- Triage needs to determine whether a break belongs in settlements, confirmations, middle office, reference data, or operations control

## When NOT to Use

- Creating a new break case — use cm-break-intake
- Assembling trade and settlement context — use cm-context-packet
- Assessing settlement risk, fail exposure, or time criticality — use cm-risk-assessment
- Routing to the correct desk or owner — use cm-break-routing
- Drafting counterparty or internal handoff communications — use cm-break-comms
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read break data and classification inputs", activeForm="Reading classification inputs")
TaskCreate(subject="Classify break type and recommend handling lane", activeForm="Classifying break type")
```

### Step 1: Read Classification Inputs

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Break ID] OR [Trade ID]")` then `ReadFileContent`
- If no context packet exists, note this as a prerequisite gap — recommend running cm-context-packet first

**Read the break taxonomy and classification playbook:**
- `SearchM365(sources=["files"], query="break taxonomy")` then `ReadFileContent` — break type definitions, pattern indicators, handling lane criteria
- `SearchM365(sources=["files"], query="break classification playbook")` then `ReadFileContent` — approved classification examples and decision logic

**Read the break tracker:**
- `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent` — current case data

**Read historical break reports (if available):**
- `SearchM365(sources=["files"], query="break report [counterparty]")` — prior break reports for pattern matching

### Step 2: Classify Break Type

Compare case attributes against the break taxonomy to determine the break type and likely root cause. Consider:

**Settlement instruction attributes:**
- SSI comparison: expected vs. received — type and nature of discrepancy
- Custodian and settlement agent alignment
- Settlement method match (DTC, Euroclear, manual)
- Currency and account discrepancies

**Trade attributes:**
- Asset class and instrument complexity
- Trade date to settlement date gap (standard vs. non-standard)
- Allocation status (fully allocated, partially allocated, unallocated)
- Quantity, price, or notional mismatches

**Counterparty attributes:**
- Prior break history with this counterparty
- Known counterparty operational issues
- Counterparty response timeliness

**Pattern indicators:**
- Same SSI issue recurring across multiple trades
- Same counterparty generating repeated breaks
- Timing patterns (late allocation, near-cutoff submission)
- Cross-market or cross-custodian complexity

### Break Taxonomy Categories

Classify into one of the taxonomy-defined categories:

| Break Type | Description | Typical Handling Lane |
|------------|-------------|----------------------|
| **SSI mismatch** | Settlement instructions do not match between counterparties | Settlements desk |
| **Allocation discrepancy** | Trade allocation details are incomplete, incorrect, or missing | Middle office |
| **Affirmation failure** | Trade not affirmed by counterparty within required timeframe | Confirmations desk |
| **Settlement timing issue** | Trade will not settle by expected date due to timing or cutoff constraints | Settlements desk |
| **Counterparty data error** | Counterparty reference data is incorrect or outdated | Reference data team |
| **Booking correction needed** | Trade booking contains errors requiring amendment | Trade support (elevated) |
| **Reference data break** | Security, instrument, or market reference data is incorrect or missing | Reference data team |

### Step 3: Assess Confidence

Rate classification confidence:

- **High confidence** — break attributes match a single break type clearly; SSI comparison, trade details, and counterparty context are consistent
- **Medium confidence** — break attributes match a primary type but some indicators point to alternatives; additional evidence may clarify
- **Low confidence** — break attributes are ambiguous or match multiple break types; analyst judgment required

### Step 4: Identify Multi-Cause Indicators

If the break exhibits characteristics of more than one taxonomy category:
- Rank by likelihood based on the strength of matching indicators
- Flag as multi-cause with primary and secondary classifications
- Note which additional evidence would resolve the ambiguity

### Step 5: Present Classification

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Break ID, Trade ID, Counterparty (short code), Settlement Date, Cutoff Proximity
- **Recommended break type** — primary classification with description from taxonomy
- **Likely root cause** — brief explanation of the probable cause based on evidence
- **Confidence level** — High / Medium / Low with explanation
- **Handling lane recommendation** — Settlements / Confirmations / Middle Office / Reference Data / Trade Support
- **Supporting evidence** — key attributes that support the classification
- **Alternative classifications** — if confidence is medium or low, list alternatives with supporting indicators
- **Pattern flags** — if this break matches a repeat counterparty or instrument pattern, flag prominently
- **Discrepancy alert** — if email narrative and system state conflict, surface the discrepancy explicitly

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find break taxonomy, classification playbook, historical break reports, break tracker |
| ReadFileContent | Read taxonomy, playbook, context packet, tracker |
| GetDriveChildren | Break case evidence folder contents for pattern matching context |

## Guardrails

- **Present classification as a recommendation only** — never auto-assign the final break type; the analyst confirms or overrides
- **Always display confidence level** and the evidence supporting the classification — analysts must see why the recommendation was made
- **Flag low-confidence classifications explicitly** — do not present uncertain classifications as definitive
- **Never recommend a booking correction or settlement action** as part of classification — classify the break type only; corrective actions are human decisions
- **Flag cases where email narrative and system state conflict** — surface the discrepancy rather than resolving it; the system of record takes precedence
- **Use only taxonomy-defined break types** — do not invent new categories; if the case does not match any defined type, recommend manual classification with the closest matches noted
- **Do not modify the break tracker** — this skill presents analysis only; tracker updates happen after analyst confirmation
- **Surface cutoff proximity** in the classification output — operations teams must always see time remaining to settlement cutoff
- **Never assess counterparty credibility or assign blame** — classify the operational break type only; relationship and counterparty management is human-owned
- **Respect information barriers** — do not reference trading activity or break details from other business lines separated by Chinese walls
