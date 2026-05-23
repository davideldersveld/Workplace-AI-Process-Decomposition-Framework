# Business Analysis Sample

## Purpose

This document shows what applying the Process Decomposition Framework looks like for a typical Business Analysis function at a large US-based enterprise. It is intended to demonstrate how a business analysis team can move from fragmented stakeholder inputs to a governed, traceable, AI-enabled requirements workflow.

The sample focuses on business analysis work performed within an enterprise portfolio, product, transformation, or IT delivery organization where analysts support projects, enhancements, process changes, and operating model updates.

## Business Analysis Context

A large enterprise business analysis function often supports work such as:

- Demand intake and problem framing.
- Stakeholder discovery.
- Requirements elicitation.
- Requirements synthesis.
- Process mapping and gap analysis.
- User story and acceptance criteria development.
- Business rule documentation.
- Scope and change control.
- Review and sign-off coordination.

This function is a strong fit for process-centric AI because it combines high volumes of unstructured signals, repeatable synthesis patterns, approval-heavy document flows, and a strong need for traceability.

## Candidate Process Inventory

An initial business analysis assessment might review several workflows before choosing one for implementation.

| Process | Volume | Business Value | Data Readiness | Error Tolerance | Approval Structure | Recommended Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Requirements intake and synthesis | High | High | High | Medium | Strong | Start here |
| Stakeholder interview summarization | High | Medium | High | Medium | Moderate | Good candidate |
| User story and acceptance criteria drafting | High | High | Medium | Medium | Strong | Good candidate |
| Requirements traceability maintenance | Medium | High | Medium | Low | Strong | Second wave |
| Change request impact assessment | Medium | High | Medium | Low | Strong | Later wave |
| UAT defect triage and business classification | High | Medium | High | Medium | Moderate | Good candidate |

## Selected Pilot Process

This sample focuses on requirements intake and synthesis for enterprise change requests.

### Why This Process Is A Good First Target

- It is a common entry point for new work.
- It aggregates many human signals such as emails, workshop notes, chats, attachments, and comments.
- It has a clear business outcome: a reviewed, structured requirements package.
- It contains repeatable AI-friendly tasks such as extraction, clustering, summarization, gap detection, conflict detection, and draft generation.
- It benefits from traceability and review controls rather than full autonomy.
- Most errors are detectable and correctable before build begins if review discipline is maintained.

## Business Objective

Reduce the time and inconsistency involved in turning stakeholder requests into a reviewed, traceable requirements package without weakening ownership, scope control, or approval discipline.

## Scope

Included in scope:

- Request intake from email, forms, chats, or planning systems.
- Discovery artifact collection.
- Stakeholder signal normalization.
- Requirement theme clustering and synthesis.
- Draft business requirements, user stories, business rules, and acceptance criteria.
- Review packet generation.
- Traceability preparation from source signals to requirements.
- Sign-off preparation and routing.

Excluded from scope:

- Final prioritization decisions for portfolio funding.
- Technical solution architecture.
- Automatic approval of scope changes.
- Production change deployment.
- Final legal or regulatory interpretation without human review.

## Process Definition

| Field | Value |
| --- | --- |
| Process name | Requirements intake and synthesis |
| Business objective | Convert fragmented stakeholder demand into a reviewed, traceable requirements package |
| Trigger | New business request, change request, enhancement request, or project discovery kickoff |
| Primary outcome | Approved or review-ready requirements package with clear scope, assumptions, decisions, and traceability |
| Process owner | business analysis manager or product operations lead |
| Technical owner | delivery platform lead or AI workflow owner |
| Primary systems | request intake platform, ticketing system, document repository, collaboration platform, email, backlog system, requirements repository |
| Primary roles | business analyst, product owner, requestor, subject matter expert, delivery manager, architect, compliance reviewer |
| SLA target | Produce initial synthesized requirements package within 5 business days of complete intake |
| Main risk domains | scope ambiguity, missing stakeholder input, contradictory requirements, approval bypass, poor traceability, regulatory omission |

## Current-State Workflow Summary

In a typical enterprise, new requests arrive through email, intake forms, planning boards, chats, or meetings. A business analyst collects background documents, interviews stakeholders, reviews existing process maps, gathers policy and system context, and tries to reconcile conflicting input into a coherent set of requirements.

The work is essential but often slowed by fragmented inputs, repeated clarifications, inconsistent documentation quality, missed dependencies, and weak traceability between source signals and final requirements. Analysts spend time reformatting notes, drafting summary documents, chasing approvals, and manually reconciling feedback rather than focusing on decision quality.

