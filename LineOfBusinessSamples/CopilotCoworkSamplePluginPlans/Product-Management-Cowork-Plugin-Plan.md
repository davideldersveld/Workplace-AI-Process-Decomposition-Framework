# Plan: Product Management — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Product Management line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Feature Request Intake and Opportunity Framing pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for Product Management, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly; Product Management's medium error tolerance (detectable before roadmap commitment) makes it a safe pilot candidate |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps well; synthesis and framing steps are naturally separable from intake and routing |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "CRM," "product analytics," "backlog system," and "support case platform" that must become Graph Connector references or SharePoint bridge data |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md | Strong — Product Management's guardrails center on preventing premature commitments and ensuring product managers retain framing and prioritization authority |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — demand clustering maps to Decision Support; opportunity brief drafting maps to Content Generation; context assembly maps to Data Aggregation |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good — Product Management produces more document artifacts (briefs, decks, review packets) than most LOBs, which aligns well with Cowork's document generation capabilities |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — Cowork provides layers 1-5 and 7-9 natively; the skill author controls capability definition and decision logic |

### Key Insight

Product Management is the most document-centric LOB in the framework sample set. The primary value of the Cowork plugin is not real-time coordination (like ITSM) or evidence management (like Security), but **synthesis and artifact generation** — turning fragmented demand signals into structured, evidence-backed opportunity briefs and review packets. This makes it an excellent fit for Cowork's content generation capabilities (Word, PowerPoint, Excel). The main guardrail concern is not safety or sensitivity (as in Security) but **preventing premature commitment** — ensuring that AI-generated briefs do not get treated as approved roadmap items before product leadership has reviewed them.

---

## Part 2: The Product Management Plugin — Skill-by-Skill Design

The Product Management sample decomposes "Feature Request Intake and Opportunity Framing" into 7 steps (PM-001 through PM-007), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| PM-001: Normalize feature request event | `pm-request-intake` | Data Aggregation | Deterministic automation | Excel (opportunity tracker), SharePoint (list), Teams (intake channel) |
| PM-002: Gather product, customer, and telemetry context | `pm-context-packet` | Data Aggregation | AI act within policy | Word (context packet), SharePoint (product data), Excel (customer data) |
| PM-003: Detect duplicates and cluster related demand | `pm-demand-cluster` | Decision Support | AI assist | Adaptive Card (cluster report), Excel (tracker update with duplicate flags) |
| PM-004: Draft opportunity statement and open questions | `pm-opportunity-brief` | Content Generation | AI draft + approve | Word (opportunity brief), Excel (tracker update) |
| PM-005: Prepare stakeholder review packet | `pm-review-packet` | Data Aggregation + Content Generation | AI draft + approve | PowerPoint (review deck), Word (evidence summary), Excel (tracker update) |
| PM-006: Route to reviewers | `pm-reviewer-router` | Decision Support | AI act within policy | Teams (messages), Outlook (review requests), Calendar (review meetings) |
| PM-007: Confirm disposition | *Not a skill — human approval step* | N/A | Human only | Teams (disposition confirmation), Calendar (prioritization session) |

### Detailed Skill Designs

#### 1. `pm-request-intake` — Normalize Feature Request Event

**Framework Step:** PM-001

**Trigger phrases:** "new feature request from", "log opportunity for", "intake this request", "customer asked for", "create opportunity case for"

**Inputs:**
- Request description (from email, Teams message, customer call notes, or intake form)
- Source channel (sales escalation, support case, customer advisory board, internal stakeholder, analytics signal)
- Requesting party (customer name, account, internal team)
- Product area (if known)

**M365 tools:**
- `SearchPeople` — resolve requesting party and account contacts
- `GetUserDetails` — pull requester profile and org context
- `SearchM365(sources=["email"])` — find the original request thread if submitted via email
- `SearchM365(sources=["teams"])` — find related discussion threads in product channels
- `ReadFileContent` — read SharePoint-hosted intake form template for field normalization
- `GetDriveChildren` — check existing opportunity tracker for potential duplicates

**Output:** Structured opportunity case record written to Excel opportunity tracker in SharePoint; confirmation via Adaptive Card showing normalized fields

