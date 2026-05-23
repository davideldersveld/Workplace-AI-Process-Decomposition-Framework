# Plan: Supply Chain Inventory Shortage and Disruption Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Supply Chain line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Inventory Shortage and Disruption Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Supply Chain, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly; supply chain's high-volume exception queues score well |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — separating classification (SC-003) from impact assessment (SC-004) is essential because they are distinct decision types with different data needs |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call | Requires significant translation — Supply Chain references ERP events, planning system data, and transportation platform signals that must become Graph Connector queries or SharePoint bridge reads |
| **Automation Boundary** | Operating mode per step | **Guardrails and confirmation gates** in SKILL.md | Strong — Supply Chain adds allocation control and customer-commitment sensitivity that must be encoded as explicit guardrails; no skill should auto-change inventory allocations |
| **Capability Mapping** | AI task pattern per step | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — classification and impact assessment are Decision Support; context assembly is Data Aggregation; summary drafting is Content Generation |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — the 1-hour SLA for high-priority exceptions means skills must be fast and focused |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — Cowork provides the runtime; the skill author controls capability definition and decision logic |

### Key Insight

Supply chain exception workflows are time-sensitive and cross-functional. The framework's value here is forcing explicit separation between classification (what type of shortage is this?), impact assessment (how bad is it?), and routing (who owns the mitigation?). Without this decomposition, a single "triage the shortage" skill would conflate three distinct decision types and become unreliable. The 1-hour SLA for high-priority exceptions also means skills must surface findings quickly via Adaptive Cards and Teams messages rather than generating lengthy documents.

---

## Part 2: The Supply Chain Shortage Triage Plugin — Skill-by-Skill Design

