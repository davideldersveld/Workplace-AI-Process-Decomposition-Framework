# Plan: Operations Exception Intake and Resolution Routing — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Operations line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Operational Exception Intake and Resolution Routing pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Operations, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — Operations exception handling scores high on volume and data readiness; the 30-minute SLA target confirms urgency |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — separating classification (OPS-003) from priority assessment (OPS-004) from routing (OPS-005) prevents a monolithic triage skill that conflates distinct decision types |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call | Requires translation — Operations references "workflow platform", "transaction system", and "work queue" that must become Graph Connector queries or SharePoint bridge reads |
| **Automation Boundary** | Operating mode per step | **Guardrails and confirmation gates** in SKILL.md | Strong — Operations adds safety considerations for exception handling in environments where misrouting can have safety or compliance consequences; guardrails must reflect this |
| **Capability Mapping** | AI task pattern per step | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — classification and priority are Decision Support; context assembly is Data Aggregation; handoff drafting is Content Generation |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — the 30-minute SLA means skills must surface findings quickly via Adaptive Cards |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — Cowork provides the runtime; the skill author controls capability definition and decision logic |

### Key Insight

Operations exception handling is the most queue-driven of the three LOBs. The framework's decomposition is essential here because operations teams process high volumes of exceptions that share a common structure (intake, classify, prioritize, route, communicate) but vary widely in content. The skill design must handle this variety through dynamic policy reads (SharePoint SOPs and routing rules) rather than hard-coded logic. The 30-minute SLA also means skills must prioritize speed of presentation over depth of analysis — Adaptive Cards and Teams messages are the primary output, not documents.

---

## Part 2: The Operations Exception Routing Plugin — Skill-by-Skill Design

