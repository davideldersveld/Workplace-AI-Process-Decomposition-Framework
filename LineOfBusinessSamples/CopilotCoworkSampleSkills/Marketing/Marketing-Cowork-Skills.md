# Marketing Campaign Request Intake and Brief Synthesis — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Campaign Request Intake and Brief Synthesis** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the campaign brief creation process — from request intake through review-ready baseline — into AI-assisted capabilities within Microsoft 365.

The workflow supports campaign request signals from email, Teams, intake forms, and direct submission, guiding each through structured normalization, brand and product context assembly, signal extraction, brief drafting, and review routing with strict brand compliance, approved claims controls, audience data privacy, scope visibility, review completeness enforcement, and audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **mktg-campaign-intake** | Normalizes inbound campaign requests into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **mktg-context-packet** | Assembles brand, product, audience, and campaign history context | AI act within policy | analysis | SearchSparkle |
| 3 | **mktg-signal-extraction** | Extracts goals, constraints, dependencies, and blockers from request artifacts | AI assist | analysis | Tag |
| 4 | **mktg-brief-draft** | Drafts a structured campaign brief from context and signals | AI draft + approve | analysis | Flag |
| 5 | **mktg-review-routing** | Routes briefs to brand, product, legal, and leadership reviewers | AI act within policy | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Campaign Intake  │  Normalize campaign request → structured case
│     (mktg-campaign-  │  Validates required fields, checks duplicates,
│      intake)         │  flags specialized review needs
│                      │  SLA: 3 business days for first review-ready brief
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble brand guidelines, product messaging,
│     (mktg-context-   │  audience profiles, prior campaign history,
│      packet)         │  channel recommendations
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Signal           │  Extract objectives, KPIs, audience, deliverables,
│     Extraction       │  timing, budget, channels, dependencies,
│     (mktg-signal-    │  blockers from request artifacts
│      extraction)     │  Output: Adaptive Card + Excel signal matrix
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Brief Draft      │  Synthesize context and signals into a structured
│     (mktg-brief-     │  campaign brief following approved templates
│      draft)          │  Output: Word brief + PowerPoint deck
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Review Routing   │  Route brief to brand, product marketing,
│     (mktg-review-    │  legal, communications, and leadership
│      routing)        │  reviewers per the routing matrix
│                      │  Output: Teams notifications + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Confirm Brief    │  Confirm brief is approved, revised, or
│     Baseline         │  escalated. Human-only step — final brief
│     (not automated)  │  approval carries marketing accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Campaign Intake | Structured field extraction, validation, duplicate checking — no AI judgment on campaign strategy or messaging |
| **AI act within policy** | Context Packet, Review Routing | Retrieves approved context from defined brand and messaging repositories, or executes routing within pre-approved review matrix rules; does not generate strategy or claims |
| **AI assist** | Signal Extraction | Surfaces extracted signals with source attribution and conflict flags; campaign manager reviews before recording |
| **AI draft + approve** | Brief Draft | AI synthesizes context and signals into a structured brief; campaign manager reviews and confirms before distribution |
| **Human only** | Confirm Baseline | Final brief approval, revision decisions, and launch authorization are always human-owned |

## Governance Controls

### Brand Compliance

- Skills only surface content from approved brand and messaging repositories — never draft, expired, under-review, or deprecated assets
- Every messaging element and claim in the brief traces to an approved source document with version reference
- Brand guidelines updates since the last campaign in a product area are flagged for the campaign manager
- Visual identity standards (logo usage, color palette, typography) are referenced from the brand book

### Approved Claims Controls

- No skill generates product claims or value propositions not backed by approved messaging documents
- This is especially critical for regulated industries (healthcare, financial services) where unapproved claims carry legal risk
- Sections where approved messaging does not fully cover the campaign's needs are flagged for product marketing input
- Competitive positioning is for internal use only — never included in external-facing sections

### Audience Data Privacy

