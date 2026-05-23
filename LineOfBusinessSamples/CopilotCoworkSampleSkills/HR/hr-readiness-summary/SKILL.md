---
name: hr-readiness-summary
description: |
  Prepares a readiness summary for onboarding review — consolidating case status,
  completed items, outstanding gaps, blockers, and required approvals into a
  reviewer-ready format.
  Use when user asks to "summarize onboarding status", "readiness review for [name]",
  "onboarding deck for manager review", "prepare readiness report",
  "onboarding summary for [case]", "status report for [employee] onboarding",
  "readiness check for [name]", or "onboarding review prep".
  Do NOT use for creating a new onboarding case (use hr-onboarding-intake),
  assembling readiness context (use hr-readiness-packet),
  detecting missing items or risks (use hr-gap-detection),
  assigning owners and routing tasks (use hr-task-routing),
  or drafting outreach or reminders (use hr-onboarding-comms).
---

## Overview

Consolidates all onboarding case artifacts — tracker data, readiness packet, gap report, task assignments, and communication history — into a reviewer-ready summary. Produces the summary in the format most appropriate for the review context: Adaptive Card for quick chat-based review, Word document for formal sign-off, or PowerPoint deck for manager-facing review presentation.

This skill operates in "AI draft plus approve" mode — the summary is presented for HR specialist review and confirmation before being shared with reviewers or managers.

## When to Use

- An onboarding case is approaching readiness review and needs a consolidated summary
- A hiring manager needs a status update on their new hire's onboarding progress
- An HR operations manager needs a review-ready summary before the readiness confirmation step
- The user wants to see the current state of an onboarding case in a summarized format

## When NOT to Use

- Creating a new onboarding case — use hr-onboarding-intake
- Assembling readiness context from scratch — use hr-readiness-packet
- Detecting specific missing items or risks — use hr-gap-detection
- Assigning task owners — use hr-task-routing
- Drafting individual outreach communications — use hr-onboarding-comms
- Final readiness confirmation — this is always a human decision (HR-ONB-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read all case artifacts", activeForm="Reading case artifacts")
TaskCreate(subject="Prepare readiness summary", activeForm="Preparing readiness summary")
```

### Step 1: Read All Case Artifacts

Locate and read all inputs:

- **Onboarding case data** — `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` for the case row
- **Readiness packet** — `SearchM365(sources=["files"], query="readiness packet [Case ID]")` then `ReadFileContent`
- **Gap report** — from hr-gap-detection output or tracker fields (missing items, blockers, readiness assessment)
- **Task assignments** — from tracker fields (who owns which tasks, completion status)
- **Communication history** — `SearchM365(sources=["email"], query="[Case ID]")` for recent correspondence related to the case
- **Employee onboarding folder** — `GetDriveChildren` to verify current document upload status

### Step 2: Produce Readiness Summary

Determine the output format based on user request or context:

**Format 1: Adaptive Card (default for quick review)**

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Employee Name, Start Date, Role, Location, Department, Hiring Manager
- **Readiness scorecard:**
  - Checklist completion percentage
  - Documents: X of Y received
  - IT provisioning: status
  - Facilities: status
  - Policy acknowledgments: status
- **Outstanding items** — each with owner, priority, and deadline
- **Blockers** — items preventing readiness with responsible party and escalation status
- **Timeline risk** — business days until start date, risk level (On Track / At Risk / Critical)
- **Recommended disposition** — Ready for Review / Needs Action / Needs Escalation / Not Ready

**Format 2: Word document (for formal sign-off)**

Generate a Word document (invoke `docx` skill) containing:

1. **Executive Summary** — one-paragraph readiness assessment with disposition recommendation
2. **Onboarding Case Details** — Case ID, Employee Name, Start Date, Role, Location, Department, Hiring Manager, Employment Type
3. **Checklist Status** — complete table of all checklist items with status (Complete / Pending / Missing / Not Applicable), owner, and deadline
4. **Gap and Risk Summary** — missing documents, readiness blockers, and policy-sensitive conditions from the gap report
5. **Task Assignment Status** — all assigned tasks with owner, status, and completion date
6. **Communication Log** — summary of key communications sent (welcome email, missing documents reminder, IT request, etc.)
7. **Timeline Assessment** — days until start date, critical path items, risk level
8. **Recommended Disposition** — Ready for Review / Conditional Ready (with specific conditions) / Not Ready (with required actions)
9. **Sign-off Section** — space for HR operations manager and hiring manager sign-off with date

**Format 3: PowerPoint deck (for manager-facing review)**

Generate a presentation (invoke `pptx` skill) containing:

1. **Title slide** — "Onboarding Readiness Review — [Employee Name]" with Case ID and date
2. **New hire overview** — name, role, location, department, start date, hiring manager
3. **Readiness scorecard** — visual status of each category (documents, IT, facilities, policies)
4. **Outstanding items** — table of items still needed with owners and deadlines
5. **Blockers and risks** — any items requiring manager action or escalation
6. **Timeline** — days until start, critical path, overall risk level
7. **Recommended next steps** — what the manager needs to do, what HR will handle

### Step 3: Update Tracker

After presenting the summary:
- Update the tracker Status to "Ready for Review" if all critical items are complete
- Update the tracker Status to "Blocked" if critical blockers remain
- Record the summary generation date and format

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, readiness packet, gap report |
| SearchM365 (email) | Find recent correspondence for communication log |
| ReadFileContent | Read all case artifacts |
| GetDriveChildren | Verify current document upload status in onboarding folder |

## Guardrails

- **Every finding must trace to a source artifact** — no fabricated status information; cite the tracker, gap report, or readiness packet for every claim
- **Clearly distinguish complete vs. pending vs. missing vs. blocked items** in all output formats
- **Flag items requiring exception approval** with explicit callouts — do not bury them in the status table
- **Present summary for review before sharing** with managers or finalizing — this is AI draft plus approve mode
- **Never state that an onboarding case is "ready"** if critical items are incomplete — use "Conditional Ready" or "Not Ready" with specific conditions
- **Protect employee PII** — summaries shared with managers should not include SSN, salary, background check details, or medical information
- **Include timeline risk prominently** — reviewers need to see how many business days remain before the start date
- **Never make the readiness disposition decision** — this skill prepares the summary; the HR operations manager makes the final readiness call (HR-ONB-007)
- **Log the summary** — record summary format, generation date, disposition recommendation, and overall readiness percentage in the tracker
