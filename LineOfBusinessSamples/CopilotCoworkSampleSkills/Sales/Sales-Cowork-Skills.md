# Sales Proposal and RFP Response Assembly — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Proposal and RFP Response Assembly** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert incoming RFPs and proposal requests into a governed, evidence-backed proposal response workflow within Microsoft 365.

The workflow supports intake signals from RFP receipt emails, deal support requests, CRM opportunity triggers, and direct proposal requests, guiding each through structured normalization, context assembly, requirement extraction, content coverage mapping, proposal drafting, and approval routing with strict controls for pricing confidentiality, approved-content-only usage, legal language protection, source traceability, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **sales-proposal-intake** | Normalizes RFP receipts and proposal requests into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **sales-context-packet** | Assembles account history, product context, competitive intelligence, and approved content assets | AI act within policy | analysis | SearchSparkle |
| 3 | **sales-rfp-extraction** | Extracts structured requirements, questions, deadlines, and constraints from RFP documents | AI assist | analysis | Tag |
| 4 | **sales-content-mapping** | Maps requirements to approved content assets and identifies coverage gaps | AI assist | analysis | Tag |
| 5 | **sales-proposal-draft** | Drafts proposal response documents and presentation decks using approved content | AI draft + approve | analysis | Flag |
| 6 | **sales-approval-routing** | Routes proposal packages to required pricing, legal, product, and executive approvers | AI act within policy | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Proposal         │  Normalize request → structured case record
│     Intake           │  Validates fields, checks duplicates,
│     (sales-proposal- │  generates Proposal ID, calculates SLA
│      intake)         │  SLA: 3 business days to first draft
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble account history, product
│     (sales-context-  │  context, competitive intel, prior
│      packet)         │  proposals, approved content inventory
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. RFP Extraction   │  Parse RFP document, extract requirements
│     (sales-rfp-      │  by category, flag ambiguities, capture
│      extraction)     │  deadlines and evaluation criteria
│                      │  Output: Requirements matrix + Adaptive Card
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Content Mapping  │  Map requirements to approved content,
│     (sales-content-  │  identify gaps, flag stale content,
│      mapping)        │  route specialist requirements
│                      │  Output: Coverage analysis + gap report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Proposal Draft   │  Draft proposal response using approved
│     (sales-proposal- │  content, templates, and context packet
│      draft)          │  with pricing and legal placeholders
│                      │  Output: PowerPoint deck and/or Word document
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Approval         │  Route to deal desk, legal, product,
│     Routing          │  executive, and finance approvers per
│     (sales-approval- │  the approval matrix
│      routing)        │  Output: Teams + Outlook + Calendar
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm          │  Confirm package is approved, revised,
│     Submission       │  or escalated. Human-only step —
│     Readiness        │  final customer-facing submission
│     (not automated)  │  remains human-owned.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Proposal Intake | Structured field extraction, validation, duplicate checking, SLA calculation — no AI judgment on deal strategy or pricing |
| **AI act within policy** | Context Packet, Approval Routing | Retrieves approved context from defined data sources or routes within defined approval matrices without exercising judgment on deal strategy |
| **AI assist** | RFP Extraction, Content Mapping | Surfaces extraction results and coverage recommendations with confidence levels; proposal manager reviews before tracker updates |
| **AI draft + approve** | Proposal Draft | AI produces proposal artifacts using approved content; proposal manager reviews and confirms before any distribution |
| **Human only** | Confirm Submission Readiness | Final customer-facing submission, pricing finalization, and contractual commitments are always human-owned |

## Governance Controls

### Pricing Confidentiality

- Skills never auto-populate pricing from historical proposals or internal pricing models
- All pricing sections in proposal drafts contain "PRICING — REQUIRES DEAL DESK APPROVAL" placeholders
- Internal pricing models, discount structures, and margin data never appear in any proposal artifact
- Deal size range is used for routing and complexity assessment — exact pricing is deal desk territory

### Approved Content Only

- Proposal drafts only use content explicitly tagged as "approved" in the content library
- Draft, expired, or under-review content is never presented as available response material without clear status flags
- Every content block in a proposal draft traces to a specific approved content asset with citation
- Content older than 6 months is flagged for freshness review before use

### Legal Language Protection

- All legal and contractual sections in proposal drafts contain "LEGAL — REQUIRES LEGAL REVIEW" placeholders
- Skills never draft legal terms, indemnification language, or liability provisions
- Non-standard terms always trigger legal review in the approval routing — no exceptions
- Legal requirements are flagged for specialist review rather than interpreted by the skill

### Source Traceability

- Every claim in a proposal draft cites the approved content asset it was sourced from
- New content drafted to fill gaps is clearly marked "NEW CONTENT — REQUIRES REVIEW"
- Requirements preserve original RFP language alongside any summarized version
- The context packet cites every data source consulted and its retrieval timestamp

