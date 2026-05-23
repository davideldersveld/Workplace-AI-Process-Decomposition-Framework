---
name: fc-journal-comms
description: |
  Drafts journal entry reviewer summaries, missing evidence follow-up requests,
  close coordination updates, escalation notices, and rework notifications
  using the journal evidence packet and approved templates.
  Use when user asks to "draft journal summary", "prepare reviewer packet",
  "write follow-up for missing support", "journal entry summary for [reviewer]",
  "close entry review packet", "missing evidence request for [case]",
  "escalation notice for journal [ID]", or "rework notification for [entry]".
  Do NOT use for creating a new journal case (use fc-journal-intake),
  assembling ledger and policy context (use fc-journal-context),
  detecting missing evidence or risks (use fc-gap-risk-detection),
  or determining the approval path (use fc-approval-routing).
---

## Overview

Drafts audience-appropriate journal entry communications — reviewer summaries for accounting managers and controllers, missing evidence follow-up requests to requestors, close coordination updates, escalation notices for high-risk entries, and rework notifications. All communications are created as Outlook drafts for accountant review before sending.

This skill operates in "AI draft plus approve" mode — every communication is created as a draft or presented for review. Nothing is sent without explicit user confirmation.

## When to Use

- A journal entry has been routed and the reviewer needs a concise summary
- Supporting evidence is missing and the requestor needs a follow-up request
- The close coordinator needs a status update on journal cases
- A high-risk or threshold-exceeding entry needs an escalation notice
- A journal case is being returned for rework with specific items to address

## When NOT to Use

- Creating a new journal case — use fc-journal-intake
- Assembling ledger, policy, and support context — use fc-journal-context
- Detecting missing evidence or control risks — use fc-gap-risk-detection
- Determining the approval path — use fc-approval-routing
- Approving or posting a journal entry — these are always human actions

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read journal case data and identify communication type", activeForm="Reading journal materials")
TaskCreate(subject="Draft communication using appropriate template", activeForm="Drafting communication")
```

### Step 1: Read Journal Case Materials

Locate and read the journal case context:

- **Journal case data** — from the tracker: `SearchM365(sources=["files"], query="journal tracker")`
- **Journal evidence packet** — from fc-journal-context output: `SearchM365(sources=["files"], query="journal packet [Case ID]")`
- **Gap and risk report** — from fc-gap-risk-detection output
- **Communication templates** — `SearchM365(sources=["files"], query="journal communication template")` or `SearchM365(sources=["files"], query="reviewer summary template")`

Read each document using `ReadFileContent`.

### Step 2: Identify Communication Type and Draft

Determine the communication type from the user's request:

| Communication Type | Audience | Key Elements |
|-------------------|----------|-------------|
| **Reviewer summary** | Accounting manager or controller | Entry details, policy context, risk flags, evidence inventory, approval level, case reference |
| **Missing evidence request** | Requestor or accountant | Specific documents needed, why they are required, how to provide them, case reference |
| **Close coordination update** | Close coordinator | Case status, timeline, any blockers, case reference |
| **Escalation notice** | Controller or VP Finance | Entry details, risk flags, why escalation is needed, evidence summary, case reference |
| **Rework notification** | Requestor or preparer | Specific items to address, what was found, deadline for response, case reference |
| **Approval confirmation** | Requestor | Confirmation that the entry has been approved and is ready for posting, case reference |

### Customer-Facing Drafts (All via Outlook)

For all communications, create an Outlook draft using `CreateDraftMessage`:

**Reviewer summary:**
- Subject: "Journal Entry Review — [Journal Case ID] — [Entity] [Period]"
- Body: Concise entry summary (accounts, amount, entity, period, rationale), applicable policy context with citations, risk flags from the gap report, supporting evidence inventory (what is available, what was flagged), required approval level, link to evidence packet in SharePoint
- Tone: formal, precise, evidence-cited

**Missing evidence request:**
- Subject: "Supporting Documentation Needed — [Journal Case ID]"
- Body: Specific documents that are missing and why each is required (policy reference), instructions for uploading to the case folder, deadline based on close schedule, case reference
- Tone: clear, actionable, non-accusatory

**Close coordination update:**
- Subject: "Journal Case Status — [Journal Case ID] — [Period]"
- Body: Current status, what is complete, what is pending, any blockers, expected timeline to review-ready state, case reference
- Tone: operational, concise

**Escalation notice:**
- Subject: "Escalation — Journal Entry Review — [Journal Case ID]"
- Body: Entry details, reason for escalation (threshold exceeded, high-risk flags, policy exception), evidence summary, recommended action, case reference
- Tone: formal, complete, urgency-appropriate

**Rework notification:**
- Subject: "Rework Required — [Journal Case ID]"
- Body: Specific items to address (list each issue with what needs to change), reference to the gap report findings, deadline for resubmission, case reference
- Tone: constructive, specific, deadline-oriented

**Approval confirmation:**
- Subject: "Approved — [Journal Case ID] — Ready for Posting"
- Body: Confirmation that the entry has been reviewed and approved, posting instructions (human action in ERP), case reference
- Tone: confirmatory, clear next steps

### Internal Communications

**Close coordination Teams update** (via `PostMessage` after user confirmation):
- Journal Case ID, entry type, current status, timeline
- No financial details in channel posts — case ID and status only

After drafting, tell the user: "I've created a draft [type] for [recipient]. Review and send when ready."

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find journal tracker, evidence packet, communication templates |
| ReadFileContent | Read case materials, templates, gap report |
| CreateDraftMessage | Create Outlook draft for all communications (never auto-send) |
| PostMessage | Teams coordination messages for internal status updates (after confirmation) |

## Guardrails

- **Always create communications as Outlook draft** — never send without explicit user confirmation
- **Include Journal Case ID, description, amount, and period** in every communication for audit traceability
- **Cite specific policy sections and evidence references** in reviewer summaries — every assertion must trace to a source document
- **Never include preliminary or unvalidated amounts** — only use figures from the confirmed journal case record
- **Never fabricate assertions** — every finding referenced in a communication must come from the evidence packet or gap report
- **Match tone to audience** — formal and precise for controller review, operational for close coordination, constructive for rework notifications
- **Protect sensitive financial data** — do not include account balances or entry amounts in Teams channel posts; limit to case ID and status
- **Respect the AI draft plus approve boundary** — every output is reviewable; nothing is sent automatically
