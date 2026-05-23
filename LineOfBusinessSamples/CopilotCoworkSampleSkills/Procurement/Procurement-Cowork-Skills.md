# Procurement Supplier Onboarding and Risk Review — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Supplier Onboarding and Risk Review** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the supplier onboarding process — from request intake through review-ready approval packet — into AI-assisted capabilities within Microsoft 365.

The workflow supports supplier request signals from email, Teams, intake forms, and direct submission, guiding each through structured normalization, context assembly, gap and risk detection, review routing, outreach drafting, and reviewer packet summarization with strict controls for sanctions screening, bank detail sensitivity, spend authority thresholds, duplicate supplier detection, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **proc-supplier-intake** | Normalizes inbound supplier requests into structured onboarding cases | Deterministic automation | analysis | TaskListLtr |
| 2 | **proc-context-packet** | Assembles supplier profile, policies, checklists, and reviewer contacts | AI act within policy | analysis | SearchSparkle |
| 3 | **proc-gap-risk-detect** | Detects missing documents, checklist gaps, and risk indicators | AI assist | analysis | Tag |
| 4 | **proc-review-routing** | Routes case to required risk, legal, tax, AP, and category reviewers | AI draft + approve | communication | Mail |
| 5 | **proc-supplier-comms** | Drafts outreach, follow-ups, escalation notices, and status updates | AI draft + approve | communication | Mail |
| 6 | **proc-reviewer-packet** | Summarizes all case artifacts into a review-ready approval packet | AI draft + approve | analysis | Flag |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Supplier Intake  │  Normalize supplier request → structured case
│     (proc-supplier-  │  Validates fields, checks duplicates,
│      intake)         │  assigns procurement analyst
│                      │  SLA: 3 business days to review-ready packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble supplier profile, onboarding
│     (proc-context-   │  policies, document checklists, screening
│      packet)         │  requirements, reviewer contacts
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Gap and Risk     │  Compare checklist against uploaded
│     Detection        │  documents; flag missing items, expired
│     (proc-gap-risk-  │  certs, sanctions risk, duplicate
│      detect)         │  supplier patterns
│                      │  Output: Adaptive Card + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Review Routing   │  Route to required reviewers per approval
│     (proc-review-    │  matrix, spend tier, and risk flags;
│      routing)        │  send notifications and calendar holds
│                      │  Output: Teams messages + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Supplier Comms   │  Draft missing document requests, reviewer
│     (proc-supplier-  │  notifications, risk escalations, AP
│      comms)          │  handoffs, and status updates
│                      │  Output: Outlook drafts + Teams messages
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Reviewer Packet  │  Consolidate all case artifacts into
│     (proc-reviewer-  │  a review-ready evidence packet with
│      packet)         │  readiness assessment
│                      │  Output: Word packet + Adaptive Card
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm          │  Confirm supplier is approved, rejected,
│     Onboarding       │  or returned for more information.
│     Disposition      │  Human-only step — final supplier
│     (not automated)  │  approval carries procurement
│                      │  accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Supplier Intake | Structured field extraction, validation, duplicate checking — no AI judgment on supplier risk or review requirements |
| **AI act within policy** | Context Packet | Retrieves approved context from defined policy repositories and checklists without exercising judgment on risk level or approval path |
| **AI assist** | Gap and Risk Detection | Surfaces checklist gaps and risk indicators with evidence; analyst reviews before tracker updates |
| **AI draft + approve** | Review Routing, Supplier Comms, Reviewer Packet | AI recommends routing, drafts communications, or assembles evidence packets; analyst reviews and confirms before actions are taken |
| **Human only** | Confirm Disposition | Final supplier approval, rejection, or return-for-info is always human-owned — carries procurement accountability and spend authority |

## Governance Controls

### Sanctions and Compliance Risk

