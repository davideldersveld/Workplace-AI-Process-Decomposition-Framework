# Plan: Business Analysis Requirements Intake — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Business Analysis line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Requirements Intake and Synthesis pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Business Analysis, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — BA references ticketing systems, backlog tools, and requirements repositories that must become M365 tool names or SharePoint bridge reads |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — BA's emphasis on traceability and stakeholder ownership means every synthesized requirement must be reviewable and attributable |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the task patterns (extract, summarize, compare, generate, route) map to Cowork skill templates |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

Business analysis workflows are signal-heavy and synthesis-intensive. Unlike Finance (rules-heavy, tabular data) or HR (case-management, checklist-driven), BA work involves converting fragmented, often contradictory stakeholder inputs into structured requirements with full traceability. The framework's decomposition discipline is especially valuable here because it separates the intake-and-extraction phase from the synthesis-and-drafting phase — preventing the common anti-pattern of a skill that tries to go from raw stakeholder input to finished requirements in one step. The 8-step decomposition ensures each skill has a clear scope: intake, discovery, extraction, synthesis, drafting, traceability, routing, and baseline.

---

## Part 2: The Business Analysis Requirements Plugin — Skill-by-Skill Design

The BA sample decomposes "Requirements Intake and Synthesis" into 8 steps (BA-REQ-001 through BA-REQ-008), identifies 7 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| BA-REQ-001: Normalize request event | `ba-request-intake` | Data Aggregation | Deterministic automation | Excel (request tracker), SharePoint (list), Outlook (notifications) |
| BA-REQ-002: Gather supporting context | `ba-discovery-packet` | Data Aggregation | AI act within policy | Word (discovery packet), SharePoint (documents, prior BRDs), Graph API (people) |
| BA-REQ-003: Extract candidate needs and constraints | `ba-signal-extraction` | Data Aggregation + Decision Support | AI assist | Excel (signal inventory), Adaptive Card (extraction summary) |
| BA-REQ-004: Cluster and reconcile themes | `ba-theme-synthesis` | Decision Support | AI assist | Word (theme report), Adaptive Card (conflict report), Excel (theme tracker) |
| BA-REQ-005: Draft requirements package | `ba-requirements-draft` | Content Generation | AI draft + approve | Word (BRD, user stories, acceptance criteria), Excel (requirements matrix) |
| BA-REQ-006: Prepare review and traceability packet | `ba-traceability-packet` | Data Aggregation + Content Generation | AI draft + approve | Excel (traceability matrix), Word (review summary), PowerPoint (review deck) |
| BA-REQ-007: Route for feedback and sign-off | `ba-review-routing` | Decision Support | AI act within policy | Teams (messages), Outlook (review requests), Graph API (stakeholder registry) |
| BA-REQ-008: Confirm disposition and baseline | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `ba-request-intake` — Normalize Request Event

**Framework Step:** BA-REQ-001

**Trigger phrases:** "new requirements request", "intake request for [project]", "set up analysis case for", "new change request from [stakeholder]", "business request for"

**Inputs:**
- Request description or business problem statement
- Requestor name or email
- Business capability or domain
- Impacted application or process
- Priority and target timeline
- Attached supporting documents

**M365 tools:**
- `SearchM365(sources=["email"])` — find the request email thread for context
- `SearchM365(sources=["files"])` — locate attached documents, prior related requests
- `GetUserDetails` — resolve requestor identity
- `SearchPeople` — identify stakeholder group and delivery contacts
- `ReadFileContent` — read existing request tracker to check for duplicates

**Output:** Structured analysis case record written to Excel request tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Analysis Case ID, Request ID, Description, Requestor, Business Domain, Impacted Application, Priority, Status, Created Date, Discovery Status, Synthesis Status, Review Status, Assigned Analyst, Target Date

**Guardrails:**
- Never create duplicate cases for the same request
- Validate that required fields are populated (description, requestor, domain)
- Confirm details with user before writing to tracker
- Log case creation with timestamp, actor, and source reference

---

#### 2. `ba-discovery-packet` — Gather Supporting Context

**Framework Step:** BA-REQ-002

**Trigger phrases:** "build discovery packet for [request]", "gather context for requirements", "what background do we have for [project]", "assemble discovery artifacts"

**Inputs:**
- Analysis case ID or request description

