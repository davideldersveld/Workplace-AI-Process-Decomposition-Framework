# Supply Chain Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Supply Chain function at a large US-based enterprise. It demonstrates how a supply chain organization can convert shortage and disruption events into a governed AI-enabled exception workflow.

## Supply Chain Context

A large enterprise supply chain organization often supports:

- supply shortage and disruption handling
- demand and inventory exception review
- order fulfillment coordination
- supplier issue escalation
- shipment delay triage
- allocation and recovery planning support
- operations communication and status reporting

These workflows combine high volumes of system events, supplier and logistics signals, operational constraints, and recurring exception handling. That makes Supply Chain a strong fit for bounded AI assistance when allocation, commitments, and financial impacts remain explicitly controlled.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Inventory shortage and disruption triage | High | High | High | Medium | Strong | Start here |
| Shipment delay exception routing | High | High | High | Medium | Strong | Good candidate |
| Supplier issue packet preparation | Medium | High | Medium | Medium | Strong | Good candidate |
| Allocation review support | Medium | High | Medium | Low | Strong | Later wave |
| Demand signal clustering | Medium | Medium | High | Medium | Moderate | Second wave |

## Selected Pilot Process

This sample focuses on inventory shortage and disruption triage.

## Business Objective

Reduce time to identify, classify, and route supply shortages while preserving inventory accuracy, allocation control, and cross-functional visibility.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Inventory shortage and disruption triage |
| Business objective | Convert shortage and disruption signals into complete, prioritized, and routed exception cases |
| Trigger | Inventory shortage threshold breached, supplier delay reported, or fulfillment exception created |
| Primary outcome | Exception is normalized, impact-assessed, routed, and prepared for mitigation review |
| Process owner | supply chain operations manager |
| Technical owner | planning and workflow automation lead |
| Primary systems | ERP, planning system, transportation platform, supplier portal, workflow platform, collaboration platform |
| Primary roles | supply planner, inventory analyst, logistics coordinator, procurement liaison, operations manager |
| SLA target | High-priority supply exceptions triaged within 1 hour |
| Main risk domains | missed customer impact, incorrect priority, duplicate handling, unsupported allocation changes, weak audit trail |

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SC-001 | Intake | Normalize shortage event | Create a usable supply exception case | completeness check | ERP or planning event | supply operations | normalized shortage case | No |
| SC-002 | Context | Gather demand, inventory, and shipment context | Assemble affected item, site, customer, and transit context | context sufficiency | planning and logistics data | supply planner | context packet | No |
| SC-003 | Triage | Classify exception type and likely cause | Determine shortage type, source of disruption, and impact area | category assignment | exception taxonomy and source data | supply planner | exception type, cause hypothesis | No |
| SC-004 | Priority | Assess business impact and mitigation path | Determine urgency, downstream impact, and escalation need | impact decision | policy and demand data | supply planner | priority, mitigation path | Sometimes |
| SC-005 | Routing | Assign owner and next action | Route to planning, logistics, procurement, or operations owner | owner selection | routing rules and operating model | supply operations | routed exception | No |
| SC-006 | Communication | Draft shortage summary and follow-up requests | Prepare cross-functional updates and requests for missing information | content generation | context packet | supply planner | draft update, handoff summary | No |
| SC-007 | Closure | Confirm triage disposition | Confirm the case is routed, escalated, or returned for more data | disposition decision | workflow state record | operations manager | final triage state | Yes for escalations |

## Signal Inventory

### Human Signals

| Signal Type | Supply Chain Example | Why It Matters |
| --- | --- | --- |
| Email | supplier delay notifications, site escalations, customer-impact notes | Captures urgency and disruption detail |
| Chat | planner coordination, logistics updates, escalation threads | Reveals blockers and workarounds |
| Files | supplier notices, shipment documents, spreadsheets | Provide supporting evidence and details |
| Comments | planner notes, mitigation rationale, manager direction | Explain why priority or routing changed |
| Edits | revised shortage classification, reassigned owner | Show churn and routing quality issues |
| Approvals | allocation review approval, escalation approval | Define controlled decision points |

### System Signals

| Signal Type | Supply Chain Example | Why It Matters |
| --- | --- | --- |
| Record state | shortage case status, inventory state, shipment state | Anchors the workflow |
| Events | stockout threshold, delay event, receipt update | Trigger progression |
| Master data | item, site, supplier, customer, lane, ownership model | Constrains impact and routing |
| History | prior shortages, prior mitigations, repeat disruption patterns | Supports better triage |
| Entitlements | who may reallocate, expedite, or approve exceptions | Prevents unauthorized changes |
| Telemetry | aging, backlog, recurring shortage categories | Supports operational improvement |

### Model Knowledge

| Knowledge Type | Supply Chain Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize disruption, rewrite updates, explain next steps | Good for drafting and synthesis |
| Enterprise retrieval | SOPs, shortage playbooks, routing rules, prior cases | Required for grounded outputs |
| Procedural knowledge | severity rubric, escalation checklist, mitigation templates | Must be owned by Supply Chain |
| Exemplars | approved shortage summaries and handoff notes | Improve consistency |
| Structured business data | inventory, demand, shipment, site ownership | Required for correct prioritization and routing |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| SC-001 | Normalize shortage event | Deterministic automation | Stable event normalization |
| SC-002 | Gather demand, inventory, and shipment context | AI act within policy | Safe to retrieve approved context |
| SC-003 | Classify exception type and likely cause | AI assist | Valuable triage aid, but should remain reviewable |
| SC-004 | Assess business impact and mitigation path | AI draft plus approve | Recommendation is useful but must remain reviewable |
| SC-005 | Assign owner and next action | AI act within policy | Safe when tied to routing rules |
| SC-006 | Draft shortage summary and follow-up requests | AI draft plus approve | High-value drafting with low execution risk |
| SC-007 | Confirm triage disposition | Human only for pilot | Final escalation and mitigation path remain human-owned |

## Translation To Technical Artifacts

### Skills

- normalize shortage intake
- build supply context packet
- classify shortage type and cause
- recommend impact and mitigation path
- recommend routing owner
- draft shortage summary and follow-up

### Tools And Plugins

- get_shortage_case
- get_inventory_and_demand_context
- get_shipment_status
- get_routing_rules
- update_exception_status
- assign_queue_owner
- create_escalation_task
- log_audit_event

## Governance Model

1. ERP and planning systems remain the sources of truth for inventory and shortage state.
2. AI may recommend and draft, but allocation changes and customer-impacting commitments remain human-owned.
3. Every write action must preserve actor, timestamp, prior value, and case linkage.
4. Routing, severity, and mitigation rules should be versioned and controlled by Supply Chain leadership.

## Evaluation Plan

### Pilot Metrics

- shortage classification accuracy
- correct routing rate
- time to triage high-priority shortages
- override rate
- repeat misrouting rate
- aging reduction for high-impact shortage cases

## Implementation Roadmap

### Wave 1

- normalize shortage events
- assemble supply context
- add classification assist
- add summary and follow-up drafting

### Wave 2

- add impact and mitigation recommendation
- add controlled routing and escalation support
- add SOP-cited triage summaries

### Wave 3

- add bounded multi-source disruption packet assembly
- add proactive detection of likely repeated shortages or site-specific risk patterns

## Summary

For a typical enterprise Supply Chain organization, inventory shortage and disruption triage is a strong first application of the Process Decomposition Framework because it is queue-based, high-volume, and highly dependent on fast context assembly and correct routing.
