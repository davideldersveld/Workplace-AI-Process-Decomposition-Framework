---
name: cs-case-routing
description: |
  Routes a service case to the correct agent, queue, or specialist team based on
  the routing rules, classification, severity, and agent availability.
  Use when user asks to "route case [ID]", "assign this case",
  "send to the right queue", "who handles this type of issue",
  "route this ticket", "assign case to [team]",
  "case routing for [ID]", or "queue assignment for [case]".
  Do NOT use for assembling context (use cs-context-packet),
  classifying the issue (use cs-issue-classifier),
  assessing severity (use cs-severity-assessment),
  drafting a response (use cs-response-drafter),
  or creating a new case (use cs-case-intake).
---

## Overview

Reads the routing rules matrix from SharePoint, cross-references against the case classification, severity, customer entitlement tier, and agent availability, and routes the case to the correct owner or queue. Sends Teams notifications to the assigned agent and posts a case card to the triage channel.

This skill operates in "AI act within policy" mode — routing is determined strictly by the approved routing rules. High-severity and escalation cases are presented for team lead review before routing.

## When to Use

- A case has been classified and severity assessed, and is ready for routing
- The user wants to determine which agent or queue should handle a case
- A case needs to be rerouted after reclassification or escalation

## When NOT to Use

- Assembling customer and account context — use cs-context-packet
- Classifying the issue type — use cs-issue-classifier
- Assessing severity and SLA path — use cs-severity-assessment
- Drafting a customer response — use cs-response-drafter
- Creating a new case — use cs-case-intake

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read routing rules and case data", activeForm="Reading routing inputs")
TaskCreate(subject="Determine routing destination and check availability", activeForm="Determining routing")
TaskCreate(subject="Send routing notifications", activeForm="Sending notifications")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Case data** — from the case tracker: `SearchM365(sources=["files"], query="service case tracker")`
- **Classification** — issue category, intent, suggested queue from cs-issue-classifier
- **Severity** — severity level, SLA path, escalation conditions from cs-severity-assessment
- **Routing rules** — `SearchM365(sources=["files"], query="routing rules")` or `SearchM365(sources=["files"], query="case routing matrix")`
- **Team roster** — `SearchM365(sources=["files"], query="service team roster")` or resolve via people tools

Read each document using `ReadFileContent`.

### Step 2: Determine Routing Destination

Apply the routing rules to determine assignment:

| Factor | How It Affects Routing |
|--------|----------------------|
| Issue category | Maps to specific queue or team in the routing matrix |
| Severity level | Critical/High may route to senior agents or escalation queue |
| Customer tier | Premium customers may have dedicated agents or priority queue |
| Agent availability | Check calendar to avoid routing to unavailable agents |
| Skills match | Route to agents with expertise in the issue category |
| Current workload | Consider queue size and agent case count if visible |
| Escalation conditions | Route to escalation manager if escalation criteria are met |

**Resolve agent identity:**
- `SearchPeople(query="<queue name or role>")` to find agents in the target queue
- `GetUserDetails(user_id="<agent>")` to get agent profile
- `ListCalendarView(start=..., end=...)` to check agent availability for the next 4 hours
- `GetManagerDetails(user_id="<agent>")` to identify team lead for escalations

**For high-severity or escalation cases:**
Present the routing recommendation for team lead review before sending notifications. Include the severity rationale and escalation conditions.

### Step 3: Send Routing Notifications

After routing is confirmed (automatically for standard cases, after team lead review for high-severity):

**Teams direct message to assigned agent** (use `PostMessage`):
- Case ID, customer name, issue summary
- Classification and severity level
- SLA deadline with countdown
- Link to case in SharePoint

**Channel post to triage channel** (use `PostChannelMessage`):
- Case card with Case ID, category, severity, assigned agent
- Do not include full customer details in channel posts — use case ID and summary only

**Update case tracker:**
- Set Assigned To, Status to "Routed", and routing timestamp

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, routing rules, team roster |
| ReadFileContent | Read routing matrix, case data |
| SearchPeople / GetUserDetails | Resolve agent identities |
| GetManagerDetails / GetDirectReportsDetails | Team structure for escalation paths |
| ListCalendarView | Check agent availability |
| PostMessage | Teams direct message to assigned agent |
| PostChannelMessage | Case card to triage channel |

## Guardrails

- **Route only to agents in the approved routing matrix** — never assign to someone outside the defined teams
- **Check agent availability** via calendar before assignment — flag conflicts
- **Never auto-route high-severity or escalation cases** — present recommendation for team lead review first
- **Include SLA deadline** in every routing notification — agents must know the time constraint
- **Protect customer privacy in channel posts** — use Case ID and summary only, not full customer details
- **Log routing decision** with rationale, assigned agent, and timestamp in the case tracker
- **Flag routing gaps** — if the routing matrix has no match for a category/severity combination, escalate to service operations manager