The Supply Chain sample decomposes "Inventory Shortage and Disruption Triage" into 7 steps (SC-001 through SC-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| SC-001: Normalize shortage event | `sc-shortage-intake` | Data Aggregation | Deterministic automation | Excel (tracker), SharePoint (list), Teams (notification) |
| SC-002: Gather demand, inventory, and shipment context | `sc-context-packet` | Data Aggregation | AI act within policy | Excel (context data), SharePoint (SOPs), Teams (planner updates) |
| SC-003: Classify exception type and likely cause | `sc-classify-exception` | Decision Support | AI assist | Adaptive Card (classification), Excel (tracker update) |
| SC-004: Assess business impact and mitigation path | `sc-impact-assess` | Decision Support | AI draft + approve | Adaptive Card (impact report), Excel (priority update) |
| SC-005: Assign owner and next action | `sc-route-exception` | Decision Support | AI act within policy | Teams (assignment messages), Excel (owner update), Calendar (deadlines) |
| SC-006: Draft shortage summary and follow-up requests | `sc-shortage-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (updates), Word (handoff summary) |
| SC-007: Confirm triage disposition | *Not a skill — human decision step* | N/A | Human only | Teams (confirmation), Outlook (sign-off) |

### Detailed Skill Designs

#### 1. `sc-shortage-intake` — Normalize Shortage Event

**Framework Step:** SC-001

**Trigger phrases:** "new shortage alert", "supply disruption for [item]", "log shortage case", "inventory exception for [site]", "supplier delay reported"

**Inputs:**
- Item or SKU identifier
- Site or warehouse location
- Shortage type indicator (stockout, delay, quality hold, supplier disruption)
- Source system reference (ERP alert ID, planning exception ID)

**M365 tools:**
- `SearchM365(sources=["email"])` — find supplier delay notification or internal alert email
- `SearchM365(sources=["files"])` — check existing tracker for duplicate shortage cases on the same item and site
- `ReadFileContent` — read current exception tracker to validate no duplicate exists
- `SearchPeople` — resolve supply planner assignment based on item category or site ownership

**Output:** Structured exception record written to Excel shortage tracker in SharePoint; Teams notification to assigned planner

**Artifact:** Excel workbook with columns: Case ID, Item/SKU, Site, Shortage Type, Source System Ref, Priority, Status, Created Date, Assigned Planner, Customer Impact Flag, Estimated Recovery Date

**Guardrails:**
- Never create duplicate cases for the same item, site, and date combination
- Validate that the item identifier follows the organization's SKU format
- Post Teams notification to the supply operations channel upon case creation
- Log creation with actor and timestamp for audit trail

---

#### 2. `sc-context-packet` — Gather Demand, Inventory, and Shipment Context

**Framework Step:** SC-002

**Trigger phrases:** "build context for shortage [ID]", "what's the situation on [item]", "assemble supply context for [case]", "gather shortage details"

**Inputs:**
- Shortage case ID or item identifier

**M365 tools:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — pull inventory levels, demand signals, open orders
- `SearchM365(sources=["connectors"], connector_ids=["tms-connector"])` — pull in-transit shipment status
- `SearchM365(sources=["files"])` — locate SOPs, shortage playbooks, and mitigation templates in SharePoint
- `ReadFileContent` — read shortage playbook and routing rules from SharePoint
- `SearchM365(sources=["email"])` — find recent supplier correspondence about delays or disruptions
- `GetDriveChildren` — check for supporting documents uploaded to the case folder

**Output:** Structured context assembled in the session, presented via Adaptive Card with key data points:
- Current inventory position (on-hand, in-transit, on-order)
- Demand exposure (open customer orders, forecasted demand)
- Shipment status (in-transit quantities, ETAs, carrier updates)
- Prior shortage history for this item or site
- Applicable SOP or playbook reference

**Artifact:** Adaptive Card for quick review; Excel tracker updated with context summary fields

**Guardrails:**
- Cite data source and timestamp for every inventory or demand figure
- Flag if data is stale (older than 24 hours for inventory, older than 4 hours for shipment status)
- Never modify inventory records or demand signals — read-only access only
- Mark context as provisional if any key data source is unavailable

---

#### 3. `sc-classify-exception` — Classify Exception Type and Likely Cause

**Framework Step:** SC-003

**Trigger phrases:** "classify this shortage", "what type of exception is [case ID]", "categorize the disruption", "what caused the shortage on [item]"

**Inputs:**
- Context packet data (from `sc-context-packet`)
- Exception taxonomy (SharePoint document)

**M365 tools:**
- `ReadFileContent` — exception taxonomy and classification rules from SharePoint
- `SearchM365(sources=["files"])` — prior cases with similar characteristics for pattern matching
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — additional item and supplier history

**Output:** Classification presented via Adaptive Card:
- Exception type (stockout, supplier delay, quality hold, demand spike, logistics disruption)
- Likely cause hypothesis with supporting evidence
- Confidence level (high, medium, low) based on available data
- Similar prior cases referenced for pattern context

**Guardrails:**
- Present classification as a recommendation, not a final determination — the planner confirms or corrects
- Never auto-update the exception type in the tracker without planner review
- Include the evidence basis for the classification so the planner can validate
- Flag low-confidence classifications explicitly for human judgment
- Log classification and any override for process improvement analysis

---

#### 4. `sc-impact-assess` — Assess Business Impact and Mitigation Path

**Framework Step:** SC-004

**Trigger phrases:** "assess impact of shortage [ID]", "how bad is the [item] disruption", "what's the customer impact", "priority recommendation for [case]", "mitigation options for shortage"

**Inputs:**
- Classified exception data
- Context packet with demand and inventory figures
- Impact and severity policy (SharePoint document)

**M365 tools:**
- `ReadFileContent` — severity rubric and escalation checklist from SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — customer order exposure and allocation data
- `SearchM365(sources=["files"])` — mitigation templates and prior case resolutions

**Output:** Impact assessment presented via Adaptive Card:
- Priority recommendation (critical, high, medium, low) with rubric citation
- Downstream customer impact summary (affected orders, revenue exposure estimate)
- Recommended mitigation path (expedite, reallocate, substitute, escalate to supplier)
- Escalation recommendation if impact exceeds threshold
- Estimated time to resolution based on mitigation path

**Guardrails:**
- Present impact assessment and priority as a recommendation requiring planner approval — never auto-set priority
- Never recommend allocation changes or customer commitment modifications without explicit human approval
- Flag any customer-impacting shortage as requiring operations manager visibility regardless of calculated priority
- Include the severity rubric criteria used so the planner can validate the recommendation
- Revenue exposure estimates must be clearly labeled as approximate and based on available order data

---

#### 5. `sc-route-exception` — Assign Owner and Next Action

**Framework Step:** SC-005

**Trigger phrases:** "route shortage [ID]", "assign this exception", "who handles [item category] shortages", "send to logistics team", "escalate shortage case"

**Inputs:**
- Impact assessment output
- Routing rules and operating model (SharePoint document)
- Exception case data

**M365 tools:**
- `ReadFileContent` — routing rules and team ownership model from SharePoint
- `SearchPeople` — resolve owner by item category, site, or function
- `GetUserDetails` — verify owner availability and role
- `PostMessage` — Teams message to assigned owner with case summary and next action
- `CreateEvent` — calendar reminder for SLA deadline based on priority level

**Output:** Routed exception:
- Teams message to assigned owner with structured case summary, priority, and expected next action
- Calendar hold for SLA deadline (1 hour for critical, 4 hours for high, 1 business day for medium)
- Excel tracker updated with assigned owner and routing timestamp

**Guardrails:**
- Route only to individuals listed in the documented ownership model — never assign to someone outside the routing rules
- For critical-priority exceptions, simultaneously notify the operations manager via Teams in addition to the assigned owner
- Escalate to operations manager if no clear owner is found in the routing rules
- Log routing decision with rationale for audit and process improvement
- Never auto-close or auto-resolve a case through routing — routing is assignment, not resolution

---

#### 6. `sc-shortage-comms` — Draft Shortage Summary and Follow-Up Requests

**Framework Step:** SC-006

**Trigger phrases:** "draft shortage update for", "prepare handoff summary for [case]", "send follow-up on [item] shortage", "shortage status email", "cross-functional update on disruption"

**Inputs:**
- Case data from Excel tracker
- Context packet
- Impact assessment
- Target audience (supply planner, logistics coordinator, procurement liaison, operations manager, customer service)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts for formal communications (never auto-send)
- `PostMessage` — Teams updates to operations channels (after confirmation)
- `SearchM365(sources=["files"])` — communication templates from SharePoint
- `ReadFileContent` — standard update formats and handoff templates

**Communication templates:**
- Cross-functional shortage status update (item, impact, mitigation status, owner, ETA)
- Supplier follow-up request for delivery commitment update
- Customer service advisory with impact scope and expected resolution
- Handoff summary for shift change or team transition
- Escalation notice to operations manager with full case context

**Guardrails:**
- Always create formal communications as Outlook draft — never send without explicit user confirmation
- Teams channel updates may be posted after user confirmation for time-sensitive operational updates
- Never include customer-specific order details in broad distribution messages — scope to need-to-know
- Match urgency tone to priority level: critical cases use direct, action-oriented language
- Include case reference number and priority in every communication
- Never share supplier-confidential information (pricing, capacity data) in customer-facing communications

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** SC-007

This is not a Cowork skill. The framework correctly identifies final triage disposition — especially escalation decisions and mitigation commitments — as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the escalation review meeting if needed
- The shortage summary artifacts — provide the evidence package for the operations manager
- The `sc-shortage-comms` skill — send the disposition confirmation and next-steps communication after the human decision is made

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

**Process decomposition prevents mega-skills.** A single "triage the shortage" skill that tries to classify, assess impact, route, and communicate would be unreliable and untestable. The framework forces separation of classification (SC-003) from impact assessment (SC-004) because they have different data requirements and different error consequences. Misclassification delays triage; incorrect impact assessment can trigger unnecessary escalation or miss customer-impacting events.

**Automation boundary assignment is critical for supply chain.** Supply chain decisions frequently involve allocation changes, customer commitments, and financial impacts. The framework's boundary analysis correctly places impact assessment in "AI draft plus approve" mode — the recommendation is valuable but must never auto-execute. This maps directly to Cowork's "present via Adaptive Card, require confirmation" pattern.

**Signal inventory forces explicit tool selection.** Supply chain signals span multiple enterprise systems (ERP, TMS, planning). The framework requires naming every input source, which translates to specific MCP tool calls and Graph Connector configurations rather than vague "pull from the planning system" instructions.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — skills directory IS the registry |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — Adaptive Cards and confirmation patterns provide human-in-the-loop |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation through quality rubric and manual testing |

**The key gap:** Supply chain's 1-hour SLA for critical exceptions means the state management gap is more acute than in other LOBs. The Excel tracker must be reliable and fast to read/write, and skills must surface findings via Adaptive Card immediately rather than generating lengthy documents that delay decision-making.

### 3.3 M365 Artifacts as First-Class Process State

Supply Chain's artifact pattern is **Excel + Teams** as the primary pair, with SharePoint as the document backbone:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (shortage tracker, priority, owner, status, SLA timestamps) | The shortage tracker workbook IS the process state — skills read current status, write classifications, record assignments, and track SLA compliance |
| **Teams** | Real-time coordination channel and routing mechanism | Skills post exception assignments, status updates, escalation notices, and shift handoff summaries to operations channels; Teams is the primary communication surface for time-sensitive supply chain work |
| **SharePoint** | Source of truth (SOPs, playbooks, routing rules, severity rubrics, mitigation templates) | Skills read policies and playbooks dynamically; supporting evidence documents are stored here |
| **Adaptive Card** | In-session decision support (classifications, impact assessments, routing recommendations) | The primary output format for time-sensitive supply chain decisions — faster than document generation |
| **Outlook** | Formal communication channel (supplier correspondence, customer advisories, cross-functional updates) | Skills draft formal communications as reviewable Outlook drafts; read incoming supplier notifications for context |
| **Calendar** | SLA tracking and deadline management | Skills create SLA deadline reminders based on priority level |
| **Word** | Formal handoff documents (shift change summaries, escalation packets) | Used for formal documentation when a case requires handoff or escalation review |
| **Graph API** | People and org data (planner assignments, team ownership) | Skills resolve owners by category, site, or function |

**The design pattern:** Supply Chain favors speed over formality. The primary output format is Adaptive Card (for in-session decisions) and Teams messages (for cross-functional coordination), not Word documents. Excel remains the canonical tracker, but the communication layer is Teams-dominant because supply chain exception handling is real-time and collaborative.

### 3.4 Federated Connectors for Third-Party Systems

The Supply Chain source file references systems that do not exist natively in M365: ERP (inventory, demand, orders), planning systems, transportation management systems (TMS), and supplier portals.

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For ERP platforms (SAP S/4HANA, Oracle SCM Cloud), planning systems (Kinaxis, Blue Yonder, o9 Solutions), and transportation management systems (Oracle TMS, SAP TM, project44), Graph Connectors index shortage events, inventory positions, and shipment statuses into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector", "tms-connector"])`. This is the target state for supply chain because real-time data access is critical for the 1-hour SLA.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, Power Automate flows push periodic snapshots of inventory positions, open orders, and shipment statuses to SharePoint lists or Excel workbooks. Skills read the SharePoint copy. For supply chain, the refresh frequency matters: daily snapshots are insufficient for critical exception triage; hourly or event-triggered refreshes are needed for high-priority data.

**Tier 3 — Manual Input with Templates**

For data that cannot be automated (supplier verbal commitments, site-specific conditions, ad-hoc quality holds), skills prompt the planner for specific data points identified in the Signal Inventory. The structured intake captures data into the Excel tracker for downstream use.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint bridge with frequent Power Automate refreshes for inventory and shipment data) and Tier 3 (manual input for supplier commitments and site-specific context). Prioritize Graph Connectors for ERP and TMS in Wave 2 because supply chain's time sensitivity makes real-time data access a material improvement.