**M365 tools:**
- `SearchM365(sources=["files"])` — prior BRDs, process maps, policy documents, SOPs, existing requirements
- `ReadFileContent` — read specific documents from SharePoint (prior decisions, architecture docs, domain glossaries)
- `GetDriveChildren` — list documents in project or domain folders
- `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])` — backlog items and prior tickets (if Graph Connector available)
- `GetUserDetails` / `SearchPeople` — resolve stakeholder contacts and SME roster
- `GetManagerDetails` — org structure for stakeholder mapping

**Output:** Word document containing:
- Request summary and business problem statement
- Current-state documentation inventory (what exists vs. what is needed)
- Prior related decisions and requirements
- Impacted system and process context
- Stakeholder roster with roles and contact details
- Policy or compliance references relevant to this request
- Known constraints and assumptions from source material

**Artifact:** Word (.docx) saved to SharePoint analysis case folder; serves as the discovery evidence base

**Guardrails:**
- Cite source document and retrieval date for every referenced artifact
- Flag if critical context is missing (no prior BRD, no process map, no SME identified)
- Never fabricate or infer business context not present in source documents
- Mark any information from informal sources (chat, email) as unverified until confirmed

---

#### 3. `ba-signal-extraction` — Extract Candidate Needs and Constraints

**Framework Step:** BA-REQ-003

**Trigger phrases:** "extract requirements from these documents", "pull needs from workshop notes", "what are the stakeholder requirements", "extract signals from [source]"

**Inputs:**
- Discovery packet (Word document)
- Source artifacts: workshop notes, stakeholder emails, meeting transcripts, intake forms, existing process maps

**M365 tools:**
- `ReadFileContent` — source documents from SharePoint (notes, emails, transcripts, forms)
- `SearchM365(sources=["email"])` — stakeholder email threads with requirements context
- `SearchM365(sources=["teams"])` — Teams discussions about the request
- `GetMeetingTranscript` — meeting transcripts from discovery workshops (if available)

**Output:** Extracted signal inventory as Excel worksheet plus Adaptive Card summary

**Extraction categories:**
- Stated business needs (explicit stakeholder requests)
- Constraints (technical, regulatory, timeline, budget)
- Assumptions (stated or implied)
- Decisions already made
- Open questions requiring clarification
- Dependencies on other systems or teams
- Conflicting statements (flagged for reconciliation)

**Logic:** Read each source artifact and extract structured signals into the categories above. Attribute each signal to its source document, author, and date. Flag signals that appear contradictory or that lack a clear owner.

**Guardrails:**
- Present extraction results for analyst review before any further processing — this is AI assist mode
- Attribute every extracted signal to a specific source document, author, and date
- Never infer stakeholder intent beyond what is explicitly stated in source material
- Flag ambiguous or contradictory signals for manual resolution rather than resolving them automatically
- Clearly distinguish "stated by stakeholder" from "inferred from context"

---

#### 4. `ba-theme-synthesis` — Cluster and Reconcile Themes

**Framework Step:** BA-REQ-004

**Trigger phrases:** "synthesize requirements themes", "cluster these needs", "reconcile stakeholder input", "what are the main themes", "find conflicts in requirements"

**Inputs:**
- Extracted signal set (Excel or structured data)
- Requirements taxonomy from SharePoint
- Discovery packet for additional context

**M365 tools:**
- `ReadFileContent` — requirements taxonomy, domain glossary, prior requirements packages from SharePoint
- `ReadFileContent` — extracted signal inventory
- `SearchM365(sources=["files"])` — prior similar requirements packages for pattern reference

**Output:** Theme synthesis report as Word document plus Adaptive Card conflict summary

**Synthesis outputs:**
- Requirement themes (grouped related needs with descriptive labels)
- Conflict report (contradictory requirements with source attribution)
- Open questions (items requiring stakeholder clarification before drafting)
- Gap identification (expected requirement areas with no stakeholder input)
- Priority signals (relative importance indicators from source material)

**Logic:** Group related extracted signals into coherent themes using the requirements taxonomy. Identify conflicts where different stakeholders have stated contradictory needs. Surface gaps where expected requirement areas have no coverage. Produce a structured theme report that becomes the input to requirements drafting.

**Guardrails:**
- Present synthesis results for analyst review before proceeding — this is AI assist mode
- Preserve source attribution through synthesis — every theme must trace to one or more extracted signals
- Never resolve conflicts autonomously — present both sides with source attribution for analyst decision
- Flag themes with single-source support as potentially incomplete
- Clearly label gap identification as "analyst should verify" rather than asserting missing requirements

