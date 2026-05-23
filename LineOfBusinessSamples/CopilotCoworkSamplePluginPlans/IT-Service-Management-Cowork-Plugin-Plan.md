# Plan: IT Service Management — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the IT Service Management line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Incident Intake and Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for ITSM, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework uses ITSM-specific terms ("CMDB", "ITSM platform", "monitoring tools") that must become specific M365 tool names plus federated connector references |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but ITSM adds heightened sensitivity around production-impacting actions and major incident escalation that require stricter guardrails than typical LOBs |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — ITSM classification and severity assessment map to Decision Support; context assembly maps to Data Aggregation; user updates map to Content Generation |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — ITSM's tight coupling to the ticketing platform means many write-back operations depend on the federated connector tier |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

ITSM processes are high-volume, time-sensitive, and deeply dependent on external systems (ITSM platform, CMDB, monitoring). The framework's decomposition discipline is especially valuable here because it prevents the common anti-pattern of building a single "triage everything" skill. Instead, each triage phase (normalize, enrich, classify, prioritize, route, communicate) becomes a focused skill with its own guardrail posture. The main translation challenge is that the ITSM platform (ServiceNow, Jira Service Management, etc.) is the canonical system of record, and Cowork must bridge to it through Graph Connectors, SharePoint sync, or manual intake rather than direct API calls.

---

## Part 2: The IT Service Management Plugin — Skill-by-Skill Design

