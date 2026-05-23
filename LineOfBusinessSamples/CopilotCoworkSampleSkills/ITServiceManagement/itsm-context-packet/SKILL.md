---
name: itsm-context-packet
description: |
  Assembles affected user profile, service and asset context, CMDB data,
  recent changes, related incidents, and prior history into a context
  packet for incident triage.
  Use when user asks to "build context for incident [ID]",
  "enrich this incident", "get context for ticket [number]",
  "what do we know about this incident",
  "incident context for [service]", "gather details for [ticket]",
  or "assemble context packet for [incident]".
  Do NOT use for creating a new incident (use itsm-incident-intake),
  classifying incident type (use itsm-classification-assist),
  assessing severity (use itsm-severity-recommend),
  routing to resolver groups (use itsm-assignment-router),
  or drafting user updates (use itsm-comms-drafter).
---

## Overview

Assembles a comprehensive context packet for an incident — affected user profile and org context, service and asset details from CMDB sync data, recent changes to the affected service, related open incidents and known issues, and prior incident history for the user or service. Produces a Word document stored in the SharePoint incident folder.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources but does not classify the incident, assess severity, or recommend resolution actions.

## When to Use

- An incident has been logged and needs user, service, and asset context before classification
- An analyst wants to understand what is known about the affected service and its recent changes
- A resolver group needs background context assembled before starting investigation
- An incident needs related-incident history and known-issue cross-reference

## When NOT to Use

- Creating a new incident record — use itsm-incident-intake
- Classifying the incident type or affected service — use itsm-classification-assist
- Assessing severity or escalation path — use itsm-severity-recommend
- Routing to a resolver group — use itsm-assignment-router
- Drafting user updates or handoff summaries — use itsm-comms-drafter
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident data and gather context sources", activeForm="Gathering context data")
TaskCreate(subject="Assemble context packet", activeForm="Assembling context packet")
```

### Step 1: Read Incident Data

Locate and read the incident tracker to identify the case:
- `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent` — find the incident record by Incident ID or reporter

Extract from the incident record:
- Incident ID, reported by, report channel
- Symptom description and affected service area
- Created date and SLA target

### Step 2: Gather User and Org Context

- `GetUserDetails` — affected user's profile, department, role, location, job title
- `GetManagerDetails` — reporting chain for escalation context
- Check if the user is flagged as VIP or executive in the directory
- `SearchM365(sources=["email"], query="from:[affected user] [service area]")` — recent emails from the affected user mentioning the service or symptoms (permission-scoped)

### Step 3: Gather Service and Asset Context

**CMDB sync data:**
- `SearchM365(sources=["files"], query="CMDB [service area]")` then `ReadFileContent` — service catalog entry, configuration items, asset details
- `SearchM365(sources=["files"], query="service ownership [service area]")` then `ReadFileContent` — service owner, support model, resolver group mapping

**Graph Connector data (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — CI records, service dependencies, and configuration details from the ITSM platform

**Asset inventory:**
- `SearchM365(sources=["files"], query="asset inventory [affected entity]")` then `ReadFileContent` — asset criticality, environment classification, business impact rating

### Step 4: Gather Recent Changes

- `SearchM365(sources=["files"], query="recent changes [service area]")` then `ReadFileContent` — changes deployed to the affected service in the last 7 days
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — change records from the ITSM platform for the affected CI (if available)

Flag any changes deployed within 48 hours of the incident report — these are potential contributing factors.

### Step 5: Gather Related Incidents and Known Issues

**Related open incidents:**
- Search the incident tracker for open incidents affecting the same service area
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — open incidents and known errors for the affected CI (if available)

**Prior incident history:**
- Search the tracker for prior incidents reported by the same user or affecting the same service in the last 90 days
- `SearchM365(sources=["files"], query="incident history [service area]")` — historical incident data

**Knowledge articles:**
- `SearchM365(sources=["files"], query="knowledge article [symptoms] OR [service area]")` then `ReadFileContent` — relevant knowledge base articles that may assist triage

**SOC and IT channel discussions:**
- `SearchM365(sources=["teams"], query="[service area] OR [symptoms]")` — recent discussions in IT channels mentioning the affected service

### Step 6: Assemble Context Packet

Produce a Word document (invoke `docx` skill) containing:

**Section 1: Incident Summary**
- Incident ID, reported by, report channel, symptom description
- Affected service area, created date, SLA target

**Section 2: Affected User Profile**
- Name, department, role, location, manager
- VIP or executive flag if applicable
- Prior incident history for this user (count and recent examples)

**Section 3: Service and Asset Context**
- Service catalog entry (service name, description, criticality, business owner)
- Configuration items and asset details
- Environment classification (production, staging, development)
- Support model and resolver group mapping

**Section 4: Recent Changes**
- Changes deployed to the affected service in the last 7 days
- Changes within 48 hours flagged as potential contributing factors
- Change owners and approval status

**Section 5: Related Open Incidents**
- Open incidents affecting the same service area
- Known issues or error records for the affected CI
- Pattern indicators if multiple users are reporting similar symptoms

**Section 6: Prior Incident History**
- Historical incidents for this service (last 90 days)
- Resolution patterns and common root causes
- Repeat incident indicators

**Section 7: Knowledge Articles**
- Relevant knowledge base articles for the reported symptoms
- Applicable troubleshooting guides or runbooks

**Section 8: Context Gaps**
- List any enrichment sources that returned no data or were unavailable
- Flag if CMDB sync data is more than 24 hours stale
- Note any Graph Connector sources that were not accessible
- Identify manual lookups the analyst may need to perform

**Document metadata:**
- Author: requesting analyst
- Created: current timestamp
- Incident reference: Incident ID

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find incident tracker, CMDB sync data, service catalog, change log, asset inventory, knowledge articles, incident history |
| SearchM365 (connectors) | Retrieve CI records, change records, open incidents, known errors from ITSM platform via Graph Connector |
| SearchM365 (email) | Find emails from affected user about the service or symptoms |
| SearchM365 (teams) | Find IT channel discussions about the affected service |
| ReadFileContent | Read tracker, CMDB data, service catalog, change log, knowledge articles |
| GetUserDetails | Affected user profile |
| GetManagerDetails | Reporting chain for escalation context |

## Guardrails

- **Only retrieve context for services and assets the requesting analyst has permission to view** — do not access data outside the analyst's authorization scope
- **Cite the source document and retrieval timestamp** for every data point in the context packet — CMDB and change data ages quickly
- **Flag if CMDB sync data is more than 24 hours stale** — stale data may lead to incorrect triage decisions
- **Do not include raw security logs, identity credential details, or infrastructure secrets** in the context packet
- **If a Graph Connector is unavailable**, note which context fields are missing and suggest manual lookup from the ITSM platform
- **Flag recent changes within 48 hours** as potential contributing factors — but do not assert causation; that is for the resolver group to determine
- **Never include resolution recommendations or root cause hypotheses** in the context packet — present facts and context only; analysis belongs in the classification and investigation skills
- **The ITSM platform is the source of truth** — note if the Excel tracker data conflicts with connector data and recommend the analyst verify in the source system
- **Never post context packet contents to general Teams channels** — the packet may contain user details and service configurations that should be shared only with authorized analysts
- **Store the context packet in the SharePoint incident folder** linked from the tracker row
