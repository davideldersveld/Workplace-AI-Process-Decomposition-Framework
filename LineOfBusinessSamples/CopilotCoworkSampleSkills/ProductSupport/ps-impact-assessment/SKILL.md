---
name: ps-impact-assessment
description: |
  Assesses customer impact and urgency for an escalated support case and
  recommends severity, response path, and incident linkage.
  Use when user asks to "assess impact for escalation [ID]",
  "what severity is this escalation",
  "check urgency for [case]",
  "is this a critical escalation",
  "customer impact for [escalation ID]",
  or "severity recommendation for [case]".
  Do NOT use for creating a new escalation record (use ps-escalation-intake),
  gathering evidence and context (use ps-evidence-packet),
  classifying the issue or defect path (use ps-defect-classifier),
  routing to engineering (use ps-engineering-routing),
  or drafting handoff communications (use ps-handoff-drafter).
---

## Overview

Evaluates the customer impact, business urgency, and technical severity of an escalated support case using the organization's severity rubric, customer entitlement data, affected user scope, and incident criteria. Recommends a severity level, response path, and incident linkage decision. Presents the impact assessment for escalation engineer review and explicit confirmation before any severity is applied.

This skill operates in "AI draft plus approve" mode — severity recommendations are generated as drafts for escalation engineer review. Sev 1 and Sev 2 recommendations require product support manager approval before routing proceeds.

## When to Use

- An escalation has been classified by ps-defect-classifier and needs a severity and impact assessment
- An escalation engineer needs to determine the urgency and response path for a case
- A severity reassessment is needed after new evidence or customer impact information arrives
- An escalation may meet incident criteria and needs evaluation for incident linkage

## When NOT to Use

- Creating a new escalation record — use ps-escalation-intake
- Gathering case evidence, logs, and telemetry context — use ps-evidence-packet
- Classifying the issue type or suspected defect path — use ps-defect-classifier
- Routing the escalation to an engineering owner or queue — use ps-engineering-routing
- Drafting the engineering handoff or customer update — use ps-handoff-drafter
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read severity rubric and case data", activeForm="Evaluating impact")
TaskCreate(subject="Present impact assessment", activeForm="Assessing severity")
```

### Step 1: Read Impact Assessment Inputs

**Read the escalation record and classification:**
- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — Escalation ID, Product Area, Issue Type, Classification, SLA Deadline

**Read the evidence packet:**
- `SearchM365(sources=["files"], query="evidence packet [escalation ID]")` then `ReadFileContent` — customer context, affected users, business impact, evidence completeness

**Read the severity rubric:**
- `SearchM365(sources=["files"], query="severity rubric")` then `ReadFileContent` — severity level definitions, criteria thresholds, and escalation triggers

**Read incident criteria:**
- `SearchM365(sources=["files"], query="incident criteria")` then `ReadFileContent` — conditions that trigger incident declaration

**Read customer entitlement data:**
- `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — customer entitlement tier, SLA terms, active incidents

### Step 2: Assess Customer Impact

Evaluate the customer impact dimensions:

| Dimension | Assessment |
|-----------|-----------|
| **Affected scope** | Single user, team, department, or entire organization |
| **Business function impact** | Which business processes are affected and how critical they are |
| **Workaround availability** | Is there a viable workaround, and how effective is it |
| **Data risk** | Is there risk of data loss, corruption, or exposure |
| **Revenue impact** | Is the issue affecting revenue-generating activities |
| **Account tier** | Enterprise, premium, or standard — affects response path |

### Step 3: Determine Recommended Severity

Apply the severity rubric to recommend a severity level:

