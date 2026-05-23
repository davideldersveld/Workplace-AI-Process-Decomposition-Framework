---
name: sc-shortage-intake
description: |
  Normalizes an inventory shortage event, supplier delay, or disruption
  alert into a structured exception case record in the shared tracker.
  Use when user asks to "new shortage alert for [item]",
  "supply disruption for [item] at [site]",
  "log shortage case for [SKU]",
  "inventory exception for [site]",
  "supplier delay reported for [item]",
  or "create supply exception for [case]".
  Do NOT use for gathering demand and inventory context (use sc-context-packet),
  classifying the exception type (use sc-classify-exception),
  assessing business impact (use sc-impact-assess),
  routing to an owner (use sc-route-exception),
  or drafting shortage communications (use sc-shortage-comms).
---

## Overview

Converts an incoming shortage event, supplier delay notification, or disruption alert into a structured exception case record in the shared Excel shortage tracker. Validates required fields, checks for duplicate cases on the same item and site, generates a unique Case ID, calculates the SLA deadline based on initial severity indicators, and assigns the supply planner based on item category or site ownership.

This skill operates in "deterministic automation" mode — it performs structured field extraction, validation, duplicate checking, and SLA calculation without exercising AI judgment on classification, priority, or mitigation path.

## When to Use

- An inventory shortage threshold has been breached and needs to be logged as an exception case
- A supplier delay notification has been received and needs to be formalized
- A fulfillment exception has been created in the ERP or planning system
- A quality hold or logistics disruption needs to be captured as a tracked case
- A planner or coordinator needs to report a supply disruption manually

## When NOT to Use

- Gathering demand, inventory, and shipment context for an existing case — use sc-context-packet
- Classifying the exception type and likely cause — use sc-classify-exception
- Assessing business impact and recommending mitigation path — use sc-impact-assess
- Routing the exception to an owner — use sc-route-exception
- Drafting shortage summaries or follow-up communications — use sc-shortage-comms
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read intake data and check for duplicates", activeForm="Processing shortage alert")
TaskCreate(subject="Create exception case record", activeForm="Creating exception case")
```

### Step 1: Identify the Intake Signal

Determine the source of the shortage event:

| Source | Detection Method | Key Fields |
|--------|-----------------|------------|
| **ERP alert** | `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` | Item/SKU, site, shortage quantity, threshold breached, alert timestamp |
| **Supplier delay email** | `SearchM365(sources=["email"], query="supplier delay [item]")` | Supplier name, item, expected delivery date, revised delivery date, reason |
| **Planning system exception** | `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` | Exception ID, item, site, exception type, planning horizon impact |
| **Teams escalation** | `SearchM365(sources=["teams"], query="shortage [item] [site]")` | Reported by, item, site, urgency description |
| **Direct request** | User provides details in the conversation | Item/SKU, site, shortage type, context |
| **Supplier notice document** | `SearchM365(sources=["files"], query="supplier notice [supplier name]")` then `ReadFileContent` | Supplier, affected items, disruption scope, estimated duration |

### Step 2: Extract and Validate Required Fields

Extract these fields from the intake signal:

| Field | Source | Validation |
|-------|--------|-----------|
| **Item / SKU** | ERP, email, or user input | Must follow the organization's SKU format; resolve via ERP connector or SharePoint item master |
| **Site / Warehouse** | ERP, email, or user input | Must match a known site in the site master |
| **Shortage Type** | Inferred from source signal | One of: Stockout, Supplier Delay, Quality Hold, Demand Spike, Logistics Disruption |
| **Source System Reference** | ERP alert ID, planning exception ID, or email message ID | Captured for traceability; not required for manual intake |
| **Reported By** | Email sender, Teams message author, or user | Resolve via `SearchPeople` and `GetUserDetails` |
| **Shortage Quantity** | ERP data or user input | Numeric value; flag if unavailable |
| **Initial Customer Impact Flag** | Inferred from open order exposure or user input | Yes/No/Unknown — triggers visibility requirements if Yes |

### Step 3: Check for Duplicate Cases

Read the shortage tracker to check for existing cases on the same item and site:

- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent`
- Check for matching Item/SKU + Site combination with Status not in (Resolved, Closed) and Created Date within the last 7 days
- If a duplicate exists:
  - Surface the existing Case ID, status, priority, and assigned planner
  - Ask the user whether to link to the existing case, create a new case, or cancel