The Operations sample decomposes "Operational Exception Intake and Resolution Routing" into 7 steps (OPS-001 through OPS-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| OPS-001: Normalize exception event | `ops-exception-intake` | Data Aggregation | Deterministic automation | Excel (tracker), SharePoint (list), Teams (notification) |
| OPS-002: Gather process and transaction context | `ops-context-packet` | Data Aggregation | AI act within policy | Excel (context data), SharePoint (SOPs, queue rules), Teams (coordination) |
| OPS-003: Classify exception type and likely cause | `ops-classify-exception` | Decision Support | AI assist | Adaptive Card (classification), Excel (tracker update) |
| OPS-004: Assess impact and aging risk | `ops-impact-assess` | Decision Support | AI draft + approve | Adaptive Card (impact report), Excel (priority update) |
| OPS-005: Assign owner or queue | `ops-route-exception` | Decision Support | AI act within policy | Teams (assignment messages), Excel (owner update), Calendar (SLA deadlines) |
| OPS-006: Draft follow-up or handoff summary | `ops-exception-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (updates), Word (handoff summary) |
| OPS-007: Confirm triage disposition | *Not a skill — human decision step* | N/A | Human only | Teams (confirmation), Outlook (sign-off) |

### Detailed Skill Designs

#### 1. `ops-exception-intake` — Normalize Exception Event

**Framework Step:** OPS-001

**Trigger phrases:** "new exception case", "log operational exception", "exception from [system]", "intake exception for [process]", "queue exception for [team]"

**Inputs:**
- Exception source (transaction processing, service delivery, quality check, manual intake)
- Process type or queue name
- Transaction or case reference
- Initial description or error detail

**M365 tools:**
- `SearchM365(sources=["email"])` — find related exception notification or handoff email
- `SearchM365(sources=["files"])` — check existing tracker for duplicate exception on the same transaction reference
- `ReadFileContent` — read current exception tracker to validate no duplicate exists
- `SearchPeople` — resolve operations analyst assignment based on queue or process type
- `SearchM365(sources=["connectors"], connector_ids=["workflow-connector"])` — pull source system record if Graph Connector is available

**Output:** Structured exception record written to Excel tracker in SharePoint; Teams notification to assigned analyst and queue channel

**Artifact:** Excel workbook with columns: Case ID, Source System, Process Type, Queue, Transaction Ref, Exception Type, Priority, Status, Created Date, Assigned Analyst, Aging (hours), SLA Deadline, Reopen Count

**Guardrails:**
- Never create duplicate cases for the same transaction reference and exception date
- Validate that the process type matches the organization's queue taxonomy
- Post Teams notification to the operations queue channel upon case creation
- Log creation with actor and timestamp for audit trail
- Flag reopened exceptions (same transaction reference with a prior closed case) with elevated visibility

---

#### 2. `ops-context-packet` — Gather Process and Transaction Context

**Framework Step:** OPS-002

**Trigger phrases:** "build context for exception [ID]", "what happened with [transaction]", "assemble case context for [exception]", "gather exception details"

**Inputs:**
- Exception case ID or transaction reference

**M365 tools:**
- `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — pull transaction details, processing history, and error logs
- `SearchM365(sources=["connectors"], connector_ids=["workflow-connector"])` — pull queue state, prior assignments, and work item history
- `SearchM365(sources=["files"])` — locate SOPs, resolution guides, and queue-specific procedures in SharePoint
- `ReadFileContent` — read SOP and routing rules from SharePoint
- `SearchM365(sources=["email"])` — find related correspondence (stakeholder notes, prior handoff emails)
- `GetDriveChildren` — check for supporting documents uploaded to the case folder (screenshots, forms, reports)
- `SearchM365(sources=["files"])` — prior similar exceptions for pattern context

**Output:** Structured context presented via Adaptive Card:
- Transaction details (type, parties, amounts, dates, error state)
- Queue and work item history (prior assignments, aging, reopen count)
- Similar prior exceptions and their resolutions
- Applicable SOP or resolution guide reference
- Supporting documents inventory

**Artifact:** Adaptive Card for quick review; Excel tracker updated with context summary fields

**Guardrails:**
- Cite data source and timestamp for every transaction detail
- Flag if source system data is unavailable or stale
- Never modify transaction records or queue state — read-only access only
- For safety-related exceptions (quality defects, compliance violations, equipment failures), flag immediately with elevated visibility regardless of other context

---

#### 3. `ops-classify-exception` — Classify Exception Type and Likely Cause

**Framework Step:** OPS-003

**Trigger phrases:** "classify this exception", "what type of exception is [case ID]", "categorize the issue", "what caused [transaction] to fail"

**Inputs:**
- Context packet data (from `ops-context-packet`)
- Exception taxonomy (SharePoint document)

**M365 tools:**
- `ReadFileContent` — exception taxonomy and classification rules from SharePoint
- `SearchM365(sources=["files"])` — prior cases with similar characteristics for pattern matching
- `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — additional transaction history for root cause analysis

**Output:** Classification presented via Adaptive Card:
- Exception type (processing error, data mismatch, system failure, manual override needed, compliance exception, quality defect)
- Likely cause hypothesis with supporting evidence
- Confidence level (high, medium, low) based on available data
- Similar prior cases and their resolutions for pattern context
- Recommended resolution approach based on exception type

**Guardrails:**
- Present classification as a recommendation, not a final determination — the analyst confirms or corrects
- Never auto-update the exception type in the tracker without analyst review
- Include the evidence basis so the analyst can validate the classification
- Flag low-confidence classifications explicitly for human judgment
- For safety-critical exception types (quality defects, compliance violations, equipment failures), apply a "safety-first" rule: always classify as the more severe type when evidence is ambiguous, and require explicit analyst downgrade with documented rationale
- Log classification and any override for process improvement analysis

---

#### 4. `ops-impact-assess` — Assess Impact and Aging Risk

**Framework Step:** OPS-004

**Trigger phrases:** "assess impact of exception [ID]", "how urgent is [case]", "priority recommendation for [exception]", "aging risk for [queue]", "what's the exposure on this"

**Inputs:**
- Classified exception data
- Context packet with transaction and queue details
- Priority rules and operational policy (SharePoint document)

**M365 tools:**
- `ReadFileContent` — priority rules, escalation checklist, and SLA definitions from SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — downstream impact data (affected processes, dependent transactions)
- `SearchM365(sources=["files"])` — prior case resolutions for similar priority levels

**Output:** Impact assessment presented via Adaptive Card:
- Priority recommendation (critical, high, medium, low) with policy citation
- Aging risk indicator (time since creation versus SLA target)
- Downstream impact summary (affected processes, dependent transactions, customer or stakeholder exposure)
- Recommended handling path (immediate resolution, standard queue, specialist referral, escalation)
- Escalation recommendation if aging or impact exceeds threshold

**Guardrails:**
- Present impact assessment and priority as a recommendation requiring analyst approval — never auto-set priority
- Never auto-close, auto-resolve, or auto-reroute based on impact assessment alone
- For safety-critical exceptions, always recommend escalation regardless of calculated priority level — safety exceptions bypass standard priority logic
- Flag any exception aging beyond 80% of its SLA window with explicit urgency callout
- Include the priority rules criteria used so the analyst can validate the recommendation
- For exceptions involving financial transactions above a defined threshold, flag for team lead review

---

#### 5. `ops-route-exception` — Assign Owner or Queue

**Framework Step:** OPS-005

**Trigger phrases:** "route exception [ID]", "assign this case", "who handles [exception type]", "send to [team] queue", "escalate exception"

**Inputs:**
- Impact assessment output
- Routing rules and operating model (SharePoint document)
- Exception case data including classification and priority

**M365 tools:**
- `ReadFileContent` — routing rules, queue definitions, and team ownership model from SharePoint
- `SearchPeople` — resolve owner by exception type, queue, or function
- `GetUserDetails` — verify owner availability and role
- `PostMessage` — Teams message to assigned owner or queue channel with case summary and next action
- `CreateEvent` — calendar reminder for SLA deadline based on priority level

**Output:** Routed exception:
- Teams message to assigned owner or queue channel with structured case summary, priority, classification, and expected next action
- Calendar hold for SLA deadline (30 minutes for critical, 2 hours for high, 4 hours for medium, 1 business day for low)
- Excel tracker updated with assigned owner, queue, and routing timestamp

**Guardrails:**
- Route only to individuals or queues listed in the documented routing rules — never assign outside the operating model
- For critical-priority exceptions, simultaneously notify the team lead via Teams in addition to the assigned resolver
- For safety-critical exceptions (quality defects, compliance violations, equipment failures), route to the designated safety or compliance queue regardless of standard routing rules, and notify the operations manager
- Escalate to queue manager if no clear owner is found in the routing rules
- Log routing decision with rationale for audit and process improvement
- Never auto-close or auto-resolve a case through routing — routing is assignment, not resolution
- Prevent circular routing — if a case has been routed to the same queue more than twice, flag for queue manager review

---

#### 6. `ops-exception-comms` — Draft Follow-Up or Handoff Summary

**Framework Step:** OPS-006

**Trigger phrases:** "draft follow-up for exception [ID]", "prepare handoff summary for [case]", "exception status update", "handoff to next shift", "missing information request for [case]"

**Inputs:**
- Case data from Excel tracker
- Context packet
- Impact assessment
- Target audience (operations analyst, specialist resolver, team lead, stakeholder, next-shift team)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts for formal communications (never auto-send)
- `PostMessage` — Teams updates to queue channels (after confirmation)
- `SearchM365(sources=["files"])` — communication templates and handoff formats from SharePoint
- `ReadFileContent` — standard update and handoff templates

**Communication templates:**
- Missing information request to source team or stakeholder
- Handoff summary for shift change (case status, open actions, pending items, SLA status)
- Escalation notice to team lead with full case context and aging data
- Status update to stakeholder with resolution progress and expected timeline
- Batch queue summary for queue manager (open cases, aging distribution, SLA compliance)

**Guardrails:**
- Always create formal communications as Outlook draft — never send without explicit user confirmation
- Teams channel updates for operational coordination may be posted after user confirmation
- Match urgency tone to priority level: critical cases use direct, action-oriented language with explicit SLA callouts
- Include case reference number, priority, and aging status in every communication
- For safety-critical exceptions, include safety classification prominently in all communications and never omit it for brevity
- Handoff summaries must include all open action items and pending dependencies — never summarize away unresolved items
- Never include personally identifiable customer information in broad queue or channel updates — scope to need-to-know

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** OPS-007

This is not a Cowork skill. The framework correctly identifies final disposition — especially escalation decisions and exception closure — as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the escalation review meeting if needed
- The exception context and impact artifacts — provide the evidence package for the team lead
- The `ops-exception-comms` skill — send the disposition confirmation and closure communication after the human decision is made

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

**Process decomposition prevents mega-skills.** Operations exception handling looks deceptively simple (intake, classify, route, close) but each step involves different data sources, different decision criteria, and different error consequences. Misclassification wastes resolver time; incorrect priority delays critical work; misrouting creates circular handoffs. The framework forces these apart into testable, individually measurable skills.

**Automation boundary assignment is essential for safety-critical environments.** Many operational exceptions touch safety, compliance, or quality systems where incorrect automated action could have serious consequences. The framework's boundary analysis correctly restricts impact assessment to "AI draft plus approve" and keeps final disposition as "human only." For Operations, this is not just good practice — it is a safety requirement.

**Signal inventory forces explicit tool selection.** Operations signals span workflow platforms, transaction systems, and work queues. The framework requires naming every input source, which prevents vague skill instructions and ensures each data dependency is explicitly mapped to an MCP tool call or Graph Connector.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — Adaptive Cards and confirmation patterns provide human-in-the-loop |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation through quality rubric and manual testing |

**The key gap:** Operations' 30-minute SLA for standard exceptions and the high volume of cases make the state management gap more pronounced. The Excel tracker must support rapid reads and writes, and skills must present findings via Adaptive Card immediately. For high-volume operations teams, the tracker may need to be a SharePoint list rather than an Excel workbook for better concurrent access.

### 3.3 M365 Artifacts as First-Class Process State

Operations' artifact pattern is **Excel + SharePoint + Teams** as the primary trio:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (exception tracker, priority, owner, status, SLA timestamps, aging) | The exception tracker IS the process state — skills read current status, write classifications, record assignments, track SLA compliance, and monitor aging |
| **SharePoint** | Source of truth (SOPs, queue rules, routing rules, resolution guides, priority policies, exception taxonomy) | Skills read policies and procedures dynamically at runtime; supporting evidence documents are stored here; SharePoint lists may replace Excel for high-concurrency environments |
| **Teams** | Real-time coordination channel, routing mechanism, and shift handoff surface | Skills post exception assignments, queue status updates, escalation notices, and shift handoff summaries; Teams is the primary communication surface for queue-based operations |
| **Adaptive Card** | In-session decision support (classifications, impact assessments, routing recommendations) | The primary output format for time-sensitive operational decisions — faster than document generation, structured for analyst action |
| **Outlook** | Formal communication channel (stakeholder updates, cross-team requests, escalation notices) | Skills draft formal communications as reviewable Outlook drafts; read incoming exception notifications and stakeholder correspondence |
| **Calendar** | SLA tracking and deadline management | Skills create SLA deadline reminders based on priority level and queue rules |
| **Word** | Formal handoff documents (shift summaries, escalation packets, batch queue reports) | Used for formal documentation when a case requires shift handoff or escalation review; less frequent than Adaptive Card or Teams output |
| **Graph API** | People and org data (analyst assignments, team structure, queue ownership) | Skills resolve owners by exception type, queue, or function |

**The design pattern:** Operations favors speed and volume throughput. The primary output format is Adaptive Card (for in-session analyst decisions) and Teams messages (for queue coordination and shift handoffs). Excel tracks all case state, but SharePoint provides the dynamic policy layer (SOPs, routing rules, priority definitions) that skills read at runtime. This allows operations leadership to update procedures without modifying skill code.

### 3.4 Federated Connectors for Third-Party Systems

The Operations source file references systems that do not exist natively in M365: workflow platforms, transaction systems, work queues, and reporting layers.

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For workflow platforms (ServiceNow, BMC Helix, Pega), transaction systems (ERP, CRM, order management), and work queue platforms (custom ticketing, case management), Graph Connectors index exception events, transaction records, and queue states into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector", "erp-transaction-connector"])`. For operations, real-time queue state matters — Graph Connectors should be configured for near-real-time indexing on high-priority queues.

For environments with IoT, SCADA, or MES (Manufacturing Execution System) integration, these systems typically feed into the workflow or transaction platform first, and the Graph Connector indexes from that intermediate layer rather than directly from the operational technology.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, Power Automate flows push queue snapshots, transaction records, and exception data to SharePoint lists or Excel workbooks. For operations, the refresh cadence must match the SLA: 30-minute SLA requires at minimum 15-minute refresh cycles for queue state data.

**Tier 3 — Manual Input with Templates**

For data that cannot be automated (verbal context from phone calls, physical inspection results, manual override rationale), skills prompt the analyst for specific data points. The structured intake captures data into the Excel tracker for downstream use.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint bridge with frequent Power Automate refreshes) and Tier 3 (manual input for context not in systems). Prioritize Graph Connectors for the primary workflow platform (ServiceNow, Pega) in Wave 2 because real-time queue state improves routing accuracy and SLA compliance.

