---
name: itsm-comms-drafter
description: |
  Drafts user-facing updates, resolver group handoff summaries, escalation
  notices, and major incident bridge summaries for incidents.
  Use when user asks to "draft user update for [incident]",
  "write handoff summary", "prepare incident update email",
  "notify the user about their ticket",
  "incident status update for [ID]", "major incident summary",
  "draft escalation notice for [incident]",
  or "resolver group briefing for [ticket]".
  Do NOT use for creating a new incident (use itsm-incident-intake),
  enriching incident context (use itsm-context-packet),
  classifying incident type (use itsm-classification-assist),
  assessing severity (use itsm-severity-recommend),
  or routing to resolver groups (use itsm-assignment-router).
---

## Overview

Drafts audience-appropriate communications for incident management — user acknowledgments, status updates, resolver group handoff summaries, escalation notices for incident managers, and major incident bridge summaries. All user-facing communications are created as Outlook drafts for analyst review. Internal coordination uses Teams messages after confirmation.

This skill operates in "AI draft plus approve" mode — every draft is presented for analyst review and confirmation before any message is sent or document is finalized.

## When to Use

- A user needs acknowledgment that their incident has been received and assigned
- A user needs a status update on their open incident
- A resolver group needs a structured handoff summary with full incident context
- An incident manager needs an escalation notice for a P1 or P2 incident
- A major incident requires bridge call details and executive summary
- A follow-up request needs to be sent to the user for additional information

## When NOT to Use

- Creating a new incident record — use itsm-incident-intake
- Assembling user, service, or asset context — use itsm-context-packet
- Classifying the incident type or affected service — use itsm-classification-assist
- Assessing severity or escalation path — use itsm-severity-recommend
- Routing to a resolver group — use itsm-assignment-router
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident data and determine communication type", activeForm="Reading incident context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Incident Context

Locate and read required inputs:

- **Incident tracker** — `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent` — incident data, classification, priority, assignment
- **Context packet** — `SearchM365(sources=["files"], query="context packet [Incident ID]")` then `ReadFileContent` — user, service, and asset context
- **Communication templates** — `SearchM365(sources=["files"], query="incident communication template")` then `ReadFileContent`
- **Tone guide** — `SearchM365(sources=["files"], query="incident communication tone guide")` then `ReadFileContent` — approved language and tone standards

Determine which communication type is needed based on the incident status, audience, and user request.

### Step 2: Draft Communication

**Communication type 1: User acknowledgment**
- Audience: end user who reported the incident
- Tone: empathetic, clear, reassuring
- Content: confirmation that the incident has been received and logged, Incident Reference number, assigned team (no internal queue names — use friendly team descriptions), what happens next (investigation timeline), what the user may need to provide, how to check status or contact support
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: User status update**
- Audience: end user
- Tone: clear, helpful, progress-focused
- Content: Incident Reference number, current investigation status in plain language, what has been done so far, expected next steps and timeline, whether user action is needed, contact information for questions
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 3: Resolver group handoff summary**
- Audience: assigned resolver group or individual resolver
- Tone: operational, structured, factual
- Content: Incident ID, category and subcategory, priority and SLA targets, symptom summary, affected service and configuration items, affected user profile, recent changes to the service (flagged if within 48 hours), related open incidents, prior incident history, knowledge article references, recommended investigation starting points, open questions
- Channel: Word document (invoke `docx` skill) for formal handoff; Teams direct message (use `PostMessage`) for quick coordination

**Communication type 4: Escalation notice for incident manager**
- Audience: incident manager or service operations manager
- Tone: concise, structured, action-oriented
- Content: Incident ID, category, priority with impact and urgency rationale, affected service and user population, SLA status (time elapsed and remaining), assigned resolver group and current investigation status, escalation trigger (why this is being escalated), recommended actions (bridge call, additional resources, service owner engagement)
- Channel: Teams direct message (use `PostMessage`) for urgent escalation; Outlook draft (use `CreateDraftMessage`) for formal escalation

**Communication type 5: Major incident bridge summary (P1/P2 only)**
- Audience: major incident team, service owners, executive stakeholders
- Tone: professional, high-level, impact-focused
- Content: Incident ID, affected service in business terms, business impact summary (user population, revenue impact, regulatory risk), current status and response posture, bridge call details (time, link), assigned incident commander, expected next update time
- Channel: Outlook draft (use `CreateDraftMessage`) for executive distribution; Teams direct message for incident team coordination
- Sensitivity: no internal hostnames, IP addresses, or infrastructure details in executive-facing summaries

**Communication type 6: Follow-up information request**
- Audience: end user
- Tone: helpful, specific, action-oriented
- Content: Incident Reference number, what additional information is needed (screenshots, error messages, steps to reproduce, time of occurrence), how to provide the information, deadline or urgency, contact information
- Channel: Outlook draft (use `CreateDraftMessage`)

### Step 3: Apply Audience-Specific Language Rules

| Audience | Internal System Names | Infrastructure Details | Queue Names | PII | Technical Jargon |
|----------|----------------------|----------------------|-------------|-----|------------------|
| End user | Not allowed | Not allowed | Not allowed (use friendly descriptions) | Own details only | Avoid — use plain language |
| Resolver group | Allowed | Allowed | Allowed | Allowed (scoped) | Appropriate |
| Incident manager | Allowed | Allowed | Allowed | Allowed (scoped) | Appropriate |
| Executive / major incident | Not allowed | Not allowed | Not allowed | Minimal (aggregate impact) | Avoid — use business language |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find incident tracker, context packet, communication templates, tone guide |
| ReadFileContent | Read incident data, context packet, templates, tone guide |
| CreateDraftMessage | Outlook drafts for user-facing updates, escalation notices, major incident summaries |
| PostMessage | Teams direct messages to resolver groups and incident managers |

## Guardrails

- **Always create user-facing communications as Outlook drafts** — never send without explicit analyst confirmation
- **Match tone and detail level to audience** — empathetic and clear for end users; operational and structured for resolver groups; high-level and impact-focused for executives
- **Include Incident ID or Reference number, current status, and next expected action** in every communication for traceability
- **Never include internal system names, queue names, or technical identifiers** in user-facing updates — use friendly team descriptions and plain language
- **Redact IP addresses, hostnames, and infrastructure details** from user-facing and executive-facing messages
- **For major incident bridge summaries**, never share security classification, internal architecture details, or investigation hypotheses — impact and status only
- **All drafts must include the label** "DRAFT — analyst review required before sending"
- **Never include resolution commitments or timelines** that have not been confirmed by the resolver group — use language like "our team is actively investigating" rather than "this will be resolved by [time]"
- **Never disclose other users' incident details** in a user communication — each user sees only their own incident status
- **Respect the audience language matrix** — every communication must be checked against the audience's authorized detail level before delivery
- **Log the communication** — record communication type, recipient, Incident ID, and timestamp in the incident tracker after sending for audit trail
- **Never recommend workarounds or remediation steps** unless they come from a confirmed knowledge article — unverified workarounds can cause additional damage
