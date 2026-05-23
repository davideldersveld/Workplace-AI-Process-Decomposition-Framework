---
name: sc-route-exception
description: |
  Routes a supply chain exception to the correct owner based on
  routing rules, item category, site ownership, and priority level.
  Use when user asks to "route shortage [ID]",
  "assign this exception to [team]",
  "who handles [item category] shortages",
  "send to logistics team",
  "escalate shortage case [ID]",
  or "assign owner for [case]".
  Do NOT use for creating a new exception case (use sc-shortage-intake),
  gathering demand and inventory context (use sc-context-packet),
  classifying the exception type (use sc-classify-exception),
  assessing business impact (use sc-impact-assess),
  or drafting shortage communications (use sc-shortage-comms).
---

## Overview

Determines the correct owner for a supply chain exception based on the documented routing rules, item category ownership model, site assignments, exception type, and priority level. Resolves owner identities, checks availability, presents the routing plan for confirmation, and then sends Teams notifications to the assigned owner and operations channel. Creates SLA deadline calendar holds and updates the tracker with the assignment.

This skill operates in "AI act within policy" mode — it follows the approved routing rules and ownership model to determine assignments and executes bounded notification actions after planner confirmation. Critical-priority routing simultaneously notifies the operations manager.

## When to Use

- An exception has been classified and impact-assessed and is ready for owner assignment
- A planner needs to determine who handles a specific item category or site
- A reroute is needed after reclassification or priority change
- An assigned owner is unavailable and the exception needs reassignment

## When NOT to Use

- Creating a new exception case record — use sc-shortage-intake
- Gathering demand, inventory, and shipment context — use sc-context-packet
- Classifying the exception type and likely cause — use sc-classify-exception
- Assessing business impact and recommending mitigation path — use sc-impact-assess
- Drafting shortage summaries or follow-up communications — use sc-shortage-comms
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Determine exception owner and check availability", activeForm="Identifying exception owner")
TaskCreate(subject="Route exception and send notifications", activeForm="Routing to owner")
```

### Step 1: Read Routing Inputs

**Read the exception case:**
- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — Case ID, Item/SKU, Site, Exception Type, Root Cause, Priority, Customer Impact Flag, SLA Deadline

**Read the routing rules and ownership model:**
- `SearchM365(sources=["files"], query="supply chain routing rules")` then `ReadFileContent` — item category to team mapping, site ownership assignments, exception type routing overrides
- `SearchM365(sources=["files"], query="supply chain operating model")` then `ReadFileContent` — team structure, escalation paths, on-call assignments

### Step 2: Determine Owner

Match the exception to the correct owner based on routing rules:

| Routing Factor | How It Affects Assignment |
|---------------|------------------------|
| **Item category** | Primary routing key — determines which planner or team owns the item |
| **Site / warehouse** | Secondary routing key — may override item category for site-specific ownership |
| **Exception type** | May route to specialized team (logistics for transit issues, procurement for supplier issues, quality for holds) |
| **Priority level** | Critical and high may route to senior planners or team leads rather than standard assignees |
| **Escalation flag** | Escalated cases route to operations manager in addition to the functional owner |

**Routing decision matrix:**

| Exception Type | Primary Owner | Escalation Path |
|---------------|---------------|----------------|
| **Stockout** | Supply planner (by item category) | Operations manager if customer-impacting |
| **Supplier delay** | Procurement liaison (by supplier) | Procurement manager if strategic supplier |
| **Quality hold** | Quality analyst (by site) | Quality manager if production-impacting |
| **Demand spike** | Demand planner (by item category) | Sales operations if customer-committed |
| **Logistics disruption** | Logistics coordinator (by lane) | Logistics manager if multi-shipment |
| **Production disruption** | Production planner (by site) | Operations manager if customer-impacting |

### Step 3: Resolve Owner Identity

- `SearchPeople` — resolve the owner by name, role, and team from the routing rules
- `GetUserDetails` — verify the owner's profile, role, and current status

### Step 4: Check Owner Availability

- `ListCalendarView` — check the assigned owner's calendar for availability
- If the owner is in meetings, OOO, or unavailable:
  - Check for a designated backup in the routing rules or operating model
  - If no backup documented, escalate to the team lead or operations manager for manual assignment
  - For critical priority: do not delay — simultaneously notify the backup and the operations manager

### Step 5: Present Routing Plan

Present the routing plan via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Item/SKU, Site, Exception Type, Priority, SLA Deadline
- **Assigned owner** — name, role, team, availability status
- **Routing rationale** — which routing rule determined this assignment (item category, site, exception type)
- **Escalation notifications** — operations manager notification for critical priority; additional notifications for flagged cases
- **SLA status** — time remaining on the SLA deadline
- **Next action** — recommended first step for the assigned owner based on the exception type and mitigation path
- **Routing plan label** — "ROUTING PLAN — planner confirmation required before sending notifications"

### Step 6: Execute Routing (After Confirmation)

**Send Teams notification to assigned owner:**
- `PostMessage` — direct message with:
  - Case ID, Priority, and SLA Deadline (prominent)
  - Item/SKU, Site, Exception Type, Root Cause
  - Customer Impact Flag and affected order summary
  - Recommended mitigation path (from impact assessment)
  - Link to the shortage tracker case
  - Expected next action

**Post to supply operations channel:**
- `PostMessage` — channel update with:
  - Case ID, Priority, Item/SKU, Site
  - Assigned owner
  - Brief exception summary
  - SLA deadline

**For critical priority — additional notifications:**
- `PostMessage` — direct message to operations manager with full case context
- Flag for escalation bridge if criteria are met

**Create SLA deadline calendar hold:**
- `CreateEvent` — calendar reminder for the assigned owner:
  - Critical: 1-hour deadline hold
  - High: 4-hour deadline hold
  - Medium: end-of-day hold
  - Low: next-business-day hold

**Update the shortage tracker:**
- Assigned Owner field
- Routing Timestamp
- Status: "Routed — Awaiting Mitigation"
- Routing Rationale

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find shortage tracker, routing rules, operating model |
| ReadFileContent | Read routing rules, ownership model, tracker data |
| SearchPeople | Resolve owner by item category, site, or function |
| GetUserDetails | Verify owner profile and role |
| ListCalendarView | Check owner availability |
| PostMessage | Send Teams notification to owner, operations channel, and operations manager |
| CreateEvent | Create SLA deadline calendar holds |
| render_ui (Adaptive Card) | Present the routing plan for confirmation |

## Guardrails

- **Route only to individuals listed in the documented ownership model** — never assign to someone outside the routing rules
- **For critical-priority exceptions, simultaneously notify the operations manager** in addition to the assigned owner — critical cases require management visibility
- **Escalate to operations manager if no clear owner is found** in the routing rules — never guess or assign to a default queue without documented authority
- **Verify owner availability before routing** — sending assignments to OOO or unavailable owners delays triage within the SLA window
- **Include the SLA deadline prominently** in every routing notification — the owner needs to know their response window
- **Never auto-close or auto-resolve a case through routing** — routing is assignment, not resolution; the owner handles mitigation
- **Log every routing decision** with rationale, assigned owner, timestamp, and confirming user in the shortage tracker
- **Flag reroutes explicitly** — if the case was previously assigned and is being rerouted, note the prior assignment and reason for change
- **Never include customer-specific order details** in broad channel notifications — scope to need-to-know
- **Include the recommended mitigation path** in the routing notification — the owner needs to know the suggested starting point, not just the problem
