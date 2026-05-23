---
name: sales-approval-routing
description: |
  Routes a proposal package to the required pricing, legal, product,
  and executive approvers based on the approval matrix.
  Use when user asks to "route proposal for approval",
  "send for deal desk review",
  "request pricing approval for [proposal ID]",
  "submit for legal review",
  "approval routing for [account]",
  or "get approvals for [proposal]".
  Do NOT use for creating a new proposal case (use sales-proposal-intake),
  gathering account context (use sales-context-packet),
  extracting RFP requirements (use sales-rfp-extraction),
  mapping requirements to approved content (use sales-content-mapping),
  or drafting the proposal response (use sales-proposal-draft).
---

## Overview

Determines the required approvers for a proposal package based on the organization's approval matrix, deal characteristics (deal size, product mix, special terms, non-standard pricing), and proposal content flags. Resolves approver identities, presents the routing plan for confirmation, and then sends Teams notifications and Outlook draft emails to each required approver with deal context and review scope. Creates calendar deadline holds for approval due dates and updates the proposal tracker with approval status.

This skill operates in "AI act within policy" mode — it follows the approved approval matrix and routing rules to determine required approvers and executes bounded notification actions after proposal manager confirmation.

## When to Use

- A proposal draft is complete and needs to be routed to required approvers before submission
- The proposal manager needs to determine which approvers are required for a specific deal
- A proposal has been revised and needs re-approval from specific reviewers
- An approval deadline is approaching and reminders need to be sent

## When NOT to Use

- Creating a new proposal case record — use sales-proposal-intake
- Gathering account history and content context — use sales-context-packet
- Extracting requirements from the RFP document — use sales-rfp-extraction
- Mapping requirements to approved content — use sales-content-mapping
- Drafting the proposal response document or deck — use sales-proposal-draft
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Determine required approvers", activeForm="Identifying required approvals")
TaskCreate(subject="Route proposal and send notifications", activeForm="Routing for approval")
```

### Step 1: Read Routing Inputs

**Read the proposal case data:**
- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent` — Proposal ID, Account Name, Deal Size, Product Areas, Complexity Rating, Due Date, Status

**Read the approval matrix:**
- `SearchM365(sources=["files"], query="approval matrix")` then `ReadFileContent` — deal size thresholds, product category rules, special terms triggers, and required approver roles

**Read the routing rules:**
- `SearchM365(sources=["files"], query="deal desk routing rules")` then `ReadFileContent` — routing policies, escalation paths, and approval sequence requirements

**Read the draft proposal status:**
- Verify that the proposal draft exists and has been reviewed by the proposal manager
- Check for content flags (pricing placeholders, legal placeholders, non-standard terms, new content sections)

### Step 2: Determine Required Approvers

Apply the approval matrix to the deal characteristics:

| Approval Type | Trigger Criteria | Approver Role |
|--------------|-----------------|---------------|
| **Deal desk / pricing** | All proposals with pricing components; elevated for deals above standard discount thresholds | Deal desk manager or pricing analyst |
| **Legal** | Proposals with non-standard terms, custom contractual language, indemnification modifications, or data handling requirements | Legal counsel |
| **Product / technical** | Proposals with custom implementation, non-standard configurations, or product capability commitments | Product manager or solutions architect |
| **Executive sponsor** | Deals above executive threshold, strategic accounts, or competitive displacement opportunities | VP Sales or regional sales leader |
| **Finance** | Deals with non-standard payment terms, multi-year commitments, or revenue recognition implications | Finance approver |
| **Security / compliance** | Proposals addressing security certifications, compliance requirements, or data residency commitments | Security officer or compliance lead |

### Step 3: Resolve Approver Identities

For each required approval type:
- `SearchPeople` — resolve the approver by role and organizational unit
- `GetUserDetails` — verify the approver's profile, role, and current status
- `GetManagerDetails` / `GetDirectReportsDetails` — resolve escalation paths if the primary approver is unavailable

**Check approver availability:**
- `ListCalendarView` — check each approver's calendar for availability within the approval window
- If an approver is OOO or unavailable before the RFP due date, identify the designated backup in the approval matrix

### Step 4: Define Approval Sequence and Deadlines

Determine the approval sequence based on the routing rules:

