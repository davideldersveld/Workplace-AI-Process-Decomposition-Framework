# Plan: Field Service Work-Order Triage and Dispatch Readiness — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Field Service line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Work-Order Triage and Dispatch Readiness pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Field Service, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Work-order triage scores high on volume and business value. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's narrow-scope principle |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call | Requires significant translation — Field Service references "field service platform", "scheduling engine", "inventory/parts system", and "asset history system" — all external to M365 |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md | Strong — Field Service boundaries are well-defined because safety, schedule integrity, and customer appointment commitments create clear control points |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — context assembly and readiness checking are strong fits for Data Aggregation and Decision Support templates |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — Cowork provides layers 1-5 and 7-9 natively; skill author controls capability definition and decision logic |

### Key Insight

Field Service work-order triage is **schedule-sensitive and dependency-heavy**. The framework's decomposition produces skills that naturally split along the intake-context-classify-prioritize-assign-communicate axis. The primary Cowork design challenge is **multi-system dependency tracking**: dispatch readiness depends on technician availability (scheduling engine), parts availability (inventory system), asset history (asset management system), and site access requirements — all of which live outside M365. The secondary challenge is **safety-critical guardrails**: incorrect dispatch (wrong technician, missing parts, unsafe site conditions) has physical-world consequences that make Field Service guardrails stricter than in purely digital workflows like Customer Service or HR.

---

## Part 2: The Field Service Plugin — Skill-by-Skill Design

