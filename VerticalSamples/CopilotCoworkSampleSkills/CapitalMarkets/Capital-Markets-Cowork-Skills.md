# Capital Markets Trade Exception and Settlement Break Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Trade Exception and Settlement Break Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the trade break triage process — from break intake through review-ready case disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports trade exception events, settlement break notifications, SSI mismatches, affirmation failures, and counterparty discrepancies, guiding each through structured intake, context assembly, break classification, risk assessment, desk routing, and communication drafting with T+1 settlement deadline tracking, information barrier enforcement, MNPI protection, booking/settlement action prohibition, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **cm-break-intake** | Normalizes trade break events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **cm-context-packet** | Assembles trade, settlement, SSI, counterparty, and procedure context into an evidence packet | AI act within policy | analysis | SearchSparkle |
| 3 | **cm-break-classifier** | Classifies break type and likely root cause, recommends handling lane | AI assist | analysis | Tag |
| 4 | **cm-risk-assessment** | Assesses settlement risk, fail exposure, time criticality, and evidence gaps | AI assist | analysis | Flag |
| 5 | **cm-break-routing** | Routes breaks to operations desks or escalation lanes per routing matrix | AI draft + approve | communication | Mail |
| 6 | **cm-break-comms** | Drafts counterparty notifications, desk handoffs, escalation memos, and deadline reminders | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Break Intake     │  Normalize trade break event → structured case record
│     (cm-break-       │  Validates required fields, checks duplicate cases
│      intake)         │  SLA: 30-minute triage window
│                      │  Same-day settlement → auto-flag critical
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble trade details, SSI comparison,
│     (cm-context-     │  affirmation status, prior break history,
│      packet)         │  counterparty context, settlement procedures,
│                      │  cutoff schedules, evidence inventory
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Break Classifier │  Compare case attributes against break taxonomy,
│     (cm-break-       │  determine break type, likely root cause,
│      classifier)     │  confidence level, and handling lane
│                      │  Output: Adaptive Card classification report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Risk Assessment  │  Assess fail risk, cutoff proximity, break
│     (cm-risk-        │  aging, evidence gaps, escalation flags,
│      assessment)     │  and priority recommendation
│                      │  Output: Adaptive Card risk dashboard
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Break Routing    │  Route to settlements, confirmations, middle
│     (cm-break-       │  office, reference data, trade support, or
│      routing)        │  operations control per routing matrix;
│                      │  enforce information barriers
│                      │  Output: Teams notifications + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Break            │  Draft counterparty break notices, desk
│     Communications   │  handoff summaries, escalation memos,
│     (cm-break-       │  client servicing notifications, and
│      comms)          │  settlement deadline reminders
│                      │  Output: Outlook drafts + Teams coordination
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm break is review-ready, escalated,
│     Disposition      │  or returned for more investigation.
│     (not automated)  │  Human-only step — regulatory accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Break Intake | Structured field extraction, date validation, duplicate checking — no AI judgment |
| **AI act within policy** | Context Packet | Retrieves approved context from defined sources; does not interpret break patterns or make risk judgments |
| **AI assist** | Break Classifier, Risk Assessment | Surfaces classification and risk findings as recommendations with confidence levels; analyst reviews before any action |
| **AI draft + approve** | Break Routing, Break Comms | AI recommends routing or drafts communications; operations analyst reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final triage disposition, booking corrections, settlement actions, and regulatory decisions are always human-owned |

## Governance Controls

### Regulatory Compliance

- **T+1 settlement deadlines** (SEC Rule 15c6-1) are tracked explicitly — settlement date, cutoff proximity, and hours remaining are surfaced in every skill output
- **Same-day settlement breaks** are auto-flagged as critical regardless of other factors
- **SEC Rule 17a-4 recordkeeping** — every break case record includes actor, timestamp, and intake source; every routing decision records actor, rationale, and confirming supervisor
- **FINRA Rule 3110 supervision** — critical and high-priority breaks require supervisor notification; escalations to operations control require explicit confirmation
- **FINRA Rule 4511 books and records** — all break case artifacts are retained in SharePoint with complete audit trail

