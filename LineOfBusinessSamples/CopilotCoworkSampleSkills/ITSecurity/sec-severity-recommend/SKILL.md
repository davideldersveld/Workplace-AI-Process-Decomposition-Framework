---
name: sec-severity-recommend
description: |
  Assesses severity, blast radius, urgency, and investigation path for
  security alert cases based on the severity matrix and environment context.
  Use when user asks to "assess severity for this alert",
  "what priority should this case be", "investigation path for this alert",
  "severity recommendation for security case [ID]",
  "how urgent is this alert", "blast radius for [case]",
  or "escalation assessment for [alert]".
  Do NOT use for creating a new case (use sec-alert-intake),
  enriching alert context (use sec-enrichment-packet),
  classifying alert type or risk (use sec-risk-classifier),
  routing to analysts (use sec-analyst-router),
  or drafting investigation summaries (use sec-investigation-drafter).
---

## Overview

Assesses severity, blast radius, urgency, and recommended investigation path for a classified security alert case. Compares case attributes against the severity matrix, factors in asset criticality and data classification, identifies escalation needs, and presents containment considerations as information only. Presents the severity recommendation for analyst review with logged rationale.

This skill operates in "AI draft plus approve" mode — every severity recommendation is presented for analyst review and confirmation. The analyst's final decision (accept or override) is logged for override tracking.

## When to Use

- An alert has been classified and needs severity and priority assessment
- An analyst wants a recommended investigation path before beginning work
- A case needs severity re-assessment after new evidence or reclassification
- Triage needs to determine escalation requirements for a case

## When NOT to Use