### Audit Trail

- Every proposal records the Proposal ID, Account Name, Opportunity ID, intake source, and creation timestamp
- Every requirement extraction records the RFP source, requirement count, and confirming user
- Every content mapping records the coverage status, matched assets, and gap analysis
- Every proposal draft records the content sources used, placeholders applied, and reviewing user
- Every approval routing records the approver, approval type, deadline, status, and routing rationale
- The Excel proposal tracker serves as the Cowork-accessible audit record

### Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Customer financial data (contract value, deal size) | Deal size range for routing only — never exact figures in proposals |
| Internal pricing models and discount structures | Never in any proposal artifact — deal desk controls all pricing |
| Competitive intelligence (battle cards, win/loss analysis) | Internal context packet only — never in customer-facing proposals |
| Internal deal strategy and negotiation context | Never in customer-facing materials |
| Account data older than 90 days | Flagged as stale; never presented as current without verification |

## Proposal Complexity Ratings

| Rating | Criteria |
|--------|---------|
| **High** | Deal above executive threshold, multi-product, custom terms, competitive displacement, or 50+ requirements |
| **Medium** | Standard deal size, single product area, some custom requirements, or 20–50 requirements |
| **Low** | Standard deal, standard product, template-friendly requirements, or fewer than 20 requirements |

## Requirement Categories

| Category | What Is Extracted |
|----------|------------------|
| **Technical** | Product capabilities, integration needs, architecture constraints, performance specifications |
| **Commercial** | Pricing format, payment terms, licensing model, SLA expectations |
| **Legal** | Contract terms, liability, indemnification, IP ownership, data handling |
| **Compliance** | Regulatory certifications, audit requirements, data residency, security standards |
| **Submission format** | Page limits, font requirements, section ordering, delivery method |
| **Evaluation criteria** | Scoring methodology, weighted categories, pass/fail criteria |

## Content Coverage Statuses

| Status | Criteria |
|--------|---------|
| **Covered** | Approved content directly addresses the requirement; content is current |
| **Partial** | Related content exists but does not fully address the requirement |
| **Gap** | No approved content found — new drafting required |
| **Stale** | Content exists but is older than 6 months or under review |

## Approval Types

| Approval Type | Trigger | Approver Role |
|--------------|---------|---------------|
| **Deal desk / pricing** | All proposals with pricing components | Deal desk manager |
| **Legal** | Non-standard terms, custom contractual language | Legal counsel |
| **Product / technical** | Custom implementation, capability commitments | Product manager or solutions architect |
| **Executive sponsor** | Deals above threshold, strategic accounts | VP Sales or regional leader |
| **Finance** | Non-standard payment terms, multi-year commitments | Finance approver |
| **Security / compliance** | Certification commitments, data residency | Security officer or compliance lead |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find proposal tracker, RFP documents, content library, templates, approval matrix, context documents |
| SearchM365 (connectors) | Context Packet, Content Mapping — pull CRM opportunity data, content management assets via Graph Connectors |
| SearchM365 (email) | Proposal Intake, Context Packet — find RFP receipt threads, deal correspondence |
| SearchM365 (teams) | Proposal Intake, Context Packet — find deal team discussions |
| ReadFileContent | All skills — read tracker, RFP documents, content assets, templates, policies |
| GetDriveChildren | Context Packet, RFP Extraction, Content Mapping — browse content library and proposal workspace folders |
| SearchPeople / GetUserDetails | Proposal Intake, Approval Routing — resolve deal team members, approver identities |
| GetManagerDetails / GetDirectReportsDetails | Approval Routing — resolve escalation paths for unavailable approvers |
| ListCalendarView | Approval Routing — check approver availability within approval window |
| PostMessage | Approval Routing — Teams notifications to approvers with deal context |
| CreateDraftMessage | Approval Routing — formal Outlook approval request emails |
| CreateEvent | Approval Routing — calendar deadline holds for approval milestones |
| render_ui (Adaptive Card) | Proposal Intake, RFP Extraction, Content Mapping, Proposal Draft, Approval Routing — decision surfaces and status summaries |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Proposal case record | Excel tracker row | Proposal Intake |
| Context packet | Word document | Context Packet |
| Requirements matrix | Excel worksheet + Adaptive Card | RFP Extraction |
| Coverage analysis | Adaptive Card + Excel update | Content Mapping |
| Proposal presentation | PowerPoint deck | Proposal Draft |
| Proposal response document | Word document | Proposal Draft |
| Approval request notifications | Teams messages + Outlook drafts | Approval Routing |
| Approval deadline holds | Calendar events | Approval Routing |

## Federated Data Access

