# Supply Chain Inventory Shortage and Disruption Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Inventory Shortage and Disruption Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert shortage events, supplier delays, and disruption alerts into a governed, evidence-backed exception handling workflow within Microsoft 365.

The workflow supports intake signals from ERP shortage alerts, supplier delay notifications, planning system exceptions, and manual reports, guiding each through structured normalization, context assembly, exception classification, impact assessment, owner routing, and communication drafting with strict controls for allocation integrity, customer commitment protection, data freshness, cross-functional visibility, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **sc-shortage-intake** | Normalizes shortage events into structured exception case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **sc-context-packet** | Assembles demand, inventory, shipment, supplier, and operational context | AI act within policy | analysis | SearchSparkle |
| 3 | **sc-classify-exception** | Classifies exception type, likely cause, and repeat patterns | AI assist | analysis | Tag |
| 4 | **sc-impact-assess** | Assesses business impact, recommends priority and mitigation path | AI draft + approve | analysis | Flag |
| 5 | **sc-route-exception** | Routes exception to owner per routing rules and ownership model | AI act within policy | communication | Mail |
| 6 | **sc-shortage-comms** | Drafts shortage summaries, supplier follow-ups, and handoff documents | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Shortage         │  Normalize event → structured case record
│     Intake           │  Validates fields, checks duplicates,
│     (sc-shortage-    │  generates Case ID, calculates SLA
│      intake)         │  SLA: 1 hour (critical) to 1 day (low)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble inventory position, demand
│     (sc-context-     │  exposure, shipment status, supplier
│      packet)         │  context, prior history, applicable SOPs
│                      │  Output: Adaptive Card (speed-first)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Exception        │  Classify exception type, determine
│     Classifier       │  likely root cause, detect repeat
│     (sc-classify-    │  patterns and regulatory flags
│      exception)      │  Output: Adaptive Card + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Impact           │  Assess customer impact, recommend
│     Assessment       │  priority, evaluate mitigation options,
│     (sc-impact-      │  check escalation criteria
│      assess)         │  Output: Adaptive Card + priority update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Exception        │  Route to owner per routing rules;
│     Routing          │  send Teams notifications, create
│     (sc-route-       │  SLA deadline calendar holds
│      exception)      │  Output: Teams messages + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Shortage         │  Draft cross-functional updates,
│     Communications   │  supplier follow-ups, customer service
│     (sc-shortage-    │  advisories, shift handoff summaries
│      comms)          │  Output: Outlook drafts + Teams posts
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm case is routed, escalated,
│     Disposition      │  or returned for more data.
│     (not automated)  │  Human-only step — escalation and
│                      │  mitigation commitments remain
│                      │  human-owned.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Shortage Intake | Structured field extraction, validation, duplicate checking, SLA calculation — no AI judgment on classification or priority |
| **AI act within policy** | Context Packet, Exception Routing | Retrieves context from approved data sources or routes within documented ownership rules without exercising judgment on priority or mitigation |
| **AI assist** | Exception Classifier | Surfaces classification recommendations with confidence levels and repeat pattern detection; planner reviews before tracker updates |
| **AI draft + approve** | Impact Assessment, Shortage Communications | AI recommends priority and mitigation path or drafts communications; planner reviews and confirms before actions are taken |
| **Human only** | Confirm Triage Disposition | Final escalation decisions, mitigation commitments, and allocation changes are always human-owned |

## Governance Controls

### Allocation Integrity

- No skill may modify inventory allocations, reorder quantities, or safety stock levels — all allocation changes require human approval with documented authority
- Reallocation recommendations in the impact assessment are presented as options, never auto-executed
- Customer order priority trade-offs (which customer gets stock first) are always human decisions
- Every allocation-related recommendation clearly states the required approval authority

### Customer Commitment Protection

- Skills may draft customer advisories but never send them automatically — customer-facing commitments require operations manager approval
- Customer delivery date changes are never auto-communicated — all delivery commitment modifications require human approval
- Customer service advisories are internal guidance for the service team, not direct customer messages
- No skill makes delivery promises or timeline commitments in any communication

