# Plan: Internal Audit Request Intake and Evidence Collection Prep — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Internal Audit line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Audit Request Intake and Evidence Collection Prep pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Internal Audit, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Internal Audit's low error tolerance and strong approval structure elevate the importance of evidence traceability and workpaper integrity. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope principle. Audit's evidence-centric steps decompose cleanly along intake-scoping-gap detection-routing-communication boundaries. |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "audit management platform", "GRC platform", and "control repository" that must become specific M365 tool names or SharePoint-bridged data stores. |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Internal Audit's independence requirements and workpaper standards demand strict guardrails. Audit conclusions, scope judgments, and sign-offs must remain explicitly human-owned. |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — evidence gap detection maps to Decision Support; audit summary drafting maps to Content Generation; scope and evidence assembly maps to Data Aggregation. |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal tool contracts; orchestration is implicit in skill instructions. Audit's review gates and workpaper standards must be encoded as guardrails and artifact patterns. |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides the runtime layers natively. The skill author controls capability definition and decision logic. Audit-specific access scoping relies on M365 tenant RBAC and SharePoint permissions on workpaper folders. |

### Key Insight

Internal Audit workflows are evidence-heavy, traceability-sensitive, and review-bound. The framework's decomposition discipline is especially valuable here because it prevents the common anti-pattern of building one broad "audit preparation" skill that conflates intake, scoping, evidence collection, and routing. By decomposing into six steps, each skill has a clear scope, a defined automation boundary, and explicit guardrails that preserve audit independence and workpaper integrity. The critical principle is that AI never draws audit conclusions — it assembles, surfaces gaps, and drafts communications, while auditors retain all judgment and sign-off authority.

---

## Part 2: The Internal Audit Evidence Prep Plugin — Skill-by-Skill Design

The Internal Audit sample decomposes "Audit Request Intake and Evidence Collection Prep" into 6 steps (IA-001 through IA-006), identifies 6 skills, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| IA-001: Normalize audit request | `audit-request-intake` | Data Aggregation | Deterministic automation | Excel (case tracker), SharePoint (list), Outlook (request emails) |
| IA-002: Gather scope, control, and prior-evidence context | `audit-evidence-packet` | Data Aggregation + Content Generation | AI act within policy | Word (audit packet), Excel (control ownership, evidence checklist), SharePoint (prior findings, control library) |
| IA-003: Identify missing evidence and scope gaps | `audit-gap-detection` | Decision Support | AI assist | Excel (evidence checklist), SharePoint (evidence files), Adaptive Card (gap report) |
| IA-004: Determine evidence-request and reviewer path | `audit-request-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Excel (reviewer matrix) |
| IA-005: Draft audit summary and evidence requests | `audit-case-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (audit summary) |
| IA-006: Confirm audit packet disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (disposition confirmation) |

### Detailed Skill Designs

#### 1. `audit-request-intake` — Normalize Audit Request

**Framework Step:** IA-001

**Trigger phrases:** "new audit request", "audit intake for [process area]", "log audit case for", "walkthrough scheduled for [control area]", "open evidence request for", "new audit engagement"

**Inputs:**
- Audit engagement or request type (scheduled audit, ad hoc request, walkthrough, follow-up)
- Process area or control area under audit
- Business unit and legal entity
- Audit period (start and end dates)
- Lead auditor assignment
- Related prior audit findings (if known)

**M365 tools:**
- `SearchPeople` — resolve auditor assignments and control owner identities
- `GetUserDetails` — pull auditor and control owner profiles
- `SearchM365(sources=["email"])` — find the original audit request or scheduling email
- `ReadFileContent` — read SharePoint-hosted audit case tracker to check for duplicate or related engagements

**Output:** Structured case record written to Excel audit case tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Engagement Type, Process Area, Business Unit, Legal Entity, Audit Period, Lead Auditor, Status, Created Date, Evidence Status, Prior Findings Count, Disposition

