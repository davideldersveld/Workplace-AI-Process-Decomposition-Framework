# Plan: Compliance and Risk Policy Exception and Case Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Compliance and Risk line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Policy Exception and Compliance Case Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Compliance and Risk, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Compliance's low error tolerance and strong approval structure elevate the importance of audit trail and guardrails. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope principle. Compliance's evidence-centric steps decompose cleanly along intake-analysis-routing-communication boundaries. |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "GRC platform", "case management platform", and "policy repository" that must become specific M365 tool names or SharePoint-bridged data stores. |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Compliance's regulatory sensitivity demands strict guardrails. Risk judgments, policy interpretation, and formal approvals must remain explicitly human-owned. |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — risk indicator detection maps to Decision Support; case summary drafting maps to Content Generation; evidence assembly maps to Data Aggregation. |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal tool contracts; orchestration is implicit in skill instructions. Compliance's approval gates and audit requirements must be encoded as guardrails and artifact patterns. |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides the runtime layers natively. The skill author controls capability definition and decision logic. Compliance-specific access scoping relies on M365 tenant RBAC and SharePoint permissions. |

### Key Insight

Compliance and Risk workflows are evidence-heavy, approval-bound, and audit-sensitive. The framework's decomposition discipline is especially valuable here because it prevents the common anti-pattern of building one broad "compliance review" skill that conflates intake, risk analysis, routing, and disposition. By decomposing into six steps, each skill has a clear scope, a defined automation boundary, and explicit guardrails that preserve compliance judgment and audit integrity. Every write action must preserve actor, timestamp, prior value, and case linkage — a requirement that shapes every skill's guardrail section.

---

## Part 2: The Compliance and Risk Case Triage Plugin — Skill-by-Skill Design