## Step-Level Decomposition

This sample decomposes the process into stages and steps that meet the framework rule: one dominant goal, one main decision type, one primary source of truth, and one accountable owner.

| Step ID | Stage | Step Name | Goal | Decision Type | Primary Source Of Truth | Owner | Outputs | Approvals Required |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BA-REQ-001 | Intake | Normalize request event | Create a usable analysis case from incoming request signals | completeness check | intake record | business analyst | normalized request case | No |
| BA-REQ-002 | Discovery | Gather supporting context | Assemble current-state documents, prior decisions, and stakeholder context | context sufficiency | intake case and repository records | business analyst | discovery packet | No |
| BA-REQ-003 | Discovery | Extract candidate needs and constraints | Pull stated needs, goals, constraints, and assumptions from source material | signal extraction | source artifacts | business analyst | extracted signal set | No |
| BA-REQ-004 | Synthesis | Cluster and reconcile themes | Group related needs, detect conflicts, and identify gaps | synthesis and conflict detection | discovery packet | business analyst | synthesized requirement themes | No |
| BA-REQ-005 | Drafting | Draft requirements package | Produce structured requirements, business rules, user stories, and acceptance criteria | content generation | approved templates and synthesized themes | business analyst | draft requirements package | No |
| BA-REQ-006 | Review | Prepare review and traceability packet | Link requirements to source signals and unresolved questions | review readiness | requirements package and traceability policy | business analyst | review packet | No |
| BA-REQ-007 | Review | Route for feedback and sign-off | Send draft package to correct reviewers and collect responses | reviewer routing | approval matrix and stakeholder registry | business analyst | routed review task | Yes |
| BA-REQ-008 | Closure | Confirm disposition and baseline | Confirm requirements are approved, revised, or returned for more discovery | baseline decision | signed review record | product owner or business sponsor | approved baseline or rework path | Yes |

## Example Step Records

### BA-REQ-004: Cluster And Reconcile Themes

```yaml
step_id: BA-REQ-004
step_name: Cluster and reconcile themes
stage: Synthesis
goal: Convert fragmented stakeholder input into coherent requirement themes and identify conflicts
trigger: Extracted signal set available
inputs:
  - workshop notes
  - stakeholder emails
  - request form
  - existing process maps
  - prior decisions
  - policy excerpts
source_of_truth:
  - discovery packet
  - requirements taxonomy
decision_type: synthesis and conflict detection
systems:
  - document repository
  - collaboration platform
  - requirements repository
human_roles:
  - business analyst
outputs:
  - requirement_themes
  - conflicts_detected
  - open_questions
approvals_required: false
exceptions:
  - directly contradictory stakeholder requests
  - missing impacted team input
  - undefined business rule owner
owner: business analysis
success_metrics:
  - synthesis_quality
  - conflict_detection_rate
  - time_to_draft
```

### BA-REQ-006: Prepare Review And Traceability Packet

```yaml
step_id: BA-REQ-006
step_name: Prepare review and traceability packet
stage: Review
goal: Create a reviewer-ready package that ties each major requirement to supporting source material
trigger: Draft requirements package completed
inputs:
  - draft requirements package
  - extracted signal set
  - stakeholder roster
  - review checklist
source_of_truth:
  - requirements package
  - traceability standard
decision_type: review readiness check
systems:
  - document repository
  - backlog system
  - review workflow platform
human_roles:
  - business analyst
outputs:
  - review_summary
  - traceability_matrix
  - unresolved_questions_list
approvals_required: false
exceptions:
  - requirement without source support
  - unresolved policy question
  - missing stakeholder review group
owner: business analysis
success_metrics:
  - review_acceptance_rate
  - traceability_coverage
  - revision_rounds
```

## Signal Inventory

### Human Signals

| Signal Type | Business Analysis Example | Why It Matters |
| --- | --- | --- |
| Email | request details, clarifications, sign-off comments | Contains intent, assumptions, and hidden scope signals |
| Chat | follow-up questions, informal agreements, issue threads | Reveals unresolved ambiguity and real stakeholder concerns |
| Files | BRDs, slide decks, process maps, policy documents, spreadsheets, screenshots | Often contain baseline context and evidence for requirements |
| Comments | document review comments, ticket comments, review annotations | Show disagreement, changes, and approval conditions |
| Edits | requirement rewrites, acceptance criteria changes, scope updates | Reveal areas of churn and weak alignment |
| Approvals | sign-off responses, reviewer acknowledgments, delegated approvals | Define decision ownership and baseline readiness |
| Meetings and calls | workshop notes, transcripts, action items, stakeholder interviews | Provide the richest source of business intent and constraints |
| Escalations | scope disputes, compliance concerns, dependency escalations | Signal higher-risk requirements or governance needs |

