---
name: fin-control-path
description: |
  Assesses risk and determines the control path for an AP invoice exception based on
  the approval matrix, threshold table, and exception classification.
  Use when user asks to "assess risk for this exception", "what's the control path",
  "recommend approval route", "evaluate exception risk",
  "control path for AP case [ID]", "risk assessment for invoice [number]",
  "what approvals are needed for this exception", or "exception risk check".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  assembling matching context (use fin-match-context),
  classifying exception type (use fin-exception-classify),
  routing to action owner (use fin-exception-routing),
  drafting outreach or approval packets (use fin-ap-comms),
  or updating system status (use fin-status-update).
---

## Overview

Reads the AP approval matrix, threshold table, and escalation policy from SharePoint, cross-references against the classified exception case data including amount, business unit, exception type, and vendor profile, and recommends the appropriate control path. Determines whether the exception can be resolved through standard processing, requires approval, or must be escalated. Identifies SOX-sensitive conditions, segregation of duties conflicts, and duplicate payment risks.

This skill operates in "AI draft plus approve" mode — the control path recommendation is presented for AP analyst review and confirmation before any routing action is taken.

## When to Use

- An exception has been classified and needs a control path determination before routing
- The user wants to know what approvals are required for a specific exception
- An AP analyst needs to assess risk before deciding how to handle an exception

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Assembling matching context — use fin-match-context
- Classifying exception type — use fin-exception-classify
- Routing to the action owner — use fin-exception-routing
- Drafting outreach or approval packets — use fin-ap-comms
- Updating case status — use fin-status-update
- Approving or releasing payment — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read approval matrix and exception case data", activeForm="Reading control inputs")
TaskCreate(subject="Assess risk and determine control path", activeForm="Determining control path")
```

### Step 1: Read Control Inputs

Locate and read required inputs:

- **Exception case data** — from the tracker: `SearchM365(sources=["files"], query="AP exception tracker")` then `ReadFileContent`
- **Classification results** — exception type, confidence score, secondary types from tracker or prior skill output
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Case ID]")` then `ReadFileContent`
- **Approval matrix** — `SearchM365(sources=["files"], query="AP approval matrix")` or `SearchM365(sources=["files"], query="AP approval threshold")` then `ReadFileContent`
- **Escalation policy** — `SearchM365(sources=["files"], query="AP escalation policy")` or `SearchM365(sources=["files"], query="controller escalation")` then `ReadFileContent`

### Step 2: Assess Risk and Determine Control Path

**Risk flag detection:**

| Risk Flag | What to Check | Severity |
|-----------|--------------|----------|
| Above materiality threshold | Invoice amount exceeds the materiality threshold | High — requires elevated approval |
| Duplicate payment risk | Classification indicates duplicate invoice candidate | Critical — must investigate before any processing |
| Vendor master mismatch | Vendor details on invoice differ from master record | High — potential fraud or data integrity issue |
| Segregation of duties conflict | PO creator is the same person who would approve the exception | High — SOX control violation |
| Low classification confidence | Exception classified with confidence below 70% | Medium — requires manual triage |
| Multi-issue exception | Multiple exception types identified | Medium — may require coordinated resolution |
| Recurring vendor exception | Same vendor has had 3+ similar exceptions in the last 6 months | Medium — may indicate systemic issue |
| Late payment risk | Invoice is approaching or past due date | Medium — may need expedited processing |

**Determine control path:**

| Control Path | Criteria | Required Action |
|-------------|----------|-----------------|
| **Auto-resolve** | Amount below auto-resolve threshold, single clear exception type with high confidence, no risk flags | Route to AP analyst for standard processing |
| **Standard approval** | Amount below materiality threshold, clear exception type, no critical risk flags | Route to cost center owner or AP manager per matrix |
| **Escalated review** | Amount above materiality threshold, OR critical risk flags present, OR low confidence classification | Route to controller or AP operations manager |
| **Manual investigation** | Duplicate payment risk, vendor master mismatch, segregation conflict, OR multi-issue with low confidence | Hold for manual investigation by AP manager or controller |

**Determine required approvers** based on the approval matrix:

| Level | Typical Criteria |
|-------|-----------------|
| **AP Analyst** | Auto-resolve cases below the first threshold |
| **Cost Center Owner** | Standard exceptions requiring business confirmation |
| **AP Manager** | Exceptions above the first threshold or with medium risk flags |
| **Controller** | Exceptions above materiality threshold, critical risk flags, or SOX-sensitive conditions |
| **CFO / VP Finance** | Exceptions above the second threshold or with fraud indicators |

### Step 3: Present Control Path Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

**Recommended Control Path:**
- Path type (Auto-resolve / Standard approval / Escalated review / Manual investigation)
- Rationale citing specific policy sections and thresholds

**Required Approvers:**
- Named approvers per the approval matrix
- Approval level required and why

**Risk Flags:**
- Each detected risk with severity, description, and recommended mitigation
- Segregation of duties check result

**Policy Citations:**
- Specific policy sections and threshold values supporting the recommendation
- Exception taxonomy rule applied

**Recommended Next Step:**
- Route for standard processing (fin-exception-routing)
- Route for escalated review (fin-exception-routing with elevated level)
- Hold for manual investigation (do not route; AP manager to review)
- Return for additional context (fin-match-context)

After user confirms the control path, update the exception tracker with: Priority (Standard / High / Critical), Control Path, Required Approval Level, and Risk Flags.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, approval matrix, threshold table, escalation policy, context packet |
| ReadFileContent | Read approval matrix, threshold table, tracker, context packet |
| SearchM365 (connectors) | Pull additional ERP data for risk validation if needed |
| SearchPeople / GetUserDetails | Resolve approver identities for segregation check |
| GetManagerDetails | Reporting chain for segregation of duties verification |

## Guardrails

- **Present recommendation for AP analyst review** before any routing action — this is AI draft plus approve mode
- **Never recommend auto-resolution for exceptions above the materiality threshold** regardless of exception type
- **Always check for segregation of duties conflicts** — the PO creator cannot approve the exception for that PO
- **Flag duplicate payment risk as critical severity** — duplicate payments are the highest-priority financial risk in AP
- **Cite the specific policy section and threshold** for every routing recommendation
- **Escalate to AP manager** if no clear control path exists in the approval matrix
- **Never auto-approve any exception** regardless of amount or exception type — approval is always a human action
- **Log the control path determination** with rationale, risk flags, required approval level, and timestamp in the tracker