- Sanctions, debarment, and compliance risk flags are always presented with elevated visibility and explicit "requires human review" labels
- No skill auto-clears a risk flag — all sanctions and compliance alerts require human review and documented disposition
- Suppliers in sanctions-sensitive geographies are flagged at intake for enhanced due diligence
- Risk flags are never suppressed or downgraded by any skill — all flags remain visible to the analyst

### Bank Detail and Tax Data Sensitivity

- Bank account details, tax identifiers (EIN/SSN), and insurance policy numbers are never included in context packets, Teams messages, Adaptive Cards, or email communications
- Bank detail collection is handled through dedicated secure processes — skills reference the secure banking form process, never include actual banking data
- The reviewer packet references sensitive documents by name and location without reproducing their content
- AP handoff communications include banking collection instructions but never banking data

### Spend Authority and Approval Thresholds

- Spend tier determines the approval authority chain — skills reference the approval matrix to route to the correct level
- Estimated annual spend above elevated thresholds triggers additional review requirements (procurement manager, VP/director)
- If estimated spend is not available, skills flag for analyst confirmation before routing — spend tier cannot be assumed
- High-spend categories combined with risk flags require explicit category manager acknowledgment before review routing

### Duplicate Supplier Detection

- Every intake checks the onboarding tracker and vendor master for existing records with the same supplier name
- Duplicate detection prevents duplicate payment risk and ensures consistency in the vendor master
- If a prior closed case exists for the same supplier, the skill presents reactivation versus new case options

### Audit Trail

- Every onboarding case records the requester, intake actor, creation timestamp, and Case ID
- Every gap and risk finding records the source evidence (document, screening system, or manual input) and confidence level
- Every routing decision records assigned reviewers, routing rationale, deadlines, and confirming analyst
- Every communication records the type, recipient, channel, and confirming analyst
- The Excel onboarding tracker serves as the Cowork-accessible audit record

### Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Bank routing numbers | Never in any generated artifact — secure collection process only |
| Tax identifiers (EIN/SSN) | Never in context packets, communications, or reviewer summaries |
| Screening results (sanctions, debarment) | Internal only — never in supplier-facing communications |
| Insurance policy numbers | Referenced by document name, not reproduced |
| Supplier financial health data | Internal reviewer use only |

## Reviewer Categories

| Reviewer | Required When | Review Scope |
|----------|--------------|-------------|
| **Category manager** | All supplier onboarding cases | Category fit, business need, supplier qualification |
| **Risk reviewer** | Risk flags, sanctions-sensitive geography, high-risk tier | Sanctions screening, debarment, compliance risk, due diligence |
| **Legal reviewer** | Contracts, IP, data processing, regulated services | Contract terms, liability, IP protection, data processing agreements |
| **Tax reviewer** | International suppliers, tax classification questions | W-8/W-9 validation, withholding, tax treaty applicability |
| **AP onboarding specialist** | All cases (final step before activation) | Banking validation, payment terms, vendor master entry |
| **Procurement manager** | Spend above elevated threshold, escalated cases | Spend authority, strategic alignment, exception approval |

## Spend Tiers

| Tier | Threshold | Approval Authority |
|------|-----------|-------------------|
| **Tier 1** | Below standard threshold | Category manager |
| **Tier 2** | Above standard, below elevated threshold | Category manager + procurement manager |
| **Tier 3** | Above elevated threshold | Category manager + procurement manager + VP/director |

## Required Onboarding Documents

| Document Type | Required When | Risk If Missing |
|--------------|--------------|----------------|
| **W-9 / W-8** | All US / international suppliers | Tax withholding and reporting compliance |
| **Insurance certificate** | Categories with liability exposure | Uninsured liability risk |
| **Banking form** | All suppliers (for payment setup) | Payment processing blocked |
| **Master service agreement** | Service providers | Uncontracted spend |
| **NDA** | Suppliers with access to confidential information | IP and confidentiality exposure |
| **Data processing agreement** | Suppliers handling personal data | Privacy and GDPR compliance |
| **Diversity certification** | Diversity-classified suppliers | Diversity spend tracking accuracy |

