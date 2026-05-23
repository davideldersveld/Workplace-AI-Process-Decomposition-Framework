---
name: fs-dispatch-packet
description: |
  Assembles a comprehensive dispatch context packet including asset history,
  site details, parts requirements, technician skills needed, and prior visit history.
  Use when user asks to "build dispatch packet for work order [ID]",
  "pull context for this job", "assemble dispatch readiness context",
  "what do we need for this work order", "dispatch context for [customer]",
  "gather context for work order [ID]", or "prepare dispatch packet".
  Do NOT use for creating a new work order (use fs-workorder-intake),
  classifying blockers (use fs-blocker-classifier),
  assessing urgency (use fs-urgency-assessment),
  routing the work order (use fs-dispatch-routing),
  or drafting communications (use fs-dispatch-comms).
---

## Overview

Assembles all context needed for dispatch readiness review into a single Word document — the dispatch context packet. Gathers asset history, site details, parts requirements, technician skill needs, prior visit history, safety notes, and key contacts from M365 and federated data sources.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources and assembles it without interpreting or overriding policies. It does not modify any source records.

## When to Use

- A work order has been created and needs context assembled before classification and routing
- The user wants to understand what is needed for a work order before dispatch
- A dispatch coordinator needs a complete briefing on a job

## When NOT to Use

- Creating a new work order — use fs-workorder-intake
- Classifying work type and identifying blockers — use fs-blocker-classifier
- Assessing urgency and dispatch path — use fs-urgency-assessment
- Routing the work order to a technician — use fs-dispatch-routing
- Drafting customer or technician communications — use fs-dispatch-comms

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read work-order record and locate context sources", activeForm="Reading work-order data")
TaskCreate(subject="Assemble dispatch context packet", activeForm="Assembling dispatch packet")
```

### Step 1: Read Work-Order Record

Locate and read the work-order record:

- `SearchM365(sources=["files"], query="work order tracker")` to find the tracker
- `ReadFileContent` to read the specific work-order row
- Extract: Work Order ID, Customer Name, Account ID, Site Address, Work Type, Issue Summary, Requested Window, Safety Flags

### Step 2: Gather Context from All Sources

Retrieve context from each source in parallel where possible:

**Asset profile:**
- `SearchM365(sources=["files"], query="[equipment type] asset history [site address]")` for asset documents in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` for asset records from the field service platform
- Collect: equipment type, model, serial number, warranty status, last service date, known issues

**Site details:**
- `SearchM365(sources=["files"], query="[site address] site access")` for site access guides and safety notes
- `SearchM365(sources=["files"], query="[customer name] site")` for site-specific documentation
- Collect: access requirements, operating hours, safety hazards, site contact, parking and entry instructions

**Prior visit history:**
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` for last 5 visits to this site or on this equipment
- `SearchM365(sources=["files"], query="service report [site address]")` for prior service reports in SharePoint
- Collect: visit dates, work performed, outcomes, technician notes, unresolved issues

**Parts assessment:**
- `SearchM365(sources=["connectors"], connector_ids=["inventory-connector"])` for parts availability
- `SearchM365(sources=["files"], query="parts list [work type] [equipment type]")` for standard parts lists
- Collect: required parts, availability status, warehouse location, estimated delivery if backordered

**Technician requirements:**
- `SearchM365(sources=["files"], query="skill matrix [work type]")` for required skills and certifications
- `SearchM365(sources=["files"], query="technician certification [work type]")` for certification requirements
- Collect: required skills, certifications, special tooling, minimum experience level

**Prior correspondence:**
- `SearchM365(sources=["email"], query="[customer name] [site address]")` for recent customer emails
- `ListChatMessages` or `SearchM365(sources=["teams"])` for internal discussions about this site
- Collect: recent issues raised, scheduling concerns, customer expectations

**Key contacts:**
- `SearchPeople(query="<customer name>")` and `GetUserDetails` for customer and account owner profiles
- `SearchPeople(query="dispatch coordinator")` for dispatch team contacts
- Collect: customer contact, site contact, account owner, dispatch coordinator, technician lead

### Step 3: Produce Dispatch Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Work-Order Summary** — ID, customer, site, work type, issue description, requested window, safety flags
2. **Asset Profile** — equipment type, model, serial number, warranty status, last service date, known issues
3. **Site Details** — address, access requirements, operating hours, safety hazards, site contact, entry instructions
4. **Prior Visit History** — last 5 visits with dates, work performed, outcomes, and technician notes
5. **Parts Assessment** — required parts, availability status, warehouse location, delivery estimates
6. **Technician Requirements** — required skills, certifications, special tooling
7. **Scheduling Context** — requested window, site operating hours, access restrictions, conflicting appointments
8. **Key Contacts** — customer, site contact, account owner, dispatch coordinator, technician lead
9. **Missing Context Flags** — any critical data that could not be retrieved, with source and reason

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find work-order tracker, asset documents, site guides, skill matrix, parts lists, service reports |
| SearchM365 (email) | Find prior customer correspondence |
| SearchM365 (teams) | Find internal discussions about the site or equipment |
| SearchM365 (connectors) | Pull asset records, visit history, parts availability from field service platform and inventory system |
| ReadFileContent | Read tracker, asset documents, site guides, service reports |
| GetDriveChildren | Browse site-specific evidence folders (service reports, photos, manuals) |
| SearchPeople / GetUserDetails | Resolve customer, account owner, dispatch coordinator, technician lead |
| ListChatMessages | Find prior Teams discussions about this customer or site |

## Guardrails

- **Cite source and retrieval date** for every data point from the field service platform, inventory system, or asset history
- **Flag missing critical context** — if asset history, site access notes, or parts availability cannot be retrieved, flag prominently in the packet
- **Flag safety advisories** — if the asset is under recall, has safety advisories, or the site has had safety incidents, flag with elevated visibility
- **Flag prior visit issues** — if previous visits had access problems, safety incidents, or repeat failures, include in the packet
- **Operate read-only** — never update field service platform, inventory, or asset management records
- **Protect sensitive information** — do not include customer financial details or internal pricing in the dispatch packet
- **Record assembly metadata** — note which sources were queried, what was found, and what was missing
