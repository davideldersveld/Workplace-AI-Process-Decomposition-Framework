# Plan: Capital Markets Trade Exception Triage — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Capital Markets line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Trade Exception and Settlement Break Triage pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

Capital markets is the most time-sensitive and regulatory-dense vertical in the framework repository. Every design decision here reflects three constraints that do not exist at the same intensity in other LOBs: settlement deadline pressure (T+1 in the US), information barrier requirements (Chinese walls), and zero tolerance for unauthorized trade or settlement actions.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). Capital markets introduces domain-specific translation challenges at every phase: trade lifecycle complexity means decomposition produces more interdependent steps, regulatory density means automation boundaries are more restrictive, time sensitivity means latency in context assembly has direct financial impact, and market data requirements mean signal inventory involves systems with strict entitlements.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly; capital markets adds settlement-deadline and fail-exposure weighting |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule is especially important here because combining triage and routing in a single skill would obscure accountability |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires heavy translation — the framework references OMS, settlement platforms, SSI repositories, and confirmations systems that must map to M365 artifacts or Graph Connectors |
| **Automation Boundary** | Operating mode per step | **Guardrails and confirmation gates** in SKILL.md | Critical — capital markets has the most restrictive boundary posture of any vertical; no skill may amend bookings, change SSIs, or authorize settlement actions |
| **Capability Mapping** | AI task pattern per step | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates; capital markets heavily uses Extract, Classify, Compare, and Route |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md**, **MCP tool calls**, **multi-skill orchestration** | Good but capital markets adds timing constraints — skills must surface cutoff proximity and settlement date in every output |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — but capital markets requires explicit attention to the governance layer (Layer 8) because entitlements, information barriers, and audit logging are non-negotiable |

### Key Insight

Capital markets amplifies two framework tensions that are mild in other verticals. First, the gap between system-of-record data (trade management, settlement platforms) and M365-accessible data is wider here than in HR or finance — the most critical signals live in non-M365 systems. Second, the automation boundary is more restrictive because even a "draft" that implies a settlement commitment can create regulatory exposure. The skill author's job in capital markets is therefore: define the capability narrowly, set boundaries aggressively, ground every output in cited trade data, and never let a skill output look like an authorized action.

---

## Part 2: The Capital Markets Plugin — Skill-by-Skill Design

