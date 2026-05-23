---
name: legal-triage-comms
description: |
  Drafts audience-appropriate communications for contract cases — triage
  summaries for counsel, missing information requests, status updates,
  escalation notices, and compliance referrals.
  Use when user asks to "draft contract summary for [case]",
  "prepare triage summary for counsel",
  "send missing info request for [case]",
  "draft follow-up to [requestor]",
  "legal case update for [case ID]",
  "escalation notice for contract [ID]",
  or "status update for [counterparty] contract".
  Do NOT use for creating a new case (use legal-contract-intake),
  assembling review context (use legal-review-packet),
  detecting clause deviations (use legal-deviation-detection),
  or determining review path (use legal-review-routing).
---

## Overview

Drafts audience-appropriate communications for contract cases throughout the triage lifecycle — triage summaries for assigned counsel, missing information requests to business requestors, status updates to stakeholders, escalation notices to senior counsel or legal operations, and compliance referral notices. All external and counsel-facing communications are created as Outlook drafts for review. Internal legal coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for analyst review and confirmation before any message is sent or document is finalized.

## When to Use

- Assigned counsel needs a triage summary with deviation overview and recommended focus areas
- A business requestor needs to provide missing contract documents or information
- A business stakeholder needs a status update on their contract request
- An escalation notice is needed for high-severity deviations or authority-threshold triggers
- A compliance referral is needed for regulatory clause deviations
- A disposition confirmation needs to be sent after the triage review is complete

## When NOT to Use

- Creating a new case record — use legal-contract-intake
- Assembling contract context or playbook materials — use legal-review-packet
- Detecting clause deviations against the playbook — use legal-deviation-detection
- Determining the review path or required approvals — use legal-review-routing
- Confirming final triage disposition — this is always a human decision (LG-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Case tracker** — `SearchM365(sources=["files"], query="contract case tracker")` then `ReadFileContent` — case data, status, assignments
- **Review packet** — `SearchM365(sources=["files"], query="review packet [Case ID]")` then `ReadFileContent` — contract context, playbook standards
- **Deviation report** — `SearchM365(sources=["files"], query="deviation report [Case ID]")` then `ReadFileContent` — deviation findings and severity
- **Communication templates** — `SearchM365(sources=["files"], query="legal communication template")` then `ReadFileContent` — approved language and formats

Determine which communication type is needed based on the case status, audience, and user request.

### Step 2: Draft Communication

**Communication type 1: Triage summary for counsel**
- Audience: assigned counsel
- Tone: formal, precise, structured
- Content: Case ID, counterparty, contract type, jurisdiction, urgency, deviation summary (count by severity level, key flagged clauses), recommended focus areas for review, review timeline target, required approvals identified, matter folder location for full materials
- Channel: Outlook draft (use `CreateDraftMessage`)
- Sensitivity: include "PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT" header

**Communication type 2: Missing information request**
- Audience: business requestor
- Tone: clear, helpful, specific
- Content: Case ID reference, what documents or information are needed (contract document, authorization, business terms, counterparty details), why the information is needed (to proceed with review), deadline for providing the information, how to submit (reply to email, upload to matter folder), contact information for questions
- Channel: Outlook draft (use `CreateDraftMessage`)
- Sensitivity: no contract terms, deviation details, or legal analysis in the request

**Communication type 3: Status update for business stakeholder**
- Audience: business requestor or business unit stakeholder
- Tone: professional, clear, non-technical
- Content: Case ID reference, current case status in plain language (received, under review, pending information, review complete), expected next steps and timeline, whether any action is needed from the requestor, contact information
- Channel: Outlook draft (use `CreateDraftMessage`)
- Sensitivity: no deviation details, no legal analysis, no internal routing information

**Communication type 4: Escalation notice**
- Audience: senior counsel, deputy general counsel, or legal operations manager
- Tone: concise, structured, action-oriented
- Content: Case ID, counterparty, contract type, urgency, reason for escalation (Level 3 or 4 deviations, authority threshold exceeded, or timeline risk), deviation severity profile, assigned counsel and current review status, recommended action (expedited review, additional authority, business approval), SLA status
- Channel: Teams direct message (use `PostMessage`) for urgent escalation; Outlook draft (use `CreateDraftMessage`) for formal escalation
- Sensitivity: include "PRIVILEGED AND CONFIDENTIAL" header; deviation summary by severity only — no clause text in the message

**Communication type 5: Compliance referral**
- Audience: compliance reviewer
- Tone: formal, factual, precise
- Content: Case ID, counterparty, contract type, regulatory clause areas requiring review (data protection, export control, anti-corruption), deviation severity for each flagged regulatory clause, matter folder reference for full deviation report
- Channel: Outlook draft (use `CreateDraftMessage`)
- Sensitivity: include "PRIVILEGED AND CONFIDENTIAL" header; reference clause areas only — no contract text in the referral

**Communication type 6: Disposition confirmation**
- Audience: business requestor
- Tone: professional, clear, reassuring
- Content: Case ID reference, final disposition (queued for full review, returned for missing information, escalated for additional approval), assigned reviewer (name only — no practice area or internal routing details), expected timeline for next steps, contact information
- Channel: Outlook draft (use `CreateDraftMessage`)
- Sensitivity: no deviation details, no internal routing rationale, no legal analysis

### Step 3: Apply Audience-Specific Language Rules

| Audience | Contract Terms | Deviation Details | Internal Routing | Legal Analysis | Privilege Header |
|----------|---------------|-------------------|-----------------|----------------|-----------------|
| Counsel | Allowed (reference) | Allowed (summary) | Allowed | Not included (counsel's role) | Required |
| Business requestor | Not allowed | Not allowed | Not allowed | Not allowed | Not required |
| Business stakeholder | Not allowed | Not allowed | Not allowed | Not allowed | Not required |
| Senior counsel / escalation | Allowed (reference) | Allowed (severity summary) | Allowed | Not included | Required |
| Compliance reviewer | Clause area reference | Severity per clause area | Allowed | Not included | Required |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, review packet, deviation report, communication templates |
| ReadFileContent | Read case data, review packet, deviation report, templates |
| CreateDraftMessage | Outlook drafts for all external and counsel-facing communications |
| PostMessage | Teams direct messages for urgent escalation and legal ops coordination |

## Guardrails

- **Always create communications as Outlook drafts** — never send without explicit analyst confirmation
- **Match tone, detail level, and sensitivity to the audience** — formal and precise for counsel; clear and non-technical for business requestors; structured and action-oriented for escalations
- **Include the Case ID reference in every communication** for traceability
- **Never include contract terms, clause text, or deviation details in communications to non-legal recipients** — business requestors and stakeholders see status and process information only
- **Never include full clause text in Teams messages or emails** — reference the matter folder for full materials
- **Mark all counsel-facing and compliance-facing drafts** with "PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT" header
- **Never include legal advice, legal analysis, or recommendations to accept or reject terms** in any communication — this skill drafts factual summaries and process communications; legal judgment belongs to counsel
- **Never disclose internal routing rationale or reviewer assignment logic** in business-facing communications — the requestor sees the assigned reviewer name and timeline, not the routing criteria
- **Never commit to review completion dates** that have not been confirmed by the assigned counsel — use language like "your contract is under review" rather than "review will be completed by [date]"
- **Never disclose other cases' details** in any communication — each requestor sees only their own case status
- **Log the communication** — record communication type, recipient, Case ID, and timestamp in the case tracker after sending for audit trail
- **If the case involves regulatory clause deviations**, ensure the compliance referral is drafted alongside the counsel summary — compliance review must not be missed
