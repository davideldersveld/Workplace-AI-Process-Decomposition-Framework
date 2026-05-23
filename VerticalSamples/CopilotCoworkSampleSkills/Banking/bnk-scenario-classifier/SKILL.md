---
name: bnk-scenario-classifier
description: |
  Classifies fraud scenario type or dispute category and recommends the handling lane
  based on the fraud taxonomy and case evidence.
  Use when user asks to "classify this fraud case", "what type of dispute is this",
  "categorize fraud scenario", "triage classification for [case ID]",
  "what kind of fraud is this", "dispute type for [case]",
  "handling lane for [case ID]", or "scenario classification for this alert".
  Do NOT use for creating a new case (use bnk-case-intake),
  assembling account and transaction context (use bnk-fraud-context-packet),
  assessing urgency or evidence gaps (use bnk-gap-risk-detection),
  routing to analyst queues (use bnk-case-routing),
  or drafting communications (use bnk-case-comms).
---

## Overview

Compares case attributes against the fraud taxonomy and dispute type definitions to recommend a scenario classification and handling lane. Matches transaction patterns, channel characteristics, customer narrative, and prior history against known scenario profiles. Presents classification with confidence level and supporting evidence for analyst review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents classification as a recommendation via Adaptive Card. The analyst confirms or overrides the classification before it is recorded.

## When to Use

- A fraud context packet has been assembled and the case needs scenario classification
- An analyst wants a recommended dispute type and handling lane before review
- A case needs reclassification after new evidence or updated context
- Triage needs to determine whether a case belongs in fraud operations, dispute operations, chargeback, or AML escalation

## When NOT to Use

- Creating a new case — use bnk-case-intake
- Assembling account and transaction context — use bnk-fraud-context-packet
- Assessing urgency, evidence gaps, or risk — use bnk-gap-risk-detection
- Routing to an analyst queue — use bnk-case-routing
- Drafting customer or analyst communications — use bnk-case-comms
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and fraud taxonomy", activeForm="Reading classification inputs")
TaskCreate(subject="Classify scenario and recommend handling lane", activeForm="Classifying fraud scenario")
```

### Step 1: Read Classification Inputs

**Read the fraud context packet:**
- `SearchM365(sources=["files"], query="context packet [Case ID]")` then `ReadFileContent`
- If no context packet exists, note this as a prerequisite gap — recommend running bnk-fraud-context-packet first

**Read the fraud taxonomy and dispute definitions:**
- `SearchM365(sources=["files"], query="fraud taxonomy")` then `ReadFileContent` — scenario types, pattern indicators, handling lane criteria
- `SearchM365(sources=["files"], query="dispute type definitions")` then `ReadFileContent` — dispute categories and handling rules

**Read the case tracker:**
- `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent` — current case data

**Read classification examples (if available):**
- `SearchM365(sources=["files"], query="fraud classification examples")` — approved scenario classification examples and playbooks

### Step 2: Classify Scenario

Compare case attributes against the fraud taxonomy to determine the scenario type. Consider:

**Transaction attributes:**
- Transaction amount and pattern (single large, multiple small, velocity)
- Merchant category and known risk indicators
- Channel (card-not-present vs. card-present, ATM, mobile, online)
- Authorization status and timing

**Customer attributes:**
- Customer narrative and reported circumstances
- Prior fraud alert and dispute history on this account
- Account age and relationship tenure
- Customer tier and product type

**Pattern indicators:**
- Same-merchant repeated alerts
- Cross-border or unusual geography
- Channel mismatch (e.g., card-present transaction in a different city than customer's location)
- Velocity anomalies (multiple transactions in short timeframe)
- Known friendly fraud indicators

### Scenario Types

Classify into one of the taxonomy-defined categories. Common categories include:

| Scenario Type | Description | Typical Handling Lane |
|---------------|-------------|----------------------|
| **Card-not-present fraud** | Unauthorized online or phone transaction | Fraud operations |
| **Card-present counterfeit** | Counterfeit card used at POS terminal | Fraud operations |
| **Account takeover** | Unauthorized access and account manipulation | Fraud operations (elevated) |
| **Friendly fraud** | Customer disputes legitimate transaction | Dispute operations |
| **Billing dispute** | Recurring charge, subscription, or merchant error | Dispute operations |
| **Merchant error** | Duplicate charge, wrong amount, or processing error | Chargeback operations |
| **Lost or stolen card** | Physical card compromised | Fraud operations |
| **ATM dispute** | Cash not dispensed or wrong amount | Dispute operations |
| **AML-related** | Suspicious activity indicators present | AML escalation (immediate) |

### Step 3: Assess Confidence

Rate classification confidence:

- **High confidence** — case attributes match a single scenario type clearly; transaction pattern, channel, and customer narrative are consistent
- **Medium confidence** — case attributes match a primary scenario type but some indicators point to alternatives; additional evidence may clarify
- **Low confidence** — case attributes are ambiguous or match multiple scenario types; analyst judgment required

### Step 4: Present Classification

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Customer Name (masked account), Transaction Reference
- **Recommended scenario type** — primary classification with description
- **Confidence level** — High / Medium / Low with explanation
- **Handling lane recommendation** — Fraud Operations / Dispute Operations / Chargeback / AML Escalation
- **Supporting evidence** — key attributes that support the classification
- **Alternative scenarios** — if confidence is medium or low, list alternatives with supporting indicators
- **AML escalation flag** — if any AML or suspicious activity indicators are present, flag prominently regardless of primary classification

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find fraud taxonomy, dispute type definitions, classification examples, case tracker |
| ReadFileContent | Read taxonomy, definitions, context packet, tracker |
| GetDriveChildren | Case evidence folder contents for pattern matching context |

## Guardrails

- **Present classification as a recommendation only** — never auto-assign the final scenario type; the analyst confirms or overrides
- **Always display confidence level** and the evidence supporting the classification — analysts must see why the recommendation was made
- **Flag low-confidence classifications explicitly** — do not present uncertain classifications as definitive
- **Never auto-clear fraud indicators or suspicious activity flags** — fraud and AML indicators can only be cleared by authorized human reviewers
- **If the case matches AML or SAR escalation criteria, flag immediately** — do not continue with standard fraud or dispute classification; AML escalation takes priority over all other classification
- **Never assess actual fraud likelihood or customer credibility** — classify the scenario type and handling lane only; the analyst determines validity
- **Use only taxonomy-defined scenario types** — do not invent new categories; if the case does not match any defined type, recommend manual classification with the closest matches noted
- **Do not modify the case tracker** — this skill presents analysis only; tracker updates happen after analyst confirmation
