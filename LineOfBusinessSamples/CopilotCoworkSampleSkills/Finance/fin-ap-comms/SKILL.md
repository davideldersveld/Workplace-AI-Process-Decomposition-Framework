---
name: fin-ap-comms
description: |
  Drafts outreach emails, approval packets, follow-up requests, and escalation
  notices for AP invoice exception cases.
  Use when user asks to "draft supplier email", "prepare approval packet",
  "write follow-up for missing receipt", "draft AP outreach",
  "invoice exception follow-up", "escalation notice for AP case",
  "draft missing PO request", "write approval summary for [case]",
  "draft vendor clarification for [invoice]", or "resolution confirmation email".
  Do NOT use for creating a new exception case (use fin-invoice-intake),
  assembling matching context (use fin-match-context),
  classifying exception type (use fin-exception-classify),
  assessing risk or control path (use fin-control-path),
  routing to action owner (use fin-exception-routing),
  or updating system status (use fin-status-update).
---

## Overview

Drafts audience-appropriate communications for AP invoice exception resolution — supplier outreach, buyer follow-ups, approval summaries, escalation notices, and resolution confirmations. All communications are created as Outlook drafts or Teams messages for AP analyst review before sending. Draws on the context packet, classification, and control path data to produce accurate, evidence-cited content.

This skill operates in "AI draft plus approve" mode — every draft is presented for AP analyst review and confirmation before any message is sent.

## When to Use

- A supplier needs to be contacted about a discrepancy or missing information
- A buyer or cost center owner needs a follow-up about a missing receipt or PO
- An approval summary needs to be prepared for a cost center owner or controller
- An escalation notice needs to be sent to the AP manager or controller
- A resolution confirmation needs to be communicated

## When NOT to Use

- Creating a new exception case — use fin-invoice-intake
- Assembling matching context — use fin-match-context
- Classifying exception type — use fin-exception-classify
- Assessing risk or determining the control path — use fin-control-path
- Routing to the action owner — use fin-exception-routing
- Updating case status in the tracker — use fin-status-update

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Exception case data** — from the tracker: `SearchM365(sources=["files"], query="AP exception tracker")` then `ReadFileContent`
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Case ID]")` then `ReadFileContent`
- **Classification and control path** — from tracker fields or prior skill outputs
- **Communication templates** — `SearchM365(sources=["files"], query="AP communication template")` or `SearchM365(sources=["files"], query="AP email template")` then `ReadFileContent`
- **Tone guides** — `SearchM365(sources=["files"], query="supplier communication guidelines")` if available

Determine which communication type is needed based on the exception type, control path, and user request.

### Step 2: Draft Communication

**Communication type 1: Missing receipt request to buyer**
- To: procurement buyer on the PO
- Tone: operational, collegial
- Content: Exception Case ID, Invoice ID, Vendor Name, PO reference, what is missing (goods receipt), requested action (confirm receipt or provide timeline), deadline based on SLA
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: Amount discrepancy clarification to supplier**
- To: vendor contact (from email correspondence or vendor master)
- Tone: professional, formal
- Content: Invoice number, PO reference, specific discrepancy (expected amount vs. invoiced amount), request for clarification or corrected invoice, reference to contract terms if applicable
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 3: Missing PO follow-up to procurement**
- To: procurement team or buyer
- Tone: operational
- Content: Invoice ID, Vendor Name, amount, request to confirm PO number or authorize non-PO processing, policy reference for non-PO invoices
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 4: Approval summary for cost center owner or AP manager**
- To: approver identified by fin-control-path or fin-exception-routing
- Tone: concise, evidence-cited
- Content: Exception Case ID, Invoice ID, Vendor Name, Amount, exception type, risk flags, supporting evidence summary, specific approval action requested, link to full context packet
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 5: Escalation notice to controller**
- To: controller or AP operations manager
- Tone: formal, factual
- Content: Exception Case ID, Invoice ID, Amount, why escalation is needed (materiality threshold, risk flags, segregation conflict, unresolved after standard routing), prior actions taken, requested decision
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 6: Resolution confirmation**
- To: relevant stakeholders (requestor, approver, buyer as appropriate)
- Tone: brief, confirmatory
- Content: Exception Case ID, Invoice ID, Vendor Name, resolution outcome, any follow-up actions, case closure date
- Channel: Outlook draft (use `CreateDraftMessage`) and Teams update (use `PostMessage` for internal notification)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, context packet, communication templates, tone guides |
| ReadFileContent | Read case data, context packet, templates |
| CreateDraftMessage | Outlook drafts for all external and formal communications |
| PostMessage | Teams messages for internal coordination updates |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create as Outlook draft** — never send without explicit AP analyst confirmation
- **Match tone to audience** — professional and formal for external supplier communication; operational for internal requests; concise and evidence-cited for approval summaries
- **Include Exception Case ID, Invoice ID, and Vendor ID** in every communication for audit traceability
- **Cite specific discrepancy details** — amounts, dates, PO references, receipt status — with evidence from the context packet
- **Never include bank account details, payment terms, or sensitive vendor financial data** in Teams messages — use Outlook for sensitive content
- **Never fabricate invoice data, amounts, or vendor details** — use only what is in the context packet and tracker
- **Respect the 2-business-day SLA** — note urgency in communications when the deadline is approaching
- **Log the communication** — record communication type, recipient, timestamp, and case reference in the tracker after sending
