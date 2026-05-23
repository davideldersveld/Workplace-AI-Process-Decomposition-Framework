# Banking Industry Sample

## Purpose

This document shows what applying the Workplace AI Process Decomposition Framework looks like for a typical Banking enterprise in the United States. It is intended to demonstrate how a banking organization can move from a high-risk operational workflow to a governed AI-enabled operating design.

The sample uses a realistic banking environment with core banking systems, fraud operations, dispute handling teams, customer servicing, and strong regulatory controls.

## Banking Context

A large US-based banking enterprise typically supports functions such as:

- deposit operations
- payments and card operations
- fraud detection and dispute handling
- customer onboarding and KYC exception handling
- lending operations and servicing
- treasury and cash management operations
- complaint management and regulatory response
- branch and contact center servicing

These workflows combine structured transaction data, customer interactions, regulatory requirements, and strict approval and audit needs. That makes Banking a strong fit for process-centric AI when account restrictions, dispute decisions, and compliance judgments remain human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Fraud alert and dispute triage | High | High | High | Low | Strong | Start here |
| KYC exception review | High | High | Medium | Low | Strong | Good candidate |
| Loan document deficiency handling | Medium | High | Medium | Medium | Strong | Good candidate |
| Wire payment exception triage | Medium | High | High | Low | Strong | Later wave |
| Consumer complaint routing | Medium | High | Medium | Low | Strong | Later wave |
| Treasury exception handling | Medium | Medium | Medium | Medium | Moderate | Second wave |

## Selected Pilot Process

This sample focuses on fraud alert and dispute triage for retail and small-business banking.

### Why This Process Is A Good First Target

- It is high-volume and operationally important.
- It has clear SLAs, routing paths, and analyst roles.
- It combines customer narrative, transaction data, and fraud signals.
- It has measurable outcomes such as triage speed, false positives, and case aging.
- AI value is concentrated in preparation, summarization, and routing rather than final account action.
- Strong controls already exist, making the human/AI boundary easier to define.

## Business Objective

Reduce time to a review-ready fraud or dispute case while preserving customer protection, regulatory compliance, account security, and analyst accountability.

## Scope

Included in scope:

- fraud and dispute case intake normalization
- transaction and account context assembly
- fraud scenario and dispute type classification
- missing evidence and risk signal detection
- routing to fraud operations, disputes, or AML-related review lanes
- customer and analyst communication drafting

Excluded from scope:

- automatic card blocking or account restriction
- final dispute decisioning
- provisional credit authorization
- SAR or suspicious activity decisioning
- final customer reimbursement decisions without human approval

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Fraud alert and dispute triage |
| Business objective | Convert alerts and customer dispute signals into complete, routed, review-ready fraud cases |
| Trigger | Fraud alert fired, dispute submitted, or customer report received |
| Primary outcome | Case is normalized, evidence-backed, prioritized, and assigned to the correct banking operations team |
| Process owner | fraud operations manager |
| Technical owner | banking operations automation lead |
| Primary systems | core banking platform, card processor, fraud platform, CRM, case management system, email, telephony transcripts, collaboration platform |
| Primary roles | fraud analyst, dispute specialist, fraud operations supervisor, customer service representative, AML liaison |
| SLA target | Standard cases triaged within 30 minutes |
| Main risk domains | false negatives, unauthorized account action, Reg E or card dispute compliance failures, privacy exposure, missed suspicious activity escalation |

## Current-State Workflow Summary

In a typical bank, transaction alerts and dispute reports arrive from monitoring systems, contact centers, digital channels, and branches. Operations teams must quickly identify the account, transaction context, customer story, prior alerts, known merchant patterns, and whether the case belongs in fraud, disputes, or another control function.

