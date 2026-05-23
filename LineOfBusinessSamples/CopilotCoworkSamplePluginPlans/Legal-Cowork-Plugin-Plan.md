# Plan: Legal Contract Intake and Clause Deviation Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Legal line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Contract Intake and Clause Deviation Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Legal, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Legal's low error tolerance elevates the importance of guardrails. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope principle. Legal's document-centric steps decompose cleanly. |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "CLM", "playbook repository", and "ticketing system" that must become specific M365 tool names or SharePoint-bridged data stores. |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Legal's privilege and confidentiality requirements demand that guardrails be especially strict. Every write action requires explicit confirmation. |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — clause deviation detection maps to Decision Support; summary drafting maps to Content Generation; context assembly maps to Data Aggregation. |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal tool contracts; orchestration is implicit in skill instructions. Legal's approval gates must be encoded as guardrails rather than a workflow engine. |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides the runtime layers natively. The skill author controls capability definition and decision logic. Legal-specific privilege scoping relies on M365 tenant RBAC. |

### Key Insight

Legal workflows are document-heavy, privilege-sensitive, and approval-bound. The framework's decomposition discipline is especially valuable here because it prevents the common anti-pattern of building one broad "contract review" skill that conflates intake, analysis, routing, and communication. By decomposing into six steps, each skill has a clear scope, a defined automation boundary, and explicit guardrails that preserve legal judgment and attorney-client privilege.

---

## Part 2: The Legal Contract Triage Plugin — Skill-by-Skill Design

The Legal sample decomposes "Contract Intake and Clause Deviation Triage" into 6 steps (LG-001 through LG-006), identifies 6 skills, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| LG-001: Normalize contract request | `legal-contract-intake` | Data Aggregation | Deterministic automation | Excel (case tracker), SharePoint (list), Outlook (request emails) |
| LG-002: Gather contract and playbook context | `legal-review-packet` | Data Aggregation + Content Generation | AI act within policy | Word (review packet), SharePoint (playbooks, policies), Graph API (people) |
| LG-003: Identify deviations and risk issues | `legal-deviation-detection` | Decision Support | AI assist | Word (contract text), SharePoint (playbook), Adaptive Card (deviation report) |
| LG-004: Determine review path and approvals | `legal-review-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Excel (approval matrix) |
| LG-005: Draft summary and follow-up requests | `legal-triage-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (summary documents) |
| LG-006: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (disposition confirmation) |

### Detailed Skill Designs

#### 1. `legal-contract-intake` — Normalize Contract Request

**Framework Step:** LG-001

**Trigger phrases:** "new contract request", "contract intake for [counterparty]", "set up legal case for", "log contract request from [requestor]", "incoming contract from"

**Inputs:**
- Requestor name or email
- Counterparty name
- Contract type (NDA, MSA, SOW, amendment, etc.)
- Business unit
- Urgency level
- Attached contract document or redline

**M365 tools:**
- `SearchPeople` — resolve requestor and business contacts
- `GetUserDetails` — pull requestor profile and department
- `SearchM365(sources=["email"])` — find the original request email thread for context
- `SearchM365(sources=["files"])` — check for attached contract document in recent uploads
- `ReadFileContent` — read SharePoint-hosted case tracker to check for duplicate cases

**Output:** Structured case record written to Excel contract case tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Counterparty, Contract Type, Business Unit, Requestor, Urgency, Status, Received Date, Assigned Counsel, Deviation Count, Disposition

**Guardrails:**
- Never create duplicate cases for the same counterparty and contract type within 30 days
- Validate that all required fields are populated before writing
- Confirm details with user via Adaptive Card before writing to tracker
- Never extract or display terms from attached contracts at this stage — intake only

---

#### 2. `legal-review-packet` — Gather Contract and Playbook Context

**Framework Step:** LG-002

**Trigger phrases:** "build review packet for", "assemble contract context", "pull playbook for [contract type]", "what's the standard for [clause type]", "prepare review materials"