### Data Freshness and Accuracy

- Every inventory, demand, and shipment data point cites its source and retrieval timestamp
- Inventory data older than 24 hours is flagged as potentially stale
- Shipment status data older than 4 hours is flagged as potentially stale
- Context is marked as provisional if any key data source is unavailable
- Revenue exposure estimates are clearly labeled as approximate

### Cross-Functional Visibility

- Critical-priority exceptions automatically trigger operations manager notification in addition to the assigned owner
- Teams channel notifications ensure the supply operations team has visibility into new cases
- Shift handoff summaries ensure continuity across team transitions
- Escalation notices include full case context for management decision-making

### Audit Trail

- Every exception case records the source event, reporting user, creation timestamp, and Case ID
- Every classification records the exception type, root cause, confidence level, and confirming planner
- Every priority decision records the recommended level, rubric justification, and confirming user
- Every routing decision records the assigned owner, routing rationale, and timestamp
- Every communication records the type, audience, channel, and confirming user
- The Excel shortage tracker serves as the Cowork-accessible audit record

### Regulatory and Special Handling

- Items subject to export controls, hazmat regulations, or cold chain requirements are flagged during classification
- Regulated items must not be routed outside approved pathways
- Single-source items (one approved supplier) are flagged for elevated supply risk awareness
- Critical items from the maintained critical items list receive priority handling

### Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Customer order details | Need-to-know only — never in broad distribution messages |
| Supplier pricing and capacity data | Commercially sensitive — never in customer-facing or broad internal communications |
| Internal priority classifications | Operational detail — not shared with suppliers or customers |
| Allocation data and trade-off analysis | Internal only — never in external communications |
| Revenue exposure estimates | Clearly labeled as approximate; internal planning use only |

## Exception Types

| Exception Type | Indicators |
|---------------|-----------|
| **Stockout** | On-hand at zero or below safety stock; demand exceeds available supply |
| **Supplier delay** | Supplier confirmed or expected to miss committed delivery date |
| **Quality hold** | Inventory quarantined due to quality issue or inspection failure |
| **Demand spike** | Demand exceeded forecast significantly; unexpected large order |
| **Logistics disruption** | Shipment delayed, damaged, or rerouted due to carrier, weather, or customs issue |
| **Production disruption** | Manufacturing line issue reducing or stopping output |

## Priority Levels

| Priority | Criteria | SLA | Response Path |
|----------|---------|-----|---------------|
| **Critical** | Customer commitments at risk with no workaround, revenue above threshold, strategic account, or safety/regulatory item | 1 hour | Immediate owner assignment, operations manager notification |
| **High** | Significant customer impact with limited workaround, multiple orders affected, or key production input | 4 hours | Same-day owner assignment, elevated triage |
| **Medium** | Moderate impact with workaround, limited exposure, or recovery expected within lead time | 1 business day | Standard triage queue |
| **Low** | Minimal impact, excess inventory available elsewhere, or informational | 2 business days | Standard queue, next planning review |

## Mitigation Paths

| Path | When Applicable |
|------|----------------|
| **Expedite** | Supplier can accelerate delivery or carrier can expedite transit |
| **Reallocate** | Inventory available at other network sites for transfer |
| **Substitute** | Alternative item or supplier available and qualified |
| **Escalate to supplier** | Root cause is supplier-side; need commitment update or recovery plan |
| **Adjust demand** | Demand can be deferred, reduced, or fulfilled from safety stock |
| **Accept and communicate** | No mitigation available; shortage must be absorbed and communicated |

## Root Cause Categories

