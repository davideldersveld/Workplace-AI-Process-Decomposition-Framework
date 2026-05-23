# Insurance Industry Sample

## Purpose

This document shows what applying the Workplace AI Process Decomposition Framework looks like for a typical Insurance enterprise in the United States. It is intended to demonstrate how an insurance organization can move from a high-volume operational workflow to a governed AI-enabled operating design.

The sample uses a realistic insurer environment with core policy systems, claims platforms, strict regulatory controls, evidence-heavy workflows, and multiple approval gates.

## Insurance Context

A large US-based insurance enterprise typically supports functions such as:

- policy administration
- first notice of loss and claims intake
- claims coverage triage
- fraud and SIU review
- underwriting exception handling
- producer and broker servicing
- subrogation and recovery support
- regulatory and consumer complaint handling

These workflows combine large document volumes, customer interactions, policy interpretation, claim evidence, and time-sensitive routing. That makes Insurance a strong fit for process-centric AI when coverage judgment, payment authority, and regulatory decisions remain explicitly human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| FNOL and coverage triage | High | High | High | Medium | Strong | Start here |
| Underwriting exception review | Medium | High | Medium | Low | Strong | Good candidate |
| Claims document deficiency handling | High | Medium | High | Medium | Strong | Good candidate |
| Fraud referral preparation | Medium | High | Medium | Low | Strong | Later wave |
| Subrogation case packet assembly | Medium | Medium | Medium | Medium | Moderate | Second wave |
| Regulatory complaint triage | Medium | High | Medium | Low | Strong | Later wave |

## Selected Pilot Process

This sample focuses on FNOL and coverage triage for personal and commercial property and casualty claims.

### Why This Process Is A Good First Target

- It is high-volume and repetitive.
- It combines structured and unstructured inputs.
- It has clear business metrics such as cycle time, claim aging, and coverage decision turnaround.
- It has explicit evidence requirements and routing rules.
- Most AI value sits in preparation, extraction, and routing rather than final claim adjudication.
- The workflow benefits from early issue identification without handing coverage authority to the model.

## Business Objective

Reduce time to a review-ready claim triage packet while preserving policy interpretation quality, claimant fairness, fraud controls, and regulatory compliance.

## Scope

Included in scope:

- first notice of loss intake normalization
- policy and loss context assembly
- initial claim type and severity classification
- missing evidence detection
- initial coverage-path recommendation
- routing to adjuster, catastrophe desk, or SIU
- claimant and agent communication drafting

Excluded from scope:

- final coverage determination
- claim payment authorization
- reserve setting approval
- denial letter issuance without reviewer approval
- formal fraud determination

## Process Definition

| Field | Value |
| --- | --- |
| Process name | FNOL and coverage triage |
| Business objective | Convert loss notifications into complete, review-ready claim triage packets |
| Trigger | New loss report received by phone, portal, email, broker, or agent |
| Primary outcome | Claim case is normalized, evidence-backed, correctly routed, and ready for adjuster review |
| Process owner | claims operations manager |
| Technical owner | insurance platform automation lead |
| Primary systems | claims platform, policy administration system, document repository, CRM, telephony transcripts, email, collaboration platform |
| Primary roles | intake specialist, claims adjuster, claim supervisor, SIU analyst, agent service representative |
| SLA target | Standard FNOL cases triaged within 4 business hours |
| Main risk domains | incorrect policy interpretation, missed fraud indicators, inadequate evidence capture, claimant miscommunication, unfair claims practice risk |

## Current-State Workflow Summary

In a typical insurer, loss notifications arrive through multiple channels and often contain incomplete facts. Intake teams gather the claimant story, locate the policy, identify the loss type, collect photos and repair documents, review prior claims, and determine whether the case should go to a standard adjuster, catastrophe workflow, bodily injury team, or SIU.