The ITSM sample decomposes "Incident Intake and Triage" into 7 steps (ITSM-001 through ITSM-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| ITSM-001: Normalize incident event | `itsm-incident-intake` | Data Aggregation | Deterministic automation | Excel (incident tracker), SharePoint (list), Teams (intake channel) |
| ITSM-002: Gather user, service, and asset context | `itsm-context-packet` | Data Aggregation | AI act within policy | Word (context packet), SharePoint (CMDB sync), Graph API (people) |
| ITSM-003: Classify incident type and affected service | `itsm-classification-assist` | Decision Support | AI assist | Excel (tracker update), Adaptive Card (classification report) |
| ITSM-004: Assess severity and escalation path | `itsm-severity-recommend` | Decision Support | AI draft + approve | Adaptive Card (severity recommendation), Excel (tracker update) |
| ITSM-005: Assign resolver group and next action | `itsm-assignment-router` | Decision Support | AI act within policy | Teams (messages), Excel (tracker update), Calendar (SLA reminders) |
| ITSM-006: Draft user update and handoff summary | `itsm-comms-drafter` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (handoff summary) |
| ITSM-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Teams (disposition confirmation), Calendar (review meeting) |

### Detailed Skill Designs

#### 1. `itsm-incident-intake` — Normalize Incident Event

**Framework Step:** ITSM-001

**Trigger phrases:** "new incident from", "log incident for", "ticket came in for", "create incident case", "intake this incident"

**Inputs:**
- Incident description (from email, Teams message, or monitoring alert)
- Affected user name or email
- Reported channel (email, chat, phone, monitoring)
- Initial symptom description

**M365 tools:**
- `SearchPeople` — resolve affected user identity
- `GetUserDetails` — pull user profile, department, location
- `SearchM365(sources=["email"])` — find the original report thread if submitted via email
- `SearchM365(sources=["teams"])` — find escalation messages in service desk channel
- `ReadFileContent` — read SharePoint-hosted intake form template for field mapping
- `GetDriveChildren` — check existing incident tracker for duplicates

**Output:** Structured incident record written to Excel incident tracker in SharePoint; confirmation via Adaptive Card showing normalized fields

**Artifact:** Excel workbook with columns: Incident ID, Reported By, Report Channel, Summary, Affected Service, Status, Priority (pending), Created Date, Assigned To (pending), SLA Target

**Guardrails:**
- Never create duplicate incidents for the same user and symptom within a 4-hour window
- Validate that required fields (reporter, symptom description, affected service area) are present before writing
- Auto-generate Incident ID using date-based sequence pattern
- Confirm details with user via Adaptive Card before writing to tracker
- Scheduled prompt variant: monitor the service desk Teams channel and intake email alias for unprocessed incidents every 15 minutes

---

#### 2. `itsm-context-packet` — Gather User, Service, and Asset Context

**Framework Step:** ITSM-002

**Trigger phrases:** "build context for incident", "enrich this incident", "get context for ticket", "what do we know about this incident"

**Inputs:**
- Incident ID or affected user name from intake record

**M365 tools:**
- `GetUserDetails` — affected user profile, department, office
- `GetManagerDetails` — reporting chain for escalation path
- `SearchM365(sources=["files"])` — CMDB sync documents, service ownership maps, asset inventories in SharePoint
- `ReadFileContent` — SharePoint-hosted service catalog, recent change log, and asset configuration data
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — pull related tickets and CI data from ITSM platform via Graph Connector (if available)
- `SearchM365(sources=["email"])` — recent emails from affected user mentioning the service or symptoms
- `SearchM365(sources=["teams"])` — related discussions in IT channels

**Output:** Word document containing:
- Affected user profile and org context
- Service and asset details from CMDB sync
- Recent changes to the affected service or asset
- Related open incidents or known issues
- Prior incident history for this user or service

**Artifact:** Word (.docx) saved to SharePoint incident folder; linked from the Excel tracker row

**Guardrails:**
- Only retrieve context for services and assets the requesting analyst has permission to view
- Cite source document and retrieval timestamp for every data point
- Flag if CMDB sync data is more than 24 hours stale
- Do not include raw security logs or identity credential details in the context packet
- If Graph Connector is unavailable, note which context fields are missing and suggest manual lookup

---

#### 3. `itsm-classification-assist` — Classify Incident Type and Affected Service

**Framework Step:** ITSM-003

**Trigger phrases:** "classify this incident", "what type of incident is this", "categorize ticket", "triage classification for"

**Inputs:**
- Context packet (Word document or data from Excel tracker)
- Incident taxonomy (SharePoint document)

**M365 tools:**
- `ReadFileContent` — incident taxonomy and service catalog from SharePoint
- `SearchM365(sources=["files"])` — similar past incidents and their classifications
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — historical classification patterns from ITSM platform

**Output:** Classification recommendation as Adaptive Card showing:
- Recommended category and subcategory
- Affected service (with confidence level)
- Similar past incidents with their resolutions
- Classification confidence score (high, medium, low)

**Logic:** Compare symptom description and context against the incident taxonomy. Cross-reference with similar past incidents. Present top 3 classification candidates ranked by confidence.

**Guardrails:**
- Present classification as a recommendation only — never auto-update the incident category without analyst confirmation
- Always show the confidence level and basis for the recommendation
- If confidence is low (below 60%), explicitly flag for manual classification
- Never classify an incident as "major" through this skill — major incident determination requires the severity assessment step
- Show the top 3 candidates, not just the top pick, so the analyst can compare

---

#### 4. `itsm-severity-recommend` — Assess Severity and Escalation Path

**Framework Step:** ITSM-004

**Trigger phrases:** "assess severity for", "what priority should this be", "escalation check for incident", "severity recommendation for"

**Inputs:**
- Classification output
- Context packet
- Severity policy matrix (SharePoint document)

**M365 tools:**
- `ReadFileContent` — severity matrix, escalation policy, and SLA targets from SharePoint
- `SearchM365(sources=["files"])` — service criticality ratings, business impact assessments
- `GetUserDetails` — affected user's role and VIP status
- `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])` — current open major incidents that might be related

**Output:** Severity recommendation as Adaptive Card showing:
- Recommended priority level (P1-P4) with rationale
- Impact assessment (number of affected users, service criticality)
- Urgency assessment (business impact timeline)
- Escalation path recommendation (standard queue vs. major incident bridge)
- SLA target for the recommended priority

**Guardrails:**
- Never auto-set severity — always present as a recommendation for analyst review
- If the recommendation is P1 or P2, add an explicit major incident trigger warning with instructions for the analyst
- Require the analyst to confirm or override before the severity is written to the tracker
- Log the recommended severity and the analyst's final decision for override tracking
- If the incident involves executive users or revenue-critical services, auto-flag for escalation review regardless of calculated severity
- Cross-reference against open major incidents to identify potential correlation

---

#### 5. `itsm-assignment-router` — Assign Resolver Group and Next Action

**Framework Step:** ITSM-005

**Trigger phrases:** "route this incident", "assign resolver group", "who handles this type of ticket", "route to the right team"

**Inputs:**
- Classification and severity output
- Assignment rules matrix (SharePoint document)
- Support model and resolver group directory (SharePoint)

**M365 tools:**
- `ReadFileContent` — assignment rules, support model, and resolver group directory from SharePoint
- `SearchPeople` — resolve resolver group leads and on-call contacts
- `GetUserDetails` — verify resolver group lead availability
- `PostMessage` — Teams notification to the assigned resolver group
- `PostChannelMessage` — post assignment summary to the incident management Teams channel
- `CreateEvent` — SLA reminder on the resolver group lead's calendar if high priority
- `ListCalendarView` — check resolver availability before assignment

**Output:** Routed incident:
- Teams message to assigned resolver group with incident summary
- Channel post in the incident management channel for visibility
- Calendar event for SLA deadline reminder (P1/P2 only)
- Updated Excel tracker with assigned group, assignee, and SLA target

**Guardrails:**
- Only assign to resolver groups listed in the approved assignment rules matrix
- If no matching rule exists, escalate to the service operations manager rather than guessing
- For P1/P2 incidents, verify that the assigned group has an active on-call contact before routing
- Present assignment recommendation for analyst review before sending Teams messages (unless operating in "act within policy" mode for standard assignments)
- Never reassign a major incident without explicit analyst confirmation
- Include the incident ID and priority in every outbound Teams message for traceability

---

#### 6. `itsm-comms-drafter` — Draft User Update and Handoff Summary

**Framework Step:** ITSM-006

**Trigger phrases:** "draft user update for", "write handoff summary", "prepare incident update email", "notify the user about their ticket", "incident status update for"

**Inputs:**
- Incident record from Excel tracker
- Context packet (Word document)
- Classification and severity data
- Assignment details
- Target audience (end user, resolver group, incident manager, executive)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts for user-facing updates (never auto-send)
- `PostMessage` — Teams coordination messages to resolver groups
- `SearchM365(sources=["files"])` — communication templates from SharePoint
- `ReadFileContent` — incident communication templates and tone guides

**Communication templates:**
- User acknowledgment (incident received, assigned, expected timeline)
- Status update to affected user (progress, next steps)
- Internal handoff summary for resolver group
- Escalation notice for incident manager
- Major incident bridge invite and summary (P1/P2 only)

**Guardrails:**
- Always create user-facing communications as Outlook drafts — never send without explicit analyst confirmation
- Match tone to audience: empathetic and clear for end users, operational and precise for resolver groups, executive summary format for leadership
- Include incident ID, current status, and next expected action in every communication
- Never include internal system names, queue names, or technical identifiers in user-facing updates
- Redact any IP addresses, hostnames, or infrastructure details from user-facing messages
- For major incidents, include the bridge call details but never share the incident's security classification externally
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable before delivery

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** ITSM-007

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step, especially for major incident confirmation and escalation decisions. In Cowork, it is supported by:

- The severity recommendation Adaptive Card from `itsm-severity-recommend` — provides the evidence for the disposition decision
- The assignment confirmation from `itsm-assignment-router` — confirms the incident is in the right queue
- The `itsm-comms-drafter` skill — sends the confirmation communications after the human decision is made
- Calendar events for review checkpoints — the analyst or incident manager confirms the final triage state

For major incidents, the human disposition step includes: confirming the major incident declaration, authorizing the incident bridge, and signing off on the containment plan. These are production-impacting decisions that must remain human-owned.

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. For ITSM, this challenge is amplified by the heavy dependency on the ITSM platform (ServiceNow, Jira Service Management, BMC) as the canonical system of record. This section analyzes how to approach that translation systematically.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in ITSM is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern in ITSM would be building one "triage this incident" skill that tries to normalize, classify, prioritize, route, and communicate in a single invocation. The framework's rule — "keep breaking down until each step has one dominant goal" — produces well-scoped skills where classification is separate from severity assessment, which is separate from routing.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support the disposition step with Adaptive Cards and summary artifacts |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions (used for classification) |
| AI draft + approve | Skill produces recommendations but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation (used for severity and communications) |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel, route to resolver group) within defined assignment rules |
| Deterministic automation | Scheduled prompt or rule-based skill for stable intake normalization |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "check the CMDB," the framework requires naming every input source. This translates to specific MCP tool calls and connector references in the SKILL.md instructions.

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
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through the quality rubric and manual testing |