| Category | Evidence Patterns |
|----------|------------------|
| **Supplier reliability** | Delivery performance below threshold, repeated delays, capacity constraints |
| **Demand variability** | Forecast miss, unexpected order, seasonal pattern not captured |
| **Planning gap** | Safety stock too low, reorder point not triggered, lead time assumption incorrect |
| **Logistics failure** | Carrier delay, port congestion, customs issue, weather event |
| **Quality issue** | Batch failure, specification change, receiving inspection rejection |
| **External disruption** | Natural disaster, geopolitical event, regulatory change |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find shortage tracker, SOPs, playbooks, taxonomy, severity rubric, routing rules, templates |
| SearchM365 (connectors) | Context Packet, Classify Exception, Impact Assessment — pull inventory, demand, shipment, supplier data via Graph Connectors |
| SearchM365 (email) | Shortage Intake, Context Packet — find supplier delay notifications, internal escalation emails |
| SearchM365 (teams) | Shortage Intake, Context Packet — find planner coordination and escalation threads |
| ReadFileContent | All skills — read tracker, SOPs, playbooks, severity rubrics, templates, bridge data |
| GetDriveChildren | Context Packet — check for supporting documents in the case folder |
| SearchPeople / GetUserDetails | Shortage Intake, Route Exception — resolve planner assignments, owner identities |
| ListCalendarView | Route Exception — check owner availability |
| PostMessage | Shortage Intake, Route Exception, Shortage Comms — Teams notifications to owners, operations channel, operations manager |
| CreateDraftMessage | Shortage Comms — Outlook drafts for supplier follow-ups and formal communications |
| CreateEvent | Route Exception — SLA deadline calendar holds |
| render_ui (Adaptive Card) | All skills — decision surfaces, context summaries, classification reports, impact assessments, routing plans |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Exception case record | Excel tracker row | Shortage Intake |
| Supply context summary | Adaptive Card | Context Packet |
| Classification report | Adaptive Card + Excel tracker update | Classify Exception |
| Impact assessment | Adaptive Card + Excel priority update | Impact Assessment |
| Owner assignment notifications | Teams messages + calendar holds | Route Exception |
| Cross-functional status update | Teams channel post | Shortage Comms |
| Supplier follow-up request | Outlook draft email | Shortage Comms |
| Customer service advisory | Teams message or Outlook draft | Shortage Comms |
| Shift handoff summary | Teams message or Word document | Shortage Comms |
| Escalation notice | Teams message | Shortage Comms |

## Federated Data Access

Supply chain exception workflows depend on ERP, planning, transportation, and supplier portal systems. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | ERP platforms (SAP S/4HANA, Oracle SCM Cloud) for inventory, demand, and order data; TMS platforms (project44, SAP TM) for shipment status; planning systems (Kinaxis, Blue Yonder) for exception events |
| **Tier 2** | SharePoint Bridge | Inventory position snapshots refreshed hourly via Power Automate; shipment status updates via event-triggered flows; supplier performance scorecards maintained in SharePoint |
| **Tier 3** | Manual Input | Supplier verbal commitments, site-specific conditions, ad-hoc quality holds captured via structured intake |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge with hourly Power Automate refreshes for inventory and shipment data) and Tier 3 (manual input for supplier commitments). Prioritize Graph Connectors for ERP and TMS in Wave 2 because the 1-hour SLA for critical exceptions makes real-time data access a material improvement.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── sc-shortage-intake/SKILL.md
├── sc-context-packet/SKILL.md
├── sc-classify-exception/SKILL.md
├── sc-impact-assess/SKILL.md
├── sc-route-exception/SKILL.md
└── sc-shortage-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Shortage tracker** — shared Excel workbook (Case ID, Item/SKU, Site, Shortage Type, Source System Ref, Priority, Status, Created Date, Assigned Planner, Customer Impact Flag, Estimated Recovery Date, SLA Deadline, Root Cause, Routing Rationale)
- **Exception taxonomy** — standard exception type definitions with classification criteria and cause indicators
- **Severity rubric** — priority level definitions with criteria thresholds and escalation triggers
- **Shortage playbooks** — SOPs for each exception type with mitigation steps and escalation criteria
- **Supply chain routing rules** — item category to team mapping, site ownership assignments, exception type routing overrides
- **Supply chain operating model** — team structure, escalation paths, on-call assignments, backup designations
- **Mitigation templates** — standard mitigation approaches by exception type
- **Communication templates** — standard formats for status updates, supplier follow-ups, customer advisories, handoff summaries, escalation notices
- **Supply operations Teams channel** — dedicated channel for exception notifications and cross-functional coordination

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Shortage Intake | "new shortage alert for [item]", "supply disruption for [item] at [site]", "log shortage case", "supplier delay reported" |
| Context Packet | "build context for shortage [ID]", "what's the situation on [item]", "assemble supply context", "gather shortage details" |
| Classify Exception | "classify this shortage", "what type of exception is [case]", "categorize the disruption", "is this a repeat shortage" |
| Impact Assessment | "assess impact of shortage [ID]", "how bad is the disruption", "what's the customer impact", "mitigation options for [case]" |
| Route Exception | "route shortage [ID]", "assign this exception", "who handles [category] shortages", "escalate shortage case" |
| Shortage Comms | "draft shortage update for [case]", "prepare handoff summary", "supplier follow-up for [item]", "cross-functional update on disruption" |