### 3.5 Governance in Cowork

Operations governance concerns center on safety compliance, audit integrity, and queue control:

| Governance Domain | Cowork Implementation | Operations-Specific Concern |
|---|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure | Operations manager owns the skill suite; queue managers own queue-specific routing rules |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC | Transaction details may contain customer PII or financial data — skills must scope visibility to need-to-know |
| **Safety compliance** | Encoded as guardrails with explicit safety-first rules | Quality defects, equipment failures, and compliance violations must always be flagged for safety review regardless of calculated priority; skills must never auto-classify safety exceptions as low priority |
| **Regulatory requirements** | Guardrails flag regulated processes and compliance-sensitive exceptions | For exceptions in regulated processes (financial services, healthcare, manufacturing quality), skills must flag regulatory reporting requirements and never auto-close without documented compliance review |
| **Audit** | Platform logs tool invocations; Excel tracker provides case history | Every triage decision (classification, priority, routing, escalation, closure) must be traceable to an actor and timestamp; the tracker serves as the audit log; circular routing and reopens must be logged |
| **Queue integrity** | Guardrails prevent unauthorized queue manipulation | Skills must not allow cases to be moved between queues without documented routing rules; circular routing detection prevents cases from bouncing indefinitely |
| **Irreversible actions** | Guardrails require human approval for all irreversible operations | No skill may auto-close, auto-resolve, or auto-escalate — these are irreversible disposition changes that require human decision with documented rationale |
| **Release management** | Skills versioned in OneDrive; SOPs and routing rules in SharePoint | Queue rules, priority definitions, escalation checklists, and exception taxonomies should live in SharePoint documents read dynamically so operations leadership can update them without modifying skills |

