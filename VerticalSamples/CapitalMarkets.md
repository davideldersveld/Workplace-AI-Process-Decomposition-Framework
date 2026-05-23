# Capital Markets Industry Sample

## Purpose

This document shows what applying the Workplace AI Process Decomposition Framework looks like for a typical Capital Markets enterprise in the United States. It is intended to demonstrate how a capital markets organization can move from a time-sensitive market operations workflow to a governed AI-enabled operating design.

The sample uses a realistic capital markets environment with trade capture, settlement, exception management, and operations control functions across front, middle, and back office teams.

## Capital Markets Context

A large capital markets enterprise typically supports functions such as:

- trade capture and allocations
- confirmations and settlement operations
- trade exceptions and fails management
- collateral and margin operations
- client onboarding and reference data controls
- corporate actions operations
- regulatory reporting and market operations control
- prime brokerage or institutional servicing

These workflows combine structured trade data, highly time-sensitive deadlines, cross-team coordination, and strict market and regulatory controls. That makes Capital Markets a strong fit for process-centric AI when booking changes, settlement commitments, and regulatory actions remain explicitly human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Trade exception and settlement break triage | High | High | High | Low | Strong | Start here |
| Corporate action exception handling | Medium | High | Medium | Low | Strong | Good candidate |
| Margin and collateral exception management | Medium | High | Medium | Low | Strong | Good candidate |
| Client onboarding exception review | Medium | High | Medium | Low | Strong | Later wave |
| Reference data break triage | High | Medium | High | Medium | Moderate | Second wave |
| Regulatory reporting exception handling | Medium | High | Medium | Low | Strong | Later wave |

## Selected Pilot Process

This sample focuses on trade exception and settlement break triage.

### Why This Process Is A Good First Target

- It is high-volume and operationally critical.
- It has clear deadlines, queues, and escalation paths.
- It combines structured market data with manual investigation notes.
- It has measurable outcomes such as break aging, settlement timeliness, and reroute rates.
- AI value is strongest in case packet assembly, classification, and routing rather than booking or settlement action.
- The process already has strong human controls, which makes safe AI boundaries explicit.

## Business Objective

Reduce time to a review-ready trade break packet while preserving settlement control, counterparty accuracy, market operations governance, and auditability.

## Scope

Included in scope:

- trade break intake normalization
- trade, allocation, SSI, and settlement context assembly
- break type and likely root-cause classification
- settlement-risk and aging assessment
- routing to the correct desk or operations function
- internal and counterparty communication drafting

Excluded from scope:

- rebooking or booking correction execution
- settlement instruction changes without approval
- affirmation or confirmation release without human review
- regulatory reporting action changes
- final cash or position movement authorization

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Trade exception and settlement break triage |
| Business objective | Convert trade breaks into complete, prioritized, and routed exception cases ready for operations handling |
| Trigger | Trade exception, affirmation break, SSI issue, or settlement mismatch created |
| Primary outcome | Break case is normalized, evidence-backed, prioritized, and assigned to the correct operations owner |
| Process owner | market operations manager |
| Technical owner | capital markets operations automation lead |
| Primary systems | order or trade management system, settlement platform, SSI repository, confirmations platform, CRM, email, collaboration platform |
| Primary roles | trade support analyst, settlements analyst, middle office analyst, confirmations specialist, operations supervisor |
| SLA target | Standard breaks triaged within 30 minutes |
| Main risk domains | failed trades, incorrect counterparty routing, wrong SSI context, settlement delay, control breaches, regulatory exposure |

## Current-State Workflow Summary

In a typical capital markets environment, breaks and settlement exceptions arise from trade capture mismatches, late allocations, incorrect SSIs, settlement timing issues, and external counterparty discrepancies. Operations teams must assemble trade facts, confirm what changed, identify the root-cause lane, and route quickly to prevent fails, client impact, and downstream operational risk.