- Skills never include personally identifiable audience data in brief documents — audience references use segment-level abstractions only
- Audience segment data older than one quarter is flagged for marketing operations review
- Customer data references in campaigns are flagged for privacy review routing

### Scope Visibility

- Scope creep indicators (new deliverables or audiences added after the original request) are flagged prominently
- Conflicting signals (aggressive timeline vs. extensive deliverables, narrow budget vs. broad scope) are surfaced for campaign manager decision
- Missing critical signals (no clear objective, no defined audience, no launch date) are flagged before brief drafting

### Review Completeness

- The review routing matrix defines minimum review requirements — no required reviewer is ever skipped
- Legal review is non-negotiable for campaigns with regulated claims, testimonials, sweepstakes, or comparative claims
- Review sequence dependencies are tracked (e.g., legal review after product marketing confirms claims)
- Timeline risks (review completion exceeding requested launch date) are flagged immediately

### Audit Trail

- Every campaign case records the requesting stakeholder, intake source, timestamp, and Campaign ID
- Every extracted signal records the source (email, Teams, document, verbal) and confirmation status
- Every brief draft records which approved messaging sources were used with section references
- Every routing decision records the assigned reviewers, deadlines, rationale, and confirming campaign manager
- The Excel campaign tracker serves as the Cowork-accessible audit record

## Campaign Types

| Type | Description | Typical Review Requirements |
|------|-------------|---------------------------|
| **Product Launch** | New product or feature introduction | Brand, product marketing, legal (if regulated), leadership |
| **Event** | Conference, webinar, trade show, field event | Brand, event marketing, legal (if sponsorship or sweepstakes) |
| **Demand Generation** | Lead generation, pipeline acceleration | Brand, product marketing |
| **Brand** | Brand awareness, thought leadership | Brand, communications |
| **Field Marketing** | Regional or field-specific campaigns | Brand, regional marketing |
| **Content** | Content marketing, editorial | Brand, product marketing |
| **Digital** | Digital advertising, social media, web | Brand, digital marketing, legal (if comparative claims) |
| **Other** | Non-standard campaigns | Brand (minimum), plus case-by-case |

## Signal Categories

| Category | What Gets Extracted | Why It Matters |
|----------|-------------------|---------------|
| **Objectives** | Campaign goals, business outcomes, KPIs | Defines what success looks like |
| **Target audience** | Who the campaign targets | Determines messaging, channels, and review requirements |
| **Deliverables** | Required assets and content types | Defines production scope and timeline |
| **Timing** | Dates, deadlines, milestones | Drives review urgency and SLA tracking |
| **Budget** | Budget range, spend constraints | Constrains channel and production options |
| **Channels** | Distribution and promotion channels | Determines asset specifications and review requirements |
| **Dependencies** | Requirements on other teams or campaigns | Identifies blockers and coordination needs |
| **Blockers** | Known obstacles or risks | Flags issues that could delay the campaign |
| **Stakeholder expectations** | Specific requests or constraints | Captures non-obvious requirements |

## Reviewer Categories

| Reviewer | Required When | Review Scope |
|----------|--------------|-------------|
| **Brand reviewer** | All campaigns | Brand voice, visual identity, messaging consistency |
| **Product marketing reviewer** | Campaigns with product claims | Claim accuracy, messaging alignment |
| **Legal reviewer** | Regulated claims, testimonials, sweepstakes, comparative claims | Legal compliance, regulatory requirements |
| **Communications reviewer** | External-facing, press-adjacent content | External messaging, PR alignment |
| **Marketing leadership** | Above budget threshold, executive-sponsored | Strategic alignment, resource allocation |
| **Regional marketing reviewer** | Cross-regional or international | Regional adaptation, local compliance |
| **Privacy reviewer** | Customer data, personalization, targeting | Data usage compliance, consent |

## Brief Sections

