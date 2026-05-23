---
name: pm-reviewer-router
description: |
  Routes an opportunity review packet to the appropriate product, design,
  engineering, and GTM reviewers based on the reviewer matrix and product
  area ownership.
  Use when user asks to "send this for review",
  "route to reviewers for [opportunity]",
  "distribute the review packet for [case]",
  "schedule the review for this opportunity",
  "get feedback from the team on [opportunity]",
  or "set up review for [opportunity ID]".
  Do NOT use for creating a new opportunity case (use pm-request-intake),
  gathering product and customer context (use pm-context-packet),
  detecting duplicates or clustering demand (use pm-demand-cluster),
  drafting the opportunity brief (use pm-opportunity-brief),
  or preparing the stakeholder review packet (use pm-review-packet).
---

## Overview

Determines the required reviewer group for an opportunity review packet based on the reviewer matrix, product area ownership, and opportunity scope. Resolves reviewer identities, checks availability, presents the routing plan for PM approval, and then sends Teams notifications, creates Outlook review request drafts, and schedules the review meeting. Updates the tracker with reviewer assignments and review deadlines.

This skill operates in "AI act within policy" mode — it follows the approved reviewer matrix to determine routing and executes bounded actions (notifications, calendar events) within defined rules after PM confirmation.

## When to Use

- A review packet has been prepared and approved by the PM and needs to be distributed to the reviewer group
- A review meeting needs to be scheduled with the appropriate cross-functional stakeholders
- A routing plan needs to be adjusted after a product area or scope change
- A new reviewer needs to be added to an existing review cycle

## When NOT to Use

