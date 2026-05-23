# Plan: Procurement Supplier Onboarding and Risk Review — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Procurement line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Supplier Onboarding and Risk Review pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Procurement, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — Procurement references "vendor master", "risk screening tools", and "sourcing platform" that must become specific M365 tool names or Graph Connector queries |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but Procurement adds spend authority thresholds and supplier data sensitivity that must be encoded as explicit guardrails |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the task patterns map to Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts"; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

Procurement workflows are document-heavy and control-intensive. The framework's automation boundary analysis is especially valuable here because Procurement has explicit approval authorities, spend thresholds, and regulatory constraints (sanctions screening, tax compliance) that must be preserved. The skill author's job is to define capabilities that accelerate document assembly and risk detection while keeping approval authority firmly human-owned.

---

## Part 2: The Procurement Supplier Onboarding Plugin — Skill-by-Skill Design

The Procurement sample decomposes "Supplier Onboarding and Risk Review" into 6 steps (PR-001 through PR-006), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| PR-001: Normalize supplier request | `proc-supplier-intake` | Data Aggregation | Deterministic automation | Excel (tracker), SharePoint (list), Outlook (notifications) |
| PR-002: Gather supplier and policy context | `proc-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (packet), SharePoint (policies, checklists), Graph API (people) |
| PR-003: Detect missing items and risk indicators | `proc-gap-risk-detect` | Decision Support | AI assist | Excel (checklist), SharePoint (documents), Adaptive Card (report) |
| PR-004: Determine review path | `proc-review-routing` | Decision Support | AI draft + approve | Teams (messages), Excel (approval matrix), Graph API (org hierarchy) |
| PR-005: Draft outreach and evidence summary | `proc-supplier-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (reviewer summary) |
| PR-006: Confirm onboarding disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `proc-supplier-intake` — Normalize Supplier Request

**Framework Step:** PR-001

**Trigger phrases:** "new supplier request", "onboard supplier [name]", "set up supplier case for", "supplier onboarding for [company]"

**Inputs:**
- Supplier company name
- Requester name or email
- Spend category
- Geography or region
- Urgency or priority indicator

**M365 tools:**
- `SearchPeople` — resolve requester identity and procurement analyst assignment
- `GetUserDetails` — pull requester profile and department context
- `SearchM365(sources=["email"])` — find original supplier request thread for context
- `SearchM365(sources=["files"])` — check for existing supplier records to detect duplicates
- `ReadFileContent` — read existing onboarding tracker to verify no duplicate case exists

**Output:** Structured case record written to Excel onboarding tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Supplier Name, Requester, Spend Category, Geography, Priority, Status, Created Date, Assigned Analyst, Risk Tier, Target Completion Date

**Guardrails:**
- Never create duplicate cases for the same supplier name and requester combination
- Validate that spend category matches the organization's approved taxonomy
- Confirm details with user before writing to tracker
- Log case creation with actor and timestamp for audit trail

---

#### 2. `proc-context-packet` — Gather Supplier and Policy Context

**Framework Step:** PR-002

**Trigger phrases:** "build supplier packet for", "assemble onboarding context for [supplier]", "what do we need for [supplier] onboarding", "gather supplier context"

**Inputs:**
- Onboarding case ID or supplier name

**M365 tools:**
- `SearchM365(sources=["files"])` — locate onboarding checklists, category-specific policies, geography-specific requirements
- `ReadFileContent` — read SharePoint policy documents (sanctions screening requirements, insurance minimums, tax documentation rules)
- `SearchM365(sources=["connectors"], connector_ids=["vendor-master-connector"])` — pull existing vendor master data if Graph Connector is available
- `GetDriveChildren` — list documents already uploaded to the supplier's onboarding folder
- `GetUserDetails` — resolve category manager and risk reviewer contacts

**Output:** Word document containing:
- Supplier profile summary (name, category, geography, requested services)
- Applicable policy requirements by category and geography
- Required document checklist (W-9, insurance certificate, banking forms, contracts)
- Screening requirements (sanctions, debarment, conflict of interest)
- Assigned reviewer contacts (category manager, risk reviewer, legal, AP)

**Artifact:** Word (.docx) saved to SharePoint supplier onboarding folder

**Guardrails:**
- Cite policy source and version for every requirement listed
- Flag if any required policy document is missing from SharePoint
- Never include bank account details or sensitive tax identifiers in the context packet
- Mark the packet as draft until reviewed by the assigned analyst

---

#### 3. `proc-gap-risk-detect` — Detect Missing Items and Risk Indicators

**Framework Step:** PR-003

**Trigger phrases:** "check supplier gaps", "what's missing for [supplier]", "supplier risk check", "audit supplier case", "onboarding completeness check"

