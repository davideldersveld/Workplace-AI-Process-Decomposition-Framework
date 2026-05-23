# Field Service Work-Order Triage and Dispatch Readiness — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Work-Order Triage and Dispatch Readiness** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the field service triage process — from work-order intake through dispatch communications — into AI-assisted capabilities within Microsoft 365.

The workflow supports inbound work orders from email, service requests, chat, and phone intake, guiding each through structured classification, blocker detection, urgency assessment, technician routing, and dispatch communications with safety controls, schedule integrity, and parts compliance at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **fs-workorder-intake** | Normalizes inbound work-order events into structured tracker records | Deterministic automation | analysis | TaskListLtr |
| 2 | **fs-dispatch-packet** | Assembles asset history, site details, parts, skills, and prior visit context | AI act within policy | analysis | SearchSparkle |
| 3 | **fs-blocker-classifier** | Classifies work type and identifies dispatch blockers with readiness scorecard | AI assist | analysis | Tag |
| 4 | **fs-urgency-assessment** | Assesses urgency, dispatch path, and escalation conditions | AI draft + approve | analysis | Flag |
| 5 | **fs-dispatch-routing** | Routes to the correct technician or queue based on skill matrix and availability | AI act within policy | communication | Mail |
| 6 | **fs-dispatch-comms** | Drafts customer confirmations, technician briefings, and escalation notifications | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Work-Order       │  Normalize inbound event → structured tracker record
│     Intake           │  Supports: email, service request, chat, phone intake
│     (fs-workorder-   │  Calculates SLA deadline (30-min triage target)
│      intake)         │  Flags safety-sensitive work types
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Dispatch Packet  │  Assemble asset history, site details, parts needs,
│     (fs-dispatch-    │  technician requirements, prior visits, key contacts
│      packet)         │  Output: Word dispatch context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Blocker          │  Classify work type, check readiness across four
│     Classifier       │  dimensions (parts, skills, access, schedule),
│     (fs-blocker-     │  produce readiness scorecard
│      classifier)     │  Output: Adaptive Card readiness report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Urgency          │  Assess urgency (Emergency/High/Standard/Scheduled),
│     Assessment       │  determine dispatch path, identify escalation conditions,
│     (fs-urgency-     │  evaluate appointment impact
│      assessment)     │  Output: Adaptive Card urgency report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Dispatch         │  Route to technician per skill matrix and routing rules,
│     Routing          │  check calendar availability, send Teams notifications
│     (fs-dispatch-    │  Output: Teams messages + tracker update
│      routing)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Dispatch         │  Draft customer appointment confirmation, technician
│     Comms            │  briefing, reschedule notice, or escalation notification
│     (fs-dispatch-    │  Output: Outlook draft + Teams briefing + Word doc
│      comms)          │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Dispatch         │  Confirm dispatch readiness — approve routing,
│     Readiness        │  authorize safety-flagged dispatch, or return for
│     (not automated)  │  blocker resolution. Human-only step.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Work-Order Intake | Structured field extraction, duplicate checking, SLA calculation — no AI judgment |
| **AI act within policy** | Dispatch Packet, Dispatch Routing | Retrieves approved context and routes per the skill matrix; does not interpret or override rules |
| **AI assist** | Blocker Classifier | Surfaces classification and blocker report as recommendation with readiness scorecard; dispatcher reviews before applying |
| **AI draft + approve** | Urgency Assessment, Dispatch Comms | AI recommends urgency or drafts communication; dispatcher reviews and confirms before any action |
| **Human only** | Dispatch Readiness | Final dispatch-readiness confirmation and escalation authorization are always human decisions |

## Governance Controls

### Safety Controls
- Safety-sensitive work types (electrical, gas, confined space, hazardous materials, elevated work) are flagged at intake and tracked throughout the workflow
- Safety-flagged work orders are never auto-dispatched — dispatch lead review is required before routing
- Technician certification is verified against the skill matrix before assignment — no technician is assigned work requiring certifications they do not hold
- Safety warnings are included prominently in all technician briefings for hazardous work

### Schedule Integrity
- Appointment windows are tracked in the work-order tracker and surfaced at every step
- Technician calendar availability is checked via Calendar before assignment
- Appointment impact is assessed during urgency evaluation — if the window will be missed, the expected delay is calculated and flagged
- Scheduling conflicts are flagged proactively, not discovered at dispatch time

