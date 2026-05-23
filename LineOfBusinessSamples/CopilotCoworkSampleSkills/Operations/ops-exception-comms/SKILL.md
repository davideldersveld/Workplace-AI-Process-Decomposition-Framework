---
name: ops-exception-comms
description: |
  Drafts follow-up communications, handoff summaries, escalation notices,
  and status updates for operational exception cases.
  Use when user asks to "draft follow-up for exception [ID]",
  "prepare handoff summary for [case]",
  "exception status update for [case]",
  "handoff to next shift", "missing information request for [case]",
  "escalation notice for [exception]",
  "shift summary for [queue]",
  or "batch queue summary".
  Do NOT use for creating a new exception case (use ops-exception-intake),
  gathering transaction context (use ops-context-packet),
  classifying exception type (use ops-classify-exception),
  assessing impact and priority (use ops-impact-assess),
  or routing to an owner or queue (use ops-route-exception).
---

## Overview

Drafts follow-up communications, handoff summaries, escalation notices, and status updates for operational exception cases. Supports multiple communication types — missing information requests, shift handoff summaries, escalation notices to team leads, status updates to stakeholders, and batch queue summaries for queue managers. Matches urgency tone to priority level and ensures safety classifications are prominently included.

This skill operates in "AI draft plus approve" mode — every communication is presented as a draft for analyst review. Formal communications are created as Outlook drafts; Teams channel updates are posted only after explicit confirmation.

## When to Use

- An exception case needs a follow-up request for missing information
- A shift change requires a handoff summary of open cases and pending actions
- An escalation needs a formal notice to the team lead with full case context
- A stakeholder needs a status update on resolution progress
- A queue manager needs a batch summary of open cases, aging, and SLA compliance

## When NOT to Use

- Creating a new exception case — use ops-exception-intake
- Gathering process and transaction context — use ops-context-packet
- Classifying exception type and likely cause — use ops-classify-exception
- Assessing impact, priority, and aging risk — use ops-impact-assess
- Assigning an owner or routing to a queue — use ops-route-exception
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and identify communication type", activeForm="Reading exception case data")
TaskCreate(subject="Draft exception communication", activeForm="Drafting exception communication")
```

### Step 1: Read Case Data

**Read the exception case:**
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent` — Case ID, Exception Type, Priority, Status, Assigned Owner, Aging, SLA Deadline, Safety Flag, Reopen Count

**Read communication templates (if available):**
- `SearchM365(sources=["files"], query="exception communication templates")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="handoff summary template")` then `ReadFileContent`

### Step 2: Identify Communication Type

Determine the appropriate communication type based on the user's request:

| Communication Type | Purpose | Audience | Channel |
|-------------------|---------|----------|---------|
| **Missing information request** | Request specific data needed to resolve the exception | Source team, stakeholder, or external party | Outlook draft (formal) |
| **Shift handoff summary** | Transfer case ownership to the incoming shift | Next-shift analyst or team | Teams message to queue channel |
| **Escalation notice** | Alert team lead to a case requiring escalation | Team lead, operations manager | Outlook draft (formal) + Teams message |
| **Status update** | Inform stakeholder on resolution progress | Requesting stakeholder, affected customer team | Outlook draft (formal) |
| **Batch queue summary** | Overview of open cases, aging, and SLA compliance | Queue manager, team lead | Teams message to management channel |

### Step 3: Draft the Communication

#### Missing Information Request

Draft content:
- Case ID and transaction reference
- What information is missing and why it is needed
- Who is being asked to provide it
- Deadline for response (aligned with case SLA)
- Impact of delay on resolution timeline

Channel: `CreateDraftMessage` — Outlook draft for formal request

#### Shift Handoff Summary

Draft content:
- **Open cases** — each case with Case ID, Exception Type, Priority, Status, Aging, SLA Deadline, Assigned Owner
- **Pending actions** — specific next steps required for each open case
- **Escalated cases** — cases awaiting team lead or manager decision
- **SLA at risk** — cases approaching or past SLA deadline
- **Safety-flagged cases** — prominently listed with safety classification
- **Cases added or resolved this shift** — what changed since the last handoff

