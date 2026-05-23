# Plan: Marketing Campaign Brief Synthesis — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Marketing line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Marketing Campaign Request Intake and Brief Synthesis pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select → Decompose → Signal Inventory → Automation Boundary → Capability Map → Translate → Architect). To evaluate how this maps to Copilot Cowork skill ideation for Marketing, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "work management platform", "DAM", and "product marketing repository" that must become specific M365 tool names and SharePoint structures |
| **Automation Boundary** | Operating mode per step (human-only → deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but Cowork enforces this through instructions and approval dialogs, not a runtime policy engine |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1–5 and 7–9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Marketing campaign brief synthesis is a convergence workflow — it pulls fragmented requests, brand context, product messaging, audience data, and stakeholder inputs into a single structured deliverable. The framework's decomposition is especially valuable here because brief creation is often treated as creative synthesis done in one pass, but actually decomposes into six distinct steps with different data sources and risk profiles. The Cowork translation narrows each step to a focused skill that reads from approved brand assets in SharePoint, drafts using sanctioned messaging, and enforces brand and legal review routing through confirmation gates — never auto-publishing claims or bypassing required reviewers.

---

## Part 2: The Marketing Campaign Brief Plugin — Skill-by-Skill Design

The Marketing sample decomposes "Campaign Request Intake and Brief Synthesis" into 6 steps (MK-001 through MK-006), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| MK-001: Normalize campaign request | `mktg-campaign-intake` | Data Aggregation | Deterministic automation | Excel (campaign tracker), SharePoint (list), Outlook (notifications) |
| MK-002: Gather brand, product, and audience context | `mktg-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (brand guidelines, product messaging), Graph API (people) |
| MK-003: Extract goals, constraints, and dependencies | `mktg-signal-extraction` | Decision Support | AI assist | Excel (signal matrix), SharePoint (request artifacts), Adaptive Card (extraction summary) |
| MK-004: Draft campaign brief | `mktg-brief-draft` | Content Generation | AI draft + approve | PowerPoint (brief deck), Word (narrative brief), SharePoint (templates) |
| MK-005: Route brand, product, and legal reviews | `mktg-review-routing` | Decision Support | AI act within policy | Teams (review messages), Outlook (review requests), Calendar (deadline holds) |
| MK-006: Confirm brief baseline | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `mktg-campaign-intake` — Normalize Campaign Request

**Framework Step:** MK-001

**Trigger phrases:** "new campaign request", "log campaign intake for [name]", "set up campaign case", "campaign request from [stakeholder]"

**Inputs:**
- Campaign name or request title
- Requesting stakeholder name
- Campaign type (product launch, event, demand gen, brand, field marketing)
- Requested launch date
- Budget range (if provided)

**M365 tools:**
- `SearchPeople` — resolve requesting stakeholder identity
- `GetUserDetails` — pull profile and department data
- `SearchM365(sources=["email"])` — find the request thread or intake form submission
- `SearchM365(sources=["files"])` — locate attached briefs, decks, or request forms
- `ReadFileContent` — read intake details from SharePoint or OneDrive
- `GetDriveChildren` — check existing campaign tracker for duplicates

**Output:** Structured case record written to Excel campaign tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Campaign ID, Campaign Name, Requester, Department, Campaign Type, Requested Launch Date, Budget Range, Status, Created Date, Assigned Campaign Manager, Priority

**Guardrails:**
- Never create duplicate cases for the same campaign name and launch date
- Validate that requested launch date allows for minimum review cycle time (SLA: 3 business days)
- Confirm details with user before writing to tracker
- Flag if campaign type requires specialized review (e.g., regulated product claims, executive communications)

---

#### 2. `mktg-context-packet` — Gather Brand, Product, and Audience Context

**Framework Step:** MK-002

**Trigger phrases:** "build campaign context for [campaign]", "assemble brief context", "what brand assets apply to [campaign]", "pull product messaging for [product]"

**Inputs:**
- Campaign case ID or campaign name
- Product or solution area
- Target audience segment

**M365 tools:**
- `GetUserDetails` — requester and campaign team profiles
- `SearchM365(sources=["files"])` — brand guidelines, product messaging documents, audience research, prior campaign briefs
- `SearchM365(sources=["connectors"], connector_ids=["marketo-connector"])` — marketing automation data for audience segments and campaign performance history (via Graph Connector)
- `ReadFileContent` — SharePoint brand guidelines, approved messaging frameworks, campaign templates
- `GetDriveChildren` — browse brand asset library and product marketing repository

**Output:** Word document containing:
- Campaign request summary and business context
- Applicable brand guidelines and messaging framework
- Product messaging pillars and approved claims
- Target audience profile and segmentation data
- Relevant prior campaign history and performance insights
- Channel recommendations based on audience and campaign type
- Open questions and dependencies identified

**Artifact:** Word (.docx) saved to SharePoint campaign workspace folder

**Guardrails:**
- Only surface content from the approved brand and messaging repositories — never include draft, expired, or under-review assets
- Cite source document and version for every messaging element referenced
- Flag if brand guidelines have been updated since the last campaign in this product area
- Do not include internal competitive positioning in external-facing sections
- Flag if audience segment data is older than one quarter

---

#### 3. `mktg-signal-extraction` — Extract Goals, Constraints, and Dependencies

**Framework Step:** MK-003

**Trigger phrases:** "extract campaign requirements", "what are the goals for [campaign]", "parse campaign request signals", "identify campaign dependencies"

**Inputs:**
- Campaign request artifacts (email threads, attached briefs, intake forms)
- Campaign case data

**M365 tools:**
- `ReadFileContent` — read request documents and attached briefs from SharePoint
- `SearchM365(sources=["email"])` — find related email threads with stakeholder clarifications or scope changes
- `SearchM365(sources=["teams"])` — find related Teams discussions about the campaign
- `GetDriveChildren` — check campaign folder for all related request artifacts

**Output:** Structured signal matrix as Adaptive Card (for quick review) plus Excel worksheet (for tracking)

**Logic:** Parse request artifacts to extract: campaign objectives and KPIs, target audience definition, required deliverables and asset types, timing and milestone constraints, budget parameters, channel requirements, dependencies on other teams or campaigns, known blockers or risks, and stakeholder expectations.

**Artifact:** Excel worksheet with columns: Signal ID, Category (Objective/Constraint/Dependency/Risk), Signal Text, Source, Status (Confirmed/Unconfirmed/Conflicting), Assigned To, Notes

**Guardrails:**
- Present extracted signals for user review before writing to the tracker — interpretation of stakeholder intent is judgment-dependent
- Flag conflicting signals (e.g., aggressive timeline with extensive deliverable list, or broad audience with narrow budget)
- Never auto-resolve conflicting requirements — present conflicts for campaign manager decision
- Preserve original stakeholder language alongside any summarized version

---

#### 4. `mktg-brief-draft` — Draft Campaign Brief

**Framework Step:** MK-004

**Trigger phrases:** "draft the campaign brief", "create brief for [campaign]", "build the brief document", "assemble campaign brief"

**Inputs:**
- Signal matrix with extracted goals and constraints
- Context packet (Word document)
- Campaign brief template from SharePoint
- Approved messaging and brand guidelines

**M365 tools:**
- `ReadFileContent` — all input artifacts from SharePoint
- `SearchM365(sources=["files"])` — campaign brief templates, approved messaging blocks, prior successful briefs
- PowerPoint generation skill (`pptx`) — for campaign brief presentation decks
- Word generation skill (`docx`) — for detailed narrative campaign briefs

**Output options:**
- **Word document** — Structured campaign brief with sections: Executive Summary, Campaign Objective, Target Audience, Key Messages, Channel Strategy, Deliverables and Timeline, Budget, Success Metrics, Open Questions, Required Reviews
- **PowerPoint deck** — Campaign brief presentation for stakeholder alignment meetings with visual campaign framework, audience profiles, channel mix, and timeline
- **Adaptive Card** — Draft status summary showing sections completed, sections needing stakeholder input, and sections flagged for specialist review

**Guardrails:**
- Always create as draft — never finalize or distribute without explicit user confirmation
- Every claim and messaging element must trace to an approved brand or product messaging source — no fabricated value propositions or unsupported claims
- Flag any section where approved messaging does not fully cover the campaign's needs — these require product marketing input
- Budget sections must use ranges from the intake rather than specific figures unless confirmed by the campaign manager
- Include source citations for every messaging block and brand element used
- Match campaign brief template formatting standards from the approved template library

---

#### 5. `mktg-review-routing` — Route Brand, Product, and Legal Reviews

**Framework Step:** MK-005

**Trigger phrases:** "route brief for review", "send for brand review", "request legal review of campaign", "submit brief to reviewers"

**Inputs:**
- Draft campaign brief
- Campaign case data (type, audience, channels, claims made)
- Review routing matrix from SharePoint

**M365 tools:**
- `ReadFileContent` — review routing matrix and routing rules from SharePoint
- `SearchPeople` — resolve reviewer identities by role and specialty
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to reviewers with review context and scope
- `CreateDraftMessage` — Outlook email with brief attached for formal review requests
- `CreateEvent` — calendar holds for review deadlines

**Output:** Routed review tasks:
- Teams messages to each required reviewer with campaign context and review scope
- Outlook draft emails with brief attachments for formal review chain
- Calendar deadline holds for review due dates
- Updated Excel tracker with review status per reviewer

**Logic:** Determine required reviewers based on campaign characteristics:
- All campaigns: brand review
- Campaigns with product claims: product marketing review
- Campaigns in regulated industries or with legal-sensitive claims: legal review
- Campaigns targeting external audiences: communications review
- Campaigns above budget threshold: marketing leadership review

**Guardrails:**
- Present routing recommendations for campaign manager review before sending any messages
- Route based strictly on the review matrix — campaign type, audience, claims, and channels determine which reviewers are required
- Never skip a required reviewer even if the campaign appears straightforward
- Escalate to marketing operations if the review matrix produces no clear routing for a campaign characteristic
- Include campaign type, audience, and key claims in every review request message so reviewers can prioritize

---

#### Step 6: Confirm Brief Baseline (Human Only)

**Framework Step:** MK-006

This is not a Cowork skill. The framework correctly identifies final brief approval as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the brief review meeting with stakeholders
- The draft brief and review artifacts — provide the evidence package for the reviewer
- The `mktg-review-routing` skill outputs — show review status across all required reviewers
- The `mktg-brief-draft` artifacts — the final brief document for baseline approval

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Marketing is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle the entire brief creation process. The framework's rule — "keep breaking down until each step has one dominant goal" — separates request normalization from context assembly from signal extraction from drafting, producing well-scoped skills that score high on the quality rubric's Scope Boundaries dimension.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather campaign context," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365(sources=["files"])` for brand guidelines, `ReadFileContent` for messaging frameworks, `SearchM365(sources=["connectors"])` for marketing automation data.

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

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (brief status, review decisions, revision history) must be externalized to M365 artifacts — specifically the Excel campaign tracker and the SharePoint campaign workspace folder structure.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the Marketing campaign brief process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **PowerPoint** | Primary deliverable artifact (campaign brief decks, stakeholder alignment presentations) | Skills generate brief presentations for stakeholder review; the deck IS the alignment tool for cross-functional teams |
| **Word** | Narrative deliverable and evidence artifacts (detailed campaign briefs, context packets, messaging references) | Skills generate the structured brief document that becomes the campaign's governing document through execution |
| **SharePoint** | Source of truth (brand guidelines, approved messaging, campaign templates, audience research, DAM content) | Skills read brand assets, messaging frameworks, and templates from SharePoint; campaign workspaces organize per-campaign artifacts |
| **Excel** | Process state store (campaign tracker, signal matrix, review status) | The campaign tracker workbook IS the process state — skills read current status, write updates, track review progress |
| **Outlook** | Communication channel, signal source, and review routing | Skills read request emails for context; draft review request emails; capture stakeholder clarifications and scope changes |
| **Teams** | Coordination channel and real-time routing | Skills post review requests, campaign status updates, and deadline reminders to marketing channels or team chats |
| **Graph API** | People and org data (campaign team, reviewers, stakeholders) | Skills resolve reviewer identities, stakeholder org context, and marketing team structure |
| **Calendar** | Time-bound process events (launch dates, review deadlines, stakeholder meetings) | Skills create review deadline holds and stakeholder alignment meetings |

**The design pattern:** The Cowork Marketing plugin uses a SharePoint-hosted Excel workbook as the canonical campaign tracker, with a per-campaign folder in SharePoint as the workspace. PowerPoint and Word artifacts are the primary brief deliverables — the Word brief is the detailed governing document; the PowerPoint deck is the stakeholder alignment tool. This gives marketing teams artifacts they already produce manually, accelerating the brief-to-review cycle.

### 3.4 Federated Connectors for Third-Party Systems

The framework references "work management platform", "DAM", "CRM", and "product marketing repository" — these often live outside M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For marketing automation platforms (Marketo, HubSpot), Graph Connectors index campaign performance data, audience segments, and lead scoring into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["marketo-connector"])`. This provides read access to campaign history and audience insights without custom integration code.

For digital asset management platforms (Bynder, Brandfolder), Graph Connectors can index approved asset metadata. Skills search for relevant assets via `SearchM365(sources=["connectors"], connector_ids=["dam-connector"])`.

For work management platforms (Asana, Monday.com, Workfront), Graph Connectors can index campaign request records and project status. Skills access request data via `SearchM365(sources=["connectors"], connector_ids=["workfront-connector"])`.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- Brand guidelines and approved messaging documents maintained in a dedicated SharePoint library by brand operations
- Campaign templates maintained in SharePoint by marketing operations
- Audience research and segmentation data synced to SharePoint lists via Power Automate from the CRM or CDP
- Prior campaign performance summaries exported to SharePoint from the marketing automation platform

Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For real-time audience data, budget figures, and creative asset availability that require system access, skills provide structured intake via Adaptive Card prompts that capture data from manual lookups, writing it into the shared Excel tracker.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for brand guidelines, templates, and audience data. Use Tier 3 (manual input) for budget and timeline specifics. Introduce Tier 1 Graph Connectors in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

The framework's governance model maps to Cowork for Marketing with specific attention to brand compliance and claim controls:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document the marketing operations team structure and escalation paths |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC; campaign-specific SharePoint permissions control workspace access |
| **Data classification** | Skill guardrails enforce brand compliance ("only surface approved messaging"), claim controls ("never include unapproved product claims"), and audience data handling |
| **Audit** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; campaign tracker captures review history |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric (0–100 scoring) provides a pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions ("only use approved claims", "always draft before send", "route regulated-industry campaigns through legal") |

**Marketing-specific governance concerns:**
- **Brand consistency** — Skills must only surface messaging from the approved brand and product messaging repositories. Expired campaigns, draft messaging, or deprecated product names must be excluded.
- **Unapproved claims** — Skills must never generate product claims or value propositions that are not backed by approved messaging documents. This is especially critical for regulated industries (healthcare, financial services).
- **Audience data privacy** — Skills must not include personally identifiable audience data in brief documents. Audience references must use segment-level abstractions.
- **Scope creep visibility** — Skills must flag when extracted signals suggest scope expansion beyond the original request, ensuring campaign managers can make informed resource decisions.
- **Review completeness** — Skills must enforce that all required reviewers (brand, legal, product) have been routed before a brief can be baselined.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Marketing Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via 8–10 should-trigger and 8–10 should-not-trigger phrases per skill
- Output quality — do generated briefs contain accurate, cited brand and product messaging? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Time to first review-ready brief — business days from intake to a brief routed for review
- Brief draft acceptance rate — percentage of `mktg-brief-draft` outputs approved without major rewrites
- Review routing accuracy — percentage of campaigns routed to the correct reviewers by `mktg-review-routing`
- Revision rounds per brief — average number of revision cycles before baseline approval
- Unapproved-claim incidents — incidents of unapproved messaging appearing in brief drafts
- Launch delay rate — percentage of campaigns delayed due to missing brief inputs or incomplete reviews

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Marketing Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel campaign tracker workbook in SharePoint with standard columns (Campaign ID, Campaign Name, Requester, Department, Campaign Type, Requested Launch Date, Budget Range, Status, Created Date, Assigned Campaign Manager, Priority, Review Status)
- Organize the brand guidelines library in SharePoint with versioned brand books, messaging frameworks, and approved claims documents
- Set up a campaign workspace folder template in SharePoint for per-campaign artifact storage
- Upload campaign brief templates (Word and PowerPoint formats) to the template library

**Skills to build:**
- `mktg-campaign-intake`
- `mktg-context-packet`
- `mktg-signal-extraction`
- `mktg-brief-draft`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5–10 real campaign requests.

### Wave 2 — Review Routing and Policy Checks

**Skills to build:**
- `mktg-review-routing`

**Promotions:**
- Promote `mktg-campaign-intake` to write mode (creates case records after confirmation)
- Promote `mktg-signal-extraction` to write-back mode (updates Excel signal matrix after user confirmation)
- Promote `mktg-context-packet` to full document generation (writes Word context packet to SharePoint)

**Additional capabilities:**
- Add review summary generation — synthesize reviewer feedback from email and Teams into a consolidated revision list
- Add policy-cited claim coverage checks — verify that every claim in the draft brief traces to an approved source

**Automation:**
- Set up a daily scheduled prompt that checks for campaigns with approaching launch dates and surfaces any with incomplete review status or unresolved signals

**Operating posture:** AI draft plus approve for brief drafting and communications. AI act within policy for review routing (bound to the review matrix).

### Wave 3 — Optimization and Proactive Intelligence

**Enhancements:**
- Introduce Graph Connectors for marketing automation platform data if available at the tenant level
- Add bounded multi-source campaign packet assembly for complex campaigns with multiple channels and asset types
- Add proactive detection of likely review blockers (campaigns with novel claims, new audience segments, or cross-regional scope)
- Refine all skills based on override patterns and reviewer feedback from Waves 1–2
- Add brand guideline freshness monitoring — flag when brand assets referenced in briefs are approaching their review date
- Add missing stakeholder detection — identify when key stakeholders referenced in the request have not been included in the review routing

**Measurement:**
- Time to first review-ready brief reduction vs. pre-pilot baseline (target: 40% reduction)
- Brief draft acceptance rate target: above 70%
- Review routing accuracy target: above 95%
- Revision rounds target: 2 or fewer per brief on average
- Unapproved-claim incident rate target: zero

---

## Part 5: Generalizing the Approach — Marketing Artifact Pattern

The Marketing LOB's primary artifact pattern is **PowerPoint + Word + SharePoint**: PowerPoint decks serve as stakeholder alignment tools, Word documents are the detailed governing briefs, and SharePoint is both the source of truth for brand assets and the workspace for campaign artifacts.

This pattern fits the cross-LOB method because:
1. **Decomposition** separated what appeared to be creative synthesis into six distinct steps with different data sources and judgment requirements
2. **Automation boundaries** naturally clustered around the critical control points — brand claims and legal language remained human-gated while context assembly and signal extraction became AI-assisted
3. **M365 artifacts** provided the natural state store — marketing teams already work in SharePoint for brand assets, Word for briefs, and PowerPoint for stakeholder presentations; the Cowork plugin accelerates that existing workflow
4. **Federated connectors** addressed the marketing automation and DAM dependencies through a pragmatic tiered approach rather than requiring full integration before the pilot could start

The method is repeatable: any LOB with synthesis-heavy, review-bound workflows can follow the same decompose → boundary → map → build → evaluate cycle.

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
