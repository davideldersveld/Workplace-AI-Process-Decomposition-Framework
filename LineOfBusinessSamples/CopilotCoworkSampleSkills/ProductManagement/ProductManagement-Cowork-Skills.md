# Product Management Feature Request Intake and Opportunity Framing — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Feature Request Intake and Opportunity Framing** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the feature request intake process — from inbound demand signal through review-ready opportunity brief — into AI-assisted capabilities within Microsoft 365.

The workflow supports feature request signals from email, Teams, customer calls, intake forms, and direct submission, guiding each through structured normalization, context assembly, duplicate detection and demand clustering, opportunity brief drafting, review packet preparation, and cross-functional reviewer routing with strict controls for preventing premature commitment, ensuring evidence traceability, protecting roadmap confidentiality, and maintaining product manager ownership of framing and prioritization decisions at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **pm-request-intake** | Normalizes inbound feature requests into structured opportunity cases | Deterministic automation | analysis | TaskListLtr |
| 2 | **pm-context-packet** | Assembles customer, product, telemetry, and backlog context | AI act within policy | analysis | SearchSparkle |
| 3 | **pm-demand-cluster** | Detects duplicates and clusters related demand into themes | AI assist | analysis | Tag |
| 4 | **pm-opportunity-brief** | Drafts evidence-backed opportunity statement with open questions | AI draft + approve | analysis | Flag |
| 5 | **pm-review-packet** | Packages brief and evidence into stakeholder review deck | AI draft + approve | analysis | Flag |
| 6 | **pm-reviewer-router** | Routes review packet to product, design, engineering, and GTM reviewers | AI act within policy | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Request Intake   │  Normalize feature request → structured case
│     (pm-request-     │  Validates fields, checks duplicates,
│      intake)         │  generates Opportunity ID
│                      │  SLA: 3 business days to review-ready brief
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble customer profile, product area
│     (pm-context-     │  context, backlog history, telemetry,
│      packet)         │  prior decisions, correspondence
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Demand Cluster   │  Compare against tracker and backlog;
│     (pm-demand-      │  identify duplicates, near-duplicates,
│      cluster)        │  and thematic clusters; calculate
│                      │  demand volume
│                      │  Output: Adaptive Card + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Opportunity      │  Draft problem statement, business
│     Brief            │  impact, evidence summary, open
│     (pm-opportunity- │  questions, and traceability
│      brief)          │  Output: Word opportunity brief
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Review Packet    │  Package brief, context, evidence into
│     (pm-review-      │  stakeholder review deck and optional
│      packet)         │  Word evidence summary
│                      │  Output: PowerPoint deck + Word summary
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Reviewer Router  │  Route to product, design, engineering,
│     (pm-reviewer-    │  and GTM reviewers per reviewer matrix;
│      router)         │  send notifications and schedule review
│                      │  Output: Teams messages + calendar event
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm          │  Confirm opportunity is accepted for
│     Disposition      │  prioritization, deferred, or declined.
│     (not automated)  │  Human-only step — prioritization
│                      │  readiness and roadmap commitment
│                      │  must remain human-owned.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Request Intake | Structured field extraction, validation, duplicate flagging — no AI judgment on opportunity merit or priority |
| **AI act within policy** | Context Packet, Reviewer Router | Retrieves approved context from defined data sources or routes within defined reviewer matrix rules without exercising judgment on opportunity merit |
| **AI assist** | Demand Cluster | Surfaces duplicate flags, demand clusters, and theme recommendations with evidence; PM reviews before tracker updates |
| **AI draft + approve** | Opportunity Brief, Review Packet | AI drafts synthesis artifacts (opportunity briefs, review decks); PM reviews and approves before distribution |
| **Human only** | Confirm Disposition | Final prioritization readiness, roadmap commitment, and disposition decisions are always human-owned |

## Governance Controls

### Commitment Prevention