The work is essential but often slowed by fragmented data across trade, settlement, and confirmations systems, repeated handoffs, and inconsistent escalation summaries.

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CM-TRD-001 | Intake | Normalize trade break event | Create a usable break case from market operations signals | completeness check | trade or break record | operations control | normalized break case | No |
| CM-TRD-002 | Context | Gather trade, settlement, and counterparty context | Assemble trade facts, SSI, affirmation, and settlement details | context sufficiency | trade and settlement systems | trade support analyst | break context packet | No |
| CM-TRD-003 | Triage | Classify break type and likely cause | Determine dominant break category and probable root cause | category assignment | break taxonomy and trade data | operations analyst | break type, root-cause hypothesis | No |
| CM-TRD-004 | Risk Prep | Assess settlement risk and time criticality | Identify urgency, fail risk, and escalation need | priority recommendation | settlement policy and break aging rules | operations analyst | priority, fail risk, escalation flags | No |
| CM-TRD-005 | Routing | Route to the correct desk or owner | Send break to settlements, confirmations, middle office, or reference data owner | owner selection | routing rules and desk matrix | operations control | routed break | Sometimes |
| CM-TRD-006 | Communication | Draft counterparty and internal handoff summaries | Prepare internal and external coordination drafts | content generation | break packet and templates | operations analyst | draft messages and handoff summary | No |
| CM-TRD-007 | Closure | Confirm triage disposition | Confirm break is ready, escalated, or returned for more investigation | disposition decision | workflow status | operations supervisor | final triage state | Yes for escalations |

## Example Step Record

### CM-TRD-004: Assess Settlement Risk And Time Criticality

```yaml
step_id: CM-TRD-004
step_name: Assess settlement risk and time criticality
stage: Risk Prep
goal: Identify urgency, fail risk, and escalation need for a trade break
trigger: Break classification completed
inputs:
  - trade details
  - settlement date
  - break aging
  - counterparty and SSI context
  - prior exception history
source_of_truth:
  - settlement rules
  - desk routing matrix
decision_type: priority recommendation
systems:
  - settlement platform
  - trade support workflow
  - confirmations platform
human_roles:
  - operations analyst
outputs:
  - priority_recommendation
  - fail_risk_flag
  - escalation_flags
approvals_required: false
exceptions:
  - same-day settlement break
  - SSI mismatch near cutoff
  - repeated counterparty discrepancy
owner: market operations
success_metrics:
  - triage_cycle_time
  - break_aging_reduction
  - reroute_rate
```

## Signal Inventory

### Human Signals

| Signal Type | Capital Markets Example | Why It Matters |
| --- | --- | --- |
| Email | counterparty break notice, client servicing escalation, internal desk handoff | Adds narrative context and timing |
| Chat | trade support coordination, settlement escalation, desk conversations | Reveals urgency and handoff blockers |
| Files | confirmations, allocation files, break reports, SSI documents | Provide the evidence package for break investigation |
| Comments | analyst notes, supervisor annotations, counterparty remarks | Explain why triage or route changed |
| Edits | corrected trade reference, updated counterparty, revised settlement details | Show operational churn and investigation progress |
| Approvals | escalation sign-off, desk manager review, client-impact approval | Define accountable review points |
| Calls and transcripts | counterparty or client servicing call summaries | Provide context for time-critical breaks |

### System Signals

| Signal Type | Capital Markets Example | Why It Matters |
| --- | --- | --- |
| Record state | break status, trade state, affirmation status, settlement state | Anchors the workflow to durable state |
| Events | trade break opened, SSI amended, settlement date change, affirmation received | Trigger progression and re-evaluation |
| Master data | desk, instrument, counterparty, custodian, settlement instructions | Constrains routing and handling path |
| History | prior fails, prior counterparty breaks, repeat instrument issues | Supports pattern detection and triage quality |
| Entitlements | who may amend SSI, route external communication, or escalate sensitive breaks | Prevents unauthorized actions |
| Telemetry | break volume, fail exposure, cutoff proximity, aged break trends | Supports urgency and anomaly detection |

### Model Knowledge

