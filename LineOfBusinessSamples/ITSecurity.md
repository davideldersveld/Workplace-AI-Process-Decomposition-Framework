# IT Security Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical IT Security function at a large US-based enterprise. It demonstrates how a security operations organization can convert security alert triage into a governed AI-enabled workflow.

## IT Security Context

A large enterprise IT security organization often supports:

- security alert triage
- incident investigation preparation
- phishing report handling
- identity and access anomaly review
- policy violation intake
- evidence collection coordination
- escalation and containment routing

These workflows combine high alert volumes, structured telemetry, analyst notes, and strong risk controls. That makes IT Security a strong fit for bounded AI assistance when containment, response actions, and incident declarations remain explicitly human-owned.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Security alert triage and investigation prep | High | High | High | Low | Strong | Start here |
| Phishing report triage | High | High | High | Medium | Strong | Good candidate |
| Identity anomaly review support | High | High | High | Low | Strong | Good candidate |
| Security case summary preparation | Medium | High | High | Medium | Strong | Second wave |
| Control alert clustering | Medium | Medium | High | Medium | Moderate | Later wave |

## Selected Pilot Process

This sample focuses on security alert triage and investigation prep.

## Business Objective

Reduce time to a review-ready security case while preserving detection quality, containment controls, evidence integrity, and analyst accountability.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Security alert triage and investigation prep |
| Business objective | Convert inbound security alerts into complete, prioritized, review-ready cases for analyst action |
| Trigger | New alert generated from SIEM, endpoint, identity, email security, or analyst intake |
| Primary outcome | Alert is normalized, enriched, risk-indicated, routed, and prepared for analyst review |
| Process owner | security operations manager |
| Technical owner | SecOps platform automation lead |
| Primary systems | SIEM, SOAR, ticketing platform, endpoint platform, identity platform, threat intel sources, collaboration platform |
| Primary roles | security analyst, incident responder, SOC lead, identity analyst, threat analyst |
| SLA target | High-priority alerts triaged within 15 minutes |
| Main risk domains | false negatives, incorrect severity, unauthorized containment, evidence loss, data sensitivity exposure |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SEC-001 | Intake | Normalize security alert | Create a usable security case from inbound telemetry | completeness check | SIEM or alert record | security operations | normalized alert case | No |
| SEC-002 | Context | Gather entity, asset, and threat context | Assemble user, host, identity, and related-alert context | context sufficiency | security telemetry and asset data | security analyst | alert context packet | No |
| SEC-003 | Triage | Classify alert type and likely risk | Determine alert category, likely threat path, and confidence | risk indication | detection taxonomy and alert record | security analyst | alert category, risk flags | No |
| SEC-004 | Priority | Assess severity and investigation path | Determine urgency, blast radius, and escalation need | severity decision | severity matrix and environment context | security analyst | priority, investigation path | Sometimes |
| SEC-005 | Routing | Assign owner and next action | Route to the correct analyst queue or incident path | owner selection | routing rules and operating model | SOC lead | assigned case | No |
| SEC-006 | Communication | Draft investigation summary and follow-up requests | Prepare analyst handoff summary or evidence request | content generation | context packet | security analyst | draft case summary, evidence request | No |
| SEC-007 | Closure | Confirm triage disposition | Confirm case is queued, escalated, or closed as benign | disposition decision | case state record | SOC lead | final triage state | Yes for incident declaration or containment |

## Signal Inventory

### Human Signals

| Signal Type | IT Security Example | Why It Matters |
| --- | --- | --- |
| Email | phishing reports, escalation notices, evidence follow-up | Captures user-reported context and urgency |
| Chat | SOC coordination, incident bridge notes, escalation threads | Reveals real-time impact and decisions |
| Files | malicious attachment samples, exported logs, screenshots | Provide primary investigation artifacts |
| Comments | analyst notes, handoff remarks, investigation hypotheses | Preserve reasoning and rationale |
| Edits | severity changes, ownership changes, updated hypothesis | Show churn and decision quality issues |
| Approvals | incident declaration, containment approval, executive escalation | Define controlled decision points |

### System Signals

| Signal Type | IT Security Example | Why It Matters |
| --- | --- | --- |
| Record state | alert status, case state, queue, SLA timer | Anchors the workflow |
| Events | related alerts, repeat detections, identity anomalies | Trigger re-evaluation |
| Master data | asset criticality, identity role, environment classification | Constrains severity and routing |
| History | prior alerts, prior incidents, prior suppressions | Supports triage quality |
| Entitlements | who may view, escalate, or contain | Prevents unauthorized actions |
| Telemetry | alert volume, false-positive trend, repeat entities | Supports prioritization and tuning |

### Model Knowledge

| Knowledge Type | IT Security Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize alerts, rewrite analyst notes, explain next steps | Good for drafting and synthesis |
| Enterprise retrieval | playbooks, triage SOPs, asset criticality rules, prior cases | Required for grounded outputs |
| Procedural knowledge | severity matrix, routing rubric, evidence checklist | Must be owned by Security Operations |
| Exemplars | approved analyst summaries and case packets | Improve consistency |
| Structured business data | alert metadata, asset context, identity roles, case state | Required for correct prioritization and routing |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| SEC-001 | Normalize security alert | Deterministic automation | Stable alert normalization |
| SEC-002 | Gather entity, asset, and threat context | AI act within policy | Safe to retrieve approved enrichment context |
| SEC-003 | Classify alert type and likely risk | AI assist | Valuable triage aid, but analyst review must remain visible |
| SEC-004 | Assess severity and investigation path | AI draft plus approve | Recommendation is useful but must remain reviewable |
| SEC-005 | Assign owner and next action | AI act within policy | Safe when tied to routing rules |
| SEC-006 | Draft investigation summary and follow-up requests | AI draft plus approve | Strong summarization value with required review |
| SEC-007 | Confirm triage disposition | Human only for pilot | Incident declaration and containment remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize alert intake
- build alert enrichment packet
- classify alert type and likely risk
- recommend severity and investigation path
- recommend analyst routing
- draft investigation summary and evidence requests

### Tools And Plugins

- get_alert_record
- get_entity_and_asset_context
- get_related_alerts
- get_playbook_context
- update_case_status
- assign_analyst_queue
- create_incident_task
- log_audit_event

## Governance Model

1. The SIEM or security case platform remains the source of truth for alert and case state.
2. AI may enrich, summarize, and recommend, but containment, incident declaration, and material response actions remain human-owned.
3. Sensitive security telemetry and identity data must remain permission-scoped.
4. Every write action must preserve actor, timestamp, prior value, and case linkage.
5. Playbook, severity, and routing rule changes should follow SecOps change control.

## Evaluation Plan

### Pilot Metrics

- alert classification quality
- correct routing rate
- time to triage high-priority alerts
- severity recommendation agreement rate
- false-negative escape review rate
- override rate
- unauthorized action incidents

## Implementation Roadmap

### Wave 1

- normalize alert intake
- assemble entity and asset context
- add alert-classification assist
- add analyst-summary drafting

### Wave 2

- add severity recommendation
- add controlled routing and incident-path support
- add playbook-cited triage summaries

### Wave 3

- add bounded multi-source investigation packet assembly
- add proactive identification of repeated alert clusters or likely escalation patterns

## Summary

For a typical enterprise IT Security organization, security alert triage and investigation preparation is a strong first application of the Process Decomposition Framework because it is high-volume, evidence-heavy, and well suited to assistive AI with strict human control over incident decisions and containment actions.