The work is essential but often slowed by fragmented signals, repeated customer questioning, weak first-pass summaries, and manual cross-checking across transaction, fraud, and case systems.

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BNK-FRD-001 | Intake | Normalize fraud or dispute event | Create a usable banking case from inbound signals | completeness check | fraud or dispute intake record | fraud operations | normalized case | No |
| BNK-FRD-002 | Context | Gather account, transaction, and customer context | Assemble account, card, transaction, and case history context | context sufficiency | core banking and fraud systems | fraud analyst | fraud context packet | No |
| BNK-FRD-003 | Triage | Classify fraud scenario or dispute type | Determine likely fraud/dispute pattern and handling lane | category assignment | fraud taxonomy and transaction data | fraud analyst | scenario type, confidence | No |
| BNK-FRD-004 | Risk Prep | Assess urgency, evidence gaps, and risk path | Identify urgency, missing evidence, and escalation needs | priority recommendation | fraud rules and dispute checklist | fraud analyst | risk flags, evidence gaps, priority recommendation | No |
| BNK-FRD-005 | Routing | Route to analyst queue or escalation lane | Send case to the correct operations or review team | owner selection | routing rules and reviewer matrix | fraud operations | routed case | Sometimes |
| BNK-FRD-006 | Communication | Draft customer and analyst communications | Prepare customer outreach, affidavit requests, and analyst summary | content generation | case packet and approved templates | fraud operations | draft messages and case summary | No |
| BNK-FRD-007 | Closure | Confirm triage disposition | Confirm case is ready, escalated, or returned for more intake | disposition decision | case workflow status | fraud supervisor | final triage state | Yes for escalations |

## Example Step Record

### BNK-FRD-004: Assess Urgency, Evidence Gaps, And Risk Path

```yaml
step_id: BNK-FRD-004
step_name: Assess urgency, evidence gaps, and risk path
stage: Risk Prep
goal: Identify urgency, evidence gaps, and whether the case requires elevated review
trigger: Fraud context packet completed
inputs:
  - transaction history
  - alert details
  - customer narrative
  - prior fraud case history
  - dispute evidence checklist
source_of_truth:
  - fraud operations rules
  - dispute checklist and routing matrix
decision_type: priority recommendation
systems:
  - fraud platform
  - core banking system
  - case management platform
human_roles:
  - fraud analyst
outputs:
  - urgency_recommendation
  - evidence_gap_list
  - escalation_flags
approvals_required: false
exceptions:
  - repeated alerts on same merchant
  - cross-border or unusual channel usage
  - disputed transaction lacks customer confirmation
owner: fraud operations
success_metrics:
  - triage_cycle_time
  - false_positive_reduction
  - analyst_rework_rate
```

## Signal Inventory

### Human Signals

| Signal Type | Banking Example | Why It Matters |
| --- | --- | --- |
| Email | customer dispute follow-up, branch escalation, merchant correspondence | Adds narrative context and timing |
| Chat | internal fraud coordination and supervisor escalation | Reveals urgency and ambiguity |
| Files | affidavits, screenshots, cardholder statements, merchant documents | Provide the primary dispute evidence |
| Comments | analyst notes, supervisor remarks, case annotations | Explain why triage or routing changed |
| Edits | corrected transaction reference, reclassified dispute reason | Show case instability and review churn |
| Approvals | account action approval, escalation sign-off, provisional credit review | Define accountable decision points |
| Calls and transcripts | cardholder call notes or branch narratives | Provide the richest initial fraud/dispute narrative |

### System Signals

| Signal Type | Banking Example | Why It Matters |
| --- | --- | --- |
| Record state | account status, card status, case status, alert state | Anchors the workflow to durable state |
| Events | fraud alert fired, dispute filed, case reopened | Trigger progression and re-evaluation |
| Master data | customer tier, account type, channel, merchant category, product type | Constrains routing and handling path |
| History | prior fraud alerts, prior disputes, prior account restrictions | Supports pattern detection and triage quality |
| Entitlements | who may view customer data, block cards, or escalate regulated cases | Prevents unauthorized actions |
| Telemetry | fraud scores, transaction velocity, repeated merchant exposure | Supports urgency and anomaly detection |

### Model Knowledge