**Inputs:**
- Case ID or counterparty name
- Contract type (determines which playbook to retrieve)

**M365 tools:**
- `GetUserDetails` — requestor and business unit context
- `GetManagerDetails` — reporting chain for approval path awareness
- `SearchM365(sources=["files"])` — clause playbooks, fallback position documents, approved templates
- `ReadFileContent` — SharePoint-hosted playbook documents, policy standards, and the contract itself
- `GetDriveChildren` — browse the legal templates library for applicable standard forms

**Output:** Word document containing:
- Contract summary (parties, type, jurisdiction, effective date)
- Applicable clause playbook extracts with approved fallback positions
- Standard template comparison baseline
- Counterparty history (prior agreements found via search)
- Required approvals based on contract type and value thresholds
- Requestor and business unit context

**Artifact:** Word (.docx) saved to SharePoint legal matters folder

**Guardrails:**
- Cite playbook source and version for every clause standard referenced
- Never include attorney work product or privileged analysis in the packet — this is factual context assembly only
- Flag if any required playbook document is not found in SharePoint
- Restrict access to the generated packet to members of the legal team SharePoint group
- Never summarize or interpret contract terms — present source text with references

---

#### 3. `legal-deviation-detection` — Identify Deviations and Risk Issues

**Framework Step:** LG-003

**Trigger phrases:** "check for deviations", "clause deviation analysis for", "what deviates from playbook", "flag non-standard terms in", "review contract against playbook"

**Inputs:**
- Review packet (Word document or case data)
- Contract document (Word or PDF from SharePoint)

**M365 tools:**
- `ReadFileContent` — contract document and clause playbook from SharePoint
- `SearchM365(sources=["files"])` — approved fallback language, prior exception approvals for this counterparty
- `GetDriveChildren` — check the matter folder for supplemental exhibits or amendments

**Output:** Deviation report as Adaptive Card (for quick review) plus detailed Word document (for the case file)

**Logic:** Compare each material clause in the contract against the playbook's approved position and acceptable fallback. Classify deviations by severity: (1) within approved range, (2) requires counsel review, (3) requires senior counsel or business approval, (4) outside policy — must escalate. Cross-reference against prior exceptions for the same counterparty.

**Guardrails:**
- Present all findings as "deviations from playbook" — never characterize findings as legal advice or risk assessments
- Every deviation must cite the specific playbook section and the specific contract clause
- Never recommend accepting or rejecting a clause — present the deviation and the playbook standard
- Flag privilege-sensitive clauses (indemnification, limitation of liability, IP assignment) with elevated visibility
- Present findings for user review via Adaptive Card before writing the deviation report to the case file
- Read-only mode — this skill does not modify the contract or any tracker; it only surfaces analysis

---

#### 4. `legal-review-routing` — Determine Review Path and Approvals

**Framework Step:** LG-004

**Trigger phrases:** "route this contract for review", "who reviews [contract type]", "assign reviewers for", "determine approval path for", "set up review chain"

**Inputs:**
- Deviation report output
- Contract case data (type, value, counterparty, jurisdiction)
- Approval matrix (SharePoint document)

**M365 tools:**
- `SearchPeople` — resolve counsel names by practice area or specialty
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `ReadFileContent` — approval matrix and routing rules from SharePoint
- `PostMessage` — Teams notification to assigned reviewers (after approval)
- `CreateDraftMessage` — Outlook draft for formal review assignment (after approval)

**Output:** Routing recommendation:
- Assigned counsel based on contract type and deviation severity
- Required approvers based on value thresholds and deviation categories
- Escalation path if deviations exceed standard authority
- Timeline recommendation based on urgency and SLA

**Guardrails:**
- Present routing recommendation for legal operations review before sending any messages or assignments
- Never auto-assign outside the approved reviewer matrix
- Escalate to senior counsel or legal operations manager if deviation severity is at level 3 or 4
- Never reveal deviation details in Teams messages — use references to the case ID and a link to the SharePoint matter folder
- Log all routing decisions to the Excel case tracker with timestamp and rationale

