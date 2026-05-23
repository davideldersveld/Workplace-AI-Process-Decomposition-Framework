# Plan: Finance AP Exception Handling — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Finance line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the AP Invoice Exception Handling pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Finance, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — Finance references ERP, invoice capture, vendor master, and PO systems that must become M365 tool names or SharePoint bridge reads |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Finance's tight control requirements (SOX, segregation of duties) reinforce the need for explicit confirmation gates |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

Finance workflows are rules-heavy, audit-sensitive, and approval-bound. The framework's decomposition discipline is especially valuable here because it prevents the common anti-pattern of building a broad "finance copilot" that lacks the control granularity Finance teams require. By decomposing AP exception handling into 8 steps with explicit automation boundaries, each skill can enforce the exact level of human oversight that the control environment demands — from deterministic intake normalization to human-only final disposition.

---

## Part 2: The Finance AP Exception Handling Plugin — Skill-by-Skill Design

The Finance sample decomposes "AP Invoice Exception Handling" into 8 steps (FIN-AP-001 through FIN-AP-008), identifies 7 skills and 9 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| FIN-AP-001: Normalize invoice event | `fin-invoice-intake` | Data Aggregation | Deterministic automation | Excel (exception tracker), SharePoint (list), Outlook (notifications) |
| FIN-AP-002: Gather matching context | `fin-match-context` | Data Aggregation | AI act within policy | Word (context packet), SharePoint (PO/receipt data), Graph API (people) |
| FIN-AP-003: Classify exception type | `fin-exception-classify` | Decision Support | AI assist | Excel (tracker update), Adaptive Card (classification report) |
| FIN-AP-004: Assess risk and control path | `fin-control-path` | Decision Support | AI draft + approve | Adaptive Card (recommendation), Excel (risk assessment) |
| FIN-AP-005: Route to action owner | `fin-exception-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Excel (owner assignment) |
| FIN-AP-006: Draft outreach or approval packet | `fin-ap-comms` | Content Generation | AI draft + approve | Outlook (drafts), Word (approval summary), Teams (messages) |
| FIN-AP-007: Update system status | `fin-status-update` | Data Aggregation | AI act within policy | Excel (tracker status), SharePoint (audit log) |
| FIN-AP-008: Confirm disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `fin-invoice-intake` — Normalize Invoice Event

**Framework Step:** FIN-AP-001

**Trigger phrases:** "new invoice exception", "AP exception for invoice [number]", "invoice failed matching", "set up exception case for"

**Inputs:**
- Invoice number or ID
- Vendor name or ID
- Invoice amount and currency
- PO reference (if available)
- Exception source (three-way match failure, missing PO, vendor mismatch)

**M365 tools:**
- `SearchM365(sources=["files"])` — locate invoice document in SharePoint
- `SearchM365(sources=["email"])` — find supplier email thread for context
- `ReadFileContent` — read invoice data from SharePoint document library
- `GetDriveChildren` — check existing exception tracker for duplicates

**Output:** Structured exception case record written to Excel exception tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Exception Case ID, Invoice ID, Vendor ID, PO ID, Amount, Currency, Exception Source, Business Unit, Status, Created Date, Priority, Assigned To

**Guardrails:**
- Never create duplicate cases for the same invoice ID
- Validate that the invoice amount is a positive number
- Confirm details with user before writing to tracker
- Log case creation with timestamp, actor, and source data

---

#### 2. `fin-match-context` — Gather Matching Context

**Framework Step:** FIN-AP-002

**Trigger phrases:** "build context packet for invoice", "gather matching data for exception", "what context do we have for [invoice]", "assemble AP case packet"

**Inputs:**
- Exception case ID or invoice number

**M365 tools:**
- `ReadFileContent` — PO documents, goods receipt records, vendor profiles from SharePoint
- `SearchM365(sources=["files"])` — contracts, policy documents, prior exception history
- `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])` — ERP invoice and PO data (if Graph Connector available)
- `GetDriveChildren` — list supporting documents in vendor or invoice folder
- `SearchPeople` — resolve buyer, cost center owner contacts

**Output:** Word document containing:
- Invoice header and line details
- Purchase order data and status
- Goods receipt status
- Vendor profile and payment terms
- Relevant AP policy extracts
- Prior exception history for this vendor
- Key contacts (buyer, cost center owner, AP manager)

**Artifact:** Word (.docx) saved to SharePoint exception folder; serves as the case evidence packet

**Guardrails:**
- Cite source system and retrieval date for every data element
- Flag if any critical context (PO, receipt, vendor profile) is unfindable
- Never include bank account details or sensitive vendor financial data in the packet
- Mark data freshness — flag context older than 5 business days

---

#### 3. `fin-exception-classify` — Classify Exception Type

**Framework Step:** FIN-AP-003

**Trigger phrases:** "classify this exception", "what type of exception is this", "triage invoice exception", "categorize AP case"

**Inputs:**
- Context packet (Word doc or case data from Excel tracker)
- Exception taxonomy from SharePoint

**M365 tools:**
- `ReadFileContent` — exception taxonomy and classification rules from SharePoint
- `ReadFileContent` — context packet document
- `SearchM365(sources=["files"])` — prior classified exceptions for pattern matching

**Output:** Exception classification as Adaptive Card (for quick review) plus Excel tracker update (after confirmation)

**Classification categories:**
- Amount mismatch (PO vs. invoice)
- Missing goods receipt
- Missing or invalid PO
- Duplicate invoice candidate
- Vendor master mismatch
- Incomplete supporting documentation
- Tax or withholding discrepancy
- Multi-issue (flag dominant plus secondary)

**Logic:** Compare invoice data against PO, receipt, and vendor data from the context packet. Apply the exception taxonomy rules. Assign dominant exception type and confidence score. Flag missing information that prevents confident classification.

**Guardrails:**
- Present classification for AP analyst review before updating tracker — this is AI assist mode
- Include confidence score and rationale with every classification
- Flag cases with multiple conflicting exception types for manual triage
- Never auto-classify cases above the materiality threshold without analyst confirmation

---

#### 4. `fin-control-path` — Assess Risk and Control Path

**Framework Step:** FIN-AP-004

**Trigger phrases:** "assess risk for this exception", "what's the control path", "recommend approval route", "evaluate exception risk"

**Inputs:**
- Classified exception case
- AP policy and approval matrix from SharePoint
- Invoice amount and business unit

**M365 tools:**
- `ReadFileContent` — approval matrix, threshold table, escalation policy from SharePoint
- `ReadFileContent` — case data and classification from Excel tracker
- `SearchM365(sources=["files"])` — prior control path decisions for similar cases

**Output:** Control path recommendation as Adaptive Card with:
- Recommended route (auto-resolve, standard approval, escalated review, manual investigation)
- Required approvers based on amount and exception type
- Risk flags (SOX-sensitive, high-value, vendor mismatch, potential duplicate)
- Policy citations supporting the recommendation

**Guardrails:**
- Present recommendation for AP analyst review before any routing action — this is AI draft plus approve
- Never recommend auto-resolution for exceptions above the materiality threshold
- Always flag segregation-of-duties conflicts
- Cite the specific policy section and threshold for every routing recommendation
- Escalate to AP manager if no clear control path exists in the approval matrix

---

#### 5. `fin-exception-routing` — Route to Action Owner

**Framework Step:** FIN-AP-005

**Trigger phrases:** "route this exception", "assign to the right owner", "send to approver", "who handles this exception"

**Inputs:**
- Control path recommendation
- Exception case data
- Approval matrix (SharePoint document)

**M365 tools:**
- `SearchPeople` — resolve approver and action owner by role or function
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to assignees with case details
- `CreateDraftMessage` — Outlook draft for formal approval requests

**Output:** Routed work items:
- Teams messages to responsible parties with exception details and required actions
- Outlook draft for formal approval requests (never auto-sent)
- Updated Excel tracker with owner assignment, routing date, and expected resolution date

**Guardrails:**
- Present routing recommendations for AP analyst review before sending any messages
- Never route to someone outside the approved approval matrix
- Verify segregation of duties — the person who created the PO cannot approve the exception
- Escalate to AP operations manager if no clear owner is found
- Include exception case ID in every communication for audit traceability

---

#### 6. `fin-ap-comms` — Draft Outreach or Approval Packet

**Framework Step:** FIN-AP-006

**Trigger phrases:** "draft supplier email", "prepare approval packet", "write follow-up for missing receipt", "draft AP outreach", "invoice exception follow-up"

**Inputs:**
- Exception case data from Excel tracker
- Context packet (Word document)
- Control path recommendation
- Target audience (supplier, buyer, cost center owner, AP manager, controller)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages (for internal routing only)
- `SearchM365(sources=["files"])` — email templates and tone guides from SharePoint
- `ReadFileContent` — case evidence for citation in outreach

**Communication templates:**
- Missing receipt request to buyer
- Amount discrepancy clarification to supplier
- Missing PO follow-up to procurement
- Approval summary for cost center owner or AP manager
- Escalation notice to controller
- Case resolution confirmation

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Match tone to audience: professional for external supplier communication, operational for internal requests
- Include exception case ID, invoice number, and vendor ID in every communication
- Cite specific discrepancy details and supporting evidence
- Never include sensitive financial data (bank details, payment terms) in Teams messages — use Outlook for sensitive content
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable

---

#### 7. `fin-status-update` — Update System Status

**Framework Step:** FIN-AP-007

**Trigger phrases:** "update exception status", "mark case as [status]", "log resolution", "update AP tracker"

**Inputs:**
- Exception case ID
- New status (routing completed, awaiting approval, approved, rejected, escalated, resolved)
- Action taken and rationale

**M365 tools:**
- `ReadFileContent` — current tracker state from Excel
- `GetDriveChildren` — verify supporting documents exist in case folder
- SharePoint list write — update tracker row

**Output:** Updated Excel tracker with:
- New status
- Action taken
- Actor and timestamp
- Next action and expected date
- Resolution rationale (if closing)

**Guardrails:**
- Only update status based on a completed prior step (routing decision, approval, or analyst action)
- Log every status change with actor identity and timestamp
- Never move to "Resolved" status without documented disposition rationale
- Flag if the case has been in the same status for more than 2 business days (aging alert)

---

#### Step 8: Confirm Disposition (Human Only)

**Framework Step:** FIN-AP-008

This is not a Cowork skill. The framework correctly identifies final disposition confirmation as a human-only step. In Cowork, it is supported by:

- The `fin-ap-comms` skill — prepare the final disposition summary for reviewer sign-off
- The `fin-status-update` skill — record the final status after human decision
- The readiness summary artifacts — provide the evidence package for the approver
- Calendar support — book the review meeting if escalated disposition requires a call

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. This section analyzes how to approach that translation systematically for Finance.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Finance is **pre-implementation control discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle all AP exceptions. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills that score high on the quality rubric's Scope Boundaries dimension.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with meeting-intel or daily-briefing |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "check the ERP," the framework requires naming every input source. For Finance, this means replacing generic "ERP read" and "vendor master lookup" with specific MCP tool calls (`ReadFileContent`, `SearchM365`, `GetDriveChildren`) targeting SharePoint-hosted data or Graph Connector sources.

### 3.2 What Cowork Provides That the Framework Assumes You Build

The framework describes a 9-layer reference architecture. In Cowork, most of these layers are platform-provided:

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation gates provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through the quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (pending approvals, prior decisions, exception history) must be externalized to M365 artifacts.

### 3.3 M365 Artifacts as First-Class Process State

This is the most important architectural insight for making the framework useful in Cowork for Finance. Each M365 artifact type serves a specific role:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | **PRIMARY** — Process state store (exception tracker, case status, metrics, audit log) | The AP exception tracker workbook IS the process state — skills read current status, write updates, track aging, and log disposition |
| **Word** | Evidence artifacts (context packets, approval summaries, policy extracts) | Skills generate case packets that become the auditable record of what was assembled and reviewed |
| **SharePoint** | Source of truth (AP policies, approval matrices, vendor data, PO records, exception taxonomy) | Skills read policies and reference data from SharePoint; uploaded evidence documents live here |
| **Outlook** | Communication channel and signal source | Skills read supplier email for context; draft outreach and approval requests as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post task assignments, status updates, and escalation notices to AP channels or analyst chats |
| **Adaptive Card** | Decision-support presentation (classification results, risk assessments, routing recommendations) | Skills present analysis findings for analyst review before any write action |
| **Graph API** | People and org data (profiles, hierarchy, approval authority) | Skills resolve approvers, cost center owners, and escalation paths for routing decisions |
| **Calendar** | Time-bound process events (escalation meetings, SLA deadlines) | Skills create calendar events for review milestones when cases require escalated disposition |

**The design pattern:** The Cowork Finance plugin uses a SharePoint-hosted Excel workbook as the canonical exception tracker, with Word, Outlook, and Adaptive Card artifacts as the evidence trail. Each skill reads from and writes to this shared state through M365 tools. This approach gives AP teams artifacts they already know how to work with, while preserving the audit trail that Finance governance demands.

### 3.4 Federated Connectors for Third-Party Systems

The framework references ERP, invoice capture platform, vendor master, purchase order system, and goods receipt data — these do not exist natively in M365. The approach follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For ERP platforms (SAP, Oracle, Dynamics) and invoice capture platforms (Coupa, Ariba, Basware), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["erp-connector"])`. This provides read access to invoice records, PO data, and vendor profiles from external systems without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks populated by Power Automate flows from the ERP. Skills interact with the SharePoint copy. Bidirectional sync — particularly for status updates back to ERP — is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For data points with no integration path, skills provide structured intake that captures data from manual lookups, writing it into the shared Excel tracker. The framework's Signal Inventory identifies exactly which data points are needed, so the skill prompts for only what is missing.

