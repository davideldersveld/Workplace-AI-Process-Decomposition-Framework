---
name: fs-dispatch-comms
description: |
  Drafts customer-facing communications, technician briefings, dispatch summaries,
  and escalation notifications for field service work orders using approved templates.
  Use when user asks to "draft customer update for work order [ID]",
  "prepare technician briefing", "write dispatch summary",
  "send appointment confirmation", "customer notification for [work order]",
  "technician briefing for [ID]", "reschedule notice for [customer]",
  or "dispatch escalation for [work order]".
  Do NOT use for creating a new work order (use fs-workorder-intake),
  assembling context (use fs-dispatch-packet),
  classifying blockers (use fs-blocker-classifier),
  assessing urgency (use fs-urgency-assessment),
  or routing the work order (use fs-dispatch-routing).
---

## Overview

Drafts audience-appropriate dispatch communications — customer appointment confirmations, reschedule notices, technician dispatch briefings, dispatch team escalation alerts, and post-visit follow-up requests. All customer-facing communications are created as Outlook drafts for dispatcher review before sending.

This skill operates in "AI draft plus approve" mode — every communication is created as a draft or presented for review. Nothing is sent to customers without explicit user confirmation.

## When to Use

- A work order has been routed and the customer needs an appointment confirmation
- A technician needs a detailed dispatch briefing document
- An appointment needs to be rescheduled and the customer needs notification
- A blocked work order needs dispatch team escalation
- A completed visit needs a customer follow-up request

## When NOT to Use

- Creating a new work order — use fs-workorder-intake
- Assembling asset, location, and parts context — use fs-dispatch-packet
- Classifying work type and identifying blockers — use fs-blocker-classifier
- Assessing urgency and dispatch path — use fs-urgency-assessment
- Routing the work order to a technician — use fs-dispatch-routing

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read work-order data and identify communication type", activeForm="Reading work-order materials")
TaskCreate(subject="Draft communication using appropriate template", activeForm="Drafting communication")
```

### Step 1: Read Work-Order Materials

Locate and read the work-order context:

- **Work-order data** — from the tracker: `SearchM365(sources=["files"], query="work order tracker")`
- **Dispatch context packet** — `SearchM365(sources=["files"], query="dispatch packet [Work Order ID]")`
- **Classification and blockers** — from the tracker or Adaptive Card outputs
- **Communication templates** — `SearchM365(sources=["files"], query="dispatch communication template")` or `SearchM365(sources=["files"], query="customer notification template")`
- **Recent customer correspondence** — `SearchM365(sources=["email"], query="[customer name] [site address]")`

Read each document using `ReadFileContent`.

### Step 2: Identify Communication Type and Draft

Determine the communication type from the user's request:

| Communication Type | Audience | Key Elements |
|-------------------|----------|-------------|
| **Appointment confirmation** | Customer | Date, time window, technician name, preparation instructions, work-order reference |
| **Reschedule notice** | Customer | Reason for change, new proposed window, apology, work-order reference |
| **Technician dispatch briefing** | Technician | Job summary, asset details, parts list, site access, safety notes, customer expectations, prior visit notes |
| **Dispatch escalation** | Dispatch team | Blocked work order details, unresolved blockers, urgency, recommended action |
| **Post-visit follow-up** | Customer | Service performed, any follow-up needed, how to report issues, work-order reference |

### Customer-Facing Drafts

For all customer-facing communications, create an Outlook draft using `CreateDraftMessage`:

**Appointment confirmation:**
- Subject: "Appointment Confirmation — [Work Type] — [Work Order ID]"
- Body: Confirm the appointment date and time window, technician name, what the customer should prepare (site access, equipment availability, parking), work-order reference number, how to reschedule if needed

**Reschedule notice:**
- Subject: "Appointment Update — [Work Order ID]"
- Body: Apologize for the change, explain the reason in customer-friendly terms (never expose internal blockers or system details), propose the new window, confirm next steps, work-order reference

**Post-visit follow-up:**
- Subject: "Service Visit Summary — [Work Order ID]"
- Body: Summarize work performed, note any follow-up actions or recommendations, provide instructions for reporting issues if the problem recurs, work-order reference

### Internal Communications

**Technician dispatch briefing** (via Word document, invoke `docx` skill):
- Work-order summary: ID, customer, site address, work type, issue description
- Asset details: equipment type, model, serial number, known issues
- Parts list: required parts, pickup location, special-order status
- Site access: entry instructions, parking, safety hazards, site contact and phone number
- Safety warnings: highlighted prominently if the work order involves hazardous equipment or conditions
- Customer expectations: requested window, any special instructions from the customer
- Prior visit notes: relevant findings from the last visit to this site or on this equipment

**Dispatch team escalation** (via `PostMessage` to dispatch lead after user confirmation):
- Work Order ID, customer name, urgency level
- Unresolved blockers preventing dispatch
- Appointment impact — will the window be missed?
- Recommended action and urgency of resolution
- Link to work-order tracker and dispatch packet

After drafting, tell the user: "I've created a draft [type] for [recipient]. Review and send when ready."

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find work-order tracker, dispatch packet, communication templates |
| SearchM365 (email) | Find recent customer correspondence for context |
| ReadFileContent | Read work-order materials, templates |
| CreateDraftMessage | Create Outlook draft for customer communications (never auto-send) |
| PostMessage | Teams messages to dispatch team for escalation notifications (after confirmation) |

## Guardrails

- **Always create customer-facing communications as Outlook draft** — never send without explicit user confirmation
- **Never include internal details in customer communications** — no urgency levels, blocker classifications, technician personal details beyond name, dispatch routing information, or system references
- **Never make outcome or cost commitments** in customer communications — appointment confirmations confirm timing and preparation only; cost estimates and outcome guarantees require service manager approval
- **Include safety warnings in technician briefings** when the work order involves hazardous equipment or site conditions — make them prominent, not buried
- **Include parts pickup instructions and special tooling** in technician briefings — missing parts are the leading cause of failed visits
- **Match tone to audience** — professional and clear for customers, operational and detailed for technician briefings, urgent and actionable for escalation notifications
- **Include work-order reference number** in every communication — subject and body
- **Never fabricate service details** — only include information from the dispatch packet and work-order tracker
