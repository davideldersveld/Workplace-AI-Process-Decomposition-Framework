# Plan: Finance Controllership Journal Entry Review — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Finance Controllership line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Manual Journal Entry Request and Approval pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Finance Controllership, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — Controllership references ERP, close management system, and ledger data that must become M365 tool names or SharePoint bridge reads |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Controllership's strict approval requirements and accounting governance reinforce the need for explicit review gates at every step |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the task patterns map to Cowork skill templates |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

Controllership workflows are the most control-sensitive processes in Finance. The framework's decomposition discipline is critical here because journal entries are SOX-auditable, require segregation of duties, and demand complete evidence trails. The key architectural constraint is that AI never posts a journal entry — it prepares, validates, and routes. By decomposing the process into 6 steps with explicit automation boundaries, the plugin ensures that accounting judgment and approval authority remain fully human-owned while AI handles the time-consuming packet preparation and evidence assembly work.

---

## Part 2: The Finance Controllership Journal Entry Plugin — Skill-by-Skill Design

The Controllership sample decomposes "Manual Journal Entry Request and Approval" into 6 steps (FC-001 through FC-006), identifies 5 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| FC-001: Normalize journal request | `fc-journal-intake` | Data Aggregation | Deterministic automation | Excel (journal tracker), SharePoint (list), Outlook (notifications) |
| FC-002: Gather ledger, policy, and support context | `fc-journal-context` | Data Aggregation | AI act within policy | Word (journal packet), SharePoint (policies, ledger data), Graph API (people) |
| FC-003: Detect missing support and control risks | `fc-gap-risk-detection` | Decision Support | AI assist | Excel (checklist), Adaptive Card (risk report), SharePoint (control checklist) |
| FC-004: Determine approval path | `fc-approval-routing` | Decision Support | AI draft + approve | Adaptive Card (recommendation), Teams (messages), Outlook (approval requests) |
| FC-005: Draft journal summary and follow-up requests | `fc-journal-comms` | Content Generation | AI draft + approve | Word (journal summary), Outlook (drafts), Teams (messages) |
| FC-006: Confirm review-readiness disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `fc-journal-intake` — Normalize Journal Request

**Framework Step:** FC-001

**Trigger phrases:** "new journal entry request", "journal case for [description]", "set up journal review for", "close adjustment request", "manual JE for"

**Inputs:**
- Journal entry description or rationale
- Requesting accountant or source
- Entity and business unit
- Account codes (debit and credit)
- Amount and currency
- Close period
- Supporting document references

**M365 tools:**
- `SearchM365(sources=["email"])` — find the request email thread for context
- `SearchM365(sources=["files"])` — locate supporting schedules or memos in SharePoint
- `ReadFileContent` — read existing journal tracker to check for duplicates
- `GetUserDetails` — resolve requestor identity and reporting chain

**Output:** Structured journal case record written to Excel journal tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Journal Case ID, Description, Requestor, Entity, Business Unit, Debit Account, Credit Account, Amount, Currency, Close Period, Status, Created Date, Support Status, Approval Path, Assigned Reviewer

**Guardrails:**
- Never create duplicate cases for the same journal description, amount, and period
- Validate that debit and credit amounts balance
- Confirm all details with user before writing to tracker
- Log case creation with timestamp, actor, and source reference
- Reject entries missing required fields (entity, accounts, amount, period)

---

#### 2. `fc-journal-context` — Gather Ledger, Policy, and Support Context

**Framework Step:** FC-002

**Trigger phrases:** "build journal packet", "gather context for journal [ID]", "assemble support for close entry", "what do we need for this journal"

**Inputs:**
- Journal case ID or description

