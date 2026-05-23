# Plan: Sales Proposal and RFP Response Assembly — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the Sales line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Sales Proposal and RFP Response Assembly pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select → Decompose → Signal Inventory → Automation Boundary → Capability Map → Translate → Architect). To evaluate how this maps to Copilot Cowork skill ideation for Sales, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework references "CRM", "pricing system", and "content library" that must become specific M365 tool names and SharePoint structures |
| **Automation Boundary** | Operating mode per step (human-only → deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but Cowork enforces this through instructions and approval dialogs, not a runtime policy engine |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1–5 and 7–9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

Sales proposal workflows are document-heavy and approval-gated. The framework's decomposition is especially valuable here because proposal assembly is often treated as a single monolithic task ("build the proposal"), but actually decomposes into seven distinct steps with different owners, data sources, and risk profiles. The Cowork translation narrows each step to a focused skill that reads from CRM-synchronized data in SharePoint, drafts using approved content assets, and enforces pricing and legal guardrails through confirmation gates — never auto-committing customer-facing language or pricing.

---

## Part 2: The Sales Proposal Plugin — Skill-by-Skill Design

The Sales sample decomposes "Proposal and RFP Response Assembly" into 7 steps (SL-001 through SL-007), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| SL-001: Normalize opportunity request | `sales-proposal-intake` | Data Aggregation | Deterministic automation | Excel (proposal tracker), SharePoint (list), Outlook (notifications) |
| SL-002: Gather account, product, and content context | `sales-context-packet` | Data Aggregation + Content Generation | AI act within policy | Word (context packet), SharePoint (content library, CRM data), Graph API (people) |
| SL-003: Extract customer requirements | `sales-rfp-extraction` | Decision Support | AI assist | Excel (requirements matrix), SharePoint (RFP documents), Adaptive Card (requirement summary) |
| SL-004: Map requirements to approved responses and gaps | `sales-content-mapping` | Decision Support | AI assist | Excel (coverage matrix), SharePoint (approved content library), Adaptive Card (gap report) |
| SL-005: Draft proposal response | `sales-proposal-draft` | Content Generation | AI draft + approve | PowerPoint (proposal deck), Word (response document), SharePoint (templates) |
| SL-006: Route pricing, legal, and product approvals | `sales-approval-routing` | Decision Support | AI act within policy | Teams (review messages), Outlook (approval requests), Calendar (deadline holds) |
| SL-007: Confirm submission readiness | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `sales-proposal-intake` — Normalize Opportunity Request

**Framework Step:** SL-001

**Trigger phrases:** "new proposal request", "RFP received for [account]", "set up proposal case for", "log deal support request"

**Inputs:**
- Account name or CRM opportunity ID
- RFP document or proposal request email
- Due date
- Account executive name

**M365 tools:**
- `SearchPeople` — resolve account executive and deal team identities
- `GetUserDetails` — pull profile data for team members
- `SearchM365(sources=["email"])` — find the RFP receipt thread or deal support request
- `SearchM365(sources=["files"])` — locate the RFP document in SharePoint or OneDrive
- `ReadFileContent` — read intake form or request details from SharePoint
- `GetDriveChildren` — check existing proposal tracker for duplicates

**Output:** Structured case record written to Excel proposal tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Proposal ID, Account Name, Opportunity ID, RFP Title, Due Date, Account Executive, Proposal Manager, Status, Created Date, Deal Size, Complexity Rating

**Guardrails:**
- Never create duplicate cases for the same opportunity and due date
- Validate that due date is in the future
- Confirm details with user before writing to tracker
- Flag if deal size exceeds threshold requiring executive sponsor involvement

---

#### 2. `sales-context-packet` — Gather Account, Product, and Content Context

**Framework Step:** SL-002

**Trigger phrases:** "build context packet for [account]", "assemble proposal context", "what do we have on [account]", "pull account history for proposal"

**Inputs:**
- Proposal case ID or account name
- Opportunity stage and product interest areas

**M365 tools:**
- `GetUserDetails` — account team profiles
- `GetManagerDetails` — reporting chain for escalation paths
- `SearchM365(sources=["files"])` — prior proposals, account plans, product collateral, pricing guides
- `SearchM365(sources=["connectors"], connector_ids=["dynamics-connector"])` — CRM opportunity data via Graph Connector
- `ReadFileContent` — SharePoint content library assets (approved case studies, product sheets, competitive positioning)
- `GetDriveChildren` — browse approved content library structure

**Output:** Word document containing:
- Account overview (industry, tier, relationship history)
- Opportunity summary (stage, size, timeline, competitive context)
- Product and solution fit analysis
- Relevant prior proposals and win/loss history
- Approved content assets identified for reuse
- Key contacts and decision makers

**Artifact:** Word (.docx) saved to SharePoint proposal workspace folder

**Guardrails:**
- Only surface content from the approved content library — never include draft or expired assets
- Cite source and last-updated date for every content asset referenced
- Flag if account data is older than 90 days or if CRM opportunity is in a stale stage
- Do not include internal competitive intelligence in customer-facing sections

---

#### 3. `sales-rfp-extraction` — Extract Customer Requirements

**Framework Step:** SL-003

**Trigger phrases:** "extract requirements from RFP", "parse customer questions", "what does the RFP ask for", "break down proposal requirements"

**Inputs:**
- RFP document (PDF or Word from SharePoint)
- Proposal case data

**M365 tools:**
- `ReadFileContent` — read the RFP document from SharePoint
- `SearchM365(sources=["files"])` — locate supplementary RFP attachments or amendments
- `GetDriveChildren` — check proposal folder for all RFP-related documents

**Output:** Structured requirements matrix as Adaptive Card (for quick review) plus Excel worksheet (for tracking)

**Logic:** Parse the RFP document to extract: numbered questions, mandatory requirements, technical specifications, compliance requirements, submission format requirements, evaluation criteria, deadlines and milestones, and terms and conditions references.

**Artifact:** Excel worksheet with columns: Requirement ID, Section, Requirement Text, Category (Technical/Commercial/Legal/Compliance), Priority (Must/Should/Nice), Response Status, Assigned To, Notes

**Guardrails:**
- Present extracted requirements for user review before writing to the tracker — requirement interpretation is judgment-dependent
- Flag ambiguous requirements that need clarification from the customer
- Never auto-classify a requirement as "not applicable" — all requirements must be human-reviewed
- Preserve original RFP language alongside any summarized version

---

#### 4. `sales-content-mapping` — Map Requirements to Approved Responses and Gaps

**Framework Step:** SL-004

**Trigger phrases:** "map requirements to content", "find approved answers for RFP", "identify response gaps", "content coverage check"

**Inputs:**
- Requirements matrix (Excel from `sales-rfp-extraction`)
- Approved content library location in SharePoint

**M365 tools:**
- `ReadFileContent` — requirements matrix and approved content assets
- `SearchM365(sources=["files"])` — search approved content library for matching responses
- `GetDriveChildren` — browse content library by category (technical, legal, compliance, case studies)

**Output:** Coverage analysis as Adaptive Card plus updated Excel requirements matrix

**Logic:** For each requirement, search the approved content library for relevant responses. Categorize each as: Covered (approved response exists), Partial (related content exists but needs adaptation), Gap (no approved content — requires new drafting), or Stale (content exists but is outdated or under review).

**Artifact:** Updated Excel requirements matrix with additional columns: Coverage Status, Matched Content Asset, Content Last Updated, Gap Notes

**Guardrails:**
- Only match against content explicitly tagged as "approved" in the content library
- Flag any matched content older than 6 months for freshness review
- Never mark a pricing or legal requirement as "covered" — these always route to specialist review
- Present gap analysis for review before updating the tracker
- Clearly distinguish between "no content found" and "content exists but does not fully address the requirement"

---

#### 5. `sales-proposal-draft` — Draft Proposal Response

**Framework Step:** SL-005

**Trigger phrases:** "draft the proposal", "build proposal deck for [account]", "assemble response document", "create RFP response"

**Inputs:**
- Requirements matrix with coverage mapping
- Context packet (Word document)
- Proposal template from SharePoint
- Approved content assets

**M365 tools:**
- `ReadFileContent` — all input artifacts from SharePoint
- `SearchM365(sources=["files"])` — proposal templates, approved content blocks, prior winning proposals
- PowerPoint generation skill (`pptx`) — for proposal presentation decks
- Word generation skill (`docx`) — for narrative RFP response documents

**Output options:**
- **PowerPoint deck** — Executive proposal presentation with account context, solution overview, differentiators, pricing summary placeholder, and implementation approach
- **Word document** — Detailed RFP response document with section-by-section answers mapped to requirements
- **Adaptive Card** — Draft status summary showing sections completed, sections needing input, and sections flagged for specialist review

**Guardrails:**
- Always create as draft — never finalize or distribute without explicit user confirmation
- Every claim must trace to an approved content asset — no fabricated product capabilities or unsupported commitments
- Pricing sections must contain placeholder markers ("PRICING — REQUIRES DEAL DESK APPROVAL") — never populate pricing from historical data
- Legal and contractual language sections must contain placeholder markers ("LEGAL — REQUIRES LEGAL REVIEW") — never draft legal terms
- Include source citations for every content block used
- Match proposal template formatting standards from the approved template library

---

#### 6. `sales-approval-routing` — Route Pricing, Legal, and Product Approvals

**Framework Step:** SL-006

**Trigger phrases:** "route proposal for approval", "send for deal desk review", "request pricing approval", "submit for legal review"

**Inputs:**
- Draft proposal package
- Proposal case data (deal size, product mix, special terms)
- Approval matrix from SharePoint

**M365 tools:**
- `ReadFileContent` — approval matrix and routing rules from SharePoint
- `SearchPeople` — resolve approver identities by role
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `PostMessage` — Teams notification to approvers with review context
- `CreateDraftMessage` — Outlook email with proposal package for formal approval requests
- `CreateEvent` — calendar holds for approval deadlines

**Output:** Routed review tasks:
- Teams messages to each required approver with deal context and review scope
- Outlook draft emails with proposal attachments for formal approval chain
- Calendar deadline holds for approval due dates
- Updated Excel tracker with approval status per reviewer

**Guardrails:**
- Present routing recommendations for proposal manager review before sending any messages
- Route based strictly on the approval matrix — deal size thresholds, product categories, and special terms determine which approvers are required
- Never skip a required approver even if the deal appears straightforward
- Escalate to sales leadership if approval matrix produces no clear routing for a deal characteristic
- Include deal size and risk flags in every approval request message

---

#### Step 7: Confirm Submission Readiness (Human Only)

**Framework Step:** SL-007

This is not a Cowork skill. The framework correctly identifies final submission readiness as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the proposal review meeting with the deal team
- The proposal draft and approval artifacts — provide the evidence package for the reviewer
- The `sales-approval-routing` skill outputs — show approval status across all required reviewers
- The `sales-proposal-draft` artifacts — the final proposal package for submission

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in Sales is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle the entire proposal process. The framework's rule — "keep breaking down until each step has one dominant goal" — separates requirement extraction from content mapping from drafting, producing well-scoped skills that score high on the quality rubric's Scope Boundaries dimension.

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "pull account data," the framework requires naming every input source. This translates to specific MCP tool calls — `SearchM365(sources=["connectors"])` for CRM data, `ReadFileContent` for content library assets, `GetDriveChildren` for folder-based asset discovery.

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

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (pending approvals, deal stage, prior decisions) must be externalized to M365 artifacts — specifically the Excel proposal tracker and the SharePoint proposal workspace folder structure.

### 3.3 M365 Artifacts as First-Class Process State

Each M365 artifact type serves a specific role in the Sales proposal process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (proposal tracker, requirements matrix, coverage analysis, approval status) | The proposal tracker workbook IS the process state — skills read current status, write updates, track requirement coverage and approval progress |
| **PowerPoint** | Primary deliverable artifact (proposal presentation decks, executive summaries) | Skills generate proposal decks using approved templates; the deck IS the customer-facing deliverable |
| **Word** | Evidence and narrative artifacts (context packets, detailed RFP responses, prior proposal references) | Skills generate response documents that become the detailed proposal package |
| **Outlook** | Communication channel, signal source, and approval routing | Skills read RFP receipt emails for context; draft approval request emails; create formal submission communications |
| **SharePoint** | Source of truth (approved content library, pricing policies, proposal templates, CRM-synced data) | Skills read approved content, templates, and policies from SharePoint; proposal workspaces organize per-deal artifacts |
| **Teams** | Coordination channel and real-time routing | Skills post approval requests, deal team updates, and deadline reminders to deal channels or chats |
| **Graph API** | People and org data (account team, approvers, deal desk contacts) | Skills resolve approver identities, org structure for escalation, and deal team membership |
| **Calendar** | Time-bound process events (RFP due dates, approval deadlines, review meetings) | Skills create deadline holds and review meeting invitations |

**The design pattern:** The Cowork Sales plugin uses a SharePoint-hosted Excel workbook as the canonical proposal tracker, with a per-proposal folder in SharePoint as the workspace. PowerPoint and Word artifacts are the primary deliverables. Outlook and Teams handle communication and approval routing. This gives sales teams artifacts they already produce manually — the automation accelerates assembly while preserving the artifact trail that deal review requires.

### 3.4 Federated Connectors for Third-Party Systems

The framework references "CRM", "pricing system", "content library", and "approval workflow" — these often live outside M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For CRM platforms (Dynamics 365, Salesforce), Graph Connectors index opportunity records, account data, and contact information into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["dynamics-connector"])` or `SearchM365(sources=["connectors"], connector_ids=["salesforce-connector"])`. This provides read access to deal data without custom integration code.

For proposal content management platforms (Seismic, Highspot), Graph Connectors can index approved content assets. Skills search for relevant content via `SearchM365(sources=["connectors"], connector_ids=["seismic-connector"])`.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- CRM opportunity snapshots synced to a SharePoint list via Power Automate
- Approved content library mirrored from the content management platform to a SharePoint document library
- Pricing policy documents maintained in SharePoint by deal desk operations
- Approval matrices maintained as SharePoint lists updated by sales operations

Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For pricing approvals and special terms that require real-time system access, skills provide structured intake via Adaptive Card prompts that capture data from manual lookups, writing it into the shared Excel tracker. The framework's Signal Inventory phase identifies exactly which data points are needed.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) for CRM data and content library access. Use Tier 3 (manual input) for pricing data. Introduce Tier 1 Graph Connectors in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

The framework's governance model maps to Cowork for Sales with specific attention to deal data confidentiality and pricing controls:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document the proposal operations team structure and escalation paths |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC; deal-specific SharePoint permissions control proposal workspace access |
| **Data classification** | Skill guardrails enforce pricing confidentiality ("never include internal pricing models in customer-facing documents"), competitive intelligence handling ("do not surface internal win/loss analysis in proposals"), and deal data sensitivity |
| **Audit** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail; proposal tracker captures who changed what and when |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric (0–100 scoring) provides a pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions ("never auto-populate pricing", "always draft before send", "route deals above threshold to executive sponsor") |

**Sales-specific governance concerns:**
- **Pricing leakage** — Skills must never populate pricing from historical proposals or internal pricing models. All pricing sections require deal desk involvement.
- **Unapproved claims** — Skills must only surface content tagged as approved in the content library. Expired, draft, or under-review content must be excluded.
- **Competitive intelligence** — Internal competitive analysis must never appear in customer-facing artifacts.
- **Deal confidentiality** — Proposal workspace permissions must be scoped to the deal team. Skills must not cross-reference data from other active deals.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Sales Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via 8–10 should-trigger and 8–10 should-not-trigger phrases per skill
- Output quality — do generated proposals contain accurate, cited information from approved sources? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Time to first draft — hours from intake to a review-ready proposal package
- Requirement extraction accuracy — percentage of RFP requirements correctly identified and categorized
- Response draft acceptance rate — percentage of `sales-proposal-draft` outputs sent for review without major rewrites
- Content coverage rate — percentage of requirements matched to approved content by `sales-content-mapping`
- Approval routing accuracy — percentage of deals routed to the correct approvers by `sales-approval-routing`
- Unapproved claim rate — incidents of unapproved content appearing in proposal drafts
- Pricing placeholder compliance — percentage of proposals where pricing sections correctly contain placeholders rather than auto-populated values

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Sales Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel proposal tracker workbook in SharePoint with standard columns (Proposal ID, Account Name, Opportunity ID, RFP Title, Due Date, Account Executive, Proposal Manager, Status, Created Date, Deal Size, Complexity Rating)
- Set up the approved content library in SharePoint with category folders (Technical, Legal, Compliance, Case Studies, Product Sheets, Competitive Positioning) and an "Approved" metadata tag
- Create a proposal workspace folder template in SharePoint for per-deal artifact storage
- Sync CRM opportunity data to a SharePoint list via Power Automate (Tier 2 bridge)

**Skills to build:**
- `sales-proposal-intake`
- `sales-context-packet`
- `sales-rfp-extraction`
- `sales-proposal-draft`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5–10 real RFP responses.

### Wave 2 — Coverage and Routing

**Skills to build:**
- `sales-content-mapping`
- `sales-approval-routing`

**Promotions:**
- Promote `sales-proposal-intake` to write mode (creates case records after confirmation)
- Promote `sales-rfp-extraction` to write-back mode (updates Excel requirements matrix after user confirmation)
- Promote `sales-context-packet` to full document generation (writes Word context packet to SharePoint)

**Automation:**
- Set up a daily scheduled prompt that checks for proposals with approaching due dates and surfaces any with incomplete requirements coverage or missing approvals

**Operating posture:** AI draft plus approve for proposal drafting and communications. AI act within policy for approval routing (bound to the approval matrix).

### Wave 3 — Optimization and Proactive Intelligence

**Enhancements:**
- Introduce Graph Connectors for CRM data if available at the tenant level
- Add bounded multi-document RFP assembly for complex proposals with multiple response volumes
- Add proactive identification of likely approval blockers (deals with special terms, non-standard pricing, new product combinations)
- Refine all skills based on override patterns and reviewer feedback from Waves 1–2
- Add content library freshness monitoring — flag approved content approaching its review date

**Measurement:**
- Time to first draft reduction vs. pre-pilot baseline (target: 40% reduction)
- Draft acceptance rate target: above 65%
- Requirement extraction accuracy target: above 90%
- Approval routing accuracy target: above 95%
- Unapproved claim rate target: zero

---

## Part 5: Generalizing the Approach — Sales Artifact Pattern

The Sales LOB's primary artifact pattern is **PowerPoint + Outlook + Excel**: proposal decks are the customer-facing deliverable, Outlook handles formal communications and approval routing, and Excel tracks process state across the proposal lifecycle.

This pattern fits the cross-LOB method because:
1. **Decomposition** separated what appeared to be a single task (proposal assembly) into seven distinct steps with different owners and risk profiles
2. **Automation boundaries** naturally clustered around the critical control points — pricing and legal content remained human-gated while context assembly and requirement extraction became AI-assisted
3. **M365 artifacts** provided the natural state store — proposal teams already work in SharePoint, PowerPoint, and Outlook; the Cowork plugin accelerates that existing workflow rather than replacing it
4. **Federated connectors** addressed the CRM dependency through a pragmatic tiered approach rather than requiring full integration before the pilot could start

The method is repeatable: any LOB with document-heavy, approval-gated workflows can follow the same decompose → boundary → map → build → evaluate cycle.

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