| Knowledge Type | Capital Markets Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize break narratives, rewrite follow-ups, explain next steps | Good for synthesis and drafting |
| Enterprise retrieval | break playbooks, settlement procedures, routing matrix, SSI guidance | Required for grounded capital markets outputs |
| Procedural knowledge | break taxonomy, cutoff rules, fail escalation process | Must be owned by market operations |
| Exemplars | approved break summaries and counterparty outreach examples | Improve consistency |
| Structured business data | trade state, settlement details, break aging, desk routing | Required for correct triage and routing |

### Signal Quality Notes

Capital markets teams should explicitly score signal sources on:

- Freshness, because break and settlement states change rapidly.
- Reliability, because email narrative and system settlement state may diverge.
- Permission sensitivity, because trade and client data are highly restricted.
- Source-of-truth status, because trade and settlement systems outrank informal communication.
- Audit requirement, because every break-handling action may be reviewed by operations control and audit.

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| CM-TRD-001 | Normalize trade break event | Deterministic automation | Stable intake normalization |
| CM-TRD-002 | Gather trade, settlement, and counterparty context | AI act within policy | Safe to retrieve approved context |
| CM-TRD-003 | Classify break type and likely cause | AI assist | Valuable triage aid, but break classification should remain visible to humans |
| CM-TRD-004 | Assess settlement risk and time criticality | AI assist | Useful for prioritization, but not a substitute for market operations judgment |
| CM-TRD-005 | Route to the correct desk or owner | AI draft plus approve | Routing recommendation is useful but must be reviewable |
| CM-TRD-006 | Draft counterparty and internal handoff summaries | AI draft plus approve | Strong drafting value with mandatory review |
| CM-TRD-007 | Confirm triage disposition | Human only for pilot | Final triage and escalation decision remain human-owned |

## AI Task Pattern Mapping

| Step ID | Dominant Task Pattern | Secondary Patterns |
| --- | --- | --- |
| CM-TRD-001 | Extract | Validate |
| CM-TRD-002 | Retrieve | Summarize |
| CM-TRD-003 | Classify | Compare |
| CM-TRD-004 | Compare | Recommend |
| CM-TRD-005 | Route | Recommend |
| CM-TRD-006 | Generate | Summarize |
| CM-TRD-007 | Decide | Escalate |

## Translation To Technical Artifacts

### Skills

| Skill | Purpose | Inputs | Outputs |
| --- | --- | --- | --- |
| Normalize break case | Convert trade break signal into a structured operations case | break event, email, workflow record | normalized break case |
| Build break context packet | Assemble trade, settlement, SSI, and counterparty context | break ID, trade ID, settlement reference | context packet |
| Classify break type | Recommend break category and likely root cause | context packet | break type, confidence |
| Detect fail risk | Identify urgency, cutoff risk, and missing evidence | context packet, risk rules | fail-risk and gap list |
| Recommend routing | Suggest desk or operations owner path | case packet, routing rules | routing recommendation |
| Draft break communications | Produce internal and external drafts | case packet, templates | draft messages and handoff summary |

### Tools And Plugins

| Tool Or Plugin | Action Type | System |
| --- | --- | --- |
| Get trade record | Read | trade management system |
| Get settlement details | Read | settlement platform |
| Get SSI context | Read | SSI repository |
| Get prior break history | Read | break management workflow |
| Get routing matrix | Read | operations policy repository |
| Update break status | Write | workflow or tracker |
| Create review task | Write | workflow platform |
| Log audit event | Write | audit or workflow store |

### Workflow

The primary workflow should coordinate:

1. Break intake normalization.
2. Trade and settlement context retrieval.
3. Break type and likely cause classification.
4. Fail-risk and aging preparation.
5. Routing to the correct desk or owner.
6. Communication draft generation.
7. Triage disposition confirmation.

### Agent Recommendation

For the first capital markets implementation, do not make a broad market operations agent the center of the runtime. Use workflows, skills, and tools with explicit desk, cutoff, and supervisory review gates. Introduce an agent only later for bounded multi-system packet assembly in complex cases.

## Capital Markets Reference Architecture

### Layer 1: Signal Intake And Normalization

