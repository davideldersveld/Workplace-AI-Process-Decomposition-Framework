---
name: compliance-case-comms
description: |
  Drafts compliance case summaries, evidence follow-up requests, escalation notices,
  and status updates for different audiences.
  Use when user asks to "draft compliance case summary", "prepare case update for",
  "send evidence request for [case ID]", "draft follow-up to [control owner]",
  "compliance case status update", "write case summary for reviewers",
  "request evidence from [name]", "draft escalation notice",
  or "update reporter on case status".
  Do NOT use for assembling context (use compliance-context-packet),
  detecting risk indicators (use compliance-risk-detection),
  routing for review (use compliance-review-routing),
  or creating a new case (use compliance-case-intake).
---

## Overview

Drafts audience-appropriate compliance communications — case summaries for reviewers, evidence follow-up requests for control owners, escalation notices for leadership, status updates for reporters, and internal audit referrals. All communications are created as Outlook drafts for user review before sending.

This skill operates in "AI draft plus approve" mode — every communication is created as a draft. Nothing is sent without explicit user confirmation.

## When to Use

- A compliance case needs a summary drafted for reviewers
- Evidence is missing and a follow-up request needs to be sent to a control owner
- A case needs to be escalated to compliance leadership or legal
- A reporter (non-anonymous) needs a status update
- An internal audit referral needs to be drafted

## When NOT to Use

- Assembling policy context — use compliance-context-packet
- Detecting risk indicators — use compliance-risk-detection
- Routing for review — use compliance-review-routing
- Creating a new case — use compliance-case-intake
- Making the final triage disposition — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case materials and identify audience", activeForm="Reading case materials")
TaskCreate(subject="Draft communication using appropriate template", activeForm="Drafting communication")
```

### Step 1: Read Case Materials

Locate and read the case context:

- **Case tracker** — `SearchM365(sources=["files"], query="compliance case tracker")` then `ReadFileContent`
- **Context packet** — `SearchM365(sources=["files"], query="context packet [case ID]")` then `ReadFileContent`
- **Risk indicator report** — from compliance-risk-detection output or uploaded file
- **Communication templates** — `SearchM365(sources=["files"], query="compliance communication template")`

### Step 2: Identify Audience and Select Template

Determine the communication type from the user's request:

| Communication Type | Audience | Template |
|-------------------|----------|----------|
| **Case summary** | Compliance analyst / reviewer | Risk indicators, evidence status, recommended review areas, policy references |
| **Evidence request** | Control owner / business process owner | Specific missing items, submission deadline, case folder link |
| **Escalation notice** | Compliance manager / legal reviewer | Severity rationale, regulatory flags, required action, SLA deadline |
| **Status update to reporter** | Reporter (when not anonymous) | Factual status only — case received, under review, additional information needed |
| **Internal audit referral** | Internal audit liaison | Control failure indicators, evidence summary, recommended audit scope |
| **Regulatory reporting flag** | Compliance leadership | Regulatory indicator details, applicable reporting requirements, recommended timeline |

### Step 3: Draft the Communication

For each communication, create an Outlook draft using `CreateDraftMessage`:

**Case Summary (to compliance reviewer):**
- Subject: "Compliance Case Review — [Case ID]: [Case Type]"
- Body: Case summary, applicable policies (with section references), risk indicators, evidence status, recommended review focus areas, SLA deadline

**Evidence Request (to control owner):**
- Subject: "Evidence Request — Compliance Case [Case ID]"
- Body: Required evidence items (from the checklist), submission deadline, case folder link, contact for questions

**Escalation Notice (to compliance manager / legal):**
- Subject: "Escalation — Compliance Case [Case ID]: [Severity]"
- Body: Severity classification with rationale, regulatory flags (if any), required action, SLA deadline, case folder link

**Status Update (to non-anonymous reporter):**
- Subject: "Status Update — Your Compliance Report [Case ID]"
- Body: Factual status only (received, under review, additional information requested). No conclusions, no severity, no risk indicators.

**Internal Audit Referral:**
- Subject: "Internal Audit Referral — Compliance Case [Case ID]"
- Body: Control failure indicators, evidence summary, affected control area, recommended audit scope

**Regulatory Reporting Flag:**
- Subject: "URGENT: Regulatory Reporting Assessment Required — [Case ID]"
- Body: Regulatory indicator details, applicable reporting requirements, recommended assessment timeline

After drafting, present the draft to the user: "I've created a draft [type] for [recipient]. Review and send when ready."

For Teams coordination messages (non-sensitive status updates to the compliance ops channel), use `PostMessage` after user confirmation.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, context packet, communication templates |
| ReadFileContent | Read case materials for summary content |
| SearchPeople / GetUserDetails | Resolve recipient identities |
| CreateDraftMessage | Create all Outlook drafts (never auto-send) |
| PostMessage | Send Teams coordination messages (after confirmation) |

## Guardrails

- **Always create as Outlook draft** — never send any compliance communication without explicit user confirmation
- **Never include conclusions or fault assessments** — all communications must present factual status only; risk judgments are the reviewer's responsibility
- **Never reveal anonymous reporter identity** — if the case was reported anonymously, status updates to the reporter are not possible; inform the user
- **Never disclose case details to unauthorized individuals** — verify recipients are on the approved reviewer list for this case
- **Include case reference in every communication** — every email must include the Case ID
- **Apply confidentiality markings** — all internal compliance communications should note "Confidential — Compliance Review Material"
- **Match tone to audience** — precise and evidence-focused for compliance reviewers, clear and action-oriented for control owners, factual and neutral for reporters
- **Regulatory urgency** — regulatory reporting flag communications must be marked URGENT and drafted immediately when regulatory indicators are detected
