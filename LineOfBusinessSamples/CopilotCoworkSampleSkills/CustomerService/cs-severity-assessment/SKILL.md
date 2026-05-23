---
name: cs-severity-assessment
description: |
  Assesses case severity, determines the SLA path, identifies escalation conditions,
  and recommends the response priority based on the severity rubric and entitlement data.
  Use when user asks to "assess severity for case [ID]", "what priority is this",
  "check SLA path", "is this an escalation",
  "priority assessment for [case]", "severity check for [customer]",
  "should this be escalated", or "SLA status for case [ID]".
  Do NOT use for assembling context (use cs-context-packet),
  classifying the issue (use cs-issue-classifier),
  routing the case (use cs-case-routing),
  drafting a response (use cs-response-drafter),
  or creating a new case (use cs-case-intake).
---

## Overview

Reads the SLA policy, severity rubric, and escalation criteria from SharePoint, cross-references against case data, classification, customer entitlement tier, and active incidents, and recommends a severity level with SLA path and escalation conditions. Presents the assessment for agent review before any priority is applied.

This skill operates in "AI draft plus approve" mode — the severity recommendation is presented for user review and confirmation before the case tracker is updated.

## When to Use

- A case has been classified and needs a severity and priority assessment
- The user wants to check whether a case meets escalation criteria
- SLA path needs to be determined based on customer entitlement and issue severity

## When NOT to Use

- Assembling customer and account context — use cs-context-packet
- Classifying the issue type — use cs-issue-classifier
- Routing the case to an agent or queue — use cs-case-routing
- Drafting a customer response — use cs-response-drafter
- Creating a new case — use cs-case-intake

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data, SLA policy, and severity rubric", activeForm="Reading severity inputs")
TaskCreate(subject="Assess severity and determine SLA path", activeForm="Assessing severity")
TaskCreate(subject="Present severity recommendation", activeForm="Preparing recommendation")
```

### Step 1: Read Severity Inputs

Locate and read required inputs:

- **Case data** — from the case tracker: `SearchM365(sources=["files"], query="service case tracker")`
- **Classification output** — issue category, intent, and confidence from cs-issue-classifier
- **Context packet** — customer tier, prior cases, entitlement: `SearchM365(sources=["files"], query="context packet [Case ID]")`
- **SLA policy** — `SearchM365(sources=["files"], query="SLA policy")` or `SearchM365(sources=["files"], query="service level agreement")`
- **Severity rubric** — `SearchM365(sources=["files"], query="severity rubric")` or `SearchM365(sources=["files"], query="escalation criteria")`
- **Active incidents** — `SearchM365(sources=["files"], query="active incident")` or `SearchM365(sources=["files"], query="known outage")`
- **Customer entitlement** — from context packet or `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])`

Read each document using `ReadFileContent`.

### Step 2: Assess Severity and SLA Path

Apply the severity rubric to classify the case:

| Factor | How It Affects Severity |
|--------|------------------------|
| Issue category | Some categories have inherent severity floors (e.g., data loss = minimum High) |
| Customer intent | Escalation intent elevates severity; inquiry intent lowers it |
| Customer tier | Premium/enterprise tiers may have elevated SLA requirements |
| Business impact | Revenue-impacting, production-blocking, or safety issues elevate severity |
| Active incident match | Case linked to a known outage may have a different handling path |
| Prior case pattern | Repeat issues for the same customer in the same area elevate severity |
| Time sensitivity | Embargoed deadlines, contractual obligations, or regulatory timelines |

**Severity Levels:**

| Severity | Description | SLA Target |
|----------|-------------|------------|
| **Critical** | Business-stopping, data loss, safety issue, or production outage | 1 hour response, 4 hours to resolution path |
| **High** | Significant impact, workaround exists but degraded, or premium customer with impacted service | 4 hours response, 1 business day to resolution path |
| **Standard** | Normal operational issue, common request, or inquiry | 15-minute routing, 1 business day first response |
| **Low** | Informational inquiry, feature request, or non-urgent enhancement | 1 business day response, 3 business days to resolution path |

**Determine escalation conditions:**
- Critical severity → immediate escalation to team lead and escalation manager
- High severity with premium customer → escalated routing to senior agent
- Case matches active incident → link to incident and route to incident response team
- Repeat pattern (3+ cases in 30 days, same issue area) → flag for systemic review
- Customer explicitly requested escalation → flag regardless of assessed severity

**Calculate SLA path:**
- Determine applicable SLA based on severity and customer entitlement tier
- Calculate remaining time based on case creation timestamp
- Flag if SLA is at risk or already breached

### Step 3: Present Severity Recommendation

Present the assessment via Adaptive Card (invoke `render-ui` skill first):

- **Recommended severity** (Critical / High / Standard / Low) with supporting rationale
- **SLA path** — applicable SLA targets and remaining time
- **Escalation conditions met** (if any) with specific criteria matched
- **Customer impact assessment** — business impact, affected services, user count if known
- **Active incident linkage** — if the case matches a known incident
- **Recommended response path** — standard queue, priority queue, immediate escalation, or incident linkage
- **SLA status** — on track, at risk, or breached

After user confirms, update the case tracker with the severity, priority, and SLA path.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, SLA policy, severity rubric, active incidents |
| SearchM365 (connectors) | Pull customer entitlement tier from CRM |
| ReadFileContent | Read SLA policy, severity rubric, case data, context packet |

## Guardrails

- **Present severity as draft recommendation** — require explicit user confirmation before applying
- **Never downgrade customer-reported severity** without documenting the rationale
- **Flag escalation conditions** even if the overall severity appears low — escalation criteria are independent
- **Cross-reference with active incidents** — cases linked to known outages should be flagged
- **Include SLA countdown** in every assessment — remaining time must be visible
- **Never auto-apply severity to the tracker** — always present for review first
- **Log severity decision** — record severity, rationale, and escalation flags in the case tracker after confirmation