Inputs arrive from trade breaks, confirmation mismatches, SSI issues, counterparty notices, and workflow events. The intake layer converts them into a normalized break event.

### Layer 2: Process Model

The process model resolves whether the break is in intake, context assembly, risk prep, routing, or disposition.

### Layer 3: Capability Registry

This layer stores skills, tools, templates, break taxonomies, desk routing rules, cutoff guidance, and communication patterns.

### Layer 4: Runtime Orchestrator

The orchestrator receives the break event, resolves current state, gathers context, selects the allowed execution unit, and records the result.

### Layer 5: Knowledge And Context Builder

The context builder assembles trade details, settlement instructions, break history, affirmation state, counterparty context, and approved procedures.

### Layer 6: Memory And State

Durable state should include break status, desk route, fail-risk flags, assigned owner, prior outputs, manual overrides, and final triage disposition.

### Layer 7: Decision And Approval Plane

This layer enforces same-day cutoff escalation, repeated fail routing, SSI sensitivity controls, and review gates for high-risk breaks.

### Layer 8: Governance And Control Plane

This layer governs access to trading data, routing rules, cutoff procedures, audit logging, and human approval constraints.

### Layer 9: Evaluation And Observability

This layer tracks both skill quality and market operations outcomes such as triage speed, break aging, routing accuracy, fail-risk detection, and reroutes.

## Governance Model For Capital Markets

### Key Governance Rules

1. Trade, settlement, and break systems remain the source of truth for trade and case state.
2. AI may prepare and recommend, but no skill may amend bookings, change SSIs, or authorize settlement action.
3. Counterparty-facing drafts must never imply a booking correction or settlement commitment without human approval.
4. Every write action must preserve actor, timestamp, prior value, and break linkage.
5. Same-day or high-fail-risk breaks must never bypass supervisory review.
6. Sensitive client, counterparty, and trading data must remain scoped to authorized operations users.

### Ownership Model

| Asset | Owner |
| --- | --- |
| Process definition | market operations manager |
| Break and routing rules | operations control and desk leadership |
| Skill contracts and orchestration logic | capital markets automation team |
| Prompt and template assets | joint ownership between operations control and technical owner |
| Data access model | market data owner and enterprise identity team |
| Evaluation benchmarks | market operations with audit and control support |

## Evaluation Plan

### Offline Evaluation Set

Build a benchmark set that includes:

- standard affirmation and SSI breaks
- same-day settlement breaks near cutoff
- repeated counterparty mismatch patterns
- low-severity noise cases that should not escalate
- breaks involving incorrect SSI or allocation context
- aged breaks that should trigger elevated review
- cases with conflicting email narrative and system state

### Acceptance Thresholds For Pilot

| Measure | Target |
| --- | --- |
| Break classification accuracy | at least 85 percent |
| Fail-risk detection rate | at least 85 percent |
| Incorrect routing rate | below 5 percent |
| Unauthorized trade or settlement actions | zero |
| Missing audit fields | zero |
| Draft handoff acceptance rate | at least 75 percent with minor edits only |

### Pilot Design

1. Start with one asset class or one operations desk.
2. Use AI assist and draft-plus-approve modes first.
3. Keep final triage, routing, and escalation approvals under supervisor ownership.
4. Review fail-risk misses, routing errors, and handoff quality weekly.
5. Expand only after stable break-aging and reroute performance.

## Implementation Roadmap For Capital Markets

### Wave 1

- normalize trade break intake events
- assemble trade, settlement, and SSI context
- add break type and root-cause classification assist
- add fail-risk and evidence-gap preparation

### Wave 2

- add routing recommendation to the correct desk or operations owner
- add counterparty and internal handoff draft generation
- add procedure-cited break packet summaries

### Wave 3

- add bounded multi-system break packet assembly for complex or repeated fails
- add proactive detection of repeated counterparty patterns, cutoff misses, and aging risks

## Summary

For a typical US capital markets enterprise, trade exception and settlement break triage is a strong first application of the Workplace AI Process Decomposition Framework because it is high-volume, deadline-driven, and highly improvable without delegating booking, settlement, or regulatory authority to the model.