---

#### 5. `ba-requirements-draft` — Draft Requirements Package

**Framework Step:** BA-REQ-005

**Trigger phrases:** "draft requirements for [request]", "generate BRD", "write user stories", "draft acceptance criteria", "prepare requirements package"

**Inputs:**
- Synthesized themes (Word document or structured data)
- Requirements templates from SharePoint
- Domain glossary and standards
- Prior approved requirements packages (exemplars)

**M365 tools:**
- `ReadFileContent` — requirements templates, BRD template, user story template, acceptance criteria format from SharePoint
- `ReadFileContent` — synthesized themes and extracted signals
- `SearchM365(sources=["files"])` — prior approved requirements packages for style and structure reference

**Output options:**
- **Word document** — Full business requirements document (BRD) with sections: Executive Summary, Business Objectives, Scope, Functional Requirements, Non-Functional Requirements, Business Rules, Assumptions, Constraints, Dependencies, Open Questions
- **Excel worksheet** — Requirements matrix with columns: Req ID, Theme, Requirement Statement, Type (functional/non-functional/business rule), Priority, Source Signal, Status, Owner
- **User stories** — Formatted as "As a [role], I want [capability], so that [benefit]" with acceptance criteria per story

**Guardrails:**
- Always create as draft for analyst review — never finalize without explicit confirmation — this is AI draft plus approve
- Every requirement must trace to at least one extracted signal or explicit analyst judgment
- Use the approved template structure from SharePoint — do not invent new sections
- Flag requirements generated from single-source or low-confidence signals
- Never add scope not present in the synthesized themes without explicit analyst direction
- Include an "Open Questions" section for items that need stakeholder clarification before finalization
- Match terminology to the domain glossary — do not introduce new terms without flagging them

---

#### 6. `ba-traceability-packet` — Prepare Review and Traceability Packet

**Framework Step:** BA-REQ-006

**Trigger phrases:** "prepare review packet", "build traceability matrix", "link requirements to sources", "review readiness check", "prepare for sign-off"

**Inputs:**
- Draft requirements package (Word or Excel)
- Extracted signal set
- Stakeholder roster
- Review checklist from SharePoint

**M365 tools:**
- `ReadFileContent` — draft requirements package, signal inventory, review checklist
- `ReadFileContent` — traceability standard and review template from SharePoint
- `GetDriveChildren` — verify all supporting documents exist in the case folder
- `SearchPeople` — resolve reviewer contacts from stakeholder roster

**Output:**
- **Excel workbook** — Traceability matrix mapping each requirement to its source signals (columns: Req ID, Requirement Statement, Source Signal ID, Source Document, Source Author, Source Date, Traceability Status)
- **Word document** — Review summary with: package completeness assessment, unresolved questions list, traceability coverage percentage, reviewer assignments, recommended review timeline
- **PowerPoint deck** — Executive review presentation with: request summary, key themes, requirement highlights, open items, and approval request (for stakeholder review meetings)

**Guardrails:**
- Present traceability packet for analyst review before routing — this is AI draft plus approve
- Flag any requirement without source support as "analyst judgment — requires explicit confirmation"
- Calculate and display traceability coverage percentage (requirements with source links / total requirements)
- Include all unresolved questions and conflicts in the review summary — do not suppress them
- Verify that every listed reviewer is in the approved stakeholder roster
- Never mark the package as "review-ready" if traceability coverage is below the minimum threshold defined in the review checklist

---

#### 7. `ba-review-routing` — Route for Feedback and Sign-Off

**Framework Step:** BA-REQ-007

**Trigger phrases:** "send for review", "route requirements to reviewers", "request sign-off", "send package to [stakeholder]", "distribute review packet"

**Inputs:**
- Review and traceability packet
- Stakeholder roster with reviewer assignments
- Approval matrix and review rules from SharePoint

**M365 tools:**
- `SearchPeople` — resolve reviewer identities
- `GetUserDetails` — confirm reviewer contact details
- `PostMessage` — Teams notification to reviewers with review summary and document links
- `CreateDraftMessage` — Outlook draft for formal review requests (sent after analyst confirmation)
- `CreateEvent` — optional review deadline calendar holds