The Compliance and Risk sample decomposes "Policy Exception and Compliance Case Triage" into 6 steps (CR-001 through CR-006), identifies 6 skills, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| CR-001: Normalize compliance case | `compliance-case-intake` | Data Aggregation | Deterministic automation | Excel (case tracker), SharePoint (list), Outlook (intake emails) |
| CR-002: Gather policy, control, and entity context | `compliance-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), Excel (control ownership), SharePoint (policies), Graph API (people) |
| CR-003: Identify risk indicators and missing evidence | `compliance-risk-detection` | Decision Support | AI assist | Excel (evidence checklist), SharePoint (evidence files), Adaptive Card (risk report) |
| CR-004: Determine review path and severity | `compliance-review-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Excel (severity matrix, routing rules) |
| CR-005: Draft case summary and follow-up requests | `compliance-case-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (case summary) |
| CR-006: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (disposition confirmation) |

### Detailed Skill Designs

#### 1. `compliance-case-intake` — Normalize Compliance Case

**Framework Step:** CR-001

**Trigger phrases:** "new compliance case", "policy exception request from [name]", "log compliance intake", "hotline report received", "open compliance case for", "new control issue from"

**Inputs:**
- Reporter or requestor name/email (may be anonymous for hotline reports)
- Case type (policy exception, suspected violation, control issue, third-party risk)
- Affected policy or control area
- Business unit and legal entity
- Urgency or initial severity assessment
- Supporting documentation (if any)

**M365 tools:**
- `SearchPeople` — resolve reporter and control owner identities (when not anonymous)
- `GetUserDetails` — pull reporter profile and department
- `SearchM365(sources=["email"])` — find the original intake email or form submission
- `ReadFileContent` — read SharePoint-hosted case tracker to check for duplicate or related cases

**Output:** Structured case record written to Excel compliance case tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Case Type, Reporter (or "Anonymous"), Affected Policy, Business Unit, Legal Entity, Severity, Status, Received Date, Assigned Analyst, Evidence Status, Disposition

**Guardrails:**
- Support anonymous reporting — never attempt to identify anonymous reporters
- Never create duplicate cases for the same issue within 30 days without explicit user confirmation
- Validate that all required fields are populated before writing
- Confirm details with user via Adaptive Card before writing to tracker
- Log case creation with timestamp and creating user in the audit column

---

#### 2. `compliance-context-packet` — Gather Policy, Control, and Entity Context

**Framework Step:** CR-002

**Trigger phrases:** "build compliance context for", "assemble case packet", "pull policy context for [case ID]", "what policies apply to [issue]", "prepare evidence packet"

**Inputs:**
- Case ID or case description
- Affected policy or control area

**M365 tools:**
- `GetUserDetails` — control owner and business unit context
- `GetManagerDetails` — reporting chain for escalation awareness
- `SearchM365(sources=["files"])` — policy documents, control standards, prior exception approvals, evidence checklists
- `ReadFileContent` — SharePoint-hosted policies, control documentation, and severity/routing matrices
- `GetDriveChildren` — browse the compliance policy library and evidence folders
- `SearchM365(sources=["connectors"], connector_ids=["grc-connector"])` — GRC platform case history (when Graph Connector available)

**Output:** Word document containing:
- Case summary (type, reporter context, affected area, entity)
- Applicable policy extracts with section references
- Control ownership and accountability chain
- Prior exceptions or findings for the same policy area
- Evidence checklist with completion status
- Required approvals based on case type and severity

**Artifact:** Word (.docx) and Excel evidence checklist, both saved to SharePoint compliance case folder

**Guardrails:**
- Cite policy source, version, and section number for every policy reference
- Never interpret policy intent — present policy text as written with references
- Flag if any required policy document is not found in SharePoint
- Restrict packet access to the compliance team SharePoint group and designated reviewers
- Never include hotline reporter identity in context packets for anonymous reports
- Preserve evidence chain of custody — log every document accessed with timestamp

---

#### 3. `compliance-risk-detection` — Identify Risk Indicators and Missing Evidence

**Framework Step:** CR-003

**Trigger phrases:** "check risk indicators for", "what evidence is missing", "compliance gap check for [case ID]", "risk assessment for case", "evidence status for"

**Inputs:**
- Context packet (Word document or case data)
- Evidence checklist (Excel)

**M365 tools:**
- `ReadFileContent` — evidence checklist, policy documents, and submitted evidence from SharePoint
- `SearchM365(sources=["files"])` — uploaded evidence files, prior findings for the same policy area
- `GetDriveChildren` — case evidence folder contents to check what has been submitted

**Output:** Risk indicator report as Adaptive Card (for quick review) plus Excel checklist update (for tracking)

**Logic:** Compare required evidence checklist items against submitted materials in the case evidence folder. Detect risk indicators: (1) missing mandatory evidence, (2) inconsistencies between reported facts and submitted evidence, (3) policy area has history of repeated exceptions, (4) case involves multiple entities or jurisdictions, (5) regulatory reporting may be required. Classify severity: low, medium, high, critical.

**Guardrails:**
- Present all findings as "indicators" — never characterize findings as conclusions, determinations, or risk judgments
- Every indicator must cite the specific evidence gap or pattern that triggered it
- Never assess intent, fault, or culpability — surface factual gaps and patterns only
- Flag regulatory-reportable indicators (anti-corruption, sanctions, data breach) with elevated visibility and separate callout
- Present findings for user review via Adaptive Card before updating any tracker or checklist
- Read-only mode — this skill does not modify case status or disposition; it only surfaces analysis

---

#### 4. `compliance-review-routing` — Determine Review Path and Severity

**Framework Step:** CR-004

**Trigger phrases:** "route this case for review", "who reviews [case type]", "assign compliance reviewers", "determine severity and routing for", "set up review chain for case"

**Inputs:**
- Risk indicator report output
- Case data (type, severity, entity, policy area)
- Severity matrix and routing rules (SharePoint documents)

**M365 tools:**
- `SearchPeople` — resolve compliance analysts, risk managers, and control owners by function
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `ReadFileContent` — severity matrix, routing rules, and reviewer assignments from SharePoint
- `PostMessage` — Teams notification to assigned reviewers (after approval)
- `CreateDraftMessage` — Outlook draft for formal review assignment or escalation notice (after approval)

**Output:** Routing recommendation:
- Assigned compliance analyst based on case type and policy area
- Required reviewers (legal, audit, control owner) based on severity and risk indicators
- Severity classification with supporting rationale
- Escalation path if severity is high or critical
- SLA deadline based on case type and severity

**Guardrails:**
- Present routing recommendation for compliance operations review before sending any messages or assignments
- Never auto-assign outside the approved reviewer matrix
- Escalate to compliance manager and legal reviewer if severity is high or critical
- Escalate to internal audit liaison if control failure indicators are present
- Never reveal case substance in Teams messages — use case IDs and links to the SharePoint case folder
- Log all routing decisions to the Excel case tracker with timestamp, actor, and rationale
- Preserve segregation of duties — the person who triages should not be the final approver

---

#### 5. `compliance-case-comms` — Draft Case Summary and Follow-Up Requests

**Framework Step:** CR-005

**Trigger phrases:** "draft compliance case summary", "prepare case update for", "send evidence request for [case ID]", "draft follow-up to [control owner]", "compliance case status update"

**Inputs:**
- Case data from Excel tracker
- Context packet (Word document)
- Risk indicator report
- Target audience (compliance reviewer, control owner, legal reviewer, audit liaison, reporter)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages to compliance ops channel (after confirmation)
- `SearchM365(sources=["files"])` — communication templates and standard response language from SharePoint
- `ReadFileContent` — case artifacts for summary content

**Communication templates:**
- Case summary to assigned compliance analyst (risk indicators, evidence status, recommended review areas)
- Evidence request to control owner or business process owner
- Escalation notice to compliance manager or legal reviewer
- Status update to reporter (when not anonymous) — factual status only, no conclusions
- Internal audit referral with control failure indicators
- Regulatory reporting flag notification to compliance leadership

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Never include case conclusions, fault assessments, or risk judgments in any communication — present factual status only
- Never include reporter identity in any communication if the report was anonymous
- Never disclose case details to individuals outside the approved reviewer list
- Match tone to audience: precise and evidence-focused for compliance reviewers, clear and action-oriented for control owners
- Include case reference number in every communication
- Mark all internal compliance communications with appropriate confidentiality markings

---

#### Step 6: Confirm Triage Disposition (Human Only)

**Framework Step:** CR-006

This is not a Cowork skill. The framework correctly identifies final case disposition as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the triage review meeting with compliance leadership
- The case summary artifacts — provide the evidence package for the reviewer
- The `compliance-case-comms` skill — send the disposition confirmation after the human decision is made
- The Excel case tracker — updated to reflect final disposition (queued for review, escalated, referred to legal, referred to audit, or returned for more evidence)

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Compliance and Risk is **structured decomposition with auditability discipline**:

**Process decomposition prevents mega-skills.** The common anti-pattern for compliance AI is building one broad "case review" tool that conflates intake, risk assessment, routing, and disposition. The framework's rule — "keep breaking down until each step has one dominant goal" — produces well-scoped skills that separate evidence gathering from risk analysis from routing from communication. This separation is essential for audit trail integrity and segregation of duties.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions. Risk detection operates here. |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation. Routing and communications operate here. |
| AI act within policy | Skill can execute bounded read and retrieval actions within defined rules. Context assembly operates here. |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed. Case intake normalization operates here. |

**Signal inventory forces explicit M365 tool selection.** Instead of vague references to "GRC platform" or "case management platform," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365`, `ReadFileContent`, `GetDriveChildren` — a direct quality improvement for compliance skills where evidence traceability is non-negotiable.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which process step is active |
| Capability registry | Built-in — skills are discovered and versioned in the skill directory |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal compliance approval workflow engine |
| Governance and control | Partial — skill instructions encode policies; M365 tenant RBAC enforces access boundaries; no built-in audit event logging beyond tool invocation logs |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine or a formal audit event log. Compliance case state (pending reviews, evidence status, severity classifications, approval decisions) must be externalized to M365 artifacts. The Excel case tracker and SharePoint case folders serve this role, but every skill must explicitly log state changes with actor, timestamp, and prior value.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the compliance case triage process:

