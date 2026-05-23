---
name: fs-blocker-classifier
description: |
  Classifies the work type and identifies dispatch blockers across parts, skills,
  access, and schedule dimensions with a readiness scorecard.
  Use when user asks to "classify this work order", "check for blockers on [ID]",
  "what could block this dispatch", "triage readiness for work order",
  "blocker check for [ID]", "dispatch readiness check",
  "what's blocking work order [ID]", or "readiness scorecard for [case]".
  Do NOT use for creating a new work order (use fs-workorder-intake),
  assembling context (use fs-dispatch-packet),
  assessing urgency (use fs-urgency-assessment),
  routing the work order (use fs-dispatch-routing),
  or drafting communications (use fs-dispatch-comms).
---

## Overview

Compares the work-order requirements against the work-type taxonomy and readiness checklist, cross-references parts availability, technician skill matrix, site access requirements, and scheduling constraints, and presents a classification with blocker report and readiness scorecard. The classification includes work type confirmation, identified blockers with severity, and an overall dispatch readiness assessment.

This skill operates in "AI assist" mode — it presents the classification and blocker report as a recommendation for dispatcher review via Adaptive Card. It does not auto-clear blockers or mark work orders as dispatch-ready.

## When to Use

- A work order has context assembled and needs classification before routing
- The user wants to check what is blocking a work order from dispatch
- A dispatcher needs a readiness scorecard for a work order

## When NOT to Use

- Creating a new work order — use fs-workorder-intake
- Assembling asset, location, and parts context — use fs-dispatch-packet
- Assessing urgency and dispatch path — use fs-urgency-assessment
- Routing the work order to a technician — use fs-dispatch-routing
- Drafting customer or technician communications — use fs-dispatch-comms

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read work-order data and readiness requirements", activeForm="Reading classification inputs")
TaskCreate(subject="Classify work type and identify blockers", activeForm="Classifying and checking blockers")
TaskCreate(subject="Present readiness scorecard", activeForm="Preparing readiness report")
```

### Step 1: Read Classification Inputs

Locate and read required inputs:

- **Work-order data** — from the tracker: `SearchM365(sources=["files"], query="work order tracker")`
- **Dispatch context packet** — from fs-dispatch-packet output: `SearchM365(sources=["files"], query="dispatch packet [Work Order ID]")`
- **Work-type taxonomy** — `SearchM365(sources=["files"], query="work type taxonomy")` or `SearchM365(sources=["files"], query="service classification")`
- **Readiness checklist** — `SearchM365(sources=["files"], query="dispatch readiness checklist")`
- **Skill matrix** — `SearchM365(sources=["files"], query="technician skill matrix")`

Read each document using `ReadFileContent`.

### Step 2: Classify Work Type

Compare the work-order description and context against the work-type taxonomy:

| Work Type | Typical Indicators |
|-----------|-------------------|
| Installation | New equipment, setup, commissioning, first-time site visit for new asset |
| Repair | Broken equipment, malfunction, error codes, performance degradation |
| Maintenance | Scheduled service, preventive maintenance, filter replacement, calibration |
| Inspection | Safety inspection, compliance check, warranty inspection, pre-purchase |

Confirm or correct the work type from intake. If the work type does not match the taxonomy, present the recommended classification with rationale.

### Step 3: Identify Blockers

Check each readiness dimension against the work-order requirements:

**Parts blockers (from dispatch packet parts assessment):**
- Required parts not in stock
- Parts backordered with delivery date after the requested window
- Special-order components needed
- Severity: Red if parts unavailable and no workaround; Yellow if delayed but obtainable before appointment

**Skill blockers (from skill matrix and dispatch packet):**
- No available technician with required certification
- Required certification expired for available technicians
- Specialized tooling not available in the service region
- Severity: Red if certification is mandatory (safety, regulatory); Yellow if preferred but workaround exists

**Access blockers (from dispatch packet site details):**
- Site access not confirmed with customer
- Safety clearance or background check pending
- Building management approval needed
- Restricted hours that conflict with the requested window
- Severity: Red if access cannot be arranged; Yellow if pending confirmation

**Schedule blockers (from dispatch packet scheduling context):**
- Requested window conflicts with all qualified technicians' availability
- Travel time exceeds feasible window
- Multiple jobs competing for the same technician and time slot
- Severity: Red if no qualified technician available in the window; Yellow if tight but feasible

### Step 4: Calculate Readiness Score

| Score | Criteria |
|-------|----------|
| **Ready** | All dimensions green — no blockers identified |
| **Needs Review** | One or more yellow blockers — dispatch feasible but risk exists |
| **Blocked** | One or more red blockers — dispatch not recommended until resolved |

### Step 5: Present Readiness Scorecard

Present via Adaptive Card (invoke `render-ui` skill first):

- **Confirmed work type** from taxonomy
- **Readiness scorecard** — green/yellow/red status for each dimension (Parts, Skills, Access, Schedule)
- **Identified blockers** with severity and description for each
- **Suggested resolution** for each blocker
- **Overall readiness score** (Ready, Needs Review, Blocked)
- **Safety flags** — if the work order involves safety-sensitive equipment, highlight prominently
- **Similar prior work orders** — up to 3 recent work orders of the same type at the same site or on the same equipment, with outcomes

After user confirms the classification, update the work-order tracker with the confirmed work type, blocker list, and readiness score.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find work-order tracker, dispatch packet, work-type taxonomy, readiness checklist, skill matrix |
| SearchM365 (connectors) | Pull prior work orders for pattern matching, parts availability |
| ReadFileContent | Read taxonomy, checklist, skill matrix, dispatch packet, tracker |

## Guardrails

- **Present classification and blockers as recommendation only** — never auto-clear a blocker or mark dispatch-ready
- **Always show the full readiness checklist** with explicit status for each dimension
- **Flag safety-related blockers with elevated visibility** — missing certifications, unsafe site conditions, equipment recalls must be prominent
- **Surface prior visit issues** for this site or equipment to help dispatchers anticipate problems
- **Never mark a work order as dispatch-ready if any red blocker exists** — red blockers must be resolved first
- **Preserve the customer's original description** alongside the classified work type
- **Log classification decision** — record confirmed work type, blockers, readiness score, and rationale in the tracker after user confirmation