- Every generated opportunity brief includes a prominent "DRAFT — NOT COMMITTED" label in the document header
- No skill uses commitment language — "will be built", "is planned", or "is committed" are never generated
- Skills use evaluation language: "is being evaluated", "is under consideration", "has been identified as an opportunity"
- Disposition decisions (accept, defer, decline) are always human-owned — no skill advances an opportunity to committed status
- Review packets are labeled as drafts and require explicit PM confirmation before distribution

### Evidence Traceability

- Every claim in an opportunity brief must trace to a source in the context packet or demand cluster output
- The context packet cites source documents and retrieval timestamps for every data point
- The review packet includes a traceability section linking every evidence claim to its source
- Demand clustering shows the basis for every match (keywords, product area, customer overlap)
- Evidence gaps are explicitly surfaced rather than hidden — open questions are a required section in every brief

### Roadmap Confidentiality

- Review packets are distributed only to approved reviewers per the reviewer matrix
- Sensitive opportunities (competitive intelligence, unreleased strategy) are flagged for restricted distribution
- Skills never post roadmap details to general Teams channels
- Reviewer routing verifies that all proposed reviewers have appropriate access before distribution

### Duplicate Detection

- Every intake checks the opportunity tracker and backlog for existing entries with similar product area and keywords
- Duplicates are flagged but never auto-merged or auto-closed — the PM decides disposition
- Resubmissions of previously declined or deferred requests are prominently flagged with the prior decision rationale
- Original request text is always preserved even when linked to a cluster

### Audit Trail

- Every opportunity case records the requester, source channel, creation timestamp, and Opportunity ID
- Every demand cluster finding records the match basis, confidence level, and confirming PM
- Every brief and review packet records the draft date, template compliance, and approving PM
- Every routing decision records assigned reviewers, notification timestamps, and confirming PM
- The Excel opportunity tracker serves as the Cowork-accessible audit record

### Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Customer revenue figures (ARR, contract value) | Tier-level indicators only (enterprise, mid-market, SMB) — never exact figures |
| NDA-protected customer feedback | Flagged for restricted distribution to assigned PM only |
| Competitive intelligence | Internal reviewer use only — never in general channels |
| Delivery estimates and timelines | Never included in any generated artifact — briefs frame problems, not delivery plans |
| Customer quotes | Paraphrased as themes — never attributed to specific individuals without source verification |

## Reviewer Roles

| Reviewer | Required When | Review Scope |
|----------|--------------|-------------|
| **Product lead** | All opportunities | Strategic fit, roadmap alignment, priority assessment |
| **Design lead** | Opportunities with UX impact | User experience implications, design feasibility, research needs |
| **Engineering lead** | Opportunities with technical implications | Technical feasibility, architecture impact, effort estimation |
| **GTM lead** | Opportunities with market or customer-facing impact | Go-to-market implications, competitive positioning |
| **Support lead** | Opportunities driven by support volume | Support impact, customer pain severity, workaround availability |
| **Sales lead** | Opportunities driven by sales escalation or revenue impact | Revenue implications, customer retention, competitive context |
| **Group product manager** | High-impact or cross-product opportunities | Cross-product dependencies, portfolio alignment |

## Request Source Channels

| Source Channel | ID Prefix | Signal Type |
|---------------|-----------|-------------|
| **Sales escalation** | SALES | Customer demand with revenue context |
| **Support case** | SUPPORT | Customer pain with severity and volume data |
| **Customer advisory board** | CAB | Strategic customer input with relationship context |
| **Internal stakeholder** | INTERNAL | Cross-functional request with business justification |
| **Analytics signal** | ANALYTICS | Telemetry-driven insight with usage data |
| **Intake form** | INTAKE | Structured submission via standard intake process |

## Demand Cluster Match Types