**M365 tools:**
- `ReadFileContent` — accounting policies, journal entry standards, approval thresholds from SharePoint
- `SearchM365(sources=["files"])` — supporting schedules, reconciliations, prior period entries, memos
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` — ledger balances, account metadata, entity details (if Graph Connector available)
- `GetDriveChildren` — list documents already uploaded to the journal case folder
- `SearchPeople` — resolve reviewer and approver contacts
- `GetManagerDetails` — reporting chain for approval routing

**Output:** Word document containing:
- Journal entry details (accounts, amounts, entity, period)
- Accounting policy extracts relevant to this entry type
- Ledger context (current account balances, prior period comparisons)
- Supporting document inventory (what has been uploaded vs. what is required)
- Approval threshold and required approvers
- Historical context (similar entries in prior periods)
- Key contacts (requestor, reviewer, approver)

**Artifact:** Word (.docx) saved to SharePoint journal case folder; serves as the reviewer evidence packet

**Guardrails:**
- Cite source and retrieval date for every data element
- Flag if any required supporting document (schedule, reconciliation, memo) is missing from the case folder
- Never include draft or unposted ledger data without clearly labeling it as preliminary
- Mark any policy extract with its version date and document reference

---

#### 3. `fc-gap-risk-detection` — Detect Missing Support and Control Risks

**Framework Step:** FC-003

**Trigger phrases:** "check journal for gaps", "what's missing for this entry", "journal risk check", "audit readiness for journal [ID]", "control review for close entry"

**Inputs:**
- Journal packet (Word doc or case data from Excel tracker)
- Control checklist from SharePoint

**M365 tools:**
- `ReadFileContent` — control checklist and evidence requirements from SharePoint
- `ReadFileContent` — journal packet document
- `GetDriveChildren` — case folder contents to verify uploaded support
- `SearchM365(sources=["files"])` — prior similar entries for pattern comparison

**Output:** Gap and risk report as Adaptive Card (for quick review) plus Excel tracker update (after confirmation)

**Detection categories:**
- Missing supporting schedule or reconciliation
- Missing or incomplete rationale memo
- Entry above materiality threshold without enhanced approval
- Unusual account combination (not seen in prior periods)
- Duplicate entry candidate (similar amount, accounts, period)
- Reversal entry without original reference
- Cross-entity entry requiring additional sign-off
- Late close adjustment requiring controller approval

**Logic:** Compare the journal packet contents against the control checklist requirements. Cross-reference with prior period entries for anomaly detection. Flag items that require additional evidence or elevated approval.

**Guardrails:**
- Present findings for accountant review before updating any tracker — this is AI assist mode
- Never mark a control item as satisfied without document evidence in the case folder
- Flag entries above materiality threshold with elevated visibility
- Clearly separate "missing evidence" (fixable) from "control risk" (requires escalation)
- Include confidence level and rationale for every risk flag

---

#### 4. `fc-approval-routing` — Determine Approval Path

**Framework Step:** FC-004

**Trigger phrases:** "route journal for approval", "who approves this entry", "send to reviewer", "approval path for journal [ID]"

**Inputs:**
- Gap and risk report
- Journal case data
- Approval matrix and threshold table (SharePoint documents)

**M365 tools:**
- `ReadFileContent` — approval matrix, threshold table, escalation policy from SharePoint
- `SearchPeople` — resolve required approvers by role
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to reviewers with case summary
- `CreateDraftMessage` — Outlook draft for formal approval requests

**Output:** Approval path recommendation as Adaptive Card with:
- Required approval level (accounting manager, controller, CFO) based on amount and entry type
- Named approvers resolved from the approval matrix
- Segregation check result (preparer cannot be approver)
- Risk flags that affect the approval path
- Policy citations supporting the routing decision

Routed work items (after confirmation):
- Teams message to assigned reviewer with journal summary and case link
- Outlook draft for formal approval request with evidence packet attached
- Updated Excel tracker with approval path, assigned reviewer, and routing date

**Guardrails:**
- Present routing recommendation for accountant review before sending any messages — this is AI draft plus approve
- Never route to someone who prepared or requested the journal entry (segregation of duties)
- Always verify the approver has sufficient authority per the threshold table
- Escalate to controller if no clear approver exists in the matrix
- Include journal case ID in every communication for audit traceability
- Never auto-approve any journal entry regardless of amount

---

#### 5. `fc-journal-comms` — Draft Journal Summary and Follow-Up Requests

**Framework Step:** FC-005

**Trigger phrases:** "draft journal summary", "prepare reviewer packet", "write follow-up for missing support", "journal entry summary for [reviewer]", "close entry review packet"

**Inputs:**
- Journal case data from Excel tracker
- Journal packet (Word document)
- Gap and risk report
- Target audience (reviewer, controller, requestor, close coordinator)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages for internal status updates
- `SearchM365(sources=["files"])` — summary templates and tone guides from SharePoint
- `ReadFileContent` — case evidence for citation in summaries

**Communication templates:**
- Journal entry reviewer summary (concise packet for accounting manager or controller)
- Missing evidence request to requestor or accountant
- Close coordination update to close coordinator
- Escalation notice to controller for high-risk or threshold-exceeding entries
- Approval confirmation request
- Rework notification with specific items to address

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Match tone to audience: formal and precise for controller review, operational for close coordination
- Include journal case ID, entry description, amount, and period in every communication
- Cite specific policy sections and evidence references in reviewer summaries
- Never include preliminary or unvalidated amounts in external communications
- Every generated summary must trace findings to source documents — no fabricated assertions
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable

---

#### Step 6: Confirm Review-Readiness Disposition (Human Only)

**Framework Step:** FC-006

This is not a Cowork skill. The framework correctly identifies final review-readiness confirmation as a human-only step. In Cowork, it is supported by:

- The `fc-journal-comms` skill — prepare the reviewer summary and approval request
- The journal packet artifacts — provide the complete evidence package
- The `fc-gap-risk-detection` skill — confirm all control items are satisfied before routing
- Calendar support — book the review meeting if the entry requires discussion before approval

The accounting manager or controller makes the final determination: approve for posting, return for rework, or escalate for additional review. This decision is logged in the Excel tracker by the `fc-journal-intake` skill (status update function) after the human decision is made.

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Controllership is **governance-first design discipline**:

**Process decomposition prevents skills that cross control boundaries.** In Controllership, a single skill that both prepares and routes a journal entry would violate the principle that preparation and approval must be separately accountable. The framework's decomposition rule naturally separates these into distinct skills with distinct guardrails.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with meeting-intel or daily-briefing |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, assemble context) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Controllership references ERP, close management system, ledger data, and policy repositories. Each must become a specific MCP tool call targeting SharePoint-hosted data, Graph Connector sources, or manual input paths.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation gates provide human-in-the-loop; no formal approval workflow engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation via quality rubric and manual testing |

**The key gap for Controllership:** There is no built-in segregation-of-duties enforcement. This must be explicitly encoded in skill guardrails — the `fc-approval-routing` skill must check that the assigned approver is not the requestor or preparer, and refuse to route if the check fails.

### 3.3 M365 Artifacts as First-Class Process State

For Controllership, the artifact pattern is **Excel tracker + Word evidence packets + Outlook approval routing**. Each artifact type serves a specific role:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | **PRIMARY** — Process state store (journal tracker, case status, control checklist completion, audit log) | The journal tracker workbook IS the process state — skills read status, write updates, track approval progress, and log disposition |
| **Word** | **PRIMARY** — Evidence artifacts (journal packets, reviewer summaries, policy extracts, rationale memos) | Skills generate comprehensive packets that become the auditable record of what was assembled, reviewed, and cited |
| **SharePoint** | Source of truth (accounting policies, approval matrices, control checklists, journal standards, supporting schedules) | Skills read policies and reference data from SharePoint; uploaded evidence documents live here |
| **Outlook** | Communication channel for formal approval routing | Skills draft approval requests and follow-up communications; never auto-send |
| **Teams** | Coordination channel for close-period status updates | Skills post status updates and escalation notices to controllership channels |
| **Adaptive Card** | Decision-support presentation (gap reports, risk flags, approval path recommendations) | Skills present analysis for accountant review before any write action |
| **Graph API** | People and org data (profiles, hierarchy, approval authority) | Skills resolve approvers and verify segregation of duties |
| **Calendar** | Time-bound process events (close deadlines, review meetings) | Skills create calendar events when entries require escalated review discussion |

**The design pattern:** Controllership's dual-primary artifact pattern (Excel for tracking, Word for evidence) reflects the function's core requirement: every journal entry must have both a status trail (tracker) and an evidence trail (packet). The Cowork plugin enforces this by making both artifacts mandatory outputs of the preparation workflow.

### 3.4 Federated Connectors for Third-Party Systems

Controllership references ERP (ledger data, account metadata, posting status), close management system (close schedule, checklist status), and document repository (policies, standards). The tiered approach:

**Tier 1 — Graph Connectors:** For ERP platforms, Graph Connectors can index ledger balances, account metadata, and journal posting status into M365 Search. Skills access via `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])`.

**Tier 2 — SharePoint as Bridge:** Close management data, approval matrices, and threshold tables maintained in SharePoint workbooks populated by Power Automate. This is the recommended starting approach for the pilot.

**Tier 3 — Manual Input:** For ledger data points not available through integration, skills capture them via structured prompts and write them to the journal case record. The control checklist identifies exactly which data points are required.

**Practical recommendation:** Start with Tier 2 for all reference data (policies, matrices, checklists) and Tier 3 for ledger data that requires manual lookup. Introduce ERP Graph Connectors in Wave 2.

### 3.5 Governance in Cowork

Controllership governance is the strictest in the enterprise. The framework's model maps to Cowork as follows:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Controllership operations manager owns the process; finance platforms automation lead owns skill contracts |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC; posting authority never delegated to skills |
| **Segregation of duties** | Encoded in `fc-approval-routing` guardrails — preparer and approver must be different people; the skill refuses to route if the check fails |
| **Audit trail** | Every skill action logged with actor, timestamp, case linkage; Word packets preserve evidence chain; Excel tracker logs all status transitions |
| **SOX compliance** | No skill can post a journal entry; all skills produce reviewable drafts; approval authority follows the matrix; evidence citation is mandatory |
| **Policy versioning** | Accounting policies read dynamically from SharePoint at runtime; policy version and date included in every citation; changes to policies follow controllership change control |
| **Release management** | Skills versioned; quality rubric provides pre-deployment gate; prompt and instruction changes require controllership approval |

**The main governance gap** is that Cowork cannot directly post to the ERP. Journal entries prepared and approved through the plugin must still be posted by an authorized accountant in the ERP. This is actually a feature, not a bug — it preserves the segregation of duties between preparation and posting that SOX requires.

### 3.6 Evaluation Approach

**Component-level evaluation (per skill):**
- Missing-support detection rate — percentage of actual gaps identified by `fc-gap-risk-detection`
- Incorrect approval-path rate — how often `fc-approval-routing` recommends the wrong approver; target below 5%
- Draft summary acceptance rate — percentage of `fc-journal-comms` summaries accepted with minor edits; target above 75%
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones?

**Process-level evaluation (end-to-end):**
- Cycle time to review-ready state — time from journal request to complete, routed approval packet
- Rework rate — percentage of packets returned for additional evidence or correction
- Missing audit-field rate — must be zero
- Segregation violations — must be zero
- Override rate — how often accountants override skill recommendations; tracked for improvement
- Close-period timeliness — percentage of journal entries prepared before close deadline

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation and Evidence Assembly

**Infrastructure setup:**
- Create the shared Excel journal tracker workbook in SharePoint with standard columns (Journal Case ID, Description, Requestor, Entity, Business Unit, Debit Account, Credit Account, Amount, Currency, Close Period, Status, Created Date, Support Status, Approval Path, Assigned Reviewer, Disposition, Disposition Date)
- Upload accounting policies, journal entry standards, approval matrices, threshold tables, and control checklists to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-case evidence (supporting schedules, reconciliations, memos)
- Populate ledger reference data and account metadata in SharePoint (Tier 2 bridge) if feasible

**Skills to build:**
- `fc-journal-intake`
- `fc-journal-context`
- `fc-gap-risk-detection`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs presented via Adaptive Card or generated Word documents for manual review. Test with 10-15 real journal entry cases from one close period.

### Wave 2 — Routing and Communication

**Skills to build:**
- `fc-approval-routing`
- `fc-journal-comms`

**Promotions:**
- Promote `fc-gap-risk-detection` to write-back mode (updates Excel tracker after accountant confirmation)
- Promote `fc-journal-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a close-period scheduled prompt that checks for journal cases approaching the close deadline and surfaces any with incomplete evidence or missing approvals