### Booking and Settlement Action Prohibition

- No skill may amend bookings, change SSIs, or authorize settlement actions
- No skill may release affirmations or confirmations
- No skill may commit to settlement dates, correction timelines, or resolution outcomes
- No skill may calculate or display financial exposure amounts
- Final disposition decisions carry regulatory accountability and remain human-only

### Information Barriers (Chinese Walls)

- Skills must not route break details across Chinese wall boundaries
- Equity desk break details must not be sent to fixed income personnel (and vice versa)
- Proprietary trading break details must not be sent to client-facing desk personnel
- Counterparty communications must be reviewed for MNPI compliance before sending
- If a routing recommendation may cross an information barrier, it is flagged for compliance review rather than executed
- Information barriers are encoded in skill guardrails since Cowork has no platform-level enforcement

### MNPI Protection

- No skill includes position sizes, pricing data, or unreleased corporate action information in any output
- Trade-level details are limited to what is directly relevant to the break
- Client account numbers are masked in all outputs; counterparty short codes are used instead
- Internal fraud scores, risk ratings, and proprietary trading signals are never surfaced

### Audit Trail

- Every break case record includes actor, timestamp, and intake source for creation audit
- Every routing decision records actor, rationale, assigned desk, assigned owner, and confirming supervisor
- Every document access logs source system, document name, retrieval timestamp, and Graph Connector index timestamp
- Every communication records type, recipient, Break ID, Trade ID, and timestamp
- The Excel break tracker serves as both operational state and examination-grade audit record

### Data Quality and Timeliness

- Graph Connector index timestamps are surfaced when trade or settlement data is sourced from connectors — data may be minutes behind the system of record
- Data sourced from email is flagged with a reliability warning — the system of record takes precedence when email narrative and system state diverge
- Cutoff proximity is calculated and displayed in every output — settlement states change rapidly
- Risk assessments include the current timestamp — they are point-in-time and may become stale

## Break Taxonomy Categories

| Break Type | Description | Handling Lane |
|------------|-------------|---------------|
| **SSI mismatch** | Settlement instructions do not match between counterparties | Settlements desk |
| **Allocation discrepancy** | Trade allocation details are incomplete, incorrect, or missing | Middle office |
| **Affirmation failure** | Trade not affirmed by counterparty within required timeframe | Confirmations desk |
| **Settlement timing issue** | Trade will not settle by expected date due to timing or cutoff constraints | Settlements desk |
| **Counterparty data error** | Counterparty reference data is incorrect or outdated | Reference data team |
| **Booking correction needed** | Trade booking contains errors requiring amendment | Trade support (elevated) |
| **Reference data break** | Security, instrument, or market reference data is incorrect or missing | Reference data team |

## Evidence Gap Categories

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing trade confirmation | Trade confirmation documentation not available | High |
| Missing SSI documentation | Settlement instructions not verified or unmatched | High |
| Missing counterparty response | Counterparty has not responded to break notice | Medium |
| Missing allocation file | Allocation details incomplete or not received | Medium |
| Missing affirmation | Trade not affirmed by counterparty | High |
| Unverified data from email | Data sourced from email not confirmed by system of record | Medium |

## Escalation Flags

| Flag | Description | Required Action |
|------|-------------|-----------------|
| **Same-day settlement fail risk** | Break will fail if not resolved before today's cutoff | Immediate supervisor notification |
| **Cutoff approaching** | Less than 2 hours to settlement cutoff | Urgent desk escalation |
| **SLA breach** | Break exceeds 30-minute triage SLA | Supervisor notification |
| **Repeat counterparty pattern** | Same counterparty with 3+ breaks in 30 days | Pattern review escalation |
| **Missing critical evidence** | Trade confirmation or SSI documentation not available | Evidence collection urgency |
| **Cross-market complexity** | Break involves multiple markets, custodians, or currencies | Operations control review |

## Priority Levels