**Practical recommendation for a Finance pilot:** Start with Tier 2 (SharePoint as bridge) for PO, receipt, and vendor data. ERP Graph Connectors require tenant admin setup and are better introduced in Wave 2 after skill workflows are proven. The Excel tracker serves as the Cowork-side state store regardless of which tier provides the upstream data.

### 3.5 Governance in Cowork

Finance requires tighter governance than many other business functions. The framework's governance model maps to Cowork as follows:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; the AP shared services manager owns the process definition; finance automation engineering owns the skill contracts |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC; segregation of duties encoded in skill guardrails |
| **Data classification** | Skill guardrails enforce handling rules: never include bank details in Teams messages, mask sensitive vendor financial data, restrict payment-related fields |
| **Audit** | Every write action logged with actor, timestamp, prior value, new value, and case linkage; SharePoint document library provides the evidence trail |
| **SOX compliance** | Skills enforce approval thresholds, segregation of duties, and evidence citation; no skill can bypass the approval matrix or auto-approve above materiality |
| **Release management** | Skills are versioned; the skill quality rubric provides a pre-deployment gate; prompt and policy changes follow controllership change control |
| **Policy enforcement** | Encoded in skill instructions ("always draft, never auto-send", "cite policy for every routing recommendation", "escalate above threshold"); dynamic policy read from SharePoint |