### Step 4: Generate Case ID

Format: `SC-YYYYMMDD-SEQ` where:
- `YYYYMMDD` is the current date
- `SEQ` is a three-digit sequence number based on the count of cases created on the same date in the tracker

### Step 5: Calculate SLA Deadline

Based on initial severity indicators (before formal impact assessment):

| Initial Indicator | SLA Target | Deadline Calculation |
|-------------------|-----------|---------------------|
| **Customer impact flagged** | 1 hour | Current time + 1 hour |
| **Critical item or site** | 1 hour | Current time + 1 hour |
| **Standard shortage** | 4 hours | Current time + 4 business hours |
| **Low-priority or informational** | 1 business day | Next business day end-of-day |

The SLA deadline is preliminary — it may be adjusted by sc-impact-assess after formal impact assessment.

### Step 6: Assign Initial Planner

Determine the initial planner assignment based on the routing rules:

- `SearchM365(sources=["files"], query="supply chain routing rules")` then `ReadFileContent` — item category to planner mapping, site ownership model
- `SearchPeople` — resolve the assigned planner by item category or site
- `GetUserDetails` — verify the planner's profile and availability

### Step 7: Present Case Record for Confirmation

Present the proposed case record via Adaptive Card (invoke `render-ui` skill first):

- **Case ID** — generated ID
- **Item / SKU** and **Site**
- **Shortage Type** — initial classification
- **Shortage Quantity** — if available
- **Source System Reference** — for traceability
- **Reported By** — who raised the alert
- **Customer Impact Flag** — Yes/No/Unknown
- **SLA Deadline** — calculated target
- **Assigned Planner** — initial assignment
- **Duplicate check result** — no duplicates found, or link to existing case

### Step 8: Write to Shortage Tracker (After Confirmation)

After the user confirms, write the case record to the Excel shortage tracker:

- Case ID, Item/SKU, Site, Shortage Type, Source System Ref
- Shortage Quantity, Customer Impact Flag, Priority ("Pending Assessment")
- Status ("Intake Complete"), Created Date, SLA Deadline
- Assigned Planner, Reported By

### Step 9: Post Intake Notification

- `PostMessage` — Teams notification to the supply operations channel with:
  - Case ID, Item/SKU, Site, Shortage Type
  - Customer Impact Flag (highlighted if Yes)
  - Assigned Planner
  - SLA Deadline

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find supplier delay notifications, internal escalation emails |
| SearchM365 (teams) | Find shortage discussions in Teams channels or chats |
| SearchM365 (files) | Find the shortage tracker, routing rules, supplier notices |
| SearchM365 (connectors) | Pull ERP shortage events, inventory alerts, planning exceptions via Graph Connector |
| ReadFileContent | Read the shortage tracker, routing rules, supplier documents |
| GetDriveChildren | Check for supporting documents in the case folder |
| SearchPeople | Resolve the assigned planner and reporting user |
| GetUserDetails | Verify planner profiles and availability |
| PostMessage | Post intake notification to the supply operations channel |
| render_ui (Adaptive Card) | Present the case record for confirmation |

## Guardrails

- **Never create duplicate cases** for the same item, site, and date combination — always check the tracker first
- **Validate item and site identifiers** against the organization's master data — reject unrecognized values and surface the issue
- **Post a Teams notification** to the supply operations channel upon case creation — visibility is critical for supply chain exceptions
- **Log creation with actor and timestamp** for audit trail — every case must be traceable to its source
- **Never modify inventory records or allocations** — intake creates a tracking record only; no system-of-record changes
- **Flag customer-impacting shortages immediately** — if the customer impact flag is Yes, the SLA is 1 hour regardless of other indicators
- **Include the Case ID in every output** for traceability across the exception lifecycle
- **Preserve the source system reference** — the link between the tracker case and the originating ERP or planning event is essential for audit
- **Never assign a priority during intake** — priority is set to "Pending Assessment" until sc-impact-assess completes a formal evaluation
