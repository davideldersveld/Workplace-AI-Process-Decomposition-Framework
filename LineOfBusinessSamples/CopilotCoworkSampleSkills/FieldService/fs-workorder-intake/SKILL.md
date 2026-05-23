---
name: fs-workorder-intake
description: |
  Normalizes inbound field service work-order events into structured records in the
  work-order tracker, calculates SLA deadline, and flags safety-sensitive work types.
  Use when user asks to "new work order", "log work order for [customer]",
  "service request from [site]", "intake work order [ID]",
  "new field service case", "create work order for [site]",
  "field service intake for [customer]", or "log service request".
  Do NOT use for assembling dispatch context (use fs-dispatch-packet),
  classifying blockers (use fs-blocker-classifier),
  assessing urgency (use fs-urgency-assessment),
  routing the work order (use fs-dispatch-routing),
  or drafting communications (use fs-dispatch-comms).
---

## Overview

Normalizes inbound field service events — emails, service requests, chat messages, and phone intake — into structured work-order records in the shared Excel tracker. Calculates the 30-minute triage SLA deadline, checks for duplicate work orders, and flags safety-sensitive work types that require elevated handling.

This skill operates in "deterministic automation" mode — structured field extraction, duplicate checking, SLA calculation, and safety flagging follow fixed rules with no AI judgment required.

## When to Use

- A new work order arrives via email, chat, form submission, or phone intake
- A service event needs to be logged and tracked
- The user wants to create a structured work-order record from an inbound request

## When NOT to Use

- Assembling asset, location, and parts context — use fs-dispatch-packet
- Classifying work type and blockers — use fs-blocker-classifier
- Assessing urgency and dispatch path — use fs-urgency-assessment
- Routing the work order to a technician — use fs-dispatch-routing
- Drafting customer or technician communications — use fs-dispatch-comms

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read inbound event and extract work-order fields", activeForm="Reading inbound event")
TaskCreate(subject="Check for duplicates and write work-order record", activeForm="Creating work-order record")
```

### Step 1: Read Inbound Event

Identify the source of the work-order event:

- **Email**: `SearchM365(sources=["email"])` with customer name or site address, then `GetMessage(message_id=...)` to read the full email
- **Chat**: `ListChatMessages(person=...)` or `SearchM365(sources=["teams"])` to find the service request message
- **File attachment**: `SearchM365(sources=["files"])` to locate service request forms or work-order documents in SharePoint
- **Field service platform**: `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` to pull the work-order record if Graph Connector is configured

Extract the following fields from the inbound event:

| Field | Description | Required |
|-------|-------------|----------|
| Customer Name | Company or individual name | Yes |
| Account ID | CRM or field service platform account identifier | If available |
| Site Address | Physical location for the service visit | Yes |
| Work Type | Installation, repair, maintenance, or inspection | Yes |
| Issue Summary | Description of the problem or service requested | Yes |
| Requested Window | Customer's preferred appointment date and time | Yes |
| Site Contact | Name and phone number of on-site contact | If available |
| Attachments | Site photos, equipment manuals, forms | If available |

**Resolve identities:**
- `SearchPeople(query="<customer name>")` to find internal account owner
- `GetUserDetails(user_id="<account owner>")` to pull dispatch coordinator profile
- `GetMyDetails` to record who created the work-order record

### Step 2: Check for Duplicates and Create Record

**Duplicate check:**
Search the work-order tracker for existing records matching the same customer, site address, and work type within the last 7 days:
- `SearchM365(sources=["files"], query="work order tracker")` to locate the tracker
- `ReadFileContent` to read existing records
- If a potential duplicate is found, present it to the user and ask whether to proceed or link to the existing work order

**Generate Work Order ID:**
Format: WO-YYYY-NNNNN (e.g., WO-2026-00142), incrementing from the last ID in the tracker.

**Calculate SLA deadline:**
- Standard work orders: 30 minutes from creation for triage completion
- Flag the SLA deadline in the tracker record

**Safety flag check:**
Flag the work order if the work type or issue description involves:
- Electrical systems (high voltage, panel work, wiring)
- Gas systems (gas lines, gas appliances, leak detection)
- Confined spaces (tanks, crawl spaces, utility vaults)
- Hazardous materials (refrigerants, asbestos, chemicals)
- Rooftop or elevated work (HVAC units, antenna, solar panels)

**Write the work-order record** to the tracker with all extracted fields, SLA deadline, safety flags, and status set to "New".

**Present confirmation** via Adaptive Card (invoke `render-ui` skill first):
- Work Order ID, Customer, Site Address, Work Type
- Issue Summary
- Requested Appointment Window
- SLA Deadline (30 minutes from now)
- Safety Flags (if any)
- Status: New

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find inbound work-order email or service request |
| SearchM365 (files) | Find work-order tracker in SharePoint |
| SearchM365 (connectors) | Pull work-order record from field service platform |
| GetMessage | Read full inbound email content |
| ReadFileContent | Read work-order tracker, service request forms |
| SearchPeople / GetUserDetails | Resolve customer contact and internal account owner |
| GetMyDetails | Current user for tracking who created the record |

## Guardrails

- **Never create duplicate work orders** for the same customer, site, and requested date — check the tracker first
- **Validate required fields** — site address and requested appointment window must be populated before writing
- **Confirm details with user** before writing to the work-order tracker
- **Calculate SLA deadline automatically** — 30 minutes from creation for standard work orders
- **Flag safety-sensitive work types** — electrical, gas, confined space, hazardous materials, and elevated work require elevated handling throughout the workflow
- **Preserve original request language** — store the customer's own description as the issue summary
- **Record creation metadata** — timestamp, created-by user, and source channel in the tracker