**The key gap for ITSM:** Cowork does not have a durable workflow state engine, and the ITSM platform (ServiceNow, etc.) is the real system of record. The Excel tracker in SharePoint serves as the Cowork-accessible process state, but it is a synchronized copy, not the source of truth. This dual-state reality must be acknowledged in every skill's guardrails.

### 3.3 M365 Artifacts as Process State

For ITSM, the primary M365 artifacts are **Teams** and **Excel**, reflecting the real-time coordination nature of incident work and the need for structured tracking.

| Artifact | Role in ITSM | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (incident tracker, SLA dashboard, classification log) | The incident tracker workbook IS the Cowork-accessible process state — skills read current status, write updates, track SLA compliance |
| **Teams** | Real-time coordination and routing (resolver group notifications, incident bridge, status updates) | Skills post assignments, escalation notices, and status updates to Teams channels and chats; the incident management channel serves as the operational nerve center |
| **Word** | Evidence artifacts (context packets, handoff summaries) | Skills generate context documents that become the reference for resolver groups and incident reviews |
| **Outlook** | Communication channel for user-facing updates | Skills draft user acknowledgments, status updates, and resolution confirmations as reviewable Outlook drafts |
| **SharePoint** | Source of truth for policies and reference data (incident taxonomy, severity matrix, assignment rules, service catalog, CMDB sync) | Skills read classification rules, severity policies, and resolver directories from SharePoint; these are the policy documents that govern skill behavior |
| **Calendar** | SLA deadline reminders and review checkpoints | Skills create calendar events for SLA breach warnings and review meetings |
| **Graph API** | People and org data (user profiles, resolver group leads, on-call contacts) | Skills resolve affected users, managers, and resolver group contacts for routing and escalation |
| **Adaptive Card** | In-flow decision support (classification recommendations, severity assessments) | Skills present triage recommendations as structured Adaptive Cards for quick analyst review without switching applications |

