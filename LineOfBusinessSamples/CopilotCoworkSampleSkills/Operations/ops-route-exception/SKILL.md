---
name: ops-route-exception
description: |
  Routes an operational exception to the correct owner or queue based
  on routing rules, exception type, priority, and the operating model.
  Use when user asks to "route exception [ID]",
  "assign this case", "who handles [exception type]",
  "send to [team] queue", "escalate exception [ID]",
  "assign owner for [case]",
  or "route [case] to the right team".
  Do NOT use for creating a new exception case (use ops-exception-intake),
  gathering transaction context (use ops-context-packet),
  classifying exception type (use ops-classify-exception),
  assessing impact and priority (use ops-impact-assess),
  or drafting follow-up communications (use ops-exception-comms).
---

## Overview

Routes an operational exception to the correct owner or queue based on the documented routing rules, exception classification, priority level, and operating model. Resolves owner identity, sends Teams assignment notifications, creates SLA deadline calendar holds, and updates the exception tracker. For critical-priority and safety-critical exceptions, also notifies the team lead and operations manager respectively.

This skill operates in "AI act within policy" mode — it executes routing within pre-approved routing rules and the documented operating model. The routing decision follows documented rules; the analyst confirms before notifications are sent.

## When to Use

- An exception has been classified and prioritized and needs to be assigned to a resolver
- A case needs reassignment to a different queue or specialist
- An escalation needs to be routed to a team lead or operations manager
- A queue manager needs to rebalance assignments across the team

## When NOT to Use

- Creating a new exception case — use ops-exception-intake
- Gathering process and transaction context — use ops-context-packet
- Classifying exception type and likely cause — use ops-classify-exception
- Assessing impact, priority, and aging risk — use ops-impact-assess
- Drafting follow-up or handoff communications — use ops-exception-comms
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read routing rules and determine assignment", activeForm="Determining exception routing")
TaskCreate(subject="Send assignment notifications and update tracker", activeForm="Routing exception")
```

### Step 1: Read Routing Inputs

**Read the exception case data:**
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent` — Case ID, Exception Type, Priority, Handling Path, Safety Flag, Queue, Reopen Count, Prior Assignments

**Read the routing rules:**
- `SearchM365(sources=["files"], query="routing rules")` then `ReadFileContent` — assignment rules by exception type, queue, priority, and function
- `SearchM365(sources=["files"], query="queue definitions")` then `ReadFileContent` — queue membership, ownership, and capacity
- `SearchM365(sources=["files"], query="operating model")` then `ReadFileContent` — team structure, specialization areas, escalation paths

### Step 2: Determine Assignment

Match the exception to the correct owner or queue based on routing rules:

| Routing Factor | How It Determines Assignment |
|---------------|------------------------------|
| **Exception type** | Different types route to different specialist teams (e.g., compliance exceptions to compliance team) |
| **Process type or queue** | Each queue has designated resolvers and a queue manager |
| **Priority level** | Critical exceptions may route directly to senior resolvers or team leads |
| **Handling path** | Specialist referrals route to domain experts; escalations route to team leads |
| **Safety flag** | Safety-critical exceptions route to the designated safety or compliance queue |

### Step 3: Resolve Owner Identity

- `SearchPeople` — resolve the assigned resolver by role, queue, or exception type
- `GetUserDetails` — verify the resolver's role and availability
- If the handling path is escalation, resolve the team lead: `GetManagerDetails`

If no clear owner is found in the routing rules, escalate to the queue manager for manual assignment.

### Step 4: Check for Circular Routing

Review the case's assignment history in the tracker:
- If this case has been routed to the **same queue more than twice**, flag as circular routing
- Circular routing indicates the exception may need reclassification, specialist intervention, or process owner review
- Present circular routing prominently for queue manager attention

### Step 5: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Transaction Reference, Exception Type, Priority, Aging, SLA Deadline
- **Recommended owner** — name, role, queue, and routing rationale
- **Routing rule citation** — the specific rule from the routing document that determines this assignment
- **Additional notifications** — team lead (for critical priority), operations manager (for safety exceptions)
- **Circular routing flag** — if the case has been routed to the same queue more than twice
- **SLA deadline** — based on priority level (30 min critical, 2 hrs high, 4 hrs medium, 1 business day low)
- **Draft label** — "ROUTING RECOMMENDATION — confirm before sending notifications"

### Step 6: Execute Routing (After Confirmation)

**Send Teams assignment notification:**
- `PostMessage` — direct message to the assigned resolver with:
  - Case ID and transaction reference
  - Exception type, priority, and safety flag (if applicable)
  - SLA deadline
  - Brief case summary (from context packet)
  - Expected next action based on exception type and handling path

**Send additional notifications for elevated cases:**
- **Critical priority:** `PostMessage` — notify the team lead in addition to the assigned resolver
- **Safety-critical exceptions:** `PostMessage` — notify the operations manager and route to the designated safety or compliance queue
- **Circular routing:** `PostMessage` — notify the queue manager with the routing history

**Create SLA deadline calendar hold:**
- `CreateEvent` — calendar reminder for the assigned resolver with:
  - SLA deadline based on priority level
  - Case ID and brief summary in the event body
  - Event title: "[Priority] Exception [Case ID] — SLA Deadline"

**Update the exception tracker:**
- Assigned Owner, Queue, Routing Date, Routing Rationale, SLA Deadline

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, routing rules, queue definitions, operating model |
| ReadFileContent | Read routing rules, queue definitions, operating model, tracker |
| SearchPeople | Resolve owner by exception type, queue, or function |
| GetUserDetails | Verify resolver availability and role |
| GetManagerDetails | Resolve team lead for escalation paths |
| PostMessage | Send Teams assignment notifications |
| CreateEvent | Create SLA deadline calendar holds |

## Guardrails

- **Route only to individuals or queues listed in the documented routing rules** — never assign outside the operating model
- **Present routing recommendation for confirmation** before sending any notifications — routing decisions are visible before execution
- **For critical-priority exceptions**, simultaneously notify the team lead in addition to the assigned resolver
- **For safety-critical exceptions**, route to the designated safety or compliance queue regardless of standard routing rules, and notify the operations manager
- **Detect circular routing** — if a case has been routed to the same queue more than twice, flag for queue manager review; do not continue routing to the same destination
- **Escalate to queue manager** if no clear owner is found in the routing rules — never leave a case unassigned
- **Never auto-close or auto-resolve** a case through routing — routing is assignment, not resolution
- **Log every routing decision** with rationale, assigned owner, queue, and timestamp for audit trail
- **Include exception type, priority, and safety flag in every assignment notification** so resolvers can prioritize effectively
- **Never include full case detail in Teams notifications** — provide a structured summary with case reference; full details are in the tracker and case workspace
- **SLA deadline calendar holds must match the priority level** — 30 minutes for critical, 2 hours for high, 4 hours for medium, 1 business day for low
- **Never modify the exception classification or priority** during routing — routing assigns ownership; changes to classification or priority require rerunning the respective skills