### 3.5 Governance in Cowork

Supply chain governance concerns center on allocation control, customer-commitment integrity, and operational safety:

| Governance Domain | Cowork Implementation | Supply Chain-Specific Concern |
|---|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure | Supply chain operations manager owns the skill suite; site planners own local routing rules |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC | Customer order data and supplier capacity data are commercially sensitive — skills must scope visibility to need-to-know |
| **Allocation control** | Encoded as guardrails: no skill may modify inventory allocations | AI must never auto-change allocations, expedite orders, or modify customer commitments — these require human approval with documented authority |
| **Customer commitments** | Guardrails prohibit auto-communication of delivery changes | Skills may draft customer advisories but must never send them automatically — customer-facing commitments require operations manager approval |
| **Audit** | Platform logs tool invocations; Excel tracker provides case history | Every triage decision (classification, priority, routing, escalation) must be traceable to an actor and timestamp |
| **Supply chain traceability** | Case linkage maintained across all skills | Each exception case must maintain traceability from initial event through classification, impact assessment, routing, and disposition for regulatory and operational audit |
| **Regulatory** | Guardrails flag regulated items and restricted trade lanes | For items subject to export controls, hazmat regulations, or cold chain requirements, skills must flag special handling requirements and never route outside approved pathways |
| **Release management** | Skills versioned in OneDrive; SOPs and playbooks in SharePoint | Severity rubrics, routing rules, and escalation checklists should live in SharePoint documents read dynamically so operations leadership can update them without modifying skills |