| Section | Content | Source |
|---------|---------|--------|
| Executive Summary | Campaign purpose and expected outcome | Signals (objectives), context (business context) |
| Campaign Objective | Primary objective and measurable KPIs | Signals (objectives, KPIs) |
| Target Audience | Audience definition with demographics and pain points | Context (audience profile), signals (audience) |
| Key Messages | Messaging theme, claims, proof points, CTA | Context (product messaging), approved claims — every claim cited |
| Channel Strategy | Channel mix with rationale and requirements | Context (channel recommendations), signals (channel requirements) |
| Deliverables and Timeline | Asset list, specifications, milestones | Signals (deliverables, timing) |
| Budget | Allocation by channel or activity | Signals (budget parameters) — ranges unless confirmed |
| Success Metrics | Measurement approach and benchmark targets | Signals (KPIs), context (prior performance) |
| Open Questions | Unresolved signals, conflicts, missing inputs | Signals (unconfirmed, conflicting) |
| Required Reviews | Brand, product, legal, leadership reviews | Tracker (specialized review flags) |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find campaign tracker, brand guidelines, messaging documents, templates, audience profiles, prior campaigns, signal matrix, review routing matrix |
| SearchM365 (connectors) | Context Packet — retrieve audience data and campaign history from marketing automation via Graph Connector |
| SearchM365 (email) | Campaign Intake, Signal Extraction — find request emails, stakeholder clarifications, scope changes |
| SearchM365 (teams) | Campaign Intake, Signal Extraction — find campaign coordination threads |
| ReadFileContent | All skills — read tracker, brand guidelines, messaging, templates, audience data, prior briefs, signal matrix, routing matrix |
| GetDriveChildren | Context Packet, Signal Extraction — browse brand asset library, campaign workspace, template repositories |
| SearchPeople / GetUserDetails | All skills — resolve stakeholders, campaign managers, reviewers |
| GetManagerDetails / GetDirectReportsDetails | Review Routing — org structure for escalation paths |
| PostMessage | Review Routing — Teams notifications to reviewers |
| CreateDraftMessage | Review Routing — formal review request emails |
| CreateEvent | Review Routing — review deadline calendar holds |
| render_ui (Adaptive Card) | Campaign Intake, Signal Extraction, Brief Draft, Review Routing — confirmations, signal summaries, draft status, routing recommendations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Campaign case record | Excel tracker row | Campaign Intake |
| Context packet | Word document | Context Packet |
| Signal matrix | Adaptive Card + Excel worksheet | Signal Extraction |
| Campaign brief (detailed) | Word document | Brief Draft |
| Campaign brief (presentation) | PowerPoint deck | Brief Draft |
| Draft status summary | Adaptive Card | Brief Draft |
| Routing recommendation | Adaptive Card | Review Routing |
| Reviewer notifications | Teams direct messages | Review Routing |
| Formal review requests | Outlook draft emails | Review Routing |
| Review deadline holds | Calendar events | Review Routing |

## Federated Data Access

Campaign brief synthesis depends on marketing automation platforms, DAM systems, CRM data, and work management tools. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Marketing automation (Marketo, HubSpot) for audience segments and campaign performance; DAM (Bynder, Brandfolder) for approved asset metadata; work management (Asana, Workfront) for request records |
| **Tier 2** | SharePoint Bridge | Brand guidelines, approved messaging, campaign templates, audience research, prior campaign summaries, and review routing matrix maintained in SharePoint via Power Automate sync |
| **Tier 3** | Manual Input | Real-time audience data, budget figures, and creative asset availability captured via structured intake prompts with "manual entry" source tagging |

