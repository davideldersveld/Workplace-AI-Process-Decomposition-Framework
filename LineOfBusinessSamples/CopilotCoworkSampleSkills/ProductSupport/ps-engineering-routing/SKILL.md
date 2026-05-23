---
name: ps-engineering-routing
description: |
  Routes an escalated support case to the correct engineering owner or
  queue based on the product area ownership map and routing rules.
  Use when user asks to "route escalation [ID] to engineering",
  "assign engineering owner for [case]",
  "send to [product area] queue",
  "who handles this defect area",
  "route this to the engineering team",
  or "engineering assignment for [escalation ID]".
  Do NOT use for creating a new escalation record (use ps-escalation-intake),
  gathering evidence and context (use ps-evidence-packet),
  classifying the issue or defect path (use ps-defect-classifier),
  assessing customer impact or severity (use ps-impact-assessment),
  or drafting handoff communications (use ps-handoff-drafter).
---

## Overview

Determines the correct engineering triage lead and queue for an escalated support case based on the engineering ownership map, product area classification, severity level, and routing rules. Resolves engineering identities, checks availability, presents the routing plan for approval, and then sends Teams notifications to the assigned engineering triage lead and posts an escalation summary to the engineering triage channel. Updates the tracker with the assignment.

This skill operates in "AI act within policy" mode — it follows the approved engineering ownership map and routing rules to determine assignments and executes bounded notification actions after escalation engineer confirmation. Sev 1 and Sev 2 routing requires product support manager approval.

## When to Use

- An escalation has been classified and impact-assessed and is ready for engineering assignment
- An escalation engineer needs to determine which engineering team owns a product area
- A reroute is needed after reclassification changes the product area
- An engineering triage lead is unavailable and the escalation needs to be reassigned

## When NOT to Use

- Creating a new escalation record — use ps-escalation-intake
- Gathering case evidence, logs, and telemetry context — use ps-evidence-packet
- Classifying the issue type or suspected defect path — use ps-defect-classifier
- Assessing customer impact or recommending severity — use ps-impact-assessment
- Drafting the engineering handoff or customer update — use ps-handoff-drafter
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Determine engineering owner and check availability", activeForm="Identifying engineering owner")
TaskCreate(subject="Route escalation and send notifications", activeForm="Routing to engineering")
```

### Step 1: Read Routing Inputs

**Read the escalation record:**
- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — Escalation ID, Product Area, Classification, Severity, Issue Type, SLA Deadline, Evidence Completeness

**Read the engineering ownership map:**
- `SearchM365(sources=["files"], query="engineering ownership map")` then `ReadFileContent` — product area to engineering team and triage lead mapping

**Read routing rules:**
- `SearchM365(sources=["files"], query="escalation routing rules")` then `ReadFileContent` — routing policies, severity-based escalation paths, queue assignments

### Step 2: Determine Engineering Owner

Match the escalation to the correct engineering triage lead based on the ownership map:

| Routing Factor | How It Affects Assignment |
|---------------|------------------------|
| **Product area** | Primary routing key — determines which engineering team owns the area |
| **Component** | Secondary routing key — may route to a specific sub-team within the product area |
| **Issue type** | May affect routing (performance issues may go to a different team than defects) |
| **Severity** | Sev 1/2 may route to senior engineering leads rather than standard triage |

### Step 3: Resolve Engineering Identities

- `SearchPeople` — resolve the engineering triage lead by name and role for the matched product area
- `GetUserDetails` — verify the triage lead's profile and current role
- `GetManagerDetails` / `GetDirectReportsDetails` — resolve escalation paths for high-severity cases requiring senior engineering attention

### Step 4: Check Engineering Availability

- `ListCalendarView` — check the assigned engineering triage lead's calendar for availability
- If the triage lead is in meetings, OOO, or unavailable:
  - Check for a designated backup in the ownership map
  - If no backup, escalate to the engineering manager for manual assignment

### Step 5: Verify Evidence Readiness

Before routing, verify that the evidence completeness score from ps-impact-assessment is above threshold:

| Readiness | Action |
|-----------|--------|
| **Ready** | Proceed with routing |
| **Conditionally ready** | Route with a note about missing evidence items |
| **Not ready** | Flag for additional evidence collection — do not route to engineering |

### Step 6: Present Routing Plan

Present the routing plan via Adaptive Card (invoke `render-ui` skill first):

- **Escalation header** — Escalation ID, Customer, Product Area, Severity, SLA Deadline
- **Assigned engineering triage lead** — name, role, team, availability status
- **Engineering queue** — which queue or team channel will receive the escalation
- **Routing rationale** — why this owner was selected (ownership map match, product area, component)
- **Evidence readiness** — evidence completeness status and any flagged gaps
- **SLA status** — time remaining on the 4-hour SLA deadline
- **Severity approval** — "Requires product support manager approval" for Sev 1 and Sev 2
- **Routing plan label** — "ROUTING PLAN — escalation engineer confirmation required before sending notifications"

### Step 7: Execute Routing (After Confirmation)

**Send Teams notification to assigned engineering triage lead:**
- `PostMessage` — direct message with:
  - Escalation ID and severity level
  - Customer name and product area
  - Issue summary and suspected defect path
  - Evidence packet link in SharePoint
  - SLA deadline and time remaining
  - Classification details (issue type, confidence level)

**Post to engineering triage channel:**
- `PostChannelMessage` — escalation summary card to the appropriate engineering triage channel with:
  - Escalation ID, severity, product area
  - Brief issue summary
  - Assigned triage lead
  - Evidence packet link

**Update the escalation tracker:**
- Assigned To: engineering triage lead name
- Engineering Queue: queue or team name
- Status: "Routed to Engineering"
- Routed Date: current timestamp
- Routing Rationale: documented reason for assignment

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find escalation tracker, engineering ownership map, routing rules |
| ReadFileContent | Read ownership map, routing rules, tracker |
| SearchPeople | Resolve engineering triage leads by product area |
| GetUserDetails | Verify engineering lead profile and role |
| GetManagerDetails / GetDirectReportsDetails | Resolve escalation paths for high-severity cases |
| ListCalendarView | Check engineering lead availability |
| PostMessage | Send Teams notification to assigned triage lead |
| PostChannelMessage | Post escalation summary to engineering triage channel |

## Guardrails

- **Route only to engineering owners listed in the approved ownership map** — do not assign to engineers outside the defined mapping
- **Require product support manager approval before routing Sev 1 or Sev 2** — high-severity routing affects engineering prioritization
- **Verify engineering lead availability** before assignment — flag OOO, meeting conflicts, or unavailability and identify a backup
- **Do not route if evidence completeness is below threshold** — an incomplete packet wastes engineering time and delays resolution; flag for additional evidence collection first
- **Include the evidence packet link and SLA deadline** in every routing notification — engineering needs immediate access to evidence and urgency context
- **Present the routing plan for escalation engineer review** before sending notifications — routing decisions are confirmed before execution
- **Log every routing decision** with rationale, assigned owner, timestamp, and confirming user in the escalation tracker
- **If no matching engineering owner exists** for the product area, escalate to the engineering manager rather than guessing — never route to a team without ownership confirmation
- **Never include customer financial data** in engineering routing notifications — customer tier is sufficient for context
- **Include classification details and confidence level** in the routing notification — engineering needs to know whether the classification is high-confidence or tentative
- **Flag reroutes explicitly** — if the escalation was previously routed and is being reassigned, note the prior assignment and reason for the change