Sales proposal workflows depend on CRM platforms, content management systems, pricing systems, and approval workflows. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | CRM platforms (Dynamics 365, Salesforce) for opportunity data, account context, and contacts; content management platforms (Seismic, Highspot) for approved content assets |
| **Tier 2** | SharePoint Bridge | CRM opportunity snapshots synced via Power Automate; approved content library mirrored from content management platform; pricing policies and approval matrices maintained in SharePoint |
| **Tier 3** | Manual Input | Pricing data from deal desk, special terms from negotiations, customer clarifications captured via structured intake |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge for CRM data and content library) and Tier 3 (manual input for pricing data). Introduce Graph Connectors for CRM and content management in Wave 2 after skill workflows are proven.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── sales-proposal-intake/SKILL.md
├── sales-context-packet/SKILL.md
├── sales-rfp-extraction/SKILL.md
├── sales-content-mapping/SKILL.md
├── sales-proposal-draft/SKILL.md
└── sales-approval-routing/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Proposal tracker** — shared Excel workbook (Proposal ID, Account Name, Opportunity ID, RFP Title, Due Date, Account Executive, Proposal Manager, Status, Created Date, Deal Size, Complexity Rating, SLA Deadline, Approval Status)
- **Approved content library** — SharePoint document library with category folders (Technical, Legal, Compliance, Case Studies, Product Sheets, Competitive Positioning) and "Approved" metadata tag
- **Proposal workspace folder template** — per-deal SharePoint folder structure for RFP documents, context packets, drafts, and approval records
- **Proposal template** — standard proposal document and presentation templates
- **Approval matrix** — deal size thresholds, product category rules, special terms triggers, and required approver roles
- **Deal desk routing rules** — routing policies, escalation paths, and approval sequence requirements
- **Pricing policy** — pricing guidelines and discount authority thresholds (deal desk reference)
- **Product area taxonomy** — product hierarchy and solution definitions for content mapping

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Proposal Intake | "new proposal request for [account]", "RFP received for [account]", "set up proposal case", "log deal support request" |
| Context Packet | "build context packet for [account]", "assemble proposal context", "what do we have on [account]", "pull account history for proposal" |
| RFP Extraction | "extract requirements from RFP", "parse customer questions", "what does the RFP ask for", "break down proposal requirements" |
| Content Mapping | "map requirements to content", "find approved answers for RFP", "identify response gaps", "content coverage check" |
| Proposal Draft | "draft the proposal for [account]", "build proposal deck", "assemble response document", "create RFP response" |
| Approval Routing | "route proposal for approval", "send for deal desk review", "request pricing approval", "submit for legal review" |

## Implementation Roadmap

### Wave 1 — Foundation

- Create the shared Excel proposal tracker in SharePoint with standard columns
- Set up the approved content library with category folders and approval metadata
- Create proposal workspace folder template for per-deal artifact storage
- Sync CRM opportunity data to SharePoint list via Power Automate
- Build skills: `sales-proposal-intake`, `sales-context-packet`, `sales-rfp-extraction`, `sales-proposal-draft`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 5–10 real RFP responses across different deal sizes and product areas

### Wave 2 — Coverage and Routing

- Build skills: `sales-content-mapping`, `sales-approval-routing`
- Promote intake to write mode (creates case records after confirmation)
- Promote extraction to write-back mode (updates requirements matrix after confirmation)
- Promote context packet to full document generation (writes Word document to SharePoint)
- Set up daily scheduled prompt: check for proposals with approaching due dates and flag incomplete coverage or missing approvals
- Introduce Graph Connectors for CRM and content management platforms
- Measurement targets: above 90% requirement extraction accuracy, above 95% approval routing accuracy

### Wave 3 — Optimization and Proactive Intelligence

- Add bounded multi-document RFP assembly for complex proposals with multiple response volumes
- Add proactive identification of likely approval blockers (special terms, non-standard pricing, new product combinations)
- Add content library freshness monitoring — flag approved content approaching review date
- Measurement targets: 40% reduction in time to first draft, above 65% draft acceptance rate, zero unapproved claim incidents

## Implementation Notes

- **Submission readiness is intentionally not automated** — final customer-facing submission, pricing finalization, and contractual commitments carry sales accountability that cannot be delegated to AI
- **Pricing and legal language are never AI-generated** — these sections always contain explicit placeholders requiring specialist review
- **The approved content library is the single source of truth** for proposal response material — skills never surface draft, expired, or unapproved content as available responses
- **Every proposal claim must trace to an approved content asset** — fabricated product capabilities or unsupported commitments are a governance violation
- **The 3 business day SLA** drives urgency across all skills — every output surfaces SLA countdown and time remaining
- **Content coverage mapping before drafting** ensures the team knows where gaps exist before writing begins — this prevents last-minute discovery of missing content
- **Approval routing follows the matrix, not judgment** — deal characteristics determine which approvers are required; skills never skip or substitute approvers
- **Internal competitive intelligence never appears in customer-facing proposals** — differentiation language is permitted; competitive battle card content is not
