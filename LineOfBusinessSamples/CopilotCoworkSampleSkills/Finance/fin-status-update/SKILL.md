---
name: fin-status-update
description: |
  Updates the AP exception tracker with new status, action taken, and resolution
  metadata for an exception case.
  Use when user asks to "update exception status", "mark case as [status]",
  "log resolution", "update AP tracker",
  "close exception case [ID]", "record action taken for [case]",
  "update status for invoice [number]", or "log disposition for AP case".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  assembling matching context (use fin-match-context),
  classifying exception type (use fin-exception-classify),
  assessing risk or control path (use fin-control-path),
  routing to action owner (use fin-exception-routing),
  or drafting outreach or approval packets (use fin-ap-comms).
---

## Overview

Updates the AP exception tracker with the current case status, action taken, actor identity, timestamp, and next action. Supports status transitions through the exception lifecycle — from routing through approval, rejection, escalation, and resolution. Enforces status transition rules, validates that prior steps are complete, and flags aging cases that have exceeded the SLA.

This skill operates in "AI act within policy" mode — it can execute bounded write actions (tracker updates) within defined rules, provided the update is based on a completed prior step. It does not make approval decisions or disposition judgments.

## When to Use

- An exception case needs its status updated after a routing decision, approval, or resolution action
- The user wants to log what action was taken on an exception
- A case needs to be marked as resolved, rejected, or escalated
- An aging check is needed to identify cases approaching or exceeding the SLA

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Assembling matching context — use fin-match-context
- Classifying exception type — use fin-exception-classify
- Assessing risk or determining the control path — use fin-control-path
- Routing to the action owner — use fin-exception-routing
- Drafting outreach or approval packets — use fin-ap-comms
- Final disposition confirmation — this is always a human decision (FIN-AP-008)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read current case state and validate transition", activeForm="Reading current case state")
TaskCreate(subject="Update tracker and verify", activeForm="Updating exception tracker")
```

### Step 1: Read Current Case State

Locate and read the current case:

- `SearchM365(sources=["files"], query="AP exception tracker")` to find the tracker
- `ReadFileContent` to read the specific exception case row
- Extract: Exception Case ID, Current Status, Assigned To, Classification, Control Path, Priority, Created Date, Last Updated

### Step 2: Validate Status Transition

**Valid status transitions:**

| Current Status | Valid Next Status | Required Condition |
|---------------|-------------------|-------------------|
| New | Classified | Exception type assigned by fin-exception-classify |
| New | Routed | Direct routing without classification (urgent cases) |
| Classified | Assessed | Control path determined by fin-control-path |
| Assessed | Routed | Action owner assigned by fin-exception-routing |
| Routed | Awaiting Approval | Approval request sent via fin-ap-comms |
| Routed | Awaiting Information | Follow-up sent for missing data via fin-ap-comms |
| Awaiting Approval | Approved | Approver has confirmed (human action) |
| Awaiting Approval | Rejected | Approver has rejected (human action) |
| Awaiting Approval | Escalated | Escalation triggered by timeline, risk, or approver request |
| Awaiting Information | Routed | Missing information received; re-routed for processing |
| Escalated | Awaiting Approval | Escalated review assigned to higher authority |
| Escalated | Manual Investigation | Controller has ordered manual investigation |
| Approved | Resolved | Invoice processed and case closed |
| Rejected | Resolved | Invoice returned to supplier and case closed |
| Manual Investigation | Resolved | Investigation complete and disposition confirmed |
| Any | Rework | Case returned to a prior step for correction |

**Validation checks:**
- Verify that the requested transition is valid from the current status
- Verify that the required condition for the transition has been met (e.g., classification must exist before moving to Classified)
- Reject invalid transitions and explain what step must be completed first

### Step 3: Update Tracker

Write the following fields to the tracker:

| Field | Value |
|-------|-------|
| Status | New status per validated transition |
| Action Taken | Description of what was done (e.g., "Classified as amount mismatch with 92% confidence") |
| Actor | Current user identity (`GetMyDetails`) |
| Timestamp | Current date and time |
| Next Action | What needs to happen next (e.g., "Route to cost center owner for approval") |
| Expected Date | Expected date for next action based on SLA |
| Resolution Rationale | If closing: reason for resolution (required for Resolved status) |

**Verify supporting documents** if closing the case:
- `GetDriveChildren` to confirm that the case folder contains the required evidence (context packet, approval record, resolution documentation)
- Flag if required documents are missing

### Step 4: Aging Check (Optional)

If requested or if the case is being reviewed:
- Calculate days since case creation vs. the 2-business-day SLA
- Flag cases exceeding the SLA with an aging alert
- Identify cases that have been in the same status for more than 2 business days without action
- Present aging summary if multiple cases are being reviewed

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker |
| ReadFileContent | Read current tracker state |
| GetDriveChildren | Verify supporting documents exist in case folder |
| GetMyDetails | Current user identity for audit trail |

## Guardrails

- **Validate status transitions** — only allow transitions per the defined transition table; reject invalid transitions with an explanation
- **Only update based on a completed prior step** — routing, approval, or analyst action must precede the status change
- **Log every status change** with actor identity, timestamp, action taken, and case linkage
- **Never move to Resolved status** without a documented disposition rationale
- **Never move to Approved status** without evidence of human approval — this skill records the human decision, it does not make it
- **Flag aging cases** that have exceeded the 2-business-day SLA or remained in the same status for more than 2 business days
- **Verify supporting documents** before allowing case closure — context packet and approval record must be in the case folder
- **Preserve audit trail integrity** — never overwrite prior status history; each update adds to the trail