**Guardrails:**
- Never create duplicate cases for the same process area and audit period without explicit user confirmation
- Validate that all required fields (process area, audit period, lead auditor) are populated before writing
- Confirm details with user via Adaptive Card before writing to tracker
- Log case creation with timestamp and creating user in the audit column
- Never assign audit engagements — only record assignments made by the audit manager

---

#### 2. `audit-evidence-packet` — Gather Scope, Control, and Prior-Evidence Context

**Framework Step:** IA-002

**Trigger phrases:** "build audit packet for", "assemble evidence context for [case ID]", "pull prior findings for [control area]", "prepare audit scope materials", "what evidence do we need for [engagement]"

**Inputs:**
- Case ID or engagement description
- Process area or control area

**M365 tools:**
- `GetUserDetails` — control owner and process owner context
- `GetManagerDetails` — reporting chain for control accountability
- `SearchM365(sources=["files"])` — audit methodology documents, evidence checklists, prior findings, prior workpapers, control narratives
- `ReadFileContent` — SharePoint-hosted control library, prior audit reports, evidence requirements, and the audit program
- `GetDriveChildren` — browse the audit evidence library and prior engagement folders
- `SearchM365(sources=["connectors"], connector_ids=["audit-mgmt-connector"])` — audit management platform data (when Graph Connector available)

**Output:** Word document containing:
- Engagement summary (type, scope, period, objectives)
- Control environment overview (control owners, process narratives, key controls)
- Prior findings and remediation status for the same control area
- Evidence requirements checklist with sources and owners
- Applicable methodology standards and testing approach references
- Key contacts (control owners, process owners, audit team members)

**Artifact:** Word (.docx) audit packet and Excel evidence requirements checklist, both saved to SharePoint audit engagement folder

**Guardrails:**
- Cite source and date for every prior finding, control narrative, or methodology reference
- Never interpret evidence sufficiency or control effectiveness — present factual context only
- Flag if any required methodology document or prior finding is not found in SharePoint
- Restrict packet access to the audit team SharePoint group
- Preserve evidence traceability — log every document accessed with timestamp and source location
- Never include draft audit conclusions or opinions in the evidence packet — this is factual assembly only

---

#### 3. `audit-gap-detection` — Identify Missing Evidence and Scope Gaps

**Framework Step:** IA-003

**Trigger phrases:** "check evidence gaps for", "what's missing for audit [case ID]", "evidence status for engagement", "audit readiness check", "scope gap analysis for"

**Inputs:**
- Audit packet (Word document or case data)
- Evidence requirements checklist (Excel)

**M365 tools:**
- `ReadFileContent` — evidence checklist, control documentation, and submitted evidence from SharePoint
- `SearchM365(sources=["files"])` — uploaded evidence artifacts, prior workpapers, control owner responses
- `GetDriveChildren` — engagement evidence folder contents to check what has been submitted

**Output:** Gap report as Adaptive Card (for quick review) plus Excel checklist update (for tracking)

**Logic:** Compare required evidence items against submitted materials in the engagement evidence folder. Detect gaps: (1) missing mandatory evidence items, (2) evidence submitted but covering wrong period, (3) control owner not yet contacted for walkthrough, (4) prior finding remediation evidence not provided, (5) scope areas with no assigned control owner. Classify gaps by impact: low (informational), medium (may delay testing), high (blocks testing), critical (scope risk — may require engagement plan revision).

**Guardrails:**
- Present all findings as "evidence gaps" or "scope observations" — never characterize findings as audit conclusions, control deficiencies, or risk assessments
- Every gap must cite the specific checklist item and the expected vs. actual evidence status
- Never assess control effectiveness, operational risk, or management performance — surface factual gaps only
- Flag gaps that could delay the audit timeline with elevated visibility
- Flag gaps related to prior finding remediation separately — these indicate whether the organization addressed known issues
- Present findings for user review via Adaptive Card before updating any tracker or checklist
- Read-only mode — this skill does not modify audit status or conclusions; it only surfaces analysis

---