Channel: `PostMessage` — Teams message to the queue channel (after confirmation)

#### Escalation Notice

Draft content:
- Case ID, Transaction Reference, Exception Type, Priority
- Reason for escalation (SLA breach, safety exception, financial threshold, circular routing, reopened with failed prior resolution)
- Full case summary including classification, impact assessment, and routing history
- Aging status and SLA compliance
- Recommended action
- Safety flag prominently included if applicable

Channel: `CreateDraftMessage` — Outlook draft for formal escalation; `PostMessage` — Teams message to team lead (after confirmation)

#### Status Update

Draft content:
- Case ID and transaction reference
- Current status and resolution progress
- Expected timeline for resolution
- Any blockers or dependencies
- Next steps

Channel: `CreateDraftMessage` — Outlook draft for stakeholder communication

#### Batch Queue Summary

Draft content:
- **Queue overview** — total open cases, cases by priority, cases by exception type
- **Aging distribution** — cases within SLA, approaching SLA, breached SLA
- **Safety-flagged cases** — count and status
- **Circular routing cases** — cases flagged for queue manager review
- **Reopened cases** — cases that were previously closed and reopened
- **Staffing** — cases per analyst, unassigned cases
- **Trend** — comparison to prior shift or prior day (if data available)

Channel: `PostMessage` — Teams message to management channel (after confirmation)

### Step 4: Match Tone to Priority

| Priority | Tone and Style |
|----------|---------------|
| **Critical** | Direct, action-oriented language. Explicit SLA callout. Lead with the required action and deadline. |
| **High** | Clear urgency with specific timeline. Highlight downstream impact. |
| **Medium** | Standard professional tone. Include timeline expectations. |
| **Low** | Informational tone. Can be batched with other updates. |

### Step 5: Present Draft for Review

Present the draft via Adaptive Card (invoke `render-ui` skill first):

- **Communication type** — Missing Info Request, Handoff, Escalation, Status Update, or Batch Summary
- **Recipient** — who will receive the communication
- **Channel** — Outlook draft or Teams message
- **Draft content** — full text of the communication
- **Safety flag** — prominently displayed if the case has a safety classification
- **Draft label** — "DRAFT COMMUNICATION — review before sending"

### Step 6: Send Communication (After Confirmation)

**For Outlook drafts:**
- `CreateDraftMessage` — create the draft in Outlook for analyst review and manual send

**For Teams messages:**
- `PostMessage` — post to the appropriate channel or direct message (after explicit confirmation)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, communication templates, handoff formats |
| ReadFileContent | Read case data, templates, tracker |
| CreateDraftMessage | Create Outlook drafts for formal communications |
| PostMessage | Post Teams updates to queue channels (after confirmation) |

## Guardrails

- **Always create formal communications as Outlook draft** — never send without explicit analyst confirmation
- **Teams channel updates may be posted after explicit confirmation** — operational coordination messages require analyst approval before posting
- **Match urgency tone to priority level** — critical cases use direct, action-oriented language with explicit SLA callouts; do not use casual tone for critical or safety exceptions
- **Include Case ID, priority, and aging status in every communication** — recipients need to quickly assess urgency
- **For safety-critical exceptions, include safety classification prominently** in all communications — never omit the safety flag for brevity or formatting
- **Handoff summaries must include ALL open action items and pending dependencies** — never summarize away unresolved items; a missing item in a handoff creates a gap in case continuity
- **Never include personally identifiable customer information** in broad queue or channel updates — scope PII to need-to-know communications (direct messages to assigned resolver or formal stakeholder updates)
- **Escalation notices must include full case context** — the team lead needs complete information to make a disposition decision; do not send a thin escalation notice
- **Never fabricate resolution timelines or progress** — if resolution status is unknown, state that rather than estimating
- **Batch queue summaries must include safety-flagged and SLA-breached cases prominently** — these are the highest-priority items for the queue manager
- **Never auto-send any communication** — all communications are drafts or require explicit confirmation before posting
- **Never omit the case reference number** from any communication — every message must be traceable to a specific exception case
