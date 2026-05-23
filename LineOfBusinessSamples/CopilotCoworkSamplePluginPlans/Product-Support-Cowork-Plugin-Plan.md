# Plan: Product Support Escalated Case Triage and Engineering Handoff — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Product Support line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Escalated Case Triage and Engineering Handoff pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Product Support, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Escalated case triage scores high on volume and data readiness. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope principle |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call | Requires translation — Product Support references "logging platform", "issue tracker", and "support platform" that must become specific M365 tool names or Graph Connector sources |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md | Strong — Product Support boundaries are well-defined because engineering severity decisions and customer commitments have clear approval gates |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — evidence assembly and technical summarization are strong fits for Data Aggregation and Content Generation templates |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — Cowork provides layers 1-5 and 7-9 natively; skill author controls capability definition and decision logic |

### Key Insight

Product Support escalation triage is **evidence-heavy and handoff-centric**. The framework's decomposition produces skills that naturally split along the intake-evidence-classify-route-handoff axis. The primary Cowork design challenge is **multi-source evidence assembly**: escalation engineers need to pull together case data, log bundles, telemetry, prior cases, and known issues from multiple systems into a coherent packet. The secondary challenge is **cross-team handoff quality** — the skill suite must produce engineering-ready artifacts that are complete enough that engineering does not need to ask support for clarification.

---

## Part 2: The Product Support Plugin — Skill-by-Skill Design

The Product Support sample decomposes "Escalated Case Triage and Engineering Handoff" into 7 steps (PS-001 through PS-007), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| PS-001: Normalize escalation event | `ps-escalation-intake` | Data Aggregation | Deterministic automation | Excel (escalation tracker), SharePoint (list), Outlook (escalation email) |
| PS-002: Gather case, telemetry, and environment context | `ps-evidence-packet` | Data Aggregation | AI act within policy | Word (evidence packet), SharePoint (logs, attachments), Graph API (people) |
| PS-003: Classify issue and likely defect path | `ps-defect-classifier` | Decision Support | AI assist | Adaptive Card (classification), Excel (tracker update) |
| PS-004: Assess customer impact and urgency | `ps-impact-assessment` | Decision Support | AI draft + approve | Adaptive Card (impact report), Excel (severity update) |
| PS-005: Route to engineering owner or queue | `ps-engineering-routing` | Decision Support | AI act within policy | Teams (routing messages), Excel (owner assignment) |
| PS-006: Draft handoff and customer-facing update | `ps-handoff-drafter` | Content Generation | AI draft + approve | Word (engineering handoff), Outlook (customer update draft), Teams (escalation notes) |
| PS-007: Confirm handoff disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `ps-escalation-intake` — Normalize Escalation Event

**Framework Step:** PS-001

**Trigger phrases:** "new escalation", "escalated case from [agent]", "product issue for [customer]", "log escalation for case [ID]", "suspected defect from support"

**Inputs:**
- Originating support case ID or escalation email
- Customer name and account identifier
- Escalating agent name
- Initial issue description and suspected product area
- Attached logs, screenshots, or reproduction notes

**M365 tools:**
- `SearchM365(sources=["email"])` — find the escalation email thread from support
- `SearchPeople` — resolve escalating agent and account owner identities
- `GetUserDetails` — pull profile data for escalating agent and customer contact
- `ReadFileContent` — extract content from attached log files or screenshots in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — pull originating support case record if Graph Connector is configured

**Output:** Structured escalation record written to Excel escalation tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Escalation ID, Originating Case ID, Customer Name, Account ID, Product Area, Issue Summary, Suspected Defect Path, Severity (pending), Status, Created Date, SLA Deadline (4 hours), Assigned To, Engineering Queue, Resolution

**Guardrails:**
- Never create duplicate escalations for the same originating case ID
- Validate that all required fields are populated before writing to tracker
- Confirm details with user before writing to tracker
- Automatically calculate SLA deadline (4 business hours from creation for standard escalations)
- Flag if originating case has an open incident link (may be a duplicate)

---

#### 2. `ps-evidence-packet` — Gather Case, Telemetry, and Environment Context

