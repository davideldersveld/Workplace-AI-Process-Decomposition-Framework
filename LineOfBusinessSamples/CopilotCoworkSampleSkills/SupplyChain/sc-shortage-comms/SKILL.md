---
name: sc-shortage-comms
description: |
  Drafts shortage status updates, cross-functional notifications,
  supplier follow-ups, and handoff summaries for a supply chain
  exception case.
  Use when user asks to "draft shortage update for [case]",
  "prepare handoff summary for [ID]",
  "send follow-up on [item] shortage",
  "shortage status email for [case]",
  "cross-functional update on disruption",
  or "supplier follow-up for [item]".
  Do NOT use for creating a new exception case (use sc-shortage-intake),
  gathering demand and inventory context (use sc-context-packet),
  classifying the exception type (use sc-classify-exception),
  assessing business impact (use sc-impact-assess),
  or routing to an owner (use sc-route-exception).
---

## Overview

Drafts communication artifacts for supply chain exception cases: cross-functional status updates, supplier follow-up requests, customer service advisories, shift handoff summaries, and escalation notices. Matches tone and content to audience, enforces strict separation between internal and external content, and ensures every communication includes the case reference and priority for traceability.

This skill operates in "AI draft plus approve" mode — every communication is generated as a draft for planner review. Formal emails are created as Outlook drafts; Teams channel updates are posted after explicit confirmation.

## When to Use

- A shortage case needs a cross-functional status update for the planning, logistics, or procurement teams
- A supplier needs a follow-up request for delivery commitment update or recovery plan
- Customer service needs an advisory about a shortage affecting customer orders
- A shift change or team transition requires a handoff summary of open shortage cases
- An escalation notice needs to be sent to the operations manager with full case context
- A follow-up update is needed on a previously communicated shortage

## When NOT to Use

