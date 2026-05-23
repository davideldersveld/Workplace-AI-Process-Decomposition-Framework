---
name: itsm-severity-recommend
description: |
  Assesses incident severity, impact, urgency, and escalation path based
  on the severity matrix, service criticality, and business impact.
  Use when user asks to "assess severity for [incident]",
  "what priority should this be", "escalation check for incident [ID]",
  "severity recommendation for [ticket]",
  "how urgent is this incident", "priority assessment for [issue]",
  or "major incident check for [ticket]".
  Do NOT use for creating a new incident (use itsm-incident-intake),
  enriching incident context (use itsm-context-packet),
  classifying incident type (use itsm-classification-assist),
  routing to resolver groups (use itsm-assignment-router),
  or drafting user updates (use itsm-comms-drafter).
---

## Overview

Assesses incident severity using a standard impact-urgency matrix, factors in service criticality and business impact, identifies escalation needs, and presents the priority recommendation with SLA targets for analyst review. Flags potential major incident triggers and cross-references against open major incidents for correlation.

This skill operates in "AI draft plus approve" mode — every severity recommendation is presented for analyst review and confirmation. The analyst's final decision (accept or override) is logged for quality tracking.

## When to Use

- An incident has been classified and needs priority assessment
- An analyst wants to determine the correct escalation path before routing
- A ticket needs severity re-assessment after new information or scope change
- Triage needs to check whether an incident qualifies as a potential major incident

## When NOT to Use