**Operating posture:** AI draft plus approve for all routing and communication skills. Every output reviewed before action. Segregation of duties enforced in routing guardrails.

### Wave 3 — Optimization and Extended Coverage

**Enhancements:**
- Introduce Graph Connectors for ERP ledger data if available at the tenant level
- Add proactive detection of likely duplicate entries or reversal mismatches
- Add close checklist summary generation (aggregate view of all journal cases by status for close coordinator)
- Add policy-cited reviewer packet support with enhanced evidence formatting
- Refine all skills based on override patterns and accountant feedback from Waves 1-2

**Measurement:**
- Cycle time to review-ready state reduction vs. pre-pilot baseline
- Draft summary acceptance rate target: above 75%
- Missing-support detection rate target: above 85%
- Incorrect approval-path rate target: below 5%
- Rework rate target: below 15%
- Segregation violations: zero
- Missing audit fields: zero

---

## Part 5: Generalizing the Approach — Controllership Artifact Patterns

Controllership's primary artifact pattern is **dual-primary: Excel tracking + Word evidence packets**. This reflects the function's core requirement that every controlled action must have both a status trail and an evidence trail.

This pattern generalizes across Controllership sub-functions:

| Controllership Process | Primary Artifacts | Secondary Artifacts |
|---|---|---|
| Manual journal entry review | Excel (journal tracker) + Word (reviewer packet) | Outlook (approval routing), Adaptive Card (risk flags) |
| Close variance investigation | Excel (variance tracker) + Word (investigation memo) | PowerPoint (review deck), Teams (coordination) |
| Reconciliation exception triage | Excel (reconciliation tracker) + Word (exception packet) | Adaptive Card (gap report), Outlook (follow-up) |
| Close checklist coordination | Excel (checklist tracker) | Teams (status updates), Outlook (reminders) |
| Audit evidence preparation | Word (evidence packets) + Excel (evidence inventory) | SharePoint (document library), Outlook (requests) |

The cross-LOB method applies with a Controllership-specific constraint: **no skill may execute a posting, approval, or certification action**. Every Controllership skill must be bounded to preparation, analysis, and routing — final authority actions remain human-owned. This is not a limitation of the pilot phase; it is a permanent architectural constraint that reflects the nature of accounting governance.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic ERP, close management, and ledger references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly — prefer skills and tools; bounded multi-source assembly in Wave 3 |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint; segregation of duties enforced; no posting authority |
| Process State | SharePoint-hosted Excel workbook + Word evidence packets | Dual-primary state: tracker for status, packets for evidence |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) + Graph Connectors for ERP | Email request, close management event, SharePoint upload, scheduled prompt check |
| Approval | CreateDraftMessage + confirmation gates + approval matrix lookup + segregation check | Human-in-the-loop mandatory; approval authority from SharePoint matrix; preparer-approver separation enforced |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via detection rate and routing accuracy; process eval via cycle time, rework rate, and audit compliance |