## Implementation Roadmap

### Wave 1 — Foundation

- Create the shared Excel shortage tracker in SharePoint with standard columns
- Upload SOPs, shortage playbooks, exception taxonomy, severity rubric, and routing rules to SharePoint
- Set up Power Automate flows for hourly inventory and shipment data refresh (Tier 2 bridge)
- Create dedicated Teams channel for supply chain exception notifications
- Build skills: `sc-shortage-intake`, `sc-context-packet`, `sc-classify-exception`, `sc-shortage-comms`
- Operate in AI assist mode — all outputs presented via Adaptive Card for planner review
- Test with 10–15 real shortage cases across different exception types

### Wave 2 — Impact Assessment and Controlled Routing

- Build skills: `sc-impact-assess`, `sc-route-exception`
- Promote intake to write mode (creates case records after confirmation)
- Promote classifier to write-back mode (updates tracker after planner confirmation)
- Promote routing to active mode (sends Teams messages and creates calendar holds after confirmation)
- Set up scheduled prompt (every 2 hours): check for cases approaching SLA deadline with unassigned owners or stale status
- Add SOP-cited triage summaries referencing specific playbook sections
- Measurement targets: above 80% classification accuracy, above 85% correct routing rate

### Wave 3 — Advanced Assembly and Proactive Detection

- Introduce Graph Connectors for ERP inventory and TMS shipment data for real-time access
- Add bounded multi-source disruption packet assembly for complex cross-system shortages
- Add proactive recurring shortage pattern detection: scheduled prompt flagging items or sites with repeated exceptions, identifying supplier reliability trends
- Measurement targets: 40% improvement in time to triage for critical cases, above 90% customer impact detection rate, below 20% override rate, 30% aging reduction for high-impact cases

## Implementation Notes

- **Triage disposition is intentionally not automated** — final escalation decisions and mitigation commitments carry operations accountability that cannot be delegated to AI
- **Speed over formality is the design principle** — Adaptive Cards and Teams messages are the primary output formats, not Word documents; the 1-hour SLA for critical exceptions demands fast decision surfaces
- **Classification and impact assessment are separate skills** — what type of exception (classification) is independent of how bad it is (impact), preventing premature priority inflation and supporting clearer decision-making
- **Allocation changes are never automated** — reallocation trades off one customer against another and requires human approval with documented authority
- **Data freshness is critical** — supply chain decisions based on stale data are worse than no decision; every data point cites its timestamp and staleness threshold
- **The Excel tracker is the canonical state** — all skills read from and write to the shared tracker; it serves as the process state store and audit record
- **Repeat pattern detection accelerates triage** — surfacing recurring shortages during classification helps identify systemic issues that need escalation beyond routine handling
- **Cross-system state synchronization is manual in Wave 1** — Power Automate flows bridge ERP and TMS data; Graph Connectors replace them in Wave 2 for real-time access