**The main governance gap** is bidirectional ERP state sync. If an AP analyst resolves an exception in Cowork (updating the Excel tracker), that resolution must also be reflected in the ERP. This requires Power Automate or a direct integration outside of Cowork. The recommended mitigation is to treat the SharePoint tracker as the coordination layer and use Power Automate to push final disposition status back to the ERP.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones?
- Classification accuracy — does `fin-exception-classify` assign the correct exception type? Target: 90% on benchmark set
- Output quality — do generated context packets contain accurate, cited information?
- Tool success rate — do M365 tool calls return expected results?

**Process-level evaluation (end-to-end):**
- Exception cycle time — time from case creation to final disposition
- First-pass resolution rate — percentage of exceptions resolved without rework
- Draft acceptance rate — percentage of `fin-ap-comms` drafts sent with minor edits only; target above 75%
- Incorrect routing rate — how often `fin-exception-routing` assignments need correction; target below 5%
- Unauthorized write actions — must be zero
- Missing audit fields — must be zero
- Override rate — how often AP analysts override skill recommendations; tracked for process improvement
- Duplicate payment incidents — must be zero attributable to skill actions

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation and Triage

**Infrastructure setup:**
- Create the shared Excel AP exception tracker workbook in SharePoint with standard columns (Exception Case ID, Invoice ID, Vendor ID, PO ID, Amount, Currency, Exception Source, Business Unit, Status, Created Date, Priority, Assigned To, Classification, Confidence Score, Resolution Date, Disposition)
- Upload AP policies, approval matrices, exception taxonomy, and threshold tables to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-case evidence (context packets, supporting documents)
- Populate PO, receipt, and vendor reference data in SharePoint (Tier 2 bridge) via Power Automate from ERP