- Creating a new opportunity case — use pm-request-intake
- Gathering product, customer, and telemetry context — use pm-context-packet
- Detecting duplicates or clustering related demand — use pm-demand-cluster
- Drafting the opportunity statement and problem framing — use pm-opportunity-brief
- Preparing the stakeholder review packet — use pm-review-packet
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Determine reviewer group and check availability", activeForm="Identifying reviewers")
TaskCreate(subject="Route review packet and schedule meeting", activeForm="Routing to reviewers")
```

### Step 1: Read Routing Inputs

**Read the opportunity case:**
- `SearchM365(sources=["files"], query="opportunity tracker")` then `ReadFileContent` — Opportunity ID, Product Area, Summary, Status, Review Deadline

**Read the reviewer matrix:**
- `SearchM365(sources=["files"], query="reviewer matrix")` then `ReadFileContent` — which roles review which product areas and opportunity types

**Read the review process guide:**
- `SearchM365(sources=["files"], query="review process guide")` then `ReadFileContent` — review timelines, scheduling rules, and escalation paths

### Step 2: Determine Required Reviewers

Match the opportunity to required reviewers based on the reviewer matrix:

| Reviewer Role | Required When | Review Scope |
|--------------|--------------|-------------|
| **Product lead** | All opportunities | Strategic fit, roadmap alignment, priority assessment |
| **Design lead** | Opportunities with UX impact | User experience implications, design feasibility, research needs |
| **Engineering lead** | Opportunities with technical implications | Technical feasibility, architecture impact, effort estimation |
| **GTM lead** | Opportunities with market or customer-facing impact | Go-to-market implications, competitive positioning, customer communication |
| **Support lead** | Opportunities driven by support volume | Support impact, customer pain severity, workaround availability |
| **Sales lead** | Opportunities driven by sales escalation or revenue impact | Revenue implications, customer retention, competitive win/loss context |
| **Group product manager** | High-impact or cross-product opportunities | Cross-product dependencies, portfolio alignment, resource allocation |

### Step 3: Resolve Reviewer Identities

- `SearchPeople` — resolve each required reviewer by role and product area
- `GetUserDetails` — verify reviewer availability and current role

If no matching reviewer is found for a required role:
- Escalate to the group product manager for manual assignment
- Note the gap in the routing plan

### Step 4: Check Reviewer Availability

- `ListCalendarView` — check each reviewer's calendar for the proposed review window
- Identify scheduling conflicts and suggest alternative times if needed

### Step 5: Present Routing Plan

Present the routing plan via Adaptive Card (invoke `render-ui` skill first):

- **Opportunity header** — Opportunity ID, Title, Product Area
- **Required reviewers** — each reviewer with name, role, review scope, and availability status
- **Review format** — meeting, async review, or hybrid (meeting + async follow-up)
- **Proposed review date/time** — based on availability analysis
- **Review deadline** — target date for reviewer feedback
- **Review packet location** — SharePoint link to the deck and evidence summary
- **Sensitive opportunity flag** — if the opportunity involves competitive intelligence or unreleased strategy, note restricted distribution
- **Routing plan label** — "ROUTING PLAN — PM confirmation required before sending notifications"

### Step 6: Execute Routing (After Confirmation)

**Send Teams notifications to each reviewer:**
- `PostMessage` — direct message to each reviewer with:
  - Opportunity title and ID
  - Review scope (what they are evaluating)
  - Review deadline
  - Link to the review packet in SharePoint
  - Link to the review meeting (if scheduled)

**Create formal review request (if needed):**
- `CreateDraftMessage` — Outlook draft for cross-functional reviewers who prefer email, with:
  - Opportunity summary
  - Review scope and deadline
  - SharePoint link to the review packet (not the document itself — enables version control)

**Schedule the review meeting:**
- `CreateEvent` — calendar event with:
  - Title: "Opportunity Review: [Opportunity Title]"
  - Attendees: all required reviewers
  - Duration: 30 minutes (15-minute deck review + 15-minute discussion)
  - Body: opportunity summary, review packet link, and agenda
  - Time: based on availability analysis

### Step 7: Update Tracker

Update the opportunity tracker:
- Reviewer Assignments: names and roles of assigned reviewers
- Review Deadline: confirmed deadline date
- Review Meeting: date and time of scheduled review
- Status: "In Review"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find opportunity tracker, reviewer matrix, review process guide |
| ReadFileContent | Read reviewer matrix, process guide, tracker |
| SearchPeople | Resolve reviewer identities by role and product area |
| GetUserDetails | Verify reviewer availability and role |
| ListCalendarView | Check reviewer calendar availability |
| PostMessage | Send Teams notifications to reviewers (after confirmation) |
| CreateDraftMessage | Create Outlook draft for formal review requests |
| CreateEvent | Schedule the review meeting |

## Guardrails

- **Only route to reviewers listed in the approved reviewer matrix** for the relevant product area — do not add reviewers outside the defined matrix without PM authorization
- **If no matching reviewer exists for a required role**, escalate to the group product manager rather than skipping the review — every required role must be covered
- **Present the routing plan for PM review** before sending any notifications or creating calendar events — routing decisions are confirmed before execution
- **Include the Opportunity ID and review deadline** in every outbound message — reviewers need traceability and urgency context
- **Never share the review packet outside the designated reviewer group** without PM authorization — especially for opportunities involving competitive intelligence or unreleased strategy
- **Attach the SharePoint link, not the document file** — this ensures reviewers always see the latest version and enables proper version control
- **For sensitive opportunities**, verify that all proposed reviewers have appropriate access to the SharePoint folder before routing — flag any access concerns to the PM
- **Never auto-schedule a review meeting without PM confirmation** — the PM approves the proposed time and attendee list
- **Never include roadmap commitments or delivery dates** in reviewer notifications — the review is for evaluation, not commitment
- **Log every routing action** — reviewer assignments, notification timestamps, meeting details, and confirming PM for audit trail
- **If a reviewer declines or is unavailable**, escalate to the PM for a replacement — do not skip a required review role
- **Check for scheduling conflicts** before proposing a review time — a meeting that conflicts with reviewers' existing commitments wastes everyone's time
