# IT Service Management Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical IT Service Management function at a large US-based enterprise. It demonstrates how an ITSM organization can convert incident intake and triage into a governed AI-enabled workflow.

## IT Service Management Context

A large enterprise ITSM organization often supports:

- incident intake and triage
- service request handling
- access request coordination
- major incident support
- problem management intake
- knowledge-assisted troubleshooting
- assignment and escalation routing

These workflows combine high volumes of tickets, structured configuration data, user-reported symptoms, service-level objectives, and strict change controls. That makes ITSM a strong fit for bounded AI assistance when remediation authority and production-impacting actions remain explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Incident intake and triage | High | High | High | Medium | Strong | Start here |
| Service request classification and routing | High | High | High | Medium | Strong | Good candidate |
| Major incident summary preparation | Medium | High | High | Medium | Strong | Good candidate |
| Knowledge article suggestion | High | Medium | High | High | Moderate | Second wave |
| Problem candidate clustering | Medium | Medium | Medium | Medium | Moderate | Later wave |

## Selected Pilot Process

This sample focuses on incident intake and triage.

## Business Objective

Reduce time to correct incident classification, prioritization, and assignment while preserving service quality, incident severity controls, and production change discipline.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Incident intake and triage |
| Business objective | Convert inbound incident signals into a correctly classified, prioritized, and assigned incident record |
| Trigger | New ticket, chat escalation, monitoring-generated ticket, or support email creates an incident |
| Primary outcome | Incident is normalized, classified, prioritized, assigned, and prepared for the right next action |
| Process owner | IT service operations manager |
| Technical owner | service management platform lead |
| Primary systems | ITSM platform, CMDB, monitoring tools, identity platform, collaboration platform, knowledge base |
| Primary roles | service desk analyst, incident manager, resolver group lead, end user support analyst |
| SLA target | Standard incidents classified and routed within 10 minutes |
| Main risk domains | incorrect severity, wrong resolver assignment, missed major incident trigger, unauthorized remediation, user data exposure |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ITSM-001 | Intake | Normalize incident event | Create a usable incident record from inbound signals | completeness check | ITSM ticket record | service desk | normalized incident | No |
| ITSM-002 | Context | Gather user, service, and asset context | Assemble affected user, service, asset, and recent change context | context sufficiency | ITSM and CMDB records | service desk | incident context packet | No |
| ITSM-003 | Triage | Classify incident type and affected service | Determine issue category, likely service, and initial diagnosis path | category assignment | taxonomy and incident record | service desk | category, service, confidence | No |
| ITSM-004 | Priority | Assess severity and escalation path | Determine impact, urgency, and escalation need | severity decision | severity policy and service metadata | service desk | priority, escalation path | Sometimes |
| ITSM-005 | Routing | Assign resolver group and next action | Route the incident to the correct owner or queue | owner selection | assignment rules and support model | service desk | assigned incident | No |
| ITSM-006 | Communication | Draft user update and handoff summary | Prepare user-facing update or internal handoff summary | content generation | incident packet and knowledge base | service desk | draft update, handoff summary | No |
| ITSM-007 | Closure | Confirm triage disposition | Confirm incident is routed, escalated, or returned for clarification | disposition decision | ticket state record | incident manager or team lead | final triage state | Yes for major incidents |

## Signal Inventory

### Human Signals

| Signal Type | ITSM Example | Why It Matters |
| --- | --- | --- |
| Email | outage reports, screenshot attachments, user clarification | Captures symptoms and urgency |
| Chat | service desk conversations, war-room notes, escalation messages | Reveals evolving impact and informal triage context |
| Files | screenshots, logs, exported error details | Provide evidence for diagnosis |
| Comments | analyst notes, resolver remarks, major incident updates | Preserve reasoning and state changes |
| Edits | corrected category, reassigned owner, updated priority | Show churn and routing quality issues |
| Approvals | emergency escalation acknowledgement, change approval dependency | Define controlled decision points |
| Calls and transcripts | support calls or incident bridge summaries | Capture unresolved user impact details |

### System Signals

| Signal Type | ITSM Example | Why It Matters |
| --- | --- | --- |
| Record state | ticket status, assignment group, SLA timer | Anchors the workflow |
| Events | monitor alerts, reopen events, threshold breaches | Trigger incident progression |
| Master data | CMDB CI, service ownership, support model | Constrains assignment and escalation |
| History | prior incidents, prior resolutions, recent changes | Supports faster triage and problem detection |
| Entitlements | who may view, assign, or escalate incidents | Prevents unauthorized actions |
| Telemetry | queue depth, breach risk, repeat incident volume | Supports prioritization |

### Model Knowledge

| Knowledge Type | ITSM Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize symptoms, rewrite updates, explain next steps | Good for interpretation and drafting |
| Enterprise retrieval | knowledge articles, runbooks, support models, service policies | Required for grounded outputs |
| Procedural knowledge | incident taxonomy, severity matrix, triage checklist | Must be owned by ITSM |
| Exemplars | approved handoff summaries and user updates | Improve consistency |
| Structured business data | ticket metadata, CMDB links, service ownership | Required for correct routing and priority |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| ITSM-001 | Normalize incident event | Deterministic automation | Stable intake normalization |
| ITSM-002 | Gather user, service, and asset context | AI act within policy | Safe to retrieve approved context |
| ITSM-003 | Classify incident type and affected service | AI assist | High-value triage task, but review should remain visible |
| ITSM-004 | Assess severity and escalation path | AI draft plus approve | Recommendation is useful but must remain reviewable |
| ITSM-005 | Assign resolver group and next action | AI act within policy | Safe when bound to assignment rules |
| ITSM-006 | Draft user update and handoff summary | AI draft plus approve | Strong drafting value with manageable review cost |
| ITSM-007 | Confirm triage disposition | Human only for pilot | Final severity and escalation disposition should remain human-owned initially |

## Translation To Technical Artifacts

### Skills

- normalize incident intake
- build incident context packet
- classify incident and affected service
- recommend severity and escalation path
- recommend assignment group
- draft user update and handoff summary

### Tools And Plugins

- get_incident_record
- get_cmdb_context
- get_recent_changes
- get_knowledge_articles
- update_ticket_status
- assign_resolver_group
- create_major_incident_task
- log_audit_event

## Governance Model

1. The ITSM platform remains the source of truth for incident state.
2. AI may recommend and draft, but production-impacting remediation and major incident decisions require human ownership.
3. Access to incident data must remain permission-scoped, especially for identity or security-sensitive incidents.
4. Every write action must capture actor, timestamp, prior value, and ticket linkage.

## Evaluation Plan

### Pilot Metrics

- incident classification accuracy
- correct assignment rate
- time to triage
- major incident trigger miss rate
- SLA breach rate
- override rate
- unauthorized action incidents

## Implementation Roadmap

### Wave 1

- normalize incident intake
- assemble service and asset context
- add classification assist
- add user update and handoff drafting

### Wave 2

- add severity recommendation
- add assignment recommendation and controlled routing
- add knowledge-cited triage summaries

### Wave 3

- add bounded multi-source incident packet assembly
- add proactive detection of likely major incidents or repeat patterns

## Summary

For a typical enterprise IT Service Management organization, incident intake and triage is a strong first application of the Process Decomposition Framework because it is high-volume, operationally measurable, and well suited to assistive AI with clear control points.