---

#### 5. `legal-triage-comms` — Draft Summary and Follow-Up Requests

**Framework Step:** LG-005

**Trigger phrases:** "draft contract summary for", "prepare triage summary", "send missing info request for", "draft follow-up to [requestor]", "legal case update for"

**Inputs:**
- Case data from Excel tracker
- Review packet (Word document)
- Deviation report
- Target audience (counsel, requestor, business approver, compliance)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages to legal ops channel (after confirmation)
- `SearchM365(sources=["files"])` — email templates and standard response language from SharePoint
- `ReadFileContent` — case artifacts for summary content

**Communication templates:**
- Triage summary to assigned counsel (deviation overview, key issues, recommended focus areas)
- Missing information request to business requestor
- Status update to business unit stakeholder
- Escalation notice to senior counsel or legal operations manager
- Compliance referral if regulatory clause deviations are detected

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Never include privileged analysis, attorney work product, or internal legal strategy in external-facing communications
- Never include full clause text in email summaries — reference the matter folder and case ID
- Match tone to audience: formal and precise for counsel, clear and non-technical for business requestors
- Include case reference number in every communication
- Mark all counsel-facing drafts with "PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT" header

---

#### Step 6: Confirm Triage Disposition (Human Only)

**Framework Step:** LG-006

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the triage review meeting with counsel
- The triage summary artifacts — provide the evidence package for the reviewer
- The `legal-triage-comms` skill — send the disposition confirmation after the human decision is made
- The Excel case tracker — updated to reflect final disposition (queued for review, escalated, or returned)

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Legal is **decomposition discipline and boundary enforcement**:

**Process decomposition prevents mega-skills.** The common anti-pattern for legal AI is building one broad "contract review" tool. The framework's rule — "keep breaking down until each step has one dominant goal" — produces well-scoped skills that separate intake from analysis from routing from communication. This is critical in legal work where privilege, confidentiality, and approval authority must be cleanly delineated.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions. Deviation detection operates here. |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation. Routing and communications operate here. |
| AI act within policy | Skill can execute bounded read and retrieval actions within defined rules. Context assembly operates here. |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed. Intake normalization operates here. |

**Signal inventory forces explicit M365 tool selection.** Instead of vague references to "CLM" or "playbook repository," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365`, `ReadFileContent`, `GetDriveChildren` — a direct quality improvement for legal skills where source traceability is non-negotiable.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which process step is active |
| Capability registry | Built-in — skills are discovered and versioned in the skill directory |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal legal approval workflow engine |
| Governance and control | Partial — skill instructions encode policies; M365 tenant RBAC enforces access boundaries |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. Legal case state (pending reviews, deviation dispositions, approval decisions) must be externalized to M365 artifacts — primarily the Excel case tracker and SharePoint matter folders.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the legal contract triage process:

| Artifact | Role in Legal Workflow | How Skills Use It |
|---|---|---|
| **Word** (PRIMARY) | Contract documents, review packets, deviation reports, triage summaries, playbook extracts | The primary work product of legal skills — contracts are Word documents, analysis is written in Word, summaries are Word documents. Skills read contracts, generate packets, and produce deviation reports as Word artifacts. |
| **Excel** | Case tracker (matter status, assignments, deviation counts, disposition) | The canonical process state store — skills read current case status and write updates after confirmation |
| **SharePoint** | Source of truth for playbooks, policies, templates, approved fallback language, matter folders | Skills read authoritative clause standards and store all case artifacts in per-matter folders |
| **Outlook** | Communication channel, request intake signal, draft outreach | Skills read incoming contract requests; draft all external and counsel-facing communications as reviewable Outlook drafts |
| **Teams** | Internal legal coordination and status routing | Skills post task assignments and status updates to legal ops channels after confirmation |
| **Adaptive Card** | Quick-review presentation for deviation findings and routing recommendations | Skills present analysis for rapid review without requiring the user to open a full document |
| **Graph API** | People data — requestor profiles, counsel lookup, org hierarchy | Skills resolve people for routing decisions and requestor context |
| **Calendar** | Review meetings, deadline tracking | Skills create calendar events for review milestones and SLA-driven deadlines |

**The design pattern:** The Cowork Legal plugin uses SharePoint as the document backbone — matter folders hold all case artifacts (contracts, packets, deviation reports, summaries). An Excel workbook serves as the case tracker for status and metrics. Word is the dominant output artifact because legal work products are inherently document-centric. Every skill reads from and writes to this shared document ecosystem through M365 tools.

### 3.4 Federated Connectors for Third-Party Systems

The framework references CLM platforms, ticketing systems, and playbook repositories. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For CLM platforms (Ironclad, Icertis, Agiloft, DocuSign CLM), Graph Connectors index contract metadata and matter records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["clm-connector"])`. This provides read access to contract status, counterparty records, and prior agreement history without custom integration code.