**Framework Step:** PS-002

**Trigger phrases:** "build evidence packet for escalation [ID]", "pull logs and context for", "assemble escalation context", "what evidence do we have for this defect"

**Inputs:**
- Escalation ID or originating case ID from the tracker

**M365 tools:**
- `ReadFileContent` — read escalation tracker row from SharePoint Excel workbook
- `SearchM365(sources=["email"])` — prior correspondence about this case (support to customer, support to engineering)
- `SearchM365(sources=["files"])` — log bundles, crash reports, HAR files, screenshots uploaded to SharePoint
- `GetDriveChildren` — list all files in the escalation's evidence folder
- `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — originating case details, prior cases for same customer or product area
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — known issues and existing defects for the suspected product area
- `ListChatMessages` — prior Teams discussions between support and engineering about this issue
- `GetUserDetails` — escalation engineer, customer success partner, and engineering owner profiles

**Output:** Word document containing:
- Escalation summary (case ID, customer, product area, issue description)
- Customer impact assessment (account tier, affected users, business impact)
- Technical environment details (product version, configuration, platform)
- Evidence inventory (list of all attached logs, screenshots, and reproduction artifacts with SharePoint links)
- Prior case and known-issue cross-reference (similar escalations, related defects)
- Reproduction steps (if available from support notes)
- Chronological timeline of case progression
- Key contacts (escalation engineer, customer success partner, engineering triage lead)

**Artifact:** Word (.docx) saved to SharePoint escalation folder; the evidence folder link is included

**Guardrails:**
- Never modify or delete original evidence files — the packet is a read-only synthesis
- Cite source and retrieval date for every data point from support platform or issue tracker
- Flag if critical evidence is missing: no logs, no reproduction steps, no environment details
- Flag if the customer has active escalations for other product areas (may indicate broader issue)
- Operate within read-only boundaries — never update support case or issue tracker records

---

#### 3. `ps-defect-classifier` — Classify Issue and Likely Defect Path

**Framework Step:** PS-003

**Trigger phrases:** "classify this escalation", "what product area is this", "categorize defect for escalation [ID]", "triage this product issue"

**Inputs:**
- Escalation data from Excel tracker
- Evidence packet (Word document)
- Issue taxonomy and product area ownership map from SharePoint

**M365 tools:**
- `ReadFileContent` — escalation tracker and evidence packet from SharePoint
- `SearchM365(sources=["files"])` — product area taxonomy, defect classification guide, ownership map from SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — prior defects in same product area for pattern matching

**Output:** Classification report as Adaptive Card containing:
- Recommended product area and component
- Issue type classification (defect, configuration, documentation, feature gap)
- Suspected defect path (the likely area of investigation for engineering)
- Confidence indicator (high, medium, low)
- Similar known issues and prior defects
- Suggested engineering queue based on product area ownership

**Logic:** Compare escalation evidence against the product area taxonomy and known-issue database. Cross-reference with prior escalation patterns. Present classification with supporting evidence and confidence score.

**Guardrails:**
- Present classification as recommendation only — never auto-assign product area without escalation engineer review
- Always show confidence level; flag low-confidence classifications prominently
- Surface similar known issues to help detect duplicates before engineering receives the case
- Never classify severity in this step — that is a separate skill with separate approval

---

#### 4. `ps-impact-assessment` — Assess Customer Impact and Urgency

**Framework Step:** PS-004

**Trigger phrases:** "assess impact for escalation [ID]", "what severity is this", "check urgency", "is this a critical escalation", "customer impact for"

**Inputs:**
- Classification output (product area, issue type, defect path)
- Evidence packet (customer tier, affected users, business impact)
- Severity rubric from SharePoint

**M365 tools:**
- `ReadFileContent` — severity rubric and escalation criteria from SharePoint
- `ReadFileContent` — escalation tracker and evidence packet
- `SearchM365(sources=["files"])` — incident criteria and response path documents
- `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — customer entitlement tier, active incidents
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — affected customer count if telemetry connector is available