#### 4. `audit-request-routing` — Determine Evidence-Request and Reviewer Path

**Framework Step:** IA-004

**Trigger phrases:** "route evidence requests for", "who owns [control area]", "assign evidence requests for [case ID]", "set up review chain for audit", "determine who needs to provide evidence"

**Inputs:**
- Gap report output
- Engagement data (scope, control area, audit period)
- Reviewer matrix and control ownership records (SharePoint documents)

**M365 tools:**
- `SearchPeople` — resolve control owners, process owners, and business liaisons by function
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths and control accountability
- `ReadFileContent` — reviewer matrix, control ownership records, and scope rules from SharePoint
- `PostMessage` — Teams notification to control owners for evidence requests (after approval)
- `CreateDraftMessage` — Outlook draft for formal evidence request or escalation notice (after approval)

**Output:** Routing recommendation:
- Assigned control owners for each evidence request based on control area and process ownership
- Required reviewers (audit manager, senior auditor) based on engagement type and gap severity
- Escalation path if evidence gaps are critical or control owners are unresponsive
- Timeline recommendations based on audit schedule and testing deadlines

**Guardrails:**
- Present routing recommendation for lead auditor review before sending any messages or assignments
- Never auto-assign outside the approved reviewer matrix and control ownership records
- Escalate to audit manager if critical evidence gaps are identified or control owners are unidentified
- Never reveal specific audit findings or preliminary observations in routing messages — use engagement IDs and references to the SharePoint engagement folder
- Log all routing decisions to the Excel case tracker with timestamp, actor, and rationale
- Preserve audit independence — never route evidence requests through individuals who are also under audit for the same engagement

---

#### 5. `audit-case-comms` — Draft Audit Summary and Evidence Requests

**Framework Step:** IA-005

**Trigger phrases:** "draft audit summary for", "prepare evidence request for [control owner]", "audit status update for [engagement]", "draft follow-up to [control owner]", "prepare audit packet summary"

**Inputs:**
- Case data from Excel tracker
- Audit packet (Word document)
- Gap report
- Target audience (audit manager, control owner, business process owner, audit committee liaison)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages to audit team channel (after confirmation)
- `SearchM365(sources=["files"])` — evidence request templates and standard communication language from SharePoint
- `ReadFileContent` — case artifacts for summary content

**Communication templates:**
- Audit engagement summary to audit manager (scope, evidence status, gap overview, recommended testing focus)
- Evidence request to control owner (specific items needed, period, format, deadline)
- Follow-up reminder for overdue evidence
- Status update to business process owner
- Escalation notice to audit manager for unresponsive control owners
- Methodology-cited packet summary for workpaper documentation

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Never include audit conclusions, preliminary findings, or risk ratings in any communication — present factual status only
- Never include internal audit team deliberations or testing strategies in communications to control owners
- Match tone to audience: methodology-precise for audit team communications, clear and professional for control owner requests
- Include engagement reference number and evidence request ID in every communication
- Mark all internal audit communications with appropriate confidentiality markings
- Evidence request drafts must specify exact items needed, the audit period covered, acceptable format, and submission deadline

---

#### Step 6: Confirm Audit Packet Disposition (Human Only)

**Framework Step:** IA-006

This is not a Cowork skill. The framework correctly identifies final audit packet disposition as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the packet review meeting with the audit manager
- The audit summary artifacts — provide the evidence package for the reviewer
- The `audit-case-comms` skill — send the disposition confirmation after the human decision is made
- The Excel case tracker — updated to reflect final disposition (ready for testing, requires additional evidence, escalated, or scope revision needed)

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Internal Audit is **decomposition discipline with evidence traceability**:

**Process decomposition prevents mega-skills.** The common anti-pattern for audit AI is building one broad "audit preparation" tool that conflates intake, scoping, evidence collection, gap analysis, and routing. The framework's rule — "keep breaking down until each step has one dominant goal" — produces well-scoped skills that separate evidence assembly from gap analysis from routing from communication. This separation is essential for workpaper quality, evidence traceability, and audit independence.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions. Gap detection operates here. |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation. Routing and communications operate here. |
| AI act within policy | Skill can execute bounded read and retrieval actions within defined rules. Evidence assembly operates here. |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed. Audit request intake operates here. |