The work is valuable but frequently slowed by fragmented evidence, repeated follow-up requests, inconsistent initial summaries, and manual cross-checking across policy, claim, and document systems.

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| INS-CLM-001 | Intake | Normalize FNOL event | Create a usable claim case from inbound loss signals | completeness check | claims intake record | claims intake operations | normalized claim case | No |
| INS-CLM-002 | Context | Gather policy, claimant, and loss context | Assemble policy, insured, prior claim, and loss details | context sufficiency | policy and claims systems | intake specialist | claim context packet | No |
| INS-CLM-003 | Triage | Classify claim type and severity | Determine dominant claim type, severity, and handling lane | category assignment | claims taxonomy and policy data | intake specialist | claim type, severity, confidence | No |
| INS-CLM-004 | Coverage Prep | Assess initial coverage path and missing evidence | Identify obvious coverage path and evidence deficiencies | coverage-path recommendation | policy language and evidence checklist | intake specialist | evidence gaps, coverage-path recommendation | No |
| INS-CLM-005 | Routing | Route to adjuster, catastrophe desk, or SIU | Send claim to the correct handling team | owner selection | routing rules and fraud triggers | claims operations | routed claim | Sometimes |
| INS-CLM-006 | Communication | Draft claimant, broker, and adjuster communications | Prepare outreach, deficiency requests, or internal summary | content generation | case packet and templates | claims operations | draft messages and adjuster summary | No |
| INS-CLM-007 | Closure | Confirm triage disposition | Confirm claim is ready, escalated, or returned for more intake | disposition decision | claim workflow status | claim supervisor | final triage state | Yes for escalations |

## Example Step Record

### INS-CLM-004: Assess Initial Coverage Path And Missing Evidence

```yaml
step_id: INS-CLM-004
step_name: Assess initial coverage path and missing evidence
stage: Coverage Prep
goal: Identify obvious coverage path and evidence gaps before adjuster assignment
trigger: Claim context packet completed
inputs:
  - policy summary
  - loss narrative
  - photo and document inventory
  - prior claim history
  - claim evidence checklist
source_of_truth:
  - policy contract and endorsements
  - intake evidence checklist
decision_type: coverage-path recommendation
systems:
  - claims platform
  - policy administration system
  - document repository
human_roles:
  - intake specialist
outputs:
  - evidence_gap_list
  - suggested_handling_lane
  - fraud_indicator_flags
approvals_required: false
exceptions:
  - policy not active on date of loss
  - insufficient loss details
  - suspicious prior claim pattern
owner: claims intake operations
success_metrics:
  - evidence_gap_detection_rate
  - triage_cycle_time
  - adjuster_rework_rate
```

## Signal Inventory

### Human Signals

| Signal Type | Insurance Example | Why It Matters |
| --- | --- | --- |
| Email | broker notice, claimant follow-up, vendor estimate submission | Contains narrative context, urgency, and supporting artifacts |
| Chat | internal claim coordination, catastrophe team updates | Reveals blockers, escalation, and changing severity |
| Files | photos, police reports, repair estimates, medical documents, proof of loss | Provide the primary evidence base for claim handling |
| Comments | adjuster notes, supervisor annotations, SIU remarks | Explain why routing or escalation changed |
| Edits | corrected date of loss, revised claimant contact info, reclassified loss type | Show intake instability and evidence churn |
| Approvals | supervisor escalation sign-off, SIU referral approval | Define accountable review points |
| Calls and transcripts | FNOL call notes and recorded claimant statements | Provide the richest source of initial loss narrative |

### System Signals

| Signal Type | Insurance Example | Why It Matters |
| --- | --- | --- |
| Record state | claim status, policy status, assignment state | Anchors the workflow to durable system state |
| Events | claim opened, photo uploaded, catastrophe flag applied | Trigger progression and re-evaluation |
| Master data | policyholder profile, product line, coverage limits, adjuster territories | Constrains routing and initial coverage path |
| History | prior claims, prior SIU referrals, reopen history | Supports pattern detection and triage quality |
| Entitlements | who may view medical details, payment info, or SIU cases | Prevents unauthorized access |
| Telemetry | claim volumes, catastrophe event zones, repeated provider submissions | Supports prioritization and anomaly detection |

### Model Knowledge