**Inputs:**
- Context packet (Word doc or case data from Excel tracker)

**M365 tools:**
- `ReadFileContent` — onboarding checklist from SharePoint
- `SearchM365(sources=["files"])` — uploaded supplier documents (W-9, insurance certs, banking forms)
- `GetDriveChildren` — supplier onboarding folder contents to verify what has been uploaded
- `SearchM365(sources=["connectors"], connector_ids=["risk-screening-connector"])` — pull screening results if Graph Connector is available

**Output:** Gap and risk report as Adaptive Card (for quick review) plus Excel worksheet update (for tracking)

**Logic:** Compare required checklist items against uploaded documents in the supplier's onboarding folder. Flag missing items, approaching deadlines, and risk indicators such as sanctioned entity matches, missing insurance coverage, expired certifications, or duplicate supplier patterns.

**Guardrails:**
- Never mark a checklist item as complete without document evidence in the SharePoint folder
- Flag sanctions, debarment, or compliance risk separately with elevated visibility and explicit "requires human review" label
- Present findings for user review before updating the Excel tracker — read-only presentation via Adaptive Card
- Never suppress or downgrade a risk flag — all flags must be visible to the analyst
- Include confidence level for risk indicators derived from document analysis versus screening systems

---

#### 4. `proc-review-routing` — Determine Review Path

**Framework Step:** PR-004

**Trigger phrases:** "route supplier for review", "who needs to review [supplier]", "assign reviewers for supplier case", "determine review path"

**Inputs:**
- Gap and risk report output
- Onboarding case data
- Approval matrix (SharePoint document)

**M365 tools:**
- `ReadFileContent` — approval matrix and routing rules from SharePoint
- `SearchPeople` — resolve reviewer identities by role or function
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to recommended reviewers (after approval)
- `CreateEvent` — optional deadline reminders on reviewer calendars (after approval)

**Output:** Routing recommendation presented via Adaptive Card:
- Recommended reviewers by type (risk, legal, tax, AP, category manager)
- Routing rationale citing specific policy triggers
- Estimated review timeline based on case priority
- After user approval: Teams messages to assigned reviewers, calendar holds for deadlines

**Guardrails:**
- Present routing recommendations for procurement analyst review before sending any messages or calendar invites
- Never auto-assign reviewers outside the documented approval matrix
- Escalate to procurement operations manager if no clear reviewer is found for a required review type
- For high-risk suppliers (sanctions flags, high-spend categories), require explicit category manager approval before routing
- Include spend authority threshold context — if estimated spend exceeds a tier boundary, flag for elevated review path

---

#### 5. `proc-supplier-comms` — Draft Outreach and Evidence Summary

**Framework Step:** PR-005

**Trigger phrases:** "draft supplier email for", "remind [requester] about missing documents", "prepare reviewer summary for [supplier]", "supplier follow-up", "send missing docs request"

**Inputs:**
- Case data from Excel tracker
- Context packet (Word document)
- Gap and risk report
- Target audience (supplier contact, requester, reviewer, AP specialist)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages (after confirmation)
- `SearchM365(sources=["files"])` — email templates and standard language from SharePoint
- `ReadFileContent` — communication templates for different outreach types

**Communication templates:**
- Missing document request to requester or supplier contact
- Reviewer assignment notification with case summary
- Risk flag escalation notice to category manager
- AP onboarding handoff with bank detail collection instructions
- Status update to requester on onboarding progress

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Never include bank account details, tax identifiers, or screening results in outgoing communications
- Match tone to audience: professional for external supplier-facing, operational for internal reviewer notifications
- Include onboarding case reference number in every communication
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable before sending

---

#### 6. `proc-reviewer-packet` — Summarize Reviewer Packet

**Framework Step:** Derived from PR-005 (evidence summary component)

**Trigger phrases:** "prepare review packet", "summarize supplier case for review", "onboarding status for [supplier]", "build approval packet"

**Inputs:**
- All prior artifacts: case record (Excel), context packet (Word), gap report, routing assignments, communication history

**M365 tools:**
- `ReadFileContent` — all case artifacts from SharePoint
- `SearchM365(sources=["email"])` — recent correspondence thread
- `GetDriveChildren` — full inventory of uploaded evidence documents
- Document generation: Word (.docx) for formal summary, Excel update for tracker status

**Output options:**
- **Adaptive Card** — Quick status view for chat-based review
- **Word document** — Formal reviewer packet with case summary, risk findings, checklist status, and evidence inventory
- **Excel update** — Tracker status set to "Ready for Review" or "Blocked — [reason]"

