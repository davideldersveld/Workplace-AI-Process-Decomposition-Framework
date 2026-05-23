# Plan: Insurance FNOL and Coverage Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Insurance line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the FNOL (First Notice of Loss) and Coverage Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

Insurance claims handling is among the most heavily regulated, document-intensive, and litigation-sensitive workflows in any enterprise. Every design decision in this plan reflects a core constraint: AI may prepare, extract, classify, and recommend, but coverage determination, payment authority, reserve setting, and formal denial remain exclusively human-owned. All skill outputs are framed as drafts for adjuster review.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). For Insurance, each phase was evaluated against the realities of claims operations: high document volume, multi-channel loss intake, strict regulatory controls per jurisdiction, evidence-chain requirements, and the ever-present risk of bad-faith litigation exposure.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — FNOL and coverage triage scores highest on volume, data readiness, and measurable cycle time; the scoring criteria map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope skill principle; insurance steps naturally decompose by intake, context, classification, evidence, routing, and communication |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "claims platform", "policy administration system", and "catastrophe feed" which must become specific M365 tool names or federated connector references |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but insurance demands stricter guardrails than most domains; every skill must include explicit prohibitions against coverage determinations, reserve recommendations, and liability assessments |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates; insurance adds a heavy emphasis on Compare (evidence vs. checklist) and Route (claim handling lane) |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Insurance claims triage is a preparation-heavy, judgment-reserved workflow. The AI value concentrates in three areas: (1) assembling scattered evidence into a coherent packet, (2) classifying and flagging before a human decides, and (3) drafting communications that an adjuster reviews. The framework's automation boundary analysis confirms that no step in the FNOL triage process should grant the model authority over coverage, payment, or formal claim disposition. This maps directly to Cowork's guardrail architecture, where every write action requires explicit human confirmation and every output is labeled as a draft for adjuster review.

---

## Part 2: The Insurance FNOL and Coverage Triage Plugin — Skill-by-Skill Design

