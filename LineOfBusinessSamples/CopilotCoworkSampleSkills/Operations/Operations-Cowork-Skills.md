# Operations Exception Intake and Resolution Routing — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Operational Exception Intake and Resolution Routing** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the exception handling process — from intake through review-ready triage — into AI-assisted capabilities within Microsoft 365.

The workflow supports exception events from transaction processing, service delivery, quality checks, and manual queue intake, guiding each through structured normalization, context assembly, classification, impact assessment, routing, and communication drafting with strict safety-first guardrails, SLA compliance tracking, circular routing detection, audit traceability, and queue throughput visibility at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **ops-exception-intake** | Normalizes inbound exception events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **ops-context-packet** | Gathers process, transaction, queue, and SOP context | AI act within policy | analysis | SearchSparkle |
| 3 | **ops-classify-exception** | Classifies exception type and likely cause | AI assist | analysis | Tag |
| 4 | **ops-impact-assess** | Assesses impact, priority, aging risk, and escalation need | AI draft + approve | analysis | Flag |
| 5 | **ops-route-exception** | Assigns owner or queue based on routing rules | AI act within policy | communication | Mail |
| 6 | **ops-exception-comms** | Drafts follow-up, handoff, escalation, and status communications | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Exception Intake │  Normalize exception event → structured case
│     (ops-exception-  │  Validates fields, checks duplicates, flags
│      intake)         │  reopened exceptions
│                      │  SLA: Standard exceptions triaged within 30 minutes
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Gather transaction details, processing history,
│     (ops-context-    │  queue state, SOPs, prior similar exceptions,
│      packet)         │  and supporting documents
│                      │  Output: Adaptive Card (speed-focused)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Classify         │  Determine exception type and likely cause
│     Exception        │  using taxonomy, context, and prior patterns
│     (ops-classify-   │  Safety-first rule: classify ambiguous cases
│      exception)      │  as more severe type
│                      │  Output: Adaptive Card with classification
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Impact Assess    │  Evaluate downstream impact, assign priority,
│     (ops-impact-     │  assess aging risk, determine handling path,
│      assess)         │  recommend escalation
│                      │  Output: Adaptive Card with priority
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Route Exception  │  Assign to correct owner or queue per
│     (ops-route-      │  routing rules; send Teams notifications;
│      exception)      │  create SLA deadline calendar holds
│                      │  Output: Teams messages + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Exception Comms  │  Draft follow-up requests, handoff summaries,
│     (ops-exception-  │  escalation notices, status updates, and
│      comms)          │  batch queue summaries
│                      │  Output: Outlook drafts + Teams messages
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm exception is resolved, escalated,
│     Disposition      │  or returned for rework. Human-only step —
│     (not automated)  │  final disposition carries operational
│                      │  accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Exception Intake | Structured field extraction, validation, duplicate checking — no AI judgment on exception severity or routing |
| **AI act within policy** | Context Packet, Route Exception | Retrieves approved context from defined sources or executes routing within pre-approved rules; does not exercise judgment on classification or priority |
| **AI assist** | Classify Exception | Surfaces classification recommendation with evidence and confidence; analyst reviews before recording |
| **AI draft + approve** | Impact Assess, Exception Comms | AI recommends priority or drafts communications; analyst reviews and confirms before actions are taken |
| **Human only** | Confirm Disposition | Final escalation or closure decision is always human-owned — carries operational accountability |

## Governance Controls

### Safety-First Guardrails

- Quality defects, compliance violations, and equipment failures are always classified as the more severe type when evidence is ambiguous
- Safety-critical exceptions bypass standard priority logic and always receive an escalation recommendation
- Safety exceptions route to the designated safety or compliance queue regardless of standard routing rules
- Safety classification is prominently included in all communications and never omitted for brevity
- Analyst downgrade of a safety classification requires explicit documented rationale

### SLA Compliance

- Standard exceptions must be classified and routed within 30 minutes
- SLA deadlines by priority: Critical (30 min), High (2 hours), Medium (4 hours), Low (1 business day)
- Exceptions aging beyond 80% of SLA window receive explicit urgency callouts
- SLA-breached exceptions receive immediate escalation recommendations
- SLA deadline calendar holds are created for every routed exception

### No Auto-Close, Auto-Resolve, or Auto-Escalate

- No skill may automatically close, resolve, or escalate an exception — these are irreversible disposition changes requiring human decision with documented rationale
- Routing is assignment, not resolution — assigning an owner does not change the exception status to resolved
- Priority is a recommendation — the analyst confirms before priority is recorded

### Circular Routing Detection

- If an exception has been routed to the same queue more than twice, it is flagged as circular routing
- Circular routing is surfaced prominently for queue manager review
- Circular routing indicates the exception may need reclassification, specialist intervention, or process owner review

### Reopened Exception Flagging

- Reopened exceptions (same transaction reference with a prior closed case) receive elevated visibility
- Reopened exceptions receive an elevated handling path since a prior resolution attempt failed
- Reopen count is tracked in the exception tracker for process improvement analysis

### Data Sensitivity