- Creating a new alert case — use sec-alert-intake
- Assembling entity, asset, or threat context — use sec-enrichment-packet
- Classifying the alert type or likely risk — use sec-risk-classifier
- Routing to an analyst queue or SOC team — use sec-analyst-router
- Drafting investigation summaries or evidence requests — use sec-investigation-drafter
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and severity inputs", activeForm="Reading severity inputs")
TaskCreate(subject="Assess severity and present recommendation", activeForm="Assessing severity")
```

### Step 1: Read Severity Inputs

**Read the case data and classification:**
- `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent` — current case data with classification
- `SearchM365(sources=["files"], query="enrichment packet [Case ID]")` then `ReadFileContent` — entity and asset context

**Read the severity matrix and escalation policy:**
- `SearchM365(sources=["files"], query="security severity matrix")` then `ReadFileContent` — severity level definitions, criteria, and thresholds
- `SearchM365(sources=["files"], query="security escalation policy")` then `ReadFileContent` — escalation triggers and notification requirements

**Read containment playbooks:**
- `SearchM365(sources=["files"], query="containment playbook [alert category]")` then `ReadFileContent` — available containment options for the classified alert type

**Check environment context:**
- `SearchM365(sources=["files"], query="asset criticality [affected entity]")` then `ReadFileContent` — asset criticality ratings and data classification
- `GetUserDetails` — affected entity's role and privilege level if user-type entity

**Check for related incidents:**
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — current active incidents that might be related; recent high-severity cases in the same environment (if available)

### Step 2: Assess Severity

Apply the severity matrix criteria:

| Severity | Criteria | SLA Target |
|----------|----------|------------|
| **Critical** | Active intrusion confirmed or likely; data exfiltration in progress; ransomware execution detected; privileged account compromise on production systems; multiple correlated high-severity alerts suggesting a campaign | 15 minutes to triage, immediate incident response activation |
| **High** | Credential compromise with evidence of misuse; malware on production or critical systems; data staging or unusual bulk access; privileged identity anomaly; phishing with confirmed credential submission | 15 minutes to triage, 1 hour to initial investigation |
| **Medium** | Endpoint malware detection on non-critical systems; policy violation with security implications; phishing delivery without confirmed credential compromise; identity anomaly without confirmed misuse | 1 hour to triage, 4 hours to initial investigation |
| **Low** | Known false-positive patterns; policy compliance drift without active threat indicators; informational alerts; blocked attack attempts with no evidence of success | 4 hours to triage, next business day investigation |

### Step 3: Assess Blast Radius

Determine the scope of potential impact:

- **Affected users** — single user, team, department, or organization-wide
- **Affected systems** — single endpoint, server group, network segment, or enterprise-wide
- **Data exposure** — no data at risk, non-sensitive data, sensitive data (PII, PCI, HIPAA), or regulated data with notification obligations
- **Service impact** — no service disruption, degraded performance, partial outage, or full service disruption

### Step 4: Assess Urgency

Determine time sensitivity:

- **Active threat** — threat actor is currently active based on real-time indicators (highest urgency)
- **Recent activity** — indicators suggest activity within the last 24 hours
- **Historical detection** — detection triggered on historical data or log analysis (lower urgency)
- **Dwell time indicators** — evidence suggests the threat has been present for an extended period (elevated urgency due to potential scope)

### Step 5: Recommend Investigation Path

| Investigation Path | When to Recommend |
|--------------------|-------------------|
| **Incident response** | Critical severity; active intrusion; data breach indicators; ransomware |
| **Deep investigation** | High severity; credential compromise; lateral movement; data staging |
| **Standard triage** | Medium severity; contained threats; policy violations; phishing without compromise |
| **Monitoring only** | Low severity; blocked attempts; known false-positive patterns; informational alerts |

### Step 6: Identify Escalation Needs

Check escalation triggers from the escalation policy:

- **SOC lead notification** — all Critical and High severity cases
- **Incident response team activation** — Critical severity or confirmed active intrusion
- **CISO notification** — Critical severity involving regulated data, executive accounts, or enterprise-wide impact
- **Legal notification** — potential data breach involving PII, PCI, or HIPAA data
- **Regulatory notification** — confirmed breach of regulated data with mandatory reporting obligations

### Step 7: Identify Containment Considerations

Present available containment options from the playbook as **information only**:

- Account actions (disable account, reset credentials, revoke sessions)
- Endpoint actions (isolate host, run scan, quarantine files)
- Network actions (block IP, restrict access, update firewall rules)
- Data actions (revoke sharing, restrict access, preserve evidence)

### Step 8: Check for Campaign Correlation

Cross-reference against open cases:
- Same detection rule across multiple entities
- Same affected entity across multiple detection rules
- Temporal clustering of related alerts
- If correlation is found, flag as potential campaign and recommend coordinated investigation

### Step 9: Present Severity Recommendation

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Alert Source, Detection Rule, Affected Entity, Classification
- **Recommended severity** — Critical / High / Medium / Low with rationale
- **Blast radius** — affected users, systems, data types, service impact
- **Urgency** — active threat, recent, historical, or dwell time indicators
- **Investigation path** — incident response, deep investigation, standard triage, or monitoring
- **Escalation recommendations** — SOC lead, incident response, CISO, legal, regulatory
- **Containment considerations** — available options from the playbook (information only, clearly labeled as requiring human authorization)
- **Campaign correlation** — related cases if found
- **SLA target** — triage and investigation deadlines for the recommended severity
- **Incident declaration advisory** — if Critical or High, explicit guidance on incident declaration procedures
- **Draft label** — "SEVERITY RECOMMENDATION — analyst review and confirmation required"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find severity matrix, escalation policy, containment playbooks, asset criticality, alert tracker, enrichment packet |
| SearchM365 (connectors) | Retrieve active incidents and related cases from Sentinel via Graph Connector |
| ReadFileContent | Read severity matrix, escalation policy, playbooks, tracker, enrichment packet |
| GetUserDetails | Affected entity's role and privilege level |

## Guardrails

- **Never auto-set severity** — always present as a recommendation for analyst review and confirmation
- **If the recommendation is Critical or High**, add an explicit incident declaration advisory with procedures for the analyst
- **Require analyst confirmation or override** before severity is written to the tracker — log the recommended severity and the analyst's final decision
- **If the alert involves privileged accounts, production systems, or regulated data**, auto-escalate visibility to the SOC lead regardless of calculated severity
- **Never recommend or execute containment actions** — only describe available options from the playbook and note that they require human authorization; containment is always a human decision
- **Never declare an incident** — incident declaration is exclusively human-owned; the skill provides evidence and recommendations to support the human decision
- **Cross-reference against open incidents** to identify potential campaign correlation — related activity should be investigated together
- **Never downgrade severity** without analyst review — if new evidence suggests lower severity, present the recommendation but do not auto-update
- **Include SLA target** in every severity recommendation — time-to-triage is operationally critical for security
- **Do not include raw IOCs, affected account credentials, or investigation hypotheses** in the severity recommendation output