| Match Type | Confidence | Description |
|-----------|------------|-------------|
| **Exact duplicate** | High | Same feature from the same customer through a different channel |
| **Cross-channel duplicate** | High | Same feature submitted via both email and Teams |
| **Resubmission** | High | Previously submitted and closed (deferred or declined) |
| **Near-duplicate** | Medium-High | Same feature with different framing from a different customer |
| **Related feature** | Medium | Different feature in the same area addressing a similar problem |
| **Adjacent demand** | Low-Medium | Feature in a related area with potential implementation dependencies |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find opportunity tracker, brief templates, strategy documents, product glossary, feature taxonomy, reviewer matrix, backlog snapshots, telemetry reports, customer feedback, exemplar briefs |
| SearchM365 (connectors) | Context Packet, Demand Cluster — retrieve backlog items via Jira connector, customer data via CRM connector |
| SearchM365 (email) | Request Intake, Context Packet — find request threads, customer correspondence |
| SearchM365 (teams) | Request Intake, Context Packet — find product channel discussions, sales escalations |
| ReadFileContent | All skills — read tracker, templates, strategy docs, context packets, backlog data |
| SearchPeople / GetUserDetails | Request Intake, Context Packet, Reviewer Router — resolve requesters, account owners, reviewers |
| PostMessage | Reviewer Router — Teams notifications to reviewers |
| CreateDraftMessage | Reviewer Router — formal review request emails as Outlook drafts |
| CreateEvent | Reviewer Router — review meeting scheduling |
| ListCalendarView | Reviewer Router — check reviewer availability |
| render_ui (Adaptive Card) | Request Intake, Demand Cluster, Opportunity Brief, Review Packet, Reviewer Router — decision surfaces and status summaries |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Opportunity case record | Excel tracker row | Request Intake |
| Context packet | Word document | Context Packet |
| Clustering report | Adaptive Card + Excel tracker update | Demand Cluster |
| Opportunity brief | Word document | Opportunity Brief |
| Review presentation | PowerPoint deck | Review Packet |
| Evidence summary | Word document (optional) | Review Packet |
| Reviewer notifications | Teams direct messages | Reviewer Router |
| Formal review requests | Outlook draft emails | Reviewer Router |
| Review meeting | Calendar event | Reviewer Router |

## Federated Data Access

Product Management depends on backlog systems, CRM, product analytics, and support case platforms. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Backlog systems (Jira, Azure DevOps, Productboard) for feature items and deduplication; CRM platforms (Salesforce, HubSpot, Dynamics 365) for customer account context; product analytics (Amplitude, Mixpanel, Pendo) for usage data |
| **Tier 2** | SharePoint Bridge | Customer account summaries synced via Power Automate; backlog snapshots exported to Excel; weekly telemetry summary reports; support case volume by product area |
| **Tier 3** | Manual Input | Customer context when CRM is unavailable; telemetry data not in SharePoint; verbal feedback and clarifications captured via structured intake prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge for customer context and backlog snapshots) and Tier 3 for analytics data. Prioritize the Jira/Azure DevOps Graph Connector in Wave 2 — backlog deduplication is one of the highest-value capabilities.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── pm-request-intake/SKILL.md
├── pm-context-packet/SKILL.md
├── pm-demand-cluster/SKILL.md
├── pm-opportunity-brief/SKILL.md
├── pm-review-packet/SKILL.md
└── pm-reviewer-router/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Opportunity tracker** — shared Excel workbook (Opportunity ID, Request Source, Requester, Customer/Account, Product Area, Summary, Status, Created Date, Assigned PM, Theme, Duplicate Flag, Evidence Links, Reviewer Assignments, Disposition)
- **Opportunity brief template** — standard template with required sections (problem statement, business impact, evidence summary, open questions, traceability)
- **Review packet template** — standard review deck template for stakeholder presentations
- **Product glossary** — terminology and feature taxonomy for consistent categorization
- **Feature taxonomy** — standard theme categories and product area hierarchy
- **Reviewer matrix** — which roles review which product areas and opportunity types
- **Product strategy principles** — strategic priorities for alignment assessment
- **Prioritization rubric** — criteria that reviewers use to evaluate opportunities
- **Exemplar briefs** — 3-5 previously approved opportunity briefs as reference documents
- **Review process guide** — review timelines, scheduling rules, and escalation paths

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Request Intake | "new feature request from [source]", "log opportunity for [feature]", "intake this request", "customer asked for [feature]" |
| Context Packet | "build context for this request", "gather product context for [opportunity]", "what do we know about this opportunity" |
| Demand Cluster | "check for duplicates", "cluster related requests", "is this a duplicate", "group similar feature requests" |
| Opportunity Brief | "draft opportunity brief", "write the opportunity statement", "frame this as an opportunity", "create the problem statement" |
| Review Packet | "prepare review packet", "build the review deck", "package this for review", "get this ready for the review meeting" |
| Reviewer Router | "send this for review", "route to reviewers", "distribute the review packet", "schedule the review" |