**Output:** Impact assessment as Adaptive Card containing:
- Recommended severity level (Sev 1 Critical, Sev 2 High, Sev 3 Standard, Sev 4 Low)
- Customer impact summary (affected users, business impact, revenue risk)
- Escalation urgency (immediate, same-day, standard SLA)
- Incident link recommendation (should this be linked to or create an incident?)
- Recommended response path (engineering triage, incident bridge, standard queue)
- Evidence completeness score (is the packet ready for engineering?)

**Guardrails:**
- Present severity as draft recommendation — require explicit user confirmation before applying
- Never downgrade a severity that was set by a previous human decision without documenting rationale
- Flag any escalation that meets incident criteria even if the overall severity appears low
- Cross-reference with active incidents to detect escalations linked to known outages
- Require escalation engineer acknowledgment for Sev 1 and Sev 2 before routing proceeds
- Include the 4-hour SLA countdown in the assessment output

---

#### 5. `ps-engineering-routing` — Route to Engineering Owner or Queue

**Framework Step:** PS-005

**Trigger phrases:** "route escalation [ID] to engineering", "assign engineering owner", "send to [product area] queue", "who handles this defect area"

**Inputs:**
- Classification and impact assessment output
- Engineering ownership map and routing rules from SharePoint
- Engineering team availability

**M365 tools:**
- `ReadFileContent` — engineering ownership map and routing matrix from SharePoint
- `SearchPeople` — resolve engineering triage leads by product area
- `GetManagerDetails` / `GetDirectReportsDetails` — engineering team structure for escalation paths
- `PostMessage` — Teams notification to assigned engineering triage lead
- `PostChannelMessage` — post escalation summary to the engineering triage channel
- `ListCalendarView` — check engineering lead availability before routing

**Output:** Routed escalation:
- Teams message to assigned engineering triage lead with escalation summary, classification, severity, evidence packet link, and SLA deadline
- Channel post to engineering triage channel with escalation card
- Updated Excel tracker with engineering owner assignment and routed timestamp

**Guardrails:**
- Route only to engineering owners listed in the approved ownership map
- Verify engineering lead availability via calendar before assignment; flag conflicts or OOO
- Require product support manager approval before routing Sev 1 or Sev 2 escalations
- Include evidence packet link and SLA deadline in every routing notification
- Log routing decision with rationale in the escalation tracker
- Never route to engineering if the evidence completeness score from impact assessment is below threshold — flag for additional evidence collection first

---

#### 6. `ps-handoff-drafter` — Draft Handoff and Customer-Facing Update

**Framework Step:** PS-006

**Trigger phrases:** "draft engineering handoff for escalation [ID]", "write handoff summary", "prepare customer update for", "escalation handoff to engineering"

**Inputs:**
- Escalation data from Excel tracker
- Evidence packet (Word document)
- Classification and impact assessment outputs
- Target audience (engineering triage lead, customer success partner, customer)

**M365 tools:**
- `ReadFileContent` — escalation tracker, evidence packet, and all case artifacts from SharePoint
- `CreateDraftMessage` — Outlook draft for customer update (never auto-send)
- `PostMessage` — Teams handoff message to engineering triage lead
- `SearchM365(sources=["files"])` — handoff templates and customer update templates from SharePoint
- `SearchM365(sources=["email"])` — recent correspondence thread for context

**Communication templates:**
- Engineering handoff summary (Word document with structured sections: problem statement, evidence inventory, reproduction steps, customer impact, recommended investigation path)
- Customer update email (acknowledgment that the issue is being investigated, expected next update timeline, any workaround available)
- Internal escalation notification to customer success partner

**Guardrails:**
- Always create customer-facing communications as Outlook draft — never send without explicit user confirmation
- Never include internal severity classifications, engineering queue names, or triage notes in customer-facing drafts
- Never make timeline commitments in customer update drafts — use language like "we are actively investigating" rather than specific dates
- Engineering handoff must include: problem statement, evidence links, reproduction steps (or note if missing), customer impact, and suggested investigation path
- Cite the evidence packet and all attached artifacts — engineering should not need to ask support for clarification
- Match tone to audience: technical and precise for engineering, empathetic and professional for customer

