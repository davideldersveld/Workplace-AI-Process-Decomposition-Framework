---
name: sec-analyst-router
description: |
  Routes security cases to the correct analyst queue based on routing rules,
  classification, severity, and SOC operating model.
  Use when user asks to "route this case", "assign analyst for this alert",
  "who handles this type of alert", "queue this for investigation",
  "analyst assignment for [case ID]", "route to SOC team",
  or "escalate this case to [team]".
  Do NOT use for creating a new case (use sec-alert-intake),
  enriching alert context (use sec-enrichment-packet),
  classifying alert type or risk (use sec-risk-classifier),
  assessing severity (use sec-severity-recommend),
  or drafting investigation summaries (use sec-investigation-drafter).
---

## Overview

Routes security cases to the correct analyst queue or incident response path based on the SOC routing rules, alert classification, severity, and operating model. Resolves analyst assignments by specialization and availability, enforces routing matrix compliance, sends Teams notifications after analyst review, and creates calendar holds for SLA-driven deadlines.

This skill operates in "AI act within policy" mode for Medium and Low severity cases with clear routing rule matches. For Critical and High severity cases, it operates in "AI draft plus approve" mode — the routing recommendation is presented for SOC lead confirmation before execution.

## When to Use

- An alert case has been classified and assessed for severity and needs analyst assignment
- The SOC needs to determine which queue or team handles a specific case
- A case needs escalation routing to the incident response team
- A case needs re-routing after reclassification or severity change

## When NOT to Use

- Creating a new alert case — use sec-alert-intake
- Assembling entity, asset, or threat context — use sec-enrichment-packet
- Classifying the alert type or likely risk — use sec-risk-classifier
- Assessing severity or investigation path — use sec-severity-recommend
- Drafting investigation summaries or evidence requests — use sec-investigation-drafter
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and routing rules", activeForm="Reading routing inputs")
TaskCreate(subject="Route case and send notifications", activeForm="Routing case")
```

### Step 1: Read Routing Inputs

**Read the case data, classification, and severity:**
- `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent` — current case data with classification and severity

**Read the routing rules and SOC operating model:**
- `SearchM365(sources=["files"], query="SOC analyst routing rules")` then `ReadFileContent` — queue structure, specialization mapping, escalation paths
- `SearchM365(sources=["files"], query="SOC operating model")` then `ReadFileContent` — team structure, on-call schedule, queue capacity

**Resolve analysts and teams:**
- `SearchPeople` — resolve analyst names by specialization and queue
- `GetUserDetails` — verify assigned analyst's current role and incident response authorization
- `ListCalendarView` — check analyst availability before assignment

### Step 2: Determine Routing

Match the case to the routing rules based on:

| Routing Factor | Source |
|----------------|--------|
| Alert category | Classification from sec-risk-classifier |
| Severity level | Severity from sec-severity-recommend |
| Entity type | Case record (user, host, ip, application) |
| Environment | Enrichment packet (production, staging, development) |
| Investigation path | Severity recommendation (incident response, deep investigation, standard triage, monitoring) |
| Insider threat flag | Classification or enrichment indicators |

### Routing Queues

| Queue | Handles | Typical Cases |
|-------|---------|---------------|
| **Tier 1 — SOC Analyst** | Standard triage, monitoring, low-complexity cases | Policy violations, blocked phishing, informational alerts, known false-positive patterns |
| **Tier 2 — Senior SOC Analyst** | Deep investigation, complex triage, multi-entity cases | Credential compromise investigation, malware analysis, data staging review |
| **Incident Response Team** | Active incidents, confirmed breaches, critical-severity cases | Active intrusion, ransomware, data exfiltration, enterprise-wide threats |
| **Identity Security Team** | Identity-focused alerts, privilege escalation, access anomalies | Account takeover, MFA bypass, privileged identity compromise |
| **Insider Threat Team** | Cases flagged as potential insider threat | Data hoarding, unauthorized access patterns, policy evasion by authorized users |
| **SOC Lead** | Escalation path when no clear routing rule matches, or severity override is needed | Ambiguous cases, routing conflicts, capacity overflow |

### Step 3: Present Routing Recommendation

**For Critical and High severity cases**, present via Adaptive Card (invoke `render-ui` skill first) for SOC lead confirmation:

- **Case header** — Case ID, Alert Source, Detection Rule, Affected Entity
- **Classification and severity** — alert category, severity level, investigation path
- **Recommended queue** — target queue from routing rules
- **Recommended analyst** — specific assignment based on specialization and availability
- **Routing rationale** — which routing rule criteria matched
- **Escalation flags** — incident response activation, CISO notification, legal hold
- **SLA status** — time elapsed since intake, deadline for triage completion
- **Draft label** — "ROUTING RECOMMENDATION — SOC lead confirmation required"

**For Medium and Low severity cases** with clear routing rule matches, proceed to execution after brief confirmation.

### Step 4: Execute Routing (After Confirmation)

**Send Teams notification to the assigned analyst:**
- `PostMessage` — direct message to the assigned analyst with:
  - Case ID, alert category, severity level
  - Investigation path and recommended next steps
  - SLA deadline
  - Link to the SharePoint case folder and enrichment packet
  - Never include raw IOCs, affected account names, or hostnames in the message — reference by Case ID only

**Post assignment to the SOC coordination channel:**
- `PostChannelMessage` — post to the SOC channel with:
  - Case ID, severity level, assigned queue
  - Never include raw IOCs, affected account names, or investigation details in channel posts

**Create SLA deadline calendar hold:**
- `CreateEvent` — calendar event for the assigned analyst with the triage SLA deadline as a visible reminder

**Update tracker:**
- Record assigned analyst, queue, routing timestamp, routing rationale, and confirming reviewer in the alert tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find alert tracker, routing rules, SOC operating model |
| ReadFileContent | Read routing rules, operating model, tracker |
| SearchPeople | Resolve analyst names by specialization |
| GetUserDetails | Verify analyst role and authorization |
| ListCalendarView | Check analyst availability |
| PostMessage | Direct message to assigned analyst |
| PostChannelMessage | SOC channel assignment notification |
| CreateEvent | SLA deadline calendar hold |

## Guardrails

- **Only assign to analysts listed in the approved SOC routing rules** — ad hoc assignments are not permitted
- **If no matching routing rule exists**, escalate to the SOC lead rather than guessing
- **For Critical and High severity cases**, verify the assigned analyst has incident response authorization before routing — present for SOC lead confirmation
- **Never route a case marked as potential incident to a Tier 1 analyst** — incident-path cases must go to Tier 2 or incident response
- **Never include raw IOCs, affected account names, or hostnames in SOC channel posts** — use Case ID references only; full details go in direct messages to the assigned analyst
- **If the case involves potential insider threat**, route only to the designated insider threat team and suppress general SOC channel visibility
- **Include Case ID and severity in every outbound Teams message** for traceability
- **Include SLA deadline** in all routing notifications — time-to-triage is operationally critical
- **Log the routing decision** — record assigned analyst, queue, rationale, and confirming reviewer in the tracker for audit
- **Never execute containment actions as part of routing** — routing assigns investigation ownership only; all response actions are human-owned
- **Never modify case classification or severity during routing** — routing uses the existing classification and severity; changes require re-running the appropriate upstream skill