**The design pattern:** The ITSM plugin uses a SharePoint-hosted Excel workbook as the Cowork-accessible incident tracker, with Teams as the primary coordination channel for real-time routing and escalation. Word documents serve as evidence packages for handoff and review. This is a coordination-heavy pattern — unlike HR (which is document-heavy) or Finance (which is spreadsheet-heavy), ITSM is fundamentally about real-time message routing with structured tracking as the backbone.

### 3.4 Federated Connectors for Third-Party Systems

ITSM is the LOB most dependent on external systems. The framework references "ITSM platform," "CMDB," "monitoring tools," and "identity platform" — none of which exist natively in M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For ITSM platforms (ServiceNow, Jira Service Management, BMC Helix), Graph Connectors index ticket records, CI data, and knowledge articles into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["servicenow-connector"])`. This provides read access to incident history, CMDB records, and known error databases without custom integration code.

For monitoring platforms (Datadog, PagerDuty, Azure Monitor), Graph Connectors can index active alerts and recent incidents, giving skills visibility into the monitoring context that triggered the incident.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- **CMDB sync**: A Power Automate flow exports service catalog, CI records, and asset data to SharePoint lists or Excel workbooks on a scheduled basis. Skills read this synchronized copy via `ReadFileContent` or `GetDriveChildren`.
- **Incident sync**: Active incidents from the ITSM platform are synced to the SharePoint Excel tracker. Bidirectional sync (writing triage results back to ServiceNow) is handled by Power Automate outside of Cowork.
- **Knowledge base**: Knowledge articles exported to SharePoint documents, searchable via `SearchM365(sources=["files"])`.

**Tier 3 — Manual Input with Templates**

For systems with no integration path, skills provide structured intake:
- The `itsm-incident-intake` skill prompts the analyst for fields that would normally come from the monitoring system
- The `itsm-context-packet` skill notes which CMDB fields are unavailable and prompts for manual entry
- All manual inputs are written to the Excel tracker with a "manual entry" source tag for audit

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint sync for CMDB and service catalog) and Tier 3 (manual intake for monitoring context). Graph Connectors for ServiceNow or Jira require tenant admin configuration and should be introduced in Wave 2 after the skill workflows are proven with the SharePoint bridge.

### 3.5 Governance in Cowork

ITSM governance has specific concerns around production impact, change control, and SLA accountability:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; the IT service operations manager owns the skill suite; resolver group leads own assignment rules |
| **Access** | M365 permissions govern data access; incident data may include identity or security-sensitive details requiring scoped visibility |
| **Data classification** | Skill guardrails enforce sensitivity rules: no internal infrastructure details in user-facing comms, no security-classified data in general channels, no PII exposure in Teams posts |
| **Audit** | Every skill invocation produces artifacts (Excel updates, Teams messages, Outlook drafts) that form a traceable record; the framework requires actor, timestamp, prior value, and ticket linkage for every write |
| **Change control** | Incident taxonomy, severity matrix, and assignment rules live in SharePoint documents that can be updated by ITSM process owners without modifying skill code; changes follow existing ITSM change management processes |
| **SLA accountability** | Skills track SLA targets in the Excel tracker; the scheduled prompt monitors for approaching breaches; but SLA enforcement remains the responsibility of the ITSM platform |
| **Major incident discipline** | Skills never auto-declare major incidents, never auto-authorize containment, and never auto-escalate to executive leadership; these remain human-owned decisions with skill-provided evidence |

**The main governance gap** is bidirectional state synchronization. If an analyst updates the incident in ServiceNow but not in the Excel tracker (or vice versa), state drift occurs. The recommended mitigation is to treat the ITSM platform as the source of truth and the Excel tracker as a working copy, with Power Automate flows handling synchronization on a 15-minute cycle.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for ITSM:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis
- Classification accuracy — does `itsm-classification-assist` recommend the correct category? Assessed via comparison against analyst final classification
- Severity agreement rate — does `itsm-severity-recommend` match the analyst's final severity? Measured as percentage of recommendations accepted without override
- Tool success rate — do M365 tool calls and connector queries return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Incident classification accuracy — percentage of incidents correctly classified on first attempt
- Correct assignment rate — percentage of incidents routed to the right resolver group without reassignment
- Time to triage — elapsed time from incident creation to classified, prioritized, and assigned state
- Major incident trigger miss rate — percentage of actual major incidents not flagged by the severity skill (critical safety metric)
- SLA breach rate — percentage of incidents exceeding their SLA target
- Override rate — how often analysts override skill recommendations (high override rate signals skill quality issues)
- Unauthorized action incidents — any cases where the skill took an action outside its permitted boundary (should be zero)

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for ITSM in Cowork:

### Wave 1 — Foundation and Read-Only Assist

**Infrastructure setup:**
- Create the shared Excel incident tracker workbook in SharePoint with standard columns (Incident ID, Reported By, Report Channel, Summary, Affected Service, Category, Priority, Status, Created Date, Assigned To, SLA Target, Resolution Notes)
- Upload incident taxonomy, severity matrix, assignment rules, and service catalog to a dedicated SharePoint document library
- Set up a SharePoint folder for CMDB sync data (service ownership, asset inventory, recent change log)
- Create a Power Automate flow to sync active incident data from the ITSM platform to the Excel tracker (if feasible in Wave 1)
- Establish the incident management Teams channel for routing and coordination

**Skills to build:**
- `itsm-incident-intake` — normalize incoming incidents into the tracker
- `itsm-context-packet` — assemble user, service, and asset context
- `itsm-classification-assist` — present classification recommendations
- `itsm-comms-drafter` — draft user updates and handoff summaries

**Operating posture:** AI assist mode for classification; deterministic for intake; AI draft plus approve for communications. All outputs are presented via Adaptive Card or generated documents for analyst review. No automated writes to external systems. Test with 20-30 real incidents over 2 weeks.

### Wave 2 — Routing and Severity

**Skills to build:**
- `itsm-severity-recommend` — present severity and escalation recommendations
- `itsm-assignment-router` — route incidents to resolver groups via Teams

**Promotions:**
- Promote `itsm-incident-intake` to write mode (creates tracker records after confirmation)
- Promote `itsm-classification-assist` to write-back mode (updates Excel tracker category after analyst confirmation)
- Promote `itsm-assignment-router` to "act within policy" mode for standard (P3/P4) incidents with clear assignment rule matches

**Automation:**
- Set up a scheduled prompt every 15 minutes to check for new unprocessed incidents in the service desk Teams channel and intake email alias
- Set up a daily scheduled prompt to flag incidents approaching SLA breach

**Operating posture:** AI draft plus approve for severity and P1/P2 routing. AI act within policy for standard P3/P4 routing with clear rule matches. All major incident escalations remain fully human-controlled.

### Wave 3 — Optimization and Proactive Detection

**Enhancements:**
- Introduce Graph Connectors for ServiceNow or Jira Service Management if available at the tenant level
- Add proactive monitoring via scheduled prompt: identify repeat incident patterns, flag likely major incidents based on volume spike or correlated alerts, surface resolver group queue depth imbalances
- Add knowledge-article citation in classification and context packets via `SearchM365(sources=["connectors"], connector_ids=["knowledge-connector"])`
- Refine all skills based on override patterns and analyst feedback from Waves 1-2

**Measurement:**
- Time-to-triage reduction vs. pre-pilot baseline: target 40% improvement
- Classification accuracy target: above 80% agreement with analyst final classification
- Correct assignment rate target: above 85% first-time correct routing
- SLA breach rate target: below 5% for P1/P2 incidents
- Draft acceptance rate target: above 70% for user communications
- Major incident miss rate: target zero (this is a safety-critical metric)

---

## Part 5: Generalizing the Approach — ITSM Artifact Pattern

ITSM's primary artifact pattern is **Teams + Excel** — real-time coordination through Teams messaging combined with structured incident tracking in Excel. This reflects the fundamental nature of incident work: it is time-sensitive, collaborative, and requires both structured data (classification, priority, assignment) and unstructured communication (handoff summaries, user updates, escalation notices).

This pattern fits the cross-LOB method as follows:

| Dimension | ITSM Pattern |
|---|---|
| Primary coordination artifact | Teams (channels, chats, routing messages) |
| Primary tracking artifact | Excel (incident tracker with structured columns) |
| Primary evidence artifact | Word (context packets, handoff summaries) |
| Primary policy artifact | SharePoint documents (taxonomy, severity matrix, assignment rules) |
| Primary communication artifact | Outlook (user-facing drafts) |
| State management approach | Excel tracker as Cowork-accessible state, ITSM platform as source of truth, Power Automate for sync |
| Guardrail posture | Heightened — production-impacting actions, major incident decisions, and containment authorization are always human-owned |

The decomposition and translation method used for HR onboarding applies directly to ITSM. The key difference is the heavier reliance on federated connectors (the ITSM platform is the real system of record) and the stricter guardrail requirements around production-impacting decisions.

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
| Process State | SharePoint-hosted Excel workbook or SharePoint list | Durable state externalized to M365 artifacts; ITSM platform is the canonical source of truth |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, monitoring alert (via connector), calendar event |
| Approval | Draft tools + Adaptive Card confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via classification accuracy, triage time, and SLA compliance |
