# Procurement Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Procurement function at a large US-based enterprise. It demonstrates how procurement teams can convert supplier onboarding and risk review into a controlled AI-enabled workflow.

## Procurement Context

A large enterprise procurement organization often supports:

- supplier onboarding
- sourcing event preparation
- purchase requisition triage
- contract intake coordination
- risk and due diligence routing
- catalog and non-catalog buying support
- approval packet preparation

These workflows combine structured vendor data, document-heavy review, control requirements, and multiple approval points. That makes Procurement a strong fit for bounded AI assistance when supplier risk and approval authority remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Supplier onboarding and risk review | High | High | High | Medium | Strong | Start here |
| Purchase requisition triage | High | High | High | Medium | Strong | Good candidate |
| Non-catalog request handling | Medium | High | Medium | Medium | Strong | Good candidate |
| Sourcing event package assembly | Medium | Medium | Medium | Medium | Moderate | Later wave |
| PO change request review | Medium | Medium | High | Low | Strong | Second wave |

## Selected Pilot Process

This sample focuses on supplier onboarding and risk review.

## Business Objective

Reduce time to a review-ready supplier onboarding packet while preserving risk controls, approval integrity, and complete audit evidence.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Supplier onboarding and risk review |
| Business objective | Convert a new supplier request into a complete, risk-aware, approval-ready onboarding case |
| Trigger | New supplier request submitted or supplier onboarding case created |
| Primary outcome | Supplier onboarding packet is complete, risk-routed, reviewed, and ready for approval |
| Process owner | procurement operations manager |
| Technical owner | source-to-pay automation lead |
| Primary systems | vendor master, sourcing platform, risk screening tools, document repository, workflow system, email |
| Primary roles | procurement analyst, requester, category manager, risk reviewer, legal reviewer, AP onboarding specialist |
| SLA target | Standard onboarding packet prepared within 3 business days |
| Main risk domains | missing due diligence, sanctions or compliance risk, duplicate supplier, approval bypass, bank detail sensitivity |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PR-001 | Intake | Normalize supplier request | Create a usable onboarding case | completeness check | supplier request record | procurement operations | normalized supplier case | No |
| PR-002 | Context | Gather supplier and policy context | Assemble supplier, category, geography, and control context | context sufficiency | vendor master and policy repository | procurement operations | onboarding packet | No |
| PR-003 | Triage | Detect missing items and risk indicators | Identify documentation gaps and risk flags | gap and risk detection | onboarding checklist and screening tools | procurement analyst | missing items, risk flags | No |
| PR-004 | Routing | Determine review path | Route to the required risk, legal, tax, or AP reviewers | control path decision | approval matrix and risk policy | procurement analyst | review path | Yes for some paths |
| PR-005 | Communication | Draft outreach and evidence summary | Prepare requests for missing information or reviewer context | content generation | onboarding packet | procurement analyst | draft outreach, reviewer summary | No |
| PR-006 | Closure | Confirm onboarding disposition | Confirm supplier is approved, rejected, or returned for more information | disposition decision | workflow status record | procurement manager | final onboarding state | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Procurement Example | Why It Matters |
| --- | --- | --- |
| Email | supplier follow-up, requester clarifications | Contains missing context and urgency |
| Chat | coordination among procurement, AP, and legal | Reveals hidden blockers and dependencies |
| Files | W-9, banking forms, insurance certificates, contracts | Often contain mandatory onboarding evidence |
| Comments | analyst notes, reviewer remarks | Explain why routing or escalation happened |
| Edits | supplier name corrections, category changes | Show identity and control risk |
| Approvals | category approval, risk sign-off | Define accountable decision points |

### System Signals

| Signal Type | Procurement Example | Why It Matters |
| --- | --- | --- |
| Record state | onboarding status, review queue, hold state | Anchors the workflow |
| Events | request submitted, screening completed, document uploaded | Trigger progression |
| Master data | supplier identifiers, spend category, geography | Constrains policy path |
| History | prior requests, duplicate supplier attempts | Supports risk review |
| Entitlements | who can approve or edit supplier records | Prevents unauthorized changes |
| Telemetry | cycle time, backlog, common missing documents | Supports process improvement |

### Model Knowledge

| Knowledge Type | Procurement Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize case, draft outreach, explain next steps | Safe for interpretation and drafting |
| Enterprise retrieval | onboarding checklist, risk policy, supplier standards | Required for grounded outputs |
| Procedural knowledge | routing rubric, reviewer matrix, template set | Must be owned by Procurement |
| Exemplars | approved reviewer summaries and outreach | Improve consistency |
| Structured business data | supplier profile, status, screening outputs | Required for accurate routing and action |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| PR-001 | Normalize supplier request | Deterministic automation | Stable event normalization |
| PR-002 | Gather supplier and policy context | AI act within policy | Safe to retrieve approved context |
| PR-003 | Detect missing items and risk indicators | AI assist | Valuable for checklist review, but should remain visible |
| PR-004 | Determine review path | AI draft plus approve | Routing recommendation is useful but must be reviewable |
| PR-005 | Draft outreach and evidence summary | AI draft plus approve | Strong drafting value with low execution risk |
| PR-006 | Confirm onboarding disposition | Human only for pilot | Final supplier approval should remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize supplier onboarding case
- build supplier context packet
- detect missing documents and risk flags
- recommend review path
- draft supplier outreach
- summarize reviewer packet

### Tools And Plugins

- get_supplier_request
- get_vendor_master_record
- get_risk_screening_results
- get_policy_and_checklist
- update_onboarding_status
- create_review_task
- log_audit_event

## Governance Model

1. The vendor master and workflow platform remain sources of truth for supplier state.
2. AI may prepare and recommend, but final supplier approval must follow named approval authority.
3. Bank detail handling and sensitive tax data require strict permission controls.
4. Every write action must preserve actor, timestamp, prior value, and case linkage.

## Evaluation Plan

### Pilot Metrics

- onboarding packet completeness rate
- incorrect review-path rate
- draft acceptance rate
- cycle time to review-ready state
- duplicate supplier detection rate
- unauthorized approval incidents

## Implementation Roadmap

### Wave 1

- normalize supplier requests
- assemble onboarding context
- add missing-item and risk assist
- add supplier and reviewer summary drafting

### Wave 2

- add review-path recommendation
- add policy-cited packet generation
- add limited status updates after approved steps

### Wave 3

- add bounded agent support for complex multi-reviewer packet assembly
- add proactive detection of likely stalled onboarding cases

## Summary

For a typical enterprise Procurement organization, supplier onboarding and risk review is a strong first application of the Process Decomposition Framework because it combines repeatable document-heavy work, explicit controls, and measurable cycle-time pain points.
