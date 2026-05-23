# Compliance and Risk Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Compliance and Risk function at a large US-based enterprise. It demonstrates how a compliance organization can convert policy exception and compliance case intake into a governed AI-enabled workflow.

## Compliance and Risk Context

A large enterprise compliance and risk function often supports:

- policy exception intake
- compliance case triage
- control issue review support
- regulatory inquiry coordination
- attestations and evidence gathering
- third-party risk request handling
- escalation and review routing

These workflows are evidence-heavy, risk-sensitive, and approval-bound. They are a strong fit for bounded AI assistance when risk judgments, policy interpretation, and formal approvals remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Policy exception and compliance case triage | High | High | High | Low | Strong | Start here |
| Control issue evidence preparation | Medium | High | High | Low | Strong | Good candidate |
| Third-party compliance request review | Medium | High | Medium | Low | Strong | Good candidate |
| Regulatory inquiry packet assembly | Medium | High | Medium | Low | Strong | Later wave |
| Attestation follow-up coordination | High | Medium | High | Medium | Moderate | Second wave |

## Selected Pilot Process

This sample focuses on policy exception and compliance case triage.

## Business Objective

Reduce time to a review-ready compliance case while preserving policy interpretation quality, evidence integrity, approval discipline, and auditability.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Policy exception and compliance case triage |
| Business objective | Convert a policy exception request or suspected compliance issue into a complete, routed, evidence-backed case |
| Trigger | Policy exception request submitted, hotline or case intake received, or control issue opened |
| Primary outcome | Case is normalized, risk-indicated, routed to the correct reviewers, and prepared for decision |
| Process owner | compliance operations manager |
| Technical owner | GRC platform automation lead |
| Primary systems | GRC platform, policy repository, case management platform, document repository, email, collaboration platform |
| Primary roles | compliance analyst, risk analyst, business control owner, legal reviewer, internal audit liaison |
| SLA target | Standard compliance cases triaged within 1 business day |
| Main risk domains | misclassified risk, missing evidence, approval bypass, policy misinterpretation, weak audit trail |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CR-001 | Intake | Normalize compliance case | Create a usable case from intake signals | completeness check | case intake record | compliance operations | normalized case | No |
| CR-002 | Context | Gather policy, control, and entity context | Assemble policy, control ownership, entity, and historical context | context sufficiency | GRC and policy records | compliance analyst | case context packet | No |
| CR-003 | Analysis | Identify risk indicators and missing evidence | Detect probable risk factors, missing support, and routing needs | risk indication | policy and case packet | compliance analyst | risk flags, evidence gaps | No |
| CR-004 | Routing | Determine review path and severity | Route to the right compliance, legal, audit, or control owners | control path decision | approval matrix and policy routing rules | compliance analyst | review path, severity level | Yes for some cases |
| CR-005 | Communication | Draft case summary and follow-up requests | Prepare reviewer summary and requests for missing evidence | content generation | case packet | compliance analyst | case summary, outreach draft | No |
| CR-006 | Closure | Confirm triage disposition | Confirm case is queued, escalated, or returned for more information | disposition decision | case status record | compliance manager | final triage state | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Compliance and Risk Example | Why It Matters |
| --- | --- | --- |
| Email | policy exception requests, reviewer comments, evidence follow-up | Captures context and declared rationale |
| Chat | analyst coordination, escalation discussion | Reveals urgency and hidden dependencies |
| Files | policies, screenshots, evidence files, exception forms, control documents | Contain the primary review evidence |
| Comments | reviewer notes, control owner remarks | Explain why risk or disposition changed |
| Edits | issue reclassification, severity changes, scope updates | Show churn and governance risk |
| Approvals | exception sign-off, control owner approval | Define accountable decision points |
| Hotline or intake narratives | reported concerns or suspected issues | Provide the starting context for case analysis |

### System Signals

| Signal Type | Compliance and Risk Example | Why It Matters |
| --- | --- | --- |
| Record state | case status, review queue, issue severity | Anchors the workflow |
| Events | case opened, evidence added, approval requested | Trigger progression |
| Master data | policy owner, control owner, legal entity, business unit | Constrains review path |
| History | prior exceptions, prior findings, past remediations | Supports risk awareness |
| Entitlements | who may view, update, or approve case data | Prevents unauthorized access |
| Telemetry | backlog, aging, repeat issue themes | Supports operational improvement |

### Model Knowledge

| Knowledge Type | Compliance and Risk Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize cases, rewrite follow-ups, explain next steps | Good for drafting and interpretation |
| Enterprise retrieval | policies, control standards, prior approved exceptions, playbooks | Required for grounded outputs |
| Procedural knowledge | triage rubric, severity matrix, routing rules, evidence checklist | Must be owned by Compliance |
| Exemplars | approved case summaries and evidence requests | Improve consistency |
| Structured business data | case metadata, entity data, control ownership | Required for correct routing and audit |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| CR-001 | Normalize compliance case | Deterministic automation | Stable intake normalization |
| CR-002 | Gather policy, control, and entity context | AI act within policy | Safe to retrieve approved context |
| CR-003 | Identify risk indicators and missing evidence | AI assist | Valuable analysis aid, but risk judgment should remain visible |
| CR-004 | Determine review path and severity | AI draft plus approve | Recommendation is useful but must be reviewable |
| CR-005 | Draft case summary and follow-up requests | AI draft plus approve | Strong summarization value with required review |
| CR-006 | Confirm triage disposition | Human only for pilot | Final case disposition should remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize compliance intake
- build policy and control context packet
- identify risk indicators and evidence gaps
- recommend review path and severity
- draft case summary
- draft evidence follow-up outreach

### Tools And Plugins

- get_compliance_case
- get_policy_artifacts
- get_control_owner_data
- get_case_history
- update_case_status
- create_review_task
- log_audit_event

## Governance Model

1. The GRC or compliance platform remains the source of truth for case state.
2. AI may summarize and recommend, but formal risk judgments, policy interpretation, and approvals remain human responsibilities.
3. Sensitive or privileged case materials must remain permission-scoped.
4. Every write action must preserve actor, timestamp, prior value, and case linkage.
5. Prompt, severity matrix, and routing rule changes should follow compliance change control.

## Evaluation Plan

### Pilot Metrics

- risk indicator precision and recall
- incorrect routing rate
- evidence gap detection rate
- triage cycle time
- missing audit-field rate
- unauthorized access incidents

## Implementation Roadmap

### Wave 1

- normalize compliance case intake
- assemble policy and control context
- add risk-indicator assist
- add case-summary and evidence-request drafting

### Wave 2

- add severity and routing recommendation
- add policy-cited case summaries
- add controlled review-task creation

### Wave 3

- add bounded multi-source evidence packet assembly
- add proactive detection of repeated policy-exception patterns

## Summary

For a typical enterprise Compliance and Risk organization, policy exception and compliance case triage is a strong first application of the Process Decomposition Framework because it is evidence-heavy, approval-bound, and well suited to assistive AI with explicit human accountability.