**Artifact:** Excel workbook with columns: Opportunity ID, Request Source, Requester, Customer/Account, Product Area, Summary, Status, Created Date, Assigned PM (pending), Theme (pending), Duplicate Flag, Evidence Links

**Guardrails:**
- Flag potential duplicates within the tracker based on product area and keyword similarity, but never auto-merge without PM confirmation
- Validate that required fields (requester, request description, source channel) are present before writing
- Auto-generate Opportunity ID using date and source prefix (e.g., PM-20260523-SALES-001)
- Confirm details with user via Adaptive Card before writing to tracker
- Never interpret a feature request as a commitment or promise — the intake record explicitly notes "Status: Intake - Not Committed"
- Scheduled prompt variant: monitor the product intake Teams channel and product feedback email alias for unprocessed requests daily

---

#### 2. `pm-context-packet` — Gather Product, Customer, and Telemetry Context

**Framework Step:** PM-002

**Trigger phrases:** "build context for this request", "gather product context for", "what do we know about this opportunity", "assemble evidence for this feature request"

**Inputs:**
- Opportunity ID or request summary from the tracker

**M365 tools:**
- `GetUserDetails` — requester profile and organizational context
- `SearchM365(sources=["files"])` — prior opportunity briefs, product strategy documents, roadmap principles, product glossary, feedback taxonomy in SharePoint
- `ReadFileContent` — SharePoint-hosted product area guides, customer segment profiles, feature taxonomy, and backlog history
- `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])` — related backlog items and their status from the product backlog system via Graph Connector (if available)
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — customer account details, tier, ARR, churn risk from CRM via Graph Connector (if available)
- `SearchM365(sources=["email"])` — recent email threads mentioning the customer or feature area
- `SearchM365(sources=["teams"])` — related discussions in product, sales, and support Teams channels

**Output:** Word document containing:
- Request context and source details
- Customer profile (account tier, ARR, churn risk, prior requests)
- Product area context (current roadmap position, recent launches, known gaps)
- Related backlog items (existing features, planned work, declined requests)
- Telemetry summary (usage data, adoption metrics, support ticket volume for this area — if available)
- Prior decisions on similar requests (approved, declined, deferred — with rationale)

**Artifact:** Word (.docx) saved to SharePoint product opportunities folder; linked from the Excel tracker row

**Guardrails:**
- Cite source document and retrieval timestamp for every data point
- Clearly distinguish confirmed data (from systems) from inferred context (from email/chat search)
- Flag if any critical context source (CRM, backlog, telemetry) returned no data or is unavailable
- Never include revenue projections or financial commitments in the context packet — only factual customer tier and account data
- If customer data is sensitive (enterprise contract, NDA-protected feedback), note the sensitivity and restrict distribution to the assigned PM

---

#### 3. `pm-demand-cluster` — Detect Duplicates and Cluster Related Demand

**Framework Step:** PM-003

**Trigger phrases:** "check for duplicates", "cluster related requests", "is this a duplicate", "group similar feature requests", "find related opportunities"

**Inputs:**
- Current opportunity case (from tracker)
- Full opportunity tracker (Excel workbook)
- Backlog history (SharePoint or Graph Connector)

**M365 tools:**
- `ReadFileContent` — opportunity tracker, feature taxonomy, and backlog status from SharePoint
- `SearchM365(sources=["files"])` — prior opportunity briefs, declined request records, and backlog items matching keywords
- `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])` — related backlog items and their current status

**Output:** Clustering recommendation as Adaptive Card showing:
- Exact duplicates found (same customer, same feature, same timeframe) with links
- Related demand signals (similar theme, different customer or channel) with links
- Demand volume count (how many unique sources have requested this or similar features)
- Suggested theme grouping (e.g., "Mobile Experience" or "Reporting Enhancements")
- Confidence level for each cluster match

**Logic:** Compare the current request's product area, keywords, and customer segment against the full opportunity tracker and backlog. Identify exact duplicates (same feature from different channels), near-duplicates (similar features with different framing), and thematic clusters (related requests that could form a unified opportunity).

