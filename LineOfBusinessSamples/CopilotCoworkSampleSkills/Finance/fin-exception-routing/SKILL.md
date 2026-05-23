---
name: fin-exception-routing
description: |
  Routes AP invoice exceptions to the correct action owner or approver based on
  the control path recommendation, approval matrix, and org hierarchy.
  Use when user asks to "route this exception", "assign to the right owner",
  "send to approver", "who handles this exception",
  "route AP case [ID] for approval", "assign exception to owner",
  "submit exception for review", or "exception routing for [invoice]".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  assembling matching context (use fin-match-context),
  classifying exception type (use fin-exception-classify),
  assessing risk or control path (use fin-control-path),
  drafting outreach or approval packets (use fin-ap-comms),
  or updating system status (use fin-status-update).
---

## Overview

Reads the control path recommendation and approval matrix, resolves the named action owner or approver via the org hierarchy, verifies segregation of duties, and routes the exception via Teams notification and Outlook draft after user confirmation. Updates the exception tracker with owner assignment and routing metadata.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for AP analyst review and confirmation before any messages are sent or tracker records updated.

## When to Use

- An exception has been assessed for risk and control path and is ready for routing
- The user wants to know who should handle or approve a specific exception
- An exception needs to be rerouted after rework or reclassification

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Assembling matching context — use fin-match-context
- Classifying exception type — use fin-exception-classify
- Assessing risk or determining the control path — use fin-control-path
- Drafting outreach or approval packets — use fin-ap-comms
- Updating case status — use fin-status-update
- Approving or releasing payment — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read control path and resolve action owner", activeForm="Resolving action owner")
TaskCreate(subject="Present routing recommendation", activeForm="Preparing routing recommendation")
TaskCreate(subject="Send routing notifications", activeForm="Sending routing notifications")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Exception case data** — from the tracker: `SearchM365(sources=["files"], query="AP exception tracker")` then `ReadFileContent`
- **Control path recommendation** — from fin-control-path output (control path, required approval level, risk flags)
- **Approval matrix** — `SearchM365(sources=["files"], query="AP approval matrix")` then `ReadFileContent`
- **Context packet** — for case summary details

### Step 2: Resolve Action Owner

Based on the control path and approval matrix, determine who should receive the exception:

| Control Path | Action Owner | How to Resolve |
|-------------|-------------|----------------|
| Auto-resolve | AP analyst (current user) | Self-assignment; `GetMyDetails` |
| Standard approval | Cost center owner or AP manager | `SearchPeople(query="cost center owner [Business Unit]")` or look up from approval matrix |
| Escalated review | Controller or AP operations manager | `SearchPeople(query="controller")` or `SearchPeople(query="AP operations manager")` |
| Manual investigation | AP manager or controller | `SearchPeople(query="AP manager")` with escalation to controller |

**Resolve named owner:**
- `SearchPeople(query="<owner name or role>")` to find the person
- `GetUserDetails(user_id="<owner>")` to confirm profile and authority
- `GetManagerDetails(user_id="<analyst>")` to verify reporting chain for escalation

**Segregation of duties check:**
- The assigned owner or approver must NOT be the same person who created the PO referenced on the invoice
- If the natural approver per the matrix created the PO, route to their manager instead
- If no valid owner can be found without a segregation conflict, escalate to the AP operations manager with an explanation

### Step 3: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Assigned action owner** — name, role, and why this person was selected
- **Control path** — Auto-resolve / Standard approval / Escalated review / Manual investigation
- **Required action** — what the owner needs to do (review and approve, investigate discrepancy, confirm receipt, etc.)
- **Segregation check result** — confirmed that owner did not create the PO
- **Risk flags** — any risk flags from fin-control-path that the owner should be aware of
- **SLA context** — remaining time within the 2-business-day SLA; flag if at risk
- **Recommended communications** — Teams notification and Outlook draft

### Step 4: Send Routing Notifications (After User Confirmation)

**Teams direct message to assigned owner** (use `PostMessage`):
- Exception Case ID, Invoice ID, Vendor Name, Amount
- Exception type and control path
- Required action summary
- Link to context packet in SharePoint
- SLA deadline context

**Outlook draft for formal routing** (use `CreateDraftMessage`):
- Subject: "AP Exception Review Request — [Exception Case ID] — [Vendor Name] [Amount]"
- Body: Exception summary, classification, control path, required action, risk flags if any, link to case packet
- To: assigned action owner

**Update exception tracker:**
- Set Assigned To, Status to "Routed", Routing Date, Expected Resolution Date (based on SLA)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, approval matrix, context packet |
| ReadFileContent | Read approval matrix, tracker, case data |
| SearchPeople / GetUserDetails | Resolve action owner and confirm identity |
| GetManagerDetails / GetDirectReportsDetails | Reporting chain for segregation check and escalation |
| PostMessage | Teams notification to action owner with case summary |
| CreateDraftMessage | Outlook draft for formal routing request |

## Guardrails

- **Present routing recommendation for review** before sending any messages — this is AI draft plus approve mode
- **Never route to someone who created the PO** for that invoice — segregation of duties is mandatory
- **Always verify the owner has sufficient authority** per the approval matrix — do not route to someone below the required level
- **Escalate to AP operations manager** if no valid owner exists without a segregation conflict
- **Include Exception Case ID in every communication** for audit traceability
- **Never auto-approve any exception** regardless of amount or type
- **Never bypass the approval matrix** — the matrix determines routing, not convenience or availability
- **Flag SLA risk** if the exception is approaching the 2-business-day deadline
- **Log the routing decision** with assigned owner, segregation check result, rationale, and timestamp in the tracker
