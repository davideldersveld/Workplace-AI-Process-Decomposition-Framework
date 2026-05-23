---
name: fc-approval-routing
description: |
  Determines the correct approval path for a journal entry based on the approval matrix,
  threshold table, and risk flags, then routes to the assigned reviewer with segregation
  of duties enforcement.
  Use when user asks to "route journal for approval", "who approves this entry",
  "send to reviewer", "approval path for journal [ID]",
  "route journal [case] for sign-off", "submit journal for review",
  "journal approval routing for [ID]", or "who needs to approve this".
  Do NOT use for creating a new journal case (use fc-journal-intake),
  assembling ledger and policy context (use fc-journal-context),
  detecting missing evidence or risks (use fc-gap-risk-detection),
  or drafting summaries and follow-ups (use fc-journal-comms).
---

## Overview

Reads the approval matrix and threshold table from SharePoint, cross-references against the journal case data, gap and risk report, entry amount, entity, and entry type, and recommends the correct approval path. Verifies segregation of duties (preparer cannot be approver), resolves named approvers, and routes the approval request via Teams and Outlook after user confirmation.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for accountant review and confirmation before any messages are sent or tracker records updated.

## When to Use

- A journal entry has passed gap and risk review and is ready for approval routing
- The user wants to determine who needs to approve a journal entry
- A journal case needs to be rerouted after rework or reclassification

## When NOT to Use

- Creating a new journal case — use fc-journal-intake
- Assembling ledger, policy, and support context — use fc-journal-context
- Detecting missing evidence or control risks — use fc-gap-risk-detection
- Drafting journal summaries or follow-up requests — use fc-journal-comms
- Approving or posting a journal entry — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read approval matrix and journal case data", activeForm="Reading approval inputs")
TaskCreate(subject="Determine approval path and verify segregation", activeForm="Determining approval path")
TaskCreate(subject="Send routing notifications", activeForm="Sending approval requests")
```

### Step 1: Read Approval Inputs

Locate and read required inputs:

- **Journal case data** — from the tracker: `SearchM365(sources=["files"], query="journal tracker")`
- **Gap and risk report** — from fc-gap-risk-detection output
- **Approval matrix** — `SearchM365(sources=["files"], query="approval matrix")` or `SearchM365(sources=["files"], query="journal approval thresholds")`
- **Threshold table** — `SearchM365(sources=["files"], query="materiality threshold")` or included in the approval matrix
- **Escalation policy** — `SearchM365(sources=["files"], query="escalation policy")` or `SearchM365(sources=["files"], query="controller escalation")`

Read each document using `ReadFileContent`.

### Step 2: Determine Approval Path

Apply the approval matrix to determine the required approval level:

| Factor | How It Affects Approval Path |
|--------|----------------------------|
| Entry amount | Higher amounts require more senior approval (accounting manager → controller → CFO) |
| Entry type | Certain types (intercompany, equity adjustments, off-cycle) may require controller regardless of amount |
| Entity | Some entities have dedicated controllers or specialized approval requirements |
| Risk flags | High-risk findings from fc-gap-risk-detection may elevate the required approval level |
| Close timing | Late close adjustments may require controller approval regardless of amount |
| Materiality | Entries above the materiality threshold require enhanced approval and documentation |

**Determine the required approval level:**

| Level | Typical Criteria |
|-------|-----------------|
| **Accounting Manager** | Standard entries below the first threshold |
| **Controller** | Entries above materiality threshold, intercompany, late adjustments, high-risk flagged |
| **CFO / VP Finance** | Entries above the second threshold, equity adjustments, extraordinary items |

**Resolve named approvers:**
- `SearchPeople(query="accounting manager [entity]")` or `SearchPeople(query="controller [entity]")`
- `GetUserDetails(user_id="<approver>")` to confirm profile and authority
- `GetManagerDetails(user_id="<requestor>")` to verify reporting chain
- `GetDirectReportsDetails(user_id="<controller>")` to identify the team for escalation

**Segregation of duties check:**
- The assigned approver must NOT be the same person who requested or prepared the journal entry
- If the requestor's direct manager is the natural approver per the matrix, check the next level up
- If no valid approver can be found, escalate to the controller with an explanation

### Step 3: Present Approval Path Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Required approval level** (Accounting Manager / Controller / CFO) with rationale
- **Named approver** resolved from the approval matrix
- **Segregation check result** — confirmed that approver is not the requestor or preparer
- **Risk flags affecting approval** — any findings from fc-gap-risk-detection that elevate the approval level
- **Policy citations** — specific policy sections and threshold values supporting the routing decision
- **Recommended action** — route for standard approval, route with elevated review, or hold for rework

### Step 4: Send Routing Notifications (After User Confirmation)

**Teams direct message to assigned reviewer** (use `PostMessage`):
- Journal Case ID, description, amount, entity, close period
- Entry type and risk summary
- Link to journal packet in SharePoint
- Approval deadline context (close period timing)

**Outlook draft for formal approval request** (use `CreateDraftMessage`):
- Subject: "Journal Entry Review Request — [Journal Case ID] — [Entity] [Period]"
- Body: Journal entry summary, approval level required, risk flags if any, link to evidence packet, case reference
- To: assigned reviewer

**Update journal tracker:**
- Set Approval Path, Assigned Reviewer, Status to "Routed for Approval", and routing timestamp

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find journal tracker, approval matrix, threshold table, escalation policy |
| ReadFileContent | Read approval matrix, threshold table, tracker |
| SearchPeople / GetUserDetails | Resolve approver identities and confirm authority |
| GetManagerDetails / GetDirectReportsDetails | Reporting chain for segregation check and escalation |
| PostMessage | Teams notification to reviewer with case summary |
| CreateDraftMessage | Outlook draft for formal approval request |

## Guardrails

- **Present routing recommendation for review** before sending any messages — this is AI draft plus approve mode
- **Never route to someone who requested or prepared the entry** — segregation of duties is mandatory
- **Always verify the approver has sufficient authority** per the threshold table — do not route to an approver below the required level
- **Escalate to controller** if no valid approver exists in the matrix for the required level
- **Include Journal Case ID in every communication** for audit traceability
- **Never auto-approve any journal entry** regardless of amount or entry type
- **Never bypass the approval matrix** — the matrix determines routing, not convenience or availability
- **Log the routing decision** with rationale, assigned reviewer, segregation check result, and timestamp in the tracker
