# HR Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical HR function at a large US-based enterprise. It demonstrates how an HR team can turn a high-volume, policy-heavy workflow into a governed AI-enabled operating model.

## HR Context

A large enterprise HR organization typically supports:

- employee onboarding and offboarding
- HR case management
- policy inquiry handling
- manager advisory support
- leave and accommodation coordination
- employee data change requests
- talent acquisition coordination
- learning and compliance tracking

These workflows often combine sensitive employee data, policy interpretation, many handoffs, and recurring documentation tasks. That makes HR a strong fit for bounded AI assistance when approvals, privacy controls, and auditability are explicit.

## Candidate Process Inventory

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Employee onboarding readiness | High | High | High | Medium | Strong | Start here |
| HR policy inquiry triage | High | High | High | Medium | Moderate | Good candidate |
| Employee data change requests | High | High | Medium | Low | Strong | Second wave |
| Leave and accommodation case prep | Medium | High | Medium | Low | Strong | Later wave |
| Offboarding coordination | Medium | High | High | Medium | Strong | Good candidate |
| Internal mobility request support | Medium | Medium | Medium | Medium | Moderate | Later wave |

## Selected Pilot Process

This sample focuses on employee onboarding readiness.

### Why This Process Is A Good First Target

- It is high-volume and recurring.
- It spans multiple teams but has a clear lifecycle.
- It involves structured and unstructured inputs.
- It has strong business metrics such as readiness date and case aging.
- Many tasks are assistive, document-heavy, and reviewable.
- Failures are usually detectable before the first day if visibility is good.

## Business Objective

Improve new-hire onboarding readiness, reduce coordination time, and increase case completeness without weakening privacy, policy compliance, or ownership.

## Scope

Included in scope:

- onboarding case intake
- missing information detection
- policy and checklist context assembly
- stakeholder routing
- draft outreach to hiring manager, HR partner, or employee
- readiness summary generation
- approval and sign-off preparation

Excluded from scope:

- final payroll setup execution
- employment eligibility legal adjudication
- benefits enrollment decisions
- termination workflows
- final manager approvals without human review

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Employee onboarding readiness |
| Business objective | Move each new hire to a review-ready onboarding state with complete evidence and clear ownership |
| Trigger | New hire record created or onboarding request submitted |
| Primary outcome | New hire onboarding package is complete, routed, reviewed, and ready before start date |
| Process owner | HR operations manager |
| Technical owner | people systems automation lead |
| Primary systems | HRIS, ATS, ticketing system, document repository, email, collaboration platform, identity workflow |
| Primary roles | HR operations specialist, recruiter, hiring manager, HR business partner, IT onboarding coordinator |
| SLA target | Complete standard onboarding readiness review within 2 business days of case creation |
| Main risk domains | missing mandatory documents, privacy breaches, incorrect role setup, policy noncompliance, missed start date |

## Current-State Workflow Summary

In a typical enterprise, a new hire event triggers a chain of HR operational work. HR specialists verify employee details, confirm manager and location information, gather required documents, check policy requirements, coordinate with IT and facilities, and chase missing data across email, chat, and systems. Much of the work is repetitive but slowed by incomplete intake, unclear ownership, and manual follow-up.

## Step-Level Decomposition

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| HR-ONB-001 | Intake | Normalize onboarding event | Create a usable onboarding case | completeness check | HRIS new hire record | HR operations | normalized onboarding case | No |
| HR-ONB-002 | Discovery | Gather readiness context | Assemble employee, role, location, and policy context | context sufficiency | HRIS and policy repository | HR operations | readiness packet | No |
| HR-ONB-003 | Triage | Identify missing items and risks | Detect gaps, blockers, and policy-sensitive conditions | gap detection | checklist and policy rules | HR operations | missing item list, risk flags | No |
| HR-ONB-004 | Routing | Assign owners and next actions | Route tasks to the correct role or queue | owner selection | responsibility matrix | HR operations | routed work items | Sometimes |
| HR-ONB-005 | Communication | Draft outreach and reminders | Prepare employee, manager, or partner communications | content generation | readiness packet | HR operations | draft outreach messages | No |
| HR-ONB-006 | Review | Prepare readiness summary | Summarize status, blockers, and required approvals | review readiness | case state | HR operations | readiness summary | No |
| HR-ONB-007 | Closure | Confirm onboarding readiness | Confirm the case is complete or escalated | readiness decision | signed review record | HR operations manager | ready, escalate, or rework state | Yes |

## Example Step Records

### HR-ONB-003: Identify Missing Items And Risks

```yaml
step_id: HR-ONB-003
step_name: Identify missing items and risks
stage: Triage
goal: Detect missing information, readiness blockers, and policy-sensitive exceptions
trigger: Readiness packet assembled
inputs:
  - new hire profile
  - role and location data
  - required document checklist
  - policy extracts
  - prior onboarding notes
source_of_truth:
  - onboarding checklist standard
  - HR policy repository
decision_type: gap detection
systems:
  - HRIS
  - document repository
  - ticketing system
human_roles:
  - HR operations specialist
outputs:
  - missing_items
  - risk_flags
  - next_action_recommendations
approvals_required: false
exceptions:
  - missing work authorization artifact
  - conflicting location data
  - privileged access request
owner: HR operations
success_metrics:
  - missing_item_detection_rate
  - readiness_cycle_time
  - reviewer_acceptance_rate
```

