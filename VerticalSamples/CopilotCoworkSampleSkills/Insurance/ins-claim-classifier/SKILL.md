---
name: ins-claim-classifier
description: |
  Classifies claim type and initial severity, and recommends the handling lane
  based on the claims taxonomy and case evidence.
  Use when user asks to "classify this claim", "what type of claim is this",
  "assess severity for [claim ID]", "claim triage classification",
  "categorize this loss", "what handling lane for [claim]",
  "severity assessment for [loss]", or "claim type for [case ID]".
  Do NOT use for creating a new claim case (use ins-fnol-intake),
  assembling policy and claimant context (use ins-claim-context),
  assessing coverage path or evidence gaps (use ins-coverage-gap-detection),
  routing to adjusters or SIU (use ins-claim-routing),
  or drafting claimant or adjuster communications (use ins-claim-comms).
---

## Overview

Compares claim case attributes against the claims taxonomy and severity rubric to recommend a claim type, severity tier, and handling lane. Matches loss narrative, date of loss, location, policy type, and prior claim history against known claim profiles. Presents classification with confidence level and supporting evidence for adjuster review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents classification as a recommendation via Adaptive Card. The adjuster confirms or overrides the classification before it is recorded.

## When to Use

- A claim context packet has been assembled and the case needs type and severity classification
- An adjuster or intake specialist wants a recommended claim category and handling lane before review
- A case needs reclassification after new evidence or updated context
- Triage needs to determine whether a claim belongs with a standard adjuster, catastrophe desk, bodily injury team, or SIU

## When NOT to Use

- Creating a new claim case — use ins-fnol-intake
- Assembling policy and claimant context — use ins-claim-context
- Assessing initial coverage path or missing evidence — use ins-coverage-gap-detection
- Routing to an adjuster, catastrophe desk, or SIU — use ins-claim-routing
- Drafting claimant, broker, or adjuster communications — use ins-claim-comms
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read claim data and classification inputs", activeForm="Reading classification inputs")
TaskCreate(subject="Classify claim type and recommend handling lane", activeForm="Classifying claim type")
```

### Step 1: Read Classification Inputs

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Claim ID]")` then `ReadFileContent`
- If no context packet exists, note this as a prerequisite gap — recommend running ins-claim-context first

**Read the claims taxonomy and severity rubric:**
- `SearchM365(sources=["files"], query="claims taxonomy")` then `ReadFileContent` — claim type definitions, severity criteria, handling lane assignments
- `SearchM365(sources=["files"], query="severity rubric")` then `ReadFileContent` — severity tier definitions and thresholds

**Read the claims tracker:**
- `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent` — current case data

**Check for catastrophe declarations:**
- `SearchM365(sources=["files"], query="catastrophe event bulletin")` — active catastrophe declarations that may affect classification
- `SearchM365(sources=["connectors"], connector_ids=["claims-platform-connector"])` — active catastrophe flags if indexed

### Step 2: Classify Claim Type

Compare case attributes against the claims taxonomy to determine the claim type. Consider:

**Loss attributes:**
- Loss description and narrative
- Date of loss and timing (single event, gradual, catastrophe period)
- Loss location and geographic indicators
- Type of damage or injury described

**Policy attributes:**
- Product line (homeowners, commercial property, auto, general liability)
- Coverage type and endorsements
- Policy status on date of loss

**Claimant attributes:**
- Prior claim history on the same policy
- Prior claims by the same claimant
- Pattern indicators (frequency, similar loss types, same providers)

### Claim Type Categories

Classify into one of the taxonomy-defined categories:

| Claim Type | Description | Typical Handling Lane |
|------------|-------------|----------------------|
| **Property damage — non-catastrophe** | Property loss from fire, water, theft, vandalism, or similar | Standard adjuster |
| **Property damage — catastrophe** | Property loss linked to a declared catastrophe event | Catastrophe desk |
| **Auto physical damage** | Vehicle damage from collision, comprehensive, or uninsured motorist | Auto adjuster |
| **Bodily injury** | Personal injury claims including medical payments | Bodily injury team |
| **General liability** | Third-party liability claims against the insured | Liability adjuster |
| **Commercial property** | Business property loss, business interruption, equipment damage | Commercial adjuster |
| **Theft or burglary** | Loss from criminal theft or burglary | Standard adjuster (with SIU screening) |
| **Water damage or mold** | Water intrusion, pipe burst, mold remediation | Standard adjuster (with specialist referral) |
| **Suspicious loss** | Loss with fraud indicators or SIU trigger flags | SIU referral (requires human confirmation) |

### Step 3: Assess Severity

Assign severity tier:

| Severity | Criteria |
|----------|----------|
| **Complex** | Bodily injury with hospitalization; commercial loss exceeding policy threshold; multi-peril event; litigation involvement; coverage disputes |
| **High** | Significant property damage; bodily injury without hospitalization; high-value auto loss; multiple claimants |
| **Medium** | Standard property damage; single-vehicle auto damage; minor injury with medical payments only |
| **Low** | Minor property damage; cosmetic vehicle damage; straightforward claims with complete evidence |

### Step 4: Assess Confidence

Rate classification confidence:

- **High confidence** — loss attributes match a single claim type clearly; narrative, location, and policy type are consistent
- **Medium confidence** — loss attributes match a primary type but some indicators point to alternatives; additional evidence may clarify
- **Low confidence** — loss attributes are ambiguous or match multiple claim types; adjuster judgment required

### Step 5: Present Classification

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Claim ID, Claimant Name, Policy Number, Date of Loss
- **Recommended claim type** — primary classification with description
- **Severity tier** — Complex / High / Medium / Low with rationale
- **Confidence level** — High / Medium / Low with explanation
- **Handling lane recommendation** — Standard Adjuster / Catastrophe Desk / Bodily Injury / Liability / Commercial / SIU Referral
- **Supporting evidence** — key attributes that support the classification
- **Alternative classifications** — if confidence is medium or low, list alternatives with supporting indicators
- **Catastrophe flag** — if the loss falls within a declared catastrophe zone or event period, flag prominently
- **Fraud indicator flags** — if prior claim patterns or loss characteristics match SIU trigger criteria, flag separately with elevated visibility
- **Draft label** — "DRAFT FOR ADJUSTER REVIEW — not a coverage or liability determination"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find claims taxonomy, severity rubric, catastrophe bulletins, claims tracker |
| SearchM365 (connectors) | Retrieve catastrophe declarations from claims platform via Graph Connector |
| ReadFileContent | Read taxonomy, rubric, context packet, tracker |

## Guardrails

- **Present classification as a recommendation only** — never auto-assign the final claim type; the adjuster confirms or overrides
- **Always display confidence level** and the evidence supporting the classification — adjusters must see why the recommendation was made
- **Flag low-confidence classifications explicitly** — do not present uncertain classifications as definitive
- **Never auto-assign severity for bodily injury or high-value property claims** without human review — these require adjuster judgment
- **All classification outputs must include the statement** "DRAFT FOR ADJUSTER REVIEW — not a coverage or liability determination"
- **Flag fraud indicators separately** with elevated visibility but never label a claim as fraudulent — SIU determination is human-only
- **Never assess coverage, liability, or claim validity** — classify the claim type and severity only; the adjuster determines coverage
- **Use only taxonomy-defined claim types** — do not invent new categories; if the case does not match any defined type, recommend manual classification with the closest matches noted
- **Do not modify the claims tracker** — this skill presents analysis only; tracker updates happen after adjuster confirmation
