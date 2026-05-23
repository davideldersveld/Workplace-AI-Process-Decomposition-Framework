---
name: sc-context-packet
description: |
  Assembles demand, inventory, shipment, supplier, and operational
  context for an open supply chain exception case.
  Use when user asks to "build context for shortage [ID]",
  "what's the situation on [item]",
  "assemble supply context for [case]",
  "gather shortage details for [ID]",
  "pull inventory and demand data for [item]",
  or "supply context for [site] [item]".
  Do NOT use for creating a new exception case (use sc-shortage-intake),
  classifying the exception type (use sc-classify-exception),
  assessing business impact (use sc-impact-assess),
  routing to an owner (use sc-route-exception),
  or drafting shortage communications (use sc-shortage-comms).
---

## Overview

Assembles a comprehensive context packet for a supply chain exception case by gathering current inventory position, demand exposure, in-transit shipment status, supplier history, prior shortage patterns, and applicable SOPs from across ERP, planning, transportation, and SharePoint systems. Presents findings via Adaptive Card for fast planner review, optimized for the 1-hour SLA on critical exceptions.

This skill operates in "AI act within policy" mode — it retrieves context from approved data sources without exercising judgment on classification, priority, or mitigation path.

## When to Use

- An exception case has been created and the planner needs supply context before classification
- A planner needs to understand the current inventory, demand, and shipment situation for an item
- A disruption has been reported and the team needs a consolidated view of the affected supply position
- An escalation review requires a complete picture of the supply situation

## When NOT to Use

