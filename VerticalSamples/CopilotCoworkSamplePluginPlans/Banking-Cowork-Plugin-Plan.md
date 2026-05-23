# Plan: Banking Fraud Alert and Dispute Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Banking line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Fraud Alert and Dispute Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

Banking is a heavily regulated vertical where customer data sensitivity, financial crime prevention requirements, and strict audit obligations shape every design decision. The automation boundary in this domain is defined not only by operational efficiency but by regulatory mandates from the OCC, FDIC, CFPB, and FinCEN.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for banking, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where regulatory density creates additional constraints.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly; banking adds regulatory approval structure as a first-class selection factor |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule is essential in banking where mixing account action with triage creates compliance risk |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — banking references "core banking platform", "fraud platform", and "card processor" that must map to M365 tools or Graph Connectors |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — banking's strict boundary between preparation and account action aligns naturally with Cowork's review-before-action pattern |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates; banking's classify and route patterns map to Decision Support |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1 through 5 and 7 through 9 natively; the skill author controls capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Banking's regulatory density means that the automation boundary phase carries more weight than in less regulated verticals. Every skill must encode not just "what can AI do" but "what must AI never do" — no account blocking, no transaction reversal, no provisional credit authorization, no SAR clearance. The framework's automation boundary translates directly to Cowork guardrails, but in banking these guardrails are regulatory requirements, not just operational preferences. This makes the guardrails section of each SKILL.md the most compliance-critical component of the entire plugin.

---

## Part 2: The Banking Fraud and Dispute Triage Plugin — Skill-by-Skill Design