| Artifact | Role in Compliance Workflow | How Skills Use It |
|---|---|---|
| **Excel** (PRIMARY) | Case tracker (status, severity, assignments, evidence status, disposition), evidence checklists, severity matrices, routing rules | The canonical process state store — skills read current case status, update evidence completion, and log routing decisions. Excel is the primary structured data artifact. |
| **SharePoint** (PRIMARY) | Source of truth for policies, control standards, evidence documents, case folders, routing matrices, prior exception records | Skills read authoritative policy documents and store all case artifacts in per-case folders. SharePoint permissions enforce access control. |
| **Word** | Context packets, case summaries, policy extracts, evidence narratives | Skills generate documents that become the auditable record of what was assembled and reviewed. Word serves as the evidence narrative layer. |
| **Outlook** | Communication channel, intake signal, draft outreach | Skills read incoming compliance requests; draft all reviewer-facing and control-owner-facing communications as reviewable Outlook drafts |
| **Teams** | Internal compliance coordination and status routing | Skills post task assignments and escalation notices to compliance channels after confirmation |
| **Adaptive Card** | Quick-review presentation for risk indicators and routing recommendations | Skills present analysis for rapid review without requiring the user to open a full document |
| **Graph API** | People data — control owners, compliance analysts, org hierarchy | Skills resolve people for routing decisions, escalation paths, and control ownership |
| **Calendar** | Review meetings, SLA deadlines, evidence submission deadlines | Skills create calendar events for review milestones and SLA-driven deadlines |