### 3.6 Evaluation Approach

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones?
- Classification accuracy — does `ops-classify-exception` assign the correct exception type? Target: above 80%
- Impact assessment quality — does `ops-impact-assess` identify the correct priority and aging risk?
- Routing accuracy — does `ops-route-exception` assign to the correct queue or owner?
- Tool success rate — do M365 tool calls and Graph Connector queries return expected results?

**Process-level evaluation (end-to-end):**
- Classification accuracy — percentage of cases correctly classified without override
- Correct routing rate — percentage of cases routed to the right owner or queue on the first attempt
- Time to triage — time from case creation to routed status (target: under 30 minutes for standard exceptions)
- Aging reduction — percentage improvement in time-to-resolution for high-priority exceptions
- Reopen rate — percentage of cases reopened after initial closure (indicates resolution quality)
- Override rate — how often analysts override skill recommendations
- Duplicate-handling rate — percentage of duplicate exceptions caught at intake
- Circular routing rate — how often cases are routed to the same queue more than twice
- Safety exception detection rate — percentage of safety-critical exceptions correctly flagged and routed to safety queue

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel exception tracker workbook in SharePoint with standard columns (Case ID, Source System, Process Type, Queue, Transaction Ref, Exception Type, Priority, Status, Created Date, Assigned Analyst, Aging Hours, SLA Deadline, Reopen Count, Safety Flag)
- Upload SOPs, exception taxonomy, queue definitions, routing rules, priority rules, and escalation checklists to a dedicated SharePoint document library
- Set up Power Automate flows to refresh transaction and queue data in SharePoint (Tier 2 bridge) on a 15-minute cadence
- Create dedicated Teams channels for each major operations queue

