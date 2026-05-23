# Product Management Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Product Management function at a large US-based enterprise. It demonstrates how a product organization can convert fragmented feature demand into a governed, review-ready opportunity framing workflow.

## Product Management Context

A large enterprise product organization often supports:

- customer and stakeholder request intake
- product feedback synthesis
- opportunity framing and problem definition
- roadmap candidate preparation
- feature hypothesis documentation
- cross-functional review and prioritization support
- dependency and impact identification

These workflows combine high volumes of human input, product telemetry, market context, and repeated synthesis work. That makes Product Management a strong fit for bounded AI assistance when prioritization and roadmap decisions remain explicitly human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Feature request intake and opportunity framing | High | High | High | Medium | Strong | Start here |
| Customer feedback clustering and synthesis | High | High | High | Medium | Moderate | Good candidate |
| PRD first-draft preparation | Medium | High | Medium | Medium | Strong | Good candidate |
| Release-readiness summary preparation | Medium | High | Medium | Medium | Moderate | Later wave |
| Prioritization packet assembly | Medium | High | Medium | Low | Strong | Second wave |

## Selected Pilot Process

This sample focuses on feature request intake and opportunity framing.

### Why This Process Is A Good First Target

- It is a common intake point for new product demand.
- It aggregates customer, sales, support, and internal stakeholder signals.
- It benefits heavily from clustering, summarization, and gap detection.
- It has a clear outcome: a review-ready opportunity brief.
- Errors are usually detectable before roadmap commitment if review discipline remains in place.
- It improves product manager leverage without delegating prioritization authority.

## Business Objective

Reduce the time and inconsistency involved in turning fragmented feature requests into a review-ready opportunity brief with clear problem framing, evidence, open questions, and traceability.

## Scope

Included in scope:

- feature request intake from multiple channels
- signal normalization and duplicate detection
- customer and telemetry context assembly
- theme clustering and opportunity framing
- draft opportunity brief generation
- reviewer packet preparation
- routing to product, design, engineering, or GTM reviewers

Excluded from scope:

- final roadmap prioritization decisions
- automatic commitment to delivery dates
- solution architecture decisions
- automatic scope approval
- direct customer promises without human review

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Feature request intake and opportunity framing |
| Business objective | Convert fragmented demand signals into a structured, evidence-backed opportunity brief |
| Trigger | New customer request, internal request, feedback batch, or product intake submission |
| Primary outcome | Opportunity brief is complete, deduplicated, review-ready, and routed for the correct stakeholders |
| Process owner | product operations manager or group product manager |
| Technical owner | product systems or workflow automation lead |
| Primary systems | product intake platform, CRM, support case platform, product analytics, backlog system, document repository, collaboration platform |
| Primary roles | product manager, product operations analyst, design lead, engineering lead, support lead, sales lead |
| SLA target | Produce a first review-ready opportunity brief within 3 business days of complete intake |
| Main risk domains | duplicate demand, weak evidence, hidden dependencies, premature commitments, missed stakeholder impact |

## Current-State Workflow Summary

In a typical enterprise, feature requests arrive through sales escalations, support cases, customer advisory boards, roadmap meetings, internal stakeholder asks, and analytics signals. Product managers and product operations analysts gather the request context, review customer history, check existing backlog items, compare similar requests, and assemble evidence before turning the request into a usable opportunity framing artifact.

The work is valuable but often slowed by fragmented inputs, inconsistent formatting, weak traceability to source signals, and repeated manual effort to summarize the same context for different stakeholders.

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PM-001 | Intake | Normalize feature request event | Create a usable opportunity case from inbound signals | completeness check | intake record | product operations | normalized opportunity case | No |
| PM-002 | Context | Gather product, customer, and telemetry context | Assemble customer, account, usage, backlog, and prior decision context | context sufficiency | source systems and backlog records | product operations | opportunity context packet | No |
| PM-003 | Synthesis | Detect duplicates and cluster related demand | Group similar requests and identify common themes | clustering decision | opportunity context packet | product operations | grouped demand themes, duplicate flags | No |
| PM-004 | Framing | Draft opportunity statement and open questions | Define user problem, business impact, evidence, and unresolved questions | framing decision | context packet and product templates | product manager | draft opportunity brief | No |
| PM-005 | Review | Prepare stakeholder review packet | Package the brief, supporting evidence, and traceability for reviewers | review readiness | opportunity brief and evidence links | product manager | review packet | No |
| PM-006 | Routing | Route to product, design, engineering, and GTM reviewers | Send the brief to the correct review group | reviewer routing | reviewer matrix and ownership model | product manager | routed review tasks | Yes |
| PM-007 | Closure | Confirm disposition | Confirm brief is accepted for prioritization, revised, or returned for more discovery | disposition decision | signed review record | product leader | accepted brief or rework path | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Product Management Example | Why It Matters |
| --- | --- | --- |
| Email | customer escalations, internal feature asks, follow-up clarifications | Capture explicit demand and urgency |
| Chat | product coordination, sales escalations, support threads | Reveal hidden context, urgency, and stakeholder sentiment |
| Files | decks, discovery notes, feedback exports, screenshots | Often contain supporting evidence and framing detail |
| Comments | backlog comments, reviewer notes, design feedback | Explain objections, dependencies, and tradeoffs |
| Edits | rewritten problem statements, scope changes, duplicate merges | Show churn and alignment issues |
| Approvals | reviewer sign-off to move into prioritization | Define accountable decision points |
| Meetings and calls | discovery interviews, customer calls, roadmap syncs | Provide nuanced user and business context |