---

#### Step 7: Confirm Handoff Disposition (Human Only)

**Framework Step:** PS-007

This is not a Cowork skill. The framework correctly identifies final handoff confirmation as a human-only step, particularly for severe cases requiring product support manager sign-off. In Cowork, it is supported by:

- The impact assessment and evidence packet — provide the evidence package for the reviewer
- The `ps-handoff-drafter` skill — send the disposition confirmation after the human decision is made
- Calendar tools — book triage review meetings for Sev 1 and Sev 2 escalations
- The escalation tracker Excel workbook — records the final disposition with timestamp and approver

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. Product Support adds a specific complexity: it is **evidence-heavy and cross-team**, requiring assembly of technical artifacts from multiple systems into a coherent handoff packet that bridges support and engineering organizations.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The framework's rule — "keep breaking down until each step has one dominant goal" — prevents a monolithic "handle escalation" skill and produces the focused intake-evidence-classify-assess-route-handoff chain. For Product Support, this is critical because evidence assembly (PS-002) and classification (PS-003) are fundamentally different capabilities that should not be combined.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with meeting artifacts and summary outputs |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather evidence," the framework requires naming every input source — logs, telemetry, prior cases, known issues, ownership maps — each translating to specific MCP tool calls.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — skill trigger phrases and instructions define which process step is active |
| Capability registry | Built-in — skills directory IS the registry |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation gates provide human-in-the-loop |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation via quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. Escalation state (pending evidence, pending classification, pending routing, handed off) must be externalized to the Excel escalation tracker in SharePoint.

### 3.3 M365 Artifacts as Process State

For Product Support, the artifact pattern centers on **Word documents as the primary handoff artifact** with **Excel as the escalation state store** and **SharePoint as the evidence repository**:

| Artifact | Role in Product Support | How Skills Use It |
|---|---|---|
| **Excel** | Escalation state store (tracker with status, severity, owner, SLA, resolution) | The escalation tracker IS the process state — skills read current status, write updates, track SLA and completeness |
| **Word** | Primary handoff artifact (evidence packets, engineering handoff summaries) | Skills generate structured documents that become the official engineering handoff — the most critical output of the entire skill suite |
| **SharePoint** | Evidence repository (logs, screenshots, HAR files, reproduction notes) plus source of truth for policies and templates | Skills read evidence files, taxonomy documents, and severity rubrics; evidence folders store all case artifacts |
| **Outlook** | Customer communication channel | Skills draft customer update emails as reviewable drafts; read inbound escalation emails for context |
| **Teams** | Cross-team coordination (support-to-engineering routing, triage channel posts) | Skills post routing notifications, escalation summaries, and handoff alerts to engineering channels |
| **Adaptive Card** | Real-time decision support (classification, impact assessment) | Skills present triage recommendations for escalation engineer review before any write action |
| **Graph API** | People and org data (engineering ownership, team structure) | Skills resolve engineering owners, check availability, and determine escalation paths |
| **Calendar** | Triage review scheduling and availability checking | Skills check engineering lead availability before routing; schedule triage reviews for severe cases |

**The design pattern:** The Cowork Product Support plugin uses a SharePoint-hosted Excel workbook as the canonical escalation tracker, with Word documents as the primary engineering handoff artifact, SharePoint folders as the evidence repository, and Teams as the cross-team coordination channel. The evidence packet Word document is the most important single output — it must be comprehensive enough that engineering can begin investigation without asking support for clarification.

### 3.4 Federated Connectors for Third-Party Systems

