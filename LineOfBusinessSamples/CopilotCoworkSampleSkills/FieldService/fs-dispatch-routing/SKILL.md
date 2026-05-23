---
name: fs-dispatch-routing
description: |
  Routes a work order to the correct technician or dispatch queue based on the
  skill matrix, routing rules, region assignment, and technician availability.
  Use when user asks to "assign technician for work order [ID]",
  "route to dispatch queue", "who should handle this job",
  "dispatch this work order", "route work order [ID]",
  "assign this job", "technician assignment for [case]",
  or "queue assignment for work order [ID]".
  Do NOT use for creating a new work order (use fs-workorder-intake),
  assembling context (use fs-dispatch-packet),
  classifying blockers (use fs-blocker-classifier),
  assessing urgency (use fs-urgency-assessment),
  or drafting communications (use fs-dispatch-comms).
---

## Overview

Reads the skill matrix and routing rules from SharePoint, cross-references against the work-order classification, urgency, blocker status, and technician availability, and routes the work order to the correct technician or dispatch queue. Sends Teams notifications to the assigned technician and posts a work-order card to the dispatch channel.

This skill operates in "AI act within policy" mode — routing is determined strictly by the approved skill matrix and routing rules. Emergency and safety-flagged work orders are presented for dispatch lead review before routing.

## When to Use

- A work order has been classified and urgency assessed, and is ready for technician assignment
- The user wants to determine which technician or queue should handle a work order
- A work order needs to be rerouted after reclassification or escalation

## When NOT to Use

- Creating a new work order — use fs-workorder-intake
- Assembling asset, location, and parts context — use fs-dispatch-packet
- Classifying work type and identifying blockers — use fs-blocker-classifier
- Assessing urgency and dispatch path — use fs-urgency-assessment
- Drafting customer or technician communications — use fs-dispatch-comms

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read routing rules and work-order data", activeForm="Reading routing inputs")
TaskCreate(subject="Determine technician assignment and check availability", activeForm="Determining assignment")
TaskCreate(subject="Send routing notifications", activeForm="Sending notifications")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Work-order data** — from the tracker: `SearchM365(sources=["files"], query="work order tracker")`
- **Classification and blockers** — work type, blocker status, readiness score from fs-blocker-classifier
- **Urgency** — urgency level, dispatch path, escalation conditions from fs-urgency-assessment
- **Skill matrix** — `SearchM365(sources=["files"], query="technician skill matrix")` or `SearchM365(sources=["files"], query="skill certification matrix")`
- **Routing rules** — `SearchM365(sources=["files"], query="dispatch routing rules")` or `SearchM365(sources=["files"], query="region assignment")`

Read each document using `ReadFileContent`.

### Step 2: Determine Technician Assignment

Apply the routing rules to determine assignment:

| Factor | How It Affects Routing |
|--------|----------------------|
| Work type | Maps to required skills and certifications in the skill matrix |
| Urgency level | Emergency and High may route to senior technicians or on-call |
| Service region | Work order must be assigned to a technician in the correct geographic region |
| Skill match | Technician must hold all required certifications for the work type |
| Availability | Technician must have calendar availability for the requested window |
| Workload | Consider current job count if visible in the tracker |
| Safety flags | Safety-flagged work orders require technicians with specific safety certifications |

**Resolve technician candidates:**
- `SearchPeople(query="<skill or role>")` to find technicians with required skills
- `GetDirectReportsDetails(user_id="<dispatch lead>")` to see the technician team roster
- `GetUserDetails(user_id="<technician>")` to confirm technician profile and region
- `ListCalendarView(start=..., end=...)` to check technician availability for the requested appointment window and a reasonable buffer

**Select the best match:**
1. Filter to technicians with required certifications
2. Filter to the correct service region
3. Filter to those available in the requested window
4. If multiple candidates, prefer the technician with fewest active jobs, then most relevant experience

**For emergency or safety-flagged work orders:**
Present the routing recommendation to the dispatch lead for review before sending notifications. Include urgency rationale, safety flags, and certification verification.

### Step 3: Send Routing Notifications

After routing is confirmed (automatically for standard work orders, after dispatch lead review for emergency or safety-flagged):

**Teams direct message to assigned technician** (use `PostMessage`):
- Work Order ID, customer name, site address
- Work type and issue summary
- Appointment window
- Parts pickup instructions (if parts required)
- Site access notes and safety warnings
- Link to dispatch packet in SharePoint
- SLA deadline

**Channel post to dispatch channel** (use `PostChannelMessage`):
- Work-order card: Work Order ID, work type, urgency, assigned technician, appointment window
- Do not include full customer details — use Work Order ID and summary only

**Update work-order tracker:**
- Set Technician Assigned, Status to "Routed", and routing timestamp

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find work-order tracker, skill matrix, routing rules |
| ReadFileContent | Read skill matrix, routing rules, tracker |
| SearchPeople / GetUserDetails | Resolve technician identities and profiles |
| GetDirectReportsDetails | Technician team structure under dispatch lead |
| ListCalendarView | Check technician calendar availability |
| PostMessage | Teams notification to assigned technician |
| PostChannelMessage | Work-order card to dispatch channel |

## Guardrails

- **Assign only to technicians who meet all skill and certification requirements** — never bypass the skill matrix
- **Verify technician availability** via calendar before assignment — flag scheduling conflicts
- **Never auto-route emergency or safety-flagged work orders** — present recommendation for dispatch lead review first
- **Never route a work order with unresolved red blockers** — flag for blocker resolution first
- **Include parts pickup instructions and site access notes** in every technician notification
- **Check service region** — never assign a technician outside their designated region without dispatch lead approval
- **Protect customer privacy in channel posts** — use Work Order ID and summary only, not full customer details
- **Log routing decision** with rationale, assigned technician, and timestamp in the work-order tracker
