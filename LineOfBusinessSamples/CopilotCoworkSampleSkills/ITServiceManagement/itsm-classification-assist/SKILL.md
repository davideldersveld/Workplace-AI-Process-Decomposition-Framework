---
name: itsm-classification-assist
description: |
  Classifies incident type, affected service, and initial diagnosis path
  based on the incident taxonomy and context packet.
  Use when user asks to "classify this incident",
  "what type of incident is this", "categorize ticket [ID]",
  "triage classification for [incident]",
  "what service is affected", "incident category for [ticket]",
  or "classify and categorize [issue]".
  Do NOT use for creating a new incident (use itsm-incident-intake),
  enriching incident context (use itsm-context-packet),
  assessing severity (use itsm-severity-recommend),
  routing to resolver groups (use itsm-assignment-router),
  or drafting user updates (use itsm-comms-drafter).
---

## Overview

Compares incident symptoms and context against the incident taxonomy to recommend a category, subcategory, affected service, and initial diagnosis path. Cross-references historical incidents for the same service or symptoms and presents classification with confidence levels and similar past incidents for analyst review.

This skill operates in "AI assist" mode — it reads and analyzes incident data but only presents classification as a recommendation via Adaptive Card. The analyst confirms or overrides the classification before it is recorded.

## When to Use

- An incident has been logged and enriched and needs type and service classification
- An analyst wants a recommended incident category before beginning investigation
- A ticket needs reclassification after new information or updated symptoms
- Triage needs to determine the affected service for routing purposes

## When NOT to Use

- Creating a new incident record — use itsm-incident-intake
- Assembling user, service, or asset context — use itsm-context-packet
- Assessing severity or escalation path — use itsm-severity-recommend
- Routing to a resolver group — use itsm-assignment-router
- Drafting user updates or handoff summaries — use itsm-comms-drafter
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident data and classification inputs", activeForm="Reading classification inputs")
TaskCreate(subject="Classify incident and present recommendation", activeForm="Classifying incident")
```

### Step 1: Read Classification Inputs

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Incident ID]")` then `ReadFileContent`
- If no context packet exists, note this as a prerequisite gap — recommend running itsm-context-packet first

**Read the incident taxonomy:**
- `SearchM365(sources=["files"], query="incident taxonomy")` then `ReadFileContent` — category definitions, subcategory structures, service-to-category mappings

**Read the service catalog:**
- `SearchM365(sources=["files"], query="service catalog")` then `ReadFileContent` — service definitions, supported applications, and service area boundaries

**Read the incident tracker:**
- `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent` — current incident data

**Check historical patterns:**
- `SearchM365(sources=["files"], query="[symptoms] incident classification")` — prior incidents with similar symptoms and their final classifications
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — historical classification patterns from the ITSM platform (if available)

### Step 2: Classify Incident Type

Compare incident attributes against the taxonomy. Consider:

**Symptom attributes:**
- Reported symptoms and error descriptions
- Affected functionality or service area
- Timing and pattern (intermittent, constant, triggered by specific action)
- Scope (single user, group, department, or widespread)

**Service attributes:**
- Service area as reported by the user
- Confirmed service from CMDB and service catalog data
- Service dependencies and upstream/downstream relationships

**Context attributes:**
- Recent changes to the affected service (potential change-related incidents)
- Related open incidents (potential widespread or major incident)
- Prior incidents for this service (recurring issue patterns)

### Incident Categories

Classify into taxonomy-defined categories. Common ITSM categories include:

| Category | Subcategory Examples | Description |
|----------|---------------------|-------------|
| **Hardware** | Laptop, desktop, peripheral, mobile device | Physical device issues |
| **Software** | Application error, crash, performance, compatibility | Software malfunction or behavior |
| **Network** | Connectivity, VPN, Wi-Fi, DNS, latency | Network access or performance issues |
| **Identity and access** | Login failure, password reset, permission request, MFA issue | Authentication and authorization issues |
| **Email and collaboration** | Outlook, Teams, SharePoint, OneDrive | Collaboration tool issues |
| **Printing** | Printer, print queue, print driver | Print service issues |
| **Telephony** | Phone system, voicemail, conference bridge | Voice communication issues |
| **Security** | Suspected phishing, malware alert, data loss concern | Security-related incidents (route to security team) |
| **Infrastructure** | Server, storage, database, cloud service | Backend infrastructure issues |
| **Business application** | ERP, CRM, HR system, finance system | Line-of-business application issues |

### Step 3: Identify Affected Service

Map the incident to the specific service from the service catalog:
- Match reported symptoms against known service descriptions
- Cross-reference with CMDB CI data if available
- If multiple services could be affected, list the primary and secondary candidates

### Step 4: Assess Classification Confidence

Rate confidence:

- **High confidence** — symptoms clearly match a single category and service; historical incidents with these symptoms consistently map to the same classification
- **Medium confidence** — primary classification is supported but some symptoms could indicate alternative categories; the service mapping has moderate ambiguity
- **Low confidence** — symptoms are ambiguous, span multiple categories, or the affected service cannot be determined from available data; manual classification recommended

### Step 5: Identify Similar Past Incidents

Search for the top 3 most similar past incidents:
- Same service area with similar symptom descriptions
- Same category with similar reported issues
- Same reporter with recurring patterns

For each similar incident, note: Incident ID, date, final category, affected service, resolution summary, and time to resolve.

### Step 6: Present Classification

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Incident header** — Incident ID, Reported By, Symptom Summary
- **Recommended category and subcategory** — primary classification with description
- **Affected service** — confirmed or candidate service from the catalog
- **Confidence level** — High / Medium / Low with reasoning
- **Top 3 similar past incidents** — Incident ID, date, category, service, resolution, and time to resolve
- **Alternative classifications** — if confidence is medium or low, list alternatives with supporting indicators
- **Recent change flag** — if a recent change to the affected service was found, flag prominently as a potential contributing factor
- **Related open incidents** — if other open incidents affect the same service, flag as potentially related or widespread
- **Draft label** — "CLASSIFICATION RECOMMENDATION — analyst review required before recording"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find incident taxonomy, service catalog, incident tracker, prior incident classifications, context packet |
| SearchM365 (connectors) | Retrieve historical classification patterns from ITSM platform via Graph Connector |
| ReadFileContent | Read taxonomy, service catalog, context packet, tracker |

## Guardrails

- **Present classification as a recommendation only** — never auto-update the incident category in the tracker without analyst confirmation
- **Always display confidence level and reasoning** — the analyst must see why the classification was recommended
- **Show the top 3 classification candidates**, not just the top pick — the analyst needs context to make the best decision
- **If confidence is low, explicitly flag for manual classification** — do not present uncertain classifications as definitive
- **Never classify an incident as major** through this skill — major incident determination requires the severity assessment step (itsm-severity-recommend) and human confirmation
- **Flag recent changes** to the affected service prominently — change-related incidents are a common pattern, but do not assert causation
- **Flag related open incidents** affecting the same service — multiple incidents on the same service may indicate a widespread issue or major incident candidate
- **If the classification maps to security**, flag for security team routing — security incidents require specialized handling
- **Do not modify the incident tracker** — this skill presents analysis only; tracker updates happen after analyst confirmation
- **Never include resolution recommendations** in the classification output — classification determines the category; resolution belongs to the resolver group
- **Use taxonomy-defined categories only** — do not invent new categories; if the incident does not match any defined type, recommend manual classification with the closest matches noted