**Skills to build:**
- `ops-exception-intake`
- `ops-context-packet`
- `ops-classify-exception`
- `ops-exception-comms` (draft mode only)

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs presented via Adaptive Card for analyst review. Test with 15-20 real exception cases across different types and queues.

### Wave 2 — Priority Assessment and Controlled Routing

**Skills to build:**
- `ops-impact-assess`
- `ops-route-exception`

**Promotions:**
- Promote `ops-exception-intake` to write mode (creates case records after confirmation)
- Promote `ops-classify-exception` to write-back mode (updates tracker after analyst confirmation)
- Promote `ops-route-exception` to active routing (sends Teams messages and creates calendar holds after confirmation)

**Automation:**
- Set up a scheduled prompt that runs every 30 minutes checking for exception cases approaching SLA deadline and surfaces any with unassigned owners, stale status, or circular routing patterns
- Add SOP-cited resolution guidance that references specific procedure sections based on exception type

**Operating posture:** AI draft plus approve for impact assessment and communications. AI act within policy for routing (bounded by documented routing rules). Every priority recommendation reviewed before assignment.

### Wave 3 — Advanced Assembly and Proactive Detection

**Enhancements:**
- Introduce Graph Connectors for workflow platform and transaction system for real-time access
- Add bounded multi-source exception packet assembly — skill pulls from transaction, queue, history, and document data to produce a complete exception brief for complex cases
- Add proactive detection of operational patterns via scheduled prompt: flag repeat exception types suggesting process defects, identify queues with chronic aging, highlight analysts with high override rates suggesting skill refinement needs, surface circular routing patterns
- Add shift handoff automation: scheduled prompt at shift boundaries generates a batch queue summary for the incoming team with open cases, aging distribution, and pending escalations
- Refine all skills based on override patterns and analyst feedback from Waves 1-2

