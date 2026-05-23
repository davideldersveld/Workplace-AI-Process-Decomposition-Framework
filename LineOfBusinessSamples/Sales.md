# Sales Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Sales function at a large US-based enterprise. It demonstrates how a sales organization can convert proposal and RFP response assembly into a governed AI-enabled workflow.

## Sales Context

A large enterprise sales organization often supports:

- opportunity qualification
- account research and planning
- proposal and RFP response preparation
- deal desk coordination
- pricing and approval packet assembly
- renewal and expansion planning
- meeting follow-up and next-step coordination

These workflows combine large volumes of documents, internal knowledge, product and pricing controls, and many approval dependencies. That makes Sales a strong fit for bounded AI assistance when commitments, pricing, and legal terms remain controlled.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Proposal and RFP response assembly | High | High | High | Medium | Strong | Start here |
| Deal desk packet preparation | Medium | High | High | Low | Strong | Good candidate |
| Opportunity qualification summary | High | Medium | High | Medium | Moderate | Good candidate |
| Renewal risk synthesis | Medium | High | Medium | Medium | Moderate | Later wave |
| Account plan preparation | Medium | Medium | Medium | Medium | Moderate | Second wave |

## Selected Pilot Process

This sample focuses on proposal and RFP response assembly.

## Business Objective

Reduce time to a review-ready proposal package while preserving message accuracy, approved claims, pricing controls, and deal review governance.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Proposal and RFP response assembly |
| Business objective | Convert customer requirements into a reviewed proposal response using approved evidence and controlled approvals |
| Trigger | RFP received, proposal request submitted, or deal support request opened |
| Primary outcome | Response package is complete, aligned to customer requirements, and routed for the correct approvals |
| Process owner | sales operations or proposal center manager |
| Technical owner | revenue systems automation lead |
| Primary systems | CRM, proposal repository, content library, pricing system, collaboration platform, approval workflow, email |
| Primary roles | account executive, sales operations analyst, proposal manager, product specialist, legal reviewer, finance approver |
| SLA target | Produce first draft response package within 3 business days of complete intake |
| Main risk domains | unapproved claims, inaccurate product statements, pricing leakage, approval bypass, outdated content |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SL-001 | Intake | Normalize opportunity request | Create a usable proposal case | completeness check | CRM and request record | sales operations | normalized proposal case | No |
| SL-002 | Context | Gather account, product, and content context | Assemble opportunity context and approved content assets | context sufficiency | CRM and content library | proposal operations | response packet | No |
| SL-003 | Extraction | Extract customer requirements | Identify requirements, questions, deadlines, and constraints | requirement extraction | RFP document and intake record | proposal operations | structured requirement set | No |
| SL-004 | Mapping | Map requirements to approved responses and gaps | Align needs to approved content, identify missing coverage | content coverage decision | approved content repository | proposal operations | mapped response plan, gap list | No |
| SL-005 | Drafting | Draft proposal response | Produce a response package using approved content and citations | content generation | mapped response plan | proposal manager | draft proposal package | No |
| SL-006 | Review | Route pricing, legal, and product approvals | Send the package to required reviewers | approval routing | approval matrix and policy | proposal manager | routed review tasks | Yes |
| SL-007 | Closure | Confirm submission readiness | Confirm package is approved, revised, or escalated | readiness decision | signed review record | sales leader or proposal manager | submission-ready package | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Sales Example | Why It Matters |
| --- | --- | --- |
| Email | customer questions, internal clarifications, approval comments | Contains deal context and obligations |
| Chat | deal desk discussion, product input, deadline coordination | Reveals dependencies and unresolved issues |
| Files | RFP, prior proposals, pricing sheets, architecture diagrams | Contain primary response requirements and evidence |
| Comments | reviewer remarks, redlines, approval conditions | Explain required changes |
| Edits | revised commitments, pricing changes, scope adjustments | Show negotiation and risk areas |
| Approvals | pricing approval, legal sign-off, executive review | Define accountable decision points |

### System Signals

| Signal Type | Sales Example | Why It Matters |
| --- | --- | --- |
| Record state | opportunity stage, deal size, approval status | Anchors the workflow |
| Events | RFP received, due date changed, approval requested | Trigger progression |
| Master data | account tier, product eligibility, pricing rules | Constrains approved response path |
| History | prior proposals, prior objections, win-loss notes | Supports reuse and quality |
| Entitlements | who can approve pricing or legal changes | Prevents unauthorized commitments |
| Telemetry | proposal cycle time, approval lag, content reuse rate | Supports process improvement |

### Model Knowledge

| Knowledge Type | Sales Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize customer asks, rewrite responses, tailor tone | Good for drafting and synthesis |
| Enterprise retrieval | approved content, product FAQs, prior winning responses, pricing policy | Required for grounded outputs |
| Procedural knowledge | proposal template, review checklist, deal desk policy | Must be owned by Sales operations |
| Exemplars | approved proposals and strong answers | Improve quality and consistency |
| Structured business data | opportunity state, product availability, pricing approvals | Required for correct commitments |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| SL-001 | Normalize opportunity request | Deterministic automation | Stable intake normalization |
| SL-002 | Gather account, product, and content context | AI act within policy | Safe to retrieve approved context |
| SL-003 | Extract customer requirements | AI assist | Valuable extraction task, but review should remain visible |
| SL-004 | Map requirements to approved responses and gaps | AI assist | Good fit for coverage analysis, but not final commitment authority |
| SL-005 | Draft proposal response | AI draft plus approve | Strong drafting value with required human review |
| SL-006 | Route pricing, legal, and product approvals | AI act within policy | Safe when bound to approved matrices |
| SL-007 | Confirm submission readiness | Human only for pilot | Final customer-facing submission should remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize proposal request
- build account and content context packet
- extract RFP requirements
- map requirements to approved content
- draft proposal response
- summarize approval packet

### Tools And Plugins

- get_opportunity_record
- get_approved_content_assets
- get_pricing_policy
- get_review_matrix
- update_proposal_status
- create_review_task
- log_audit_event

## Governance Model

1. CRM and approved content repositories remain sources of truth for opportunity state and response assets.
2. AI may draft and map, but pricing, legal language, and final customer commitments require human approval.
3. Outdated or unapproved marketing claims must not be surfaced as approved response content.
4. Every generated response should preserve source references where applicable.

## Evaluation Plan

### Pilot Metrics

- requirement extraction accuracy
- response draft acceptance rate
- approval routing accuracy
- time to first draft
- unapproved claim rate
- final review rework rate

## Implementation Roadmap

### Wave 1

- normalize proposal requests
- assemble account and content context
- add requirement extraction assist
- add draft proposal support

### Wave 2

- add gap detection and content mapping
- add approval packet summaries
- add controlled routing to legal, finance, and product reviewers

### Wave 3

- add bounded agent support for large multi-document RFP assembly
- add proactive identification of likely approval blockers

## Summary

For a typical enterprise Sales organization, proposal and RFP response assembly is a strong first application of the Process Decomposition Framework because it is knowledge-heavy, time-sensitive, and highly improvable without giving up control over pricing, legal terms, or customer commitments.
