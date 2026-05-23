---
name: ins-claim-comms
description: |
  Drafts claimant communications, broker updates, adjuster summaries, evidence
  deficiency notices, and escalation memos for insurance claims.
  Use when user asks to "draft claimant letter for [claim ID]",
  "send deficiency notice", "prepare adjuster summary",
  "broker update for this claim", "claim follow-up communication",
  "draft acknowledgment for [claimant]", "missing evidence request for [claim]",
  "SIU referral memo", or "reservation of rights draft for [claim ID]".
  Do NOT use for creating a new claim case (use ins-fnol-intake),
  assembling policy and claimant context (use ins-claim-context),
  classifying claim type or severity (use ins-claim-classifier),
  assessing coverage path or evidence gaps (use ins-coverage-gap-detection),
  or routing to adjusters or SIU (use ins-claim-routing).
---

## Overview

Drafts audience-appropriate communications for insurance claims coordination — claimant acknowledgment letters, evidence deficiency notices, broker and agent status updates, adjuster assignment summaries, SIU referral memos, and reservation of rights letter drafts. All claimant-facing communications use approved templates and are created as Outlook drafts for adjuster review. Internal coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for adjuster or claims supervisor review and confirmation before any message is sent.

## When to Use

- A claimant needs acknowledgment that their loss report has been received
- Missing evidence needs to be requested from the claimant
- A broker or agent needs a status update on a claim
- An adjuster needs an assignment summary with full case context
- An SIU referral memo needs to be prepared (internal only)
- A reservation of rights letter draft needs to be created (requires legal review)

## When NOT to Use

- Creating a new claim case — use ins-fnol-intake
- Assembling policy and claimant context — use ins-claim-context
- Classifying the claim type or severity — use ins-claim-classifier
- Assessing initial coverage path or missing evidence — use ins-coverage-gap-detection
- Routing to an adjuster, catastrophe desk, or SIU — use ins-claim-routing
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read claim data and determine communication type", activeForm="Reading claim context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Claim case data** — `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent`
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Claim ID]")` then `ReadFileContent`
- **Classification and evidence assessment** — from tracker fields or prior skill outputs
- **Communication templates** — `SearchM365(sources=["files"], query="claims communication template")` or `SearchM365(sources=["files"], query="claimant letter template")` then `ReadFileContent`
- **Jurisdiction-specific language** — `SearchM365(sources=["files"], query="[jurisdiction] claims language requirements")` then `ReadFileContent`

Determine which communication type is needed based on the case status, evidence gaps, and user request.

### Step 2: Draft Communication

**Communication type 1: Claimant acknowledgment letter**
- To: claimant (email on file)
- Tone: professional, empathetic, procedural
- Content: confirmation that the loss report has been received, Claim Reference number, assigned adjuster contact (if available), what happens next (investigation timeline), what the claimant may need to provide, contact information for questions
- Template: use approved claimant acknowledgment template
- Jurisdiction: include state-mandated acknowledgment language if applicable
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: Evidence deficiency notice**
- To: claimant
- Tone: clear, helpful, specific
- Content: Claim Reference number, specific documents still needed (proof of loss, photos, police report, repair estimate, medical records), how to submit, deadline for submission, what happens if not received, contact for questions
- Template: use approved deficiency notice template
- Jurisdiction: include state-mandated proof of loss timeline language if applicable
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 3: Broker or agent status update**
- To: broker or agent of record
- Tone: professional, concise, informative
- Content: Claim Reference number, current claim status, assigned adjuster, evidence status, expected next steps, timeline for adjuster review
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 4: Adjuster assignment summary**
- To: assigned adjuster
- Tone: operational, structured, factual
- Content: Claim ID, claim type and severity, classification confidence, evidence completeness status, key facts from context packet, coverage path considerations (framed as considerations, not determinations), fraud indicator flags if present, open questions, recommended next steps, SLA status, regulatory deadline status
- Channel: Outlook draft (use `CreateDraftMessage`) for formal handoff; Teams message (use `PostMessage`) for quick coordination

**Communication type 5: SIU referral memo**
- To: SIU team (internal only — never claimant-facing)
- Tone: factual, structured, objective
- Content: Claim ID, claimant name, policy number, loss type, specific SIU trigger indicators that were flagged, prior claim history relevant to the referral, evidence inventory, recommended investigation focus areas
- Channel: Outlook draft (use `CreateDraftMessage`) — restricted distribution

**Communication type 6: Reservation of rights letter draft**
- To: claimant (requires legal review before sending)
- Tone: formal, neutral, legally precise
- Content: Claim Reference number, acknowledgment of claim receipt, identification of policy provisions under review, statement that investigation is ongoing, reservation of all rights under the policy, contact information
- Template: use approved reservation of rights template
- Channel: Outlook draft (use `CreateDraftMessage`) — flagged as requiring legal review

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find claims tracker, context packet, communication templates, regulatory language guides |
| ReadFileContent | Read case data, context packet, templates, jurisdiction-specific language |
| CreateDraftMessage | Outlook drafts for all claimant-facing and formal communications |
| PostMessage | Teams messages for internal adjuster coordination |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create claimant-facing communications as Outlook draft** — never send without explicit adjuster or supervisor confirmation
- **Never include coverage opinions, liability assessments, or reserve amounts** in any communication — all language must be neutral and procedural
- **Never use commitment language** ("your claim is covered", "we will pay", "you are entitled to") — all claimant communications must avoid implying a coverage determination
- **All drafts must include the label** "DRAFT — requires adjuster/supervisor review before sending"
- **Claimant-facing communications must comply** with jurisdiction-specific unfair claims practices act requirements — read state-specific language from SharePoint at runtime
- **Include the Claim Reference number** in every communication for audit traceability
- **Reservation of rights letters must be flagged** as requiring legal review before sending — these carry significant legal implications
- **SIU-related content must never appear** in claimant-facing or broker-facing communications — SIU referral memos are internal only and restricted to authorized SIU personnel
- **Mask claimant SSN, financial account numbers, and medical details** in all communication outputs unless explicitly required by the communication type
- **Match tone to audience** — empathetic and procedural for claimants; professional and concise for brokers; operational and structured for adjusters; factual and objective for SIU
- **Log the communication** — record communication type, recipient, Claim ID, and timestamp in the claims tracker after sending for audit trail
- **Never make employment or performance assessments** in adjuster summary communications
- **Respect litigation hold flags** — if a claim has an active litigation hold, flag all communications as subject to legal hold procedures
