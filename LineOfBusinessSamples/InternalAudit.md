# Internal Audit Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Internal Audit function at a large US-based enterprise. It demonstrates how an internal audit organization can convert audit request and evidence collection into a governed AI-enabled workflow.

## Internal Audit Context

A large enterprise internal audit organization often supports:

- audit request intake
- control walkthrough preparation
- evidence request coordination
- issue validation support
- testing packet preparation
- stakeholder review and follow-up
- audit trail and workpaper support

These workflows are evidence-heavy, process-oriented, and review-bound. They are a strong fit for bounded AI assistance when audit judgment, conclusions, and sign-off remain explicitly human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Audit request intake and evidence collection prep | High | High | High | Low | Strong | Start here |
| Control walkthrough summary preparation | Medium | High | Medium | Medium | Strong | Good candidate |
| Testing packet assembly | Medium | High | Medium | Low | Strong | Good candidate |
| Issue validation and follow-up routing | Medium | High | Medium | Low | Strong | Second wave |
| Workpaper summary generation | High | Medium | Medium | Medium | Moderate | Later wave |

## Selected Pilot Process

This sample focuses on audit request intake and evidence collection prep.

## Business Objective

Reduce time to a review-ready audit packet while preserving evidence integrity, control traceability, review quality, and audit independence.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Audit request intake and evidence collection prep |
| Business objective | Convert audit requests into complete, evidence-backed, review-ready audit cases |
| Trigger | Audit request initiated, walkthrough scheduled, or evidence request case opened |
| Primary outcome | Audit case is normalized, scoped, evidence-routed, and ready for auditor review |
| Process owner | internal audit operations manager |
| Technical owner | audit workflow or GRC automation lead |
| Primary systems | audit management platform, GRC platform, document repository, workflow system, collaboration platform, email |
| Primary roles | auditor, audit manager, control owner liaison, business process owner, audit operations coordinator |
| SLA target | Standard audit requests prepared for evidence follow-up within 2 business days |
| Main risk domains | incomplete evidence, weak traceability, premature conclusions, approval bypass, inadequate workpaper support |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IA-001 | Intake | Normalize audit request | Create a usable audit case from intake signals | completeness check | audit management record | audit operations | normalized audit case | No |
| IA-002 | Context | Gather scope, control, and prior-evidence context | Assemble objective, control ownership, prior findings, and evidence requirements | context sufficiency | audit platform and control repository | auditor | audit packet | No |
| IA-003 | Review Prep | Identify missing evidence and scope gaps | Detect missing artifacts, unclear ownership, and traceability gaps | gap detection | audit packet and evidence checklist | auditor | evidence gap list, open questions | No |
| IA-004 | Routing | Determine evidence-request and reviewer path | Route requests to the right control owners and review chain | reviewer routing | reviewer matrix and scope rules | auditor | routed requests and review path | Yes for some cases |
| IA-005 | Communication | Draft audit summary and evidence requests | Prepare reviewer summary and evidence follow-up drafts | content generation | audit packet | auditor | summary draft, evidence request drafts | No |
| IA-006 | Closure | Confirm audit packet disposition | Confirm packet is review-ready, escalated, or returned for more scope clarification | disposition decision | workflow state record | audit manager | final prep state | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Internal Audit Example | Why It Matters |
| --- | --- | --- |
| Email | audit scheduling, evidence follow-up, reviewer comments | Captures intent, ownership, and timing |
| Chat | coordination with control owners and audit team | Reveals blockers and handoff issues |
| Files | process narratives, screenshots, reports, control evidence, workpapers | Contain the primary audit evidence |
| Comments | auditor notes, manager review comments, owner explanations | Explain scope and evidence quality judgments |
| Edits | revised evidence requests, scope changes, updated ownership | Show churn and potential traceability issues |
| Approvals | manager review, scope sign-off, issue escalation | Define controlled review points |

### System Signals

| Signal Type | Internal Audit Example | Why It Matters |
| --- | --- | --- |
| Record state | audit request status, evidence status, review state | Anchors the workflow |
| Events | request opened, evidence uploaded, manager review requested | Trigger progression |
| Master data | control owner, process owner, scope definition, entity | Constrains routing and ownership |
| History | prior findings, prior evidence requests, prior walkthroughs | Supports better scoping and follow-up |
| Entitlements | who may access or approve workpapers and evidence | Prevents unauthorized changes |
| Telemetry | aging, pending evidence, review cycles | Supports process improvement |

### Model Knowledge

| Knowledge Type | Internal Audit Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize requests, rewrite evidence follow-ups, structure packets | Good for synthesis and drafting |
| Enterprise retrieval | audit methodology, evidence checklist, control library, prior findings | Required for grounded outputs |
| Procedural knowledge | review checklist, routing rules, workpaper standards | Must be owned by Internal Audit |
| Exemplars | approved evidence requests and strong audit summaries | Improve consistency |
| Structured business data | case metadata, control ownership, prior issue records | Required for correct routing and traceability |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| IA-001 | Normalize audit request | Deterministic automation | Stable intake normalization |
| IA-002 | Gather scope, control, and prior-evidence context | AI act within policy | Safe to retrieve approved context |
| IA-003 | Identify missing evidence and scope gaps | AI assist | Valuable review aid, but audit judgment must remain visible |
| IA-004 | Determine evidence-request and reviewer path | AI draft plus approve | Recommendation is useful but must remain reviewable |
| IA-005 | Draft audit summary and evidence requests | AI draft plus approve | Strong drafting value with required review |
| IA-006 | Confirm audit packet disposition | Human only for pilot | Final audit readiness and review decisions remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize audit intake
- build audit evidence packet
- detect evidence gaps and scope issues
- recommend reviewer and owner path
- draft audit summary
- draft evidence requests

### Tools And Plugins

- get_audit_request
- get_control_and_scope_context
- get_prior_findings
- get_reviewer_matrix
- update_audit_status
- create_evidence_request_task
- log_audit_event

## Governance Model

1. The audit management or GRC platform remains the source of truth for audit case state.
2. AI may summarize and recommend, but audit conclusions, scope judgments, and sign-offs remain human responsibilities.
3. Evidence and workpapers must remain permission-scoped and traceable.
4. Every write action must preserve actor, timestamp, prior value, and case linkage.

## Evaluation Plan

### Pilot Metrics

- evidence-gap detection rate
- incorrect reviewer-routing rate
- draft acceptance rate
- time to review-ready audit packet
- traceability coverage rate
- override rate

## Implementation Roadmap

### Wave 1

- normalize audit requests
- assemble scope and evidence context
- add evidence-gap detection assist
- add audit summary and evidence-request drafting

### Wave 2

- add reviewer and owner routing recommendation
- add controlled request creation
- add methodology-cited packet summaries

### Wave 3

- add bounded multi-source evidence packet assembly
- add proactive detection of likely missing control owners or recurring evidence gaps

## Summary

For a typical enterprise Internal Audit organization, audit request intake and evidence collection prep is a strong first application of the Process Decomposition Framework because it is evidence-heavy, traceability-sensitive, and improved materially by better packet assembly and controlled review routing.