**Signal inventory forces explicit M365 tool selection.** Instead of vague references to "audit management platform" or "control repository," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365`, `ReadFileContent`, `GetDriveChildren` — a direct quality improvement for audit skills where evidence traceability is foundational.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which process step is active |
| Capability registry | Built-in — skills are discovered and versioned in the skill directory |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal audit review workflow engine |
| Governance and control | Partial — skill instructions encode policies; M365 tenant RBAC enforces access boundaries; no built-in workpaper management or evidence chain-of-custody system |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine or a formal workpaper management system. Audit case state (evidence status, reviewer assignments, scope decisions, prior finding references) must be externalized to M365 artifacts. The Excel case tracker and SharePoint engagement folders serve this role, but every skill must explicitly maintain evidence traceability and state logging.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the audit request and evidence collection process:

| Artifact | Role in Audit Workflow | How Skills Use It |
|---|---|---|
| **Excel** (PRIMARY) | Case tracker (engagement status, assignments, evidence status, prior findings), evidence checklists, reviewer matrices, control ownership records | The canonical process state store — skills read current engagement status, update evidence completion tracking, and log routing decisions. Excel is the primary structured data artifact for audit tracking. |
| **Word** (PRIMARY) | Audit packets, engagement summaries, control narratives, prior findings summaries, evidence request narratives | Skills generate documents that become part of the workpaper trail. Word is the primary narrative artifact — audit packets, scope documents, and summaries are all Word documents. |
| **SharePoint** (PRIMARY) | Source of truth for audit methodology, control library, evidence documents, prior workpapers, engagement folders | Skills read authoritative methodology documents and store all engagement artifacts in per-engagement folders. SharePoint permissions enforce workpaper access control. |
| **Outlook** | Communication channel, request intake signal, draft evidence requests and summaries | Skills read incoming audit requests; draft all control-owner-facing and management-facing communications as reviewable Outlook drafts |
| **Teams** | Internal audit team coordination and status routing | Skills post task assignments and escalation notices to audit team channels after confirmation |
| **Adaptive Card** | Quick-review presentation for evidence gaps and routing recommendations | Skills present gap analysis for rapid review without requiring the user to open a full document |
| **Graph API** | People data — control owners, process owners, audit team members, org hierarchy | Skills resolve people for routing decisions, escalation paths, and control ownership accountability |
| **Calendar** | Walkthrough scheduling, evidence submission deadlines, review meetings | Skills create calendar events for walkthrough dates, evidence deadlines, and review milestones |

**The design pattern:** The Cowork Internal Audit plugin uses Excel, Word, and SharePoint as a triple backbone. Excel workbooks provide structured engagement tracking and evidence checklists. Word documents serve as the narrative workpaper layer (audit packets, summaries, control narratives). SharePoint hosts the methodology library, control library, prior workpapers, and per-engagement evidence folders. Every skill reads from and writes to this shared ecosystem through M365 tools, with explicit evidence traceability logging on every access and state change.

### 3.4 Federated Connectors for Third-Party Systems

The framework references audit management platforms, GRC platforms, and control repositories. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For audit management platforms (TeamMate, AuditBoard, Workiva, Diligent), Graph Connectors index engagement metadata, finding records, and evidence status into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["audit-mgmt-connector"])`. This provides read access to engagement status, prior findings, and evidence collection progress without custom integration code.