- Customer PII is never included in broad queue or channel updates — scoped to need-to-know communications
- Transaction details are scoped to what the assigned analyst needs for resolution
- Financial transaction amounts above the defined threshold trigger team lead review

### Audit Trail

- Every exception case records the source system, intake actor, and creation timestamp
- Every classification records the exception type, confidence level, evidence basis, and classifying analyst
- Every priority assessment records the recommended priority, policy criteria used, and assessing analyst
- Every routing decision records the assigned owner, queue, routing rationale, and timestamp
- Every override (classification, priority, or routing) records the original recommendation, analyst's decision, and rationale
- The Excel exception tracker serves as the Cowork-accessible audit record

## Exception Types

| Type | Definition | Key Indicators |
|------|-----------|----------------|
| **Processing error** | Error in standard transaction processing | Failed validation, incorrect calculation, missing step, timeout |
| **Data mismatch** | Discrepancy between systems or data sources | Conflicting values, missing records, out-of-sync state |
| **System failure** | Technical infrastructure or application failure | System error codes, connectivity loss, application crash |
| **Manual override needed** | Standard processing cannot handle the case | Edge case, exception to rules, non-standard transaction |
| **Compliance exception** | Regulatory or policy violation detected | Policy threshold breach, regulatory flag, audit finding |
| **Quality defect** | Product or service quality issue | Customer complaint, inspection failure, specification deviation |

## Priority Levels

| Priority | Criteria | SLA Deadline | Handling Path |
|----------|---------|-------------|---------------|
| **Critical** | Safety exception, compliance violation, major customer impact, process line blocked, financial exposure above threshold | 30 minutes | Immediate resolution + team lead notification |
| **High** | Multiple transactions affected, customer-facing impact, approaching regulatory deadline, dependent team blocked | 2 hours | Priority queue + specialist if needed |
| **Medium** | Single transaction affected, internal process delay, no immediate customer impact, workaround available | 4 hours | Standard queue processing |
| **Low** | Minor discrepancy, no downstream impact, informational exception, can be batched | 1 business day | Batched processing |

## Communication Types

| Type | Purpose | Channel |
|------|---------|---------|
| **Missing information request** | Request specific data needed for resolution | Outlook draft |
| **Shift handoff summary** | Transfer case ownership to incoming shift | Teams queue channel |
| **Escalation notice** | Alert team lead to escalation-requiring case | Outlook draft + Teams |
| **Status update** | Inform stakeholder on resolution progress | Outlook draft |
| **Batch queue summary** | Overview of open cases, aging, and SLA compliance | Teams management channel |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find exception tracker, SOPs, taxonomy, routing rules, priority rules, escalation checklist, templates, prior cases |
| SearchM365 (connectors) | Context Packet, Classify Exception, Impact Assess — retrieve transaction details, queue state, and processing history via Graph Connector |
| SearchM365 (email) | Exception Intake, Context Packet — find exception notifications, stakeholder correspondence, handoff emails |
| SearchM365 (teams) | Exception Intake, Context Packet — find coordination threads and queue discussions |
| ReadFileContent | All skills — read tracker, SOPs, taxonomy, routing rules, priority rules, templates, queue definitions |
| GetDriveChildren | Context Packet — browse case workspace for supporting documents |
| SearchPeople / GetUserDetails | Exception Intake, Route Exception — resolve analyst and resolver assignments |
| GetManagerDetails | Route Exception — resolve team lead for escalation paths |
| PostMessage | Exception Intake, Route Exception, Exception Comms — Teams notifications to queue channels, resolvers, and team leads |
| CreateDraftMessage | Exception Comms — formal communications as Outlook drafts |
| CreateEvent | Route Exception — SLA deadline calendar holds |
| render_ui (Adaptive Card) | Context Packet, Classify Exception, Impact Assess, Route Exception, Exception Comms — in-session decision surfaces |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Exception case record | Excel tracker row | Exception Intake |
| Context packet | Adaptive Card | Context Packet |
| Classification recommendation | Adaptive Card | Classify Exception |
| Impact assessment | Adaptive Card | Impact Assess |
| Routing recommendation | Adaptive Card | Route Exception |
| Assignment notifications | Teams direct messages | Route Exception |
| SLA deadline holds | Calendar events | Route Exception |
| Missing information requests | Outlook draft emails | Exception Comms |
| Shift handoff summaries | Teams channel messages | Exception Comms |
| Escalation notices | Outlook draft emails + Teams messages | Exception Comms |
| Status updates | Outlook draft emails | Exception Comms |
| Batch queue summaries | Teams channel messages | Exception Comms |

## Federated Data Access

Operational exception handling depends on workflow platforms, transaction systems, and work queue data. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Workflow platforms (ServiceNow, BMC Helix, Pega) for queue state and work item history; transaction systems (ERP, CRM, order management) for transaction details and processing history; near-real-time indexing recommended for high-priority queues |
| **Tier 2** | SharePoint Bridge | SOPs, exception taxonomy, routing rules, priority rules, escalation checklists, and queue definitions maintained in SharePoint; Power Automate flows push queue snapshots on 15-minute refresh cycles |
| **Tier 3** | Manual Input | Verbal context from phone calls, physical inspection results, manual override rationale captured via structured intake prompts with "manual entry" source tagging |