### System Signals

| Signal Type | Business Analysis Example | Why It Matters |
| --- | --- | --- |
| Record state | request status, review status, sign-off status, backlog state | Anchors the workflow to durable state |
| Events | request submitted, review requested, comment added, sign-off completed | Trigger process progression |
| Master data | stakeholder roster, application inventory, business capability map, org hierarchy | Helps routing, traceability, and impact framing |
| History | prior enhancement requests, prior decisions, prior defects, prior requirement changes | Supports reuse, consistency, and impact assessment |
| Entitlements | who can review, edit, approve, or baseline | Prevents unauthorized changes or false approvals |
| Telemetry | review cycle time, number of revisions, defect leakage, feedback volume | Supports process improvement and workload planning |

### Model Knowledge

| Knowledge Type | Business Analysis Use | Constraint |
| --- | --- | --- |
| General language knowledge | summarize interviews, rewrite requirements, explain tradeoffs | Useful for synthesis and drafting, not final authority on business intent |
| Enterprise retrieval | policies, prior BRDs, process maps, domain glossaries, historical decisions | Required for grounded, enterprise-specific outputs |
| Procedural knowledge | requirements templates, quality rubric, review checklist, traceability standard | Must be maintained by the BA practice |
| Exemplars | approved requirements packages, strong user stories, good acceptance criteria | Improves consistency and accelerates drafting |
| Structured business data | backlog metadata, app inventory, org data, approval matrix | Required for routing, traceability, and governance |

### Signal Quality Notes

Business analysis teams should explicitly score signal sources on:

- Freshness, because stakeholder intent and scope may shift quickly.
- Reliability, because informal chat or verbal agreements may conflict with approved documentation.
- Permission sensitivity, because early-stage initiatives may include confidential strategy or employee data.
- Source-of-truth status, because final requirements must trace back to approved and attributable sources.
- Audit requirement, because later disputes often depend on who requested what and when.

## Automation Boundary

Each step gets one primary operating mode.

| Step ID | Step Name | Operating Mode | Why |
| --- | --- | --- | --- |
| BA-REQ-001 | Normalize request event | Deterministic automation | Stable intake normalization and metadata capture |
| BA-REQ-002 | Gather supporting context | AI act within policy | Safe to retrieve and assemble approved context |
| BA-REQ-003 | Extract candidate needs and constraints | AI assist | High-value extraction, but analysts should review source fidelity |
| BA-REQ-004 | Cluster and reconcile themes | AI assist | Good fit for synthesis and conflict surfacing, but not final decision ownership |
| BA-REQ-005 | Draft requirements package | AI draft plus approve | Strong drafting value, but outputs require analyst ownership |
| BA-REQ-006 | Prepare review and traceability packet | AI draft plus approve | Traceability can be prepared automatically but must be reviewed |
| BA-REQ-007 | Route for feedback and sign-off | AI act within policy | Safe when tied to approved reviewer registry and workflow rules |
| BA-REQ-008 | Confirm disposition and baseline | Human only for pilot | Baseline approval should stay human-owned initially |

## AI Task Pattern Mapping

| Step ID | Dominant Task Pattern | Secondary Patterns |
| --- | --- | --- |
| BA-REQ-001 | Extract | Validate |
| BA-REQ-002 | Retrieve | Summarize |
| BA-REQ-003 | Extract | Classify |
| BA-REQ-004 | Summarize | Compare |
| BA-REQ-005 | Generate | Structure |
| BA-REQ-006 | Compare | Summarize |
| BA-REQ-007 | Route | Recommend |
| BA-REQ-008 | Decide | Escalate |

## Translation To Technical Artifacts

This is where the process becomes an implementation design.

### Skills

| Skill | Purpose | Inputs | Outputs |
| --- | --- | --- | --- |
| Normalize request intake | Convert incoming request into a structured analysis case | form, email, chat, ticket | normalized case record |
| Build discovery packet | Assemble documents, prior decisions, and impacted systems context | request ID, business area, app list | discovery packet |
| Extract stakeholder needs | Pull needs, constraints, assumptions, and decisions from source material | workshop notes, emails, comments, docs | extracted signal set |
| Synthesize requirement themes | Group related needs, detect conflicts, and identify open questions | extracted signal set, taxonomy | requirement themes, conflict list |
| Draft requirements package | Generate business requirements, user stories, rules, and acceptance criteria | themes, templates, standards | draft package |
| Build traceability matrix | Map requirements to supporting source signals | draft package, signal set | traceability matrix |
| Summarize review package | Produce concise reviewer summary and unresolved issues | draft package, traceability matrix, comments | review summary |

