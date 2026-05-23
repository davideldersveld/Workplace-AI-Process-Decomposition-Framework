# Plan: Corporate Communications Announcement Brief — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Corporate Communications line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Corporate Communications Announcement Request Intake and Message Brief Synthesis pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select → Decompose → Signal Inventory → Automation Boundary → Capability Map → Translate → Architect). To evaluate how this maps to Copilot Cowork skill ideation for Corporate Communications, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "work management platform", "brand and policy repository", and "approval workflow" that must become specific M365 tool names and SharePoint structures |
| **Automation Boundary** | Operating mode per step (human-only → deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but Cowork enforces this through instructions and approval dialogs, not a runtime policy engine; especially critical for Corp Comms where premature release is a top risk |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1–5 and 7–9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Corporate Communications workflows carry elevated sensitivity because they involve messages that represent the organization externally and to employees. The framework's decomposition is especially valuable here because brief creation is often treated as a single editorial task, but actually decomposes into six distinct steps with different data sources, sensitivity profiles, and approval requirements. The Cowork translation narrows each step to a focused skill that reads from approved messaging repositories in SharePoint, drafts using sanctioned language, and enforces multi-stakeholder review routing through confirmation gates — never auto-publishing or distributing messages that have not been through the required legal, HR, and executive review chain.

---

## Part 2: The Corporate Communications Plugin — Skill-by-Skill Design

The Corporate Communications sample decomposes "Announcement Request Intake and Message Brief Synthesis" into 6 steps (CC-001 through CC-006), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| CC-001: Normalize communications request | `comms-request-intake` | Data Aggregation | Deterministic automation | Excel (brief tracker), SharePoint (list), Outlook (notifications) |
| CC-002: Gather business, audience, and policy context | `comms-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (policy repository, announcement history), Graph API (people) |
| CC-003: Extract key message needs and risks | `comms-message-extraction` | Decision Support | AI assist | Excel (message themes matrix), SharePoint (request artifacts), Adaptive Card (risk flags) |
| CC-004: Draft message brief and open questions | `comms-brief-draft` | Content Generation | AI draft + approve | Word (message brief), SharePoint (templates), Adaptive Card (draft summary) |
| CC-005: Route legal, HR, executive, and sponsor reviews | `comms-review-routing` | Decision Support | AI act within policy | Teams (review messages), Outlook (review requests), Calendar (deadline holds) |
| CC-006: Confirm brief disposition | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `comms-request-intake` — Normalize Communications Request

**Framework Step:** CC-001

**Trigger phrases:** "new announcement request", "communications brief needed for [topic]", "log comms request", "set up announcement case for [event]"

**Inputs:**
- Announcement topic or title
- Requesting sponsor name and department
- Communication type (organizational change, product announcement, executive communication, crisis response, policy update, event announcement)
- Target audience (internal, external, both)
- Requested timing or embargo date

**M365 tools:**
- `SearchPeople` — resolve requesting sponsor and communications lead identities
- `GetUserDetails` — pull profile and department data for sponsor
- `SearchM365(sources=["email"])` — find the request thread or executive directive
- `SearchM365(sources=["files"])` — locate attached context documents, talking points, or prior drafts
- `ReadFileContent` — read intake details from SharePoint or email attachments
- `GetDriveChildren` — check existing brief tracker for duplicates or related active announcements

**Output:** Structured case record written to Excel brief tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Brief ID, Announcement Title, Sponsor, Department, Communication Type, Target Audience, Requested Date, Embargo Status, Sensitivity Level, Status, Created Date, Assigned Communications Manager, Required Reviewers

**Guardrails:**
- Never create duplicate cases for the same announcement topic and timing
- Validate that requested date allows for minimum review cycle time (SLA: 2 business days)
- Confirm details with user before writing to tracker
- Auto-flag communications types that require elevated review: crisis response, organizational change, executive communications, and anything marked as sensitive or embargoed
- Never expose embargo dates or sensitive topic details in channel-wide Teams posts — use direct messages only

---

#### 2. `comms-context-packet` — Gather Business, Audience, and Policy Context

**Framework Step:** CC-002

**Trigger phrases:** "build context for [announcement]", "assemble brief context", "what policies apply to [topic]", "pull announcement history for [topic area]"

**Inputs:**
- Brief case ID or announcement title
- Communication type and target audience

**M365 tools:**
- `GetUserDetails` — sponsor and communications team profiles
- `GetManagerDetails` — executive chain for escalation and approval routing
- `SearchM365(sources=["files"])` — brand guidelines, communications policies, channel guidance, prior announcement briefs, talking point archives
- `ReadFileContent` — SharePoint policy documents, brand voice guidelines, channel taxonomy, audience segment definitions
- `GetDriveChildren` — browse policy repository and announcement history library
- `SearchM365(sources=["email"])` — recent related correspondence for context on the announcement topic

**Output:** Word document containing:
- Announcement request summary and business context
- Applicable communications policies and brand voice guidelines
- Prior announcements on the same or related topics (with dates and channels used)
- Target audience profile and channel recommendations
- Sensitivity assessment (legal risk, employee impact, media exposure, executive visibility)
- Known stakeholders who must review or be informed
- Open questions and unresolved dependencies

**Artifact:** Word (.docx) saved to SharePoint brief workspace folder

**Guardrails:**
- Only surface content from the approved policy and brand voice repositories — never include draft policies or deprecated guidelines
- Cite source document and version for every policy element referenced
- Flag if any referenced policy document has been updated since the last announcement on this topic
- Explicitly identify any sensitive topic flags (litigation, personnel actions, regulatory matters, M&A) and note that these require legal and/or HR review
- Do not include internal-only context (such as HR investigation details or litigation strategy) in any artifact that might be shared externally

---

#### 3. `comms-message-extraction` — Extract Key Message Needs and Risks

**Framework Step:** CC-003

**Trigger phrases:** "extract message themes from [request]", "what are the key messages for [announcement]", "identify risks for [communication]", "parse announcement signals"

**Inputs:**
- Request artifacts (email threads, attached talking points, executive directives, sponsor briefings)
- Brief case data

**M365 tools:**
- `ReadFileContent` — read request documents and attached materials from SharePoint
- `SearchM365(sources=["email"])` — find related email threads with sponsor or executive clarifications
- `SearchM365(sources=["teams"])` — find related Teams discussions about the announcement topic
- `GetDriveChildren` — check brief folder for all related request artifacts

**Output:** Structured message themes and risk flags as Adaptive Card (for quick review) plus Excel worksheet (for tracking)

**Logic:** Parse request artifacts to extract: key message themes and talking points, target audience segments and channel implications, timing and sequencing requirements (who hears first, embargo dates), sensitivity indicators (media exposure, employee impact, regulatory relevance, executive visibility), dependencies on other announcements or events, conflicting messages or stakeholder expectations, and open questions requiring sponsor clarification.

**Artifact:** Excel worksheet with columns: Theme ID, Category (Key Message/Audience/Timing/Risk/Dependency), Theme Text, Source, Sensitivity Flag (Yes/No), Status (Confirmed/Unconfirmed/Conflicting), Assigned To, Notes

**Guardrails:**
- Present extracted themes and risk flags for user review before writing to the tracker — message interpretation in communications is high-judgment work
- Flag any message themes that reference sensitive topics (litigation, personnel changes, regulatory matters) with elevated visibility
- Never auto-resolve conflicting messages from different stakeholders — present conflicts for communications manager decision
- Preserve original sponsor or executive language alongside any summarized version — in communications, exact wording matters
- Flag if the requested timing conflicts with known embargo dates or other active announcements

---

#### 4. `comms-brief-draft` — Draft Message Brief and Open Questions

**Framework Step:** CC-004

**Trigger phrases:** "draft the communications brief", "create message brief for [announcement]", "build the announcement brief", "assemble comms brief"

**Inputs:**
- Message themes and risk flags (from `comms-message-extraction`)
- Context packet (Word document)
- Communications brief template from SharePoint
- Brand voice guidelines and approved messaging

**M365 tools:**
- `ReadFileContent` — all input artifacts from SharePoint
- `SearchM365(sources=["files"])` — communications brief templates, brand voice guidelines, prior approved briefs on similar topics
- Word generation skill (`docx`) — for the detailed message brief document

**Output options:**
- **Word document** — Structured message brief with sections: Executive Summary, Announcement Objective, Target Audience and Channels, Key Messages (with source citations), Timing and Sequencing Plan, Sensitivity Assessment and Risk Mitigation, Required Reviews (legal, HR, executive, sponsor), Open Questions, Appendix (prior related announcements)
- **Adaptive Card** — Draft status summary showing sections completed, sections needing sponsor input, sensitive sections flagged for elevated review, and unresolved open questions

**Guardrails:**
- Always create as draft — never finalize, distribute, or share with reviewers without explicit user confirmation
- Every key message must trace to a sponsor-provided source or approved messaging document — no fabricated organizational positions or unsupported claims
- Sensitive topic sections must be clearly marked and require explicit acknowledgment before inclusion in the brief
- Channel recommendations must align with the communications policy channel taxonomy — no ad hoc channel suggestions
- Timing recommendations must respect known embargo dates and sequencing rules (e.g., employees before media, board before public)
- Include source citations for every message element and policy reference used
- Match communications brief template formatting standards from the approved template library
- Never include specific personnel names in organizational change briefs without sponsor confirmation

---

#### 5. `comms-review-routing` — Route Legal, HR, Executive, and Sponsor Reviews

**Framework Step:** CC-005

**Trigger phrases:** "route brief for review", "send for legal review", "submit brief to executive review", "request HR review of announcement"

**Inputs:**
- Draft message brief
- Brief case data (communication type, sensitivity level, audience, channels)
- Review routing matrix from SharePoint (approval matrix and channel rules)

**M365 tools:**
- `ReadFileContent` — review routing matrix and approval rules from SharePoint
- `SearchPeople` — resolve reviewer identities by role and specialty
- `GetManagerDetails` / `GetDirectReportsDetails` — executive chain for escalation paths
- `PostMessage` — Teams direct message to reviewers with review context and scope (never channel posts for sensitive announcements)
- `CreateDraftMessage` — Outlook email with brief attached for formal review requests
- `CreateEvent` — calendar holds for review deadlines

**Output:** Routed review tasks:
- Teams direct messages to each required reviewer with announcement context and review scope
- Outlook draft emails with brief attachments for formal review chain
- Calendar deadline holds for review due dates
- Updated Excel tracker with review status per reviewer

**Logic:** Determine required reviewers based on announcement characteristics:
- All announcements: communications lead review and sponsor sign-off
- Announcements involving legal topics (litigation, regulatory, compliance): legal review
- Announcements involving people matters (organizational change, leadership changes, layoffs): HR review
- Announcements involving executives or representing the company externally: executive communications review
- Announcements with media exposure: media relations review
- Crisis or time-sensitive announcements: escalated review with shortened timelines

**Guardrails:**
- Present routing recommendations for communications manager review before sending any messages
- Route based strictly on the approval matrix — communication type, sensitivity level, and audience determine which reviewers are required
- Never skip a required reviewer even if the announcement appears routine
- Use Teams direct messages (not channel posts) for sensitive announcement review routing — embargo and sensitivity must be preserved
- Escalate to communications leadership if the approval matrix produces no clear routing for an announcement characteristic
- Include sensitivity level, audience scope, and timing in every review request message so reviewers can prioritize
- Never disclose embargoed content to anyone not on the approved reviewer list

---

#### Step 6: Confirm Brief Disposition (Human Only)

**Framework Step:** CC-006

This is not a Cowork skill. The framework correctly identifies final brief disposition as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the brief review meeting with stakeholders and reviewers
- The draft brief and review artifacts — provide the evidence package for the approver
- The `comms-review-routing` skill outputs — show review status across all required reviewers (legal, HR, executive, sponsor)
- The `comms-brief-draft` artifacts — the final message brief for disposition decision (approved, revised, or returned for rework)

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Corporate Communications is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle the entire communications brief process. The framework's rule — "keep breaking down until each step has one dominant goal" — separates request normalization from context assembly from message extraction from drafting, producing well-scoped skills that score high on the quality rubric's Scope Boundaries dimension.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather announcement context," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365(sources=["files"])` for policy documents and announcement history, `ReadFileContent` for brand voice guidelines, `SearchM365(sources=["email"])` for sponsor directives and stakeholder correspondence.

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

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (brief status, review decisions, embargo tracking) must be externalized to M365 artifacts — specifically the Excel brief tracker and the SharePoint brief workspace folder structure. For Corporate Communications, this gap is more consequential than in other LOBs because embargo enforcement and multi-stakeholder review sequencing require reliable state tracking.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the Corporate Communications brief process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Word** | Primary deliverable artifact (message briefs, talking points, context packets, executive summaries) | Skills generate the structured brief document that governs the announcement through review and release; Word IS the primary artifact for communications |
| **Outlook** | Communication channel, signal source, and formal review routing | Skills read sponsor directives and stakeholder correspondence; draft formal review request emails; capture executive and legal feedback; Outlook is the formal channel for sensitive review routing |
| **Teams** | Coordination channel and real-time routing (direct messages for sensitive topics) | Skills post review requests and status updates via direct message for sensitive announcements; use channel posts only for non-sensitive operational coordination |
| **Excel** | Process state store (brief tracker, message theme matrix, review status) | The brief tracker workbook IS the process state — skills read current status, write updates, track review progress and sensitivity flags |
| **SharePoint** | Source of truth (communications policies, brand voice guidelines, channel taxonomy, announcement history, brief templates) | Skills read policies, guidelines, and prior announcements from SharePoint; brief workspaces organize per-announcement artifacts |
| **Graph API** | People and org data (executive chain, reviewers, sponsors, media contacts) | Skills resolve reviewer identities, executive reporting chains for approval escalation, and communications team structure |
| **Calendar** | Time-bound process events (embargo dates, review deadlines, announcement timing) | Skills create review deadline holds and coordinate announcement timing with stakeholder calendars |

**The design pattern:** The Cowork Corporate Communications plugin uses a SharePoint-hosted Excel workbook as the canonical brief tracker, with a per-announcement folder in SharePoint as the workspace. Word documents are the primary artifact — the message brief IS the deliverable. Outlook handles formal review routing and captures the approval trail. Teams provides real-time coordination but is used carefully: direct messages for sensitive topics, never channel posts for embargoed content. This gives communications teams artifacts they already produce manually, with the critical addition of structured state tracking for the multi-stakeholder review process.

### 3.4 Federated Connectors for Third-Party Systems

The framework references "work management platform", "brand and policy repository", and "approval workflow" — these may live outside M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For media monitoring platforms (Meltwater, Cision), Graph Connectors index media coverage, sentiment data, and press mentions into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["meltwater-connector"])`. This provides read access to media landscape context for announcements with external exposure.

For work management platforms (Asana, Monday.com), Graph Connectors can index communications request records and project status. Skills access request data via `SearchM365(sources=["connectors"], connector_ids=["asana-connector"])`.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- Communications policies and brand voice guidelines maintained in a dedicated SharePoint library by communications operations
- Announcement history and talking point archives maintained in SharePoint, indexed by topic and date
- Review routing matrices maintained as SharePoint lists updated by communications operations
- Channel taxonomy and audience segment definitions maintained in SharePoint

Skills interact with the SharePoint copy. For media monitoring, periodic exports from the monitoring platform to SharePoint provide sufficient context for brief preparation.

**Tier 3 — Manual Input with Templates**

For real-time media landscape data, executive availability for approvals, and sensitive context that exists only in conversation, skills provide structured intake via Adaptive Card prompts that capture data from manual input, writing it into the shared Excel tracker.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for policies, guidelines, and announcement history. Use Tier 3 (manual input) for media context and executive availability. Introduce Tier 1 Graph Connectors in Wave 2 after the skill workflows are proven and media monitoring integration needs are validated.

### 3.5 Governance in Cowork

The framework's governance model maps to Cowork for Corporate Communications with specific attention to media sensitivity, executive approval chains, and premature release prevention:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document the communications operations team structure and escalation paths |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC; announcement-specific SharePoint permissions control workspace access for sensitive topics |
| **Data classification** | Skill guardrails enforce message sensitivity ("never expose embargoed content in channel posts"), executive communication handling ("do not share executive draft language beyond approved reviewers"), and personnel information controls |
| **Audit** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; brief tracker captures who reviewed what and when |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric (0–100 scoring) provides a pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions ("always draft before send", "use direct messages for sensitive topics", "enforce embargo dates", "route organizational changes through HR review") |

**Corporate Communications-specific governance concerns:**
- **Premature release** — Skills must never distribute, share, or post announcement content that has not completed the required review cycle. Every output is a draft until human disposition.
- **Embargo enforcement** — Skills must track and respect embargo dates. Embargoed content must never appear in channel-wide Teams posts, shared documents accessible to unauthorized users, or email distribution lists.
- **Executive language fidelity** — When executives provide specific language, skills must preserve it exactly and flag it as "executive-provided — do not paraphrase" in the brief.
- **Sensitive topic handling** — Announcements involving litigation, personnel actions, regulatory matters, or M&A require elevated review (legal, HR, executive) and must be handled with restricted distribution throughout the workflow.
- **Channel discipline** — Skills must respect the communications policy channel taxonomy. Internal announcements must not be shared through external channels; employee-first announcements must sequence internal distribution before any external release.
- **Multi-stakeholder review integrity** — Skills must not present a brief as "review complete" until all required reviewers have responded. Partial review status must be clearly visible.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Corporate Communications Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via 8–10 should-trigger and 8–10 should-not-trigger phrases per skill
- Output quality — do generated briefs contain accurate, cited messaging from approved sources and sponsor inputs? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Time to first review-ready brief — business days from intake to a brief routed for review (target: under 2 business days)
- Brief draft acceptance rate — percentage of `comms-brief-draft` outputs approved without major rewrites
- Reviewer routing accuracy — percentage of announcements routed to the correct reviewers by `comms-review-routing`
- Revision rounds per brief — average number of revision cycles before disposition
- Missed required-review incidents — incidents where a required reviewer was not routed
- Override rate on message-risk flags — how often communications managers override sensitivity flags
- Embargo compliance — zero incidents of embargoed content exposed before release date

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Corporate Communications Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel brief tracker workbook in SharePoint with standard columns (Brief ID, Announcement Title, Sponsor, Department, Communication Type, Target Audience, Requested Date, Embargo Status, Sensitivity Level, Status, Created Date, Assigned Communications Manager, Required Reviewers, Review Status)
- Organize the communications policy library in SharePoint with versioned policies, brand voice guidelines, channel taxonomy, and review routing matrices
- Set up a brief workspace folder template in SharePoint for per-announcement artifact storage
- Upload communications brief templates (Word format) to the template library
- Build the announcement history archive in SharePoint, indexed by topic and date

**Skills to build:**
- `comms-request-intake`
- `comms-context-packet`
- `comms-message-extraction`
- `comms-brief-draft`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5–10 real announcement requests, including at least 2 sensitive-topic cases.

### Wave 2 — Review Routing and Policy Checks

**Skills to build:**
- `comms-review-routing`

**Promotions:**
- Promote `comms-request-intake` to write mode (creates case records after confirmation)
- Promote `comms-message-extraction` to write-back mode (updates Excel message themes matrix after user confirmation)
- Promote `comms-context-packet` to full document generation (writes Word context packet to SharePoint)

**Additional capabilities:**
- Add review summary generation — synthesize reviewer feedback from email and Teams into a consolidated revision list
- Add policy-cited claim and channel checks — verify that every key message traces to an approved source and every channel recommendation aligns with the channel taxonomy

**Automation:**
- Set up a daily scheduled prompt that checks for briefs with approaching announcement dates and surfaces any with incomplete review status, unresolved risk flags, or active embargo constraints

**Operating posture:** AI draft plus approve for brief drafting and communications. AI act within policy for review routing (bound to the approval matrix). Heightened caution for sensitive-topic briefs — these remain in AI assist mode until trust is established.

### Wave 3 — Optimization and Proactive Intelligence

**Enhancements:**
- Introduce Graph Connectors for media monitoring platform data if available at the tenant level
- Add bounded multi-source announcement packet assembly for complex communications requiring talking points, FAQ documents, and channel-specific message variants
- Add proactive detection of likely review blockers (announcements with cross-functional dependencies, tight timelines, or novel sensitivity profiles)
- Refine all skills based on override patterns and reviewer feedback from Waves 1–2
- Add timing conflict detection — flag when a planned announcement date conflicts with other active announcements or organizational events
- Add missing stakeholder detection — identify when key reviewers referenced in the policy matrix have not been included in the review routing

**Measurement:**
- Time to first review-ready brief reduction vs. pre-pilot baseline (target: 35% reduction)
- Brief draft acceptance rate target: above 70%
- Reviewer routing accuracy target: above 95%
- Revision rounds target: 2 or fewer per brief on average
- Missed required-review incident rate target: zero
- Embargo compliance rate target: 100%

---

## Part 5: Generalizing the Approach — Corporate Communications Artifact Pattern

The Corporate Communications LOB's primary artifact pattern is **Word + Outlook + Teams**: Word documents are the governing brief deliverable, Outlook handles formal review routing and approval chains, and Teams provides real-time coordination with careful channel discipline for sensitive topics.

This pattern fits the cross-LOB method because:
1. **Decomposition** separated what appeared to be editorial synthesis into six distinct steps with different data sources, sensitivity profiles, and stakeholder ownership
2. **Automation boundaries** naturally clustered around the critical control points — message approval and release authority remained human-gated while context assembly and signal extraction became AI-assisted
3. **M365 artifacts** provided the natural state store — communications teams already work in Word for briefs, Outlook for formal review chains, and Teams for coordination; the Cowork plugin accelerates that existing workflow while adding structured state tracking that is often informal in current practice
4. **Federated connectors** addressed the media monitoring and work management dependencies through a pragmatic tiered approach rather than requiring full integration before the pilot could start
5. **Sensitivity-aware design** distinguished this LOB from others: the guardrail architecture explicitly addresses embargo enforcement, premature release prevention, and executive language fidelity — concerns that do not arise in most other LOB workflows

The method is repeatable: any LOB with synthesis-heavy, review-bound, sensitivity-gated workflows can follow the same decompose → boundary → map → build → evaluate cycle.

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
| Approval | CreateDraftMessage + PostMessage + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time and acceptance rate |