**Guardrails:**
- Every finding must trace to a source artifact — no fabricated status information
- Clearly distinguish complete versus incomplete versus blocked items in all output formats
- Flag any items requiring exception approval with explicit callouts
- Mask sensitive financial details (bank routing numbers, tax IDs) in summary documents
- Present summary for review before finalizing

---

#### Step 6: Confirm Onboarding Disposition (Human Only)

**Framework Step:** PR-006

This is not a Cowork skill. The framework correctly identifies final supplier approval as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the review meeting with required approvers
- The reviewer packet artifacts — provide the evidence package for the decision maker
- The `proc-supplier-comms` skill — send the approval or rejection notification after the human decision is made

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Procurement is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle the entire supplier onboarding lifecycle. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills. Separating gap detection (PR-003) from review routing (PR-004) is a prime example: these are distinct decision types (risk identification versus control path selection) that would become muddled in a single skill.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Procurement signals like "supplier follow-up email", "W-9 uploaded to SharePoint", and "risk screening completed" translate to specific MCP tool calls rather than vague instructions.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation patterns provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. Procurement's "onboarding case state" (pending documents, in review, approved, rejected) must be externalized to M365 artifacts — specifically the Excel tracker in SharePoint.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the Procurement process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (onboarding tracker, checklist status, approval matrix) | The onboarding tracker workbook IS the process state — skills read current status, write updates, track completeness percentage, and record review assignments |
| **SharePoint** | Source of truth (policies, checklists, approval matrices, uploaded evidence) | Skills read onboarding requirements and control policies from SharePoint; supplier evidence documents (W-9, insurance, contracts) are stored here |
| **Word** | Evidence artifacts (context packets, reviewer summaries, policy extracts) | Skills generate packets that become the auditable record of what was assembled and reviewed — the reviewer reads a Word packet, not raw data |
| **Outlook** | Communication channel and signal source | Skills read incoming supplier correspondence for context; draft outgoing requests and notifications as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post reviewer assignments, status updates, and escalation notices to procurement channels or chats |
| **Graph API** | People and org data (profiles, hierarchy, category managers) | Skills resolve reviewer identities, org structure, and approval authority chains |
| **Calendar** | Time-bound process events (deadlines, review meetings) | Skills create calendar events for review milestones and SLA deadline reminders |
| **Adaptive Card** | In-session decision support (gap reports, routing recommendations) | Skills present structured findings for analyst review before any write action |

**The design pattern:** The Cowork Procurement plugin uses a SharePoint-hosted Excel workbook as the canonical case tracker, with Word documents as the evidence packets, SharePoint folders as the document repository, and Outlook drafts as the reviewable communication layer. Each skill reads from and writes to this shared state through M365 tools.

**Primary artifact pattern for Procurement: Excel + SharePoint + Word.** Procurement is document-evidence-heavy (uploaded forms, certificates, contracts) with structured tracking (case status, checklist completion, reviewer assignments) and formal written outputs (reviewer packets, policy-cited summaries).

### 3.4 Federated Connectors for Third-Party Systems