For e-discovery platforms (Relativity, Exterro), Graph Connectors can index matter metadata for cross-referencing during triage. Skills do not interact with e-discovery data directly but can surface whether a counterparty is involved in active litigation.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- Clause playbooks and approved fallback language maintained as Word documents in a dedicated SharePoint library
- Approval matrices maintained as Excel workbooks or SharePoint lists
- Counterparty history maintained in an Excel workbook populated by Power Automate flows from the CLM
- Matter status synced from the CLM to a SharePoint list via Power Automate

Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For data that has no integration path (e.g., verbal instructions from counsel, negotiation context from phone calls), skills provide structured intake that captures the information and writes it to the case tracker. The framework's Signal Inventory identifies exactly which data points are needed so the skill prompts for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge for playbooks, approval matrices, and counterparty history) and Tier 3 (manual input for request context). Graph Connectors for CLM integration require tenant admin setup and should be introduced in Wave 2 after skill workflows are proven.

### 3.5 Governance in Cowork

Legal workflows impose the most stringent governance requirements of any LOB due to attorney-client privilege, regulatory obligations, and confidentiality duties.

| Governance Domain | Cowork Implementation |
|---|---|
| **Privilege protection** | Skills never produce attorney work product or legal advice; deviation detection presents factual comparisons against playbook standards only. All counsel-facing drafts include privilege headers. Skills never expose privileged analysis in non-privileged channels. |
| **Confidentiality** | Matter folders in SharePoint are access-scoped to the legal team. Skills never include contract terms or deviation details in Teams messages or emails to non-legal recipients. Case references use IDs, not substance. |
| **Access control** | M365 tenant RBAC governs what data the skill can reach. SharePoint permissions on matter folders enforce need-to-know. Skills cannot bypass SharePoint folder-level permissions. |
| **Data classification** | Skill guardrails enforce handling rules: no contract text in email summaries, no deviation details in Teams, privileged markings on counsel communications. |
| **Audit trail** | The Excel case tracker logs all routing decisions with timestamp and actor. SharePoint document versioning preserves all changes to case artifacts. Platform-level logs capture tool invocations. |
| **Change control** | Playbooks, approval matrices, and routing rules are maintained in SharePoint documents that skills read at runtime. Legal operations can update policies without modifying skill definitions. Skill instruction changes follow legal ops change control. |
| **Regulatory compliance** | Skills flag regulatory-sensitive clauses (data protection, export control, anti-corruption) with elevated visibility. Compliance referral drafts are generated when regulatory deviations are detected. |

