---
name: sc-classify-exception
description: |
  Classifies a supply chain exception by type, likely cause, and
  disruption pattern, with duplicate and repeat detection.
  Use when user asks to "classify this shortage [ID]",
  "what type of exception is [case ID]",
  "categorize the disruption for [item]",
  "what caused the shortage on [item]",
  "triage this supply exception",
  or "is this a repeat shortage".
  Do NOT use for creating a new exception case (use sc-shortage-intake),
  gathering demand and inventory context (use sc-context-packet),
  assessing business impact (use sc-impact-assess),
  routing to an owner (use sc-route-exception),
  or drafting shortage communications (use sc-shortage-comms).
---

## Overview

Compares exception evidence against the exception taxonomy, shortage playbooks, and prior case patterns to determine the exception type, likely root cause, and whether this is a repeat disruption pattern. Surfaces classification recommendations with confidence levels and supporting evidence for planner review via Adaptive Card.

This skill operates in "AI assist" mode — it reads and analyzes exception data but only presents classification recommendations. The supply planner reviews and confirms before any tracker updates are made.

## When to Use

- An exception case has context assembled and needs classification before impact assessment
- A planner needs to determine whether a shortage matches a known disruption pattern
- A new shortage needs to be checked for repeat patterns against the tracker and prior cases
- A reclassification is needed after new evidence changes the suspected cause

## When NOT to Use