| Knowledge Type | Banking Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize customer narratives, rewrite clarification requests, explain next steps | Good for synthesis and drafting |
| Enterprise retrieval | dispute procedures, fraud playbooks, routing matrix, evidence checklists | Required for grounded banking outputs |
| Procedural knowledge | fraud taxonomy, Reg E handling guidance, review thresholds | Must be owned by banking operations |
| Exemplars | approved case summaries and customer follow-up examples | Improve consistency |
| Structured business data | transaction state, account state, alert data, routing rules | Required for correct triage and routing |

### Signal Quality Notes

Banking teams should explicitly score signal sources on:

- Freshness, because fraud and dispute cases are highly time-sensitive.
- Reliability, because customer narrative and transaction telemetry may conflict.
- Permission sensitivity, because account and transaction data are highly restricted.
- Source-of-truth status, because core banking and card systems outrank informal communications.
- Audit requirement, because every triage action may be reviewed for regulatory and operational compliance.

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| BNK-FRD-001 | Normalize fraud or dispute event | Deterministic automation | Stable intake normalization |
| BNK-FRD-002 | Gather account, transaction, and customer context | AI act within policy | Safe to retrieve approved context |
| BNK-FRD-003 | Classify fraud scenario or dispute type | AI assist | Valuable triage aid, but scenario classification should remain visible to analysts |
| BNK-FRD-004 | Assess urgency, evidence gaps, and risk path | AI assist | Useful for prioritization, but not a substitute for banking risk judgment |
| BNK-FRD-005 | Route to analyst queue or escalation lane | AI draft plus approve | Routing recommendation is useful but must be reviewable |
| BNK-FRD-006 | Draft customer and analyst communications | AI draft plus approve | Strong drafting value with mandatory review |
| BNK-FRD-007 | Confirm triage disposition | Human only for pilot | Final triage and escalation decision remain human-owned |

## AI Task Pattern Mapping

| Step ID | Dominant Task Pattern | Secondary Patterns |
| --- | --- | --- |
| BNK-FRD-001 | Extract | Validate |
| BNK-FRD-002 | Retrieve | Summarize |
| BNK-FRD-003 | Classify | Compare |
| BNK-FRD-004 | Compare | Recommend |
| BNK-FRD-005 | Route | Recommend |
| BNK-FRD-006 | Generate | Summarize |
| BNK-FRD-007 | Decide | Escalate |

## Translation To Technical Artifacts

### Skills

| Skill | Purpose | Inputs | Outputs |
| --- | --- | --- | --- |
| Normalize banking case | Convert inbound fraud/dispute signal into a structured case | alert, report, transcript, email | normalized case |
| Build fraud context packet | Assemble transaction, account, and case history context | case ID, account ID, transaction ID | context packet |
| Classify fraud or dispute type | Recommend scenario and handling lane | context packet | scenario type, confidence |
| Detect evidence gaps | Identify missing documents and escalation flags | case packet, evidence checklist | gap list, urgency recommendation |
| Recommend routing | Suggest fraud, disputes, chargebacks, or escalation path | case packet, routing rules | routing recommendation |
| Draft banking communications | Produce customer and analyst drafts | case packet, templates | draft messages and case summary |

### Tools And Plugins

| Tool Or Plugin | Action Type | System |
| --- | --- | --- |
| Get account profile | Read | core banking system |
| Get transaction details | Read | payments or card platform |
| Get prior case history | Read | case management or fraud platform |
| Get fraud score context | Read | fraud analytics platform |
| Get dispute checklist | Read | operations policy repository |
| Update triage status | Write | case workflow or tracker |
| Create review task | Write | workflow platform |
| Log audit event | Write | audit or workflow store |

### Workflow

The primary workflow should coordinate:

1. Event intake normalization.
2. Account and transaction context retrieval.
3. Scenario and dispute type classification.
4. Evidence gap and urgency preparation.
5. Routing to the correct analyst lane.
6. Communication draft generation.
7. Triage disposition confirmation.

### Agent Recommendation

For the first banking implementation, do not make a broad banking operations agent the center of the runtime. Use workflows, skills, and tools with explicit fraud, dispute, and supervisory review gates. Introduce an agent only later for bounded multi-document case assembly.

## Banking Reference Architecture

### Layer 1: Signal Intake And Normalization

