---
name: sec-enrichment-packet
description: |
  Assembles entity, asset, threat context, related alerts, prior investigation
  history, and applicable playbook references into an enrichment packet for
  security alert cases.
  Use when user asks to "enrich this alert", "build context for security case",
  "what do we know about this entity", "gather threat context for [case ID]",
  "alert enrichment for [entity]", "assemble context packet for [case]",
  or "security case context for [alert]".
  Do NOT use for creating a new case (use sec-alert-intake),
  classifying alert type or risk (use sec-risk-classifier),
  assessing severity (use sec-severity-recommend),
  routing to analysts (use sec-analyst-router),
  or drafting investigation summaries (use sec-investigation-drafter).
---

## Overview

Assembles a comprehensive enrichment packet for a security alert case — affected entity profile, asset criticality, environment classification, related alerts from the last 30 days, prior investigation history, applicable playbook references, and threat context. Produces a Word document classified as internal/confidential and stored in the SharePoint security cases folder.

This skill operates in "AI act within policy" mode — it retrieves approved enrichment context from defined sources but does not interpret threat attribution, make containment recommendations, or assess severity.

## When to Use

- A security alert case has been created and needs entity, asset, and threat context before classification
- An analyst wants to understand what is known about an affected user, host, or application
- A case needs related-alert history and prior investigation context assembled
- An analyst needs the applicable playbook reference for a specific detection rule

## When NOT to Use

- Creating a new alert case — use sec-alert-intake
- Classifying the alert type or likely risk — use sec-risk-classifier
- Assessing severity or investigation path — use sec-severity-recommend
- Routing to an analyst queue or SOC team — use sec-analyst-router
- Drafting investigation summaries or evidence requests — use sec-investigation-drafter
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and gather enrichment sources", activeForm="Gathering enrichment data")
TaskCreate(subject="Assemble enrichment packet", activeForm="Assembling enrichment packet")
```

### Step 1: Read Case Data

Locate and read the alert tracker to identify the case:
- `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent` — find the case record by Case ID or affected entity

Extract from the case record:
- Case ID
- Alert source and detection rule
- Affected entity and entity type
- Alert description
- Created date and SLA target

### Step 2: Gather Entity and Asset Context

**User entity enrichment** (if entity type is user):
- `GetUserDetails` — affected user's profile, department, role, location, job title
- `GetManagerDetails` — reporting chain for escalation context
- `SearchM365(sources=["email"], query="[affected user]")` — recent emails from or to the affected user if the alert involves phishing or email-based threats (permission-scoped to the requesting analyst's access)

**Host or application entity enrichment:**
- `SearchM365(sources=["files"], query="asset inventory [entity name]")` then `ReadFileContent` — asset criticality, environment classification, service ownership
- `SearchM365(sources=["connectors"], connector_ids=["defender-connector"])` — endpoint context, device health, recent detections on the affected host (if available)

**Identity context:**
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — identity anomalies, sign-in patterns, or privilege escalation events for the affected entity (if available)

### Step 3: Gather Related Alerts and Prior History

**Related alerts:**
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — alerts for the same entity in the last 30 days
- Search the alert tracker for prior cases involving the same affected entity or same detection rule

**Prior investigation history:**
- `SearchM365(sources=["files"], query="investigation [affected entity]")` — prior investigation reports or case summaries in SharePoint

**SOC discussions:**
- `SearchM365(sources=["teams"], query="[affected entity] OR [detection rule]")` — recent SOC channel discussions mentioning the entity or rule

### Step 4: Gather Playbook and Threat Context

**Playbook reference:**
- `SearchM365(sources=["files"], query="playbook [detection rule] OR [alert category]")` then `ReadFileContent` — applicable detection playbook, response procedures, and investigation steps

**Threat context:**
- `SearchM365(sources=["files"], query="threat intel [detection rule] OR [attack technique]")` then `ReadFileContent` — curated threat intelligence briefs relevant to the detection rule or attack pattern

### Step 5: Assemble Enrichment Packet

Produce a Word document (invoke `docx` skill) containing:

**Section 1: Case Summary**
- Case ID, alert source, detection rule, affected entity, entity type
- Alert description
- Created date and SLA target

**Section 2: Entity Profile**
- User profile (name, department, role, location, manager) or asset profile (hostname, criticality, environment, service owner)
- Privilege level or asset classification
- If the entity is a privileged user or executive, flag for elevated handling

**Section 3: Asset and Environment Context**
- Asset criticality rating
- Environment classification (production, staging, development, internet-facing)
- Data classification of systems or data the entity can access
- Service ownership and support contacts

**Section 4: Related Alerts (Last 30 Days)**
- Table of related alerts with Case ID, date, detection rule, status, and outcome
- Pattern summary if multiple related alerts exist

**Section 5: Prior Investigation History**
- Summary of prior investigations involving this entity
- Outcomes and any remediation actions taken previously

**Section 6: Playbook Reference**
- Applicable playbook name and link
- Key investigation steps from the playbook
- Recommended evidence collection targets

**Section 7: Threat Context**
- Detection rule description and known attack patterns
- Relevant threat intelligence brief summary
- MITRE ATT&CK technique reference if applicable

**Section 8: Enrichment Gaps**
- List any enrichment sources that returned no data or were unavailable
- Flag any Graph Connector sources that were not accessible
- Note any manual enrichment the analyst may need to perform

**Document metadata:**
- Sensitivity label: Confidential — Security Operations
- Author: requesting analyst
- Created: current timestamp
- Case reference: Case ID

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find alert tracker, asset inventory, playbooks, threat intel briefs, prior investigation reports |
| SearchM365 (connectors) | Retrieve alerts, endpoint context, identity anomalies from Sentinel and Defender via Graph Connector |
| SearchM365 (email) | Find emails related to phishing or email-based alerts (permission-scoped) |
| SearchM365 (teams) | Find SOC channel discussions about the affected entity |
| ReadFileContent | Read tracker, asset data, playbooks, threat intel, prior reports |
| GetUserDetails | Affected user profile |
| GetManagerDetails | Reporting chain for escalation context |

## Guardrails

- **Only retrieve context for entities the requesting analyst has permission to view** — do not access data outside the analyst's authorization scope
- **Never include raw credentials, tokens, or decrypted payloads** in the enrichment packet
- **Cite the source system and retrieval timestamp** for every data point in the packet — enrichment data ages quickly in security contexts
- **Flag if any critical enrichment source returned no data** — missing SIEM, endpoint, or identity context materially affects triage quality
- **Mark the output document** with sensitivity label: Confidential — Security Operations
- **Never post enrichment packet contents to general Teams channels** — only to the designated SOC channel or direct messages to authorized analysts
- **If the alert involves executive accounts or privileged identities**, flag for elevated handling and restricted distribution in the packet header
- **Never include investigation hypotheses or threat attribution** in the enrichment packet — present facts and context only; analysis belongs in the classification and investigation summary skills
- **Never include containment recommendations** — enrichment assembles context, it does not prescribe response actions
- **If prior investigations for this entity were flagged as insider threat**, note the existence of prior elevated handling without disclosing investigation details — refer the analyst to the insider threat team
- **Store the enrichment packet in the SharePoint security cases folder** with version history enabled — investigation artifacts must be preserved for audit