**Output:** Routed review tasks:
- Teams messages to assigned reviewers with review summary, document links, and expected response date
- Outlook drafts for formal review requests with attached review packet
- Calendar events for review deadlines or review meetings
- Updated Excel tracker with review status (Sent for Review, reviewer names, sent date)

**Guardrails:**
- This skill operates as AI act within policy — routing is executed when it follows the approved reviewer registry and workflow rules
- Verify all reviewers are in the approved stakeholder roster before sending
- Include analysis case ID and request reference in every communication
- Set reasonable review deadlines per the review standard (not less than 2 business days)
- Never route for sign-off if the traceability packet flags unresolved blocking items
- Create Outlook communications as drafts for formal requests; Teams messages may be sent directly for routine status updates

---

#### Step 8: Confirm Disposition and Baseline (Human Only)

**Framework Step:** BA-REQ-008

This is not a Cowork skill. The framework correctly identifies final disposition and baseline approval as a human-only step. In Cowork, it is supported by:

- The `ba-traceability-packet` skill — provides the complete evidence package for reviewer decision
- The `ba-review-routing` skill — routes the package to the correct reviewers and tracks responses
- The `ba-requirements-draft` skill — produces the artifact that gets baselined
- Calendar support — book the review meeting for requirements walkthrough

The product owner or business sponsor makes the final determination: approve and baseline, request revisions, or return for additional discovery. This decision is logged in the Excel tracker after the human decision is made.

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Business Analysis is **signal-to-structure discipline**:

**Process decomposition separates extraction from synthesis from drafting.** The most common BA anti-pattern is a skill that tries to read raw stakeholder input and produce finished requirements in one step. The framework's decomposition forces a pipeline: intake, discovery, extraction, synthesis, drafting, traceability, routing. Each stage produces a discrete, reviewable artifact.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with meeting-intel or daily-briefing |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, route to reviewers) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** BA work is uniquely signal-heavy — stakeholder emails, workshop notes, meeting transcripts, chat threads, document comments, and form submissions all contain requirements signals. The framework requires naming every source, which translates to specific MCP tool calls: `SearchM365(sources=["email"])` for stakeholder threads, `GetMeetingTranscript` for workshop recordings, `SearchM365(sources=["teams"])` for chat discussions, and `ReadFileContent` for uploaded documents.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, files, and meeting transcripts are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, `GetMeetingTranscript`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation gates provide human-in-the-loop; no formal approval workflow engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation via quality rubric and manual testing |

**The key gap for BA:** Cowork does not have a native requirements repository or backlog integration. The framework's "requirements repository" and "backlog system" must be either accessed via Graph Connectors (Jira, Azure DevOps) or replicated in SharePoint. The Excel tracker and Word documents serve as the Cowork-side requirements state until a formal repository integration is established.

### 3.3 M365 Artifacts as First-Class Process State

For Business Analysis, the artifact pattern is **Word-centric with Excel tracking and PowerPoint for review**. Each artifact type serves a specific role:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Word** | **PRIMARY** — Requirements artifacts (BRDs, discovery packets, theme reports, review summaries, user stories) | Skills generate the core deliverables that become the official requirements record; Word is the natural format for document-centric BA work products |
| **Excel** | **PRIMARY** — Process state and structured data (request tracker, signal inventory, requirements matrix, traceability matrix) | The request tracker workbook IS the process state; the traceability matrix provides the audit linkage between requirements and source signals |
| **PowerPoint** | Decision-support artifacts (review decks, stakeholder presentations) | Skills create executive-facing presentations for requirements review meetings; reviewers consume decks, not raw documents |
| **SharePoint** | Source of truth (templates, taxonomies, prior BRDs, process maps, policies, glossaries) | Skills read templates and reference materials from SharePoint; all case artifacts stored here |
| **Outlook** | Communication channel for formal review routing | Skills draft review requests and follow-up communications; formal sign-off requests go through Outlook |
| **Teams** | Coordination channel and signal source | Skills read Teams discussions for requirements signals; post review notifications and status updates |
| **Adaptive Card** | Decision-support presentation (extraction summaries, conflict reports, synthesis results) | Skills present analysis for analyst review during intermediate steps |
| **Graph API** | People and org data (stakeholder roster, reviewer contacts, org hierarchy) | Skills resolve reviewers, stakeholders, and SMEs for routing and traceability |
| **Meeting Transcripts** | Signal source (workshop notes, stakeholder interviews) | Skills extract requirements signals from recorded discovery sessions |

