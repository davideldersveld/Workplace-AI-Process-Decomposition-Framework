# Marketing Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Marketing function at a large US-based enterprise. It demonstrates how a marketing organization can convert campaign request intake and brief synthesis into a governed AI-enabled workflow.

## Marketing Context

A large enterprise marketing organization often supports:

- campaign intake and brief creation
- content request triage
- brand and compliance review routing
- audience and channel planning
- event brief preparation
- field marketing support
- asset production coordination

These workflows combine unstructured business requests, many dependencies, brand controls, legal review, and repetitive synthesis work. That makes Marketing a strong fit for bounded AI assistance when claims, approvals, and channel governance remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Campaign request intake and brief synthesis | High | High | High | Medium | Strong | Start here |
| Content review and approval routing | High | High | High | Medium | Strong | Good candidate |
| Event brief preparation | Medium | Medium | Medium | Medium | Moderate | Good candidate |
| Audience and channel recommendation | Medium | High | Medium | Medium | Moderate | Later wave |
| Asset metadata tagging and reuse detection | High | Medium | High | High | Low | Second wave |

## Selected Pilot Process

This sample focuses on campaign request intake and brief synthesis.

## Business Objective

Reduce time to a review-ready campaign brief while preserving brand consistency, approved claims, compliance routing, and clear ownership.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Campaign request intake and brief synthesis |
| Business objective | Convert fragmented marketing requests into a structured, review-ready campaign brief |
| Trigger | New campaign request, launch request, or field marketing request submitted |
| Primary outcome | Campaign brief is complete, traceable, and routed for the correct reviews |
| Process owner | marketing operations manager |
| Technical owner | martech automation lead |
| Primary systems | work management platform, DAM, CRM, product marketing repository, email, collaboration platform, approval workflow |
| Primary roles | marketing manager, campaign manager, content strategist, product marketing lead, brand reviewer, legal reviewer |
| SLA target | Produce a first review-ready brief within 3 business days of complete intake |
| Main risk domains | unapproved claims, unclear audience, missing compliance review, brand inconsistency, hidden scope expansion |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MK-001 | Intake | Normalize campaign request | Create a usable campaign case | completeness check | work request record | marketing operations | normalized campaign case | No |
| MK-002 | Context | Gather brand, product, and audience context | Assemble request, message, product, and audience context | context sufficiency | request record and approved repositories | marketing operations | campaign packet | No |
| MK-003 | Extraction | Extract goals, constraints, and dependencies | Identify objective, audience, deliverables, timing, and blockers | signal extraction | request artifacts | marketing operations | structured request signals | No |
| MK-004 | Synthesis | Draft campaign brief | Create a structured brief with goals, audience, channels, assets, and open questions | content generation | campaign packet and templates | campaign manager | draft brief | No |
| MK-005 | Routing | Route brand, product, and legal reviews | Send the brief to required reviewers | approval routing | review matrix | campaign manager | routed review tasks | Yes |
| MK-006 | Closure | Confirm brief baseline | Confirm the brief is approved, revised, or escalated | readiness decision | signed review record | marketing lead | baseline brief or rework path | Yes |

## Signal Inventory

### Human Signals

| Signal Type | Marketing Example | Why It Matters |
| --- | --- | --- |
| Email | launch requests, stakeholder clarifications, review comments | Contains hidden scope and timing signals |
| Chat | campaign coordination, asset dependencies | Reveals unresolved issues and urgency |
| Files | briefs, decks, product sheets, creative assets | Contain baseline content and evidence |
| Comments | review notes, brand feedback, legal edits | Explain required changes |
| Edits | message changes, audience shifts, asset list changes | Show scope churn and risk |
| Approvals | brand approval, legal approval, launch sign-off | Define controlled review points |

### System Signals

| Signal Type | Marketing Example | Why It Matters |
| --- | --- | --- |
| Record state | request status, review status, launch readiness | Anchors the workflow |
| Events | request submitted, asset updated, review requested | Trigger progression |
| Master data | audience definitions, product list, channel taxonomy | Constrains planning choices |
| History | prior campaigns, prior review comments, asset reuse | Supports quality and consistency |
| Entitlements | who may approve claims or launch | Prevents unauthorized publication |
| Telemetry | cycle time, revision count, asset reuse rate | Supports process improvement |

### Model Knowledge

| Knowledge Type | Marketing Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize requests, rewrite brief language, structure messaging | Good for synthesis and drafting |
| Enterprise retrieval | brand guidelines, approved claims, product messaging, campaign templates | Required for grounded outputs |
| Procedural knowledge | brief template, review checklist, launch rubric | Must be owned by Marketing operations |
| Exemplars | approved briefs and strong launch packets | Improve consistency |
| Structured business data | audience segments, channel metadata, product availability | Required for accurate routing and planning |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| MK-001 | Normalize campaign request | Deterministic automation | Stable request normalization |
| MK-002 | Gather brand, product, and audience context | AI act within policy | Safe to retrieve approved context |
| MK-003 | Extract goals, constraints, and dependencies | AI assist | Valuable extraction task, but should stay reviewable |
| MK-004 | Draft campaign brief | AI draft plus approve | Strong drafting value with required human review |
| MK-005 | Route brand, product, and legal reviews | AI act within policy | Safe when bound to approved routing rules |
| MK-006 | Confirm brief baseline | Human only for pilot | Final brief approval should stay human-owned |

## Translation To Technical Artifacts

### Skills

- normalize campaign intake
- build campaign context packet
- extract goals and constraints
- draft campaign brief
- summarize review comments
- recommend reviewer routing

### Tools And Plugins

- get_campaign_request
- get_brand_guidelines
- get_product_messaging
- get_review_matrix
- update_brief_status
- create_review_task
- log_audit_event

## Governance Model

1. The work management platform remains the source of truth for campaign state.
2. AI may synthesize and draft, but brand, legal, and launch approvals remain human-owned.
3. Unapproved claims or outdated product statements must not be treated as approved content.
4. Prompt, template, and review-rule changes should follow marketing operations governance.

## Evaluation Plan

### Pilot Metrics

- brief draft acceptance rate
- review routing accuracy
- time to first review-ready brief
- revision rounds per brief
- unapproved-claim incidents
- launch delay rate due to missing brief inputs

## Implementation Roadmap

### Wave 1

- normalize campaign requests
- assemble campaign context
- add request extraction assist
- add brief drafting support

### Wave 2

- add reviewer routing
- add review summary generation
- add policy-cited claim coverage checks

### Wave 3

- add bounded multi-source campaign packet assembly
- add proactive detection of likely review blockers or missing stakeholders

## Summary

For a typical enterprise Marketing organization, campaign request intake and brief synthesis is a strong first application of the Process Decomposition Framework because it is synthesis-heavy, approval-bound, and easy to measure through turnaround time and review quality.
