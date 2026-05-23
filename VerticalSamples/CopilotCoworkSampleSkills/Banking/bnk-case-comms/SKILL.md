---
name: bnk-case-comms
description: |
  Drafts customer communications, analyst handoff summaries, affidavit requests,
  escalation notices, and case follow-ups for fraud and dispute cases.
  Use when user asks to "draft customer message for [case ID]",
  "prepare affidavit request", "send analyst summary", "fraud case follow-up",
  "dispute notification draft", "missing evidence follow-up",
  "draft escalation notice for [case]", "customer acknowledgment for dispute",
  "branch coordination message", or "case handoff note for [analyst]".
  Do NOT use for creating a new case (use bnk-case-intake),
  assembling account and transaction context (use bnk-fraud-context-packet),
  classifying fraud scenario or dispute type (use bnk-scenario-classifier),
  assessing urgency or evidence gaps (use bnk-gap-risk-detection),
  or routing to analyst queues (use bnk-case-routing).
---

## Overview

Drafts audience-appropriate communications for fraud and dispute case coordination — customer acknowledgments, affidavit requests, missing evidence follow-ups, analyst case summaries and handoff notes, supervisor escalation notifications, and branch coordination messages. All customer-facing communications use approved templates and are created as Outlook drafts for analyst review. Internal coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for fraud analyst review and confirmation before any message is sent.

## When to Use

- A customer needs acknowledgment that their dispute has been received
- A customer needs to submit an affidavit or written statement
- Missing evidence needs to be requested from the customer
- An analyst needs a case summary for handoff or review
- A supervisor needs an escalation notification
- A branch needs coordination on a case that originated there

## When NOT to Use

- Creating a new case — use bnk-case-intake
- Assembling account and transaction context — use bnk-fraud-context-packet
- Classifying the fraud scenario — use bnk-scenario-classifier
- Assessing urgency or evidence gaps — use bnk-gap-risk-detection
- Routing to an analyst queue — use bnk-case-routing
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Case data** — `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent`
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Case ID]")` then `ReadFileContent`
- **Classification and risk assessment** — from tracker fields or prior skill outputs
- **Communication templates** — `SearchM365(sources=["files"], query="fraud communication template")` or `SearchM365(sources=["files"], query="dispute customer template")` then `ReadFileContent`

Determine which communication type is needed based on the case status, evidence gaps, and user request.

### Step 2: Draft Communication

**Communication type 1: Customer dispute acknowledgment**
- To: customer (email on file)
- Tone: professional, reassuring, procedural
- Content: confirmation that the dispute has been received, Case Reference number, what happens next (investigation timeline), what the customer may need to provide (affidavit, statement), contact information for questions
- Template: use approved customer acknowledgment template
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: Affidavit or statement request**
- To: customer
- Tone: clear, helpful, specific
- Content: Case Reference, explanation of what is needed (fraud affidavit or written dispute statement), how to submit (secure portal link or mail), deadline for submission, what happens if not received, contact for questions
- Template: use approved affidavit request template
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 3: Missing evidence follow-up**
- To: customer
- Tone: helpful, clear, appropriately urgent
- Content: Case Reference, specific evidence items still needed, how to submit, revised deadline, impact of continued delay on investigation timeline
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 4: Analyst case summary and handoff note**
- To: assigned analyst or next-stage reviewer
- Tone: operational, structured, factual
- Content: Case ID, scenario classification, urgency level, evidence status, key facts from context packet, open questions, recommended next steps, SLA status, regulatory deadline status
- Channel: Outlook draft (use `CreateDraftMessage`) for formal handoff; Teams message (use `PostMessage`) for quick coordination

**Communication type 5: Supervisor escalation notification**
- To: fraud operations supervisor
- Tone: factual, structured, urgent where appropriate
- Content: Case ID, what triggered the escalation (high-value, AML indicators, SLA breach, unresolved routing), current case status, actions taken so far, recommended resolution, regulatory timeline impact
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 6: Branch coordination message**
- To: originating branch contact
- Tone: concise, operational
- Content: Case ID, current status, any additional information needed from the branch, expected resolution timeline, analyst contact for questions
- Channel: Outlook draft (use `CreateDraftMessage`) or Teams message (use `PostMessage`)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, context packet, communication templates |
| ReadFileContent | Read case data, context packet, templates |
| CreateDraftMessage | Outlook drafts for all customer-facing and formal communications |
| PostMessage | Teams messages for internal analyst coordination |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create customer communications as Outlook draft** — never send without explicit analyst confirmation
- **Never include commitment language** regarding provisional credit amounts, reimbursement timelines, investigation outcomes, or case disposition — use only procedural next-steps language approved in templates
- **Never disclose fraud investigation details** in customer-facing communications — no fraud scores, internal risk assessments, SAR status, or investigation methodology
- **Never disclose suspicious activity information** — BSA/AML regulations prohibit tipping off subjects of suspicious activity investigations
- **Use only approved templates** for customer communications — no freeform customer messaging; all customer-facing language must follow approved patterns
- **Mask account and card numbers** to last four digits in all communication outputs
- **Include the Case Reference number** in every communication for audit traceability
- **Match tone to audience** — professional and reassuring for customers; operational and structured for analysts; factual and urgent for escalations
- **Never make employment or disciplinary commitments** in branch coordination messages
- **Log the communication** — record communication type, recipient, Case ID, and timestamp in the case tracker after sending for audit trail
- **Respect data classification** — customer PII (SSN, full account numbers, date of birth) must never appear in any communication output; fraud investigation details must never appear in customer-facing drafts