**Skills to build:**
- `fin-invoice-intake`
- `fin-match-context`
- `fin-exception-classify`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs presented via Adaptive Card or generated documents for manual review. Test with one AP queue or business unit, 10-20 real exception cases.

### Wave 2 — Communication and Routing

**Skills to build:**
- `fin-control-path`
- `fin-exception-routing`
- `fin-ap-comms`

**Promotions:**
- Promote `fin-exception-classify` to write-back mode (updates Excel tracker after analyst confirmation)
- Promote `fin-invoice-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for exception cases approaching the 2-business-day SLA and surfaces any with incomplete status or missing assignments

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action. Segregation of duties enforced in routing guardrails.

### Wave 3 — Status Management and Optimization

**Skills to build:**
- `fin-status-update`

**Enhancements:**
- Introduce Graph Connectors for ERP data if available at the tenant level
- Add proactive monitoring via scheduled prompt: flag aging exceptions, highlight recurring vendor issues, identify exception patterns that suggest process improvements
- Add approval summary generation for controller-level review
- Refine all skills based on override patterns and analyst feedback from Waves 1-2

**Measurement:**
- Exception cycle time reduction vs. pre-pilot baseline
- Classification accuracy target: above 90%
- Draft acceptance rate target: above 75%
- Incorrect routing rate target: below 5%
- Unauthorized write actions: zero
- Missing audit fields: zero
- Duplicate payment incidents: zero

---

## Part 5: Generalizing the Approach — Finance Artifact Patterns

Finance's primary artifact pattern is **Excel-centric with Word evidence and Outlook communication**. Financial data is inherently tabular and auditable — the exception tracker workbook serves simultaneously as the process state store, the audit log, and the operational dashboard.

This pattern generalizes across Finance sub-functions:

| Finance Process | Primary Artifact | Secondary Artifacts |
|---|---|---|
| AP exception handling | Excel (exception tracker) | Word (context packets), Outlook (outreach), Adaptive Card (classification) |
| Vendor master changes | Excel (change request tracker) | Word (approval packets), Teams (notifications) |
| Expense audit | Excel (audit tracker) | Outlook (clarification drafts), Adaptive Card (risk flags) |
| Journal entry approval | Excel (journal tracker) | Word (approval summaries), Outlook (routing) |
| Month-end variance | Excel (variance analysis) | PowerPoint (review deck), Word (investigation notes) |

The cross-LOB method applies: decompose into bounded steps, assign automation boundaries, map signals to M365 tools, identify the canonical tracker artifact (Excel for Finance), define first-class outputs, and encode governance in guardrails. Finance's distinguishing requirement is that every skill must support auditability, segregation of duties, and policy citation — these are non-negotiable guardrail patterns for any Finance Cowork plugin.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic ERP, invoice capture, and vendor master references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly — prefer skills and tools; introduced in Wave 3 for complex multi-system packet assembly |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint; SOX and segregation rules enforced |
| Process State | SharePoint-hosted Excel workbook | Durable state externalized to the AP exception tracker workbook |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) + Graph Connectors for ERP | Email arrival, Teams message, SharePoint upload, scheduled prompt check |
| Approval | CreateDraftMessage + confirmation gates + approval matrix lookup | Human-in-the-loop via Cowork's review-before-action patterns; approval authority from SharePoint matrix |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis and classification accuracy; process eval via cycle time, draft acceptance rate, and audit compliance |