**The design pattern:** The Cowork BA plugin uses a SharePoint-hosted Excel workbook as the request tracker and traceability store, with Word documents as the primary deliverable format. This dual-artifact pattern (Excel for structured tracking, Word for narrative deliverables) reflects the reality that BA work produces both structured data (requirements matrices, traceability links) and narrative documents (BRDs, review summaries) as first-class outputs.

### 3.4 Federated Connectors for Third-Party Systems

The framework references ticketing systems (Jira, ServiceNow), backlog tools (Azure DevOps, Rally), requirements repositories (DOORS, Jama), and document management systems. The tiered approach:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For backlog and ticketing platforms (Jira, Azure DevOps, ServiceNow), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])`. This provides read access to existing tickets, backlog items, and prior requirements from external systems.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain reference data in SharePoint: stakeholder registries, requirements taxonomies, review checklists, and template libraries. Prior BRDs and process maps are uploaded to SharePoint document libraries. Skills interact with the SharePoint copy.

**Tier 3 — Manual Input with Templates**

For data from workshops, interviews, or systems with no integration path, skills capture stakeholder input through structured prompts and document upload. The signal extraction skill processes these manually-provided artifacts the same way it processes system-sourced ones.

**Practical recommendation for a BA pilot:** Start with Tier 2 (SharePoint for templates, taxonomies, and prior artifacts) and Tier 3 (manual upload of workshop notes, emails, and documents). BA work is inherently document-heavy, so SharePoint is a natural fit. Introduce Jira/DevOps Graph Connectors in Wave 2 for backlog integration.

### 3.5 Governance in Cowork

Business Analysis governance centers on traceability, attribution, and change control. The framework's model maps to Cowork as follows:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | BA manager or practice lead owns the process; delivery platform lead owns skill contracts; each requirement has a named business owner |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC; early-stage initiative data may be confidential |
| **Traceability** | Every requirement must trace to source signals via the traceability matrix; the `ba-traceability-packet` skill enforces this |
| **Attribution** | Extracted signals attributed to source document, author, and date; synthesized themes linked to supporting signals |
| **Change control** | Baseline approval is human-only; scope additions flagged by skills; version history preserved in SharePoint |
| **Audit** | Excel tracker logs all status transitions; Word documents provide the artifact trail; every skill action traceable to actor and timestamp |
| **Release management** | Skills versioned; quality rubric provides pre-deployment gate; template and taxonomy changes require BA practice approval |
| **Policy enforcement** | Encoded in skill instructions; dynamic template and taxonomy read from SharePoint; review checklist enforced before routing |

**The main governance gap** is that Cowork cannot enforce that generated requirements are actually reviewed before they enter a backlog or delivery system. The mitigation is to make the review routing step mandatory in the workflow (the `ba-review-routing` skill will not route unless traceability coverage meets the minimum threshold) and to treat the Excel tracker's status field as the authoritative record of review state.

### 3.6 Evaluation Approach

**Component-level evaluation (per skill):**
- Extraction accuracy — does `ba-signal-extraction` correctly identify stated needs, constraints, and conflicts from source material?
- Synthesis acceptance rate — percentage of themes from `ba-theme-synthesis` accepted by analysts with minor edits; target above 85%
- Draft package acceptance rate — percentage of `ba-requirements-draft` outputs accepted with minor edits; target above 75%
- Traceability coverage — percentage of requirements with source signal linkage; target 100% for critical requirements
- Routing correctness — does `ba-review-routing` send to the right reviewers? Target: below 5% incorrect routing

**Process-level evaluation (end-to-end):**
- Time to first review-ready draft — days from complete intake to routed review packet
- Number of review cycles — target reduction vs. pre-pilot baseline
- Requirement defect leakage — requirements that cause downstream delivery defects
- Traceability coverage at baseline — percentage of baselined requirements with full source traceability
- Stakeholder satisfaction — clarity, completeness, and responsiveness of requirements packages
- Scope change frequency after baseline — indicator of synthesis quality
- Unattributed scope additions — must be zero in pilot

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Intake and Discovery

**Infrastructure setup:**
- Create the shared Excel request tracker workbook in SharePoint with standard columns (Analysis Case ID, Request ID, Description, Requestor, Business Domain, Impacted Application, Priority, Status, Created Date, Discovery Status, Synthesis Status, Review Status, Assigned Analyst, Target Date, Baseline Date, Disposition)
- Upload requirements templates (BRD template, user story template, acceptance criteria format), domain glossaries, requirements taxonomies, and review checklists to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-case artifacts (discovery documents, source material, drafts, review packets)
- Upload exemplar requirements packages (prior approved BRDs) for style and quality reference

**Skills to build:**
- `ba-request-intake`
- `ba-discovery-packet`
- `ba-signal-extraction`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs presented via Adaptive Card or generated documents for manual review. Test with one portfolio, product domain, or business unit using 5-10 real requests.

### Wave 2 — Synthesis and Drafting

**Skills to build:**
- `ba-theme-synthesis`
- `ba-requirements-draft`
- `ba-traceability-packet`

**Promotions:**
- Promote `ba-signal-extraction` to write-back mode (updates signal inventory Excel after analyst confirmation)
- Promote `ba-request-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a weekly scheduled prompt that checks for analysis cases approaching their target date and surfaces any with incomplete discovery or stalled synthesis

