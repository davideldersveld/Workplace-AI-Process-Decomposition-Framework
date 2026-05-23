# Legal Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Legal function at a large US-based enterprise. It demonstrates how a legal operations team can convert contract intake and clause deviation triage into a governed AI-enabled workflow.

## Legal Context

A large enterprise legal organization often supports:

- contract intake and triage
- NDA review
- clause deviation analysis
- legal request routing
- policy interpretation support
- redline summary preparation
- approval and escalation coordination

Legal workflows are document-heavy, risk-sensitive, and approval-bound. They are a good fit for bounded AI assistance when legal judgment, privilege, and approval authority remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Contract intake and clause deviation triage | High | High | High | Low | Strong | Start here |
| NDA review routing | High | Medium | High | Medium | Strong | Good candidate |
| Legal request intake and classification | High | Medium | High | Medium | Moderate | Good candidate |
| Policy exception review support | Medium | High | Medium | Low | Strong | Later wave |
| Obligation extraction from executed agreements | Medium | High | High | Medium | Strong | Second wave |

## Selected Pilot Process

This sample focuses on contract intake and clause deviation triage.

## Business Objective

Reduce time to a review-ready contract packet while preserving legal quality, privilege boundaries, clause policy compliance, and named approval authority.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Contract intake and clause deviation triage |
| Business objective | Convert inbound contract requests into a complete, review-ready legal case with identified deviations and required approvals |
| Trigger | Contract request submitted, third-party paper received, or redlined agreement uploaded |
| Primary outcome | Contract case is normalized, deviations are surfaced, and the correct review path is established |
| Process owner | legal operations manager |
| Technical owner | legal workflow automation lead |
| Primary systems | CLM, document repository, ticketing system, email, collaboration platform, policy playbook repository |
| Primary roles | legal operations specialist, counsel, business requestor, approver, compliance reviewer |
| SLA target | Standard contract requests triaged within 1 business day |
| Main risk domains | unapproved clause deviations, privilege exposure, missing approvals, unsupported commitments, regulatory noncompliance |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| LG-001 | Intake | Normalize contract request | Create a usable legal case | completeness check | CLM or request record | legal operations | normalized contract case | No |
| LG-002 | Context | Gather contract and playbook context | Assemble document set, clause standards, and request context | context sufficiency | CLM and playbook repository | legal operations | review packet | No |
| LG-003 | Analysis | Identify deviations and risk issues | Surface clause deviations, missing sections, and risk markers | deviation detection | playbook and contract text | legal operations | deviation list, risk flags | No |
| LG-004 | Routing | Determine review path and approvals | Route to counsel and required reviewers based on deviation profile | control path decision | approval matrix and policy | legal operations | review route | Yes |
| LG-005 | Communication | Draft summary and follow-up requests | Prepare summary for counsel or requests for missing information | content generation | review packet | legal operations | draft summary, outreach draft | No |
| LG-006 | Closure | Confirm triage disposition | Confirm contract is queued for review, escalated, or returned | disposition decision | workflow state record | counsel or legal ops lead | triage disposition | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Legal Example | Why It Matters |
| --- | --- | --- |
| Email | request details, negotiation context, approval comments | Captures intent and urgency |
| Chat | internal legal coordination | Reveals blockers and context |
| Files | contracts, redlines, exhibits, policies | Contain the primary review content |
| Comments | redline notes, reviewer remarks | Explain why issues matter |
| Edits | clause changes, fallback position updates | Show negotiation movement |
| Approvals | business approval, legal escalation, exception sign-off | Define controlled decision points |

### System Signals

| Signal Type | Legal Example | Why It Matters |
| --- | --- | --- |
| Record state | matter status, review queue, approval status | Anchors the workflow |
| Events | document uploaded, playbook updated, approval requested | Trigger progression |
| Master data | contract type, jurisdiction, counterparty, business unit | Constrains review path |
| History | prior similar agreements, prior exceptions | Supports consistency and risk awareness |
| Entitlements | who may access privileged documents or approve terms | Prevents unauthorized access |
| Telemetry | triage cycle time, common deviation themes | Supports legal ops improvement |

### Model Knowledge

| Knowledge Type | Legal Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize issues, rewrite clause explanations | Good for interpretation and drafting |
| Enterprise retrieval | clause playbooks, fallback language, policy standards | Required for grounded outputs |
| Procedural knowledge | triage rubric, matter types, approval routing | Must be owned by Legal |
| Exemplars | approved issue summaries, standard fallback positions | Improve consistency |
| Structured business data | matter metadata, approver registry, counterparty record | Required for routing and audit |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| LG-001 | Normalize contract request | Deterministic automation | Stable request normalization |
| LG-002 | Gather contract and playbook context | AI act within policy | Safe to retrieve approved context |
| LG-003 | Identify deviations and risk issues | AI assist | Strong analysis aid, but legal judgment must remain visible |
| LG-004 | Determine review path and approvals | AI draft plus approve | Recommendation is useful, but routing must remain reviewable |
| LG-005 | Draft summary and follow-up requests | AI draft plus approve | Strong summarization value with low execution risk |
| LG-006 | Confirm triage disposition | Human only for pilot | Final legal triage decision should remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize contract intake
- build contract review packet
- identify clause deviations
- recommend review path
- draft legal issue summary
- draft missing-information request

### Tools And Plugins

- get_contract_case
- get_clause_playbook
- get_approver_matrix
- update_matter_status
- create_review_task
- log_audit_event

## Governance Model

1. The CLM or legal workflow platform remains the source of truth for matter status.
2. AI may surface deviations and draft summaries, but legal advice and approval remain human responsibilities.
3. Privileged or highly sensitive legal materials must stay permission-scoped.
4. Prompt, playbook, and routing rule changes should follow legal operations change control.

## Evaluation Plan

### Pilot Metrics

- deviation detection recall
- incorrect routing rate
- summary acceptance rate
- triage cycle time
- missing approval incidents
- privileged-access incidents

## Implementation Roadmap

### Wave 1

- normalize contract intake
- assemble clause and playbook context
- add deviation detection assist
- add issue summary drafting

### Wave 2

- add review-path recommendation
- add missing-information outreach drafting
- add policy-cited triage summaries

### Wave 3

- add bounded multi-document issue packet assembly
- add proactive detection of likely high-risk deviation themes

## Summary

For a typical enterprise Legal organization, contract intake and clause deviation triage is a strong first application of the Process Decomposition Framework because it is repetitive, document-heavy, risk-aware, and well suited to assistive AI with strict human control over legal judgment.