## Signal Inventory

### Human Signals

| Signal Type | HR Example | Why It Matters |
| --- | --- | --- |
| Email | hiring manager clarifications, employee document follow-up | Contains intent, missing information, and urgency |
| Chat | recruiter and HR operations coordination | Reveals informal dependencies and blockers |
| Files | offer packet, identification documents, policy forms | Often contain mandatory onboarding evidence |
| Comments | case notes, reviewer annotations, escalation remarks | Explain why a case is blocked or approved |
| Edits | role change, location correction, start date update | Show churn and impact readiness calculations |
| Approvals | manager approval, exception sign-off | Establish accountable decision points |
| Meetings and calls | onboarding review sync notes | Surface unresolved issues and commitments |
| Escalations | urgent start date issue, privacy concern | Signal high-risk handling needs |

### System Signals

| Signal Type | HR Example | Why It Matters |
| --- | --- | --- |
| Record state | onboarding case status, start date, document status | Anchors the workflow to durable state |
| Events | hire created, start date changed, document uploaded | Trigger progression or re-evaluation |
| Master data | job code, location, cost center, manager hierarchy | Constrains routing and checklist applicability |
| History | prior onboarding delays, reopened cases | Supports prioritization and exception handling |
| Entitlements | who may view, edit, or approve sensitive data | Prevents unauthorized exposure |
| Telemetry | readiness backlog, aging, frequent blockers | Supports operational improvement |

### Model Knowledge

| Knowledge Type | HR Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize issues, draft reminders, explain next steps | Safe for interpretation and drafting |
| Enterprise retrieval | onboarding policy, location rules, role checklist, FAQs | Required for current company-specific outputs |
| Procedural knowledge | onboarding rubric, escalation rules, templates | Must be owned by HR operations |
| Exemplars | approved readiness summaries, good outreach examples | Improves consistency |
| Structured business data | hire data, org hierarchy, checklist state | Required for accurate routing and status |

## Automation Boundary

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| HR-ONB-001 | Normalize onboarding event | Deterministic automation | Stable event and metadata capture |
| HR-ONB-002 | Gather readiness context | AI act within policy | Safe to retrieve approved context |
| HR-ONB-003 | Identify missing items and risks | AI assist | Valuable for gap detection, but review should remain visible |
| HR-ONB-004 | Assign owners and next actions | AI draft plus approve | Routing suggestion is useful, but should be reviewable |
| HR-ONB-005 | Draft outreach and reminders | AI draft plus approve | High drafting value with manageable review cost |
| HR-ONB-006 | Prepare readiness summary | AI draft plus approve | Good fit for summarization before manager review |
| HR-ONB-007 | Confirm onboarding readiness | Human only for pilot | Final readiness decision should stay human-owned initially |

## Translation To Technical Artifacts

### Skills

| Skill | Purpose |
| --- | --- |
| Normalize onboarding case | Convert new hire event to structured case |
| Build readiness packet | Assemble employee, role, and checklist context |
| Detect missing items | Identify missing evidence and risks |
| Recommend routing | Suggest next owner and queue |
| Draft onboarding outreach | Generate employee or manager reminders |
| Summarize readiness | Produce reviewer-ready case summary |

### Tools And Plugins

| Tool Or Plugin | Action Type | System |
| --- | --- | --- |
| Get new hire record | Read | HRIS |
| Get onboarding checklist | Read | policy or workflow system |
| Get manager and org data | Read | directory or HRIS |
| Update case status | Write | ticketing or workflow system |
| Create approval task | Write | approval platform |
| Post communication draft | Write | email or collaboration platform |
| Log audit event | Write | workflow or audit store |

### Workflow

The primary workflow coordinates intake normalization, readiness context assembly, missing item detection, routing, draft communication, summary preparation, and readiness review.

### Agent Recommendation

Do not center the first HR implementation on a broad HR agent. Use workflows, skills, and tools with explicit privacy and approval controls. Introduce an agent later only for bounded multi-source case assembly where the risk is controlled.

## Governance Model For HR

1. Sensitive employee data must be permission-scoped and masked where appropriate.
2. The HRIS remains the source of truth for employee and case status.
3. AI may recommend and draft, but final readiness and exception approvals remain human-owned.
4. Every write action must capture actor, timestamp, prior value, and case linkage.
5. Prompt, template, and checklist changes should follow HR operations change control.

## Evaluation Plan

### Pilot Metrics

- readiness cycle time
- missing item detection rate
- draft acceptance rate
- reroute rate
- overdue onboarding case rate
- privacy or permission incidents
- reviewer trust and override rate

### Pilot Design

1. Start with one region, business unit, or employee population.
2. Use assistive and draft-plus-approve modes first.
3. Keep final readiness approval under human ownership.
4. Review exceptions, overrides, and privacy concerns weekly.

## Implementation Roadmap

### Wave 1

- normalize onboarding events
- build readiness packet assembly
- add missing item detection
- add draft manager and employee outreach

### Wave 2

- add routing recommendations
- add readiness summaries
- add policy-cited exception preparation

### Wave 3

- add bounded multi-source case assembly for complex onboarding cases
- add proactive prediction of likely readiness delays

## Summary

For a typical enterprise HR organization, employee onboarding readiness is a strong first application of the Process Decomposition Framework. It is repetitive, policy-aware, document-heavy, and easy to control with explicit review and approval boundaries.