### 3.6 Evaluation Approach

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones?
- Classification accuracy — does `sc-classify-exception` assign the correct exception type? Target: above 80%
- Impact assessment quality — does `sc-impact-assess` identify the correct priority and downstream exposure?
- Tool success rate — do M365 tool calls and Graph Connector queries return expected results?

**Process-level evaluation (end-to-end):**
- Shortage classification accuracy — percentage of cases correctly classified without override
- Correct routing rate — percentage of cases routed to the right owner on the first attempt
- Time to triage for high-priority shortages — time from case creation to routed status (target: under 1 hour for critical)
- Override rate — how often planners override skill recommendations (classification, priority, routing)
- Repeat misrouting rate — how often the same type of exception is repeatedly misrouted
- Aging reduction — percentage improvement in time-to-resolution for high-impact cases
- Customer impact detection rate — percentage of customer-impacting shortages correctly flagged

---

## Part 4: Implementation Roadmap

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel shortage tracker workbook in SharePoint with standard columns (Case ID, Item/SKU, Site, Shortage Type, Source, Priority, Status, Created Date, Assigned Planner, Customer Impact Flag, Estimated Recovery Date, SLA Deadline)
- Upload SOPs, shortage playbooks, exception taxonomy, severity rubric, and routing rules to a dedicated SharePoint document library
- Set up Power Automate flows to refresh inventory and shipment data in SharePoint (Tier 2 bridge) on an hourly cadence
- Create a dedicated Teams channel for supply chain exception notifications

