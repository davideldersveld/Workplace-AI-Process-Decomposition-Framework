---
name: hr-gap-detection
description: |
  Detects missing documents, readiness blockers, and policy-sensitive conditions
  in an onboarding case by comparing the readiness packet against the checklist.
  Use when user asks to "check onboarding gaps", "what's missing for [name]",
  "onboarding readiness check", "audit onboarding case",
  "gap check for onboarding [ID]", "what do we still need for [employee]",
  "onboarding blocker check", or "readiness audit for [name]".
  Do NOT use for creating a new onboarding case (use hr-onboarding-intake),
  assembling readiness context (use hr-readiness-packet),
  assigning owners and routing tasks (use hr-task-routing),
  drafting outreach or reminders (use hr-onboarding-comms),
  or preparing readiness summaries (use hr-readiness-summary).
---

## Overview

Compares the onboarding readiness packet against the required document checklist, cross-references with the employee's onboarding folder to verify what has been uploaded, and presents a gap and risk report. The report separates missing documents (fixable by uploading), readiness blockers (requiring action from another team), and policy-sensitive conditions (requiring elevated review), with severity levels and recommended actions for each finding.

This skill operates in "AI assist" mode — it presents findings for HR specialist review via Adaptive Card. It does not auto-clear checklist items or update the tracker without user confirmation.

## When to Use

- A readiness packet has been assembled and needs review before routing or summarization
- The user wants to check whether an onboarding case meets readiness requirements
- An HR specialist wants to understand what gaps or blockers exist for a specific new hire

## When NOT to Use

