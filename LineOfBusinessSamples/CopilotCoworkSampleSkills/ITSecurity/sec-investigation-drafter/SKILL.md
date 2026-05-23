---
name: sec-investigation-drafter
description: |
  Drafts investigation summaries, analyst handoff briefs, evidence preservation
  requests, escalation briefs, and executive notifications for security cases.
  Use when user asks to "draft investigation summary",
  "prepare case handoff", "write evidence request for [case ID]",
  "summarize this security case", "create analyst briefing for [alert]",
  "draft escalation brief", "executive notification for [incident]",
  or "evidence preservation request for [case]".
  Do NOT use for creating a new case (use sec-alert-intake),
  enriching alert context (use sec-enrichment-packet),
  classifying alert type or risk (use sec-risk-classifier),
  assessing severity (use sec-severity-recommend),
  or routing to analysts (use sec-analyst-router).
---

## Overview

Drafts audience-appropriate investigation communications for security cases — analyst handoff summaries, evidence preservation requests, SOC lead escalation briefs, executive notifications, and follow-up requests for additional telemetry. Matches detail level to audience authorization and enforces strict controls on IOC exposure, investigation hypotheses, and data sensitivity.

This skill operates in "AI draft plus approve" mode — every draft is presented for analyst review and confirmation before any message is sent or document is finalized.

## When to Use

- An assigned analyst needs a structured handoff summary with full case context
- Evidence needs to be preserved and a formal request must be sent to IT, identity, or data custodian teams
- The SOC lead needs an escalation brief for a Critical or High severity case
- Executive leadership needs a notification draft for a confirmed or potential security incident
- Additional telemetry or log access needs to be requested from platform teams

## When NOT to Use

- Creating a new alert case — use sec-alert-intake
- Assembling entity, asset, or threat context — use sec-enrichment-packet
- Classifying the alert type or likely risk — use sec-risk-classifier
- Assessing severity or investigation path — use sec-severity-recommend
- Routing to an analyst queue or SOC team — use sec-analyst-router
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Alert tracker** — `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent` — case data, classification, severity, assignment
- **Enrichment packet** — `SearchM365(sources=["files"], query="enrichment packet [Case ID]")` then `ReadFileContent` — entity, asset, and threat context
- **Communication templates** — `SearchM365(sources=["files"], query="security investigation template")` or `SearchM365(sources=["files"], query="evidence request template")` then `ReadFileContent`
- **Prior investigation summaries** — `SearchM365(sources=["files"], query="investigation summary [similar case]")` then `ReadFileContent` — exemplars for consistent formatting

Determine which communication type is needed based on the case status, severity, audience, and user request.

### Step 2: Draft Communication

**Communication type 1: Analyst handoff summary**
- Audience: investigating analyst (full technical access)
- Tone: operational, structured, factual
- Content: Case ID, alert source, detection rule, affected entity, classification and confidence, severity and rationale, enrichment context summary, related alerts and prior history, applicable playbook and recommended investigation steps, evidence inventory, open questions, SLA status and deadlines
- Channel: Word document (invoke `docx` skill) stored in SharePoint case folder; Teams direct message to the assigned analyst with case summary and link to the full document
- Sensitivity: Confidential — Security Operations

**Communication type 2: Evidence preservation request**
- Audience: IT operations, identity team, or data custodian (limited security context)
- Tone: clear, formal, specific
- Content: Case ID (no classification or severity details), what evidence to preserve (logs, access records, email archives, endpoint snapshots), preservation period, legal hold advisory if applicable, point of contact for questions, deadline for preservation confirmation
- Channel: Outlook draft (use `CreateDraftMessage`) — never auto-send
- Sensitivity: do not include alert classification, severity, investigation hypotheses, or IOCs in evidence requests to non-SOC teams

**Communication type 3: Escalation brief for SOC lead**
- Audience: SOC lead or incident response team lead (full security context)
- Tone: concise, structured, action-oriented
- Content: Case ID, classification, severity, blast radius, urgency assessment, current investigation status, escalation rationale, recommended actions (containment options as information, incident declaration consideration), timeline of events, related cases if any
- Channel: Word document for formal escalation; Teams direct message for urgent cases
- Sensitivity: Confidential — Security Operations

**Communication type 4: Executive notification**
- Audience: CISO, executive leadership (no technical details)
- Tone: professional, high-level, impact-focused
- Content: incident summary in business terms, business impact assessment, affected services or data types (no specific hostnames or account names), current status and response posture, expected timeline for resolution, notification obligations if applicable
- Channel: Outlook draft (use `CreateDraftMessage`) — requires analyst and SOC lead review before sending
- Sensitivity: no IOCs, no hostnames, no account names, no investigation hypotheses, no technical indicators; PII redacted unless analyst explicitly confirms inclusion

**Communication type 5: Follow-up request for additional telemetry**
- Audience: platform team, network operations, or security engineering (technical, limited investigation context)
- Tone: specific, actionable, time-bounded
- Content: Case ID, what additional data is needed (specific log types, time windows, systems), why it is needed (without disclosing investigation hypotheses), deadline for delivery, point of contact
- Channel: Outlook draft (use `CreateDraftMessage`) or Teams direct message depending on urgency

### Step 3: Apply Audience-Specific Sensitivity Controls

| Audience | IOCs | Account Names | Hostnames | Classification | Investigation Details |
|----------|------|---------------|-----------|----------------|----------------------|
| Investigating analyst | Allowed (in secure document) | Allowed | Allowed | Allowed | Allowed |
| SOC lead / IR team | Allowed | Allowed | Allowed | Allowed | Allowed |
| IT operations / data custodian | Not allowed | Not allowed | Allowed (for evidence scope) | Not allowed | Not allowed |
| Executive leadership | Not allowed | Not allowed | Not allowed | Summary only | Not allowed |
| Platform / network teams | Not allowed | Not allowed | Allowed (for log scope) | Not allowed | Not allowed |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find alert tracker, enrichment packet, communication templates, prior investigation summaries |
| ReadFileContent | Read case data, enrichment packet, templates, prior summaries |
| CreateDraftMessage | Outlook drafts for evidence requests, executive notifications, formal communications |
| PostMessage | Teams direct messages to analysts and SOC leads for urgent coordination |

## Guardrails

- **Always create external-facing communications as Outlook drafts** — never send without explicit analyst confirmation
- **Match detail level to audience** — full technical detail for investigating analysts; operational summary for SOC leads; executive summary with no IOCs, hostnames, account names, or investigation hypotheses for leadership
- **Never include raw IOCs, IP addresses, hashes, domain names, or affected account credentials** in communications to audiences outside the SOC
- **Never include investigation hypotheses or attribution speculation** in any written artifact — only state confirmed findings and open questions
- **Redact all PII, employee names, and account identifiers** from executive-facing summaries unless the analyst explicitly confirms inclusion
- **For cases involving potential data breach**, include a legal hold advisory in the investigation summary and evidence preservation requests
- **Include Case ID and reference links** in every communication for traceability
- **All drafts must include the label** "DRAFT — analyst review required before sending"
- **Respect the audience sensitivity matrix** — every communication must be checked against the audience's authorized detail level before delivery
- **Never disclose insider threat investigation details** to non-designated teams — insider threat cases have restricted visibility
- **Never recommend or authorize containment actions** in any communication — containment is always human-authorized through the SIEM/SOAR platform
- **Store investigation summary documents in SharePoint** with sensitivity labels and version history — investigation artifacts are auditable records
- **Never post investigation details to general Teams channels** — use direct messages to authorized analysts only; channel posts use Case ID references only
