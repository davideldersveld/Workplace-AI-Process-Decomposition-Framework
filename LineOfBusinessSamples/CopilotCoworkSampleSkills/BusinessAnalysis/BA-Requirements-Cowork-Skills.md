# Business Analysis Requirements Intake and Synthesis — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Business Analysis Requirements Intake and Synthesis** workflow as a set of seven Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert incoming business requests, stakeholder input, and discovery evidence into a governed, traceable requirements package within Microsoft 365.

The workflow supports intake signals from email requests, Teams conversations, stakeholder workshops, change requests, and manual submissions, guiding each through structured normalization, context assembly, signal extraction, theme synthesis, requirements drafting, traceability packaging, and review routing with strict controls for source attribution, conflict transparency, scope discipline, traceability coverage, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **ba-request-intake** | Normalizes business requests into structured analysis case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **ba-discovery-packet** | Assembles prior documents, stakeholder contacts, policy references, and related decisions | AI act within policy | analysis | SearchSparkle |
| 3 | **ba-signal-extraction** | Extracts candidate needs, constraints, assumptions, and conflicts from source documents | AI assist | analysis | DocumentText |
| 4 | **ba-theme-synthesis** | Clusters signals into themes, detects conflicts, and identifies gaps | AI assist | analysis | DataPie |
| 5 | **ba-requirements-draft** | Drafts BRD, user stories, acceptance criteria, and requirements matrix | AI draft + approve | writing | Document |
| 6 | **ba-traceability-packet** | Builds traceability matrix, review summary, and executive review deck | AI draft + approve | analysis | CheckmarkCircle |
| 7 | **ba-review-routing** | Routes review packet to stakeholders for feedback and sign-off | AI act within policy | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Request          │  Normalize request → structured case record
│     Intake           │  Validates fields, checks duplicates,
│     (ba-request-     │  generates Case ID (AC-YYYY-NNN),
│      intake)         │  assigns analyst, sets initial priority
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Discovery        │  Assemble prior BRDs, process maps,
│     Packet           │  policies, architecture docs, stakeholder
│     (ba-discovery-   │  roster, constraints, decisions
│      packet)         │  Output: Word document (8 sections)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Signal           │  Extract needs, constraints, assumptions,
│     Extraction       │  dependencies, conflicts, open questions
│     (ba-signal-      │  from discovery artifacts
│      extraction)     │  Output: Adaptive Card + Excel inventory
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Theme            │  Cluster signals into themes, detect
│     Synthesis        │  stakeholder conflicts, identify gaps,
│     (ba-theme-       │  surface priority signals
│      synthesis)      │  Output: Adaptive Card + Word report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Requirements     │  Draft BRD, user stories with acceptance
│     Draft            │  criteria, and requirements matrix from
│     (ba-requirements-│  synthesized themes
│      draft)          │  Output: Word BRD + Excel matrix
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Traceability     │  Build traceability matrix, assess review
│     Packet           │  readiness, create review summary and
│     (ba-traceability-│  executive review deck
│      packet)         │  Output: Excel + Word + PowerPoint
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Review           │  Route review packet to stakeholders,
│     Routing          │  send Teams notifications, create
│     (ba-review-      │  Outlook draft review requests
│      routing)        │  Output: Teams messages + Outlook drafts
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  8. Confirm          │  Approve and baseline, request revisions,
│     Disposition      │  or return for additional discovery.
│     (not automated)  │  Human-only step — baseline approval
│                      │  and disposition decisions remain
│                      │  human-owned.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Request Intake | Structured field extraction, validation, duplicate checking, case ID generation — no AI judgment on requirements content |
| **AI act within policy** | Discovery Packet, Review Routing | Retrieves context from approved data sources or routes within documented reviewer registry rules without exercising judgment on requirements content |
| **AI assist** | Signal Extraction, Theme Synthesis | Surfaces extraction and clustering recommendations with confidence levels and conflict detection; analyst reviews before tracker updates |
| **AI draft + approve** | Requirements Draft, Traceability Packet | AI drafts requirements documents and traceability matrices; analyst reviews and confirms before any deliverable is finalized or routed |
| **Human only** | Confirm Disposition and Baseline | Final baseline approval, disposition decisions, and sign-off are always human-owned |