- Creating a new onboarding case — use hr-onboarding-intake
- Assembling readiness context — use hr-readiness-packet
- Assigning task owners and routing work — use hr-task-routing
- Drafting outreach or reminders — use hr-onboarding-comms
- Preparing readiness summaries — use hr-readiness-summary
- Making employment eligibility decisions — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read readiness packet and onboarding checklist", activeForm="Reading review inputs")
TaskCreate(subject="Detect gaps, blockers, and policy-sensitive conditions", activeForm="Analyzing for gaps and risks")
TaskCreate(subject="Present gap and risk report", activeForm="Preparing gap report")
```

### Step 1: Read Review Inputs

Locate and read required inputs:

- **Onboarding case data** — from the tracker: `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent`
- **Readiness packet** — from hr-readiness-packet output: `SearchM365(sources=["files"], query="readiness packet [Case ID]")` or `SearchM365(sources=["files"], query="onboarding packet [Employee Name]")` then `ReadFileContent`
- **Onboarding checklist** — `SearchM365(sources=["files"], query="onboarding checklist")` then `ReadFileContent`
- **Location-specific requirements** — `SearchM365(sources=["files"], query="[location] onboarding requirements")` then `ReadFileContent`
- **Employee onboarding folder contents** — `GetDriveChildren` to verify what documents have been uploaded

### Step 2: Detect Gaps and Assess Risks

Compare the readiness packet against the checklist. For each checklist item, determine whether it is satisfied, partially satisfied, or missing.

**Missing document detection:**

| Gap Type | What to Check | Severity |
|----------|--------------|----------|
| Missing identification documents | Government-issued ID, passport, or other required identification not uploaded | High — cannot complete I-9 verification |
| Missing tax forms | W-4, state tax withholding, or equivalent not submitted | High — payroll cannot be set up |
| Missing policy acknowledgments | Required policy sign-offs (handbook, code of conduct, IT acceptable use) not completed | Medium — can be completed on Day 1 if flagged |
| Missing work authorization | I-9 or work authorization documentation incomplete or expired | Critical — legal compliance requirement |
| Missing IT provisioning request | Equipment, account, or access request not submitted to IT | High — employee cannot work without systems access |
| Missing manager confirmation | Hiring manager has not confirmed start date, seat assignment, or team details | Medium — may delay Day 1 logistics |

**Readiness blocker detection:**

| Blocker Type | What to Check | Severity |
|-------------|--------------|----------|
| IT provisioning not started | No IT setup request logged; employee starts in fewer than 5 business days | High — systems access at risk for Day 1 |
| Background check pending | Background check initiated but not cleared; start date approaching | High — may need to delay start |
| Facilities not confirmed | Badge, workspace, or building access not arranged | Medium — may affect Day 1 experience |
| Benefits enrollment window | Benefits enrollment deadline approaching without employee action | Medium — time-sensitive |
| Offer letter unsigned | Offer letter sent but not returned signed | High — employment not formally accepted |

**Policy-sensitive condition detection:**

| Condition | What to Check | Severity |
|-----------|--------------|----------|
| Work authorization gap | I-9 documentation incomplete, expired, or flagged for reverification | Critical — legal compliance; flag for HR operations manager |
| Privileged access request | Role requires elevated system access (admin, financial systems, PII access) | High — requires additional security review |
| International hire | Employee located outside the US or on a work visa requiring sponsorship | High — additional compliance requirements apply |
| Rehire | Employee previously employed by the company; prior records may need review | Medium — may have existing accounts or policy history |
| Contractor conversion | Converting from contractor to employee; benefits and systems transitions required | Medium — requires coordination across HR, IT, and finance |

**Timeline risk assessment:**
- Calculate business days remaining until start date
- Flag cases with fewer than 3 business days remaining and incomplete items as "At Risk"
- Flag cases with fewer than 1 business day remaining and incomplete critical items as "Critical Timeline Risk"

### Step 3: Present Gap and Risk Report

Present via Adaptive Card (invoke `render-ui` skill first):

**Missing Documents:**
- Each missing item with: what is missing, why it is required, who needs to provide it, and suggested deadline
- Severity indicator (Critical / High / Medium)

**Readiness Blockers:**
- Each blocker with: what is blocked, responsible team or person, impact if not resolved, and recommended action
- Severity indicator (High / Medium)

**Policy-Sensitive Conditions:**
- Each condition with: what was detected, why it requires attention, recommended escalation path
- Severity indicator (Critical / High / Medium)

**Timeline Assessment:**
- Business days until start date
- Number of critical items remaining
- Overall timeline risk (On Track / At Risk / Critical)

**Overall Readiness Assessment:**
- **Ready for Review** — all checklist items satisfied, no high-severity gaps or blockers
- **Needs Documents** — missing items that can be resolved by uploading documents or completing forms
- **Needs Action** — blockers requiring action from another team (IT, facilities, hiring manager)
- **Needs Escalation** — policy-sensitive conditions requiring HR operations manager review
- **Critical** — work authorization or legal compliance issues that must be resolved before start date

After user confirms the findings, update the onboarding tracker with the gap assessment results and checklist completion percentage.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, readiness packet, checklists, location requirements |
| ReadFileContent | Read checklist, readiness packet, tracker |
| GetDriveChildren | Verify uploaded documents in the employee's onboarding folder |

## Guardrails

- **Present findings for HR specialist review** before updating any tracker — this is AI assist mode
- **Never mark a checklist item as complete** without document evidence in the onboarding folder
- **Flag work authorization issues as critical severity** — these are legal compliance requirements with serious consequences
- **Clearly separate missing documents from readiness blockers from policy-sensitive conditions** — each requires different action paths
- **Include severity and recommended action** for every finding — HR specialists need actionable guidance
- **Never auto-clear a gap or risk flag** — only a human reviewer can determine that a flagged issue is resolved
- **Protect employee privacy** — do not surface medical, disability, or accommodation details in the gap report; reference them only as "accommodation coordination pending" if on the checklist
- **Calculate timeline risk** using business days, not calendar days — account for weekends and holidays
- **Log the gap assessment** — record findings, severity, checklist completion percentage, and overall readiness in the tracker after user confirmation