**Recommended pilot approach:** Start with Tier 2 (SharePoint for SOPs, routing rules, and queue data with frequent Power Automate refreshes) and Tier 3 for context not in systems. Introduce Graph Connectors for the primary workflow platform in Wave 2 because real-time queue state improves routing accuracy and SLA compliance.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── ops-exception-intake/SKILL.md
├── ops-context-packet/SKILL.md
├── ops-classify-exception/SKILL.md
├── ops-impact-assess/SKILL.md
├── ops-route-exception/SKILL.md
└── ops-exception-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Exception tracker** — shared Excel workbook (Case ID, Source System, Process Type, Queue, Transaction Ref, Exception Type, Priority, Status, Created Date, Assigned Analyst, Aging Hours, SLA Deadline, Reopen Count, Safety Flag)
- **Exception taxonomy** — classification categories, definitions, and decision criteria
- **SOPs and resolution guides** — standard operating procedures by process type and exception type
- **Routing rules** — assignment rules by exception type, queue, priority, and function
- **Queue definitions** — queue membership, ownership, capacity, and operating hours
- **Priority rules** — priority definitions, criteria, thresholds, and SLA targets
- **Escalation checklist** — escalation triggers, approval requirements, and notification rules
- **Communication templates** — handoff summary format, escalation notice template, status update template
- **Operating model** — team structure, specialization areas, and queue ownership

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Exception Intake | "new exception case", "log operational exception", "exception from [system]", "intake exception for [process]" |
| Context Packet | "build context for exception [ID]", "what happened with [transaction]", "assemble case context", "gather exception details" |
| Classify Exception | "classify this exception", "what type of exception is [case ID]", "categorize the issue", "what caused [transaction] to fail" |
| Impact Assess | "assess impact of exception [ID]", "how urgent is [case]", "priority recommendation", "aging risk for [case]" |
| Route Exception | "route exception [ID]", "assign this case", "who handles [exception type]", "send to [team] queue", "escalate exception" |
| Exception Comms | "draft follow-up for exception [ID]", "prepare handoff summary", "exception status update", "handoff to next shift", "batch queue summary" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel exception tracker in SharePoint with standard columns
- Upload SOPs, exception taxonomy, queue definitions, routing rules, priority rules, and escalation checklists to SharePoint
- Set up Power Automate flows to refresh transaction and queue data on 15-minute cadence
- Create dedicated Teams channels for each major operations queue
- Build skills: `ops-exception-intake`, `ops-context-packet`, `ops-classify-exception`, `ops-exception-comms`
- Operate in AI assist mode — all outputs presented via Adaptive Card for analyst review
- Test with 15-20 real exception cases across different types and queues

### Wave 2 — Priority Assessment and Controlled Routing

- Build skills: `ops-impact-assess`, `ops-route-exception`
- Promote intake to write mode (creates case records after confirmation)
- Promote classification to write-back mode (updates tracker after analyst confirmation)
- Promote routing to active mode (sends Teams messages and creates calendar holds after confirmation)
- Set up scheduled prompt: check every 30 minutes for cases approaching SLA deadline with unassigned owners or stale status
- Add SOP-cited resolution guidance based on exception type
- Measurement targets: above 80% classification accuracy, above 85% correct routing rate

### Wave 3 — Advanced Assembly and Proactive Detection

- Introduce Graph Connectors for workflow platform and transaction system for real-time access
- Add bounded multi-source exception packet assembly for complex cases
- Add proactive detection: flag repeat exception types suggesting process defects, identify queues with chronic aging, highlight analysts with high override rates, surface circular routing patterns
- Add shift handoff automation: scheduled prompt at shift boundaries generates batch queue summary
- Measurement targets: 40% time-to-triage reduction, above 80% classification accuracy, above 85% correct routing rate, below 20% override rate, below 10% reopen rate, 100% safety exception detection rate

## Implementation Notes

- **Triage disposition is intentionally not automated** — final escalation or closure decision carries operational accountability and cannot be delegated to AI
- **Safety-first is a permanent architectural constraint** — quality defects, compliance violations, and equipment failures are always classified and routed at the more severe level when evidence is ambiguous
- **The workflow platform is the source of truth** — the Excel exception tracker is a Cowork-accessible working copy; state drift is mitigated by Power Automate sync
- **No skill may auto-close, auto-resolve, or auto-escalate** — these are irreversible disposition changes requiring human decision with documented rationale
- **Adaptive Card is the primary output format** — the 30-minute SLA demands speed; Adaptive Cards are faster than document generation and structured for analyst action
- **Routing rules, priority rules, and escalation logic live in SharePoint** — operations leadership can update procedures without modifying skill code
- **Every triage decision is logged** — classification, priority, routing, and override records support process improvement analysis
- **Circular routing is detected and flagged** — cases bouncing between the same queues indicate process issues that need human intervention
- **Reopened exceptions are never treated as new cases** — they receive elevated handling because a prior resolution attempt failed