### Parts and Skills Compliance
- The blocker classifier checks parts availability, technician certifications, and special tooling requirements before declaring dispatch readiness
- No work order is marked dispatch-ready if any red blocker exists across the four dimensions (parts, skills, access, schedule)
- Parts pickup instructions and special tooling requirements are included in every technician notification

### Customer Communication Control
- All customer-facing communications are created as Outlook drafts — never sent without explicit dispatcher confirmation
- No outcome or cost commitments are made in customer communications — appointment confirmations confirm timing and preparation only
- Internal details (urgency levels, blocker classifications, routing information) never appear in customer-facing drafts

### SLA Awareness
- SLA deadline is calculated at intake (30 minutes for standard triage) and tracked in the work-order tracker
- Every skill surfaces the SLA countdown
- Urgency assessment determines the dispatch path based on urgency level and appointment commitments
- Scheduled prompts can monitor for approaching SLA breaches (recommended: every 30 minutes)

## Urgency Levels and Dispatch Paths

| Urgency | Description | Dispatch Path |
|---------|-------------|---------------|
| **Emergency** | Safety hazard, active outage, or imminent property damage | Immediate dispatch — dispatch lead must confirm |
| **High** | Confirmed appointment at risk, premium customer impacted, or repeat failure | Next-available slot — priority queue |
| **Standard** | Normal work order with adequate lead time before appointment | Scheduled within normal queue |
| **Scheduled** | Preventive maintenance, inspection, or non-urgent installation | Planned maintenance window |

## Readiness Scorecard Dimensions

| Dimension | What It Checks | Red Blocker Example |
|-----------|---------------|-------------------|
| **Parts** | Required parts in stock and available | Parts backordered past the appointment window |
| **Skills** | Technician with required certifications available | No certified technician for gas line work in the region |
| **Access** | Site access confirmed and clearances obtained | Building management approval not yet received |
| **Schedule** | Technician available in the requested window | All qualified technicians booked during the window |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Intake, Dispatch Packet — find inbound work-order emails and prior correspondence |
| SearchM365 (files) | All skills — find tracker, taxonomy, policies, skill matrix, templates, asset documents |
| SearchM365 (teams) | Dispatch Packet — find internal discussions about the customer site |
| SearchM365 (connectors) | Intake, Dispatch Packet, Blocker Classifier, Urgency — field service platform and inventory data |
| ReadFileContent | All skills — read tracker, taxonomy, policies, skill matrix, dispatch packet, templates |
| GetDriveChildren | Dispatch Packet — browse site-specific evidence folders |
| GetMessage | Intake, Dispatch Comms — read original inbound email |
| SearchPeople / GetUserDetails | Intake, Dispatch Packet, Routing — resolve identities |
| GetMyDetails | Intake — current user for tracking |
| GetDirectReportsDetails | Routing — technician team structure under dispatch lead |
| ListCalendarView | Routing — check technician availability |
| CreateDraftMessage | Dispatch Comms — customer appointment confirmations and updates (never auto-send) |
| PostMessage | Routing, Dispatch Comms — Teams notifications to technicians and dispatch team |
| PostChannelMessage | Routing — work-order card to dispatch channel |
| render_ui (Adaptive Card) | Blocker Classifier, Urgency Assessment, Intake — readiness scorecards and urgency reports |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Work-order record | Excel tracker row | Work-Order Intake |
| Dispatch context packet | Word document | Dispatch Packet |
| Readiness scorecard | Adaptive Card | Blocker Classifier |
| Urgency assessment | Adaptive Card | Urgency Assessment |
| Technician notification | Teams direct message | Dispatch Routing |
| Dispatch channel card | Teams channel post | Dispatch Routing |
| Appointment confirmation | Outlook draft | Dispatch Comms |
| Reschedule notice | Outlook draft | Dispatch Comms |
| Post-visit follow-up | Outlook draft | Dispatch Comms |
| Technician dispatch briefing | Word document | Dispatch Comms |
| Dispatch team escalation | Teams message | Dispatch Comms |

## Federated Data Access