Inputs arrive from alerts, digital channels, call transcripts, branch notices, and dispute submissions. The intake layer converts them into a normalized banking case event.

### Layer 2: Process Model

The process model resolves whether the case is in intake, fraud prep, evidence review, routing, or disposition.

### Layer 3: Capability Registry

This layer stores skills, tools, templates, fraud taxonomies, routing rules, evidence checklists, and communication patterns.

### Layer 4: Runtime Orchestrator

The orchestrator receives the case event, resolves state, gathers context, selects the allowed execution unit, and records the result.

### Layer 5: Knowledge And Context Builder

The context builder assembles transaction data, account details, prior alerts, dispute procedures, fraud scores, and approved templates.

### Layer 6: Memory And State

Durable state should include case status, route, evidence gaps, analyst assignment, prior outputs, manual overrides, and final triage disposition.

### Layer 7: Decision And Approval Plane

This layer enforces dispute handling rules, high-risk escalation, restricted account action gates, and review steps for regulated or suspicious cases.

### Layer 8: Governance And Control Plane

This layer governs access to banking data, routing rules, procedure versioning, audit logging, and human approval constraints.

### Layer 9: Evaluation And Observability

This layer tracks both skill quality and banking outcomes such as triage speed, false-positive rates, case aging, routing accuracy, and escalation misses.

## Governance Model For Banking

### Key Governance Rules

1. Core banking, fraud, and card systems remain the source of truth for account, transaction, and case state.
2. AI may prepare and recommend, but no skill may block accounts, reverse transactions, or authorize provisional credit.
3. Fraud, AML, and suspicious-activity indicators must never be auto-cleared.
4. Every write action must preserve actor, timestamp, prior value, and case linkage.
5. Customer-facing drafts must avoid commitment language beyond approved procedural updates.
6. High-risk, repeat-pattern, or regulated cases must always route through explicit human review paths.

### Ownership Model

| Asset | Owner |
| --- | --- |
| Process definition | fraud operations manager |
| Triage and routing rules | fraud operations and dispute operations leadership |
| Skill contracts and orchestration logic | banking operations automation team |
| Prompt and template assets | joint ownership between operations and technical owner |
| Data access model | banking data owner and enterprise identity team |
| Evaluation benchmarks | fraud and dispute operations with quality and audit support |

## Evaluation Plan

### Offline Evaluation Set

Build a benchmark set that includes:

- clearly fraudulent card transactions
- customer disputes with incomplete intake data
- repeated merchant alerts on same customer
- low-severity false-positive alerts
- high-risk cases involving cross-border or unusual channel activity
- disputes requiring affidavit or missing-evidence follow-up
- cases that should escalate to another control lane

### Acceptance Thresholds For Pilot

| Measure | Target |
| --- | --- |
| Fraud/dispute classification accuracy | at least 85 percent |
| Evidence gap detection rate | at least 85 percent |
| Incorrect routing rate | below 5 percent |
| Unauthorized account actions | zero |
| Missing audit fields | zero |
| Customer draft acceptance rate | at least 75 percent with minor edits only |

### Pilot Design

1. Start with one fraud or dispute queue and one product set.
2. Use AI assist and draft-plus-approve modes first.
3. Keep final triage, account action, and escalation approvals under human ownership.
4. Review false negatives, routing errors, and communication quality weekly.
5. Expand only after stable analyst trust and strong audit performance.

## Implementation Roadmap For Banking

### Wave 1

- normalize fraud and dispute intake events
- assemble transaction and account context
- add scenario and dispute classification assist
- add evidence gap and urgency preparation

### Wave 2

- add routing recommendation to fraud, disputes, or escalation lanes
- add customer and analyst communication drafting
- add procedure-cited case packet summaries

### Wave 3

- add bounded multi-document case packet assembly for complex or repeat-pattern cases
- add proactive detection of repeated fraud patterns and recurring intake deficiencies

## Summary

For a typical US banking enterprise, fraud alert and dispute triage is a strong first application of the Workplace AI Process Decomposition Framework because it is high-volume, operationally important, and highly improvable without delegating account action or regulated decision authority to the model.
