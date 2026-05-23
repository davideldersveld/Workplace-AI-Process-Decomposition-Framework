---
name: proc-supplier-comms
description: |
  Drafts outreach communications for supplier onboarding — missing
  document requests, reviewer notifications, risk escalation notices,
  AP handoff instructions, and status updates.
  Use when user asks to "draft supplier email for [supplier]",
  "remind [requester] about missing documents",
  "supplier follow-up for [case]",
  "send missing docs request for [supplier]",
  "prepare reviewer notification for [case]",
  "status update for [supplier] onboarding",
  or "AP handoff for [supplier]".
  Do NOT use for creating a new supplier case (use proc-supplier-intake),
  gathering supplier and policy context (use proc-context-packet),
  detecting missing items or risk indicators (use proc-gap-risk-detect),
  determining review path (use proc-review-routing),
  or summarizing the reviewer packet (use proc-reviewer-packet).
---

## Overview

Drafts outreach communications for supplier onboarding cases — missing document requests to requesters or supplier contacts, reviewer assignment notifications, risk flag escalation notices to category managers, AP onboarding handoff instructions, and status updates to requesters. Matches tone to audience (professional for external supplier-facing, operational for internal) and ensures sensitive data is never included in communications.

This skill operates in "AI draft plus approve" mode — every communication is presented as a draft for analyst review. Outlook emails are created as drafts; Teams messages are posted only after explicit confirmation.

## When to Use

- A supplier case has missing documents and the requester or supplier contact needs to be notified
- Reviewers have been assigned and need formal notification with case context
- A risk flag requires escalation to the category manager
- The AP team needs a handoff with banking detail collection instructions
- The requester needs a status update on onboarding progress
- A follow-up is needed for stalled or delayed items

## When NOT to Use

- Creating a new supplier case — use proc-supplier-intake
- Gathering supplier and policy context — use proc-context-packet
- Detecting missing documents or risk indicators — use proc-gap-risk-detect
- Determining the review path — use proc-review-routing
- Summarizing the case for reviewer approval — use proc-reviewer-packet
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and identify communication type", activeForm="Reading supplier case data")
TaskCreate(subject="Draft supplier communication", activeForm="Drafting supplier communication")
```

### Step 1: Read Case Data

**Read the onboarding case:**
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` — Case ID, Supplier Name, Requester, Status, Missing Items, Risk Flags, Assigned Reviewers

**Read communication templates (if available):**
- `SearchM365(sources=["files"], query="supplier onboarding communication templates")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="missing document request template")` then `ReadFileContent`

### Step 2: Identify Communication Type

| Communication Type | Purpose | Audience | Channel |
|-------------------|---------|----------|---------|
| **Missing document request** | Request specific documents needed to complete onboarding | Requester or supplier contact | Outlook draft |
| **Reviewer notification** | Notify assigned reviewer with case summary and review scope | Category manager, risk reviewer, legal, tax, AP | Teams message or Outlook draft |
| **Risk escalation notice** | Alert category manager to a supplier with elevated risk flags | Category manager, procurement manager | Outlook draft |
| **AP handoff** | Provide AP specialist with case context and banking collection instructions | AP onboarding specialist | Outlook draft |
| **Status update** | Update requester on onboarding progress | Requester | Outlook draft |
| **Follow-up reminder** | Re-request stalled items or prompt overdue reviewers | Requester, reviewer, or supplier contact | Outlook draft |

### Step 3: Draft the Communication

#### Missing Document Request

Draft content:
- Case ID and supplier name
- Specific documents that are missing with descriptions of what is needed
- Where and how to submit documents (SharePoint upload link or email)
- Deadline for submission (aligned with SLA)
- Impact of delay on onboarding timeline
- Professional, helpful tone for external audiences; operational for internal requesters

Channel: `CreateDraftMessage` — Outlook draft

#### Reviewer Notification

Draft content:
- Case ID, Supplier Name, Spend Category, Geography
- Review scope (what specifically this reviewer is evaluating)
- Risk flags summary (if applicable, without sensitive details)
- Review deadline
- Link to the supplier onboarding folder for supporting materials
- Link to the context packet document

Channel: `PostMessage` — Teams direct message (after confirmation), or `CreateDraftMessage` for formal email notification

#### Risk Escalation Notice

Draft content:
- Case ID, Supplier Name, Risk Tier
- Specific risk flags with evidence summary
- Screening status and findings
- Recommended action (enhanced due diligence, hold pending review, escalate to compliance)
- Urgency level

Channel: `CreateDraftMessage` — Outlook draft for formal escalation

#### AP Handoff

Draft content:
- Case ID, Supplier Name, payment terms context
- Checklist of banking and payment items needed
- Instructions for secure bank detail collection (reference to the secure banking form process — never include actual banking details)
- Deadline for vendor master activation

Channel: `CreateDraftMessage` — Outlook draft

#### Status Update

Draft content:
- Case ID and supplier name
- Current status (percentage complete, items pending, reviews in progress)
- Expected timeline for next milestone
- Any blockers or items requiring requester action

Channel: `CreateDraftMessage` — Outlook draft

### Step 4: Present Draft for Review

Present the draft via Adaptive Card (invoke `render-ui` skill first):

- **Communication type** — Missing Docs, Reviewer Notification, Risk Escalation, AP Handoff, Status Update, Follow-up
- **Recipient** — who will receive the communication
- **Channel** — Outlook draft or Teams message
- **Draft content** — full text
- **Sensitive data check** — confirmation that no bank details, tax IDs, or screening results are included
- **Draft label** — "DRAFT COMMUNICATION — review before sending"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, communication templates |
| ReadFileContent | Read case data, templates, tracker |
| CreateDraftMessage | Create Outlook drafts for formal communications |
| PostMessage | Post Teams messages for reviewer coordination (after confirmation) |

## Guardrails

- **Always create formal communications as Outlook draft** — never send without explicit analyst confirmation
- **Teams messages for reviewer coordination** may be posted after explicit confirmation
- **Never include bank account details, tax identifiers (EIN/SSN), or screening results** in any outgoing communication — reference the secure process or SharePoint folder instead
- **Match tone to audience** — professional and helpful for external supplier-facing communications; operational and direct for internal reviewer notifications
- **Include the onboarding case reference number** in every communication for traceability
- **Never reveal specific risk flag details to the supplier** — risk escalation notices are internal only; supplier-facing communications reference "additional documentation requirements"
- **AP handoff communications must reference the secure banking form process** — never include instructions to send bank details via email
- **Never fabricate onboarding timelines or completion estimates** — if the timeline is uncertain, state that rather than guessing
- **Follow-up reminders should include the specific items that are overdue** — vague reminders are less effective than specific document or action requests
- **Never auto-send any communication** — all communications are drafts or require explicit confirmation before posting