| Level | Criteria |
|-------|----------|
| **Critical** | Same-day settlement break; approaching cutoff with unresolved issue; prior fail on same counterparty within 30 days; high-value trade with fail exposure |
| **High** | Next-business-day settlement; SSI mismatch not yet responded to; break aging exceeds 30-minute SLA; multiple evidence gaps |
| **Standard** | Future-dated settlement with adequate time; break classified and evidence substantially complete; no fail-risk indicators |
| **Low** | Future-dated settlement with complete evidence; minor discrepancy with clear resolution path; no counterparty or timing risk |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Break Intake, Context Packet — find break notifications, counterparty correspondence |
| SearchM365 (files) | All skills — find break tracker, break taxonomy, routing matrix, cutoff schedules, SSI reference, templates |
| SearchM365 (connectors) | Break Intake, Context Packet, Risk Assessment — retrieve trade and settlement data from OMS and settlement platform via Graph Connector |
| ReadFileContent | All skills — read tracker, taxonomy, routing matrix, cutoff schedules, SSI reference, context packets, templates |
| GetDriveChildren | Context Packet — browse break case evidence folders |
| SearchPeople / GetUserDetails | All skills — resolve analysts, desk owners, supervisors, counterparty relationship managers |
| GetManagerDetails / GetDirectReportsDetails | Break Routing — org structure for escalation paths |
| CreateDraftMessage | Break Comms — Outlook drafts for counterparty and formal communications (never auto-send) |
| PostMessage | Break Routing, Break Comms — Teams notifications to desks and urgent deadline reminders |
| CreateEvent | Break Routing — calendar holds for settlement cutoff deadlines |
| ListCalendarView | Break Routing — check analyst availability for urgent assignments |
| render_ui (Adaptive Card) | Break Intake, Break Classifier, Risk Assessment, Break Routing — confirmations, classifications, risk dashboards |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Trade break case record | Excel tracker row | Break Intake |
| Trade break context packet | Word document | Context Packet |
| Break classification report | Adaptive Card | Break Classifier |
| Settlement risk dashboard | Adaptive Card | Risk Assessment |
| Routing recommendation | Adaptive Card | Break Routing |
| Desk assignment notifications | Teams direct messages | Break Routing |
| Settlement cutoff calendar holds | Calendar events | Break Routing |
| Counterparty break notification | Outlook draft | Break Comms |
| Internal desk handoff summary | Outlook draft or Teams message | Break Comms |
| Operations control escalation memo | Outlook draft | Break Comms |
| Client servicing notification | Outlook draft | Break Comms |
| Settlement deadline reminder | Teams message | Break Comms |

## Federated Data Access

Capital markets trade break workflows depend on external systems (OMS, settlement platform, SSI repository, confirmations engine). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | OMS trade records, settlement platform status, and SSI data indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Break taxonomy, routing matrix, cutoff schedules, SSI snapshots, and counterparty reference data maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Trade-specific details not available through integration, captured via structured prompts from analyst OMS lookups |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (taxonomy, routing matrix, cutoff schedules, SSI snapshots) and Tier 3 for trade-specific details from manual OMS lookups. Introduce Graph Connectors for OMS and settlement platform data in Wave 3 after skill workflows are proven and connector options are evaluated for the firm's specific trade and settlement systems.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── cm-break-intake/SKILL.md
├── cm-context-packet/SKILL.md
├── cm-break-classifier/SKILL.md
├── cm-risk-assessment/SKILL.md
├── cm-break-routing/SKILL.md
└── cm-break-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Trade break tracker** — shared Excel workbook (Break ID, Trade ID, Asset Class, Instrument, Counterparty, Settlement Date, Break Source, Break Type, Priority, Fail Risk, Status, Created Timestamp, Assigned To, Desk, Disposition, Cutoff Proximity, Audit Log)
- **Break taxonomy** — break type definitions, pattern indicators, handling lane criteria, classification playbook
- **Settlement cutoff schedules** — cutoff times by market, custodian, and currency
- **SSI reference** — standing settlement instructions for counterparties
- **Desk routing matrix** — who handles what by break type, asset class, priority level, and escalation threshold
- **Fail escalation procedures** — fail escalation rules, supervisor notification thresholds, operations control triggers
- **Settlement procedures** — settlement handling rules, T+1 compliance guidance, affirmation and confirmation requirements
- **Communication templates** — approved templates for counterparty break notice, desk handoff summary, escalation memo, client servicing notification, deadline reminder
- **Break case folder structure** — per-break folders in SharePoint for evidence storage

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Break Intake | "new trade break", "settlement exception for [trade ID]", "log break case", "trade fail alert", "SSI mismatch on [counterparty]" |
| Context Packet | "build break packet for [trade ID]", "assemble context for break [ID]", "what do we know about this trade exception", "pull settlement details" |
| Break Classifier | "classify this break", "what type of break is [ID]", "root cause analysis for trade exception", "categorize the settlement mismatch" |
| Risk Assessment | "assess fail risk for [trade ID]", "how urgent is this break", "settlement risk check", "is this break near cutoff", "check aging on break [ID]" |
| Break Routing | "route this break", "who handles [break type]", "assign break to the right desk", "send to settlements", "escalate to operations control" |
| Break Comms | "draft counterparty notice for [break ID]", "prepare handoff summary", "write internal escalation email", "settlement break communication" |