- Creating a new exception case record — use sc-shortage-intake
- Gathering demand, inventory, and shipment context — use sc-context-packet
- Classifying the exception type and likely cause — use sc-classify-exception
- Assessing business impact and recommending mitigation path — use sc-impact-assess
- Routing the exception to an owner — use sc-route-exception
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and communication templates", activeForm="Preparing shortage communications")
TaskCreate(subject="Draft communications", activeForm="Drafting shortage updates")
```

### Step 1: Read Communication Inputs

**Read case data:**
- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — full case record including classification, priority, assigned owner, mitigation path, SLA status

**Read context packet data:**
- Review inventory position, demand exposure, shipment status, supplier context, and customer impact

**Read communication templates:**
- `SearchM365(sources=["files"], query="shortage communication template")` then `ReadFileContent` — standard update formats
- `SearchM365(sources=["files"], query="handoff template supply chain")` then `ReadFileContent` — shift handoff format
- `SearchM365(sources=["files"], query="escalation template supply chain")` then `ReadFileContent` — escalation notice format

### Step 2: Identify Required Communications

Determine which communications are needed based on the user's request and case state:

| Communication Type | Audience | Channel | When Needed |
|-------------------|----------|---------|-------------|
| **Cross-functional status update** | Planning, logistics, procurement teams | Teams channel post | When multiple teams need visibility into the shortage status |
| **Supplier follow-up request** | Supplier contact | Outlook draft | When a delivery commitment update or recovery plan is needed from the supplier |
| **Customer service advisory** | Customer service team | Teams message or Outlook draft | When a shortage affects customer orders and service needs awareness |
| **Shift handoff summary** | Incoming shift planner | Teams message or Word document | At shift change when open cases need continuity |
| **Escalation notice** | Operations manager | Teams message | When a case meets escalation criteria or requires management decision |
| **Follow-up update** | Previously notified stakeholders | Teams or Outlook draft | When new information is available on a previously communicated shortage |

### Step 3: Draft Cross-Functional Status Update

For Teams channel updates, draft a structured status message:

- **Case header** — Case ID, Priority, Item/SKU, Site
- **Exception summary** — type, root cause, current status
- **Impact summary** — affected orders, revenue exposure, customer tiers
- **Mitigation status** — what actions are in progress, expected timeline
- **Owner and next action** — who is handling, what happens next
- **SLA status** — time remaining or SLA met/breached

### Step 4: Draft Supplier Follow-Up Request

Create an Outlook draft email (`CreateDraftMessage`):

- **To:** Supplier contact (resolved from case data or user input)
- **Subject:** "Delivery Update Request: [Item/SKU] — PO [reference]"
- **Body:**
  - Reference to the original order and expected delivery date
  - Request for updated delivery commitment or recovery plan
  - Specific questions about root cause, revised timeline, and interim supply options
  - Professional, relationship-appropriate tone
  - Urgency level matched to case priority

### Step 5: Draft Customer Service Advisory

For customer-impacting shortages, draft a notification for the customer service team:

- **Affected items and scope** — which items, which customer orders
- **Expected impact** — delivery delays, partial fulfillments, backorder status
- **Customer communication guidance** — what to tell customers, what not to promise
- **Workaround or alternative** — substitute items, expedited shipping options
- **Resolution timeline** — expected recovery date (general, not specific)
- **Escalation path** — who to contact if customer pushes back

**Important:** This is an internal advisory to the service team, not a direct customer communication. Customer-facing language is suggested as guidance for the service team, not as auto-generated customer messages.

### Step 6: Draft Shift Handoff Summary

For shift changes or team transitions, produce a summary of open cases:

- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — all open cases assigned to the outgoing planner or team

**Handoff content per case:**
- Case ID, Priority, Item/SKU, Site
- Current status and last action taken
- Next expected action and timeline
- Key contacts and dependencies
- Any pending decisions or approvals

**Format options:**
- Teams message for quick handoffs
- Word document (invoke `docx` skill) for formal shift transitions with multiple open cases

### Step 7: Draft Escalation Notice

For cases meeting escalation criteria, draft a notice to the operations manager:

- `PostMessage` (after confirmation) — Teams message with:
  - Case ID, Priority, Item/SKU, Site
  - Full exception summary with classification and root cause
  - Customer impact assessment with revenue exposure
  - Mitigation options evaluated and recommended path
  - Why this case requires management decision
  - Recommended action from the planner

### Step 8: Present Drafts for Review

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Communications prepared** — list of draft types with audience and channel
- **Content separation check** — confirmation that internal details (allocation data, supplier capacity, internal priority) do not appear in customer-facing or supplier communications
- **Tone check** — operational and direct for internal teams; professional and relationship-aware for suppliers; guidance-oriented for customer service
- **Priority alignment** — communication urgency matches the case priority level
- **Draft label** — "SHORTAGE COMMUNICATIONS — planner review required before distribution"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find shortage tracker, communication templates, handoff templates, escalation templates |
| ReadFileContent | Read case data, templates, and tracker for open cases |
| CreateDraftMessage | Create Outlook draft emails for supplier follow-ups and formal communications |
| PostMessage | Post Teams updates to operations channels and send escalation notices (after confirmation) |
| render_ui (Adaptive Card) | Present draft communications for review |

## Guardrails

- **Always create formal communications as Outlook drafts** — never send without explicit planner confirmation
- **Teams channel updates may be posted after confirmation** for time-sensitive operational updates — but never auto-post without review
- **Never include customer-specific order details in broad distribution messages** — scope to need-to-know; use general impact descriptions for wide audiences
- **Never share supplier-confidential information** (pricing, capacity data, contract terms) in customer-facing or broad internal communications
- **Never share internal priority classifications or allocation data** with suppliers or customers — these are internal operational details
- **Match urgency tone to priority level** — critical cases use direct, action-oriented language; medium/low cases use standard operational tone
- **Include the Case ID and priority in every communication** for traceability and urgency context
- **Never make delivery commitments or recovery timeline promises** in any communication — use language like "we are actively working on mitigation" and "we will provide an update by [general timeframe]"
- **Customer service advisories are internal guidance, not customer communications** — never auto-generate direct customer messages; provide talking points for the service team
- **Never auto-send any communication** — all outputs are drafts or require explicit confirmation before posting
- **Flag if case data has been updated since the communication was drafted** — stale communications may not reflect the latest status
