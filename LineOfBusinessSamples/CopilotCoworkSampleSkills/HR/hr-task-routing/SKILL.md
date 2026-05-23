---
name: hr-task-routing
description: |
  Assigns onboarding task owners and routes work items to the correct role or queue
  based on the gap report, responsibility matrix, and org hierarchy.
  Use when user asks to "assign onboarding tasks", "route onboarding work",
  "who handles [task] for this hire", "assign tasks for onboarding [ID]",
  "route onboarding items to owners", "send onboarding tasks to teams",
  "task assignment for [employee] onboarding", or "delegate onboarding work".
  Do NOT use for creating a new onboarding case (use hr-onboarding-intake),
  assembling readiness context (use hr-readiness-packet),
  detecting missing items or risks (use hr-gap-detection),
  drafting outreach or reminders (use hr-onboarding-comms),
  or preparing readiness summaries (use hr-readiness-summary).
---

## Overview

Reads the gap report and onboarding case data, cross-references with the responsibility matrix from SharePoint, resolves named task owners via the org hierarchy, and routes onboarding work items to the correct people via Teams notifications. Optionally creates calendar holds for key deadlines. Updates the onboarding tracker with owner assignments after user confirmation.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for HR specialist review and confirmation before any messages are sent or tracker records updated.

## When to Use

- A gap report has been completed and onboarding tasks need to be assigned to owners
- The user wants to know who is responsible for specific onboarding items
- Onboarding tasks need to be rerouted after a change in start date, location, or role

## When NOT to Use

- Creating a new onboarding case — use hr-onboarding-intake
- Assembling readiness context — use hr-readiness-packet
- Detecting missing documents or risks — use hr-gap-detection
- Drafting outreach emails or reminders — use hr-onboarding-comms
- Preparing readiness summaries — use hr-readiness-summary
- Making employment or readiness decisions — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read gap report and responsibility matrix", activeForm="Reading routing inputs")
TaskCreate(subject="Determine task owners and present recommendations", activeForm="Determining task assignments")
TaskCreate(subject="Send routing notifications", activeForm="Sending task assignments")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Onboarding case data** — from the tracker: `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent`
- **Gap report** — from hr-gap-detection output (missing items, blockers, policy-sensitive conditions)
- **Responsibility matrix** — `SearchM365(sources=["files"], query="onboarding responsibility matrix")` or `SearchM365(sources=["files"], query="RACI onboarding")` then `ReadFileContent`
- **Readiness packet** — for role, location, and contact details

### Step 2: Determine Task Owners

Based on the gap report and responsibility matrix, determine who should own each outstanding item:

| Task Category | Typical Owner | How to Resolve |
|--------------|---------------|----------------|
| Missing employee documents (ID, tax forms, policy acknowledgments) | Employee (via hiring manager) | Hiring manager from case record |
| IT provisioning (laptop, accounts, VPN, software) | IT onboarding coordinator | `SearchPeople(query="IT onboarding coordinator")` or from responsibility matrix |
| Facilities setup (badge, workspace, parking) | Facilities coordinator | `SearchPeople(query="facilities coordinator [location]")` |
| Work authorization / I-9 | HR operations specialist | `SearchPeople(query="HR operations")` or current user |
| Background check follow-up | HR operations or third-party vendor | HR operations from responsibility matrix |
| Benefits enrollment coordination | Benefits team | `SearchPeople(query="benefits coordinator")` |
| Manager confirmations (start date, team, seat) | Hiring manager | From case record; `GetUserDetails` to confirm |
| HRBP notification | HR business partner | `SearchPeople(query="HR business partner [department]")` |

**Resolve named owners:**
- `SearchPeople(query="<owner name or role>")` to find the person
- `GetUserDetails(user_id="<owner>")` to confirm profile and contact information
- `GetManagerDetails(user_id="<owner>")` for escalation path if the owner is unresponsive

**Priority assignment:**
- Critical items (work authorization, background check with approaching start date): assign with urgency flag
- High items (IT provisioning, missing ID documents): assign with deadline aligned to start date minus 2 business days
- Medium items (policy acknowledgments, facilities): assign with deadline aligned to start date

### Step 3: Present Routing Recommendations

Present via Adaptive Card (invoke `render-ui` skill first):

For each outstanding onboarding item:
- **Task description** — what needs to be done
- **Assigned owner** — name and role
- **Priority** — Critical / High / Medium
- **Deadline** — based on start date and item urgency
- **Communication channel** — Teams notification, calendar hold, or both

**Summary:**
- Total tasks to be assigned
- Number of distinct owners
- Any tasks with no clear owner (escalate to HR operations manager)
- Timeline risk if tasks are not completed by deadlines

### Step 4: Send Routing Notifications (After User Confirmation)

**Teams direct messages to task owners** (use `PostMessage`):
- Onboarding Case ID, Employee Name, Start Date
- Specific task assigned with description and deadline
- Priority indicator
- Link to onboarding folder in SharePoint for document uploads

**Calendar deadline reminders** (use `CreateEvent` — optional, for critical items):
- Event title: "Onboarding Deadline — [Task] — [Employee Name]"
- Set for the task deadline date
- Attendee: task owner

**Update onboarding tracker:**
- Set Assigned To for each task, Status to "Tasks Routed", and routing timestamp

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, responsibility matrix, readiness packet |
| ReadFileContent | Read responsibility matrix, tracker, gap report |
| SearchPeople / GetUserDetails | Resolve task owners by name, role, or function |
| GetManagerDetails / GetDirectReportsDetails | Reporting chain for escalation |
| PostMessage | Teams notifications to task owners with assignment details |
| CreateEvent | Calendar deadline reminders for critical onboarding items |

## Guardrails

- **Present routing recommendations for review** before sending any messages — this is AI draft plus approve mode
- **Never auto-assign to someone outside the responsibility matrix** — only route to people identified in the matrix or confirmed by the HR specialist
- **Escalate to the HR operations manager** if no clear owner can be found for a task
- **Include Onboarding Case ID in every communication** for audit traceability
- **Respect start date deadlines** — set task deadlines relative to the employee's start date, not the current date
- **Never route work authorization or background check tasks to the employee directly** — these go through HR operations
- **Protect employee PII in routing messages** — include only the information needed for the task owner to complete their work; never include SSN, salary, or background check details
- **Log the routing decision** with task assignments, owners, deadlines, and timestamp in the tracker