## Risk Indicators

| Risk Category | Indicators | Severity |
|--------------|-----------|----------|
| **Sanctions or debarment** | OFAC, EU sanctions, or debarment list match | Critical |
| **Duplicate supplier** | Matching name, address, or tax ID in vendor master | High |
| **Expired insurance** | Insurance certificate past expiration date | High |
| **Sanctions-sensitive geography** | Supplier in comprehensively sanctioned country | High |
| **Insufficient coverage** | Insurance amounts below category minimums | Medium |
| **Missing critical documents** | W-9, banking form, or required contract not received | Medium |
| **High-spend tier** | Estimated spend exceeds elevated threshold | Medium |
| **Missing screening** | Required screening not yet completed | Medium |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find onboarding tracker, policies, checklists, approval matrix, templates, supplier documents, screening results |
| SearchM365 (connectors) | Context Packet, Gap/Risk Detection — retrieve vendor master data and screening results via Graph Connector |
| SearchM365 (email) | Supplier Intake, Reviewer Packet — find request emails, supplier correspondence, reviewer comments |
| SearchM365 (teams) | Supplier Intake — find supplier requests in Teams channels |
| ReadFileContent | All skills — read tracker, policies, checklists, approval matrix, templates, context packets |
| GetDriveChildren | Context Packet, Gap/Risk Detection, Reviewer Packet — inventory documents in supplier onboarding folder |
| SearchPeople / GetUserDetails | Supplier Intake, Context Packet, Review Routing — resolve requester, analyst, and reviewer identities |
| GetManagerDetails / GetDirectReportsDetails | Review Routing — resolve escalation paths for elevated spend tiers |
| PostMessage | Review Routing, Supplier Comms — Teams notifications to reviewers and coordination channels |
| CreateDraftMessage | Supplier Comms — formal communications as Outlook drafts |
| CreateEvent | Review Routing — review deadline calendar holds |
| render_ui (Adaptive Card) | Supplier Intake, Gap/Risk Detection, Review Routing, Reviewer Packet — decision surfaces and status summaries |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Onboarding case record | Excel tracker row | Supplier Intake |
| Context packet | Word document | Context Packet |
| Gap and risk report | Adaptive Card + Excel tracker update | Gap/Risk Detection |
| Routing recommendation | Adaptive Card | Review Routing |
| Reviewer notifications | Teams direct messages | Review Routing |
| Review deadline holds | Calendar events | Review Routing |
| Missing document requests | Outlook draft emails | Supplier Comms |
| Risk escalation notices | Outlook draft emails | Supplier Comms |
| AP handoff instructions | Outlook draft emails | Supplier Comms |
| Status updates | Outlook draft emails | Supplier Comms |
| Reviewer packet (formal) | Word document | Reviewer Packet |
| Reviewer packet (quick view) | Adaptive Card | Reviewer Packet |

## Federated Data Access