For GRC platforms (ServiceNow GRC, Archer), Graph Connectors can index control records and risk assessments for cross-referencing during audit preparation. Skills can surface whether a control area has open risk findings from the GRC platform.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- Audit methodology documents and evidence checklists maintained as Word documents and Excel workbooks in a dedicated SharePoint library
- Control library and control ownership records maintained as Excel workbooks or SharePoint lists
- Prior findings and remediation tracking maintained in an Excel workbook populated by Power Automate flows from the audit management platform
- Reviewer matrices and escalation paths maintained as Excel workbooks in SharePoint

Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For data that has no integration path (e.g., walkthrough observations, verbal explanations from control owners, ad hoc audit requests from management), skills provide structured intake that captures the information and writes it to the case tracker. The framework's Signal Inventory identifies exactly which data points are needed so the skill prompts for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge for methodology, control library, and prior findings) and Tier 3 (manual input for walkthrough observations and ad hoc requests). Graph Connectors for audit management platform integration require tenant admin setup and should be introduced in Wave 2 after skill workflows are proven.

### 3.5 Governance in Cowork

Internal Audit workflows impose stringent governance requirements due to audit independence standards, workpaper integrity, evidence chain of custody, and professional standards compliance (IIA Standards).

| Governance Domain | Cowork Implementation |
|---|---|
| **Audit independence** | Skills never draw audit conclusions, assess control effectiveness, or rate risk. All skill outputs present factual assembly and gap identification only. Routing guardrails prevent assigning evidence requests to individuals under audit for the same engagement. Skills do not influence audit scope or testing strategy. |
| **Workpaper integrity** | All generated documents (audit packets, summaries, evidence checklists) are saved to versioned SharePoint engagement folders. SharePoint document versioning preserves all changes. Skills never modify submitted evidence documents — they only read and reference. |
| **Evidence chain of custody** | Skills log every evidence document accessed with timestamp and source location. Evidence folders in SharePoint use versioning and restricted permissions. Skills never move, rename, or delete evidence files. All evidence references include document name, version, location, and access timestamp. |
| **Access control** | M365 tenant RBAC governs data access. SharePoint permissions on engagement folders and workpaper libraries enforce need-to-know. Audit workpapers are restricted to the audit team SharePoint group. Skills cannot bypass folder-level permissions. |
| **Professional standards** | Skills reference IIA Standards and organizational audit methodology in generated packets. Methodology documents are maintained in SharePoint and read dynamically at runtime. Skills flag when required methodology steps have no corresponding evidence. |
| **Segregation of duties** | Routing guardrails enforce that evidence requestors are not also evidence approvers. Skills flag conflicts when a control owner is asked to provide evidence for their own control effectiveness assessment. |
| **Change control** | Methodology documents, evidence checklists, and reviewer matrices are maintained in SharePoint documents that skills read at runtime. Audit operations can update methodology without modifying skill definitions. Skill instruction changes follow audit operations change control. |
| **Data classification** | Skill guardrails enforce handling rules: no audit observations in Teams messages to non-audit recipients, no preliminary findings in any automated output, confidentiality markings on all internal audit communications. |

**The main governance gap** is formal workpaper management. Cowork does not have a built-in workpaper management system with cross-referencing, tick marks, and review sign-off. The mitigation is to use SharePoint engagement folders with versioning as the workpaper repository and the Excel case tracker for status tracking. For organizations with strict IIA Standards requirements, the Cowork plugin should be positioned as preparation support only — formal workpaper documentation and sign-off should remain in the dedicated audit management platform.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Internal Audit:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Evidence gap detection accuracy — does `audit-gap-detection` find the gaps that auditors find on manual review? Assessed by comparing skill output to auditor review on 10+ engagements
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Evidence gap detection rate — percentage of actual evidence gaps identified by the skill vs. manual auditor review
- Incorrect reviewer-routing rate — percentage of evidence requests routed to the wrong control owner or missing a required reviewer
- Draft acceptance rate — percentage of `audit-case-comms` drafts sent without major substantive edits
- Time to review-ready audit packet — time from case creation to "ready for testing" status
- Traceability coverage rate — percentage of evidence items in the packet with complete source references
- Override rate — how often auditors override skill recommendations for routing or gap classification

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel audit case tracker in SharePoint with standard columns (Case ID, Engagement Type, Process Area, Business Unit, Legal Entity, Audit Period, Lead Auditor, Status, Created Date, Evidence Status, Prior Findings Count, Disposition, Audit Log)
- Upload audit methodology documents, evidence checklists, control library, and prior findings summaries to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-engagement evidence and workpapers
- Upload the reviewer matrix and control ownership records as SharePoint-hosted Excel workbooks