### Tools And Plugins

| Tool Or Plugin | Action Type | System |
| --- | --- | --- |
| Get intake request | Read | request management platform |
| Get prior decisions and artifacts | Read | document repository |
| Get stakeholder roster | Read | directory or stakeholder registry |
| Get backlog records | Read | backlog or work management system |
| Create review task | Write | workflow or approval platform |
| Update requirement status | Write | requirements repository or backlog system |
| Post reviewer packet | Write | collaboration platform or document system |
| Log audit event | Write | workflow or audit store |

### Workflow

The primary workflow should coordinate:

1. Intake normalization.
2. Discovery packet assembly.
3. Signal extraction.
4. Theme synthesis and conflict detection.
5. Draft package generation.
6. Traceability packet creation.
7. Review routing.
8. Baseline or rework decision.

### Agent Recommendation

For the first business analysis implementation, do not make a broad autonomous analyst agent the center of the runtime. Use workflows plus skills and tools.

An agent can be introduced later for bounded multi-source synthesis, but only if:

1. Source retrieval boundaries are explicit.
2. Every generated requirement is traceable.
3. Human review gates remain mandatory.
4. Scope changes cannot be silently introduced.

## Sample Skill Contract

```yaml
skill_id: BA-SKILL-REQ-SYNTHESIS
name: Synthesize requirements themes
purpose: Convert extracted stakeholder input into coherent requirement themes, conflicts, and open questions
trigger: Extracted signal set available
required_inputs:
  - extracted_signal_set
  - requirements_taxonomy
  - discovery_packet
optional_inputs:
  - prior_requirements_package
  - stakeholder_comments
output_schema:
  requirement_themes: list[string]
  conflicts_detected: list[string]
  open_questions: list[string]
  reviewer_summary: string
source_of_truth:
  - discovery packet
  - approved requirements taxonomy
  - enterprise glossary
tool_dependencies:
  - get_prior_decisions_and_artifacts
  - get_backlog_records
  - get_stakeholder_roster
approvals_required: false
latency_target_seconds: 15
audit_fields:
  - request_id
  - case_id
  - model_version
  - prompt_version
  - source_references
success_metrics:
  - synthesis_acceptance_rate
  - conflict_detection_rate
  - time_to_first_draft
failure_handling:
  - missing_source_material
  - contradictory_input_without_owner
  - unclassified_requirement_theme
```

## Business Analysis Reference Architecture

### Layer 1: Signal Intake And Normalization

Inputs arrive from request forms, email, workshop notes, collaboration channels, document repositories, and backlog systems. The intake layer converts them into a normalized analysis event.

Recommended normalized fields:

- Analysis case ID.
- Request ID.
- Requestor.
- Business capability or domain.
- Impacted application or process.
- Source system.
- Trigger event.
- Priority.
- Current status.
- Stakeholder group.
- Permission context.
- Attached artifacts.
- Retention class.

### Layer 2: Process Model

The process model resolves whether the request is in intake, discovery, synthesis, review, or closure. It also determines which templates, reviewers, policies, and traceability requirements apply.

### Layer 3: Capability Registry

This layer stores versioned skill definitions, tool contracts, review templates, requirements taxonomies, policies, sign-off rules, and prompt assets used by the BA practice.

### Layer 4: Runtime Orchestrator

The orchestrator receives the analysis event, resolves current state, gathers context, selects the allowed execution unit, enforces review and sign-off controls, and records the result.

### Layer 5: Knowledge And Context Builder

The context builder assembles:

- intake request details
- workshop notes and transcripts
- prior decisions and existing requirements
- process maps and SOPs
- policy or compliance references
- impacted system inventory
- reviewer roster and sign-off rules
- approved exemplars for requirements packages

### Layer 6: Memory And State

Durable state should include:

- current request status
- discovery completeness status
- extracted signal set
- open questions
- conflicts detected
- review comments
- sign-off status
- baseline version
- manual overrides

### Layer 7: Decision And Approval Plane

This layer enforces:

- mandatory human review before baseline approval
- reviewer routing based on domain and impacted systems
- escalation when conflicting requirements lack an accountable owner
- fallback when source coverage is insufficient
- traceability coverage checks before sign-off

### Layer 8: Governance And Control Plane