- Creating a new exception case record — use sc-shortage-intake
- Classifying the exception type and likely cause — use sc-classify-exception
- Assessing business impact and recommending mitigation path — use sc-impact-assess
- Routing the exception to an owner — use sc-route-exception
- Drafting shortage summaries or follow-up communications — use sc-shortage-comms
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather inventory, demand, and shipment data", activeForm="Assembling supply context")
TaskCreate(subject="Present context packet for review", activeForm="Reviewing supply context")
```

### Step 1: Read Exception Case Data

- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — retrieve the case record including Case ID, Item/SKU, Site, Shortage Type, Customer Impact Flag, SLA Deadline

### Step 2: Gather Inventory Position

**Current inventory levels:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — on-hand inventory at the affected site and all network sites for the item
- If connector unavailable: `SearchM365(sources=["files"], query="inventory snapshot [item]")` then `ReadFileContent` — SharePoint bridge data

**Capture:**
- On-hand quantity at affected site
- On-hand quantity at other network sites (potential reallocation sources)
- Safety stock level and current position relative to safety stock
- Allocated quantity vs. available-to-promise
- Last receipt date and quantity

### Step 3: Gather Demand Exposure

**Open orders and forecasted demand:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — open customer orders, planned demand, forecast for the next 30 days
- If connector unavailable: `SearchM365(sources=["files"], query="demand exposure [item]")` then `ReadFileContent`

**Capture:**
- Open customer orders (count, total quantity, earliest required date)
- Planned production or distribution demand
- Forecasted demand for next 7, 14, and 30 days
- Customer names and tiers for affected orders (if available)

### Step 4: Gather Shipment and Supply Pipeline

**In-transit and on-order:**
- `SearchM365(sources=["connectors"], connector_ids=["tms-connector"])` — in-transit shipments with ETAs, carrier, and status
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — open purchase orders and expected receipt dates

**Capture:**
- In-transit quantities with ETAs and current status (on-time, delayed, at-risk)
- Open purchase order quantities and expected receipt dates
- Supplier lead time for the item
- Any known supply pipeline disruptions

### Step 5: Gather Supplier Context

- `SearchM365(sources=["email"], query="[supplier name] [item] delay")` — recent supplier correspondence about delays or disruptions
- `SearchM365(sources=["files"], query="supplier scorecard [supplier name]")` then `ReadFileContent` — supplier performance history and reliability data
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — supplier order history for the item

**Capture:**
- Supplier name and relationship status
- Recent delivery performance (on-time rate)
- Any active supplier disruption notices
- Alternative suppliers for the item (if documented)

### Step 6: Gather Prior Shortage History

- `SearchM365(sources=["files"], query="shortage tracker [item]")` then `ReadFileContent` — prior shortage cases for the same item or site
- Look for patterns: recurring shortages, seasonal patterns, supplier-specific issues

**Capture:**
- Prior shortage cases (last 90 days) for this item or site
- Resolution methods used previously
- Time to resolution for similar cases
- Whether this is a repeat disruption pattern

### Step 7: Identify Applicable SOPs and Playbooks

- `SearchM365(sources=["files"], query="shortage playbook [shortage type]")` then `ReadFileContent` — standard operating procedures for this type of exception
- `SearchM365(sources=["files"], query="mitigation template [item category]")` then `ReadFileContent` — mitigation approach templates

**Capture:**
- Applicable SOP or playbook with reference link
- Recommended mitigation steps from the playbook
- Escalation criteria from the playbook

### Step 8: Assess Data Completeness

Evaluate the completeness of gathered context:

| Data Area | Status | Staleness Check |
|-----------|--------|----------------|
| **Inventory position** | Available / Unavailable / Partial | Flag if older than 24 hours |
| **Demand exposure** | Available / Unavailable / Partial | Flag if older than 24 hours |
| **Shipment status** | Available / Unavailable / Partial | Flag if older than 4 hours |
| **Supplier context** | Available / Unavailable / Partial | Flag if no recent correspondence |
| **Prior history** | Available / None found | Informational — absence is not a gap |
| **Applicable SOP** | Found / Not found | Flag if no playbook matches the exception type |

### Step 9: Present Context Packet

Present the assembled context via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Item/SKU, Site, Shortage Type, SLA Deadline, Customer Impact Flag
- **Inventory position** — on-hand, safety stock status, allocated vs. available, network availability
- **Demand exposure** — open orders count, total quantity, earliest required date, affected customers
- **Supply pipeline** — in-transit quantities with ETAs, open POs, expected receipts
- **Supplier context** — supplier name, recent performance, active disruption notices
- **Prior shortage history** — recent cases, resolution methods, pattern indicators
- **Applicable SOP** — playbook reference and recommended approach
- **Data completeness** — status of each data area with staleness flags
- **Context label** — "SUPPLY CONTEXT — data as of [timestamp]; review before classification"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find shortage tracker, inventory snapshots, supplier scorecards, SOPs, playbooks, mitigation templates |
| SearchM365 (connectors) | Pull real-time inventory, demand, shipment, and supplier data via Graph Connectors |
| SearchM365 (email) | Find supplier correspondence about delays or disruptions |
| ReadFileContent | Read all reference documents, tracker, and bridge data |
| GetDriveChildren | Check for supporting documents in the case folder |
| render_ui (Adaptive Card) | Present the context packet for planner review |

## Guardrails

- **Cite data source and timestamp for every inventory or demand figure** — planners must know how current the data is
- **Flag if data is stale** — inventory data older than 24 hours or shipment status older than 4 hours must be explicitly flagged
- **Never modify inventory records, demand signals, or shipment data** — this skill is strictly read-only
- **Mark context as provisional** if any key data source (inventory, demand, or shipment) is unavailable — downstream decisions should not rely on incomplete context
- **Never include supplier-confidential information** (pricing, capacity commitments, contract terms) in the context packet — operational data only
- **Do not interpret or classify the exception** — context assembly is separate from classification; present facts without judgment
- **Include the Case ID and SLA deadline** in every output for traceability and urgency awareness
- **Optimize for speed** — the 1-hour SLA for critical exceptions means the context packet must be assembled and presented quickly; Adaptive Card is the primary output format
- **Never fabricate supply data** — if a data source is unavailable, report it as unavailable rather than estimating values
