# Finance Controllership Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Finance Controllership function at a large US-based enterprise. It demonstrates how a controllership organization can convert manual journal entry requests into a governed AI-enabled review and approval workflow.

## Finance Controllership Context

A large enterprise controllership organization often supports:

- manual journal entry request handling
- close variance investigation
- account reconciliation review support
- close checklist coordination
- policy exception handling
- audit support and evidence preparation
- control review and sign-off routing

These workflows are document-heavy, control-sensitive, and approval-bound. That makes Controllership a strong fit for bounded AI assistance when accounting judgment, sign-off authority, and posting controls remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Manual journal entry request and approval | High | High | High | Low | Strong | Start here |
| Close variance investigation support | Medium | High | Medium | Medium | Strong | Good candidate |
| Reconciliation exception triage | Medium | High | Medium | Low | Strong | Good candidate |
| Close checklist summary preparation | High | Medium | High | Medium | Moderate | Second wave |
| Audit evidence packet assembly | Medium | High | High | Low | Strong | Later wave |

## Selected Pilot Process

This sample focuses on manual journal entry request and approval.

### Why This Process Is A Good First Target

- It is recurring and operationally important during close cycles.
- It has explicit policies, approvers, and evidence requirements.
- It includes structured and unstructured inputs that benefit from normalization and synthesis.
- It has clear metrics such as cycle time, approval turnaround, and rework rate.
- Most AI value is assistive and review-oriented rather than autonomous posting.

## Business Objective

Reduce the time required to prepare a review-ready journal entry packet while preserving accounting policy compliance, segregation of duties, auditability, and approver accountability.

## Scope

Included in scope:

- journal entry request intake
- support document and policy context assembly
- missing evidence detection
- risk and approval path recommendation
- draft journal entry summary generation
- reviewer packet preparation
- routing to required approvers

Excluded from scope:

- autonomous journal posting
- override of approval thresholds
- accounting policy adjudication without controller review
- final financial statement certification

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Manual journal entry request and approval |
| Business objective | Convert journal entry requests into complete, review-ready, approval-routed packets |
| Trigger | Manual journal entry request submitted or close-period adjustment initiated |
| Primary outcome | Journal entry packet is normalized, evidence-backed, correctly routed, and ready for approval |
| Process owner | controllership operations manager |
| Technical owner | finance platforms automation lead |
| Primary systems | ERP, close management system, document repository, workflow platform, email, collaboration platform |
| Primary roles | accountant, accounting manager, controller, finance systems analyst, close coordinator |
| SLA target | Standard journal entries prepared for review within 1 business day of complete intake |
| Main risk domains | unsupported entries, approval bypass, incorrect account mapping, duplicate entries, weak audit trail |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FC-001 | Intake | Normalize journal request | Create a usable journal case from intake signals | completeness check | close management or request record | accounting operations | normalized journal case | No |
| FC-002 | Context | Gather ledger, policy, and support context | Assemble account, entity, policy, support, and historical context | context sufficiency | ERP and policy repository | accountant | journal packet | No |
| FC-003 | Review Prep | Detect missing support and control risks | Identify missing evidence, risky attributes, and policy-sensitive conditions | gap and risk detection | journal packet and control checklist | accountant | missing items, risk flags | No |
| FC-004 | Routing | Determine approval path | Route to the correct accounting manager, controller, or specialist reviewer | approval path decision | approval matrix and accounting policy | accountant | routed approval path | Yes |
| FC-005 | Communication | Draft journal summary and follow-up requests | Prepare reviewer summary and missing-evidence outreach | content generation | journal packet | accountant | draft summary, outreach draft | No |
| FC-006 | Closure | Confirm review-readiness disposition | Confirm the entry is ready for approval, escalated, or returned for rework | disposition decision | workflow status record | accounting manager | final prep state | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Controllership Example | Why It Matters |
| --- | --- | --- |
| Email | request rationale, approver comments, evidence follow-up | Captures intent and missing support context |
| Chat | close coordination and escalation discussion | Reveals urgency and blockers |
| Files | journal support, schedules, reconciliations, memos | Contain the primary review evidence |
| Comments | reviewer notes, controller remarks, audit annotations | Explain why routing or rework occurred |
| Edits | amount changes, account changes, revised rationale | Show control-sensitive churn |
| Approvals | manager approval, controller sign-off | Define accountable decision points |

### System Signals

| Signal Type | Controllership Example | Why It Matters |
| --- | --- | --- |
| Record state | journal status, close status, approval state | Anchors the workflow |
| Events | request submitted, support uploaded, approval requested | Trigger progression |
| Master data | entity, account, cost center, threshold matrix | Constrains approval path |
| History | prior similar entries, reversals, prior rejections | Supports review quality |
| Entitlements | who may prepare, approve, or post | Prevents unauthorized actions |
| Telemetry | cycle time, approval lag, rework frequency | Supports process improvement |

### Model Knowledge

| Knowledge Type | Controllership Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize rationale, rewrite support requests, explain next steps | Good for synthesis and drafting |
| Enterprise retrieval | accounting policy, journal standards, approval matrix, prior approved examples | Required for grounded outputs |
| Procedural knowledge | review checklist, close rubric, routing rules | Must be owned by Controllership |
| Exemplars | approved journal summaries and strong support packets | Improve consistency |
| Structured business data | ledger context, account metadata, approval thresholds | Required for correct routing and control checks |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| FC-001 | Normalize journal request | Deterministic automation | Stable intake normalization |
| FC-002 | Gather ledger, policy, and support context | AI act within policy | Safe to retrieve approved context |
| FC-003 | Detect missing support and control risks | AI assist | Valuable review aid, but accounting judgment must remain visible |
| FC-004 | Determine approval path | AI draft plus approve | Recommendation is useful but must remain reviewable |
| FC-005 | Draft journal summary and follow-up requests | AI draft plus approve | Strong summarization value with required review |
| FC-006 | Confirm review-readiness disposition | Human only for pilot | Final readiness and approval decisions remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize journal intake
- build ledger and support packet
- detect missing evidence and control risks
- recommend approval path
- draft journal summary
- draft evidence follow-up requests

### Tools And Plugins

- get_journal_request
- get_ledger_context
- get_accounting_policy
- get_approval_matrix
- update_journal_status
- create_approval_task
- log_audit_event

## Governance Model

1. The ERP and close workflow platform remain the sources of truth for journal state and approval state.
2. AI may summarize and recommend, but journal approval and posting remain human responsibilities.
3. Every generated reviewer packet should preserve evidence links and policy references where applicable.
4. Every write action must preserve actor, timestamp, prior value, and journal linkage.
5. Prompt, checklist, and routing-rule changes should follow controllership change control.

## Evaluation Plan

### Pilot Metrics

- missing-support detection rate
- incorrect approval-path rate
- draft summary acceptance rate
- cycle time to review-ready state
- rework rate
- missing audit-field rate

## Implementation Roadmap

### Wave 1

- normalize journal requests
- assemble ledger and support context
- add missing-evidence and control-risk assist
- add journal summary drafting

### Wave 2

- add approval-path recommendation
- add controlled routing to approvers
- add policy-cited reviewer packet support

### Wave 3

- add bounded multi-source journal packet assembly for complex close entries
- add proactive detection of likely rework or duplicate-entry patterns

## Summary

For a typical enterprise Finance Controllership organization, manual journal entry request and approval is a strong first application of the Process Decomposition Framework because it is recurring, highly controlled, and well suited to assistive AI that improves packet quality without weakening accounting governance.