| Severity | Criteria | Response Path |
|----------|---------|--------------|
| **Sev 1 — Critical** | Complete loss of service, data at risk, or critical business function blocked for enterprise customer with no workaround | Immediate engineering engagement, incident bridge consideration, product support manager approval required |
| **Sev 2 — High** | Major feature impaired, significant business impact, or enterprise customer blocked with limited workaround | Same-day engineering engagement, elevated priority routing |
| **Sev 3 — Standard** | Feature issue with available workaround, moderate business impact, or non-critical function affected | Standard SLA engineering engagement |
| **Sev 4 — Low** | Minor issue, cosmetic problem, documentation question, or feature request with minimal business impact | Standard queue, next available cycle |

### Step 4: Evaluate Incident Linkage

Determine whether the escalation should be linked to or trigger an incident:

| Decision | Criteria |
|----------|---------|
| **Link to existing incident** | Active incident exists for the same product area and symptom pattern |
| **Create new incident** | Escalation meets incident criteria (multiple customers affected, service degradation, data risk) and no existing incident covers it |
| **No incident action** | Escalation is isolated to a single customer or known configuration issue |

**Check active incidents:**
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — active incidents for the same product area
- `SearchM365(sources=["files"], query="active incidents")` then `ReadFileContent` — incident list if connector not available

### Step 5: Assess Evidence Completeness for Routing

Determine whether the evidence packet is sufficient for engineering handoff:

| Readiness Level | Criteria |
|----------------|---------|
| **Ready for engineering** | Log bundle present, reproduction steps documented, environment details captured, customer impact quantified |
| **Conditionally ready** | Most evidence present but minor gaps remain (missing specific log type, partial repro steps) |
| **Not ready** | Critical evidence missing (no logs, no repro steps, no environment details) — recommend additional collection before routing |

### Step 6: Present Impact Assessment

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Escalation header** — Escalation ID, Customer, Product Area, Classification
- **Customer impact summary** — affected scope, business function impact, workaround availability
- **Recommended severity** — Sev 1/2/3/4 with rubric-based justification
- **Response path** — recommended engineering engagement timeline and priority
- **Incident linkage** — link to existing, create new, or no action — with rationale
- **Evidence completeness** — ready, conditionally ready, or not ready for engineering
- **SLA status** — time remaining on the 4-hour SLA deadline
- **Approval requirement** — "Requires product support manager approval" for Sev 1 and Sev 2
- **Draft label** — "IMPACT ASSESSMENT — escalation engineer confirmation required before severity is applied"

### Step 7: Apply Severity (After Confirmation)

After the escalation engineer confirms (and product support manager approves for Sev 1/2):
- Update the Severity field in the escalation tracker
- Update Status to "Assessed — Ready for Routing" (or "Assessed — Evidence Needed" if not ready)
- Record the severity rationale and confirming user

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find escalation tracker, evidence packet, severity rubric, incident criteria |
| SearchM365 (connectors) | Pull customer entitlement data, active incidents from support and issue tracker connectors |
| ReadFileContent | Read severity rubric, incident criteria, evidence packet, tracker |

## Guardrails

- **Present severity as a draft recommendation** — require explicit escalation engineer confirmation before applying to the tracker
- **Require product support manager approval for Sev 1 and Sev 2** — high-severity declarations affect engineering prioritization and customer expectations
- **Never downgrade a severity** that was set by a previous human decision without documenting the rationale — severity downgrades require explicit justification
- **Flag any escalation that meets incident criteria** even if the overall severity appears low — incident detection should not depend solely on severity level
- **Cross-reference with active incidents** — detect escalations that are part of known outages to avoid duplicate investigation
- **Include the 4-hour SLA countdown** in every assessment output — urgency context affects routing decisions
- **Never fabricate customer impact data** — if affected user count or business impact is unknown, report it as unknown rather than estimating
- **Flag evidence completeness issues before routing** — an escalation with insufficient evidence should not be handed to engineering, as it will result in a back-and-forth that wastes time
- **Never make timeline commitments** in the impact assessment — severity determines priority, not delivery dates
- **Distinguish between customer-reported severity and assessed severity** — if the customer considers the issue critical but the rubric suggests standard, note both perspectives