**The main governance gap** is formal privilege tagging. Cowork does not have a built-in mechanism to classify outputs as privileged vs. non-privileged. The mitigation is to enforce this through skill guardrails (privilege headers, restricted distribution channels) and SharePoint permissions (privileged matter folders with restricted access).

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Legal:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Deviation detection recall — does `legal-deviation-detection` find the deviations that counsel finds on manual review? Assessed by comparing skill output to counsel review on 10+ contracts
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Deviation detection recall — percentage of actual clause deviations identified by the skill vs. manual counsel review
- Incorrect routing rate — percentage of contracts routed to the wrong reviewer or missing a required approver
- Summary acceptance rate — percentage of `legal-triage-comms` drafts sent without major substantive edits
- Triage cycle time — time from case creation to "queued for review" status
- Missing approval incidents — cases where a required approval step was bypassed
- Privileged-access incidents — cases where privileged material was exposed outside the legal team
- Override rate — how often legal operations specialists override skill recommendations

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel contract case tracker in SharePoint with standard columns (Case ID, Counterparty, Contract Type, Business Unit, Requestor, Urgency, Status, Received Date, Assigned Counsel, Deviation Count, Disposition)
- Upload clause playbooks, approved fallback language documents, and standard contract templates to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-matter case artifacts
- Upload the approval matrix as a SharePoint-hosted Excel workbook

**Skills to build:**
- `legal-contract-intake`
- `legal-review-packet`
- `legal-deviation-detection`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5-10 real contract cases.

### Wave 2 — Routing and Communication

**Skills to build:**
- `legal-triage-comms`
- `legal-review-routing`

**Promotions:**
- Promote `legal-deviation-detection` to write-back mode (saves deviation report to matter folder after user confirmation)
- Promote `legal-contract-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for contract cases with approaching SLA deadlines and surfaces any with incomplete triage status

**Integrations:**
- Introduce Graph Connectors for CLM data if available at the tenant level
- Add policy-cited deviation summaries that reference specific playbook section numbers

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action.

### Wave 3 — Optimization and Expansion

**Enhancements:**
- Add bounded multi-document analysis (compare deviations across related agreements for the same counterparty)
- Add proactive detection of high-risk deviation patterns across the contract portfolio via scheduled prompt
- Refine deviation detection thresholds based on counsel override patterns from Waves 1-2
- Add NDA-specific review workflow as a second skill suite using the same framework

**Measurement:**
- Triage cycle time reduction vs. pre-pilot baseline
- Summary acceptance rate target: above 70%
- Deviation detection recall target: above 80%
- Incorrect routing rate target: below 5%
- Privileged-access incident target: zero

---

## Part 5: Generalizing the Approach — Legal's Primary Artifact Pattern

Legal's natural primary artifact is **Word** — contracts, review packets, deviation reports, triage summaries, and playbook extracts are all document-centric. The Excel case tracker provides structured state tracking, and SharePoint serves as the document backbone for matter folders, playbooks, and policies.

This artifact pattern fits the cross-LOB method as follows:

1. **Decompose** — legal steps separate cleanly because each has a distinct goal (intake vs. analysis vs. routing vs. communication)
2. **Map signals to M365 tools** — replace "CLM" with SharePoint + Graph Connectors; replace "playbook repository" with SharePoint document library
3. **Identify the process state artifact** — the Excel case tracker is the canonical tracker; SharePoint matter folders are the document backbone
4. **Define first-class outputs** — Word documents dominate; Adaptive Cards provide quick-review surfaces; Outlook drafts handle all external communication
5. **Encode governance in guardrails** — privilege protection, confidentiality, and access control are the dominant governance concerns; every skill guardrail section must address these explicitly

The framework's decomposition is universal. The Word-centric artifact mapping and privilege-first guardrail posture are what make it concrete for Legal in Copilot Cowork.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references (CLM, playbook repository) |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint playbooks |
| Process State | SharePoint-hosted Excel workbook + matter folders | Durable state externalized to M365 artifacts; matter folders hold all case documents |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, document upload, calendar event |
| Approval | CreateDraftMessage + Adaptive Card + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; privilege-scoped |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time, deviation recall, and routing accuracy |