**Skills to build:**
- `sc-shortage-intake`
- `sc-context-packet`
- `sc-classify-exception`
- `sc-shortage-comms` (draft mode only)

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs presented via Adaptive Card for planner review. Test with 10-15 real shortage cases across different exception types.

### Wave 2 — Impact Assessment and Controlled Routing

**Skills to build:**
- `sc-impact-assess`
- `sc-route-exception`

**Promotions:**
- Promote `sc-shortage-intake` to write mode (creates case records after confirmation)
- Promote `sc-classify-exception` to write-back mode (updates tracker after planner confirmation)
- Promote `sc-route-exception` to active routing (sends Teams messages and creates calendar holds after confirmation)

**Automation:**
- Set up a scheduled prompt that runs every 2 hours checking for shortage cases approaching SLA deadline and surfaces any with unassigned owners or stale status
- Add SOP-cited triage summaries that reference specific playbook sections

**Operating posture:** AI draft plus approve for impact assessment and communications. AI act within policy for routing (bounded by documented routing rules). Every impact recommendation reviewed before priority is set.

### Wave 3 — Advanced Assembly and Proactive Detection

**Enhancements:**
- Introduce Graph Connectors for ERP inventory and TMS shipment data for real-time access
- Add bounded multi-source disruption packet assembly — skill pulls from inventory, shipment, supplier, and demand data to produce a complete disruption brief
- Add proactive detection of recurring shortage patterns via scheduled prompt: flag items or sites with repeated exceptions, identify supplier reliability trends, suggest preventive measures
- Refine all skills based on override patterns and planner feedback from Waves 1-2

**Measurement:**
- Time to triage reduction versus pre-pilot baseline (target: 40% improvement for high-priority cases)
- Classification accuracy target: above 80%
- Correct routing rate target: above 85%
- Override rate target: below 20%
- Customer impact detection target: above 90%
- Aging reduction target: 30% improvement for high-impact cases

---

## Part 5: Generalizing the Approach — Supply Chain's Artifact Pattern

Supply Chain's primary M365 artifact pattern is **Excel + Teams**. This reflects three characteristics of supply chain exception workflows:

1. **Structured tracking with SLA sensitivity** — exception cases, priority levels, owner assignments, and resolution timestamps are tabular and live in Excel, but the 1-hour SLA for critical cases means the tracker must be updated and read quickly
2. **Real-time cross-functional coordination** — supply chain triage involves planners, logistics, procurement, and customer service working concurrently; Teams is the primary coordination surface, not email
3. **Speed over formality** — the primary output format is Adaptive Card and Teams message, not Word document; formal documentation (handoff summaries, escalation packets) is secondary to fast triage

This pattern differs from document-heavy LOBs (Procurement, Legal) where Word and SharePoint dominate, and from approval-chain LOBs (Finance, HR) where formal written artifacts are primary. The cross-LOB method remains the same: decompose the process, assign automation boundaries, map signals to M365 tools, identify the primary artifact, and build Wave 1 as read-only. Supply Chain's unique contribution is demonstrating that Adaptive Cards and Teams messages can serve as the primary skill output when speed matters more than document formality.

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
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, ERP event via Graph Connector, calendar event |
| Approval | Adaptive Card presentation + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via triage time and routing accuracy |