- Creating a new incident record — use itsm-incident-intake
- Assembling user, service, or asset context — use itsm-context-packet
- Classifying the incident type or affected service — use itsm-classification-assist
- Routing to a resolver group — use itsm-assignment-router
- Drafting user updates or handoff summaries — use itsm-comms-drafter
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident data and severity inputs", activeForm="Reading severity inputs")
TaskCreate(subject="Assess severity and present recommendation", activeForm="Assessing severity")
```

### Step 1: Read Severity Inputs

**Read the incident data and classification:**
- `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent` — current incident data with classification
- `SearchM365(sources=["files"], query="context packet [Incident ID]")` then `ReadFileContent` — user, service, and asset context

**Read the severity matrix and escalation policy:**
- `SearchM365(sources=["files"], query="severity matrix")` then `ReadFileContent` — impact-urgency matrix, priority level definitions, SLA targets
- `SearchM365(sources=["files"], query="escalation policy")` then `ReadFileContent` — escalation triggers and notification requirements

**Check service criticality:**
- `SearchM365(sources=["files"], query="service criticality [service area]")` then `ReadFileContent` — business criticality rating, revenue impact, user population
- `GetUserDetails` — affected user's role and VIP status

**Check for related major incidents:**
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — current open major incidents that might be related (if available)
- Search the incident tracker for open P1/P2 incidents affecting the same service

### Step 2: Assess Impact

Determine the scope of impact using the standard ITIL impact model:

| Impact Level | Criteria |
|--------------|----------|
| **Enterprise-wide** | All users or critical business services affected; revenue-impacting; regulatory deadline at risk |
| **Department or site** | Entire department, office location, or significant user group affected |
| **Multiple users** | Group of users affected but contained to a team or functional area |
| **Single user** | One user affected; workaround may be available |

### Step 3: Assess Urgency

Determine time sensitivity:

| Urgency Level | Criteria |
|---------------|----------|
| **Critical** | Service is completely unavailable; no workaround; business-critical operation blocked; regulatory or compliance deadline at risk |
| **High** | Service is severely degraded; workaround exists but is insufficient; significant productivity impact |
| **Medium** | Service is impaired but functional; workaround available; moderate productivity impact |
| **Low** | Inconvenience or cosmetic issue; full workaround available; minimal business impact |

### Step 4: Calculate Priority

Apply the impact-urgency matrix to determine priority:

| | Critical Urgency | High Urgency | Medium Urgency | Low Urgency |
|---|---|---|---|---|
| **Enterprise-wide** | P1 | P1 | P2 | P3 |
| **Department/site** | P1 | P2 | P2 | P3 |
| **Multiple users** | P2 | P2 | P3 | P4 |
| **Single user** | P2 | P3 | P3 | P4 |

### Priority Levels and SLA Targets

| Priority | Description | Response SLA | Resolution SLA | Escalation Path |
|----------|-------------|-------------|----------------|-----------------|
| **P1 — Critical** | Major business impact; service down for large population; no workaround | 15 minutes | 4 hours | Incident manager, service owner, major incident bridge |
| **P2 — High** | Significant impact; service severely degraded; limited workaround | 30 minutes | 8 hours | Incident manager, resolver group lead |
| **P3 — Standard** | Moderate impact; service impaired but functional; workaround available | 2 hours | 24 hours | Resolver group |
| **P4 — Low** | Minor impact; inconvenience; full workaround available | 4 hours | 72 hours | Resolver group |

### Step 5: Identify Escalation Needs

Check escalation triggers:

- **Major incident trigger** — P1 or P2 with enterprise-wide impact; multiple correlated incidents on the same service; revenue-critical service completely unavailable
- **Incident manager notification** — all P1 and P2 incidents
- **Service owner notification** — P1 incidents or any incident on a revenue-critical service
- **Executive notification** — P1 incidents affecting customer-facing services or regulatory compliance
- **VIP escalation** — incidents affecting executive users or VIP-flagged reporters regardless of calculated priority

### Step 6: Check for Major Incident Correlation

Cross-reference against open incidents:
- Multiple open incidents on the same service within a short time window
- Similar symptoms reported by multiple users
- Monitoring alerts correlated with user-reported incidents
- If correlation is found, flag as potential major incident candidate

### Step 7: Present Severity Recommendation

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Incident header** — Incident ID, Reported By, Category, Affected Service
- **Impact assessment** — level with supporting evidence (user population, service criticality)
- **Urgency assessment** — level with supporting evidence (workaround availability, business deadline)
- **Recommended priority** — P1 / P2 / P3 / P4 with the matrix rationale
- **SLA targets** — response and resolution targets for the recommended priority
- **Escalation recommendations** — incident manager, service owner, executive notifications as applicable
- **Major incident trigger warning** — if the incident meets major incident criteria, display a prominent warning with instructions for the analyst
- **Related open incidents** — if correlation with other incidents is found, flag as potential major incident candidate
- **VIP flag** — if the reporter or affected users include VIP or executive personnel
- **Draft label** — "SEVERITY RECOMMENDATION — analyst review and confirmation required"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find severity matrix, escalation policy, service criticality, incident tracker, context packet |
| SearchM365 (connectors) | Retrieve open major incidents from ITSM platform via Graph Connector |
| ReadFileContent | Read severity matrix, escalation policy, service criticality, tracker, context packet |
| GetUserDetails | Affected user's role and VIP status |

## Guardrails

- **Never auto-set priority** — always present as a recommendation for analyst review and confirmation
- **If the recommendation is P1 or P2**, add an explicit major incident trigger warning with procedures for the analyst to follow
- **Require analyst confirmation or override** before priority is written to the tracker — log the recommended priority and the analyst's final decision
- **If the incident involves executive users, VIP reporters, or revenue-critical services**, auto-flag for escalation review regardless of calculated priority
- **Never declare a major incident** — major incident declaration is exclusively human-owned; the skill provides evidence and recommendations to support the human decision
- **Cross-reference against open major incidents** to identify potential correlation — related incidents should be linked and investigated together
- **Never downgrade priority** without analyst review — if new evidence suggests lower priority, present the recommendation but do not auto-update
- **Include SLA targets** in every severity recommendation — time-to-triage and time-to-resolve are operationally critical
- **Never recommend remediation actions** — severity assessment determines priority and escalation path; the resolver group determines remediation
- **Do not include internal infrastructure details** (hostnames, IP addresses, server names) in the severity recommendation output — use service names and business impact language
