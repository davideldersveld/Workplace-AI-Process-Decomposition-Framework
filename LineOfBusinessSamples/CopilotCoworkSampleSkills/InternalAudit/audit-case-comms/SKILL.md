---
name: audit-case-comms
description: |
  Drafts audit engagement summaries, evidence request follow-ups, status updates,
  and escalation notices for audit case communications.
  Use when user asks to "draft audit summary for [case]",
  "prepare evidence request for [control owner]", "audit status update",
  "draft follow-up to [control owner]", "prepare audit packet summary",
  "remind [owner] about evidence", "escalate overdue evidence for [case]",
  "audit engagement summary for [manager]", or "evidence follow-up for [case ID]".
  Do NOT use for creating a new audit case (use audit-request-intake),
  assembling scope and evidence context (use audit-evidence-packet),
  detecting evidence gaps (use audit-gap-detection),
  or routing evidence requests (use audit-request-routing).
---

## Overview

Drafts audience-appropriate communications for audit engagement coordination — audit manager summaries, evidence requests to control owners, overdue evidence follow-ups, status updates to business process owners, escalation notices, and methodology-cited packet summaries. All formal communications are created as Outlook drafts for lead auditor review before sending. Internal audit team coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for lead auditor review and confirmation before any message is sent.

## When to Use

- An audit manager needs an engagement summary with scope, evidence status, and gap overview
- A control owner needs a formal evidence request specifying items, period, format, and deadline
- An overdue evidence request needs a follow-up reminder
- A business process owner needs a status update on the engagement
- An unresponsive control owner needs escalation to their management
- An audit packet summary needs to be prepared for workpaper documentation

## When NOT to Use

- Creating a new audit case — use audit-request-intake
- Assembling scope and evidence context — use audit-evidence-packet
- Detecting evidence gaps — use audit-gap-detection
- Routing evidence requests to owners — use audit-request-routing
- Confirming audit packet disposition — this is always a human decision (IA-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case artifacts and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting audit communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Audit case data** — `SearchM365(sources=["files"], query="audit case tracker")` then `ReadFileContent` for the case row
- **Audit evidence packet** — `SearchM365(sources=["files"], query="audit packet [Case ID]")` then `ReadFileContent`
- **Gap report** — from audit-gap-detection output (missing items, scope gaps, timeline risk)
- **Communication templates** — `SearchM365(sources=["files"], query="audit communication template")` or `SearchM365(sources=["files"], query="evidence request template")` then `ReadFileContent`

Determine which communication type is needed based on the gap report, case status, and user request.

### Step 2: Draft Communication

**Communication type 1: Audit engagement summary to audit manager**
- To: audit manager
- Tone: methodology-precise, structured
- Content: Case ID, engagement type, scope statement, audit period, lead auditor, current evidence collection status (X of Y items received), gap overview (count by severity), recommended testing focus areas, timeline assessment, key risks or blockers
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: Evidence request to control owner**
- To: control owner (resolved by name and email)
- Tone: clear, professional, specific
- Content: Engagement reference (Case ID), specific evidence items needed (item IDs and descriptions), audit period the evidence must cover, acceptable format for each item, submission deadline, where to submit (SharePoint engagement folder link), lead auditor contact for questions
- Channel: Outlook draft (use `CreateDraftMessage`)
- Mark with appropriate confidentiality marking

**Communication type 3: Overdue evidence follow-up**
- To: control owner (with cc to their manager if second follow-up)
- Tone: factual, professional, appropriately urgent
- Content: Case ID, original request reference, specific items still outstanding, original deadline (now passed), revised deadline, impact of continued delay on audit timeline, submission instructions
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 4: Status update to business process owner**
- To: business process owner
- Tone: concise, informational
- Content: Case ID, engagement scope summary, evidence collection progress, any areas where business process owner action is needed, expected timeline for completion
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 5: Escalation notice to audit manager**
- To: audit manager (for unresponsive control owners or critical timeline risk)
- Tone: factual, structured, urgent where appropriate
- Content: Case ID, what is blocked, which control owners are unresponsive, how long evidence has been overdue, prior follow-up actions taken, impact on audit timeline, recommended resolution
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 6: Methodology-cited packet summary**
- To: audit file / workpaper documentation
- Tone: methodology-precise, suitable for workpaper inclusion
- Content: Engagement summary, scope definition with methodology references, evidence collected and outstanding, testing approach citations from the audit program, IIA Standards references, key contacts and control ownership, overall readiness assessment for the workpaper trail
- Channel: Word document (invoke `docx` skill) for workpaper filing

### Step 3: Post Internal Coordination (After Confirmation)

For internal audit team coordination that does not require Outlook formality:
- `PostMessage` — Teams message to audit team members for status coordination, task handoffs, or quick updates
- Only after lead auditor confirms the content

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, audit packet, gap report, communication templates |
| ReadFileContent | Read case artifacts and templates |
| CreateDraftMessage | Outlook drafts for all formal audit communications |
| PostMessage | Teams messages for internal audit team coordination |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create as Outlook draft** — never send without explicit lead auditor confirmation
- **Never include audit conclusions, preliminary findings, or risk ratings** in any communication — present factual status and evidence needs only
- **Never include internal audit team deliberations or testing strategies** in communications to control owners or business process owners — these are confidential to the audit function
- **Match tone to audience** — methodology-precise for audit team and workpaper communications; clear and professional for control owner requests; concise for business process owner updates
- **Include engagement reference number** (Case ID) and evidence request item IDs in every communication for audit trail traceability
- **Mark all internal audit communications** with appropriate confidentiality markings
- **Evidence request drafts must specify exact items needed** — item IDs, descriptions, audit period covered, acceptable format, and submission deadline; vague requests undermine evidence quality
- **Respect data classification boundaries** — no audit observations, preliminary findings, or testing strategies in any outgoing communication; factual status and evidence requests only
- **Log the communication** — record communication type, recipient, Case ID, and timestamp in the case tracker after sending for audit trail
- **Preserve audit independence** — never include language that could be construed as pre-judging control effectiveness or management performance
