---
name: bnk-gap-risk-detection
description: |
  Assesses urgency, detects evidence gaps, identifies escalation flags, and tracks
  regulatory compliance timelines for fraud and dispute cases.
  Use when user asks to "check evidence gaps", "what's missing for [case ID]",
  "assess fraud case urgency", "risk assessment for this dispute",
  "case readiness check", "evidence status for [case]",
  "Reg E deadline check", "SLA status for [case ID]",
  or "escalation flags for this case".
  Do NOT use for creating a new case (use bnk-case-intake),
  assembling account and transaction context (use bnk-fraud-context-packet),
  classifying fraud scenario or dispute type (use bnk-scenario-classifier),
  routing to analyst queues (use bnk-case-routing),
  or drafting communications (use bnk-case-comms).
---

## Overview

Compares required evidence and handling milestones against case status to identify evidence gaps, urgency factors, escalation triggers, and regulatory compliance timeline risks. Detects missing customer statements, unsigned affidavits, pending merchant responses, approaching Reg E deadlines, card network chargeback windows, and SLA breaches. Presents findings for analyst review before any tracker updates.

This skill operates in "AI assist" mode — it reads and analyzes but only presents findings via Adaptive Card. No write actions occur without explicit analyst confirmation.

## When to Use

- A fraud or dispute case needs an evidence completeness check
- An analyst wants to assess case urgency before prioritizing their queue
- Regulatory deadlines (Reg E, card network) are approaching and need monitoring
- A case has been open for a while and needs a status review
- SLA compliance needs to be checked across pending cases

## When NOT to Use

- Creating a new case — use bnk-case-intake
- Assembling account and transaction context — use bnk-fraud-context-packet
- Classifying fraud scenario or dispute type — use bnk-scenario-classifier
- Routing to an analyst queue — use bnk-case-routing
- Drafting customer or analyst communications — use bnk-case-comms
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and evidence checklist", activeForm="Reading case materials")
TaskCreate(subject="Assess gaps, urgency, and risk", activeForm="Analyzing case risks")
```

### Step 1: Read Case Materials

**Read the case record and context packet:**
- `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="context packet [Case ID]")` then `ReadFileContent`

**Read the evidence checklist:**
- `SearchM365(sources=["files"], query="dispute evidence checklist")` then `ReadFileContent`

**Read handling rules and regulatory guidance:**
- `SearchM365(sources=["files"], query="fraud operations rules")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="Reg E handling guidance")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="card network chargeback deadlines")` then `ReadFileContent`

**Check the case evidence folder:**
- `GetDriveChildren` — list all submitted evidence in the case folder
- `SearchM365(sources=["email"], query="[Case ID] affidavit OR statement")` — check for pending customer responses

### Step 2: Detect Evidence Gaps

For each item on the dispute evidence checklist, check whether it has been received:

**Evidence Gap Categories:**

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing customer statement | No written statement from the cardholder or account holder | High |
| Missing affidavit | Fraud affidavit not submitted or unsigned | High |
| Missing transaction documentation | Supporting transaction records not provided | Medium |
| Missing merchant response | Merchant has not responded to chargeback inquiry | Medium |
| Missing identity verification | Customer identity not confirmed for account takeover cases | Critical |
| Missing police report | Law enforcement report requested but not received (for high-value fraud) | Medium |

### Step 3: Assess Urgency and Escalation Flags

**Urgency factors:**

| Factor | Description | Priority Impact |
|--------|-------------|-----------------|
| **High-value transaction** | Disputed amount exceeds threshold defined in fraud rules | Elevates to Immediate |
| **Account takeover indicators** | Multiple unauthorized transactions or account changes | Elevates to Immediate |
| **Cross-border activity** | Transactions in unusual geographies | Elevates to High |
| **Repeated merchant pattern** | Same merchant appears in multiple alerts on this or related accounts | Elevates to High |
| **Customer vulnerability** | Elderly, disabled, or recently compromised account | Elevates to High |
| **Velocity anomaly** | Multiple transactions in short timeframe | Elevates to High |

**Escalation flags:**

| Flag | Description | Required Action |
|------|-------------|-----------------|
| **AML/SAR indicators** | Suspicious activity patterns that may require SAR filing | Immediate AML team escalation |
| **Reg E provisional credit deadline** | 10-business-day provisional credit window approaching | Regulatory compliance escalation |
| **Card network chargeback window** | Network-specific deadline for initiating chargeback | Chargeback team notification |
| **SLA breach** | Case has exceeded or is about to exceed the 30-minute triage SLA | Supervisor notification |
| **Repeat fraud on same account** | Multiple fraud events within 90 days | Elevated review and possible account action referral |
| **Internal fraud indicators** | Patterns suggesting employee involvement | Immediate special investigations referral |

### Step 4: Track Regulatory Timelines

**Reg E timeline (electronic fund transfers and debit card disputes):**
- 10 business days — provisional credit decision deadline from dispute receipt
- 45 calendar days — investigation completion deadline (standard)
- 90 calendar days — investigation completion deadline (new accounts, POS, foreign transactions)

**Card network timelines:**
- Visa: 120 calendar days from transaction date for most dispute types
- Mastercard: 120 calendar days from transaction date for most dispute types
- Network-specific deadlines vary by dispute reason code

Calculate days remaining for each applicable deadline and flag any within 5 business days.

### Step 5: Present Risk Assessment

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Customer Name (masked account), Dispute Type, Current Status
- **Urgency recommendation** — Immediate / Standard / Low Priority with supporting factors
- **Evidence scorecard** — X of Y required items received, list of missing items with severity
- **Escalation flags** — each flag with description, required action, and responsible party
- **Regulatory timeline status** — Reg E deadline, card network deadline, days remaining, risk level
- **SLA status** — time elapsed since intake, SLA deadline, compliance status
- **Recommended next steps** — prioritized list (evidence follow-up, escalation, analyst assignment)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, context packet, evidence checklist, fraud rules, Reg E guidance |
| ReadFileContent | Read case data, checklists, procedures, regulatory guidance |
| GetDriveChildren | List submitted evidence in case folder |
| SearchM365 (email) | Check for pending customer responses or affidavit submissions |

## Guardrails

- **Never mark an evidence item as complete** without verifying the document exists in the SharePoint case folder
- **Flag Reg E provisional credit deadlines explicitly** — these are federal regulatory requirements, not operational preferences; missing a Reg E deadline creates compliance liability
- **Separate AML-related escalation flags from standard fraud or dispute flags** — AML escalation routes to a different team with different access controls and must not be mixed with general fraud triage
- **Present all findings for analyst review** before updating the Excel tracker — this skill is read-only until the analyst confirms
- **Never recommend closing, de-prioritizing, or clearing a case** without analyst confirmation
- **Include SLA deadline status** in every assessment output — the 30-minute triage window is operationally critical
- **Never assess actual fraud validity** or determine whether the customer's claim is legitimate — surface factual evidence gaps and timeline risks only
- **Flag internal fraud indicators immediately** for special investigations referral — do not continue standard triage processing
- **Never disclose investigation status details** in the risk assessment that would be visible to non-authorized personnel