The Procurement source file references systems that do not exist natively in M365: vendor master, sourcing platform, risk screening tools, and workflow systems. The approach follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For ERP vendor master systems (SAP MM, Oracle Supplier Hub), procurement platforms (Coupa, SAP Ariba, Jaggaer), and risk screening services (Dun and Bradstreet, World-Check, LexisNexis), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["sap-vendor-connector"])`. This provides read access to supplier profiles, screening status, and onboarding workflow state without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks populated by Power Automate flows from the third-party system. The approval matrix, risk screening results, and vendor classification data can live in SharePoint as regularly refreshed exports. Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For screening tools or niche systems with no integration path, skills provide structured intake that captures data from manual lookups. The framework's Signal Inventory identifies exactly which data points are needed (screening status, insurance expiry, bank verification result), so the skill can prompt for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge for approval matrices and checklists) and Tier 3 (manual input for screening results). Graph Connectors for ERP and risk platforms require tenant admin setup and are better introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

Procurement-specific governance concerns require targeted guardrail design:

| Governance Domain | Cowork Implementation | Procurement-Specific Concern |
|---|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure | Procurement operations manager owns the skill suite; category managers own policy content |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC | Supplier bank details and tax identifiers require restricted access — skills must never surface these in Teams messages or Adaptive Cards |
| **Data classification** | Skill guardrails enforce handling rules | Bank routing numbers, EIN/SSN, insurance policy numbers are sensitive — mask in all generated artifacts except dedicated secure forms |
| **Spend authority** | Encoded in skill instructions and approval matrix | Skills must reference spend tier thresholds when recommending review paths — a $50K supplier versus a $5M supplier triggers different approval chains |
| **Audit** | Platform logs tool invocations; SharePoint artifacts provide document trail | Every case action (create, update, route, approve) must be traceable to an actor and timestamp — the Excel tracker serves as the audit log |
| **Sanctions and compliance** | Risk flags surfaced but never suppressed | Skills must never auto-clear a risk flag — all sanctions, debarment, or compliance alerts require human review and documented disposition |
| **Release management** | Skills versioned in OneDrive; quality rubric provides pre-deployment gate | Policy changes (new screening requirements, updated approval thresholds) should live in SharePoint documents read dynamically, not hard-coded in skill instructions |

### 3.6 Evaluation Approach

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do generated packets contain accurate, policy-cited information? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Onboarding packet completeness rate — percentage of cases with all required documents and reviews identified
- Incorrect review-path rate — how often the routing recommendation must be overridden
- Draft acceptance rate — percentage of `proc-supplier-comms` drafts sent without major edits
- Cycle time to review-ready state — time from case creation to "Ready for Review" status (target: 3 business days)
- Duplicate supplier detection rate — percentage of duplicate requests caught at intake
- Risk flag miss rate — percentage of genuine risk indicators not surfaced by `proc-gap-risk-detect`
- Unauthorized approval incidents — any case where approval bypassed the documented authority chain

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel onboarding tracker workbook in SharePoint with standard columns (Case ID, Supplier Name, Requester, Spend Category, Geography, Priority, Status, Created Date, Assigned Analyst, Risk Tier, Target Completion Date, Checklist Completion %)
- Upload onboarding policies, category-specific checklists, geography-specific requirements, and the approval matrix to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-supplier onboarding evidence (W-9, insurance, banking, contracts)

**Skills to build:**
- `proc-supplier-intake`
- `proc-context-packet`
- `proc-gap-risk-detect`
- `proc-supplier-comms` (draft mode only)

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5-10 real onboarding cases.

### Wave 2 — Routing and Policy-Cited Outputs

**Skills to build:**
- `proc-review-routing`
- `proc-reviewer-packet`

**Promotions:**
- Promote `proc-supplier-intake` to write mode (creates case records after confirmation)
- Promote `proc-gap-risk-detect` to write-back mode (updates Excel tracker after user confirmation)
- Promote `proc-supplier-comms` to send-after-approval mode for internal reviewer notifications

**Automation:**
- Set up a daily scheduled prompt that checks for onboarding cases approaching SLA deadline (3 business days) and surfaces any with incomplete status or unassigned reviewers

**Operating posture:** AI draft plus approve for routing and communication skills. Policy-cited outputs reference specific SharePoint documents. Every output reviewed before action.

### Wave 3 — Advanced Assembly and Proactive Detection

**Enhancements:**
- Introduce Graph Connectors for vendor master and risk screening data if available at the tenant level
- Add bounded multi-reviewer packet assembly — skill assembles a complete approval packet pulling from multiple reviewers' findings
- Add proactive detection of likely stalled onboarding cases via scheduled prompt: flag cases with no activity for 2+ business days, highlight recurring missing document patterns, suggest process improvements
- Refine all skills based on override patterns and reviewer feedback from Waves 1-2

**Measurement:**
- Cycle time reduction versus pre-pilot baseline (target: 30% improvement)
- Draft acceptance rate target: above 70%
- Gap detection accuracy target: above 85%
- Incorrect routing rate target: below 10%
- Duplicate detection rate target: above 90%

---

## Part 5: Generalizing the Approach — Procurement's Artifact Pattern

Procurement's primary M365 artifact pattern is **Excel + SharePoint + Word**. This reflects three characteristics of procurement workflows:

1. **Structured tracking** — supplier cases, checklist completion, and approval status are inherently tabular and live in Excel
2. **Document-heavy evidence** — W-9 forms, insurance certificates, contracts, and banking forms are stored in SharePoint and must be inventory-checked by skills
3. **Formal written outputs** — reviewer packets and policy-cited summaries are Word documents that become the auditable record

This pattern differs from communication-heavy LOBs (Customer Service, IT Service Management) where Teams and Outlook dominate, and from presentation-heavy LOBs (Sales, Marketing) where PowerPoint is primary. The cross-LOB method remains the same: decompose the process, assign automation boundaries, map signals to M365 tools, identify the primary artifact, and build Wave 1 as read-only.

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
| Process State | SharePoint-hosted Excel workbook or SharePoint list | Durable state externalized to M365 artifacts |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, form submission, calendar event |
| Approval | Confirmation gates + CreateDraftMessage + reviewer routing | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time and acceptance rate |