**Measurement:**
- Time to triage reduction versus pre-pilot baseline (target: 40% improvement)
- Classification accuracy target: above 80%
- Correct routing rate target: above 85%
- Override rate target: below 20%
- Reopen rate target: below 10%
- Aging reduction target: 30% improvement for high-priority cases
- Safety exception detection target: 100% (zero misses is the target for safety-critical items)

---

## Part 5: Generalizing the Approach — Operations' Artifact Pattern

Operations' primary M365 artifact pattern is **Excel + SharePoint + Teams**. This reflects three characteristics of operations exception workflows:

1. **High-volume structured tracking** — exception cases, priority levels, owner assignments, SLA timestamps, and aging metrics are tabular and live in Excel (or SharePoint lists for high-concurrency environments); the tracker is the central process state artifact
2. **Dynamic policy layer** — SOPs, routing rules, exception taxonomies, and priority definitions live in SharePoint and are read dynamically by skills at runtime; this allows operations leadership to update procedures without modifying skill code
3. **Real-time queue coordination** — operations teams work in queues with SLA pressure; Teams is the primary surface for assignment notifications, escalation alerts, shift handoffs, and queue status updates

This pattern shares characteristics with Supply Chain (speed, Teams-dominant communication, queue-based) and Procurement (structured tracking, document-heavy policies). Operations' unique contribution is the emphasis on **queue throughput and SLA compliance** as primary metrics, and the need for **safety-first guardrails** that override standard priority logic for safety-critical exceptions. The cross-LOB method remains the same: decompose the process, assign automation boundaries, map signals to M365 tools, identify the primary artifact, and build Wave 1 as read-only.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint |
| Process State | SharePoint-hosted Excel workbook or SharePoint list | Durable state externalized to M365 artifacts; SharePoint list preferred for high-concurrency queues |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, workflow event via Graph Connector, queue update |
| Approval | Adaptive Card presentation + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via triage time, routing accuracy, and SLA compliance |