**The design pattern:** The Cowork Compliance plugin uses Excel and SharePoint as the dual backbone — Excel workbooks provide structured case tracking, evidence checklists, and severity matrices, while SharePoint hosts the policy library and per-case evidence folders. Word documents serve as the narrative evidence layer (context packets, case summaries). Every skill reads from and writes to this shared ecosystem through M365 tools, with explicit audit logging on every state change.

### 3.4 Federated Connectors for Third-Party Systems

The framework references GRC platforms, case management platforms, and policy repositories. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For GRC platforms (ServiceNow GRC, Archer, OneTrust, LogicGate), Graph Connectors index compliance case metadata, control records, and finding history into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["grc-connector"])`. This provides read access to case status, control ownership, and prior findings without custom integration code.

For regulatory intelligence platforms (Thomson Reuters Regulatory Intelligence, LexisNexis), Graph Connectors can index regulatory updates for cross-referencing during case triage. Skills can surface whether a policy area has pending regulatory changes.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- Policy documents and control standards maintained as Word documents and Excel workbooks in a dedicated SharePoint library
- Severity matrices and routing rules maintained as Excel workbooks
- Prior exception and finding history maintained in an Excel workbook populated by Power Automate flows from the GRC platform
- Control ownership data synced from the GRC platform to a SharePoint list via Power Automate

Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For data that has no integration path (e.g., hotline narratives, verbal reports, external audit findings), skills provide structured intake that captures the information and writes it to the case tracker. The framework's Signal Inventory identifies exactly which data points are needed so the skill prompts for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge for policies, severity matrices, and control ownership) and Tier 3 (manual input for hotline reports and case context). Graph Connectors for GRC integration require tenant admin setup and should be introduced in Wave 2 after skill workflows are proven.

### 3.5 Governance in Cowork

Compliance and Risk workflows impose stringent governance requirements due to regulatory obligations, evidence integrity, audit trail requirements, and segregation of duties.

| Governance Domain | Cowork Implementation |
|---|---|
| **Audit trail integrity** | Every write action logged with actor, timestamp, prior value, and case linkage in the Excel tracker. SharePoint document versioning preserves all changes. Platform-level logs capture tool invocations. Skills never update case status without logging the change. |
| **Segregation of duties** | Skill guardrails enforce that the person who triages is not the final approver. Routing recommendations flag conflicts when the assigned analyst is also the control owner. Skills do not allow self-approval of routing or disposition. |
| **Evidence chain of custody** | Skills log every evidence document accessed with timestamp. Evidence folders in SharePoint use versioning and restricted permissions. Skills never modify submitted evidence — they only read and reference. |
| **Anonymous reporting protection** | Skills never attempt to identify anonymous reporters. Case records support anonymous case types. Communications never disclose reporter identity for anonymous cases. |
| **Regulatory sensitivity** | Skills flag regulatory-reportable indicators (anti-corruption, sanctions, data breach) with elevated visibility and separate escalation. Regulatory referral drafts are generated automatically when regulatory indicators are detected. |
| **Access control** | M365 tenant RBAC governs data access. SharePoint permissions on case folders enforce need-to-know. Skills cannot bypass folder-level permissions. Sensitive case types (whistleblower, executive misconduct) have restricted access groups. |
| **Change control** | Policies, severity matrices, and routing rules are maintained in SharePoint documents that skills read at runtime. Compliance operations can update policies without modifying skill definitions. Skill instruction changes follow compliance change control. |
| **Data classification** | Skill guardrails enforce handling rules: no case substance in Teams messages, no conclusions in any automated output, confidentiality markings on all internal compliance communications. |

**The main governance gap** is formal audit event logging. Cowork does not have a built-in, tamper-resistant audit log for compliance events. The mitigation is to externalize audit logging to the Excel case tracker (with versioning in SharePoint) and to design skills that explicitly log every state-changing action. For organizations with strict regulatory audit requirements, supplementing with Power Automate flows that write to a dedicated audit log is recommended.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Compliance and Risk:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Risk indicator precision and recall — does `compliance-risk-detection` surface genuine risk indicators without excessive false positives? Assessed by comparing skill output to analyst review on 10+ cases
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Risk indicator precision and recall — percentage of actual risk factors identified vs. false positives
- Incorrect routing rate — percentage of cases routed to the wrong reviewer or missing a required approver
- Evidence gap detection rate — percentage of actual evidence gaps identified by the skill vs. manual analyst review
- Triage cycle time — time from case creation to "queued for review" or "escalated" status
- Missing audit-field rate — percentage of case records with incomplete audit trail fields
- Unauthorized access incidents — cases where case materials were accessed outside the approved reviewer list
- Override rate — how often compliance analysts override skill recommendations

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel compliance case tracker in SharePoint with standard columns (Case ID, Case Type, Reporter, Affected Policy, Business Unit, Legal Entity, Severity, Status, Received Date, Assigned Analyst, Evidence Status, Disposition, Audit Log)
- Upload policy documents, control standards, evidence checklists, and severity matrices to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-case evidence and documentation
- Upload the routing rules and reviewer assignments as SharePoint-hosted Excel workbooks

**Skills to build:**
- `compliance-case-intake`
- `compliance-context-packet`
- `compliance-risk-detection`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5-10 real compliance cases across different case types.

### Wave 2 — Routing and Communication

**Skills to build:**
- `compliance-case-comms`
- `compliance-review-routing`

**Promotions:**
- Promote `compliance-risk-detection` to write-back mode (updates evidence checklist after user confirmation)
- Promote `compliance-case-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for compliance cases with approaching SLA deadlines and surfaces any with incomplete evidence or unassigned reviewers
- Set up a weekly scheduled prompt that detects repeated policy-exception patterns across the case portfolio