The Banking sample decomposes "Fraud Alert and Dispute Triage" into 7 steps (BNK-FRD-001 through BNK-FRD-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| BNK-FRD-001: Normalize fraud or dispute event | `bnk-case-intake` | Data Aggregation | Deterministic automation | Excel (case tracker), SharePoint (intake records), Outlook (alert notifications) |
| BNK-FRD-002: Gather account, transaction, and customer context | `bnk-fraud-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (procedures), Graph API (people) |
| BNK-FRD-003: Classify fraud scenario or dispute type | `bnk-scenario-classifier` | Decision Support | AI assist | Adaptive Card (classification report), Excel (tracker update), SharePoint (fraud taxonomy) |
| BNK-FRD-004: Assess urgency, evidence gaps, and risk path | `bnk-gap-risk-detection` | Decision Support | AI assist | Adaptive Card (risk assessment), Excel (evidence checklist), SharePoint (dispute procedures) |
| BNK-FRD-005: Route to analyst queue or escalation lane | `bnk-case-routing` | Decision Support | AI draft + approve | Teams (routing messages), Excel (tracker assignment), SharePoint (routing matrix) |
| BNK-FRD-006: Draft customer and analyst communications | `bnk-case-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (analyst notifications), Word (case summary) |
| BNK-FRD-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (disposition confirmation) |

### Detailed Skill Designs

#### 1. `bnk-case-intake` — Normalize Fraud or Dispute Event

**Framework Step:** BNK-FRD-001

**Trigger phrases:** "new fraud alert", "dispute case for [customer]", "normalize this fraud event", "intake fraud report", "new dispute submission"

**Inputs:**
- Fraud alert details (alert ID, transaction reference, channel)
- Customer name or account number
- Dispute type or fraud indicator
- Source channel (digital, branch, contact center, monitoring system)

**M365 tools:**
- `SearchM365(sources=["email"])` — find the alert notification or customer dispute email
- `SearchM365(sources=["files"])` — locate intake form templates and normalization checklists
- `ReadFileContent` — read intake normalization procedures from SharePoint
- `SearchPeople` — resolve customer service representative or branch contact identities

**Output:** Structured case record written to the Excel fraud and dispute case tracker in SharePoint; confirmation via Adaptive Card showing normalized case fields

**Artifact:** Excel workbook with columns: Case ID, Alert ID, Customer Name, Account Number, Transaction Reference, Dispute Type, Fraud Indicator, Channel, Status, Created Date, Assigned To, SLA Deadline, Priority

**Guardrails:**
- Never create duplicate cases for the same alert ID or transaction reference
- Validate that all mandatory intake fields are populated before writing to tracker
- Confirm case details with user before writing to the tracker
- Never modify account status, card status, or transaction state
- Log the intake source and timestamp for audit traceability

---

#### 2. `bnk-fraud-context-packet` — Gather Account, Transaction, and Customer Context

**Framework Step:** BNK-FRD-002

**Trigger phrases:** "build fraud context packet", "assemble case context for [case ID]", "gather transaction history for this dispute", "what do we know about this fraud case"

**Inputs:**
- Case ID or customer account number from the intake tracker

**M365 tools:**
- `ReadFileContent` — read case record from the Excel tracker, fraud procedures, and evidence checklists from SharePoint
- `SearchM365(sources=["files"])` — locate prior case documents, customer correspondence, and uploaded evidence
- `SearchM365(sources=["email"])` — find related customer communications and alert threads
- `SearchM365(sources=["connectors"], connector_ids=["core-banking-connector"])` — retrieve account and transaction data from indexed core banking records
- `GetDriveChildren` — list documents in the case evidence folder
- `GetUserDetails` — resolve analyst and supervisor identities

**Output:** Word document containing:
- Customer and account profile summary
- Transaction details for the disputed or flagged activity
- Prior fraud alert and dispute history
- Relevant merchant and channel context
- Applicable fraud procedures and dispute handling rules
- Evidence inventory (what has been received, what is missing)

**Artifact:** Word (.docx) saved to the case folder in SharePoint

**Guardrails:**
- Never include full account numbers or card numbers in generated documents — mask to last four digits
- Cite the source system and retrieval timestamp for every data element
- Flag if any required data source is unavailable or returns stale results
- Do not surface SAR or suspicious activity investigation status — these are restricted to AML teams
- Read-only access to all banking data — no write operations to core systems

---

#### 3. `bnk-scenario-classifier` — Classify Fraud Scenario or Dispute Type

**Framework Step:** BNK-FRD-003

**Trigger phrases:** "classify this fraud case", "what type of dispute is this", "categorize fraud scenario", "triage classification for [case ID]"

**Inputs:**
- Fraud context packet (Word document or case data from Excel tracker)
- Fraud taxonomy document from SharePoint

**M365 tools:**
- `ReadFileContent` — fraud taxonomy, dispute type definitions, and handling lane criteria from SharePoint
- `SearchM365(sources=["files"])` — approved classification examples and scenario playbooks
- `GetDriveChildren` — case evidence folder contents for pattern matching context

**Output:** Classification report as Adaptive Card (for quick analyst review) showing:
- Recommended scenario type (e.g., card-not-present fraud, account takeover, friendly fraud, billing dispute, merchant error)
- Confidence level (high, medium, low)
- Handling lane recommendation (fraud operations, dispute operations, chargeback, AML escalation)
- Supporting evidence summary

**Logic:** Compare case attributes against the fraud taxonomy and dispute type definitions in SharePoint. Match transaction patterns, channel characteristics, and customer narrative against known scenario profiles.

**Guardrails:**
- Present classification as a recommendation only — never auto-assign the final scenario type
- Always display the confidence level and the evidence supporting the classification
- Flag cases where classification confidence is below the threshold defined in the taxonomy document
- Never auto-clear fraud indicators or suspicious activity flags
- If the case matches AML or SAR escalation criteria, flag immediately without further classification detail

---

#### 4. `bnk-gap-risk-detection` — Assess Urgency, Evidence Gaps, and Risk Path

**Framework Step:** BNK-FRD-004

**Trigger phrases:** "check evidence gaps", "what's missing for [case ID]", "assess fraud case urgency", "risk assessment for this dispute", "audit case readiness"

**Inputs:**
- Fraud context packet (Word document)
- Dispute evidence checklist from SharePoint
- Case data from Excel tracker

**M365 tools:**
- `ReadFileContent` — dispute evidence checklist, fraud operations rules, and Reg E handling guidance from SharePoint
- `SearchM365(sources=["files"])` — uploaded evidence documents in the case folder
- `GetDriveChildren` — case evidence folder contents to verify what has been received
- `SearchM365(sources=["email"])` — check for pending customer responses or affidavit submissions

**Output:** Risk assessment as Adaptive Card (for analyst review) plus Excel tracker update (after confirmation):
- Urgency recommendation (immediate, standard, low priority)
- Evidence gap list with specific missing items
- Escalation flags (repeated pattern, cross-border activity, high-value transaction, regulatory timeline risk)
- Reg E or card network compliance timeline status

**Guardrails:**
- Never mark an evidence item as complete without document verification in the SharePoint case folder
- Flag Reg E provisional credit deadlines explicitly — these are regulatory, not operational
- Present all findings for analyst review before updating the Excel tracker
- Separate AML-related escalation flags from standard fraud or dispute flags — these route differently
- Never recommend closing or de-prioritizing a case without analyst confirmation
- Include the SLA deadline status in every assessment output

---

#### 5. `bnk-case-routing` — Route to Analyst Queue or Escalation Lane

**Framework Step:** BNK-FRD-005

**Trigger phrases:** "route this fraud case", "assign case to analyst", "who handles this dispute type", "recommend routing for [case ID]", "escalate this case"

**Inputs:**
- Classification output (scenario type, handling lane)
- Risk assessment output (urgency, escalation flags)
- Case data from Excel tracker
- Routing matrix document from SharePoint

**M365 tools:**
- `ReadFileContent` — routing matrix and reviewer assignment rules from SharePoint
- `SearchPeople` — resolve analyst and supervisor identities by name or function
- `GetManagerDetails` / `GetDirectReportsDetails` — fraud operations org structure for escalation paths
- `PostMessage` — Teams notification to assigned analyst or escalation queue (after approval)
- `CreateDraftMessage` — Outlook draft for cases requiring email-based routing
- `ListCalendarView` — check analyst availability for urgent cases

**Output:** Routing recommendation presented for review:
- Recommended analyst or queue
- Routing rationale citing the routing matrix criteria
- Escalation path if applicable
- Teams messages to assigned parties (after user approval)
- Updated Excel tracker with analyst assignment and routing timestamp

**Guardrails:**
- Present routing recommendation for fraud operations review before sending any messages
- Never auto-route to AML or suspicious activity review lanes — these require explicit supervisor approval
- Never assign a case to an analyst outside the approved routing matrix
- Escalate to the fraud operations supervisor if no clear routing path exists
- Enforce segregation of duties — the analyst who performed intake should not be auto-assigned to review
- Log routing decision, rationale, and approver for audit trail

---

#### 6. `bnk-case-comms` — Draft Customer and Analyst Communications

**Framework Step:** BNK-FRD-006

**Trigger phrases:** "draft customer message for [case ID]", "prepare affidavit request", "send analyst summary", "fraud case follow-up", "dispute notification draft"

**Inputs:**
- Case data from Excel tracker
- Fraud context packet (Word document)
- Classification and risk assessment outputs
- Target audience (customer, analyst, supervisor, branch)
- Communication templates from SharePoint

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages to analyst channels (after approval)
- `SearchM365(sources=["files"])` — approved communication templates from SharePoint
- `ReadFileContent` — template content and case summary documents

**Communication templates:**
- Customer acknowledgment of dispute receipt
- Affidavit or statement request to customer
- Missing evidence follow-up to customer
- Analyst case summary and handoff note
- Supervisor escalation notification
- Branch coordination message

**Guardrails:**
- Always create as Outlook draft — never send customer communications without explicit user confirmation
- Never include commitment language regarding provisional credit, reimbursement timeline, or case outcome
- Never disclose fraud investigation details, SAR status, or internal fraud scores in customer-facing communications
- Use only approved templates for customer communications — no freeform customer messaging
- Include the case reference number in every communication
- Mask account and card numbers to last four digits in all outputs
- Match tone to audience: procedural for customers, operational for analysts, summary for supervisors

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** BNK-FRD-007

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step. In banking, this is a regulatory and operational requirement — final fraud or dispute disposition decisions carry compliance accountability that cannot be delegated to AI. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the disposition review meeting
- The case summary artifacts — provide the evidence package for the fraud supervisor
- The `bnk-case-comms` skill — send the disposition confirmation email after the human decision is made
- The Excel tracker — record the final disposition, approver identity, and timestamp for audit

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. Banking adds a layer of regulatory complexity that shapes every translation decision. This section analyzes how to approach that translation systematically.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in banking is **pre-implementation discipline in a regulated environment**:

**Process decomposition prevents mega-skills and compliance boundary violations.** The common Cowork anti-pattern is building one broad skill that tries to handle an entire domain. In banking, a mega-skill that combines case intake with account action creates regulatory risk. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills that respect the boundary between preparation and account action.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` for disposition reviews |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions — used for classification and risk assessment |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation — used for routing and communications |
| AI act within policy | Skill can execute bounded read and retrieval actions (gather context, assemble packet) within defined rules — used for context assembly |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed — used for intake normalization |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather account context," the framework requires naming every input source. This translates to specific MCP tool calls in the SKILL.md instructions — critical in banking where accessing the wrong data source or missing a required source creates compliance gaps.

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
| Decision and approval plane | Partial — draft tools and user confirmation provide human-in-the-loop; no formal banking approval workflow engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom regulatory policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through the quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine or a formal regulatory audit trail. The framework's "Process State" concept (case status, routing history, disposition decisions, evidence chain) must be externalized to M365 artifacts. In banking, this state must also satisfy audit and examination requirements, making the artifact design more critical than in less regulated verticals.

### 3.3 M365 Artifacts as First-Class Process State

This is the most important architectural insight for making the framework useful in Cowork for banking. Each M365 artifact type serves a specific role in the process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (case tracker, evidence checklist, routing log, SLA monitoring) | The fraud and dispute case tracker workbook IS the process state — skills read case status, write updates, track evidence completeness, and log routing decisions |
| **Word** | Evidence artifacts (context packets, case summaries, procedure extracts) | Skills generate documents that become the auditable record of what was assembled, classified, and reviewed |
| **SharePoint** | Source of truth (fraud taxonomy, dispute procedures, routing matrix, evidence checklists, Reg E guidance, approved templates) | Skills read procedures and checklists from SharePoint; case evidence documents are stored in per-case folders |
| **Outlook** | Communication channel and signal source | Skills read incoming fraud alerts and customer dispute emails for context; draft outgoing communications as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post routing notifications, analyst assignments, escalation alerts, and supervisor notifications to channels or chats |
| **Adaptive Card** | Real-time decision support display | Skills present classification results, risk assessments, and gap reports as Adaptive Cards for analyst review without persisting to a document |
| **Graph API** | People and org data (analyst profiles, supervisor hierarchy, routing matrix resolution) | Skills resolve analyst identities, org structure, and reporting chains for routing decisions |
| **Calendar** | Time-bound process events (SLA deadlines, review meetings, escalation windows) | Skills create calendar events for disposition reviews and SLA deadline reminders |

**The design pattern:** Instead of a database-backed case management system, the Cowork banking plugin uses a SharePoint-hosted Excel workbook as the canonical case tracker, with Word documents as the evidence and context trail, and SharePoint document libraries as the case folder structure. Each skill reads from and writes to this shared state through M365 tools. This is less formally rigorous than a purpose-built banking case system, but it works within the M365 ecosystem and provides a document-based audit trail that examiners can review.

### 3.4 Federated Connectors for Third-Party Banking Systems

The framework references systems like "core banking platform", "fraud platform", "card processor", and "case management system" — these do not exist natively in M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For core banking platforms (FIS, Fiserv, Jack Henry), fraud platforms (NICE Actimize, FICO Falcon, Featurespace), and KYC/AML systems (Accuity, LexisNexis), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["core-banking-connector"])`. This provides read access to account profiles, transaction summaries, and prior case history without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks that are populated by Power Automate flows from the banking system. Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork. This is particularly relevant for routing matrices, fraud taxonomy tables, and evidence checklists that change infrequently.

**Tier 3 — Manual Input with Templates**

For systems with no integration path, skills provide structured intake that captures data from manual analyst lookups, writing it into the shared Excel tracker. The framework's Signal Inventory phase identifies exactly which data points are needed, so the skill can prompt for only what is missing rather than asking broad questions. This is common during pilot for fraud score context and card processor details.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) and Tier 3 (manual input). Graph Connectors for core banking systems require tenant admin setup, security review, and data classification approval. Introduce them in Wave 2 after the skill workflows are proven and the data access model is approved by the banking data owner.

### 3.5 Governance in Cowork for Banking

Banking governance requirements are substantially more demanding than general enterprise governance. The framework's governance model maps to Cowork with banking-specific regulatory overlays:

| Governance Domain | Cowork Implementation | Banking-Specific Requirements |
|---|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure and escalation paths | Fraud operations manager owns process definition; banking operations automation lead owns technical implementation; dual ownership is required by internal control standards |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC | Customer account data, transaction data, and fraud scores require role-based access controls aligned with banking data classification; SOX-relevant data access must be logged |
| **Data classification** | Skill guardrails enforce data handling rules (mask account numbers, restrict fraud scores) | GLBA requires customer financial data protection; account numbers, SSNs, and fraud investigation details must never appear unmasked in generated artifacts |
| **Segregation of duties** | Skill guardrails prevent same-person intake and review | OCC and FDIC examination guidance requires separation between case preparation and disposition authority; skills must not auto-assign reviewers who performed intake |
| **Audit trail** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail | Every case action must record actor, timestamp, prior value, and case linkage; BSA/AML examination requires demonstrable triage completeness |
| **Regulatory compliance** | Encoded in skill instructions and procedures read from SharePoint | Reg E provisional credit timelines, card network chargeback windows, BSA/AML escalation rules, and CFPB complaint handling requirements must be enforced through guardrails |
| **Policy enforcement** | Encoded in skill instructions ("always draft, never auto-send", "require confirmation before routing") | Dynamic policy content lives in SharePoint documents that skills read at runtime; when procedures change, operations updates SharePoint without modifying the skill |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric (0-100 scoring) provides a pre-deployment gate | Changes to fraud triage skills should undergo change management review by fraud operations and compliance before deployment |

**The main governance gap** is formal regulatory policy versioning and audit-grade logging. Cowork's platform-level logging captures tool invocations, but it does not produce the structured audit records that banking examiners expect. The recommended mitigation is to treat the Excel case tracker as the audit log — every skill write includes the actor, action, timestamp, and prior value as additional columns. For examination readiness, periodic exports of the tracker provide the documentary evidence trail.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for banking:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis (8 to 10 should-trigger and 8 to 10 should-not-trigger phrases per skill)
- Output quality — do generated context packets contain accurate, cited information? Assessed via manual review of 10 or more outputs against known case data
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing with representative case data
- Guardrail compliance — does the skill correctly refuse to perform prohibited actions (account blocking, SAR clearance, unmasked data output)? Assessed via adversarial testing

**Process-level evaluation (end-to-end):**
- Triage cycle time — time from alert or dispute receipt to review-ready case status, targeting 30-minute SLA
- Classification accuracy — percentage of fraud or dispute scenario types correctly identified, targeting at least 85 percent
- Evidence gap detection rate — percentage of actual missing evidence items identified, targeting at least 85 percent
- Incorrect routing rate — percentage of cases routed to the wrong analyst queue, targeting below 5 percent
- Unauthorized account actions — must be zero; any skill-initiated account modification is a critical failure
- Missing audit fields — must be zero; every case record must have complete traceability
- Customer draft acceptance rate — percentage of drafts sent with minor edits only, targeting at least 75 percent
- False negative rate — percentage of genuinely fraudulent cases that the classification skill missed or under-prioritized
- Analyst override rate — how often analysts change the skill's classification or routing recommendation

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation (Intake, Context, Classification, and Risk)

**Infrastructure setup:**
- Create the shared Excel fraud and dispute case tracker workbook in SharePoint with standard columns (Case ID, Alert ID, Customer Name, Account Number masked, Transaction Reference, Dispute Type, Fraud Indicator, Channel, Status, Created Date, Assigned To, SLA Deadline, Priority, Evidence Status, Classification, Routing, Disposition, Audit Log)
- Upload fraud taxonomy, dispute type definitions, evidence checklists, Reg E handling guidance, and routing matrix to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-case evidence storage
- Establish approved communication templates in SharePoint (customer acknowledgment, affidavit request, missing evidence follow-up, analyst handoff, escalation notification)

**Skills to build:**
- `bnk-case-intake`
- `bnk-fraud-context-packet`
- `bnk-scenario-classifier`
- `bnk-gap-risk-detection`

**Operating posture:** AI assist mode for classification and risk assessment. Context assembly operates within policy for read-only retrieval. Intake normalization runs as deterministic automation. All outputs are presented via Adaptive Card or generated documents for analyst review. No write-back to the case tracker without confirmation. Test with one fraud or dispute queue and one product set.

### Wave 2 — Routing and Communications

**Skills to build:**
- `bnk-case-routing`
- `bnk-case-comms`

**Promotions:**
- Promote `bnk-gap-risk-detection` to write-back mode (updates Excel tracker after analyst confirmation)
- Promote `bnk-case-intake` to write mode (creates case records after confirmation)
- Promote `bnk-scenario-classifier` to write-back mode (records classification in tracker after analyst confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for cases approaching SLA deadlines and surfaces any with incomplete status, missing evidence, or unassigned routing
- Set up a weekly scheduled prompt that summarizes case volume, classification distribution, routing patterns, and evidence gap trends

**Operating posture:** AI draft plus approve for all routing and communication skills. Every output reviewed by a fraud analyst or dispute specialist before action. Customer-facing communications require explicit send confirmation.

### Wave 3 — Optimization and Pattern Detection

**Enhancements:**
- Introduce Graph Connectors for core banking and fraud platform data if available at the tenant level and approved by the banking data owner
- Add proactive monitoring via scheduled prompt: flag repeated merchant patterns, recurring intake deficiencies, cases approaching regulatory deadlines, and classification accuracy drift
- Add bounded multi-document case packet assembly for complex or repeat-pattern cases — Word document combining context packet, classification rationale, evidence inventory, and routing recommendation into a single review-ready case file
- Refine all skills based on analyst override patterns, false negative analysis, and routing accuracy feedback from Waves 1 and 2

**Measurement:**
- Triage cycle time reduction versus pre-pilot baseline
- Classification accuracy target: at least 85 percent
- Evidence gap detection rate target: at least 85 percent
- Incorrect routing rate target: below 5 percent
- Customer draft acceptance rate target: at least 75 percent
- Unauthorized account actions: zero
- Missing audit fields: zero
- Analyst trust indicators: declining override rate across waves

---

## Part 5: Generalizing the Approach — Banking's Artifact Pattern

Banking's primary M365 artifact pattern is **Excel + Word + SharePoint** — structured case tracking combined with document-heavy evidence and procedure management. This places it in the same artifact family as HR and compliance, but with significantly higher regulatory overlay.

The distinguishing characteristics of banking for cross-LOB generalization:

1. **Guardrails are regulatory, not just operational.** In HR, a guardrail like "confirm before sending" is best practice. In banking, "never disclose SAR status to the customer" is a federal legal requirement. The framework's automation boundary phase must weight regulatory prohibitions as hard constraints, not preferences.

2. **Audit trail is examination-grade.** Banking examiners (OCC, FDIC, state regulators) expect structured evidence of every triage decision. The Excel tracker must serve as both operational state and audit record, with actor, timestamp, and prior value for every change.

3. **Third-party system dependency is high.** Banking workflows depend on core banking, fraud, card, and KYC systems that are not in M365. The federated connector strategy (Graph Connectors, SharePoint bridge, manual input) is more critical here than in verticals where most data lives in M365 natively.

4. **Segregation of duties is structural.** Banking skills must enforce separation between preparation and decision, between intake and review, and between fraud triage and AML escalation. This maps directly to Cowork's skill decomposition — each skill handles one step, and the guardrails prevent cross-boundary action.

The framework's decomposition methodology is universal. Banking demonstrates that the M365 artifact mapping and guardrail architecture must scale to meet the regulatory requirements of the specific vertical.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Banking-Specific Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common case tracker; banking processes have regulatory ownership requirements |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill; banking steps must respect the preparation-versus-action boundary |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability with banking-specific guardrails |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic "core banking" and "fraud platform" references; Graph Connectors bridge external systems |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session; banking workflows must preserve case state across skills |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly — prefer skills and tools; agents should not have access to account action capabilities |
| Policy or Guardrail | Guardrails section in SKILL.md | Banking guardrails encode regulatory requirements (Reg E, GLBA, BSA/AML); dynamic policy read from SharePoint procedures |
| Process State | SharePoint-hosted Excel workbook | Durable state externalized to M365 artifacts; must serve as both operational tracker and examination-grade audit record |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email alerts, Teams escalations, calendar SLA deadlines, SharePoint evidence uploads |
| Approval | CreateDraftMessage + confirmation gates + human disposition step | Banking approvals carry regulatory accountability; disposition decisions remain human-only |
| Evaluation | Quality rubric scoring + process-level banking metrics | Component eval via trigger analysis and guardrail compliance testing; process eval via triage cycle time, classification accuracy, and zero-tolerance metrics |