### System Signals

| Signal Type | Product Management Example | Why It Matters |
| --- | --- | --- |
| Record state | intake status, backlog state, review status | Anchors the workflow to durable state |
| Events | request submitted, duplicate linked, review requested | Trigger workflow progression |
| Master data | product area, customer segment, owner, application map | Constrains routing and impact framing |
| History | prior requests, prior declines, prior launches | Supports reuse and better framing |
| Entitlements | who may edit, review, or baseline opportunity briefs | Prevents unauthorized changes |
| Telemetry | product usage, churn indicators, support volume, adoption metrics | Provides evidence for value and urgency |

### Model Knowledge

| Knowledge Type | Product Management Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize feedback, rewrite problem statements, structure briefs | Good for synthesis and drafting |
| Enterprise retrieval | prior briefs, roadmap principles, product glossary, feedback taxonomy | Required for grounded outputs |
| Procedural knowledge | brief template, review checklist, prioritization rubric | Must be owned by Product |
| Exemplars | approved opportunity briefs and strong problem statements | Improve consistency |
| Structured business data | telemetry, customer tier, backlog metadata | Required for evidence-backed framing |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| PM-001 | Normalize feature request event | Deterministic automation | Stable intake normalization and metadata capture |
| PM-002 | Gather product, customer, and telemetry context | AI act within policy | Safe to retrieve approved context |
| PM-003 | Detect duplicates and cluster related demand | AI assist | Strong fit for clustering, but merge decisions should stay reviewable |
| PM-004 | Draft opportunity statement and open questions | AI draft plus approve | High drafting value with required product manager ownership |
| PM-005 | Prepare stakeholder review packet | AI draft plus approve | Strong summarization and packaging value |
| PM-006 | Route to product, design, engineering, and GTM reviewers | AI act within policy | Safe when tied to approved reviewer rules |
| PM-007 | Confirm disposition | Human only for pilot | Prioritization readiness should remain human-owned initially |

## Translation To Technical Artifacts

### Skills

- normalize opportunity intake
- build product and telemetry context packet
- cluster duplicate demand signals
- draft opportunity brief
- build reviewer packet
- summarize stakeholder feedback

### Tools And Plugins

- get_intake_record
- get_product_usage_context
- get_customer_and_support_context
- get_backlog_history
- update_opportunity_status
- create_review_task
- log_audit_event

## Governance Model

1. The intake system and backlog platform remain the sources of truth for case and status data.
2. AI may synthesize and draft, but prioritization decisions and roadmap commitments remain human-owned.
3. Every generated opportunity brief should preserve source references and open questions.
4. Prompt, taxonomy, and review-rule changes should follow Product operations governance.

## Evaluation Plan

### Pilot Metrics

- duplicate detection quality
- brief draft acceptance rate
- time to first review-ready brief
- reviewer routing accuracy
- percentage of briefs with source-linked evidence
- override rate on clustering or framing decisions

## Implementation Roadmap

### Wave 1

- normalize feature request intake
- assemble customer and telemetry context
- add duplicate detection assist
- add opportunity brief drafting

### Wave 2

- add stakeholder review packet generation
- add controlled reviewer routing
- add source-cited evidence summaries

### Wave 3

- add bounded multi-source discovery synthesis for large signal sets
- add proactive identification of likely duplicate opportunities across products or segments

## Summary

For a typical enterprise Product Management organization, feature request intake and opportunity framing is a strong first application of the Process Decomposition Framework because it is synthesis-heavy, high-volume, and operationally improvable without delegating product strategy decisions to the model.