**Integrations:**
- Introduce Graph Connectors for GRC platform data if available at the tenant level
- Add policy-cited severity classifications that reference specific policy section numbers
- Add controlled review-task creation in the case tracker

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action.

### Wave 3 — Optimization and Expansion

**Enhancements:**
- Add bounded multi-source evidence packet assembly (aggregate evidence from email, SharePoint, and Teams into a unified case packet)
- Add proactive detection of repeated policy-exception patterns and recurring control failures via scheduled prompt
- Refine risk indicator thresholds based on analyst override patterns from Waves 1-2
- Add third-party compliance request review as a second skill suite using the same framework

**Measurement:**
- Triage cycle time reduction vs. pre-pilot baseline
- Case summary acceptance rate target: above 70%
- Risk indicator detection precision target: above 75%
- Evidence gap detection rate target: above 85%
- Incorrect routing rate target: below 5%
- Missing audit-field rate target: below 3%
- Unauthorized access incident target: zero

---

## Part 5: Generalizing the Approach — Compliance and Risk's Primary Artifact Pattern

Compliance and Risk's natural primary artifacts are **Excel and SharePoint** — case tracking, evidence checklists, severity matrices, and routing rules are structured tabular data, while policies, control standards, and evidence documents live in SharePoint libraries. Word documents serve as the narrative evidence layer for context packets and case summaries.

This artifact pattern fits the cross-LOB method as follows:

1. **Decompose** — compliance steps separate cleanly because each has a distinct goal (intake vs. evidence assembly vs. risk detection vs. routing vs. communication)
2. **Map signals to M365 tools** — replace "GRC platform" with SharePoint + Graph Connectors; replace "case management platform" with Excel tracker + SharePoint case folders
3. **Identify the process state artifact** — the Excel case tracker is the canonical tracker; SharePoint case folders are the evidence backbone; severity matrices and routing rules are Excel workbooks in SharePoint
4. **Define first-class outputs** — Excel dominates for structured state; Word documents provide narrative evidence; Adaptive Cards provide quick-review surfaces; Outlook drafts handle all external communication
5. **Encode governance in guardrails** — audit trail integrity, segregation of duties, evidence chain of custody, anonymous reporting protection, and regulatory sensitivity are the dominant governance concerns; every skill guardrail section must address these explicitly

The framework's decomposition is universal. The Excel-and-SharePoint-centric artifact mapping and audit-first guardrail posture are what make it concrete for Compliance and Risk in Copilot Cowork.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references (GRC platform, case management) |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint policy library |
| Process State | SharePoint-hosted Excel workbook + case folders | Durable state externalized to M365 artifacts; case folders hold all evidence documents |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, hotline report, form submission, document upload |
| Approval | CreateDraftMessage + Adaptive Card + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; audit-logged |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time, risk indicator precision, and evidence gap detection rate |
