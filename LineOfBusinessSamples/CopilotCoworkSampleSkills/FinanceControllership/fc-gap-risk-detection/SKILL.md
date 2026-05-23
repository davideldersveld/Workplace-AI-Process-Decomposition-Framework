---
name: fc-gap-risk-detection
description: |
  Detects missing supporting evidence, control risks, and policy-sensitive conditions
  in a journal entry case by comparing the evidence packet against the control checklist.
  Use when user asks to "check journal for gaps", "what's missing for this entry",
  "journal risk check", "audit readiness for journal [ID]",
  "control review for close entry", "gap check for journal [case]",
  "evidence review for [entry]", or "risk assessment for journal [ID]".
  Do NOT use for creating a new journal case (use fc-journal-intake),
  assembling ledger and policy context (use fc-journal-context),
  determining the approval path (use fc-approval-routing),
  or drafting summaries and follow-ups (use fc-journal-comms).
---

## Overview

Compares the journal evidence packet against the control checklist requirements, cross-references with prior period entries for anomaly detection, and presents a gap and risk report. The report separates missing evidence (fixable by uploading documents) from control risks (requiring escalation or additional approval), with confidence levels and rationale for every finding.

This skill operates in "AI assist" mode — it presents findings for accountant review via Adaptive Card. It does not auto-clear control items or update the tracker without user confirmation.

## When to Use

- A journal packet has been assembled and needs review before routing for approval
- The user wants to check whether a journal entry meets audit readiness requirements
- An accountant wants to understand what gaps or risks exist for a specific entry

## When NOT to Use

- Creating a new journal case — use fc-journal-intake
- Assembling ledger, policy, and support context — use fc-journal-context
- Determining the approval path — use fc-approval-routing
- Drafting journal summaries or follow-up requests — use fc-journal-comms
- Posting a journal entry to the ERP — this is always a human action outside of Cowork

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read journal packet and control checklist", activeForm="Reading review inputs")
TaskCreate(subject="Detect gaps and assess control risks", activeForm="Analyzing for gaps and risks")
TaskCreate(subject="Present gap and risk report", activeForm="Preparing risk report")
```

### Step 1: Read Review Inputs

Locate and read required inputs:

- **Journal case data** — from the tracker: `SearchM365(sources=["files"], query="journal tracker")`
- **Journal evidence packet** — from fc-journal-context output: `SearchM365(sources=["files"], query="journal packet [Case ID]")` or `SearchM365(sources=["files"], query="evidence packet [Case ID]")`
- **Control checklist** — `SearchM365(sources=["files"], query="journal control checklist")` or `SearchM365(sources=["files"], query="close control checklist")`
- **Case folder contents** — `GetDriveChildren` to verify what supporting documents are actually uploaded
- **Prior period entries** — from the tracker or `SearchM365(sources=["files"], query="journal tracker")` filtered for similar entries

Read each document using `ReadFileContent`.

### Step 2: Detect Gaps and Assess Risks

Compare the journal packet against the control checklist. For each control item, determine whether it is satisfied, partially satisfied, or missing.

**Missing evidence detection:**

| Gap Type | What to Check | Severity |
|----------|--------------|----------|
| Missing supporting schedule | Required schedule not in case folder | High — cannot approve without it |
| Missing reconciliation | Reconciliation required by policy but not uploaded | High — required for balance validation |
| Missing or incomplete rationale memo | Entry has no written rationale or rationale is too vague | Medium — can be remediated quickly |
| Missing prior-period reference | Reversal or reclassification entry without reference to original | High — audit requirement |

**Control risk detection:**

| Risk Type | What to Check | Severity |
|-----------|--------------|----------|
| Above materiality threshold | Entry amount exceeds the materiality threshold without enhanced approval documentation | High — requires controller or CFO approval |
| Unusual account combination | Debit-credit account pair not seen in prior periods for this entity | Medium — may indicate misclassification |
| Duplicate entry candidate | Similar amount, accounts, and period to an existing case | High — potential duplicate posting |
| Reversal without original | Reversal entry with no linked original entry reference | High — audit integrity risk |
| Cross-entity entry | Entry spans multiple legal entities | Medium — requires additional sign-off per policy |
| Late close adjustment | Entry submitted after the standard close cutoff date | Medium — requires controller approval |
| Segregation concern | Requestor is the same person likely to approve | High — segregation of duties violation |

**Prior period comparison:**
- Search for entries with the same accounts, entity, and similar amounts in the last 4 close periods
- Flag entries that are significantly different from prior period patterns (amount more than 2x prior period, new account combinations)
- Note if this is a recurring entry type and whether prior instances were approved without issues

**Confidence assessment for each finding:**
- **High confidence** — clear evidence that a required item is missing or a control threshold is exceeded
- **Medium confidence** — reasonable indication but could have an explanation not visible in the data
- **Low confidence** — possible concern but may be normal for this entry type

### Step 3: Present Gap and Risk Report

Present via Adaptive Card (invoke `render-ui` skill first):

**Missing Evidence section:**
- Each missing item with: what is missing, why it is required (policy reference), and how to resolve it
- Severity indicator (High / Medium)

**Control Risk section:**
- Each risk finding with: what was detected, why it matters, confidence level, and recommended action
- Severity indicator (High / Medium)

**Prior Period Comparison:**
- Similar entries from prior periods with amounts and outcomes
- Flag any significant deviations from prior patterns

**Overall Readiness Assessment:**
- **Ready for Routing** — all control items satisfied, no high-severity gaps or risks
- **Needs Evidence** — missing documents that must be uploaded before routing
- **Needs Escalation** — control risks that require elevated review or additional approval
- **Needs Rework** — fundamental issues (imbalance, duplicate, segregation concern) that must be resolved

After user confirms the findings, update the journal tracker with the gap and risk assessment results.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find journal tracker, evidence packet, control checklist, prior period entries |
| ReadFileContent | Read control checklist, journal packet, tracker |
| GetDriveChildren | Verify uploaded support documents in case folder |

## Guardrails

- **Present findings for accountant review** before updating any tracker — this is AI assist mode
- **Never mark a control item as satisfied** without document evidence in the case folder
- **Flag entries above materiality threshold** with elevated visibility — these require enhanced approval
- **Clearly separate missing evidence from control risks** — missing evidence is fixable; control risks may require escalation
- **Include confidence level and rationale** for every finding — accountants need to understand why something was flagged
- **Never auto-clear a risk flag** — only a human reviewer can determine that a flagged risk is acceptable
- **Cross-reference with prior period entries** — anomalies relative to historical patterns are important review signals
- **Log the gap and risk assessment** — record findings, severity, and overall readiness in the tracker after user confirmation
