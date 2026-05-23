---
name: cm-break-comms
description: |
  Drafts counterparty communications, internal handoff summaries, escalation notices,
  and settlement deadline reminders for trade break cases.
  Use when user asks to "draft counterparty notice for [break ID]",
  "prepare handoff summary", "write internal escalation email",
  "break follow-up to [counterparty]", "settlement break communication",
  "draft desk handoff for [break]", "client servicing notification for [case]",
  "counterparty outreach for [trade exception]", or "cutoff reminder for [break]".
  Do NOT use for creating a new break case (use cm-break-intake),
  assembling trade and settlement context (use cm-context-packet),
  classifying break type or root cause (use cm-break-classifier),
  assessing settlement risk or time criticality (use cm-risk-assessment),
  or routing to desks or operations owners (use cm-break-routing).
---

## Overview

Drafts audience-appropriate communications for trade break case coordination — counterparty break notifications, internal desk handoff summaries, operations control escalation memos, client servicing notifications, and settlement deadline reminders. All counterparty-facing communications use approved templates and are created as Outlook drafts for analyst review. Internal coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for operations analyst review and confirmation before any message is sent.

## When to Use

- A counterparty needs to be notified of an SSI mismatch or settlement discrepancy
- An internal desk needs a structured handoff summary for a routed break
- Operations control needs an escalation notification for a critical or aged break
- Client servicing needs notification of a client-impacting settlement break
- A settlement cutoff deadline reminder needs to be sent to the assigned owner

## When NOT to Use

- Creating a new break case — use cm-break-intake
- Assembling trade and settlement context — use cm-context-packet
- Classifying the break type or likely root cause — use cm-break-classifier
- Assessing settlement risk, fail exposure, or time criticality — use cm-risk-assessment
- Routing to the correct desk or owner — use cm-break-routing
- Confirming triage disposition — this is always a human decision (CM-TRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read break data and determine communication type", activeForm="Reading break context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Break case data** — `SearchM365(sources=["files"], query="trade break tracker")` then `ReadFileContent`
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Break ID] OR [Trade ID]")` then `ReadFileContent`
- **Classification and risk assessment** — from tracker fields or prior skill outputs
- **Communication templates** — `SearchM365(sources=["files"], query="break communication template")` or `SearchM365(sources=["files"], query="counterparty outreach template")` then `ReadFileContent`

Determine which communication type is needed based on the case status, routing, and user request.

### Step 2: Draft Communication

**Communication type 1: Counterparty break notification**
- To: counterparty operations contact
- Tone: factual, neutral, professional
- Content: Break reference ID, Trade ID, settlement date, nature of the discrepancy (SSI mismatch, allocation issue, etc.), specific items requiring counterparty action or confirmation, deadline for response, operations contact for questions
- Template: use approved counterparty outreach template
- Channel: Outlook draft (use `CreateDraftMessage`)
- Include cutoff proximity if settlement is within 2 business days

**Communication type 2: Internal desk handoff summary**
- To: assigned desk analyst or team
- Tone: operational, structured, factual
- Content: Break ID, Trade ID, counterparty (short code), settlement date, break type and classification, priority and fail risk, evidence status, key facts from context packet, open questions, recommended next steps, SLA status, cutoff proximity
- Channel: Outlook draft (use `CreateDraftMessage`) for formal handoff; Teams message (use `PostMessage`) for urgent coordination

**Communication type 3: Operations control escalation memo**
- To: operations control or supervisor
- Tone: factual, structured, urgent where appropriate
- Content: Break ID, Trade ID, what triggered the escalation (same-day fail risk, SLA breach, repeat counterparty pattern, unresolved routing), current case status, actions taken so far, recommended resolution, cutoff proximity, regulatory timeline impact
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 4: Client servicing notification**
- To: client servicing team or relationship manager
- Tone: professional, factual, non-alarmist
- Content: Break ID, Trade ID (no client account numbers), nature of the settlement issue, expected impact on client settlement, actions being taken, expected resolution timeline, operations contact for questions
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 5: Settlement deadline reminder**
- To: assigned break owner
- Tone: urgent, concise
- Content: Break ID, Trade ID, settlement date, cutoff time, hours remaining, current break status, what must be resolved before cutoff, escalation path if resolution is not possible
- Channel: Teams message (use `PostMessage`) for immediacy; Outlook draft for formal record

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find break tracker, context packet, communication templates |
| ReadFileContent | Read case data, context packet, templates |
| CreateDraftMessage | Outlook drafts for all counterparty-facing and formal communications |
| PostMessage | Teams messages for internal desk coordination and urgent deadline reminders |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create counterparty-facing communications as Outlook draft** — never auto-send external messages; analyst must review and confirm
- **Counterparty drafts must never imply a booking correction, settlement commitment, or position disclosure** — use only factual break description and requested actions
- **Every draft must include Break ID, Trade ID, and settlement date** for traceability — all communications must be linkable to the case record
- **Internal Teams messages require user confirmation** before posting
- **Do not include client account numbers, position sizes, or pricing data** in any communication — use counterparty short codes and trade-level details only
- **Do not disclose information across information barrier boundaries** — counterparty communications must be reviewed for MNPI compliance; do not reference unrelated business line activity
- **Match tone to audience** — factual and neutral for counterparty-facing; operational and specific for internal desk handoffs; urgent and concise for deadline reminders
- **Never make settlement commitments** in any communication — do not promise settlement dates, SSI corrections, or resolution timelines; use procedural language only
- **Surface cutoff proximity** in every communication where settlement is within 2 business days — the recipient must immediately understand the time pressure
- **Use only approved templates** for counterparty communications — no freeform external messaging; all counterparty-facing language must follow approved patterns
- **Log the communication** — record communication type, recipient, Break ID, Trade ID, and timestamp in the break tracker after sending for audit trail (SEC Rule 17a-4)
- **Respect data classification** — trade details beyond what is needed for break resolution must not appear in communications; sensitive counterparty or client data follows need-to-know restrictions