**Guardrails:**
- Present clustering as recommendations only — never auto-merge or auto-link opportunities without PM confirmation
- Always show the basis for each cluster match (which keywords, which product area, which customer overlap)
- If confidence is low (below 50%), flag the match as "possible" rather than "likely"
- Never auto-close a request as a duplicate — only flag and recommend, letting the PM decide
- Preserve the original request text even when linking to a cluster — do not overwrite or summarize away the requester's exact words

---

#### 4. `pm-opportunity-brief` — Draft Opportunity Statement and Open Questions

**Framework Step:** PM-004

**Trigger phrases:** "draft opportunity brief", "write the opportunity statement", "frame this as an opportunity", "create the problem statement for", "draft the brief for this feature request"

**Inputs:**
- Context packet (Word document)
- Demand cluster output
- Opportunity brief template (SharePoint document)
- Product strategy principles (SharePoint document)

**M365 tools:**
- `ReadFileContent` — opportunity brief template, product strategy principles, prioritization rubric, and exemplar briefs from SharePoint
- `SearchM365(sources=["files"])` — approved opportunity briefs as exemplars for tone, structure, and depth

**Output:** Word document following the organization's opportunity brief template, containing:
- Opportunity title and ID
- Problem statement (user-centric, evidence-backed)
- Business impact (customer reach, revenue potential, strategic alignment)
- Evidence summary (demand volume, customer quotes, telemetry indicators)
- Open questions (unknowns, dependencies, risks, areas needing discovery)
- Traceability section (links to source requests, context packet, cluster analysis)

**Artifact:** Word (.docx) saved to SharePoint product opportunities folder; linked from the Excel tracker row; tracker status updated to "Brief Drafted"

**Guardrails:**
- Always generate as a draft for PM review — never publish or distribute without explicit PM confirmation
- Follow the organization's brief template structure exactly; do not invent new sections
- Every claim in the brief must trace to a source in the context packet or demand cluster output
- Clearly separate confirmed facts from hypotheses and open questions
- Never include delivery estimates, timelines, or commitment language — the brief is a problem framing artifact, not a project plan
- Never state that a feature "will be built" or "is planned" — use language like "is being evaluated" or "is under consideration"
- Include a prominent "DRAFT - NOT COMMITTED" watermark instruction in the document header
- Respect the framework's "AI draft plus approve" boundary — the PM reviews, edits, and approves before the brief advances

---

#### 5. `pm-review-packet` — Prepare Stakeholder Review Packet

**Framework Step:** PM-005

**Trigger phrases:** "prepare review packet", "build the review deck", "package this for review", "create the stakeholder packet for", "get this ready for the review meeting"

**Inputs:**
- Opportunity brief (Word document)
- Context packet (Word document)
- Demand cluster analysis
- Reviewer list and review template (SharePoint documents)

**M365 tools:**
- `ReadFileContent` — review packet template, reviewer checklist, and approved exemplar packets from SharePoint
- `SearchM365(sources=["files"])` — supporting evidence documents (customer feedback exports, telemetry reports, competitor analysis)

**Output options:**
- **PowerPoint deck** — Stakeholder review presentation with: opportunity summary, problem statement, evidence highlights, demand volume, open questions, recommended next steps, and traceability links. Formatted for a 15-minute review discussion.
- **Word document** — Detailed review packet with full evidence appendix for reviewers who prefer reading to presentation
- **Excel update** — Tracker status set to "Review Ready" with reviewer assignments

**Artifact:** PowerPoint (.pptx) and/or Word (.docx) saved to SharePoint product opportunities folder

**Guardrails:**
- Always generate as a draft for PM review — never distribute to reviewers without explicit PM confirmation
- Every evidence claim in the deck must trace to a source document
- Include the opportunity ID and "DRAFT" label on every slide
- Clearly distinguish confirmed evidence from hypotheses
- Never include delivery estimates, cost projections, or commitment language in the review packet
- Match the organization's review deck template if one exists in SharePoint
- Include an "Open Questions" slide that surfaces unknowns rather than hiding them
- Respect the framework's "AI draft plus approve" boundary — the PM reviews and approves the packet before it goes to stakeholders

---

#### 6. `pm-reviewer-router` — Route to Reviewers

**Framework Step:** PM-006