The Field Service sample decomposes "Work-Order Triage and Dispatch Readiness" into 7 steps (FS-001 through FS-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| FS-001: Normalize work-order event | `fs-workorder-intake` | Data Aggregation | Deterministic automation | Excel (work-order tracker), SharePoint (list), Outlook (work-order notification) |
| FS-002: Gather asset, location, technician, and parts context | `fs-dispatch-packet` | Data Aggregation | AI act within policy | Word (dispatch context packet), SharePoint (asset docs, SOPs), Graph API (people) |
| FS-003: Classify work type and likely blockers | `fs-blocker-classifier` | Decision Support | AI assist | Adaptive Card (classification and blocker report), Excel (tracker update) |
| FS-004: Assess urgency and dispatch path | `fs-urgency-assessment` | Decision Support | AI draft + approve | Adaptive Card (urgency report), Excel (priority update) |
| FS-005: Assign queue or technician path | `fs-dispatch-routing` | Decision Support | AI act within policy | Teams (routing messages), Excel (assignment update), Calendar (availability check) |
| FS-006: Draft customer or technician summary | `fs-dispatch-comms` | Content Generation | AI draft + approve | Outlook (customer update draft), Teams (technician briefing), Word (dispatch summary) |
| FS-007: Confirm dispatch readiness | *Not a skill — human approval step* | N/A | Human only | Calendar (dispatch review), Outlook (dispatch confirmation) |

### Detailed Skill Designs

#### 1. `fs-workorder-intake` — Normalize Work-Order Event

**Framework Step:** FS-001

**Trigger phrases:** "new work order", "log work order for [customer]", "service request from [site]", "intake work order [ID]", "new field service case"

**Inputs:**
- Customer name, site address, or account identifier
- Work-order type (installation, repair, maintenance, inspection)
- Issue description or service request details
- Requested appointment window
- Any attached files (site photos, equipment manuals, forms)

**M365 tools:**
- `SearchM365(sources=["email"])` — find the inbound work-order email or service request
- `SearchPeople` — resolve customer contact and internal account owner
- `GetUserDetails` — pull internal account owner and dispatch coordinator profiles
- `ReadFileContent` — extract content from attached service request forms in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` — pull field service platform work-order record if Graph Connector is configured

**Output:** Structured work-order record written to Excel work-order tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Work Order ID, Customer Name, Account ID, Site Address, Work Type, Issue Summary, Requested Window, Parts Required (pending), Technician Assigned (pending), Priority (pending), Status, Created Date, SLA Deadline (30 min triage), Dispatch Ready, Blockers, Resolution

**Guardrails:**
- Never create duplicate work orders for the same customer site and requested date
- Validate that site address and requested appointment window are populated before writing
- Confirm details with user before writing to tracker
- Automatically calculate SLA deadline (30 minutes from creation for standard work orders)
- Flag if work-order type involves safety-sensitive equipment (electrical, gas, confined space)

---

#### 2. `fs-dispatch-packet` — Gather Asset, Location, Technician, and Parts Context

**Framework Step:** FS-002

**Trigger phrases:** "build dispatch packet for work order [ID]", "pull context for this job", "assemble dispatch readiness context", "what do we need for this work order"

**Inputs:**
- Work-order ID or customer site identifier from the tracker

**M365 tools:**
- `ReadFileContent` — read work-order tracker row from SharePoint Excel workbook
- `SearchM365(sources=["files"])` — asset history documents, site access guides, service playbooks, equipment manuals in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` — asset records, prior visit history, equipment configuration
- `SearchM365(sources=["connectors"], connector_ids=["inventory-connector"])` — parts availability and location
- `SearchM365(sources=["email"])` — prior correspondence about this site or equipment
- `ListChatMessages` — prior Teams discussions about this customer site or equipment
- `GetDriveChildren` — list files in the site-specific evidence folder (prior service reports, site photos)
- `GetUserDetails` — dispatch coordinator, technician lead, and customer liaison profiles

**Output:** Word document containing:
- Work-order summary (ID, customer, site, work type, issue description)
- Asset profile (equipment type, model, serial number, warranty status, last service date)
- Site details (address, access requirements, safety notes, site contact)
- Prior visit history (last 5 visits with outcomes and technician notes)
- Parts assessment (required parts, availability status, warehouse location)
- Technician requirements (required skills, certifications, tooling)
- Scheduling context (requested window, site operating hours, access restrictions)
- Key contacts (customer, site contact, dispatch coordinator, technician lead)

**Artifact:** Word (.docx) saved to SharePoint work-order folder

**Guardrails:**
- Cite source and retrieval date for every data point from field service platform or asset system
- Flag if critical context is missing: no asset history, no site access notes, no parts availability
- Flag if asset is under active recall or has safety advisories
- Flag if prior visits to this site had safety incidents or access issues
- Operate within read-only boundaries — never update field service platform or inventory records

---

#### 3. `fs-blocker-classifier` — Classify Work Type and Likely Blockers

**Framework Step:** FS-003

**Trigger phrases:** "classify this work order", "check for blockers on [ID]", "what could block this dispatch", "triage readiness for work order"

**Inputs:**
- Work-order data from Excel tracker
- Dispatch context packet (Word document)
- Work-type taxonomy and readiness checklist from SharePoint

**M365 tools:**
- `ReadFileContent` — work-order tracker and dispatch packet from SharePoint
- `SearchM365(sources=["files"])` — work-type taxonomy, readiness checklist, skill matrix from SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` — prior work orders for same site or equipment type for pattern matching
- `SearchM365(sources=["connectors"], connector_ids=["inventory-connector"])` — current parts availability for required components

**Output:** Classification and blocker report as Adaptive Card containing:
- Confirmed work type (from taxonomy)
- Readiness checklist status (green/yellow/red for each prerequisite)
- Identified blockers with severity:
  - Parts blockers (required parts unavailable or backordered)
  - Skill blockers (no available technician with required certification)
  - Access blockers (site access not confirmed, safety clearance pending)
  - Schedule blockers (requested window conflicts with technician availability)
- Suggested resolution for each blocker
- Overall dispatch readiness score (Ready, Blocked, Needs Review)

**Logic:** Compare work-order requirements against the readiness checklist. Cross-reference parts availability, technician skill matrix, and site access requirements. Flag any prerequisite that is not met.

**Guardrails:**
- Present classification and blockers as recommendation only — never auto-clear a blocker
- Always show readiness checklist with explicit status for each item
- Flag safety-related blockers (missing certifications, unsafe site conditions, equipment recalls) with elevated visibility
- Surface prior visit issues for this site or equipment to help dispatchers anticipate problems
- Never mark a work order as dispatch-ready if any red blocker exists

---

#### 4. `fs-urgency-assessment` — Assess Urgency and Dispatch Path

**Framework Step:** FS-004

**Trigger phrases:** "assess urgency for work order [ID]", "what priority is this job", "check dispatch path", "is this an emergency dispatch"

**Inputs:**
- Classification and blocker output
- Dispatch context packet (customer tier, appointment commitment, asset criticality)
- Dispatch policy and priority rubric from SharePoint

**M365 tools:**
- `ReadFileContent` — dispatch policy, priority rubric, and escalation criteria from SharePoint
- `ReadFileContent` — work-order tracker and dispatch packet
- `SearchM365(sources=["files"])` — appointment SLA documents, customer contract terms
- `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` — customer tier, active contract terms, appointment history

**Output:** Urgency assessment as Adaptive Card containing:
- Recommended priority level (Emergency, High, Standard, Scheduled)
- Appointment impact assessment (will the customer-committed window be met?)
- Dispatch path recommendation (immediate dispatch, next-available slot, scheduled maintenance window)
- Escalation conditions met (if any: safety issue, VIP customer, repeated failure)
- Blocker resolution urgency (which blockers need immediate attention vs. can wait)
- Recommended technician skill level

**Guardrails:**
- Present urgency as draft recommendation — require explicit user confirmation before applying
- Never downgrade urgency on a work order with a safety flag without dispatch lead approval
- Flag any work order where the requested appointment window will be missed given current blockers
- Cross-reference with repeat-visit data: if this is a return visit for the same issue, escalate urgency
- Include SLA countdown in the assessment output
- Emergency dispatch recommendations always require dispatch lead confirmation

---

#### 5. `fs-dispatch-routing` — Assign Queue or Technician Path

**Framework Step:** FS-005

**Trigger phrases:** "assign technician for work order [ID]", "route to dispatch queue", "who should handle this job", "dispatch this work order"

**Inputs:**
- Classification, blocker, and urgency outputs
- Skill matrix and routing rules from SharePoint
- Technician availability

**M365 tools:**
- `ReadFileContent` — skill matrix, routing rules, and region assignment map from SharePoint
- `SearchPeople` — resolve technicians by skill and region
- `GetDirectReportsDetails` — technician team structure under dispatch lead
- `ListCalendarView` — check technician availability and existing job schedule
- `PostMessage` — Teams notification to assigned technician with job briefing
- `PostChannelMessage` — post work-order summary to dispatch channel

**Output:** Routed work order:
- Teams message to assigned technician with work-order summary, dispatch packet link, parts list, site address, customer contact, and appointment window
- Channel post to dispatch channel with work-order card
- Updated Excel tracker with technician assignment, dispatch queue, and routed timestamp

**Guardrails:**
- Assign only to technicians who meet the skill and certification requirements from the work-type classification
- Verify technician availability via calendar before assignment; flag scheduling conflicts
- Never auto-assign emergency or safety-flagged work orders — present recommendation for dispatch lead review
- Include parts pickup instructions and site access notes in every technician notification
- Check that assigned technician is in the correct service region
- Log routing decision with rationale in the work-order tracker
- Never route a work order that has unresolved red blockers — flag for blocker resolution first

---

#### 6. `fs-dispatch-comms` — Draft Customer or Technician Summary

**Framework Step:** FS-006

**Trigger phrases:** "draft customer update for work order [ID]", "prepare technician briefing", "write dispatch summary", "send appointment confirmation"

**Inputs:**
- Work-order data from Excel tracker
- Dispatch context packet (Word document)
- Classification, blocker, and urgency outputs
- Target audience (customer, technician, dispatch team, escalation manager)

**M365 tools:**
- `CreateDraftMessage` — Outlook draft for customer appointment confirmation or update (never auto-send)
- `PostMessage` — Teams briefing to technician with job details
- `SearchM365(sources=["files"])` — communication templates and customer notification templates from SharePoint
- `ReadFileContent` — approved templates for appointment confirmation, reschedule notice, and dispatch summary
- `SearchM365(sources=["email"])` — recent customer correspondence for context

**Communication templates:**
- Customer appointment confirmation (date, time window, technician name, preparation instructions)
- Customer reschedule notification (reason, new proposed window, apology language)
- Technician dispatch briefing (Word document with: job summary, asset details, parts list, site access, safety notes, customer expectations, prior visit notes)
- Dispatch team escalation notification (blocked work order requiring management intervention)
- Post-visit follow-up request to customer

**Guardrails:**
- Always create customer-facing communications as Outlook draft — never send without explicit user confirmation
- Never include internal priority classifications, technician personal details beyond name, or dispatch routing info in customer communications
- Never make commitments about outcome or cost in customer appointment confirmations — only confirm timing and preparation
- Include safety warnings in technician briefings when the work order involves hazardous equipment or site conditions
- Include parts pickup instructions and special tooling requirements in technician briefings
- Match tone to audience: professional and clear for customer-facing, operational and detailed for technician briefings
- Include work-order reference number in every communication

---

#### Step 7: Confirm Dispatch Readiness (Human Only)

**Framework Step:** FS-007

This is not a Cowork skill. The framework correctly identifies final dispatch-readiness confirmation as a human-only step, particularly for escalations, safety-flagged work orders, and cases with unresolved blockers. In Cowork, it is supported by:

- The blocker report and urgency assessment — provide the evidence package for the dispatch lead
- The `fs-dispatch-comms` skill — send the dispatch confirmation and customer notification after the human decision is made
- Calendar tools — schedule dispatch review slots for complex or safety-sensitive work orders
- The work-order tracker Excel workbook — records the final dispatch readiness decision with timestamp and approver

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. Field Service adds a distinct complexity: it is **schedule-driven, dependency-heavy, and safety-sensitive**, with physical-world consequences for dispatch errors that make it the most guardrail-intensive of the three service LOBs.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The framework's rule — "keep breaking down until each step has one dominant goal" — prevents a monolithic "dispatch work order" skill and produces the focused intake-context-classify-assess-assign-communicate chain. For Field Service, this is critical because blocker detection (FS-003) and urgency assessment (FS-004) are fundamentally different: one is about readiness prerequisites, the other is about business priority.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with dispatch review artifacts and summary outputs |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "check dispatch readiness," the framework requires naming every dependency — parts availability, technician certification, site access, asset history — each translating to specific MCP tool calls or connector queries.

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

**The key gap:** Cowork does not have a durable workflow state engine. Work-order state (pending context, blocked, dispatch-ready, assigned, dispatched) must be externalized to the Excel work-order tracker in SharePoint. Additionally, Cowork does not natively integrate with scheduling engines or inventory systems — these require Graph Connectors or SharePoint bridge patterns.

### 3.3 M365 Artifacts as Process State

For Field Service, the artifact pattern centers on **Excel as the work-order state store**, **Word as the dispatch packet artifact**, and **Teams and Outlook as the coordination and customer communication channels**:

| Artifact | Role in Field Service | How Skills Use It |
|---|---|---|
| **Excel** | Work-order state store (tracker with status, priority, assignment, blockers, dispatch readiness) | The work-order tracker IS the process state — skills read current status, write updates, track blockers and SLA compliance |
| **Word** | Primary dispatch artifact (context packets, technician briefings, dispatch summaries) | Skills generate dispatch packets that become the official technician briefing — must contain asset details, parts list, site access, and safety notes |
| **SharePoint** | Source of truth for policies, SOPs, asset documents, and templates; also stores site photos, service reports, and equipment manuals | Skills read skill matrices, dispatch policies, readiness checklists, and safety guidelines; work-order folders store all case artifacts |
| **Outlook** | Customer communication channel | Skills draft appointment confirmations, reschedule notices, and follow-up requests as reviewable drafts |
| **Teams** | Internal dispatch coordination (technician notifications, dispatch channel posts, escalation notices) | Skills post technician briefings, blocker alerts, and routing notifications to dispatch channels and technician chats |
| **Adaptive Card** | Real-time decision support (blocker reports, urgency assessments, readiness scorecards) | Skills present readiness assessments and blocker reports for dispatcher review before any routing action |
| **Graph API** | People and org data (technician profiles, team structure, skill certifications) | Skills resolve technicians by skill, check certifications, and determine escalation paths |
| **Calendar** | Technician availability and appointment scheduling | Skills check technician calendars before assignment; manage appointment windows for customer visits |

**The design pattern:** The Cowork Field Service plugin uses a SharePoint-hosted Excel workbook as the canonical work-order tracker, with Word documents as the technician dispatch briefing, SharePoint folders for site-specific documentation and evidence, Teams as the dispatch coordination channel, and Outlook as the customer communication channel. The blocker classification Adaptive Card is the key decision-support surface — dispatch coordinators need to see readiness status at a glance before approving dispatch.

### 3.4 Federated Connectors for Third-Party Systems

Field Service depends heavily on external systems: field service platforms (Dynamics 365 Field Service, ServiceMax, SAP Field Service Management), scheduling engines (IFS, ClickSoftware), inventory/parts systems (SAP, Oracle), and asset management/IoT platforms (Azure IoT, PTC ThingWorx). The federated access approach follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For field service platforms (Dynamics 365 Field Service, ServiceMax) and inventory systems, Graph Connectors index work-order records, asset histories, and parts availability into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["fsp-connector"])` and `SearchM365(sources=["connectors"], connector_ids=["inventory-connector"])`. This provides read access to work-order state, asset profiles, and parts status without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, synchronized data is maintained in SharePoint:
- Technician skill matrix (Excel in SharePoint, maintained by dispatch operations)
- Parts availability lookup (SharePoint list, synced from inventory system via Power Automate)
- Asset history and warranty status (SharePoint list, synced from asset management system)
- Site access requirements and safety notes (SharePoint document library, maintained by field operations)
- Service playbooks and equipment manuals (SharePoint document library)

**Tier 3 — Manual Input with Templates**

For real-time scheduling data, IoT telemetry, and systems with no integration path, skills provide structured intake that captures data from manual lookups. The dispatch coordinator enters technician availability, parts confirmation, and site access status into the work-order tracker. The `fs-dispatch-packet` skill then synthesizes what is available.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for skill matrix, parts availability, and asset history, and Tier 3 (manual input) for scheduling and IoT data. Graph Connectors for the field service platform and inventory system are best introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

Field Service governance is shaped by three specific concerns: **safety** (incorrect dispatch can create hazardous conditions), **schedule integrity** (missed appointments erode customer trust and incur SLA penalties), and **parts and skills compliance** (dispatching a technician without the right parts, certifications, or tooling wastes a visit).

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure, skill matrix, and escalation paths |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC; field service platform data accessed through permissioned connectors |
| **Safety controls** | Skills flag safety-sensitive work types (electrical, gas, confined space) with elevated visibility; never auto-dispatch safety-flagged work orders; require dispatch lead confirmation |
| **Schedule integrity** | Appointment windows tracked in work-order tracker; skills check technician calendar availability before assignment; flag conflicts proactively |
| **Parts and skills compliance** | Skills cross-reference technician certifications and parts availability against work-type requirements; never route a work order to a technician who lacks required certification |
| **Customer communication** | All customer-facing drafts require human review; no outcome or cost commitments in appointment confirmations |
| **Audit** | Platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; work-order tracker records every state change with timestamp |
| **Release management** | Skills versioned in OneDrive; quality rubric scoring provides pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions; dynamic policy content (dispatch rules, skill matrix, safety guidelines) read from SharePoint at runtime |

**The main governance gap** is real-time scheduling integration. Cowork cannot directly query the scheduling engine to check technician availability with precision — it can check Calendar for blocked time, but not for route optimization or travel time. The mitigation is to use the SharePoint bridge (technician availability synced from scheduling engine) for Wave 1, and introduce Graph Connectors or direct API integration in later waves.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do generated dispatch packets, blocker reports, and technician briefings meet dispatch coordinator expectations? Assessed via dispatch lead review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Work-type classification accuracy — percentage of work orders classified correctly on first pass
- Correct routing rate — percentage of work orders routed to the right technician or queue
- Time to dispatch-ready state — measured from work-order creation to dispatch-ready status
- Blocker detection rate — percentage of actual blockers identified before dispatch
- First-time-fix impact proxy — percentage of dispatched work orders completed without a return visit (compared to pre-pilot baseline)
- Appointment reschedule rate due to missing preparation — percentage of dispatches that fail due to missing parts, wrong technician skills, or access issues
- Draft acceptance rate — percentage of customer communications sent without major edits
- Override rate — how often dispatchers override skill recommendations
- Safety incident rate — any incident linked to a work order processed through the skill suite (target: zero)

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel work-order tracker workbook in SharePoint with standard columns (Work Order ID, Customer Name, Account ID, Site Address, Work Type, Issue Summary, Requested Window, Parts Required, Technician Assigned, Priority, Status, SLA Deadline, Dispatch Ready, Blockers, Safety Flags, Resolution, Last Updated, Updated By)
- Upload dispatch policies, skill matrix, readiness checklists, safety guidelines, and communication templates to a dedicated SharePoint document library
- Configure SharePoint bridge data: technician skill matrix, parts availability lookup, asset history, site access notes
- Create a SharePoint folder structure for per-work-order documents (service reports, site photos, equipment manuals)

**Skills to build:**
- `fs-workorder-intake`
- `fs-dispatch-packet`
- `fs-blocker-classifier`
- `fs-dispatch-comms`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 10-15 real work orders across multiple work types (installation, repair, maintenance) and regions.

### Wave 2 — Routing and Urgency

**Skills to build:**
- `fs-urgency-assessment`
- `fs-dispatch-routing`

**Promotions:**
- Promote `fs-workorder-intake` to write mode (creates work-order records after confirmation)
- Promote `fs-blocker-classifier` to write-back mode (updates Excel tracker blockers and readiness status after user confirmation)
- Promote `fs-dispatch-routing` to bounded write mode (posts routing messages to Teams dispatch channels and technician chats within approved skill matrix and region rules)

**Integrations:**
- Introduce Graph Connectors for field service platform and inventory system if available at the tenant level
- Add parts availability checking: `fs-blocker-classifier` cross-references parts requirements against inventory connector

**Automation:**
- Set up a scheduled prompt (every 30 minutes) that checks the work-order tracker for work orders approaching the 30-minute triage SLA or with unresolved blockers near appointment windows, surfacing warnings via Teams notifications to dispatch lead
- Set up a daily scheduled prompt that summarizes work-order volume, average triage time, blocker frequency by type, and dispatch readiness rate

**Operating posture:** AI draft plus approve for urgency assessment and customer-facing communications. AI act within policy for routing within approved skill matrix and region rules. Every safety-flagged work order and customer communication reviewed before action.

### Wave 3 — Optimization and Proactive Detection

**Enhancements:**
- Add bounded multi-source dispatch packet assembly: a skill that aggregates data from field service connector, inventory connector, asset history, email threads, and SharePoint documents into a comprehensive technician briefing
- Add proactive repeat-visit detection: scheduled prompt that analyzes recent completed work orders for patterns indicating likely repeat visits (same site, same equipment, same issue type) and surfaces findings to dispatch operations
- Add parts dependency monitoring: scheduled prompt that checks work orders with upcoming appointments against current parts availability and flags potential shortages before dispatch
- Add dispatch-day readiness check: scheduled prompt that runs each morning, reviews all work orders scheduled for the day, and produces a readiness scorecard via Adaptive Card for the dispatch lead
- Refine all skills based on override patterns and dispatcher feedback from Waves 1-2

**Measurement:**
- Work-type classification accuracy target: above 85%
- Correct routing rate target: above 90%
- Time to dispatch-ready state target: under 20 minutes for standard work orders
- Blocker detection rate target: above 80%
- Appointment reschedule rate due to missing prep target: below 5%
- First-time-fix improvement target: measurable improvement over pre-pilot baseline
- Draft acceptance rate target: above 70%
- Safety incident rate target: zero

---

## Part 5: Generalizing the Approach — Field Service Artifact Pattern

Field Service's primary artifact pattern is **Excel (work-order tracking and blocker management)** supported by **Word (dispatch packets and technician briefings)**, **Teams (dispatch coordination)**, and **Calendar (technician scheduling and appointment management)**.

This LOB demonstrates the **schedule-and-dependency variant** of the framework-to-Cowork translation:

1. **The primary process state artifact is the Excel work-order tracker with a strong focus on blocker tracking** — unlike HR (where the tracker tracks a linear progression) or Customer Service (where the tracker tracks case status), Field Service's tracker must track multiple independent dependencies (parts, skills, access, schedule) that all must be satisfied before dispatch. The blocker classification skill's Adaptive Card readiness scorecard is the most-used decision surface.

2. **Calendar is a first-class process artifact, not just a scheduling tool** — Field Service is the only LOB where technician calendar availability is a hard constraint on routing. The `ListCalendarView` tool is used not just for scheduling meetings but for determining whether dispatch is feasible. This elevates Calendar from a supporting role to a primary process dependency.

3. **Safety guardrails are the strictest governance requirement** — while Customer Service worries about unauthorized commitments and Product Support worries about evidence completeness, Field Service must prevent unsafe dispatch. Skills that involve safety-sensitive work types (electrical, gas, confined space) have additional confirmation gates, certification verification, and elevated visibility flags. This pattern applies to any LOB with physical-world consequences (Manufacturing, Facilities, Construction).

4. **Multi-system dependency resolution is the primary data challenge** — Field Service requires more federated data sources than any other LOB in this set: field service platform, scheduling engine, inventory system, asset management, and potentially IoT telemetry. The SharePoint bridge pattern is essential for Wave 1, and Graph Connectors become critical for Wave 2 efficiency.

The decomposition method (one goal per step, boundary-driven guardrails, M365 artifact grounding) works identically for Field Service as it does for HR, Customer Service, and Product Support. The difference is which concerns dominate: HR is document-centric; Customer Service is communication-centric; Product Support is evidence-centric; Field Service is dependency-and-schedule-centric.

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
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via dispatch readiness rate and blocker detection |
| Field Service Platform / Scheduling Engine | Graph Connectors or SharePoint bridge | External system data accessed via `SearchM365(sources=["connectors"])` or synced to SharePoint |
| Asset History / IoT | SharePoint bridge or Graph Connector | Asset data synced to SharePoint lists; IoT telemetry via manual input in Wave 1 |
| Skill Matrix | SharePoint Excel workbook | Technician skills, certifications, and regions maintained in SharePoint and read by routing skills |
| Dispatch Packet | Word document generated by `fs-dispatch-packet` | The primary technician-facing artifact containing all context needed for a successful site visit |
| Safety Flag | Elevated guardrail in skill instructions | Safety-sensitive work types trigger additional confirmation gates and certification verification |
