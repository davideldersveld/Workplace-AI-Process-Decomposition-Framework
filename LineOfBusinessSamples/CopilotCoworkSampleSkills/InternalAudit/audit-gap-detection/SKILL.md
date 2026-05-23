---
name: audit-gap-detection
description: |
  Detects missing evidence, scope gaps, traceability issues, and timeline risks
  for audit engagements by comparing submitted materials against evidence requirements.
  Use when user asks to "check evidence gaps for [case]", "what's missing for audit [case ID]",
  "evidence status for engagement", "audit readiness check",
  "scope gap analysis for [process area]", "evidence completeness review",
  "what evidence is still needed", or "audit gap report for [engagement]".
  Do NOT use for creating a new audit case (use audit-request-intake),
  assembling scope and evidence context (use audit-evidence-packet),
  routing evidence requests (use audit-request-routing),
  or drafting audit communications (use audit-case-comms).
---

## Overview

Compares required evidence items against submitted materials in the engagement evidence folder to identify gaps that could delay or block audit testing. Detects missing mandatory evidence, evidence covering the wrong period, uncontacted control owners, unaddressed prior finding remediation, and scope areas with no assigned ownership. Classifies each gap by impact severity and presents findings for auditor review.

This skill operates in "AI assist" mode — it reads and analyzes but only presents findings via Adaptive Card for auditor review. It does not modify audit status, update conclusions, or write to the case tracker without explicit user confirmation.

## When to Use

- An audit engagement is in evidence collection and the auditor needs a completeness check
- Evidence submission deadlines are approaching and the team needs to know what is still missing
- The auditor wants to assess readiness for testing before the walkthrough or fieldwork phase
- A follow-up check is needed after evidence requests were sent

## When NOT to Use

- Creating a new audit case — use audit-request-intake
- Assembling the initial evidence packet — use audit-evidence-packet
- Routing evidence requests to owners — use audit-request-routing
- Drafting communications about gaps — use audit-case-comms
- Confirming audit packet disposition — this is always a human decision (IA-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read evidence packet and checklist", activeForm="Reading evidence materials")
TaskCreate(subject="Detect gaps and prepare report", activeForm="Analyzing evidence gaps")
```

### Step 1: Read Evidence Materials

**Read the audit evidence packet:**
- `SearchM365(sources=["files"], query="audit packet [Case ID]")` then `ReadFileContent`
- If no packet exists, note this as a prerequisite gap — recommend running audit-evidence-packet first

**Read the evidence requirements checklist:**
- `SearchM365(sources=["files"], query="evidence checklist [Case ID]")` then `ReadFileContent`
- This is the authoritative list of what evidence is required

**Browse the engagement evidence folder:**
- `GetDriveChildren` — list all submitted evidence artifacts in the engagement folder
- Record each file name, upload date, and file type

**Read the case tracker:**
- `SearchM365(sources=["files"], query="audit case tracker")` then `ReadFileContent` — current case status and evidence status

### Step 2: Compare Requirements Against Submissions

For each item on the evidence requirements checklist, check whether:

1. **Evidence is submitted** — a corresponding document exists in the engagement evidence folder
2. **Evidence covers the correct period** — the document's timeframe matches the audit period
3. **Evidence format is acceptable** — the document type matches the required format
4. **Source is appropriate** — the evidence was provided by the expected control owner or process owner

### Step 3: Classify Gaps

**Gap Category 1: Missing Mandatory Evidence**
- Required evidence items with no corresponding submission in the engagement folder
- Severity: **High** (blocks testing for that control) or **Critical** (blocks the entire engagement)

**Gap Category 2: Wrong-Period Evidence**
- Evidence submitted but covering a different timeframe than the audit period
- Severity: **Medium** (may require supplemental evidence) or **High** (renders evidence unusable)

**Gap Category 3: Uncontacted Control Owner**
- Control owner identified in the evidence requirements but no evidence request sent or no response received
- Severity: **Medium** (if timeline permits follow-up) or **High** (if testing deadline is approaching)

**Gap Category 4: Prior Finding Remediation Not Evidenced**
- Prior audit findings flagged for follow-up but no remediation evidence provided
- Severity: **Medium** (if finding was low severity) or **High** (if finding was high or critical severity)

**Gap Category 5: Unassigned Scope Area**
- Process area or control within scope has no identified control owner
- Severity: **High** (cannot route evidence requests without ownership)

**Gap Category 6: Traceability Gap**
- Evidence item submitted but no clear link to the specific control or checklist item it satisfies
- Severity: **Low** (informational — auditor can establish the link during review)

### Step 4: Assess Timeline Risk

Calculate business days remaining until key milestones:
- **Evidence submission deadline** — based on the audit timeline
- **Testing start date** — when fieldwork is scheduled to begin
- **Report draft deadline** — when the audit report is due

Timeline risk levels:
- **On Track** — 5+ business days to next milestone with no high-severity gaps
- **At Risk** — fewer than 5 business days to next milestone OR high-severity gaps requiring action
- **Critical** — fewer than 2 business days to next milestone OR critical gaps unresolved

### Step 5: Present Gap Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Engagement header** — Case ID, Process Area, Audit Period, Lead Auditor
- **Evidence scorecard** — X of Y items received, X gaps identified, overall completeness percentage
- **Gap list** — each gap with: checklist item reference, gap category, description, severity, responsible party, recommended action
- **Prior finding follow-up** — remediation status for each flagged finding
- **Timeline risk** — business days to next milestone, risk level, critical path items
- **Recommended actions** — prioritized list of what needs to happen next (sorted by severity then deadline proximity)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find audit packet, evidence checklist, submitted evidence, case tracker |
| ReadFileContent | Read evidence checklist, audit packet, control documentation |
| GetDriveChildren | List submitted evidence in engagement folder |

## Guardrails

- **Present all findings as "evidence gaps" or "scope observations"** — never characterize findings as audit conclusions, control deficiencies, or risk assessments
- **Every gap must cite the specific checklist item** and the expected vs. actual evidence status — no unsupported assertions
- **Never assess control effectiveness, operational risk, or management performance** — surface factual gaps only; the auditor makes all professional judgments
- **Flag gaps related to prior finding remediation separately** — these indicate whether the organization addressed known issues and deserve distinct visibility
- **Flag timeline-critical gaps with elevated visibility** — items that could delay testing or the audit report
- **Present findings for auditor review via Adaptive Card** before updating any tracker or checklist — this skill is read-only until the user confirms
- **Read-only mode by default** — this skill does not modify audit status, evidence status, or conclusions; it only surfaces analysis
- **Never reveal preliminary audit observations** in gap descriptions — gap language must be limited to evidence completeness and procedural status, not audit substance