The Insurance sample decomposes "FNOL and Coverage Triage" into 7 steps (INS-CLM-001 through INS-CLM-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| INS-CLM-001: Normalize FNOL event | `ins-fnol-intake` | Data Aggregation | Deterministic automation | Excel (claim tracker), SharePoint (intake forms), Outlook (loss notifications) |
| INS-CLM-002: Gather policy, claimant, and loss context | `ins-claim-context` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (policy documents, evidence files), Graph API (people) |
| INS-CLM-003: Classify claim type and severity | `ins-claim-classifier` | Decision Support | AI assist | Excel (classification log), Adaptive Card (classification report), SharePoint (claim taxonomy) |
| INS-CLM-004: Assess initial coverage path and missing evidence | `ins-coverage-gap-detection` | Decision Support | AI assist | Excel (evidence checklist), Word (coverage analysis draft), Adaptive Card (gap report) |
| INS-CLM-005: Route to adjuster, catastrophe desk, or SIU | `ins-claim-routing` | Decision Support | AI draft + approve | Teams (routing notifications), Excel (assignment tracker), Calendar (review deadlines) |
| INS-CLM-006: Draft claimant, broker, and adjuster communications | `ins-claim-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (internal messages), Word (adjuster summary) |
| INS-CLM-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (disposition review), Outlook (sign-off confirmation) |

### Detailed Skill Designs

#### 1. `ins-fnol-intake` — Normalize FNOL Event

**Framework Step:** INS-CLM-001

**Trigger phrases:** "new loss reported", "FNOL for [claimant name]", "set up claim for", "loss notification from [broker/agent]", "new claim intake"

**Inputs:**
- Claimant name or policy number
- Date of loss
- Loss type (property, auto, liability, etc.)
- Loss description or narrative
- Reporting channel (email, phone transcript, portal, broker)

**M365 tools:**
- `SearchM365(sources=["email"])` — find the inbound loss notification email or broker submission
- `SearchPeople` — resolve claimant, agent, and broker identities
- `GetUserDetails` — pull profile data for internal contacts
- `ReadFileContent` — read attached FNOL forms or phone transcript summaries from SharePoint
- `GetDriveChildren` — check existing claim tracker for duplicate entries

**Output:** Structured claim case record written to the Excel claim tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Claim ID, Claimant Name, Policy Number, Date of Loss, Loss Type, Severity, Reporting Channel, Status, Created Date, Assigned To, Jurisdiction, Product Line

**Guardrails:**
- Never create duplicate claim cases for the same policy number and date of loss
- Validate that date of loss is not in the future
- Confirm all case details with the user before writing to the tracker
- Never auto-populate coverage status or reserve amounts
- Log the intake source channel for audit trail purposes

---

#### 2. `ins-claim-context` — Gather Policy, Claimant, and Loss Context

**Framework Step:** INS-CLM-002

**Trigger phrases:** "build claim context for", "assemble claim packet for [claim ID]", "gather policy details for this loss", "what do we know about claim [number]"

**Inputs:**
- Claim ID or claimant name
- Policy number (if available)

**M365 tools:**
- `SearchM365(sources=["files"])` — policy documents, endorsements, prior claim summaries, evidence checklists
- `SearchM365(sources=["connectors"], connector_ids=["policy-admin-connector"])` — policy administration system data via Graph Connector
- `ReadFileContent` — SharePoint policy documents, claim handling guidelines, endorsement schedules
- `GetDriveChildren` — evidence folder contents for the claim
- `SearchM365(sources=["email"])` — correspondence related to the loss or claimant
- `GetUserDetails` — resolve adjuster and supervisor contacts

**Output:** Word document containing:
- Policy summary (coverage type, limits, deductible, effective dates, named insureds)
- Claimant and insured profile
- Loss narrative summary
- Prior claim history extract
- Evidence inventory (documents received to date)
- Applicable jurisdiction and regulatory notes

**Artifact:** Word (.docx) saved to the claim's SharePoint folder; the `docx` skill handles generation

**Guardrails:**
- Never include coverage opinions or reserve recommendations in the context packet
- Cite policy document source and version for every extract
- Flag if the policy was not active on the date of loss — present as a factual finding, not a coverage determination
- Mask claimant SSN, financial account numbers, and medical details in generated documents unless explicitly required by the claim type
- Flag if any required policy document is unfindable in SharePoint

---

#### 3. `ins-claim-classifier` — Classify Claim Type and Severity

**Framework Step:** INS-CLM-003

**Trigger phrases:** "classify this claim", "what type of claim is this", "assess severity for [claim ID]", "claim triage classification", "categorize this loss"

**Inputs:**
- Claim context packet (Word doc or case data from Excel tracker)
- Claims taxonomy reference (SharePoint document)

**M365 tools:**
- `ReadFileContent` — claim context packet, claims taxonomy document, severity rubric from SharePoint
- `SearchM365(sources=["files"])` — claim handling guidelines, catastrophe event bulletins
- `SearchM365(sources=["connectors"], connector_ids=["claims-platform-connector"])` — active catastrophe declarations

**Output:** Classification report as Adaptive Card (for quick adjuster review) plus Excel tracker update (claim type, severity, confidence score, handling lane recommendation)

**Logic:** Compare loss narrative, date of loss, location, and policy type against the claims taxonomy. Assign claim type (property damage, bodily injury, liability, auto physical damage, catastrophe, etc.), severity tier (low, medium, high, complex), and confidence level. Flag claims that match multiple categories or catastrophe event zones.

**Guardrails:**
- Classification is always presented as a recommendation for adjuster review, never as a final determination
- Include confidence score with every classification — flag low-confidence cases prominently
- Never auto-assign severity for bodily injury or high-value property claims without human review
- Present findings for user review before updating the Excel tracker
- All classification outputs must include the statement: "DRAFT FOR ADJUSTER REVIEW — not a coverage or liability determination"

---

#### 4. `ins-coverage-gap-detection` — Assess Initial Coverage Path and Missing Evidence

**Framework Step:** INS-CLM-004

**Trigger phrases:** "check evidence gaps for [claim ID]", "what's missing on this claim", "coverage prep check", "evidence audit for claim", "assess coverage path"

**Inputs:**
- Claim context packet (Word document)
- Evidence checklist (SharePoint document, varies by claim type and product line)
- Photo and document inventory from the claim folder

**M365 tools:**
- `ReadFileContent` — evidence checklist from SharePoint, policy endorsements, claim handling standards
- `GetDriveChildren` — claim evidence folder contents to check what has been uploaded
- `SearchM365(sources=["files"])` — uploaded claim documents (photos, police reports, repair estimates, proof of loss forms)
- `SearchM365(sources=["connectors"], connector_ids=["claims-platform-connector"])` — fraud indicator flags from claims platform

**Output:** Gap report as Adaptive Card (for quick review) plus Word document (formal evidence gap analysis) plus Excel worksheet update (for tracking)

**Logic:** Compare required checklist items against uploaded documents in the claim's evidence folder. Identify missing documents, approaching regulatory deadlines, fraud indicator flags (suspicious prior claim patterns, repeated provider submissions, conflicting narratives), and policy-sensitive conditions (inactive policy on DOL, endorsement exclusions, jurisdiction-specific requirements).

**Guardrails:**
- Never mark an evidence item as complete without document evidence in the SharePoint folder
- Never state that coverage exists or does not exist — frame all findings as "potential coverage path considerations for adjuster review"
- Flag fraud indicators separately with elevated visibility but never label a claim as fraudulent
- All gap findings must include the statement: "DRAFT ANALYSIS — coverage determination requires adjuster review"
- Flag jurisdiction-specific evidence requirements (e.g., state-mandated proof of loss timelines)
- Present findings for user review before updating the Excel tracker

---

#### 5. `ins-claim-routing` — Route to Adjuster, Catastrophe Desk, or SIU

**Framework Step:** INS-CLM-005

**Trigger phrases:** "route this claim", "assign adjuster for [claim ID]", "where should this claim go", "recommend handling lane", "claim routing recommendation"

**Inputs:**
- Classification report output
- Gap detection output
- Routing rules matrix (SharePoint document)
- Fraud trigger checklist (SharePoint document)
- Adjuster territory and specialization assignments

**M365 tools:**
- `ReadFileContent` — routing rules matrix, fraud trigger checklist, adjuster assignment guide from SharePoint
- `SearchPeople` — resolve adjuster names by territory and specialization
- `GetManagerDetails` / `GetDirectReportsDetails` — claims operations org structure for escalation
- `PostMessage` — Teams notification to assigned adjuster or team with claim summary
- `CreateDraftMessage` — Outlook draft for formal routing notification
- `CreateEvent` — calendar holds for SLA-driven review deadlines

**Output:** Routing recommendation with:
- Recommended handling lane (standard adjuster, catastrophe desk, bodily injury team, SIU, complex claims unit)
- Assigned adjuster or team
- Teams message to the receiving party with claim summary
- Calendar event for SLA review deadline
- Updated Excel tracker with routing assignment

**Guardrails:**
- Present routing recommendation for claims operations review before sending any messages or notifications
- Never auto-route to SIU without explicit human confirmation — SIU referrals carry regulatory and legal implications
- Never auto-assign claims above a configurable severity threshold without supervisor review
- Escalate to the claims operations manager if no clear routing path is found
- Include SLA deadline in all routing notifications
- Routing rationale must be captured in the claim tracker for audit purposes

---

#### 6. `ins-claim-comms` — Draft Claimant, Broker, and Adjuster Communications

**Framework Step:** INS-CLM-006

**Trigger phrases:** "draft claimant letter for [claim ID]", "send deficiency notice", "prepare adjuster summary", "broker update for this claim", "claim follow-up communication"

**Inputs:**
- Claim case data from Excel tracker
- Claim context packet (Word document)
- Gap detection report
- Classification and routing outputs
- Target audience (claimant, broker/agent, assigned adjuster, supervisor)
- Communication templates from SharePoint

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages to internal claim team
- `SearchM365(sources=["files"])` — communication templates from SharePoint
- `ReadFileContent` — approved letter templates, regulatory language guides

**Communication templates:**
- Claimant acknowledgment letter (FNOL received, next steps, assigned contact)
- Evidence deficiency notice to claimant (missing documents, deadline for submission)
- Broker/agent status update
- Adjuster assignment summary (internal, with full case context)
- SIU referral memo (internal only, never sent to claimant)
- Reservation of rights letter draft (requires legal review before sending)

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Never include coverage opinions, liability assessments, or reserve amounts in any communication
- Never use commitment language ("your claim is covered", "we will pay") — all language must be neutral and procedural
- Claimant-facing communications must comply with jurisdiction-specific unfair claims practices act requirements
- Include claim reference number in every communication
- Reservation of rights letters must be flagged as requiring legal review before sending
- SIU-related content must never appear in claimant-facing or broker-facing communications
- All drafts must include the label: "DRAFT — requires adjuster/supervisor review before sending"

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** INS-CLM-007

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step. In insurance, this is non-negotiable: the claim supervisor or senior adjuster must confirm the handling lane, validate the evidence state, and approve the triage outcome. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the disposition review meeting
- The claim context packet and gap report artifacts — provide the evidence package for the reviewer
- The `ins-claim-comms` skill — send the disposition confirmation email after the human decision is made
- The Excel claim tracker — updated to final triage status by the reviewer

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. For Insurance, this translation carries additional weight because of regulatory scrutiny, litigation exposure, and the document-intensive nature of claims handling.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle an entire claims workflow. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills that score high on the quality rubric's Scope Boundaries dimension. For insurance, this is critical: a single "claims handler" skill would inevitably drift into coverage opinions.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation | Insurance-Specific Notes |
|---|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` | Final coverage determination, reserve setting, payment authorization, formal denial |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions | Claim classification, evidence gap detection — adjuster sees recommendation, decides |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation | Routing recommendations, claimant communications — every output reviewed before action |
| AI act within policy | Skill can execute bounded write actions (update tracker, assemble packet) within defined rules | Context assembly, evidence retrieval — safe to execute within policy boundaries |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed | FNOL intake normalization — structured extraction with no interpretation |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather claim context," the framework requires naming every input source. This translates to specific MCP tool calls in the SKILL.md instructions.

### 3.2 What Cowork Provides That the Framework Assumes You Build

The framework describes a 9-layer reference architecture. In Cowork, most of these layers are platform-provided:

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — `/mnt/user-config/.claude/skills/` IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and confirmation gates provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through the quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (claim status, prior decisions, evidence chain, routing history) must be externalized to M365 artifacts. For insurance, this is where first-class artifact support becomes essential — the claim file IS the process state.

### 3.3 M365 Artifacts as First-Class Process State

This is the most important architectural insight for making the framework useful in Cowork. Insurance is heavily document-centric, and each M365 artifact type serves a specific role in the claims process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (claim tracker, evidence checklist status, classification log, reserve tracking) | The claim tracker workbook IS the process state — skills read current claim status, write triage updates, track evidence completeness, and log routing decisions |
| **Word** | Evidence artifacts (claim context packets, coverage analysis drafts, adjuster summaries, reservation of rights letter drafts) | Skills generate documents that become the auditable claim file record; every Word artifact is a draft for human review |
| **PowerPoint** | Decision-support artifacts (catastrophe event briefings, claim portfolio reviews, management status decks) | Skills create decks that support supervisor reviews — the claims manager reviews a deck for portfolio status, not raw data |
| **SharePoint** | Source of truth (policy documents, endorsement schedules, evidence files, claim handling guidelines, routing matrices, fraud trigger checklists, communication templates) | Skills read policies and checklists from SharePoint; uploaded evidence (photos, police reports, repair estimates, proof of loss forms) lives here; the claim folder structure IS the evidence chain |
| **Outlook** | Communication channel and signal source | Skills read incoming loss notifications for context; draft claimant acknowledgments, deficiency notices, and adjuster summaries as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post routing notifications, adjuster assignments, escalation notices, and catastrophe team updates to channels or chats |
| **Graph API** | People and org data (adjuster profiles, territory assignments, supervisor hierarchy) | Skills resolve adjusters, claims supervisors, SIU analysts, and broker contacts for routing decisions |
| **Calendar** | Time-bound process events (SLA deadlines, disposition review meetings, regulatory response deadlines) | Skills create calendar events for triage SLA milestones, evidence submission deadlines, and review meetings |

**The design pattern:** Instead of a database-backed claims workflow engine, the Cowork Insurance plugin uses a SharePoint-hosted Excel workbook as the canonical claim tracker, with Word documents as the evidence and analysis trail, and SharePoint folders as the claim file structure. Each skill reads from and writes to this shared state through M365 tools. This is less formally rigorous than a purpose-built claims management system, but it works within the M365 ecosystem and gives claims teams artifacts they already know how to work with.

### 3.4 Federated Connectors for Third-Party Systems

The framework references systems like "claims platform", "policy administration system", "catastrophe feed", and "SIU database" — these do not exist natively in M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For policy administration systems (Guidewire PolicyCenter, Duck Creek Policy, Majesco), claims management platforms (Guidewire ClaimCenter, Duck Creek Claims), and fraud detection systems (SIU platforms, NICB databases), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["policy-admin-connector"])`. This provides read access to policy summaries, claim status, prior claim history, and fraud indicators without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks that are populated by Power Automate flows from the third-party system. Policy summaries, evidence checklists, routing matrices, and catastrophe event bulletins are maintained in SharePoint. Skills interact with the SharePoint copy. Bidirectional sync for claim status updates is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For systems with no integration path (smaller carriers, legacy platforms, reinsurance portals, state regulatory filing systems), skills provide structured intake that captures data from manual lookups, writing it into the shared Excel tracker. The framework's Signal Inventory phase identifies exactly which data points are needed, so the skill can prompt for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for policy and claim data, and Tier 3 (manual input) for actuarial tools and reinsurance platforms. Graph Connectors for Guidewire or Duck Creek require tenant admin setup and are better introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

Insurance governance is driven by state regulatory requirements (Department of Insurance rules vary by jurisdiction), unfair claims practices acts, bad-faith litigation exposure, claimant privacy requirements, and reserving authority limits. The framework's governance model maps to Cowork as follows:

| Governance Domain | Cowork Implementation | Insurance-Specific Requirements |
|---|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure and escalation paths | Claims operations manager owns process definition; claims leadership owns routing rules; legal owns reservation of rights language |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC | SIU case materials restricted to SIU analysts; medical records restricted to bodily injury adjusters; claimant financial data access logged |
| **Data classification** | Skill guardrails enforce PII handling rules | Claimant SSN, medical records, and financial data must be masked in generated documents; SIU indicators never appear in claimant-facing outputs |
| **Audit** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail | Every triage decision must be traceable; write actions preserve actor, timestamp, prior value, and claim linkage per governance rules |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric provides a pre-deployment gate | Routing rules and fraud triggers must be validated by claims operations before skill updates go live |
| **Policy enforcement** | Encoded in skill instructions ("always draft, never auto-send", "require confirmation before updating tracker") | Coverage opinions prohibited; reserve recommendations prohibited; liability assessments prohibited; SIU referrals require human confirmation; claimant communications require unfair claims practices act compliance by jurisdiction |
| **Regulatory compliance** | Dynamic policy content read from SharePoint at runtime | State-specific claims handling timelines, mandatory disclosure language, and evidence requirements maintained in SharePoint and read dynamically — not hard-coded into skill instructions |
| **Litigation hold** | Skill guardrails flag active litigation holds | If a claim is flagged with a litigation hold in the tracker, skills must prevent deletion or modification of evidence artifacts |

**The main governance gap** is formal policy versioning across jurisdictions. Insurance claims handling rules vary by state, product line, and claim type. The recommended mitigation is to keep all policy content, routing rules, evidence checklists, and regulatory language in SharePoint documents organized by jurisdiction. Skills read these dynamically at runtime, so claims operations can update state-specific requirements without modifying the skill instructions.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork and insurance-specific metrics:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do generated documents contain accurate, cited, and regulation-compliant information? Assessed via manual review of 10+ outputs by senior adjusters
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing against the SharePoint document library

**Process-level evaluation (end-to-end):**
- Triage cycle time — time from FNOL creation to "Ready for Adjuster Review" status (target: under 4 business hours)
- Claim type classification accuracy — percentage of claims correctly classified by type and severity (target: at least 85 percent)
- Evidence gap detection rate — percentage of actual missing documents identified by `ins-coverage-gap-detection` (target: at least 85 percent)
- Incorrect routing rate — percentage of claims routed to the wrong handling lane (target: below 5 percent)
- Draft acceptance rate — percentage of `ins-claim-comms` drafts sent without major edits (target: at least 75 percent)
- Unauthorized claim actions — any instance of the system making coverage determinations, reserve changes, or payment authorizations (target: zero)
- Missing audit fields — any triage action without actor, timestamp, and claim linkage (target: zero)
- Adjuster rework rate — how often adjusters must re-triage after receiving the AI-prepared packet

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork and the insurance pilot design:

### Wave 1 — Foundation (Intake, Context, and Detection)

**Infrastructure setup:**
- Create the shared Excel claim tracker workbook in SharePoint with standard columns (Claim ID, Claimant Name, Policy Number, Date of Loss, Loss Type, Severity, Jurisdiction, Product Line, Reporting Channel, Status, Created Date, Assigned To, Handling Lane, Evidence Completion %, Fraud Flag)
- Upload claims handling guidelines, evidence checklists (by product line and state), routing matrices, fraud trigger checklists, and communication templates to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-claim evidence (photos, police reports, repair estimates, proof of loss forms, medical documents)
- Upload claims taxonomy and severity rubric documents

**Skills to build:**
- `ins-fnol-intake`
- `ins-claim-context`
- `ins-claim-classifier`
- `ins-coverage-gap-detection`

**Operating posture:** AI assist mode only for classification and gap detection. Skills read and analyze but do not write to systems without confirmation. All outputs are presented via Adaptive Card or generated documents for manual adjuster review. Test with one product line (e.g., homeowners property damage) and one intake team. Run 20-30 real FNOL cases through the pilot.

### Wave 2 — Communication and Routing

**Skills to build:**
- `ins-claim-routing`
- `ins-claim-comms`

**Promotions:**
- Promote `ins-coverage-gap-detection` to write-back mode (updates Excel tracker after user confirmation)
- Promote `ins-fnol-intake` to write mode (creates claim case records after confirmation)
- Promote `ins-claim-classifier` to write-back mode (logs classification to tracker after confirmation)

**Integrations:**
- Introduce Graph Connectors for policy administration system data if available at the tenant level
- Set up Power Automate flows to synchronize claim status between the Excel tracker and the claims management platform

**Automation:**
- Set up a daily scheduled prompt that checks for claims approaching SLA deadlines (4-hour triage target) and surfaces any with incomplete evidence or unassigned routing

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action. Weekly review of fraud indicator flags, routing accuracy, and communication quality by claims operations.

### Wave 3 — Optimization and Expansion

**Enhancements:**
- Add bounded multi-document claim packet assembly for complex losses (multi-peril, bodily injury, commercial claims)
- Add proactive detection of repeated suspicious patterns (same provider, same location, frequent claims on same policy) via scheduled prompt
- Introduce Graph Connectors for claims platform and SIU system if available
- Add catastrophe event monitoring — scheduled prompt that checks for declared catastrophe events and flags affected open claims
- Expand to additional product lines (auto, commercial property, general liability)

**Measurement:**
- Triage cycle time reduction vs. pre-pilot baseline
- Draft acceptance rate target: above 75 percent
- Evidence gap detection accuracy target: above 85 percent
- Incorrect routing rate target: below 5 percent
- Unauthorized claim actions target: zero
- Missing audit fields target: zero
- Adjuster rework rate target: below 10 percent

---

## Part 5: Generalizing the Approach — Insurance's Artifact Pattern

Insurance's natural M365 artifact pattern is **Excel (claim tracking and evidence checklists) + Word (context packets, coverage analysis drafts, communication drafts) + SharePoint (policy documents, evidence files, regulatory references)**. This reflects the domain's core characteristic: claims handling is fundamentally a document assembly, evidence review, and human judgment workflow.

The Insurance pattern demonstrates a critical principle for heavily regulated industries: the guardrail architecture must be designed before the skill logic. In domains with bad-faith litigation exposure, unfair claims practices act requirements, and per-jurisdiction regulatory variation, the prohibitions (what the skill must never do) are more important than the capabilities (what the skill can do). Every skill in this plan includes explicit prohibitions against coverage determinations, reserve recommendations, and liability assessments. This "guardrails-first" approach applies equally to other regulated verticals such as healthcare claims processing, financial services compliance, and government benefits administration.

The framework's decomposition method works well for insurance because claims triage naturally breaks into discrete steps with clear handoff points. The key adaptation is that insurance requires dynamic policy content (read from SharePoint at runtime, organized by jurisdiction and product line) rather than static rules encoded in skill instructions. This ensures that when state regulations change or new catastrophe events are declared, claims operations can update the reference documents without modifying the skills themselves.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common claim tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic "claims platform" and "policy admin system" references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools; introduce agent only for complex multi-document claim assembly in later waves |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy and regulatory content read from SharePoint; coverage prohibitions are non-negotiable |
| Process State | SharePoint-hosted Excel workbook and claim folder structure | Durable state externalized to M365 artifacts; the claim tracker IS the process state; the evidence folder IS the claim file |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival (loss notification), Teams message (adjuster coordination), form submission (portal FNOL), broker communication |
| Approval | Draft tools + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; SIU referrals and routing require explicit confirmation |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via triage cycle time, classification accuracy, gap detection rate, and routing accuracy |
| Governance | Skill guardrails + SharePoint policy documents + M365 RBAC | Insurance adds: jurisdiction-specific regulatory compliance, litigation hold awareness, unfair claims practices act compliance, SIU referral protocols, and claimant privacy requirements |