**Trigger phrases:** "send this for review", "route to reviewers", "distribute the review packet", "schedule the review for this opportunity", "get feedback from the team on this"

**Inputs:**
- Review packet (PowerPoint and/or Word)
- Reviewer matrix (SharePoint document listing which roles review which product areas)
- Product area and opportunity scope

**M365 tools:**
- `ReadFileContent` — reviewer matrix, review process guide from SharePoint
- `SearchPeople` — resolve reviewer identities by role (product lead, design lead, engineering lead, GTM lead)
- `GetUserDetails` — verify reviewer availability and current role
- `PostMessage` — Teams notification to reviewers with review packet summary and link
- `CreateDraftMessage` — Outlook draft for formal review request with packet attached (for cross-functional reviewers who prefer email)
- `CreateEvent` — schedule review meeting with appropriate attendees
- `ListCalendarView` — check reviewer availability for scheduling the review session

**Output:** Routed review tasks:
- Teams messages to each reviewer with opportunity summary and link to review packet in SharePoint
- Outlook draft for formal review request (if cross-functional review is needed)
- Calendar event for the review session
- Updated Excel tracker with reviewer assignments and review deadline

**Guardrails:**
- Only route to reviewers listed in the approved reviewer matrix for the relevant product area
- If no matching reviewer exists for a required role, escalate to the group product manager rather than skipping the review
- Present routing plan for PM review before sending any messages or creating calendar events
- Include the opportunity ID and review deadline in every outbound message
- Never share the review packet outside the designated reviewer group without PM authorization
- Attach the review packet link (SharePoint URL), not the document itself, to enable version control
- For sensitive opportunities (competitive intelligence, unreleased strategy), verify that all reviewers have appropriate access before routing

---

#### Step 7: Confirm Disposition (Human Only)

**Framework Step:** PM-007

This is not a Cowork skill. The framework correctly identifies final disposition as a human-only step. Prioritization readiness, roadmap decisions, and commitment authority must remain with product leadership. In Cowork, this step is supported by:

- The review packet artifacts from `pm-review-packet` — provides the evidence package for the reviewer group
- The review meeting scheduled by `pm-reviewer-router` — provides the forum for the disposition decision
- The `pm-request-intake` skill — can update the tracker status to "Accepted," "Deferred," or "Declined" after the human decision is communicated

The disposition step includes: confirming the opportunity is ready for prioritization, deciding whether to proceed, defer, or decline, recording the rationale, and communicating the decision back to the original requesters. These are product strategy decisions that must remain human-owned.

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. For Product Management, this challenge is moderate — product work is naturally document-centric, and M365's document generation capabilities (Word, PowerPoint, Excel) are a strong fit. The main translation challenge is bridging to product-specific systems (backlog, CRM, analytics) that hold the evidence data. This section analyzes how to approach that translation systematically.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Product Management is **pre-implementation discipline for synthesis work**:

**Process decomposition prevents synthesis sprawl.** The common Cowork anti-pattern in Product Management would be building one "create an opportunity brief" skill that tries to intake, gather context, cluster, frame, package, and route in a single invocation. The framework's decomposition separates these into focused skills where context gathering is independent of framing, and framing is independent of packaging.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation for Product Management |
|---|---|
| Human only | Do not build a skill; support disposition and prioritization decisions with review artifacts only |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions (used for duplicate detection and demand clustering) |
| AI draft + approve | Skill produces documents but uses draft mode, shows output before publishing, requires explicit PM confirmation (used for opportunity briefs and review packets) |
| AI act within policy | Skill can execute bounded actions (update tracker, route to reviewers) within defined reviewer matrix rules |
| Deterministic automation | Scheduled prompt for stable request intake normalization |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "check the backlog," the framework requires naming every input source. This translates to specific MCP tool calls and connector references.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which process step is active |
| Capability registry | Built-in — `/mnt/user-config/.claude/skills/` IS the registry |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in for M365 data; product-specific data (CRM, analytics, backlog) requires Graph Connectors or SharePoint bridge |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; product strategy governance requires PM oversight of generated artifacts |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through quality rubric and process metrics |