Business-analysis-specific control considerations include:

- Generated requirements must be attributable to source signals or explicit analyst judgment.
- Scope additions introduced by the model must be visible and reviewable.
- Review comments and change history must be preserved.
- Approved baseline artifacts must be versioned.
- Prompt, template, and taxonomy changes should be controlled by the BA practice.

### Layer 9: Evaluation And Observability

Track both skill quality and business outcomes.

Component metrics:

- extraction accuracy
- synthesis acceptance rate
- traceability coverage accuracy
- draft package acceptance rate
- routing correctness

Business metrics:

- time to first review-ready draft
- number of review cycles
- requirement defect leakage into delivery
- percentage of requirements with traceable source support
- stakeholder satisfaction with clarity and completeness
- scope change frequency after baseline

## Governance Model For Business Analysis

Business analysis requires strong governance around ownership, traceability, and change control.

### Key Governance Rules

1. AI may draft and synthesize, but baseline approval must follow named business ownership.
2. Every major requirement should be traceable to source evidence, explicit stakeholder decision, or documented analyst judgment.
3. The model must not silently add scope, policy interpretation, or business rules without visibility.
4. Every write action and status change must preserve author, timestamp, and rationale.
5. Generated review packets should include unresolved questions and confidence caveats where applicable.
6. Changes to prompts, templates, taxonomies, and review rules should be versioned and approved.

### Ownership Model

| Asset | Owner |
| --- | --- |
| Process definition | business analysis manager or practice lead |
| Review standards and traceability rules | BA practice lead with governance stakeholders |
| Skill contracts and orchestration logic | delivery platform or automation engineering |
| Prompt, template, and taxonomy assets | joint ownership between BA practice and technical owner |
| Data access model | information owner and enterprise identity team |
| Evaluation benchmarks | business analysis practice with delivery support |

## Evaluation Plan

### Offline Evaluation Set

Build a benchmark set that includes:

- clear single-stakeholder enhancement requests
- multi-stakeholder requests with conflicting priorities
- requests with missing business objective
- requests with hidden compliance implications
- noisy discovery notes with repeated or contradictory statements
- requests that should be split into separate requirement themes
- requests with insufficient source support for sign-off

### Acceptance Thresholds For Pilot

Suggested thresholds before expanding autonomy:

| Measure | Target |
| --- | --- |
| Requirement theme synthesis acceptance | at least 85 percent with minor edits only |
| Traceability coverage | 100 percent for critical requirements |
| Incorrect reviewer routing | below 5 percent |
| Baseline approval without human review | zero |
| Missing audit fields | zero |
| Unattributed scope additions | zero tolerated in pilot |

### Pilot Design

1. Start with one portfolio, product domain, or business unit.
2. Use AI assist and draft-plus-approve modes first.
3. Keep final baseline approval under human ownership.
4. Review synthesis errors, missed dependencies, and traceability gaps weekly.
5. Expand only after consistent analyst trust and low-severity failure patterns.

## Implementation Roadmap For Business Analysis

### Wave 1

- Normalize request intake events.
- Build discovery packet assembly.
- Add interview and artifact summarization.
- Add extraction of candidate needs, constraints, and assumptions.
- Add draft reviewer summaries.

### Wave 2

- Add requirement theme synthesis.
- Add draft requirements package generation.
- Add traceability matrix preparation.
- Add reviewer routing and checklist enforcement.

### Wave 3

- Add bounded multi-source synthesis agent for larger discovery sets.
- Add change request impact pre-assessment.
- Add proactive detection of likely missing stakeholders or missing requirement areas.

## Sample Deliverables Produced By This Exercise

If a business analysis team applied the Process Decomposition Framework to this workflow, the expected artifacts would be:

- A scored BA process inventory.
- A decomposed requirements intake and synthesis workflow.
- A signal inventory for stakeholder and system inputs.
- An automation boundary matrix.
- Skill contracts for core BA capabilities.
- Tool contracts for repository, workflow, and backlog actions.
- A BA-specific governance matrix.
- An evaluation benchmark set and pilot plan.

## Summary

For a typical large-enterprise Business Analysis function, requirements intake and synthesis is a strong first example of how to apply the Process Decomposition Framework. It has enough structure to govern, enough variability to benefit from AI, and enough review discipline to keep ownership and traceability intact.

The key design choice is not to create a generic analyst chatbot. It is to start with a bounded requirements workflow, decompose it into steps and decisions, assign the correct automation mode to each step, and translate only the appropriate parts into skills, tools, workflows, and approvals.