| Knowledge Type | Insurance Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize claimant narratives, rewrite deficiency requests, explain next steps | Good for interpretation and drafting |
| Enterprise retrieval | claim handling guidelines, policy summaries, evidence checklists, fraud indicators | Required for grounded insurance outputs |
| Procedural knowledge | FNOL rubric, routing matrix, SIU triggers, catastrophe handling rules | Must be owned by claims operations |
| Exemplars | approved adjuster summaries and claimant outreach examples | Improve consistency and speed |
| Structured business data | policy details, claim state, loss history, claim severity rules | Required for accurate routing and triage |

### Signal Quality Notes

Insurance teams should explicitly score signal sources on:

- Freshness, because active loss conditions and claim facts change quickly.
- Reliability, because claimant narrative and system facts may conflict.
- Permission sensitivity, because claims often include medical, financial, or protected personal data.
- Source-of-truth status, because policy and claim systems must outrank informal conversation.
- Audit requirement, because every triage decision may later be reviewed in a claim file audit or dispute.

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| INS-CLM-001 | Normalize FNOL event | Deterministic automation | Stable intake normalization and metadata capture |
| INS-CLM-002 | Gather policy, claimant, and loss context | AI act within policy | Safe to retrieve approved context and assemble the packet |
| INS-CLM-003 | Classify claim type and severity | AI assist | Valuable triage aid, but initial claim categorization should remain visible to humans |
| INS-CLM-004 | Assess initial coverage path and missing evidence | AI assist | Useful for preparation, but not a substitute for coverage judgment |
| INS-CLM-005 | Route to adjuster, catastrophe desk, or SIU | AI draft plus approve | Routing recommendation is useful but must be reviewable |
| INS-CLM-006 | Draft claimant, broker, and adjuster communications | AI draft plus approve | High drafting value with mandatory review |
| INS-CLM-007 | Confirm triage disposition | Human only for pilot | Final triage and escalation decision should remain human-owned |

## AI Task Pattern Mapping

| Step ID | Dominant Task Pattern | Secondary Patterns |
| --- | --- | --- |
| INS-CLM-001 | Extract | Validate |
| INS-CLM-002 | Retrieve | Summarize |
| INS-CLM-003 | Classify | Compare |
| INS-CLM-004 | Compare | Recommend |
| INS-CLM-005 | Route | Recommend |
| INS-CLM-006 | Generate | Summarize |
| INS-CLM-007 | Decide | Escalate |

## Translation To Technical Artifacts

### Skills

| Skill | Purpose | Inputs | Outputs |
| --- | --- | --- | --- |
| Normalize FNOL case | Convert inbound loss notification into a structured claim case | claimant notice, email, transcript, forms | normalized claim case |
| Build claim context packet | Assemble policy, claimant, prior claim, and evidence context | claim ID, policy ID, claimant ID | context packet |
| Classify claim type | Recommend claim type and initial severity | context packet | claim type, severity, confidence |
| Detect evidence gaps | Identify missing documents and fraud indicators | context packet, evidence checklist | gap list, risk flags |
| Recommend routing | Suggest adjuster, catastrophe, bodily injury, or SIU path | case packet, routing rules | routing recommendation |
| Draft claim communications | Produce claimant, agent, and internal adjuster drafts | case packet, templates | draft messages and summary |

### Tools And Plugins

| Tool Or Plugin | Action Type | System |
| --- | --- | --- |
| Get claim record | Read | claims platform |
| Get policy summary | Read | policy administration system |
| Get prior claim history | Read | claims platform |
| Get catastrophe event context | Read | catastrophe or weather event feed |
| Get evidence checklist | Read | policy or claim operations repository |
| Update claim triage status | Write | claims workflow or tracker |
| Create adjuster review task | Write | workflow platform |
| Log audit event | Write | audit or workflow store |

### Workflow

The primary workflow should coordinate:

1. FNOL intake normalization.
2. Policy and claim context retrieval.
3. Claim type and severity classification.
4. Evidence gap and initial coverage-path preparation.
5. Routing to the correct handling lane.
6. Draft communication generation.
7. Triage disposition confirmation.

### Agent Recommendation

For the first insurance implementation, do not make a broad claims agent the center of the runtime. Use workflows, skills, and tools with explicit adjuster, SIU, and supervisor review gates. Introduce an agent only later for bounded multi-document case assembly in complex claims.

## Insurance Reference Architecture