## Implementation Roadmap

### Wave 1 — Foundation and Intake

- Create the shared Excel opportunity tracker in SharePoint with standard columns
- Upload opportunity brief template, review packet template, product glossary, feature taxonomy, and reviewer matrix to SharePoint
- Upload 3-5 exemplar opportunity briefs as reference documents
- Create a SharePoint folder structure for per-opportunity evidence packages
- Set up the product intake Teams channel for request monitoring
- Build skills: `pm-request-intake`, `pm-context-packet`, `pm-demand-cluster`, `pm-opportunity-brief`
- Operate in AI assist/draft mode — all outputs presented for PM review
- Test with 15-20 real feature requests over 2 weeks

### Wave 2 — Review and Routing

- Build skills: `pm-review-packet`, `pm-reviewer-router`
- Promote intake to write mode (creates tracker records after confirmation)
- Promote demand cluster to write-back mode (updates tracker with duplicate flags and themes after PM confirmation)
- Set up daily scheduled prompt: check for new unprocessed requests in the intake channel and email alias
- Set up weekly scheduled prompt: flag opportunities in "Intake" status for more than 5 business days
- Introduce Graph Connectors for Jira/Azure DevOps (backlog deduplication) and CRM (customer context)
- Measurement targets: above 70% brief draft acceptance rate, above 80% duplicate detection accuracy

### Wave 3 — Optimization and Proactive Synthesis

- Introduce additional Graph Connectors for product analytics platforms
- Add proactive monitoring: identify stale opportunities, surface recurring themes, flag high-demand clusters crossing product areas
- Add cross-product duplicate detection for organizations with multiple product lines
- Add source-cited evidence summaries with automatic customer quote and telemetry extraction
- Measurement targets: under 3 business day time to first review-ready brief, above 90% reviewer routing accuracy, 100% evidence traceability rate, 30% disposition cycle time reduction

## Implementation Notes

- **Disposition is intentionally not automated** — final prioritization readiness and roadmap commitment carry product leadership accountability that cannot be delegated to AI
- **No skill generates commitment language** — every generated artifact uses evaluation language ("under consideration", "being evaluated") and includes "DRAFT — NOT COMMITTED" labels
- **The opportunity brief is the core synthesis artifact** — it transforms fragmented demand signals into a structured, evidence-backed problem framing that enables informed human decision-making
- **The backlog system is the source of truth** for committed items — the Excel opportunity tracker is a Cowork-accessible working copy for intake pipeline state; committed items live in Jira/Azure DevOps
- **Evidence quality depends on federated connector access** — CRM and backlog connectors significantly improve context quality; without them, skills rely on SharePoint bridge data and manual input
- **Reviewer matrix and templates live in SharePoint** and are read dynamically — product operations can update review processes, brief templates, and routing rules without modifying skills
- **Every brief preserves open questions and evidence gaps** — honest uncertainty is more valuable than false confidence in product opportunity framing
- **The 3 business day SLA** drives urgency for review-ready brief — every skill surfaces SLA status and remaining time