## Governance Controls

### Source Attribution and Traceability

- Every extracted signal traces to a specific source document, author, and date
- Every requirement traces to at least one signal or is explicitly marked "Analyst judgment — requires confirmation"
- The traceability matrix calculates coverage percentage — packages below 90% are flagged as not review-ready
- Unlinked requirements (no source support) are visibly flagged for review
- Information from informal sources (chat, email threads) is labeled "unverified until confirmed"

### Conflict Transparency

- Contradictory stakeholder signals are always surfaced with both sides and full source attribution
- Conflicts are never resolved automatically — every conflict is marked "Requires analyst resolution"
- The theme synthesis report includes a dedicated Conflict Report section
- The review summary carries forward all unresolved conflicts for reviewer visibility

### Scope Discipline

- No skill introduces requirements not present in the extracted signal inventory
- Themes supported by only one source are flagged as potentially incomplete
- Gap identification is advisory ("analyst should verify"), not an assertion of missing requirements
- Requirements from low-confidence or single-source signals receive visible flags
- The requirements draft includes an Open Questions section — unresolved items are never suppressed

### Template and Glossary Compliance

- Requirements drafts follow approved templates from SharePoint when available
- Domain terminology is checked against the enterprise glossary — new terms are flagged
- BRD structure follows the organization's standard sections (Executive Summary through Open Questions)
- User stories follow the standard format: As a [role], I want [capability], so that [benefit]

### Review Integrity

- Review routing verifies traceability coverage meets the minimum threshold before distributing
- Blocking items (unresolved critical conflicts, coverage below threshold) prevent routing
- All reviewers must be in the approved stakeholder roster before notifications are sent
- Review deadlines enforce a minimum 2 business day review period
- Formal review requests are created as Outlook drafts — never auto-sent

### Audit Trail

- Every analysis case records the source event, requestor, creation timestamp, and Case ID
- Every signal records the source document, author, date, confidence level, and extraction category
- Every theme records supporting signal IDs, source diversity, and conflict status
- Every requirement records its traceability status (Linked, Analyst Judgment, or Unlinked)
- Every routing decision records the reviewer names, sent date, and review status
- The request tracker spreadsheet serves as the Cowork-accessible audit record

## Signal Categories

| Category | Description | Example |
|----------|-------------|---------|
| **Stated Business Needs** | Explicit stakeholder requests or desired capabilities | "We need the system to support bulk uploads" |
| **Constraints** | Technical, regulatory, timeline, or budget limitations | "Must comply with SOX requirements" |
| **Assumptions** | Stated or implied assumptions about the solution space | "Assumes current vendor contract will be renewed" |
| **Decisions Already Made** | Pre-existing decisions that constrain the solution | "Leadership approved the cloud-first approach" |
| **Open Questions** | Items requiring stakeholder clarification | "Unclear whether mobile access is in scope" |
| **Dependencies** | Dependencies on other systems or teams | "Requires data feed from the finance system" |
| **Conflicting Statements** | Contradictory signals from different sources | "Marketing wants self-service; Compliance wants approval workflow" |

## Requirements Package Outputs

| Output | Format | Produced By |
|--------|--------|-------------|
| **Business Requirements Document (BRD)** | Word document (10 sections: Executive Summary through Open Questions) | Requirements Draft |
| **User Stories with Acceptance Criteria** | Word document (Given/When/Then format, grouped by theme) | Requirements Draft |
| **Requirements Matrix** | Excel workbook (Req ID, Theme, Statement, Type, Priority, Source Signal, Status, Owner) | Requirements Draft |

## Requirement Types

| Type | Description |
|------|-------------|
| **Functional** | Capabilities the system or process must provide, organized by theme |
| **Non-Functional** | Performance, security, compliance, usability, and availability requirements |
| **Business Rules** | Rules that constrain or govern system or process behavior |

## Traceability Statuses