The capital markets sample decomposes "Trade Exception and Settlement Break Triage" into 7 steps (CM-TRD-001 through CM-TRD-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| CM-TRD-001: Normalize trade break event | `cm-break-intake` | Data Aggregation | Deterministic automation | Excel (break tracker), SharePoint (list), Outlook (notifications) |
| CM-TRD-002: Gather trade and settlement context | `cm-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (trade docs, SSI files), Graph API (people) |
| CM-TRD-003: Classify break type and likely cause | `cm-break-classifier` | Decision Support | AI assist | Excel (tracker update), Adaptive Card (classification report) |
| CM-TRD-004: Assess settlement risk and time criticality | `cm-risk-assessment` | Decision Support | AI assist | Excel (risk flags), Adaptive Card (risk dashboard), SharePoint (settlement rules) |
| CM-TRD-005: Route to correct desk or owner | `cm-break-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Excel (tracker update) |
| CM-TRD-006: Draft counterparty and internal handoff summaries | `cm-break-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (handoff summary) |
| CM-TRD-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `cm-break-intake` — Normalize Trade Break Event

**Framework Step:** CM-TRD-001

**Trigger phrases:** "new trade break", "settlement exception for [trade ID]", "log break case", "trade fail alert", "SSI mismatch on [counterparty]"

**Inputs:**
- Trade ID or break reference
- Break source (email alert, system notification, counterparty notice)
- Asset class and instrument type
- Settlement date
- Counterparty name

**M365 tools:**
- `SearchM365(sources=["email"])` — find break notification email or counterparty notice
- `SearchM365(sources=["connectors"], connector_ids=["oms-connector"])` — pull trade record from OMS/EMS if Graph Connector available
- `SearchPeople` — resolve operations analyst and desk ownership
- `GetDriveChildren` — check existing break tracker for duplicate cases
- `ReadFileContent` — read break taxonomy reference from SharePoint

**Output:** Structured break case record written to Excel break tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Break ID, Trade ID, Asset Class, Instrument, Counterparty, Settlement Date, Break Source, Break Type (pending), Priority (pending), Status, Created Timestamp, Assigned To, Desk

**Guardrails:**
- Never create duplicate cases for the same trade ID and settlement date
- Validate that settlement date is present and parseable
- Confirm details with user before writing to tracker
- Never infer trade details not present in the source signal — flag missing fields explicitly
- Include timestamp of intake for audit trail

---

#### 2. `cm-context-packet` — Gather Trade, Settlement, and Counterparty Context

**Framework Step:** CM-TRD-002

**Trigger phrases:** "build break packet for [trade ID]", "assemble context for break [ID]", "what do we know about this trade exception", "pull settlement details for [counterparty]"

**Inputs:**
- Break case ID or trade ID from the Excel tracker

**M365 tools:**
- `ReadFileContent` — trade confirmation documents, SSI files, allocation files from SharePoint
- `SearchM365(sources=["files"])` — settlement instructions, counterparty reference data, prior break reports
- `SearchM365(sources=["email"])` — counterparty correspondence, internal desk handoff emails
- `SearchM365(sources=["connectors"], connector_ids=["settlement-connector"])` — settlement platform data if indexed
- `GetUserDetails` — resolve operations analyst and manager contacts
- `GetDriveChildren` — enumerate documents in the break case folder

**Output:** Word document containing:
- Trade summary (ID, asset class, instrument, quantity, price, trade date, settlement date)
- Counterparty and SSI details
- Affirmation and confirmation status
- Settlement instruction comparison (expected vs. received)
- Prior break history for this counterparty or instrument
- Relevant email thread excerpts with timestamps
- List of missing or unverifiable data points

**Artifact:** Word (.docx) saved to SharePoint break case folder

**Guardrails:**
- Cite source system and retrieval timestamp for every data point
- Flag any data point sourced from email rather than system of record with a reliability warning
- Never present stale settlement data without noting the retrieval time — settlement states change rapidly
- Mask client account numbers in generated documents; use counterparty short codes only
- Do not include MNPI (material non-public information) from unrelated business lines — respect information barriers

---

#### 3. `cm-break-classifier` — Classify Break Type and Likely Cause

**Framework Step:** CM-TRD-003

**Trigger phrases:** "classify this break", "what type of break is [ID]", "root cause analysis for trade exception", "categorize the settlement mismatch"

**Inputs:**
- Context packet (Word document or break case data from Excel tracker)
- Break taxonomy reference (SharePoint document)

**M365 tools:**
- `ReadFileContent` — break taxonomy and classification playbook from SharePoint
- `ReadFileContent` — context packet from prior skill output
- `SearchM365(sources=["files"])` — historical break reports for pattern matching

**Output:** Classification report as Adaptive Card (for quick review) plus Excel tracker update (break type column)

**Logic:** Compare break characteristics against the taxonomy: SSI mismatch, allocation discrepancy, affirmation failure, settlement timing issue, counterparty data error, booking correction needed, or reference data break. Assign confidence level. If multiple categories apply, rank by likelihood and flag as multi-cause.

**Guardrails:**
- Always present classification as a recommendation, never as a determination — final classification belongs to the operations analyst
- Include confidence level (high, medium, low) for every classification
- Flag cases where email narrative and system state conflict — surface the discrepancy rather than resolving it
- Never recommend a booking correction or settlement action as part of classification
- Present findings for user review before updating the Excel tracker

---

#### 4. `cm-risk-assessment` — Assess Settlement Risk and Time Criticality

**Framework Step:** CM-TRD-004

**Trigger phrases:** "assess fail risk for [trade ID]", "how urgent is this break", "settlement risk check", "is this break near cutoff", "check aging on break [ID]"

**Inputs:**
- Break case data from Excel tracker
- Context packet
- Settlement rules and cutoff schedules (SharePoint documents)
- Break aging data

**M365 tools:**
- `ReadFileContent` — settlement cutoff schedules and fail escalation rules from SharePoint
- `ReadFileContent` — context packet and tracker data
- `SearchM365(sources=["files"])` — prior fail history for this counterparty or instrument
- `SearchM365(sources=["connectors"], connector_ids=["settlement-connector"])` — real-time settlement status if indexed

**Output:** Risk assessment as Adaptive Card with:
- Priority recommendation (critical, high, standard, low)
- Fail risk flag (yes/no with rationale)
- Time to settlement cutoff
- Break age in hours
- Escalation recommendation (immediate supervisor, desk head, operations control)
- Missing evidence gaps that increase risk uncertainty

**Artifact:** Excel tracker updated with priority and fail-risk columns; Adaptive Card for real-time review

**Guardrails:**
- Same-day settlement breaks must always be flagged as critical regardless of other factors
- Never downgrade a fail-risk flag once set — only a human supervisor may reduce priority
- Surface cutoff proximity in every output — operations teams must always see time remaining
- Do not calculate financial exposure amounts — that requires authorized position and pricing data
- Present risk assessment for user review; do not auto-update tracker for critical or high-priority breaks

---

#### 5. `cm-break-routing` — Route to Correct Desk or Owner

**Framework Step:** CM-TRD-005

**Trigger phrases:** "route this break", "who handles [break type]", "assign break [ID] to the right desk", "send to settlements", "escalate to operations control"

**Inputs:**
- Break case data including classification and risk assessment
- Routing matrix (SharePoint document)
- Desk responsibility matrix

**M365 tools:**
- `ReadFileContent` — desk routing matrix and escalation rules from SharePoint
- `SearchPeople` — resolve desk owners by function or name
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to the assigned desk or individual
- `CreateEvent` — deadline reminder on calendar for settlement cutoff

**Output:** Routed break:
- Teams message to the assigned desk or operations owner with break summary, priority, and required action
- Calendar hold for settlement cutoff deadline if same-day or next-day
- Updated Excel tracker with assigned owner and desk columns

**Guardrails:**
- Present routing recommendation for operations supervisor review before sending any messages
- Never route to someone outside the desk responsibility matrix
- Critical and high-priority breaks must include the operations supervisor on all routing notifications
- Escalations to operations control require explicit user confirmation — never auto-escalate
- Never include trade booking details or settlement instructions in Teams messages — link to the SharePoint break case folder instead
- Information barrier compliance: do not route break details across Chinese wall boundaries (e.g., do not send equity trading desk information to fixed income desk personnel unless authorized)

---

#### 6. `cm-break-comms` — Draft Counterparty and Internal Handoff Summaries

**Framework Step:** CM-TRD-006

**Trigger phrases:** "draft counterparty notice for [break ID]", "prepare handoff summary", "write internal escalation email", "break follow-up to [counterparty]", "settlement break communication"

**Inputs:**
- Break case data from Excel tracker
- Context packet (Word document)
- Classification and risk assessment
- Target audience (counterparty, internal desk, operations control, client servicing)
- Communication templates from SharePoint

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages (internal only, with confirmation)
- `SearchM365(sources=["files"])` — communication templates from SharePoint
- `ReadFileContent` — approved counterparty outreach examples

**Communication templates:**
- Counterparty break notification (SSI mismatch, allocation discrepancy, settlement timing)
- Internal desk handoff summary
- Operations control escalation memo
- Client servicing notification (for client-impacting breaks)
- Settlement deadline reminder

**Guardrails:**
- Always create counterparty-facing communications as Outlook draft — never auto-send external messages
- Counterparty drafts must never imply a booking correction, settlement commitment, or position disclosure
- Every draft must include break reference ID, trade ID, and settlement date for traceability
- Internal Teams messages require user confirmation before posting
- Do not include client account numbers, position sizes, or pricing data in any communication
- Do not disclose information across information barrier boundaries — counterparty communications must be reviewed for MNPI compliance
- Match tone to audience: factual and neutral for counterparty-facing, operational and specific for internal desk handoffs
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable before delivery

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** CM-TRD-007

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step. In capital markets, this is non-negotiable because disposition decisions may trigger booking corrections, settlement instruction changes, or regulatory reporting actions — all of which require explicit human authorization.

In Cowork, this step is supported by:

- The `cm-readiness-summary` pattern — a scheduled prompt that surfaces break cases approaching cutoff without confirmed disposition
- The risk assessment artifacts — provide the evidence package for the supervisor
- The `cm-break-comms` skill — sends the sign-off confirmation or escalation notice after the human decision is made
- `ListCalendarView` / `CreateEvent` — book the review meeting for complex or escalated breaks

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. Capital markets amplifies this challenge because the most critical data lives in non-M365 systems (OMS, settlement platforms, confirmations engines) and the regulatory environment demands stricter controls than the framework's generic governance model provides.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in capital markets is **pre-implementation discipline with regulatory awareness**:

**Process decomposition prevents mega-skills and preserves accountability.** In capital markets, combining classification and routing in a single skill would blur the audit trail for who recommended what and when. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces skills with clear accountability boundaries that regulators and internal audit can inspect.

**Automation boundary assignment maps to guardrails architecture with regulatory teeth.** The five operating modes translate directly, but capital markets adds a regulatory overlay:

| Framework Mode | Cowork Implementation | Capital Markets Overlay |
|---|---|---|
| Human only | Do not build a skill; support with evidence assembly | Required for any action that commits to a booking, settlement, or regulatory filing |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions | Required for classification and risk assessment — outputs are recommendations only |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation | Required for all counterparty-facing and routing communications; drafts must not imply settlement commitments |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules | Permitted only for context retrieval and tracker updates to non-sensitive fields |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed | Permitted for intake normalization only |

**Signal inventory forces explicit M365 tool selection with entitlement awareness.** In capital markets, not every user may access every trade record. The signal inventory phase forces the skill author to document which data sources require which entitlements — this translates to explicit permission checks in skill instructions.

### 3.2 What Cowork Provides That the Framework Assumes You Build

The framework describes a 9-layer reference architecture. In Cowork, most layers are platform-provided, but capital markets exposes specific gaps:

| Framework Layer | Cowork Provides It As | Capital Markets Gap |
|---|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, files via MCP tools | Trade system signals require Graph Connectors or SharePoint bridge |
| Process model | Implicit — skill trigger phrases and instructions | No formal break-state machine; state tracked in Excel |
| Capability registry | Built-in — skills directory | Adequate |
| Runtime orchestrator | Built-in — session manages tool selection and execution | No cutoff-aware scheduling; must use scheduled prompts |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, Graph API tools | Trade data freshness not guaranteed; must timestamp every retrieval |
| Memory and state | Partial — session memory persists within inline scheduled tasks | Break state must be externalized to Excel tracker; no real-time state sync with OMS |
| Decision and approval plane | Partial — draft tools and confirmation gates | No formal supervisory approval workflow; must use Teams messages and calendar events |
| Governance and control | Partial — skill instructions encode policies | No information barrier enforcement at platform level; must encode in guardrails |
| Evaluation and observability | Limited — no built-in metrics | Must track triage cycle time and fail rates externally |

### 3.3 M365 Artifacts as Process State

This is the most important architectural insight for capital markets in Cowork. Each M365 artifact type serves a specific role in the trade exception process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (break tracker, risk flags, routing assignments, break aging) | The break tracker workbook IS the process state — skills read current break status, write classification updates, record priority and routing assignments |
| **Word** | Evidence artifacts (context packets, handoff summaries, investigation notes) | Skills generate documents that become the auditable record of what was assembled and assessed for each break case |
| **SharePoint** | Source of truth (break taxonomy, routing matrix, settlement rules, SSI reference, cutoff schedules, communication templates) | Skills read operational policies and reference data from SharePoint; break case folders hold all evidence per case |
| **Outlook** | Communication channel and signal source | Skills read incoming counterparty break notices and internal escalation emails; draft all outgoing communications as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post routing notifications, desk handoffs, and cutoff alerts to channels or chats |
| **Graph API** | People and org data (desk ownership, escalation chains, supervisor structure) | Skills resolve desk owners, operations supervisors, and escalation paths for routing decisions |
| **Calendar** | Time-bound process events (settlement cutoffs, review deadlines) | Skills create calendar holds for settlement cutoff deadlines and break review meetings |
| **Adaptive Card** | Real-time operational dashboard | Skills present classification, risk assessment, and routing recommendations as interactive cards for immediate desk review |

**The design pattern:** The Cowork capital markets plugin uses a SharePoint-hosted Excel workbook as the canonical break tracker, with Word context packets as the evidence trail and Adaptive Cards as the real-time operational interface. This is less formally rigorous than a purpose-built trade operations workflow engine, but it works within the M365 ecosystem and gives operations teams artifacts they already know how to audit. The key difference from HR is speed — break tracker updates must happen within the 30-minute SLA window, so skills must minimize round-trips and present information in the most immediately actionable format (Adaptive Cards for triage, Word documents for escalation evidence).

### 3.4 Federated Connectors for Third-Party Systems

Capital markets has the heaviest third-party system dependency of any vertical. The framework references OMS/EMS, settlement platforms, SSI repositories, confirmations engines, and more. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For trade management systems (Charles River, Bloomberg AIM, Aladdin), settlement platforms (DTCC, Omgeo/CTM), and market data providers (Bloomberg Terminal, Refinitiv Eikon), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["oms-connector"])`. This provides read access to trade records, settlement status, and SSI data without custom integration code. Critical caveat: Graph Connector indexing has latency — trade data may be minutes behind the system of record. Skills must surface the index timestamp.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks populated by Power Automate flows from the third-party system. Common bridge patterns for capital markets:
- Daily SSI snapshots exported to SharePoint from the SSI repository
- Break reports exported from the OMS to a SharePoint document library
- Counterparty reference data maintained in a SharePoint list synced from the CRM
- Settlement cutoff schedules maintained as SharePoint documents updated by operations control

**Tier 3 — Manual Input with Templates**

For systems with no integration path, skills provide structured intake via prompts that capture data from manual lookups, writing it into the shared Excel tracker. The framework's Signal Inventory phase identifies exactly which data points are needed, so the skill can prompt for only what is missing.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for settlement rules, routing matrix, and break taxonomy. Use Tier 3 (manual input) for trade-specific details that analysts look up in the OMS. Plan Graph Connectors for Wave 3 after skill workflows are proven and the tenant admin team has evaluated connector options for the firm's specific OMS and settlement platforms.

### 3.5 Governance in Cowork — Capital Markets Regulatory Requirements

Capital markets governance extends far beyond the framework's generic model. Cowork skill design must address specific regulatory and compliance requirements:

| Governance Domain | Cowork Implementation | Capital Markets Regulatory Requirement |
|---|---|---|
| **SEC/FINRA compliance** | Skill guardrails prohibit any output that implies a settlement commitment or booking correction; all drafts reviewed before delivery | SEC Rule 15c6-1 (T+1 settlement), FINRA Rules 11860 (COD/DVP), trade reporting obligations |
| **Information barriers (Chinese walls)** | Skill instructions explicitly prohibit cross-desk data sharing; routing logic checks desk authorization before sending Teams messages | SEC/FINRA information barrier requirements; firms must prevent flow of MNPI between business units |
| **MNPI protection** | Skills never include position sizes, pricing data, or unreleased corporate action information in communications; guardrails flag potential MNPI content | Insider trading regulations; Regulation FD; firm compliance policies |
| **Trade surveillance** | All skill outputs include break ID, trade ID, timestamps, and actor identity for audit trail; no skill may suppress or alter break records | FINRA Rule 3110 (supervision), SEC Rule 17a-4 (recordkeeping) |
| **Best execution** | Skills do not make or imply execution quality assessments; break classification is limited to operational categories | SEC Rule 606, FINRA Rule 5310 (best execution) |
| **T+1 settlement deadlines** | Every skill output surfaces settlement date and cutoff proximity; same-day breaks are auto-flagged critical | SEC Rule 15c6-1; industry move to T+1 increases urgency of break resolution |
| **Recordkeeping** | Every skill write action logged with timestamp, actor, prior value, and break linkage; all artifacts retained in SharePoint | SEC Rule 17a-4, FINRA Rule 4511 (books and records) |
| **Access control** | Skills respect M365 RBAC; sensitive fields (client accounts, positions) masked in outputs; SharePoint permissions govern break case folder access | Firm-level data classification; need-to-know restrictions on trading data |

**The main governance gap** is that Cowork does not enforce information barriers at the platform level. The skill author must encode Chinese wall restrictions in skill instructions — for example, "do not route equity desk break details to fixed income personnel" — and rely on SharePoint permissions and M365 RBAC to restrict data access. For firms with strict information barrier requirements, this must be supplemented by the firm's existing compliance monitoring tools.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for capital markets:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis (8-10 should-trigger and 8-10 should-not-trigger phrases per skill)
- Output quality — do context packets contain accurate, cited trade data? Assessed via manual review against OMS source records
- Tool success rate — do M365 tool calls return expected results within SLA windows? Assessed via dry-run testing with realistic break scenarios
- Classification accuracy — does `cm-break-classifier` agree with expert analyst classification? Target: at least 85 percent

**Process-level evaluation (end-to-end):**
- Triage cycle time — time from break intake to "review-ready" status; target: under 30 minutes for standard breaks
- Break classification accuracy — percentage of breaks correctly categorized; target: at least 85 percent
- Fail-risk detection rate — percentage of actual fail risks correctly flagged; target: at least 85 percent
- Incorrect routing rate — percentage of breaks sent to wrong desk; target: below 5 percent
- Unauthorized actions — any skill output that implies a booking correction or settlement commitment; target: zero
- Missing audit fields — any break case record with incomplete traceability; target: zero
- Draft handoff acceptance rate — percentage of `cm-break-comms` drafts sent with minor edits only; target: at least 75 percent
- Break aging reduction — average break age compared to pre-pilot baseline
- Reroute rate — how often `cm-break-routing` assignments need to be changed by supervisors

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for capital markets. Start with one asset class or one operations desk for each wave.

### Wave 1 — Foundation (Intake, Context, Classification, Risk)

**Infrastructure setup:**
- Create the shared Excel break tracker workbook in SharePoint with standard columns (Break ID, Trade ID, Asset Class, Instrument, Counterparty, Settlement Date, Break Source, Break Type, Priority, Fail Risk, Status, Created Timestamp, Assigned To, Desk, Disposition)
- Upload break taxonomy, settlement cutoff schedules, routing matrix, and SSI reference documents to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-break case evidence (context packets, correspondence, investigation notes)
- Establish Power Automate flows to export daily break reports and SSI snapshots from source systems to SharePoint (Tier 2 bridge)

**Skills to build:**
- `cm-break-intake`
- `cm-context-packet`
- `cm-break-classifier`
- `cm-risk-assessment`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 20-30 real break cases from a single desk, covering standard breaks, same-day settlement breaks, and repeated counterparty patterns.

### Wave 2 — Routing and Communication

**Skills to build:**
- `cm-break-routing`
- `cm-break-comms`

**Promotions:**
- Promote `cm-break-classifier` to write-back mode (updates Excel tracker break type column after user confirmation)
- Promote `cm-risk-assessment` to write-back mode (updates priority and fail-risk columns after user confirmation)
- Promote `cm-break-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a scheduled prompt that checks for break cases approaching settlement cutoff without confirmed disposition and surfaces them as critical alerts
- Set up a daily scheduled prompt that reports break aging trends and identifies repeat counterparty patterns

**Operating posture:** AI draft plus approve for all routing and communication skills. Every output reviewed before action. Supervisor copied on all critical and high-priority routing notifications.

### Wave 3 — Complex Cases and Proactive Detection

**Enhancements:**
- Introduce Graph Connectors for OMS and settlement platform data if available at the tenant level
- Add bounded multi-system break packet assembly for complex or repeated fails — an orchestrated sequence that runs `cm-context-packet`, `cm-break-classifier`, and `cm-risk-assessment` in sequence and produces a comprehensive case package
- Add proactive detection via scheduled prompt: identify repeat counterparty mismatch patterns, cutoff proximity risks across the break book, and aged breaks trending toward fail
- Expand to additional asset classes or desks based on Wave 1-2 performance

**Measurement:**
- Triage cycle time target: under 30 minutes for standard breaks, under 15 minutes for same-day
- Break classification accuracy target: above 85 percent
- Fail-risk detection rate target: above 85 percent
- Incorrect routing rate target: below 5 percent
- Unauthorized trade or settlement actions: zero
- Draft handoff acceptance rate target: above 75 percent
- Break aging reduction target: 30 percent improvement over pre-pilot baseline

---

## Part 5: Generalizing the Approach — Capital Markets Artifact Pattern

Capital markets represents the **time-critical, regulatory-dense** end of the framework's applicability spectrum. Its natural artifact pattern is:

| Artifact Role | M365 Artifact | Why |
|---|---|---|
| Process state | Excel (break tracker with case lifecycle columns) | Break data is inherently tabular; operations teams already work in spreadsheets for exception tracking |
| Evidence trail | Word (context packets, investigation summaries) | Auditable evidence packages require structured documents with cited sources |
| Operational interface | Adaptive Card (classification, risk, routing dashboards) | Time sensitivity demands immediate, in-context decision support — not document navigation |
| Reference data | SharePoint (taxonomy, routing matrix, cutoff schedules, SSI reference) | Operational policies must be centrally managed and version-controlled |
| Communication | Outlook drafts + Teams messages | Counterparty communications require review before send; internal coordination happens in real time |

The generalizable lesson from capital markets is that **time sensitivity drives artifact format selection**. In HR, the primary output is a Word document because reviewers have days to read it. In capital markets, the primary operational output is an Adaptive Card because desk analysts have minutes. The Word document still exists — but as the audit trail, not the decision interface.

For any LOB applying this framework where SLA pressure is high, the recommendation is: use Adaptive Cards for triage-speed decisions, Excel for state tracking, Word for auditable evidence, and Outlook drafts for communications that require review. SharePoint holds the reference data that grounds every output.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Capital Markets Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing the break tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill; capital markets steps must not combine triage and action |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability with regulatory guardrails |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace OMS, settlement platform, and SSI repository references via Graph Connectors or SharePoint bridge |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation; capital markets adds cutoff-aware scheduling via scheduled prompts |
| Agent | Subagent (general-purpose or deep-research) | Deferred to Wave 3 for complex multi-system packet assembly only |
| Policy or Guardrail | Guardrails section in SKILL.md | Capital markets guardrails must address information barriers, MNPI, settlement commitments, and audit trail — more restrictive than any other vertical |
| Process State | SharePoint-hosted Excel workbook | Break tracker is the canonical state; every skill reads and writes to it |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) + Graph Connectors | Trade system signals require connectors; email and Teams provide supplementary context |
| Approval | Confirmation gates + `CreateDraftMessage` + supervisor notification | Capital markets adds mandatory supervisor review for critical breaks and escalations |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via triage cycle time, classification accuracy, fail-risk detection, and routing accuracy |