- Creating a new exception case record — use sc-shortage-intake
- Gathering demand, inventory, and shipment context — use sc-context-packet
- Assessing business impact and recommending mitigation path — use sc-impact-assess
- Routing the exception to an owner — use sc-route-exception
- Drafting shortage summaries or follow-up communications — use sc-shortage-comms
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read exception data and classification references", activeForm="Analyzing exception")
TaskCreate(subject="Present classification recommendation", activeForm="Classifying exception")
```

### Step 1: Read Classification Inputs

**Read the exception case and context:**
- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — Case ID, Item/SKU, Site, Shortage Type (initial), context summary
- Review the context packet data (inventory position, demand exposure, shipment status, supplier context)

**Read classification references:**
- `SearchM365(sources=["files"], query="exception taxonomy")` then `ReadFileContent` — standard exception type definitions with classification criteria
- `SearchM365(sources=["files"], query="shortage playbook")` then `ReadFileContent` — playbook-defined disruption patterns and cause indicators
- `SearchM365(sources=["files"], query="supply chain routing rules")` then `ReadFileContent` — routing implications of each exception type

### Step 2: Classify Exception Type

Compare the exception evidence against the taxonomy:

| Exception Type | Indicators |
|---------------|-----------|
| **Stockout** | On-hand inventory at zero or below safety stock; demand exceeds available supply; no in-transit replenishment within the demand window |
| **Supplier delay** | Supplier has confirmed or is expected to miss the committed delivery date; in-transit shipment delayed beyond original ETA |
| **Quality hold** | Inventory exists but is quarantined due to quality issue; receiving inspection failure; customer return defect pattern |
| **Demand spike** | Demand has exceeded forecast by a significant margin; unexpected large order; promotional demand not reflected in plan |
| **Logistics disruption** | Shipment in transit is delayed, damaged, or rerouted due to carrier issue, weather, port congestion, or customs hold |
| **Production disruption** | Manufacturing or assembly line issue has reduced or stopped output; raw material shortage affecting production |

### Step 3: Determine Likely Root Cause

Based on the classification and evidence, identify the most probable root cause:

| Cause Category | Evidence Patterns |
|---------------|-------------------|
| **Supplier reliability** | Supplier delivery performance below threshold, repeated delays, capacity constraints |
| **Demand variability** | Forecast miss, unexpected order, seasonal pattern not captured |
| **Planning gap** | Safety stock set too low, reorder point not triggered, lead time assumption incorrect |
| **Logistics failure** | Carrier delay, port congestion, customs issue, weather event |
| **Quality issue** | Batch failure, specification change, receiving inspection rejection |
| **External disruption** | Natural disaster, geopolitical event, regulatory change, supplier bankruptcy |

### Step 4: Check for Repeat Patterns

Search for prior cases with similar characteristics:

- `SearchM365(sources=["files"], query="shortage tracker [item]")` then `ReadFileContent` — prior cases for the same item
- `SearchM365(sources=["files"], query="shortage tracker [site]")` then `ReadFileContent` — prior cases at the same site
- `SearchM365(sources=["files"], query="shortage tracker [supplier]")` then `ReadFileContent` — prior cases involving the same supplier

**For each pattern match found:**
- Prior Case ID and date
- Exception type and root cause
- Resolution method and time to resolution
- Whether the current case appears to be the same disruption continuing or a new occurrence

**Flag repeat patterns prominently:**
- Same item + same supplier with 3+ cases in 90 days → "Recurring supplier reliability issue"
- Same site with 3+ cases in 90 days → "Site-level systemic issue"
- Same item across multiple sites → "Item-level supply chain vulnerability"

### Step 5: Assess Classification Confidence

| Confidence Level | Criteria |
|-----------------|---------|
| **High** | Clear evidence matches a single exception type; cause indicators are consistent; prior cases confirm the pattern |
| **Medium** | Evidence supports the classification but some ambiguity exists; could be multiple exception types; cause is probable but not confirmed |
| **Low** | Evidence is thin or contradictory; multiple exception types are plausible; cause cannot be determined from available data |

### Step 6: Identify Regulatory or Special Handling Flags

Check whether the item or situation requires special handling:

| Flag Type | Criteria |
|-----------|---------|
| **Regulated item** | Item is subject to export controls, FDA requirements, hazmat regulations, or cold chain requirements |
| **Critical item** | Item is on the critical items list maintained by supply chain operations |
| **Single-source** | Item has only one approved supplier — supply risk is elevated |
| **Customer-committed** | Open customer orders with firm delivery commitments affected |

### Step 7: Present Classification Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Item/SKU, Site, SLA Deadline
- **Recommended classification:**
  - Exception type (stockout, supplier delay, quality hold, demand spike, logistics disruption, production disruption)
  - Likely root cause with evidence basis
  - Confidence level (high, medium, low)
- **Repeat pattern detection** — prior cases matching this pattern, frequency, and trend
- **Special handling flags** — regulated item, critical item, single-source, customer-committed
- **Applicable playbook** — SOP or playbook reference for this exception type
- **Suggested next step** — which downstream skill (sc-impact-assess or additional context gathering)
- **Classification label** — "CLASSIFICATION RECOMMENDATION — planner review required before tracker update"

### Step 8: Update Tracker (After Confirmation)

After the planner confirms the classification:
- Update Shortage Type field with the confirmed exception type
- Add Root Cause field with the confirmed cause
- Add Repeat Pattern Flag if applicable
- Update Status to "Classified — Pending Impact Assessment"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find shortage tracker, exception taxonomy, shortage playbooks, routing rules, prior case data |
| SearchM365 (connectors) | Pull additional item and supplier history from ERP via Graph Connector |
| ReadFileContent | Read all classification reference documents and tracker data |
| render_ui (Adaptive Card) | Present the classification recommendation for planner review |

## Guardrails

- **Present classification as a recommendation only** — never auto-assign exception type or root cause without planner review
- **Always show confidence level** — flag low-confidence classifications prominently so the planner knows additional investigation may be needed
- **Surface repeat patterns prominently** — recurring disruptions indicate systemic issues that may need escalation beyond routine triage
- **Never classify priority or impact in this step** — impact assessment is a separate skill (sc-impact-assess) with its own review requirements
- **Show the evidence basis for every classification decision** — which inventory data, supplier signals, or prior cases drove the recommendation
- **Flag regulatory and special handling requirements** — items subject to export controls, hazmat, or cold chain must be identified before routing
- **Never auto-close a case as a duplicate** — flag matches and let the planner decide whether to merge, link, or keep separate
- **Preserve the original shortage type from intake** — if the classification changes the type, show both the original and recommended values
- **Log classification and any override** — overrides are valuable data for improving classification accuracy
- **Include the Case ID and SLA deadline** in every output for traceability and urgency awareness