**Recommended pilot approach:** Start with Tier 2 (SharePoint for brand guidelines, templates, messaging, and audience data) and Tier 3 for budget and timeline specifics. Introduce Graph Connectors for marketing automation in Wave 2 after skill workflows are proven.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── mktg-campaign-intake/SKILL.md
├── mktg-context-packet/SKILL.md
├── mktg-signal-extraction/SKILL.md
├── mktg-brief-draft/SKILL.md
└── mktg-review-routing/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Campaign tracker** — shared Excel workbook (Campaign ID, Campaign Name, Requester, Department, Campaign Type, Requested Launch Date, Budget Range, Status, Created Date, Assigned Campaign Manager, Priority, Review Status)
- **Brand guidelines** — master brand book, visual identity standards, campaign-type-specific guidelines
- **Messaging framework** — corporate messaging architecture, product messaging pillars
- **Approved claims** — product claims, value propositions, and proof points with version tracking
- **Campaign brief templates** — Word and PowerPoint templates per campaign type
- **Audience profiles** — segment definitions, demographics, firmographics, channel preferences
- **Prior campaign history** — briefs, performance summaries, and lessons learned
- **Review routing matrix** — reviewer assignment rules by campaign type, audience, claims, channels, and budget
- **Campaign workspace folder template** — per-campaign SharePoint folder for artifact storage

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Campaign Intake | "new campaign request", "log campaign intake for [name]", "set up campaign case", "campaign request from [stakeholder]" |
| Context Packet | "build campaign context for [campaign]", "assemble brief context", "pull product messaging for [product]", "what brand assets apply" |
| Signal Extraction | "extract campaign requirements", "what are the goals for [campaign]", "identify campaign dependencies", "parse request signals" |
| Brief Draft | "draft the campaign brief", "create brief for [campaign]", "build the brief document", "prepare the brief deck" |
| Review Routing | "route brief for review", "send for brand review", "request legal review of campaign", "submit brief to reviewers" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel campaign tracker in SharePoint with standard columns
- Organize the brand guidelines library in SharePoint with versioned brand books and messaging frameworks
- Set up campaign workspace folder template in SharePoint
- Upload campaign brief templates (Word and PowerPoint formats)
- Build skills: `mktg-campaign-intake`, `mktg-context-packet`, `mktg-signal-extraction`, `mktg-brief-draft`
- Operate in AI assist mode — all outputs presented for campaign manager review
- Test with 5–10 real campaign requests over 2 weeks

### Wave 2 — Review Routing and Policy Checks

- Build skill: `mktg-review-routing`
- Promote intake to write mode (creates case records after confirmation)
- Promote signal extraction to write-back mode (updates signal matrix after confirmation)
- Add review summary generation for consolidating reviewer feedback
- Add policy-cited claim coverage checks (verify every claim traces to an approved source)
- Set up daily scheduled prompt: check for campaigns approaching launch dates with incomplete reviews
- Introduce Graph Connectors for marketing automation data if available

### Wave 3 — Optimization and Proactive Intelligence

- Add bounded multi-source campaign packet assembly for complex multi-channel campaigns
- Add proactive detection of likely review blockers (novel claims, new audiences, cross-regional scope)
- Add brand guideline freshness monitoring
- Add missing stakeholder detection
- Measurement targets: 40% time-to-first-brief reduction, above 70% draft acceptance rate, above 95% routing accuracy, 2 or fewer revision rounds average, zero unapproved-claim incidents

## Implementation Notes

- **Brief baseline approval is intentionally not automated** — final brief approval carries marketing accountability and budget commitment that cannot be delegated to AI
- **No skill generates unapproved claims** — this is a permanent architectural constraint; every product claim, value proposition, and proof point must trace to an approved source document
- **The work management platform is the source of truth** — the Excel campaign tracker is a Cowork-accessible working copy; state drift is mitigated by Power Automate sync
- **Brand compliance is the strictest guardrail** — only approved, current brand assets are surfaced; expired, draft, or deprecated messaging is excluded from all outputs
- **Signal extraction preserves original stakeholder language** — summaries are interpretations; the original wording is the evidence
- **Conflicting signals are surfaced, never auto-resolved** — the campaign manager resolves conflicts; the skill provides visibility
- **The 3 business day SLA** drives urgency for first review-ready brief — every skill surfaces SLA status
- **Audience data privacy is enforced** — no personally identifiable data in brief documents; segment-level abstractions only
- **Internal competitive positioning never appears in external-facing content** — competitive insights are for internal planning only
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., extracting signals from a new stakeholder email, updating context after a brand guideline change, or re-drafting after reviewer feedback)