### Layer 1: Signal Intake And Normalization

Inputs arrive from phone transcripts, web forms, email, broker submissions, and claim events. The intake layer converts them into a normalized claim event.

### Layer 2: Process Model

The process model resolves whether the claim is in intake, triage, evidence collection, routing, or disposition.

### Layer 3: Capability Registry

This layer stores skills, tools, templates, claim taxonomies, routing rules, fraud triggers, and policy references.

### Layer 4: Runtime Orchestrator

The orchestrator receives the claim event, resolves current state, gathers context, selects the allowed execution unit, and records the result.

### Layer 5: Knowledge And Context Builder

The context builder assembles policy details, endorsements, claim history, catastrophe indicators, evidence checklist items, and prior handling examples.

### Layer 6: Memory And State

Durable state should include claim status, routing state, evidence gaps, adjuster assignment, prior outputs, manual overrides, and final triage disposition.

### Layer 7: Decision And Approval Plane

This layer enforces fraud referral triggers, catastrophe routing, supervisor approvals, and review gates for high-severity or high-value claims.

### Layer 8: Governance And Control Plane

This layer governs access to claim data, policy versioning, routing rules, audit logging, and human approval constraints.

### Layer 9: Evaluation And Observability

This layer tracks both task quality and claim-handling outcomes such as triage time, evidence completeness, routing accuracy, and adjuster rework.

## Governance Model For Insurance

### Key Governance Rules

1. The policy and claims platforms remain the source of truth for policy status, coverage context, and claim state.
2. AI may prepare and recommend, but final coverage interpretation and claim handling authority remain human-owned.
3. No skill may authorize payment, denial, or reserve changes.
4. Every write action must preserve actor, timestamp, prior value, and claim linkage.
5. Fraud or SIU indicators must never be auto-cleared or hidden.
6. Customer-facing drafts must avoid commitment language unless approved by the assigned human reviewer.

### Ownership Model

| Asset | Owner |
| --- | --- |
| Process definition | claims operations manager |
| Coverage-path and routing rules | claims operations with claims leadership |
| Skill contracts and orchestration logic | insurance platform automation team |
| Prompt and template assets | joint ownership between claims operations and technical owner |
| Data access model | insurance data owner and enterprise identity team |
| Evaluation benchmarks | claims operations with quality and audit support |

## Evaluation Plan

### Offline Evaluation Set

Build a benchmark set that includes:

- straightforward property losses with complete evidence
- incomplete FNOL notices missing critical facts
- suspicious repeated loss patterns
- catastrophe-driven claims requiring alternate routing
- bodily injury or high-severity claims needing escalation
- policy-inactive or endorsement-sensitive claims
- claims with conflicting claimant narrative and system facts

### Acceptance Thresholds For Pilot

| Measure | Target |
| --- | --- |
| Claim type classification accuracy | at least 85 percent |
| Evidence gap detection rate | at least 85 percent |
| Incorrect routing rate | below 5 percent |
| Unauthorized claim actions | zero |
| Missing audit fields | zero |
| Claimant draft acceptance rate | at least 75 percent with minor edits only |

### Pilot Design

1. Start with one product line and one claim intake team.
2. Use AI assist and draft-plus-approve modes first.
3. Keep final triage and handling lane approval under supervisor or adjuster ownership.
4. Review fraud indicators, routing errors, and communication quality weekly.
5. Expand only after stable evidence completeness and routing quality.

## Implementation Roadmap For Insurance

### Wave 1

- normalize FNOL intake events
- assemble claim and policy context
- add claim type and severity assist
- add missing evidence and fraud-indicator detection

### Wave 2

- add routing recommendation to adjuster, catastrophe desk, or SIU
- add claimant, broker, and adjuster draft communications
- add policy-cited claim packet summaries

### Wave 3

- add bounded multi-document claim packet assembly for complex losses
- add proactive detection of repeated suspicious patterns and recurring evidence gaps

## Summary

For a typical US insurance enterprise, FNOL and coverage triage is a strong first application of the Workplace AI Process Decomposition Framework because it is high-volume, evidence-heavy, and operationally measurable while still allowing clear human control over coverage judgment and claim authority.