## Implementation Roadmap

### Wave 1 — Foundation (Intake, Context, Classification, Risk)

- Create the shared Excel trade break tracker in SharePoint
- Upload break taxonomy, settlement cutoff schedules, SSI reference, routing matrix, and fail escalation procedures
- Create per-break case evidence folder structure in SharePoint
- Establish Power Automate flows to export daily SSI snapshots and break reports from source systems
- Build skills: `cm-break-intake`, `cm-context-packet`, `cm-break-classifier`, `cm-risk-assessment`
- Operate in AI assist mode — all outputs presented for analyst review
- Test with one asset class and one operations desk, covering standard breaks, same-day settlement breaks, and repeat counterparty patterns

### Wave 2 — Routing and Communications

- Build skills: `cm-break-routing`, `cm-break-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote classifier and risk assessment to write-back mode (updates tracker after analyst confirmation)
- Set up scheduled prompt to check for breaks approaching settlement cutoff without confirmed disposition
- Set up daily scheduled prompt to report break aging trends and repeat counterparty patterns

### Wave 3 — Optimization and Proactive Detection

- Introduce Graph Connectors for OMS and settlement platform data
- Add proactive monitoring: repeat counterparty mismatch patterns, cutoff proximity risks across the break book, aged breaks trending toward fail
- Add bounded multi-system break packet assembly for complex or repeated fails
- Measurement targets: 85%+ classification accuracy, 85%+ fail-risk detection rate, below 5% incorrect routing rate, 75%+ draft handoff acceptance rate, zero unauthorized booking or settlement actions, zero missing audit fields

## Implementation Notes

- **Triage disposition is intentionally not automated** — final break disposition decisions carry regulatory accountability that cannot be delegated to AI
- **No skill performs booking or settlement actions** — this is a permanent architectural constraint; booking corrections, SSI changes, settlement authorization, and affirmation releases are always human-owned
- **Information barriers are encoded in guardrails** — Cowork has no platform-level Chinese wall enforcement; skills rely on guardrail instructions and SharePoint permissions to prevent cross-desk data leakage
- **The 30-minute SLA** drives urgency throughout the workflow — every skill surfaces SLA status and cutoff proximity in its output
- **Cutoff proximity appears in every output** — settlement states change rapidly; operations teams must always see time remaining
- **Graph Connector latency** is explicitly handled — skills surface the index timestamp when trade data is sourced from connectors, since data may be minutes behind the system of record
- **The Adaptive Card is the primary operational interface** — time sensitivity demands immediate decision support at the desk, not document navigation
- **The Excel tracker serves dual duty** — operational state store and examination-grade audit record; every skill write includes actor, action, timestamp, and prior value
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a risk assessment on an existing break, or drafting a counterparty notice without re-running the full workflow)
