---
name: ps-defect-classifier
description: |
  Classifies an escalated support case by product area, issue type, and
  suspected defect path, with duplicate and known-issue detection.
  Use when user asks to "classify this escalation [ID]",
  "what product area is this defect in",
  "categorize defect for escalation [ID]",
  "triage this product issue",
  "is this a known issue",
  or "check if this is a duplicate defect".
  Do NOT use for creating a new escalation record (use ps-escalation-intake),
  gathering evidence and context (use ps-evidence-packet),
  assessing customer impact or severity (use ps-impact-assessment),
  routing to engineering (use ps-engineering-routing),
  or drafting handoff communications (use ps-handoff-drafter).
---

## Overview

Compares escalation evidence against the product area taxonomy, defect classification guide, known issues database, and prior escalation patterns to determine the product area, issue type, suspected defect path, and potential duplicates. Surfaces classification recommendations with confidence levels and supporting evidence for escalation engineer review via Adaptive Card.

This skill operates in "AI assist" mode — it reads and analyzes escalation data but only presents classification recommendations. The escalation engineer reviews and confirms before any tracker updates are made.

## When to Use

- An escalation has an evidence packet assembled and needs product area and defect path classification
- An escalation engineer wants to determine whether an escalation matches a known issue or existing defect
- A new escalation needs to be checked for duplicates against the issue tracker and prior escalations
- A reclassification is needed after new evidence changes the suspected product area

## When NOT to Use

- Creating a new escalation record — use ps-escalation-intake
- Gathering case evidence, logs, and telemetry context — use ps-evidence-packet
- Assessing customer impact or recommending severity — use ps-impact-assessment
- Routing the escalation to an engineering owner or queue — use ps-engineering-routing
- Drafting the engineering handoff or customer update — use ps-handoff-drafter
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read evidence and classification references", activeForm="Analyzing escalation")
TaskCreate(subject="Present classification recommendation", activeForm="Classifying defect")
```

### Step 1: Read Classification Inputs

**Read the escalation record:**
- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — Escalation ID, Product Area (current), Issue Summary, Suspected Defect Path

**Read the evidence packet:**
- `SearchM365(sources=["files"], query="evidence packet [escalation ID]")` then `ReadFileContent` — technical details, environment, reproduction steps, case timeline

**Read classification references:**
- `SearchM365(sources=["files"], query="product area taxonomy")` then `ReadFileContent` — standard product area hierarchy with component definitions
- `SearchM365(sources=["files"], query="defect classification guide")` then `ReadFileContent` — issue type definitions (defect, configuration, documentation, feature gap) with classification criteria
- `SearchM365(sources=["files"], query="engineering ownership map")` then `ReadFileContent` — product area to engineering team mapping

### Step 2: Classify Product Area and Component

Compare the escalation's symptoms, affected feature, and technical evidence against the product area taxonomy:

- Match the reported issue to the most specific product area and component
- Consider the technical environment (product version, platform, configuration) as additional classification signal
- If the issue spans multiple product areas, identify the primary area and note secondary areas

### Step 3: Determine Issue Type

Classify the issue into one of the standard types:

| Issue Type | Criteria |
|-----------|---------|
| **Defect** | Product behavior differs from documented specification or expected behavior |
| **Configuration** | Issue caused by customer-specific configuration, environment, or integration setup |
| **Documentation** | Product works as designed but documentation is missing, incorrect, or misleading |
| **Feature gap** | Customer needs functionality that does not currently exist in the product |
| **Performance** | Product functions correctly but response time, throughput, or resource usage is unacceptable |
| **Regression** | Functionality that previously worked correctly has broken in a recent release |

### Step 4: Determine Suspected Defect Path

Based on the classification and evidence, recommend the likely investigation path:

- Which code area, service, or component is most likely involved
- What the engineering team should investigate first
- Whether the issue is likely in the application layer, infrastructure, API, or data layer

### Step 5: Check for Known Issues and Duplicates

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — search for known issues and existing defects matching the product area, symptoms, and error patterns

**Via SharePoint (if manual sync):**
- `SearchM365(sources=["files"], query="known issues [product area]")` then `ReadFileContent` — known issues list

**Check for duplicates:**
- Compare against the escalation tracker for other open escalations with the same product area and similar symptoms
- Compare against the issue tracker for existing open defects matching the pattern

For each match found:
- Issue or escalation ID
- Summary and current status
- Match basis (symptom overlap, product area, error pattern)
- Confidence level (high, medium, low)
- Workaround available (yes/no, with details if yes)

### Step 6: Assess Classification Confidence

| Confidence Level | Criteria |
|-----------------|---------|
| **High** | Clear symptom match to product area, evidence supports the classification, prior cases confirm the pattern |
| **Medium** | Product area is likely but evidence is partial, or the issue could span multiple areas |
| **Low** | Symptoms are ambiguous, evidence is thin, or the issue does not clearly match any existing taxonomy entry |

### Step 7: Present Classification Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Escalation header** — Escalation ID, Customer, Current Product Area, Issue Summary
- **Recommended classification:**
  - Product area and component
  - Issue type (defect, configuration, documentation, feature gap, performance, regression)
  - Suspected defect path
  - Confidence level with basis
- **Known issue matches** — list of matching known issues with status, workaround availability, and match confidence
- **Duplicate escalation matches** — list of similar open escalations with IDs and match basis
- **Existing defect matches** — list of related open defects in the issue tracker
- **Suggested engineering queue** — based on product area ownership map
- **Evidence quality note** — whether the evidence is sufficient for engineering to begin investigation
- **Draft label** — "CLASSIFICATION RECOMMENDATION — escalation engineer review required before tracker update"

### Step 8: Update Tracker (After Confirmation)

After the escalation engineer confirms the classification:
- Update the Product Area field
- Update the Suspected Defect Path field
- Add known issue cross-references to the tracker notes
- Update Status to "Classified — Pending Impact Assessment"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find escalation tracker, evidence packet, product area taxonomy, defect classification guide, ownership map, known issues list |
| SearchM365 (connectors) | Pull known issues and existing defects from issue tracker connector |
| ReadFileContent | Read all classification reference documents |

## Guardrails

- **Present classification as recommendation only** — never auto-assign product area, issue type, or defect path without escalation engineer review
- **Always show confidence level** — flag low-confidence classifications prominently so the engineer knows additional investigation may be needed
- **Surface known issue matches prominently** — if a matching known issue exists with a workaround, this can resolve the escalation faster than a full engineering investigation
- **Never classify severity in this step** — severity assessment is a separate skill (ps-impact-assessment) with its own approval requirements
- **Show the basis for every classification decision** — which symptoms, evidence, or patterns drove the recommendation
- **Flag when the issue spans multiple product areas** — cross-component issues may need coordinated investigation
- **Never auto-close an escalation as a duplicate** — only flag matches and let the escalation engineer decide whether to merge, link, or keep separate
- **Preserve the original suspected defect path** from intake — if the classification changes the path, show both the original and recommended values
- **Flag if evidence quality is insufficient for the recommended classification** — a low-evidence classification should be presented as tentative
- **Never fabricate known issue matches** — if no matches are found, report "No matching known issues found" rather than suggesting possible connections without evidence