Product Support depends on external systems: support platforms (Zendesk, ServiceNow), issue trackers (Jira, Azure DevOps), logging platforms (Splunk, Datadog, Application Insights), and telemetry systems. The federated access approach follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For support platforms (Zendesk, ServiceNow) and issue trackers (Jira, Azure DevOps), Graph Connectors index case records, known issues, and defect history into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` and `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])`. This provides read access to case data, known issues, and prior defects without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, synchronized data is maintained in SharePoint:
- Known issues list (SharePoint list, synced from issue tracker via Power Automate)
- Engineering ownership map (Excel in SharePoint, maintained by product support operations)
- Product area taxonomy and defect classification guide (SharePoint document, maintained by product support)
- Log bundles and evidence files (uploaded to SharePoint escalation folders by support agents or via Power Automate)

**Tier 3 — Manual Input with Templates**

For logging platform data, crash reports, and telemetry that cannot be automatically synced, skills provide structured intake that captures data from manual lookups. The escalation engineer pastes log excerpts, adds screenshots, and uploads HAR files to the SharePoint evidence folder. The `ps-evidence-packet` skill then inventories and synthesizes what is available.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for issue tracker data and ownership maps, and Tier 3 (manual evidence upload) for logs and telemetry. Graph Connectors for the support platform and issue tracker are best introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

Product Support governance is shaped by three specific concerns: **evidence integrity** (logs, reproduction artifacts, and case data must not be altered), **severity decision control** (severity declarations affect engineering prioritization and customer expectations), and **cross-team handoff quality** (incomplete handoffs waste engineering time and delay resolution).

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure, engineering ownership map, and escalation paths |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC; support platform and issue tracker data accessed through permissioned connectors |
| **Evidence integrity** | Skills never modify original evidence files — the evidence packet is a read-only synthesis with source citations |
| **Severity control** | Severity recommendations require explicit escalation engineer confirmation; Sev 1 and Sev 2 require product support manager approval before routing |
| **Handoff quality** | Engineering handoff documents must include: problem statement, evidence inventory, reproduction steps, customer impact, and suggested investigation path — skills flag if any section is empty |
| **Customer communication** | All customer-facing drafts require human review; no timeline commitments allowed in drafted responses |
| **Audit** | Platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; escalation tracker records every state change |
| **Release management** | Skills versioned in OneDrive; quality rubric scoring provides pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions; dynamic policy content (severity rubric, ownership map, taxonomy) read from SharePoint at runtime |

**The main governance gap** is cross-system state synchronization. When the escalation tracker in SharePoint shows "Routed to Engineering," the actual issue tracker (Jira, Azure DevOps) should also reflect this. In Wave 1, this synchronization is manual. In Wave 2, Power Automate flows can bridge the gap.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do generated evidence packets, classifications, and handoff documents meet engineering expectations? Assessed via engineering triage lead review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Handoff completeness rate — percentage of engineering handoff packets that contain all required sections without gaps
- Correct routing rate — percentage of escalations routed to the correct engineering queue
- Time to engineering-ready packet — measured from escalation creation to packet delivered to engineering
- Severity recommendation agreement rate — percentage of severity recommendations accepted by escalation engineer
- Duplicate-defect detection rate — percentage of escalations flagged as potential duplicates before reaching engineering
- Draft acceptance rate — percentage of customer update drafts sent without major edits
- Override rate — how often escalation engineers override skill recommendations
- Evidence completeness score — average completeness score across all evidence packets

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel escalation tracker workbook in SharePoint with standard columns (Escalation ID, Originating Case ID, Customer Name, Account ID, Product Area, Issue Summary, Suspected Defect Path, Severity, Status, Created Date, SLA Deadline, Assigned To, Engineering Queue, Evidence Completeness, Resolution, Last Updated, Updated By)
- Upload severity rubric, product area taxonomy, engineering ownership map, and handoff templates to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-escalation evidence (logs, screenshots, HAR files, reproduction notes)
- Configure SharePoint bridge data: known issues list (synced from issue tracker), engineering ownership map

**Skills to build:**
- `ps-escalation-intake`
- `ps-evidence-packet`
- `ps-defect-classifier`
- `ps-handoff-drafter`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 10-15 real escalations across multiple product areas and severity levels.

### Wave 2 — Routing and Impact Assessment

**Skills to build:**
- `ps-impact-assessment`
- `ps-engineering-routing`

**Promotions:**
- Promote `ps-escalation-intake` to write mode (creates escalation records after confirmation)
- Promote `ps-defect-classifier` to write-back mode (updates Excel tracker classification after user confirmation)
- Promote `ps-engineering-routing` to bounded write mode (posts routing messages to Teams engineering channels within approved ownership map)

**Integrations:**
- Introduce Graph Connectors for support platform and issue tracker if available at the tenant level
- Add known-issue and duplicate detection: `ps-defect-classifier` cross-references against the known issues list and flags potential matches

**Automation:**
- Set up a scheduled prompt (every 30 minutes) that checks the escalation tracker for cases approaching 4-hour SLA breach and surfaces warnings via Teams notifications to product support manager
- Set up a daily scheduled prompt that summarizes escalation volume, average time to engineering-ready, and severity distribution

**Operating posture:** AI draft plus approve for impact assessment and customer-facing communications. AI act within policy for routing to approved engineering queues. Every severity declaration and customer communication reviewed before action.

### Wave 3 — Optimization and Proactive Detection

**Enhancements:**
- Add bounded multi-source escalation packet assembly: a skill that aggregates data from support connector, issue tracker connector, email threads, and SharePoint evidence folders into a single comprehensive engineering handoff
- Add proactive defect pattern detection: scheduled prompt that analyzes recent escalations for recurring product areas, repeated issue types, or increasing severity trends and surfaces findings to the product support manager
- Add evidence gap monitoring: skill that reviews in-progress escalations and flags those with incomplete evidence before the 4-hour SLA deadline
- Refine all skills based on override patterns and engineering feedback from Waves 1-2

**Measurement:**
- Handoff completeness rate target: above 90%
- Correct routing rate target: above 85%
- Time to engineering-ready packet target: under 3 hours for standard escalations
- Severity agreement rate target: above 80%
- Duplicate detection rate target: above 60% of actual duplicates caught before engineering receives them
- Draft acceptance rate target: above 70%

---

## Part 5: Generalizing the Approach — Product Support Artifact Pattern

Product Support's primary artifact pattern is **Word (evidence packets, engineering handoff summaries)** supported by **Excel (escalation state tracking)** and **SharePoint (evidence repository)**.

This LOB demonstrates the **evidence-centric variant** of the framework-to-Cowork translation:

1. **The primary output artifact is the Word evidence packet** — unlike HR (where multiple artifact types share importance) or Customer Service (where communication channels dominate), Product Support's entire workflow converges on producing one high-quality Word document: the engineering handoff. The skill suite's effectiveness is measured primarily by the completeness and clarity of this document.

2. **SharePoint serves dual duty as evidence repository and policy store** — evidence folders contain raw technical artifacts (logs, screenshots, HAR files) that the evidence packet skill inventories and synthesizes. The same SharePoint library hosts the taxonomy, severity rubric, and ownership map that other skills consume.

3. **Cross-team handoff quality is the primary governance concern** — while Customer Service focuses on SLA compliance and unauthorized commitments, Product Support focuses on ensuring engineering receives a complete, well-structured packet. Skills are designed to flag evidence gaps before routing rather than after.

4. **Duplicate detection is a high-value secondary capability** — Product Support benefits significantly from surfacing known issues and prior defects during classification. This prevents engineering from receiving duplicate escalations and accelerates resolution when a workaround already exists.

The decomposition method (one goal per step, boundary-driven guardrails, M365 artifact grounding) works identically for Product Support as it does for HR and Customer Service. The difference is which artifact dominates: HR is tracking-centric (Excel, Word); Customer Service is communication-centric (Outlook, Teams); Product Support is evidence-centric (Word, SharePoint).

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
| Approval | CreateDraftMessage + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via handoff completeness and routing accuracy |
| Support Platform / Issue Tracker | Graph Connectors or SharePoint bridge | External system data accessed via `SearchM365(sources=["connectors"])` or synced to SharePoint |
| Evidence / Log Bundle | SharePoint folder + ReadFileContent | Technical evidence stored in SharePoint escalation folders; skills inventory and synthesize via MCP tools |
| Engineering Handoff | Word document generated by `ps-handoff-drafter` | The primary output artifact of the entire skill suite |
