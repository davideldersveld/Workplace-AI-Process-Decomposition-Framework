---
name: cs-response-drafter
description: |
  Drafts customer-facing responses, internal handoff summaries, and escalation packets
  for service cases using knowledge articles and approved response templates.
  Use when user asks to "draft response for case [ID]", "write customer reply",
  "prepare handoff summary", "escalation summary for [case]",
  "first response for [customer]", "draft acknowledgment for [case]",
  "write follow-up for case [ID]", or "internal handoff for [case]".
  Do NOT use for assembling context (use cs-context-packet),
  classifying the issue (use cs-issue-classifier),
  assessing severity (use cs-severity-assessment),
  routing the case (use cs-case-routing),
  or creating a new case (use cs-case-intake).
---

## Overview

Drafts audience-appropriate service communications — customer acknowledgments, first responses with resolution guidance, internal handoff summaries for receiving agents, escalation packets for managers, and follow-up requests for missing information. All customer-facing communications are created as Outlook drafts for agent review before sending.

This skill operates in "AI draft plus approve" mode — every communication is created as a draft or presented for review. Nothing is sent to customers without explicit user confirmation.

## When to Use

- A case has been routed and the assigned agent needs a first response drafted
- A case is being handed off to another agent and needs an internal summary
- A case is being escalated and needs an escalation packet
- A customer needs a follow-up request for missing information
- A case acknowledgment needs to be sent

## When NOT to Use

- Assembling customer and account context — use cs-context-packet
- Classifying the issue type — use cs-issue-classifier
- Assessing severity and SLA path — use cs-severity-assessment
- Routing the case — use cs-case-routing
- Creating a new case — use cs-case-intake
- Making the final triage disposition — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and identify communication type", activeForm="Reading case materials")
TaskCreate(subject="Draft communication using appropriate template", activeForm="Drafting communication")
```

### Step 1: Read Case Materials

Locate and read the case context:

- **Case data** — from the case tracker: `SearchM365(sources=["files"], query="service case tracker")`
- **Context packet** — from cs-context-packet output: `SearchM365(sources=["files"], query="context packet [Case ID]")`
- **Classification and severity** — from the tracker or Adaptive Card outputs
- **Knowledge articles** — `SearchM365(sources=["files"], query="[issue category] knowledge article")` or `SearchM365(sources=["files"], query="[issue type] resolution")`
- **Response templates** — `SearchM365(sources=["files"], query="response template [communication type]")`

Read each document using `ReadFileContent`.

### Step 2: Identify Communication Type and Draft

Determine the communication type from the user's request:

| Communication Type | Audience | Key Elements |
|-------------------|----------|-------------|
| **Acknowledgment** | Customer | Case reference, confirmation of receipt, expected response timeline, next steps |
| **First response** | Customer | Issue understanding, resolution guidance from knowledge articles, next steps, case reference |
| **Follow-up request** | Customer | Specific missing information needed, how to provide it, case reference |
| **Internal handoff** | Receiving agent | Case summary, classification, severity, customer context, prior actions, what needs to happen next |
| **Escalation packet** | Escalation manager | Full case history, classification, severity rationale, customer impact, prior resolution attempts, recommended action |
| **Resolution confirmation** | Customer | What was done, confirmation that issue is resolved, how to reopen if needed, case reference |

### Customer-Facing Drafts

For all customer-facing communications, create an Outlook draft using `CreateDraftMessage`:

**Acknowledgment:**
- Subject: "Re: [Original Subject] — Case [Case ID]"
- Body: Thank the customer, confirm the issue is understood (restate briefly), provide expected response timeline based on SLA, include case reference number

**First Response:**
- Subject: "Re: [Original Subject] — Case [Case ID]"
- Body: Acknowledge the issue, provide resolution steps from knowledge articles (cite which article), explain next steps, include case reference
- Tone: empathetic, clear, and action-oriented

**Follow-up Request:**
- Subject: "Re: [Original Subject] — Additional Information Needed — Case [Case ID]"
- Body: Explain what specific information is needed and why, how to provide it, timeline, case reference

**Resolution Confirmation:**
- Subject: "Re: [Original Subject] — Resolved — Case [Case ID]"
- Body: Summarize what was done, confirm resolution, provide instructions to reopen if the issue recurs, case reference

### Internal Communications

**Internal Handoff** (via `PostMessage` to receiving agent after user confirmation):
- Case ID, customer name, classification, severity, SLA deadline
- Summary of issue and customer context
- Prior actions taken and what needs to happen next
- Link to case folder in SharePoint

**Escalation Packet** (via Word document, invoke `docx` skill):
- Full case history and timeline
- Classification and severity with rationale
- Customer profile and entitlement
- Prior resolution attempts
- Customer impact assessment
- Recommended action and urgency

After drafting, tell the user: "I've created a draft [type] for [recipient]. Review and send when ready."

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, context packet, knowledge articles, response templates |
| ReadFileContent | Read case materials, knowledge articles, templates |
| CreateDraftMessage | Create Outlook draft replies to customer (never auto-send) |
| PostMessage | Teams handoff messages to agents (after confirmation) |

## Guardrails

- **Always create customer-facing communications as Outlook draft** — never send without explicit user confirmation
- **Never include internal details in customer-facing drafts** — no severity levels, agent names, routing information, internal case notes, or system references
- **Never make commitments** — no credits, refunds, timeline promises, or guarantees in drafted responses; these require supervisor approval
- **Match tone to audience** — empathetic and clear for customers, operational and precise for internal handoffs, comprehensive and evidence-based for escalation packets
- **Include case reference** in every communication — Case ID must appear in subject and body
- **Cite knowledge articles** used in response drafting — include article title or ID
- **Never fabricate resolution steps** — only include guidance from knowledge articles or confirmed procedures
- **Preserve customer dignity** — never use dismissive language, blame the customer, or minimize the issue regardless of severity