**The key gap for Product Management:** Cowork does not have native access to product analytics (usage data, adoption metrics, churn indicators) or backlog systems (Jira, Azure DevOps, Productboard). These are the evidence systems that make opportunity briefs credible. Without them, the skills can still synthesize from email, Teams, and SharePoint-stored data, but the evidence quality is lower. Graph Connectors or SharePoint bridge patterns are essential for a high-quality pilot.

### 3.3 M365 Artifacts as Process State

For Product Management, the primary M365 artifacts are **Word + PowerPoint + Excel**, reflecting the synthesis-heavy, document-centric nature of product opportunity work.

| Artifact | Role in Product Management | How Skills Use It |
|---|---|---|
| **Word** | Primary synthesis artifact (context packets, opportunity briefs, review summaries) | Skills generate the core product artifacts — opportunity briefs are Word documents that follow the organization's template; context packets provide the evidence foundation |
| **PowerPoint** | Decision-support artifact (stakeholder review decks, opportunity presentations) | Skills create review presentations that support the human prioritization step — reviewers evaluate a deck, not raw data |
| **Excel** | Process state store (opportunity tracker, demand volume log, reviewer assignments) | The opportunity tracker workbook IS the Cowork-accessible process state — skills read current status, write updates, track review progress, and log demand clustering results |
| **SharePoint** | Source of truth for templates and evidence (brief templates, strategy documents, feedback exports, product glossary, reviewer matrices) | Skills read templates, exemplars, and reference data from SharePoint; generated artifacts are stored here for version control |
| **Teams** | Coordination channel (reviewer notifications, intake discussions, cross-functional feedback) | Skills post review requests and opportunity summaries to product Teams channels; intake monitoring |
| **Outlook** | Communication channel for formal review requests and requester follow-up | Skills draft review request emails and status update messages as reviewable Outlook drafts |
| **Calendar** | Review session scheduling and intake deadlines | Skills create review meeting events and track review cycle timelines |
| **Graph API** | People and org data (reviewer identities, requester profiles, cross-functional contacts) | Skills resolve reviewers, requesters, and stakeholders for routing and context |
| **Adaptive Card** | In-flow decision support (demand cluster visualization, duplicate alerts) | Skills present clustering results and duplicate flags as structured cards for quick PM review |

**The design pattern:** The Product Management plugin uses a SharePoint-hosted Excel workbook as the opportunity tracker, with Word documents as the primary synthesis artifacts (opportunity briefs, context packets) and PowerPoint decks as the review-ready decision support artifacts. This is a document-generation-heavy pattern — the plugin's core value is turning fragmented signals into polished, traceable, review-ready documents that product managers can confidently present to stakeholders.

### 3.4 Federated Connectors for Third-Party Systems

Product Management depends on several external systems for evidence and context. The framework references "CRM," "product analytics," "backlog system," and "support case platform." The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For backlog systems (Jira, Azure DevOps, Productboard), Graph Connectors index feature items, epics, and their status into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])`. This provides read access to existing backlog items, planned features, and declined requests for deduplication and context.

For CRM platforms (Salesforce, HubSpot, Dynamics 365), Graph Connectors index account data, opportunity records, and customer interactions. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` for customer tier, ARR, and churn risk data.

For product analytics platforms (Amplitude, Mixpanel, Pendo), connector availability is limited. When available, they index usage summaries and feature adoption metrics.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- **Customer context**: A Power Automate flow exports customer account summaries (tier, ARR, segment) from CRM to a SharePoint list on a weekly basis.
- **Backlog snapshot**: Active and recently closed backlog items exported to a SharePoint Excel workbook, refreshed daily.
- **Telemetry summaries**: Product analytics team publishes weekly usage summary reports to a SharePoint document library that skills can read.
- **Support case data**: Support ticket volume by product area exported to SharePoint for demand signal correlation.

**Tier 3 — Manual Input with Templates**

For data that cannot be bridged:
- The `pm-request-intake` skill prompts the PM for customer context (tier, account, prior requests) when CRM data is unavailable
- The `pm-context-packet` skill notes which evidence sources are missing and suggests the PM add them manually to the context packet
- All manual inputs are written to the tracker with a "manual entry" source tag

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint bridge for customer context and backlog snapshots) and Tier 3 (manual input for analytics data). If the organization uses Jira or Azure DevOps, prioritize the Graph Connector setup in Wave 2 — backlog deduplication is one of the highest-value capabilities.