**Operating posture:** AI draft plus approve for all synthesis and drafting skills. Every output reviewed before further processing.

### Wave 3 — Review Routing and Optimization

**Skills to build:**
- `ba-review-routing`

**Enhancements:**
- Introduce Graph Connectors for Jira or Azure DevOps if available at the tenant level
- Add change request impact pre-assessment (compare new request against existing baselined requirements)
- Add proactive detection of likely missing stakeholders or missing requirement areas
- Add bounded multi-source synthesis for larger discovery sets with many source artifacts
- Refine all skills based on analyst feedback and override patterns from Waves 1-2

**Measurement:**
- Time to first review-ready draft reduction vs. pre-pilot baseline
- Synthesis acceptance rate target: above 85%
- Draft package acceptance rate target: above 75%
- Traceability coverage target: 100% for critical requirements
- Incorrect reviewer routing target: below 5%
- Unattributed scope additions: zero
- Baseline approval without human review: zero

---

## Part 5: Generalizing the Approach — Business Analysis Artifact Patterns

Business Analysis's primary artifact pattern is **Word-centric with Excel structured data**. BA work produces both narrative deliverables (BRDs, discovery packets, review summaries) and structured tracking data (request trackers, signal inventories, traceability matrices, requirements matrices). Word handles the narrative; Excel handles the structured data.

This pattern generalizes across BA sub-functions:

| BA Process | Primary Artifacts | Secondary Artifacts |
|---|---|---|
| Requirements intake and synthesis | Word (BRD, discovery packet) + Excel (request tracker, traceability matrix) | PowerPoint (review deck), Adaptive Card (conflict report), Outlook (review routing) |
| Stakeholder interview summarization | Word (interview summary, key findings) | Excel (signal inventory), Teams (follow-up questions) |
| User story and acceptance criteria drafting | Word (user stories with AC) + Excel (story matrix) | Adaptive Card (review summary), Teams (clarification threads) |
| Requirements traceability maintenance | Excel (traceability matrix) | Word (coverage report), Adaptive Card (gap flags) |
| Change request impact assessment | Word (impact memo) + Excel (impact matrix) | PowerPoint (stakeholder presentation), Outlook (review requests) |

The cross-LOB method applies: decompose into bounded steps, assign automation boundaries, map signals to M365 tools, identify the canonical tracker artifact (Excel for tracking, Word for deliverables), define first-class outputs, and encode governance in guardrails. BA's distinguishing requirement is that every generated requirement must be traceable to source signals and attributable to a named stakeholder or explicit analyst judgment — this traceability chain is the non-negotiable guardrail pattern for any BA Cowork plugin.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic ticketing, backlog, and requirements repository references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly — prefer skills and tools; bounded multi-source synthesis introduced in Wave 3 |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic template and taxonomy read from SharePoint; traceability enforced |
| Process State | SharePoint-hosted Excel workbook + Word deliverables | Dual state: tracker for status and structured data, Word documents for narrative deliverables |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint, Meeting Transcripts) + Graph Connectors for backlog tools | Email, Teams chat, meeting transcript, SharePoint upload, form submission |
| Approval | CreateDraftMessage + confirmation gates + stakeholder roster lookup | Human-in-the-loop via Cowork's review-before-action patterns; baseline approval always human-owned |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via extraction accuracy and synthesis acceptance; process eval via time-to-draft, traceability coverage, and defect leakage |