**Skills to build:**
- `audit-request-intake`
- `audit-evidence-packet`
- `audit-gap-detection`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5-10 real audit engagements across different engagement types.

### Wave 2 — Routing and Communication

**Skills to build:**
- `audit-case-comms`
- `audit-request-routing`

**Promotions:**
- Promote `audit-gap-detection` to write-back mode (updates evidence checklist after user confirmation)
- Promote `audit-request-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for audit engagements with approaching evidence deadlines and surfaces any with incomplete evidence status
- Set up a weekly scheduled prompt that identifies overdue evidence requests and unresponsive control owners

**Integrations:**
- Introduce Graph Connectors for audit management platform data if available at the tenant level
- Add methodology-cited packet summaries that reference specific audit program sections
- Add controlled evidence request task creation in the case tracker

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action.

### Wave 3 — Optimization and Expansion

**Enhancements:**
- Add bounded multi-source evidence packet assembly (aggregate evidence from email, SharePoint, Teams, and the control library into a unified engagement packet)
- Add proactive detection of recurring evidence gaps and unresponsive control owner patterns across the engagement portfolio via scheduled prompt
- Refine gap detection thresholds based on auditor override patterns from Waves 1-2
- Add control walkthrough summary preparation as a second skill suite using the same framework

**Measurement:**
- Time to review-ready packet reduction vs. pre-pilot baseline
- Draft acceptance rate target: above 70%
- Evidence gap detection rate target: above 85%
- Incorrect routing rate target: below 5%
- Traceability coverage rate target: above 90%
- Override rate target: below 20% (indicating skill recommendations are reliable)

---

## Part 5: Generalizing the Approach — Internal Audit's Primary Artifact Pattern

Internal Audit's natural primary artifacts are **Excel, Word, and SharePoint** — evidence checklists, case tracking, and control ownership are structured tabular data in Excel; audit packets, summaries, and control narratives are document-centric Word artifacts; and the methodology library, control library, evidence documents, and workpapers live in SharePoint libraries.

This artifact pattern fits the cross-LOB method as follows:

1. **Decompose** — audit steps separate cleanly because each has a distinct goal (intake vs. evidence assembly vs. gap detection vs. routing vs. communication)
2. **Map signals to M365 tools** — replace "audit management platform" with SharePoint + Graph Connectors; replace "control repository" with SharePoint-hosted control library and Excel control ownership records
3. **Identify the process state artifact** — the Excel case tracker is the canonical tracker; SharePoint engagement folders are the evidence backbone; the control library and methodology documents are SharePoint-hosted reference materials
4. **Define first-class outputs** — Excel dominates for structured tracking and evidence checklists; Word provides the narrative workpaper layer (audit packets and summaries); Adaptive Cards provide quick-review surfaces; Outlook drafts handle all external evidence requests
5. **Encode governance in guardrails** — audit independence, workpaper integrity, evidence chain of custody, and professional standards compliance are the dominant governance concerns; every skill guardrail section must address these explicitly

The framework's decomposition is universal. The triple-backbone artifact pattern (Excel + Word + SharePoint) and independence-first guardrail posture are what make it concrete for Internal Audit in Copilot Cowork.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references (audit management platform, GRC, control repository) |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic methodology read from SharePoint audit library |
| Process State | SharePoint-hosted Excel workbook + engagement folders | Durable state externalized to M365 artifacts; engagement folders hold all evidence and workpapers |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, walkthrough scheduling, evidence upload, audit request |
| Approval | CreateDraftMessage + Adaptive Card + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; independence-preserving |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via time to ready, gap detection rate, and traceability coverage |