### 3.5 Governance in Cowork

Product Management governance centers on preventing premature commitment and protecting roadmap confidentiality:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; the product operations manager or group PM owns the skill suite; individual PMs own their opportunity briefs |
| **Access** | M365 permissions govern data access; roadmap strategy documents and unreleased feature plans require restricted SharePoint access |
| **Commitment prevention** | Skill guardrails enforce non-commitment language: every draft brief includes "DRAFT - NOT COMMITTED" headers; skills never use "will be built" or "is planned" language; disposition decisions are always human-owned |
| **Roadmap confidentiality** | Review packets are distributed only to approved reviewers per the reviewer matrix; skills never post roadmap details to general Teams channels; competitive intelligence references are flagged for restricted distribution |
| **Traceability** | Every claim in an opportunity brief must link to a source signal (email, support case, customer request); the context packet serves as the evidence chain |
| **Audit** | Artifacts in SharePoint provide a document trail; the Excel tracker logs status changes with timestamps; reviewer assignments are recorded |
| **Template governance** | Brief templates, review checklists, and taxonomy definitions live in SharePoint documents that product operations can update without modifying skill code |

**The main governance gap** is version control across the synthesis chain. If the context packet is updated after the opportunity brief is drafted, the brief may contain stale evidence. The recommended mitigation is to timestamp all evidence citations in the brief and flag if the context packet has been modified since the brief was last generated.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Product Management:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis
- Duplicate detection quality — does `pm-demand-cluster` correctly identify duplicates and related demand? Assessed via PM review of 30+ clustering outputs
- Brief draft quality — does `pm-opportunity-brief` produce briefs that follow the template, cite evidence, and avoid commitment language? Assessed via PM review of 20+ generated briefs
- Review packet quality — does `pm-review-packet` produce decks suitable for stakeholder presentation? Assessed via reviewer feedback
- Tool success rate — do M365 tool calls and connector queries return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Duplicate detection accuracy — percentage of actual duplicates correctly identified (and percentage of false duplicate flags)
- Brief draft acceptance rate — percentage of generated briefs used by PMs without major rewrites
- Time to first review-ready brief — elapsed time from intake to "Review Ready" status (target: under 3 business days)
- Reviewer routing accuracy — percentage of briefs routed to the correct reviewer group
- Evidence traceability rate — percentage of briefs with source-linked evidence for every claim
- Override rate on clustering and framing decisions — how often PMs override skill recommendations
- Disposition cycle time — elapsed time from "Review Ready" to final disposition

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Product Management in Cowork:

### Wave 1 — Foundation and Intake

**Infrastructure setup:**
- Create the shared Excel opportunity tracker workbook in SharePoint with standard columns (Opportunity ID, Request Source, Requester, Customer/Account, Product Area, Summary, Status, Created Date, Assigned PM, Theme, Duplicate Flag, Evidence Links, Reviewer Assignments, Disposition)
- Upload opportunity brief template, review packet template, product glossary, feature taxonomy, and reviewer matrix to a dedicated SharePoint document library
- Upload exemplar opportunity briefs (3-5 approved past briefs) as reference documents for the drafting skill
- Create a SharePoint folder structure for per-opportunity evidence packages
- Set up the product intake Teams channel for request monitoring
- Create a Power Automate flow to sync customer account summaries from CRM to a SharePoint list (if feasible in Wave 1)

**Skills to build:**
- `pm-request-intake` — normalize incoming feature requests into the tracker
- `pm-context-packet` — assemble product, customer, and telemetry context
- `pm-demand-cluster` — detect duplicates and cluster related demand
- `pm-opportunity-brief` — draft opportunity statements and open questions

**Operating posture:** Deterministic for intake; AI act within policy for context gathering; AI assist for clustering; AI draft plus approve for opportunity briefs. All outputs are presented for PM review before any distribution. Test with 15-20 real feature requests over 2 weeks.

### Wave 2 — Review and Routing

**Skills to build:**
- `pm-review-packet` — prepare stakeholder review decks and evidence summaries
- `pm-reviewer-router` — route review packets to the appropriate reviewer group