| Status | Meaning |
|--------|---------|
| **Linked** | Requirement traces to one or more signals with clear source attribution |
| **Analyst Judgment** | Requirement was added by analyst direction, not from a source signal — requires explicit confirmation |
| **Unlinked** | Requirement has no source support — flagged for review |

## Review Readiness Checks

| Check | Criteria |
|-------|---------|
| Requirement IDs | All requirements have assigned Req IDs |
| Traceability coverage | Coverage percentage meets minimum threshold (typically 90%+) |
| Open questions | All open questions are documented |
| Conflict resolution | All conflicts are documented with resolution status |
| Stakeholder roster | Complete roster with all reviewers identified |
| Template compliance | BRD structure follows the approved template |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find request tracker, prior BRDs, process maps, policies, templates, taxonomy, glossary, signal inventory, theme reports, traceability packets |
| SearchM365 (email) | Request Intake, Discovery Packet — find request emails, decision threads, stakeholder communications |
| SearchM365 (teams) | Request Intake, Discovery Packet, Signal Extraction — find related discussions, informal context, coordination threads |
| ReadFileContent | All skills — read tracker, templates, prior BRDs, policies, discovery packets, signal inventories, theme reports |
| GetDriveChildren | Discovery Packet, Traceability Packet — browse project folders, verify case folder completeness |
| SearchPeople / GetUserDetails | Request Intake, Discovery Packet, Review Routing — resolve requestor, stakeholder, and reviewer identities |
| GetManagerDetails / GetDirectReportsDetails | Discovery Packet — map org structure for stakeholder roster |
| GetMyDetails | Request Intake — get current user info for Assigned Analyst field |
| GetMessage | Request Intake — read full email content for request details |
| GetMeetingTranscript | Signal Extraction — extract signals from meeting recordings |
| PostMessage | Review Routing — send Teams review notifications to stakeholders |
| CreateDraftMessage | Review Routing — create formal Outlook review request drafts |
| CreateEvent | Review Routing — create review deadline calendar holds |
| render_ui (Adaptive Card) | Request Intake, Signal Extraction, Theme Synthesis, Traceability Packet — decision surfaces, extraction summaries, theme reports, readiness assessments |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Analysis case record | Excel tracker row | Request Intake |
| Discovery packet | Word document (8 sections) | Discovery Packet |
| Signal inventory | Adaptive Card summary + Excel workbook | Signal Extraction |
| Theme synthesis report | Adaptive Card summary + Word report (6 sections) | Theme Synthesis |
| Business requirements document | Word document (10 sections) | Requirements Draft |
| User stories | Word document (As a/I want/So that + Given/When/Then) | Requirements Draft |
| Requirements matrix | Excel workbook | Requirements Draft |
| Traceability matrix | Excel workbook | Traceability Packet |
| Review summary | Word document | Traceability Packet |
| Executive review deck | PowerPoint (5 slides) | Traceability Packet |
| Review readiness card | Adaptive Card | Traceability Packet |
| Review notifications | Teams messages | Review Routing |
| Formal review requests | Outlook draft emails | Review Routing |
| Review deadline holds | Calendar events | Review Routing |

## Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Stakeholder input and interview notes | Attributed to source; informal sources labeled "unverified until confirmed" |
| Conflicting stakeholder positions | Presented with attribution to both sides; never resolved by AI |
| Draft requirements | Clearly labeled as drafts; never finalized without analyst confirmation |
| Priority and scope decisions | Analyst-owned; AI surfaces recommendations but does not commit |
| Traceability coverage gaps | Visibly flagged; packages below threshold blocked from review routing |

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── ba-request-intake/SKILL.md
├── ba-discovery-packet/SKILL.md
├── ba-signal-extraction/SKILL.md
├── ba-theme-synthesis/SKILL.md
├── ba-requirements-draft/SKILL.md
├── ba-traceability-packet/SKILL.md
└── ba-review-routing/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Request tracker** — shared Excel workbook (Analysis Case ID, Request ID, Description, Requestor, Business Domain, Impacted Application, Priority, Status, Created Date, Discovery Status, Synthesis Status, Review Status, Assigned Analyst, Target Date)
- **Requirements templates** — BRD template, user story format, acceptance criteria template following the organization's standard structure
- **Requirements taxonomy** — standard categories for grouping requirements (functional, non-functional, business rules, data, integration, security, compliance)
- **Domain glossary** — enterprise terminology definitions for consistency checking
- **Stakeholder roster template** — standard format for documenting stakeholders with name, role, department, email, and relevance
- **Review checklist** — standard criteria for assessing review readiness and traceability coverage thresholds
- **Traceability standard** — organization's requirements for source-to-requirement linkage and minimum coverage thresholds
- **Prior approved BRDs** — reference packages for template structure and style consistency

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Request Intake | "new requirements request", "intake request for [project]", "set up analysis case for [topic]", "new change request from [stakeholder]" |
| Discovery Packet | "build discovery packet for [request]", "gather context for requirements", "what background do we have for [project]", "discovery for analysis case [ID]" |
| Signal Extraction | "extract requirements from these documents", "pull needs from workshop notes", "what are the stakeholder requirements", "extract signals from [source]" |
| Theme Synthesis | "synthesize requirements themes", "cluster these needs", "reconcile stakeholder input", "find conflicts in requirements" |
| Requirements Draft | "draft requirements for [request]", "generate BRD", "write user stories", "draft acceptance criteria", "prepare requirements package" |
| Traceability Packet | "prepare review packet", "build traceability matrix", "link requirements to sources", "review readiness check" |
| Review Routing | "send for review", "route requirements to reviewers", "request sign-off", "distribute review packet", "notify reviewers" |

## Implementation Roadmap

### Wave 1 — Intake and Discovery

- Create the shared request tracker in SharePoint with standard columns
- Upload requirements templates, taxonomy, glossary, and review checklist to SharePoint
- Build skills: `ba-request-intake`, `ba-discovery-packet`, `ba-signal-extraction`
- Operate in AI assist mode — all outputs presented via Adaptive Card or document for analyst review
- Test with 5–10 real requirements requests across different business domains

### Wave 2 — Synthesis and Drafting

- Build skills: `ba-theme-synthesis`, `ba-requirements-draft`, `ba-traceability-packet`
- Promote intake to write mode (creates case records in tracker after confirmation)
- Promote signal extraction to write-back mode (updates signal inventory after analyst confirmation)
- Promote requirements draft to template-aware mode (matches approved BRD template structure)
- Add traceability coverage calculation and review readiness assessment
- Measurement targets: above 90% traceability coverage, above 85% signal attribution accuracy

### Wave 3 — Review Routing and Optimization

- Build skill: `ba-review-routing`
- Promote routing to active mode (sends Teams notifications and creates Outlook drafts after confirmation)
- Add Graph Connectors for Jira/Azure DevOps requirements synchronization
- Add change request impact pre-assessment (comparing new requests against baselined requirements)
- Add proactive gap detection: scheduled prompt flagging cases with aging open questions or unresolved conflicts
- Measurement targets: 30% reduction in requirements cycle time, above 95% traceability at review, below 15% rework rate after baseline

## Implementation Notes

- **Baseline disposition is intentionally not automated** — final approval and baseline decisions carry accountability that cannot be delegated to AI
- **Extraction, synthesis, and drafting are separate skills** — separating signal extraction from theme synthesis from requirements drafting prevents the common anti-pattern of going from raw stakeholder input to finished requirements in one step, enabling analyst review at each stage
- **Conflicts are never auto-resolved** — stakeholder conflicts represent legitimate differences in business priorities that require human judgment to reconcile
- **Traceability is enforced, not optional** — every requirement must link to source evidence or be explicitly flagged as analyst judgment; unlinked requirements cannot be silently included
- **Templates and glossary compliance maintain organizational consistency** — requirements that introduce new terms or deviate from approved templates are flagged, not silently accepted
- **The request tracker is the canonical state** — all skills read from and write to the shared tracker; it serves as the process state store and audit record
- **Review routing enforces minimum quality gates** — packages with unresolved blocking items or insufficient traceability coverage cannot be distributed to reviewers
- **Informal sources are treated as provisional** — information from chat or email threads is labeled "unverified until confirmed" to distinguish it from formally documented requirements
