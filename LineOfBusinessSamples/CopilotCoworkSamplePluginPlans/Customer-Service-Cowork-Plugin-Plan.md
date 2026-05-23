# Plan: Customer Service Case Intake and Resolution Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Customer Service line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Case Intake and Resolution Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Customer Service, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" Customer Service case triage scores high on all dimensions. |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — Customer Service references "CRM", "case management platform", and "telephony platform" that must become specific M365 tool names or Graph Connector sources |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — Customer Service boundaries are well-defined because SLA and customer-commitment controls are already formalized |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1-5 and 7-9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Customer Service case triage is communication-heavy and time-sensitive. The framework's decomposition produces skills that naturally split along the intake-classify-route-respond axis. The primary Cowork design challenge is not decomposition but rather **federated data access**: CRM and case management state lives outside M365, so the skill suite depends heavily on Graph Connectors or SharePoint bridge patterns to function. The secondary challenge is **SLA awareness** — several skills need to surface time-bound constraints (15-minute routing SLA) that the Cowork platform does not natively track.

---

## Part 2: The Customer Service Plugin — Skill-by-Skill Design

The Customer Service sample decomposes "Case Intake and Resolution Triage" into 7 steps (CS-001 through CS-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| CS-001: Normalize case event | `cs-case-intake` | Data Aggregation | Deterministic automation | Excel (case tracker), SharePoint (list), Outlook (intake email) |
| CS-002: Gather customer and case context | `cs-context-packet` | Data Aggregation | AI act within policy | Word (context packet), SharePoint (account docs), Graph API (people) |
| CS-003: Classify issue and intent | `cs-issue-classifier` | Decision Support | AI assist | Adaptive Card (classification), Excel (tracker update) |
| CS-004: Assess severity and SLA path | `cs-severity-assessment` | Decision Support | AI draft + approve | Adaptive Card (severity report), Excel (priority update) |
| CS-005: Route to owner or queue | `cs-case-routing` | Decision Support | AI act within policy | Teams (routing messages), Excel (owner assignment) |
| CS-006: Draft first response or handoff | `cs-response-drafter` | Content Generation | AI draft + approve | Outlook (draft reply), Word (handoff summary), Teams (escalation notes) |
| CS-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `cs-case-intake` — Normalize Case Event

**Framework Step:** CS-001

**Trigger phrases:** "new service case", "log case for [customer]", "intake from [channel]", "new ticket from", "customer email case"

**Inputs:**
- Customer name, email, or account identifier
- Inbound channel (email, chat, form, call transcript)
- Issue description or message body
- Any attachments (screenshots, logs, invoices)

**M365 tools:**
- `SearchM365(sources=["email"])` — find the inbound customer email or thread
- `SearchPeople` — resolve customer contact if known internally
- `GetUserDetails` — pull internal account owner profile
- `ReadFileContent` — extract content from attached files in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — pull CRM account record if Graph Connector is configured

**Output:** Structured case record written to Excel case tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Customer Name, Account ID, Channel, Issue Summary, Category (pending), Priority (pending), Status, Created Date, SLA Deadline, Assigned To, Resolution

**Guardrails:**
- Never create duplicate cases for the same customer email thread
- Validate that all required fields are populated before writing to tracker
- Confirm details with user before writing to tracker
- Automatically calculate SLA deadline (15 minutes from creation for standard cases)

---

#### 2. `cs-context-packet` — Gather Customer and Case Context

**Framework Step:** CS-002

**Trigger phrases:** "build context for case [ID]", "pull customer history for", "assemble case context", "what do we know about this customer"

**Inputs:**
- Case ID or customer name from the case tracker

**M365 tools:**
- `ReadFileContent` — read case tracker row from SharePoint Excel workbook
- `SearchM365(sources=["email"])` — prior correspondence with this customer
- `SearchM365(sources=["files"])` — account documents, contracts, entitlement records in SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — CRM account profile, prior cases, entitlement tier
- `ListChatMessages` — prior Teams discussions about this customer or account
- `GetUserDetails` — account owner and assigned service agent profiles

**Output:** Word document containing:
- Customer profile and account summary
- Entitlement and service tier details
- Prior case history (last 5 cases with outcomes)
- Product or service context relevant to current issue
- Known issues or active incidents affecting this customer
- Account owner and escalation contacts

**Artifact:** Word (.docx) saved to SharePoint case folder

**Guardrails:**
- Mask financial account details (credit card numbers, billing specifics) in generated documents
- Cite source and retrieval date for every data point from CRM or prior cases
- Flag if customer entitlement data is unavailable or stale (older than 30 days)
- Operate within read-only boundaries — never update CRM or account records

---

#### 3. `cs-issue-classifier` — Classify Issue and Intent

**Framework Step:** CS-003

**Trigger phrases:** "classify this case", "what type of issue is this", "categorize case [ID]", "triage this ticket"

**Inputs:**
- Case data from Excel tracker
- Context packet (Word document)
- Inbound message content

**M365 tools:**
- `ReadFileContent` — case tracker and context packet from SharePoint
- `SearchM365(sources=["files"])` — issue taxonomy and classification guide from SharePoint
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — prior case classifications for pattern matching

**Output:** Classification report as Adaptive Card containing:
- Recommended issue category (from taxonomy)
- Detected customer intent (request, complaint, inquiry, escalation)
- Confidence indicator (high, medium, low)
- Similar prior cases with resolutions
- Suggested queue based on category

**Logic:** Compare inbound issue description against the service taxonomy document in SharePoint. Cross-reference with prior case patterns from CRM. Present classification with confidence score and supporting evidence.

**Guardrails:**
- Present classification as recommendation only — never auto-assign category without user review
- Always show confidence level; flag low-confidence classifications prominently
- Surface similar prior cases to support the classification rationale
- Never classify based on customer identity alone (prevent bias toward account tier)

---

#### 4. `cs-severity-assessment` — Assess Severity and SLA Path

**Framework Step:** CS-004

**Trigger phrases:** "assess severity for case [ID]", "what priority is this", "check SLA path", "is this an escalation"

**Inputs:**
- Classification output (issue type, intent, confidence)
- Context packet (customer tier, prior cases, entitlement)
- SLA policy document from SharePoint

**M365 tools:**
- `ReadFileContent` — SLA policy and severity rubric from SharePoint
- `ReadFileContent` — case tracker and context packet
- `SearchM365(sources=["files"])` — escalation criteria documents
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — customer entitlement tier and active incidents

**Output:** Severity assessment as Adaptive Card containing:
- Recommended severity level (Critical, High, Standard, Low)
- SLA path and timeline
- Escalation conditions met (if any)
- Customer impact assessment
- Recommended response path (standard queue, priority queue, immediate escalation)

**Guardrails:**
- Present severity as draft recommendation — require explicit user confirmation before applying
- Never downgrade a customer-reported severity without documenting the rationale
- Flag any case that meets escalation criteria even if the overall severity appears low
- Cross-reference with active incident list to detect cases linked to known outages
- Include SLA countdown in the assessment output

---

#### 5. `cs-case-routing` — Route to Owner or Queue

**Framework Step:** CS-005

**Trigger phrases:** "route case [ID]", "assign this case", "send to the right queue", "who handles this type of issue"

**Inputs:**
- Classification and severity output
- Routing rules document from SharePoint
- Org model and team availability

**M365 tools:**
- `ReadFileContent` — routing rules matrix from SharePoint
- `SearchPeople` — resolve queue owners and specialists by function
- `GetManagerDetails` / `GetDirectReportsDetails` — team structure for escalation paths
- `PostMessage` — Teams notification to assigned agent or queue channel
- `PostChannelMessage` — post case summary to the team's triage channel
- `ListCalendarView` — check assigned agent availability before routing

**Output:** Routed case:
- Teams message to assigned agent with case summary, classification, severity, and SLA deadline
- Channel post to triage channel with case card
- Updated Excel tracker with owner assignment and routed timestamp

**Guardrails:**
- Route only to agents listed in the approved routing matrix
- Verify agent availability via calendar before assignment; flag conflicts
- Never auto-route high-severity or escalation cases — present recommendation for team lead review
- Include SLA deadline in every routing notification
- Log routing decision with rationale in the case tracker

---

#### 6. `cs-response-drafter` — Draft First Response or Handoff Summary

**Framework Step:** CS-006

**Trigger phrases:** "draft response for case [ID]", "write customer reply", "prepare handoff summary", "escalation summary for", "first response for"

**Inputs:**
- Case data from Excel tracker
- Context packet (Word document)
- Classification and severity output
- Knowledge base articles from SharePoint
- Target audience (customer, internal team, escalation manager)

**M365 tools:**
- `CreateDraftMessage` — Outlook draft reply to customer (never auto-send)
- `PostMessage` — Teams handoff message to receiving agent or escalation manager
- `SearchM365(sources=["files"])` — response templates and knowledge articles from SharePoint
- `ReadFileContent` — approved response templates

**Communication templates:**
- Acknowledgment email to customer (with case reference and expected response time)
- Internal handoff summary for receiving agent
- Escalation packet for escalation manager
- Follow-up request for missing information from customer

**Guardrails:**
- Always create customer-facing communications as Outlook draft — never send without explicit user confirmation
- Never include internal severity, agent names, or routing details in customer-facing drafts
- Never make commitments (credits, refunds, timeline promises) in drafted responses
- Match tone to audience: empathetic and clear for customer-facing, operational and precise for internal
- Include case reference number in every communication
- Cite knowledge articles used in response drafting

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** CS-007

This is not a Cowork skill. The framework correctly identifies final triage confirmation as a human-only step, particularly for escalations requiring supervisor sign-off. In Cowork, it is supported by:

- The severity assessment and routing outputs — provide the evidence package for the reviewer
- The `cs-response-drafter` skill — send the disposition confirmation after the human decision is made
- Calendar tools — book triage review meetings for complex or high-severity cases
- The case tracker Excel workbook — records the final disposition with timestamp and approver

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. Customer Service adds a layer of complexity because it is **communication-centric and time-sensitive**, with SLA constraints that the Cowork platform does not natively enforce.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle an entire domain. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills. For Customer Service, this prevents a monolithic "handle case" skill and instead produces the focused intake-classify-route-respond chain.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with meeting artifacts and summary outputs |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather customer context," the framework requires naming every input source. This translates to specific MCP tool calls in the SKILL.md instructions.

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

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (case status, SLA timer, prior decisions) must be externalized to M365 artifacts — specifically the Excel case tracker and SharePoint document library.

### 3.3 M365 Artifacts as Process State

For Customer Service, the artifact pattern centers on **Outlook and Teams as primary communication channels** with **Excel as the case state store**:

| Artifact | Role in Customer Service | How Skills Use It |
|---|---|---|
| **Excel** | Case state store (case tracker with status, priority, SLA, owner, resolution) | The case tracker workbook IS the process state — skills read current status, write updates, track SLA compliance |
| **Word** | Evidence artifacts (context packets, handoff summaries) | Skills generate context packets that become the auditable record of what was assembled for each case |
| **Outlook** | Primary customer communication channel and signal source | Skills read inbound customer emails for case intake; draft outgoing responses as reviewable drafts |
| **Teams** | Internal coordination channel and real-time routing | Skills post case assignments, escalation notices, and handoff summaries to team channels and agent chats |
| **SharePoint** | Source of truth for policies, templates, and case documents | Skills read SLA policies, routing rules, response templates, and knowledge articles; case folders store evidence |
| **Adaptive Card** | Real-time decision support (classification, severity, routing recommendations) | Skills present triage recommendations for agent review before any write action |
| **Graph API** | People and org data (agent profiles, team structure, availability) | Skills resolve agents, check availability, and determine escalation paths |
| **Calendar** | Agent availability and triage review scheduling | Skills check agent calendars before routing; schedule review meetings for complex cases |

**The design pattern:** The Cowork Customer Service plugin uses a SharePoint-hosted Excel workbook as the canonical case tracker, with Outlook as the customer-facing channel, Teams as the internal coordination channel, and Word documents as the evidence trail. SLA deadlines are tracked as calculated fields in the Excel tracker and surfaced in Adaptive Cards and Teams notifications.

### 3.4 Federated Connectors for Third-Party Systems

Customer Service depends heavily on external systems: CRM (Salesforce, Dynamics 365, ServiceNow), case management platforms, telephony/chat platforms (Genesys, Five9), and knowledge bases (Guru, Confluence). The federated access approach follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For CRM platforms (Salesforce, Dynamics 365) and case management platforms (ServiceNow, Zendesk), Graph Connectors index customer records, case history, and entitlement data into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])`. This provides read access to account profiles, prior cases, and entitlement tiers without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, synchronized data is maintained in SharePoint lists or Excel workbooks populated by Power Automate flows from the CRM. Skills interact with the SharePoint copy. Key bridge artifacts:
- Customer entitlement lookup table (Excel in SharePoint, synced from CRM)
- Issue taxonomy and routing rules (SharePoint document, maintained by service operations)
- Knowledge article index (SharePoint list, synced from knowledge base platform)

**Tier 3 — Manual Input with Templates**

For telephony transcripts, chat logs, and systems with no integration path, skills provide structured intake that captures data from manual lookups, writing it into the case tracker. The framework's Signal Inventory phase identifies exactly which data points are needed, so the skill can prompt for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for CRM data and Tier 3 (manual input) for telephony and chat content. Graph Connectors require tenant admin setup and are better introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

Customer Service governance is shaped by three specific concerns: **customer data sensitivity** (PII, account details), **SLA compliance** (time-bound obligations), and **unauthorized commitments** (credits, refunds, timeline promises).

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure and escalation paths |
| **Access** | M365 permissions govern data reach; Graph API respects tenant RBAC; CRM data accessed through permissioned connectors |
| **Data classification** | Skill guardrails enforce PII handling: mask financial details, never include internal routing info in customer-facing drafts |
| **SLA enforcement** | SLA deadlines tracked in Excel tracker; scheduled prompts surface approaching breaches; Adaptive Cards display countdown |
| **Commitment control** | No skill may make customer commitments (credits, refunds, timelines); all customer-facing drafts require human review |
| **Audit** | Platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; Excel tracker records every state change with timestamp |
| **Release management** | Skills versioned in OneDrive; quality rubric scoring provides pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions; dynamic policy content read from SharePoint routing rules and SLA documents at runtime |

**The main governance gap** is real-time SLA enforcement. Cowork does not have a native timer or escalation engine. The mitigation is a scheduled prompt that runs at regular intervals (every 10-15 minutes), reads the case tracker, and surfaces cases approaching or breaching SLA deadlines via Teams notifications.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do generated classifications, severity assessments, and drafts match expert judgment? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Classification accuracy — percentage of cases classified correctly on first pass
- Correct routing rate — percentage of cases routed to the right queue or agent
- Time to first useful response — measured from case creation to draft response ready
- SLA breach rate — percentage of cases exceeding the 15-minute routing target
- Reopen rate — percentage of cases reopened after initial triage
- Draft acceptance rate — percentage of `cs-response-drafter` drafts sent without major edits
- Override rate — how often agents override skill recommendations
- Unauthorized commitment incidents — any instance where a draft contained a commitment (target: zero)

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel case tracker workbook in SharePoint with standard columns (Case ID, Customer Name, Account ID, Channel, Issue Summary, Category, Priority, Status, SLA Deadline, Created Date, Assigned To, Resolution, Last Updated, Updated By)
- Upload SLA policies, routing rules, issue taxonomy, and response templates to a dedicated SharePoint document library
- Configure SharePoint bridge data: customer entitlement lookup table, knowledge article index
- Create a SharePoint folder structure for per-case evidence documents

**Skills to build:**
- `cs-case-intake`
- `cs-context-packet`
- `cs-issue-classifier`
- `cs-response-drafter`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 10-15 real cases across multiple issue types and channels.

### Wave 2 — Routing and Severity

**Skills to build:**
- `cs-severity-assessment`
- `cs-case-routing`

**Promotions:**
- Promote `cs-case-intake` to write mode (creates case records after confirmation)
- Promote `cs-issue-classifier` to write-back mode (updates Excel tracker category after user confirmation)
- Promote `cs-case-routing` to bounded write mode (posts routing messages to Teams channels within approved routing matrix)

**Automation:**
- Set up a scheduled prompt (every 15 minutes) that checks the case tracker for cases approaching SLA breach and surfaces warnings via Teams notifications
- Set up a daily scheduled prompt that summarizes case volume, average triage time, and SLA compliance for the operations manager

**Operating posture:** AI draft plus approve for severity assessment and customer-facing communications. AI act within policy for routing to approved queues. Every customer-facing output reviewed before action.

### Wave 3 — Optimization and Proactive Support

**Enhancements:**
- Introduce Graph Connectors for CRM data if available at the tenant level
- Add escalation packet assembly: multi-source aggregation skill that pulls case data, prior correspondence, customer history, and knowledge articles into a single escalation Word document
- Add proactive reopen detection: scheduled prompt that analyzes recently closed cases for patterns that predict reopening
- Add knowledge gap detection: skill that identifies cases where no knowledge article matched, surfacing gaps to the content team
- Refine all skills based on override patterns and agent feedback from Waves 1-2

**Measurement:**
- Classification accuracy target: above 85%
- Correct routing rate target: above 90%
- Time to first useful response target: under 10 minutes for standard cases
- SLA breach rate target: below 5%
- Draft acceptance rate target: above 70%
- Unauthorized commitment incidents target: zero

---

## Part 5: Generalizing the Approach — Customer Service Artifact Pattern

Customer Service's primary artifact pattern is **Outlook + Teams (case communication, escalation routing)** supported by **Excel (case state tracking)**.

This LOB demonstrates the **communication-centric variant** of the framework-to-Cowork translation:

1. **The primary process state artifact is the Excel case tracker** — but unlike HR (where the tracker is the center of gravity), Customer Service skills spend most of their time reading from and writing to Outlook and Teams. The tracker is the coordination backbone, not the primary workspace.

2. **Adaptive Cards are the primary decision-support surface** — Customer Service agents work in real-time and need inline triage recommendations, not long documents. The classification, severity, and routing skills all present their outputs as Adaptive Cards first, with document artifacts as secondary evidence.

3. **SLA awareness is a cross-cutting concern** — Every skill that touches the case tracker needs to surface SLA deadlines. This is achieved through calculated fields in Excel and scheduled prompts for breach monitoring.

4. **Customer-facing guardrails are the strictest governance requirement** — No skill may auto-send to customers, make commitments, or include internal routing details in external communications. This pattern applies to any LOB with external-facing communication (Sales, Account Management, Field Service).

The decomposition method (one goal per step, boundary-driven guardrails, M365 artifact grounding) works identically for Customer Service as it does for HR. The difference is which artifacts dominate: HR is document-centric (Word, Excel); Customer Service is communication-centric (Outlook, Teams, Adaptive Card).

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
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time, routing accuracy, and SLA compliance |
| CRM / Case Platform | Graph Connectors or SharePoint bridge | External system data accessed via `SearchM365(sources=["connectors"])` or synced to SharePoint |
| SLA Timer | Excel calculated field + scheduled prompt | No native timer in Cowork; SLA tracked in tracker and monitored by recurring prompt |