Field Service depends on external systems (field service platform, scheduling engine, inventory/parts system, asset management). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Field service platform work-order records, asset histories, parts availability indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Technician skill matrix, parts availability lookup, asset history, site access notes synced via Power Automate |
| **Tier 3** | Manual Input | Scheduling data, IoT telemetry, systems with no integration path |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for skill matrix, parts availability, and asset history, and Tier 3 for scheduling and IoT data. Introduce Graph Connectors for the field service platform and inventory system in Wave 2.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── fs-workorder-intake/SKILL.md
├── fs-dispatch-packet/SKILL.md
├── fs-blocker-classifier/SKILL.md
├── fs-urgency-assessment/SKILL.md
├── fs-dispatch-routing/SKILL.md
└── fs-dispatch-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Work-order tracker** — shared Excel workbook (Work Order ID, Customer, Account ID, Site Address, Work Type, Issue Summary, Requested Window, Parts Required, Technician Assigned, Priority, Status, SLA Deadline, Dispatch Ready, Blockers, Safety Flags, Resolution)
- **Work-type taxonomy** — document defining work types, readiness checklists, and category-to-queue mappings
- **Dispatch policy and priority rubric** — document defining urgency levels, dispatch paths, and escalation criteria
- **Technician skill matrix** — Excel workbook defining technician skills, certifications, regions, and availability
- **Routing rules** — defines which technicians and queues handle which work types and urgency levels
- **Communication templates** — approved templates for appointment confirmations, reschedule notices, technician briefings, and escalation notifications
- **Site access guides** — documents with site-specific access requirements, safety notes, and contact information

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Work-Order Intake | "new work order", "log work order for Acme Corp", "service request from 123 Main St", "intake work order" |
| Dispatch Packet | "build dispatch packet for WO-2026-00142", "pull context for this job", "what do we need for this work order" |
| Blocker Classifier | "classify this work order", "check for blockers on WO-2026-00142", "what could block this dispatch", "readiness scorecard" |
| Urgency Assessment | "assess urgency for WO-2026-00142", "what priority is this job", "is this an emergency dispatch", "check dispatch path" |
| Dispatch Routing | "assign technician for WO-2026-00142", "route to dispatch queue", "who should handle this job", "dispatch this work order" |
| Dispatch Comms | "draft appointment confirmation for WO-2026-00142", "prepare technician briefing", "write reschedule notice", "dispatch escalation" |

## Implementation Roadmap

### Wave 1 — Foundation
- Create the shared Excel work-order tracker in SharePoint
- Upload dispatch policies, skill matrix, readiness checklists, safety guidelines, and communication templates
- Configure SharePoint bridge data: technician skill matrix, parts availability lookup, asset history, site access notes
- Build skills: `fs-workorder-intake`, `fs-dispatch-packet`, `fs-blocker-classifier`, `fs-dispatch-comms`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 10-15 real work orders across multiple work types and regions

### Wave 2 — Routing and Urgency
- Build skills: `fs-urgency-assessment`, `fs-dispatch-routing`
- Promote intake to write mode (creates records after confirmation)
- Promote blocker classifier to write-back mode (updates tracker after confirmation)
- Set up scheduled prompt (every 30 minutes) for SLA breach monitoring
- Set up daily summary prompt for work-order volume, triage time, and dispatch readiness rate
- Introduce Graph Connectors for field service platform and inventory system if available

### Wave 3 — Optimization and Proactive Detection
- Add bounded multi-source dispatch packet assembly
- Add proactive repeat-visit detection (analyze completed work orders for return-visit patterns)
- Add parts dependency monitoring (check upcoming appointments against parts availability)
- Add dispatch-day readiness check (morning scorecard for all scheduled work orders)
- Measurement targets: 85%+ classification accuracy, 90%+ correct routing rate, under 20 minutes to dispatch-ready, 80%+ blocker detection rate, below 5% appointment reschedule rate due to missing prep, zero safety incidents

## Implementation Notes

- **Dispatch readiness confirmation is intentionally not automated** — final dispatch authorization and safety-flagged approvals are human-only decisions
- **All customer-facing communications are Outlook drafts** — nothing is sent without explicit dispatcher confirmation
- **The blocker classifier's readiness scorecard is the primary decision surface** — dispatchers need to see parts/skills/access/schedule status at a glance
- **Calendar is a first-class process artifact** — technician availability is a hard routing constraint, not just a scheduling convenience
- **Safety guardrails are the strictest governance requirement** — incorrect dispatch has physical-world consequences; safety-sensitive work types have additional confirmation gates, certification verification, and elevated visibility
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., checking blockers for an existing work order, or drafting a reschedule notice without re-running the full workflow)