Supplier onboarding depends on vendor master systems, risk screening tools, and procurement platforms. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | ERP vendor master systems (SAP MM, Oracle Supplier Hub) for supplier profiles and classification; risk screening services (Dun & Bradstreet, World-Check, LexisNexis) for sanctions and compliance screening; procurement platforms (Coupa, SAP Ariba) for workflow state |
| **Tier 2** | SharePoint Bridge | Approval matrices, onboarding checklists, category policies, geography requirements, and screening result exports maintained in SharePoint via Power Automate sync |
| **Tier 3** | Manual Input | Screening results from niche tools, verbal clarifications, and manual override rationale captured via structured intake prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint for approval matrices, checklists, and policies) and Tier 3 for screening results. Introduce Graph Connectors for ERP and risk platforms in Wave 2 after skill workflows are proven.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── proc-supplier-intake/SKILL.md
├── proc-context-packet/SKILL.md
├── proc-gap-risk-detect/SKILL.md
├── proc-review-routing/SKILL.md
├── proc-supplier-comms/SKILL.md
└── proc-reviewer-packet/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Onboarding tracker** — shared Excel workbook (Case ID, Supplier Name, Requester, Spend Category, Geography, Priority, Status, Created Date, Assigned Analyst, Risk Tier, Target Completion Date, Checklist Completion %)
- **Onboarding checklist** — master checklist with required items by spend category and geography
- **Category-specific policies** — onboarding requirements per spend category
- **Geography-specific requirements** — regulatory and compliance requirements per supplier geography
- **Approval matrix** — reviewer requirements by spend tier, risk level, category, and geography
- **Screening requirements** — sanctions, debarment, conflict of interest, and financial health policies
- **Insurance requirements** — minimum coverage amounts by spend category
- **Communication templates** — missing document request, reviewer notification, escalation notice, AP handoff, status update
- **Supplier onboarding folder template** — per-supplier SharePoint folder for evidence documents

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Supplier Intake | "new supplier request", "onboard supplier [name]", "set up supplier case for [company]" |
| Context Packet | "build supplier packet for [supplier]", "assemble onboarding context", "what do we need for [supplier] onboarding" |
| Gap/Risk Detection | "check supplier gaps", "what's missing for [supplier]", "supplier risk check", "onboarding completeness check" |
| Review Routing | "route supplier for review", "who needs to review [supplier]", "assign reviewers for supplier case" |
| Supplier Comms | "draft supplier email", "remind about missing documents", "supplier follow-up", "AP handoff" |
| Reviewer Packet | "prepare review packet", "summarize supplier case for review", "build approval packet" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel onboarding tracker in SharePoint with standard columns
- Upload onboarding policies, checklists, insurance requirements, and approval matrix to SharePoint
- Create per-supplier SharePoint folder template for evidence documents
- Build skills: `proc-supplier-intake`, `proc-context-packet`, `proc-gap-risk-detect`, `proc-supplier-comms`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 5-10 real onboarding cases

### Wave 2 — Routing and Policy-Cited Outputs

- Build skills: `proc-review-routing`, `proc-reviewer-packet`
- Promote intake to write mode (creates case records after confirmation)
- Promote gap detection to write-back mode (updates tracker after confirmation)
- Promote comms to send-after-approval for internal reviewer notifications
- Set up daily scheduled prompt: check for cases approaching SLA deadline with incomplete status or unassigned reviewers
- Measurement targets: above 70% draft acceptance rate, below 10% incorrect routing rate

### Wave 3 — Advanced Assembly and Proactive Detection

- Introduce Graph Connectors for vendor master and risk screening data
- Add bounded multi-reviewer packet assembly for complex multi-reviewer cases
- Add proactive detection: flag cases with no activity for 2+ business days, highlight recurring missing document patterns, suggest process improvements
- Measurement targets: 30% cycle time reduction, above 85% gap detection accuracy, above 90% duplicate detection rate, zero unauthorized approval incidents

## Implementation Notes

- **Onboarding disposition is intentionally not automated** — final supplier approval carries procurement accountability and spend authority that cannot be delegated to AI
- **No skill auto-clears a risk flag** — sanctions, debarment, and compliance alerts always require human review and documented disposition
- **Bank details and tax identifiers are never surfaced in any generated artifact** — this is a permanent architectural constraint for data sensitivity
- **The vendor master is the source of truth** — the Excel onboarding tracker is a Cowork-accessible working copy; state drift is mitigated by Power Automate sync
- **Spend authority thresholds drive review routing** — the approval matrix in SharePoint defines which approval chain applies; skills read this dynamically so procurement leadership can update thresholds without modifying skills
- **Onboarding policies live in SharePoint and are read dynamically** — category requirements, geography rules, and insurance minimums can be updated without skill modifications
- **Every case action is logged** — creation, gap findings, routing decisions, communications, and overrides are recorded for audit trail
- **The 3 business day SLA** drives urgency for review-ready packet — every skill surfaces SLA status and remaining time