| Sequence | Approvals | Rationale |
|----------|----------|-----------|
| **Parallel** | Pricing and legal reviews can often run simultaneously | Saves time when approvals are independent |
| **Sequential** | Executive approval after pricing is finalized; security after technical review | Some approvals depend on prior review outputs |
| **Deadline** | Calculate approval due dates working backward from the RFP submission deadline, allowing buffer for revisions |

### Step 5: Present Routing Plan

Present the routing plan via Adaptive Card (invoke `render-ui` skill first):

- **Proposal header** — Proposal ID, Account Name, Deal Size, Due Date
- **Required approvers** — list of each approver with: name, role, approval type, availability status
- **Approval sequence** — parallel vs. sequential, with deadlines for each approval
- **Deal risk flags** — deal characteristics that triggered elevated approval requirements
- **Content flags** — pricing placeholders, legal placeholders, new content sections awaiting review
- **Timeline** — approval deadline for each reviewer, with buffer before RFP due date
- **Routing plan label** — "APPROVAL ROUTING — proposal manager confirmation required before sending notifications"

### Step 6: Execute Routing (After Confirmation)

**Send Teams notifications to each approver:**
- `PostMessage` — direct message to each approver with:
  - Proposal ID and account name
  - Their specific approval scope (pricing, legal, technical, executive, etc.)
  - Deal size and key risk flags relevant to their review
  - Link to the proposal draft in SharePoint
  - Approval deadline
  - Instructions for providing approval or feedback

**Create Outlook draft emails for formal approval requests:**
- `CreateDraftMessage` — draft email to each approver with:
  - Subject: "Approval Request: [Account Name] Proposal — [Approval Type] Review"
  - Proposal summary and review scope
  - Link to the proposal package in SharePoint
  - Deadline and expected turnaround

**Create calendar deadline holds:**
- `CreateEvent` — calendar hold for the proposal manager showing:
  - Approval deadlines for each reviewer
  - RFP submission deadline
  - Buffer time for revisions after approvals

**Update the proposal tracker:**
- Status: "In Approval Review"
- Approval status per reviewer: "Pending [Approver Name] — [Approval Type]"
- Approval deadline dates

### Step 7: Monitor and Follow Up

If the user requests follow-up on pending approvals:
- Check the proposal tracker for approval status
- Identify overdue approvals (past their deadline)
- Draft reminder messages for overdue approvers via `PostMessage`

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find proposal tracker, approval matrix, deal desk routing rules, proposal draft |
| ReadFileContent | Read the approval matrix, routing rules, tracker, and proposal status |
| SearchPeople | Resolve approver identities by role and organizational unit |
| GetUserDetails | Verify approver profiles and roles |
| GetManagerDetails / GetDirectReportsDetails | Resolve escalation paths for unavailable approvers |
| ListCalendarView | Check approver availability within the approval window |
| PostMessage | Send Teams notifications to approvers with deal context and review scope |
| CreateDraftMessage | Create formal Outlook approval request emails |
| CreateEvent | Create calendar deadline holds for approval milestones |
| render_ui (Adaptive Card) | Present the routing plan for confirmation |

## Guardrails

- **Present routing recommendations for proposal manager review before sending any messages** — approval routing decisions are confirmed before execution
- **Route based strictly on the approval matrix** — deal size thresholds, product categories, and special terms determine which approvers are required; never skip or substitute approvers
- **Never skip a required approver** even if the deal appears straightforward — approval matrix compliance is non-negotiable
- **Escalate to sales leadership if the approval matrix produces no clear routing** for a deal characteristic — never guess which approver to involve
- **Include deal size and risk flags in every approval request** — approvers need context to make informed decisions
- **Flag approver unavailability** and identify backup approvers before routing — sending approval requests to OOO reviewers delays the proposal
- **Create customer-facing communications as Outlook drafts only** — formal approval requests are created as drafts for the proposal manager to send
- **Never include internal pricing models, discount structures, or margin data** in approval notifications — include deal size range and pricing type only
- **Track approval status in the proposal tracker** — every routing decision is logged with approver, approval type, deadline, and status
- **Include the Proposal ID in every notification** for traceability across the approval workflow
- **Approval routing for deals with non-standard terms must include legal** — no exceptions to legal review for contractual modifications