**Promotions:**
- Promote `pm-request-intake` to write mode (creates tracker records after confirmation)
- Promote `pm-demand-cluster` to write-back mode (updates tracker with duplicate flags and theme assignments after PM confirmation)
- Promote `pm-reviewer-router` to "act within policy" mode for standard product areas with clear reviewer matrix matches

**Automation:**
- Set up a daily scheduled prompt to check for new unprocessed feature requests in the intake Teams channel and email alias
- Set up a weekly scheduled prompt to flag opportunities that have been in "Intake" status for more than 5 business days without advancing

**Graph Connector introduction:**
- If Jira or Azure DevOps is available, configure the Graph Connector for backlog item search (critical for deduplication quality)
- If CRM Graph Connector is available, configure for customer account context

**Operating posture:** AI draft plus approve for review packets. AI act within policy for routing with clear reviewer matrix matches. All generated documents require PM approval before stakeholder distribution.

### Wave 3 — Optimization and Proactive Synthesis

**Enhancements:**
- Introduce additional Graph Connectors for product analytics platforms if available
- Add proactive monitoring via scheduled prompt: identify opportunities with stale status, surface recurring request themes that have not been captured as opportunities, flag high-demand clusters that cross product areas
- Add cross-product duplicate detection for organizations with multiple product lines
- Add source-cited evidence summaries that pull the most relevant customer quotes, telemetry data points, and support ticket references into the brief automatically
- Refine all skills based on PM override patterns, reviewer feedback, and brief acceptance rates from Waves 1-2

**Measurement:**
- Time to first review-ready brief: target under 3 business days (vs. pre-pilot baseline)
- Brief draft acceptance rate target: above 70% (briefs used with minor or no edits)
- Duplicate detection accuracy target: above 80% true positive rate with less than 10% false positive rate
- Reviewer routing accuracy target: above 90% correct first-time routing
- Evidence traceability rate target: 100% of briefs have source-linked evidence
- Disposition cycle time target: 30% reduction vs. pre-pilot baseline

---

## Part 5: Generalizing the Approach — Product Management Artifact Pattern

Product Management's primary artifact pattern is **Word + PowerPoint + Excel** — synthesis documents (opportunity briefs, context packets) in Word, decision-support presentations (review decks) in PowerPoint, and structured tracking (opportunity pipeline) in Excel. This reflects the fundamental nature of product work: it is synthesis-heavy, document-centric, and culminates in review-ready artifacts that enable human decision-making.

This pattern fits the cross-LOB method as follows:

| Dimension | Product Management Pattern |
|---|---|
| Primary synthesis artifact | Word (opportunity briefs, context packets) |
| Primary presentation artifact | PowerPoint (stakeholder review decks) |
| Primary tracking artifact | Excel (opportunity tracker with structured columns) |
| Primary policy artifact | SharePoint documents (brief templates, product glossary, reviewer matrix, strategy principles) |
| Primary coordination artifact | Teams (reviewer notifications, intake channel) |
| Primary communication artifact | Outlook (formal review requests, requester follow-up — always drafts) |
| State management approach | Excel tracker as Cowork-accessible state, backlog system as source of truth for committed items, Power Automate for sync |
| Guardrail posture | Moderate — centered on preventing premature commitment, ensuring evidence traceability, and protecting roadmap confidentiality |

The decomposition and translation method used for HR onboarding applies directly to Product Management. The key differences are: (1) the output artifacts are more document-heavy (Word briefs and PowerPoint decks are the primary value), (2) the guardrail focus is on commitment prevention rather than safety or sensitivity, and (3) the evidence quality depends heavily on federated connector access to CRM and backlog systems.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references; product systems bridged via Graph Connectors |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint; commitment prevention is the primary guardrail theme |
| Process State | SharePoint-hosted Excel workbook | Durable state externalized to M365 artifacts; backlog system is the canonical source of truth for committed items |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) + Graph Connectors for CRM/backlog/analytics | Product demand signals come from email, Teams, and federated systems |
| Approval | Draft tools + Adaptive Card confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; prioritization and roadmap commitment always human-owned |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via brief acceptance rate, duplicate detection accuracy, and time to review-ready brief |
