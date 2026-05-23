# Insurance FNOL and Coverage Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **FNOL (First Notice of Loss) and Coverage Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the claims intake and triage process — from loss notification through review-ready case disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports loss notifications from phone, portal, email, broker, and agent channels, guiding each through structured intake, context assembly, claim classification, evidence gap detection, adjuster routing, and communication drafting with coverage determination prohibition, unfair claims practices act compliance, SIU referral controls, claimant privacy protection, litigation hold awareness, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **ins-fnol-intake** | Normalizes FNOL events into structured claim case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **ins-claim-context** | Assembles policy, claimant, loss narrative, prior history, and evidence into a context packet | AI act within policy | analysis | SearchSparkle |
| 3 | **ins-claim-classifier** | Classifies claim type and severity, recommends handling lane | AI assist | analysis | Tag |
| 4 | **ins-coverage-gap-detection** | Detects missing evidence, screens fraud indicators, tracks regulatory deadlines | AI assist | analysis | Flag |
| 5 | **ins-claim-routing** | Routes claims to adjusters, catastrophe desk, or SIU per routing matrix | AI draft + approve | communication | Mail |
| 6 | **ins-claim-comms** | Drafts claimant letters, deficiency notices, adjuster summaries, and escalation memos | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. FNOL Intake      │  Normalize loss notification → structured claim case
│     (ins-fnol-       │  Validates required fields, checks duplicate claims
│      intake)         │  SLA: 4 business hours triage window
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Claim Context    │  Assemble policy summary, claimant profile,
│     Packet           │  loss narrative, prior claim history, evidence
│     (ins-claim-      │  inventory, jurisdiction and regulatory notes,
│      context)        │  applicable handling guidelines
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Claim Classifier │  Compare case attributes against claims taxonomy,
│     (ins-claim-      │  determine claim type, severity tier,
│      classifier)     │  confidence level, and handling lane
│                      │  Output: Adaptive Card classification report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Coverage and Gap │  Detect missing evidence against checklist,
│     Detection        │  identify coverage path considerations,
│     (ins-coverage-   │  screen fraud indicators, track jurisdiction-
│      gap-detection)  │  specific regulatory deadlines
│                      │  Output: Adaptive Card gap report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Claim Routing    │  Route to standard adjuster, catastrophe desk,
│     (ins-claim-      │  bodily injury team, commercial unit, complex
│      routing)        │  claims, or SIU per routing matrix;
│                      │  enforce severity-based supervisor review
│                      │  Output: Teams notifications + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Claim            │  Draft claimant acknowledgments, evidence
│     Communications   │  deficiency notices, broker updates, adjuster
│     (ins-claim-      │  summaries, SIU referral memos, reservation
│      comms)          │  of rights letter drafts
│                      │  Output: Outlook drafts + Teams coordination
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm claim is review-ready, escalated,
│     Disposition      │  or returned for more intake. Human-only
│     (not automated)  │  step — coverage and claim authority.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | FNOL Intake | Structured field extraction, date validation, duplicate checking — no AI judgment |
| **AI act within policy** | Claim Context Packet | Retrieves approved context from defined sources; does not interpret coverage or make liability assessments |
| **AI assist** | Claim Classifier, Coverage and Gap Detection | Surfaces classification and gap findings as recommendations with confidence levels; adjuster reviews before any action |
| **AI draft + approve** | Claim Routing, Claim Comms | AI recommends routing or drafts communications; adjuster or claims supervisor reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final coverage determination, payment authorization, reserve setting, denial decisions, and formal SIU determinations are always human-owned |

## Governance Controls

### Coverage and Claim Authority Prohibition

- No skill may make coverage determinations or state that coverage exists or does not exist
- No skill may authorize payments, set reserves, or approve denials
- No skill may make liability assessments or assign fault
- No skill may use commitment language ("your claim is covered", "we will pay", "you are entitled to")
- All skill outputs are labeled "DRAFT FOR ADJUSTER REVIEW — not a coverage or liability determination"
- Final disposition decisions carry regulatory and litigation accountability and remain human-only

### Regulatory Compliance

- **Unfair claims practices act** compliance is enforced per jurisdiction — state-specific language and timeline requirements are read dynamically from SharePoint, not hard-coded
- **State-mandated acknowledgment deadlines** are tracked and surfaced in evidence gap reports
- **State-mandated proof of loss timelines** are flagged with jurisdiction-specific requirements
- **Mandatory disclosure language** requirements are included in claimant communication templates by jurisdiction
- **4-business-hour SLA** for standard FNOL triage is tracked from intake through disposition

### Fraud and SIU Controls

- Fraud indicators are flagged with elevated visibility but never labeled as fraudulent — SIU determination is exclusively human-owned
- Fraud indicators can never be auto-cleared — once flagged, only a human reviewer may dismiss
- SIU referral routing requires explicit human confirmation — SIU referrals carry regulatory and legal implications
- SIU-related content never appears in claimant-facing or broker-facing communications
- SIU referral memos are internal only and restricted to authorized SIU personnel

### Data Privacy and Masking

- Claimant SSN is masked in all generated outputs
- Financial account numbers are masked in all generated outputs
- Medical details are restricted to bodily injury claim documentation and require explicit confirmation for inclusion
- Full claimant PII never appears in Teams messages — claim folders are linked instead
- SIU investigation details are never surfaced in general claims documentation

### Litigation Hold Awareness

- If a claim is flagged with a litigation hold in the tracker, skills prevent deletion or modification of evidence artifacts
- All communications for claims under litigation hold are flagged as subject to legal hold procedures
- Reservation of rights letters are flagged as requiring legal review before sending

### Audit Trail

- Every claim record includes actor, timestamp, and intake source for creation audit
- Every routing decision records actor, rationale, assigned adjuster, handling lane, and confirming reviewer
- Every document access logs source system, document name, and retrieval timestamp
- Every communication records type, recipient, Claim ID, and timestamp
- The Excel claims tracker serves as both operational state and examination-grade audit record

### Customer Communication Controls

- All claimant-facing communications use approved templates — no freeform claimant messaging
- No commitment language regarding coverage, payment, or claim outcome
- No disclosure of fraud investigation details, SIU status, internal scores, or investigation methodology
- All claimant communications are Outlook drafts — nothing is sent without adjuster or supervisor confirmation
- All drafts are labeled "DRAFT — requires adjuster/supervisor review before sending"

## Claim Type Categories

| Claim Type | Description | Handling Lane |
|------------|-------------|---------------|
| **Property damage — non-catastrophe** | Property loss from fire, water, theft, vandalism, or similar | Standard adjuster |
| **Property damage — catastrophe** | Property loss linked to a declared catastrophe event | Catastrophe desk |
| **Auto physical damage** | Vehicle damage from collision, comprehensive, or uninsured motorist | Auto adjuster |
| **Bodily injury** | Personal injury claims including medical payments | Bodily injury team |
| **General liability** | Third-party liability claims against the insured | Liability adjuster |
| **Commercial property** | Business property loss, business interruption, equipment damage | Commercial adjuster |
| **Theft or burglary** | Loss from criminal theft or burglary | Standard adjuster (with SIU screening) |
| **Water damage or mold** | Water intrusion, pipe burst, mold remediation | Standard adjuster (with specialist referral) |
| **Suspicious loss** | Loss with fraud indicators or SIU trigger flags | SIU referral (requires human confirmation) |

## Severity Tiers

| Severity | Criteria |
|----------|----------|
| **Complex** | Bodily injury with hospitalization; commercial loss exceeding policy threshold; multi-peril event; litigation involvement; coverage disputes |
| **High** | Significant property damage; bodily injury without hospitalization; high-value auto loss; multiple claimants |
| **Medium** | Standard property damage; single-vehicle auto damage; minor injury with medical payments only |
| **Low** | Minor property damage; cosmetic vehicle damage; straightforward claims with complete evidence |

## Evidence Gap Categories

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing proof of loss | Signed proof of loss form not submitted | High |
| Missing photos or documentation | Loss scene photos, damage photos, or supporting documentation not provided | High |
| Missing police report | Police report requested but not received (theft, vandalism, auto accident) | Medium |
| Missing repair estimate | Contractor or body shop estimate not submitted | Medium |
| Missing medical documentation | Medical records or bills not provided (bodily injury claims) | High |
| Missing witness statement | Witness statements referenced but not collected | Medium |
| Missing subrogation documentation | Third-party liability evidence not assembled | Low |

## Fraud Indicator Flags

| Indicator | Description | Visibility |
|-----------|-------------|------------|
| **Repeated claims** | 3+ claims on the same policy within 24 months | Elevated |
| **Same-provider pattern** | Same repair vendor, medical provider, or attorney across multiple claims | Elevated |
| **Conflicting narratives** | Claimant statement conflicts with evidence or system records | Elevated |
| **Recent policy change** | Coverage increase or endorsement change within 90 days before loss | Flagged |
| **Prior SIU referral** | Claimant or policy has prior SIU investigation history | Elevated |
| **Suspicious timing** | Loss reported close to policy cancellation, renewal, or premium increase | Flagged |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | FNOL Intake, Claim Context — find loss notifications, claimant correspondence |
| SearchM365 (files) | All skills — find claims tracker, claims taxonomy, routing matrix, evidence checklists, fraud triggers, templates, regulatory references |
| SearchM365 (connectors) | Claim Context, Claim Classifier, Coverage Gap Detection — retrieve policy and claims data from policy admin and claims platform via Graph Connector |
| ReadFileContent | All skills — read tracker, taxonomy, routing matrix, checklists, policy documents, templates, regulatory language |
| GetDriveChildren | Claim Context, Coverage Gap Detection — browse claim evidence folders |
| SearchPeople / GetUserDetails | All skills — resolve adjusters, agents, brokers, claims supervisors, SIU analysts |
| GetManagerDetails / GetDirectReportsDetails | Claim Routing — claims operations org structure for escalation paths |
| CreateDraftMessage | Claim Routing, Claim Comms — Outlook drafts for claimant communications and formal notifications (never auto-send) |
| PostMessage | Claim Routing, Claim Comms — Teams notifications to adjusters and coordination channels |
| CreateEvent | Claim Routing — calendar holds for SLA review deadlines |
| render_ui (Adaptive Card) | FNOL Intake, Claim Classifier, Coverage Gap Detection, Claim Routing — confirmations, classifications, gap reports |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Claim case record | Excel tracker row | FNOL Intake |
| Claim context packet | Word document | Claim Context |
| Classification report | Adaptive Card | Claim Classifier |
| Evidence gap report | Adaptive Card | Coverage Gap Detection |
| Routing recommendation | Adaptive Card | Claim Routing |
| Adjuster assignment notifications | Teams direct messages | Claim Routing |
| SLA deadline calendar holds | Calendar events | Claim Routing |
| Claimant acknowledgment letter | Outlook draft | Claim Comms |
| Evidence deficiency notice | Outlook draft | Claim Comms |
| Broker/agent status update | Outlook draft | Claim Comms |
| Adjuster assignment summary | Outlook draft or Teams message | Claim Comms |
| SIU referral memo | Outlook draft (restricted) | Claim Comms |
| Reservation of rights letter draft | Outlook draft (requires legal review) | Claim Comms |

## Federated Data Access

Insurance claims workflows depend on external systems (claims platform, policy administration, SIU database, catastrophe feeds). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Policy administration system data, claims platform status, prior claim history, and fraud indicator flags indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Claims taxonomy, evidence checklists, routing matrices, fraud trigger checklists, communication templates, catastrophe event bulletins, and regulatory language maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Actuarial data, reinsurance details, and data points not available through integration, captured via structured prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (taxonomy, checklists, routing matrix, fraud triggers, templates, regulatory language) and Tier 3 for claims platform details from manual lookups. Introduce Graph Connectors for Guidewire or Duck Creek data in Wave 2 after skill workflows are proven and tenant admin has evaluated connector options.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── ins-fnol-intake/SKILL.md
├── ins-claim-context/SKILL.md
├── ins-claim-classifier/SKILL.md
├── ins-coverage-gap-detection/SKILL.md
├── ins-claim-routing/SKILL.md
└── ins-claim-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Claims tracker** — shared Excel workbook (Claim ID, Claimant Name, Policy Number, Date of Loss, Loss Type, Severity, Jurisdiction, Product Line, Reporting Channel, Status, Created Date, Assigned To, Handling Lane, Evidence Completion %, Fraud Flag, SLA Deadline, Audit Log)
- **Claims taxonomy** — claim type definitions, severity rubric, handling lane criteria
- **Evidence checklists** — required documents by claim type, product line, and jurisdiction
- **Fraud trigger checklist** — SIU referral criteria, suspicious pattern indicators, thresholds
- **Routing matrix** — who handles what by claim type, severity, product line, jurisdiction, and escalation level
- **Communication templates** — approved templates for claimant acknowledgment, deficiency notice, broker update, adjuster summary, SIU referral memo, reservation of rights letter
- **Regulatory language guides** — state-specific unfair claims practices act requirements, mandatory disclosure language, proof of loss timelines, acknowledgment deadlines
- **Claim evidence folder structure** — per-claim folders in SharePoint for photos, police reports, repair estimates, proof of loss forms, medical documents
- **Catastrophe event bulletins** — active catastrophe declarations affecting claim classification and routing

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| FNOL Intake | "new loss reported", "FNOL for [claimant]", "set up claim for [policy]", "loss notification from [broker]", "new claim intake" |
| Claim Context | "build claim context for [claim ID]", "assemble claim packet", "gather policy details for this loss", "what do we know about claim [number]" |
| Claim Classifier | "classify this claim", "what type of claim is this", "assess severity for [claim ID]", "claim triage classification" |
| Coverage Gap Detection | "check evidence gaps for [claim ID]", "what's missing on this claim", "coverage prep check", "fraud indicator check" |
| Claim Routing | "route this claim", "assign adjuster for [claim ID]", "where should this claim go", "recommend handling lane" |
| Claim Comms | "draft claimant letter for [claim ID]", "send deficiency notice", "prepare adjuster summary", "broker update for this claim" |

## Implementation Roadmap

### Wave 1 — Foundation (Intake, Context, Classification, Gap Detection)

- Create the shared Excel claims tracker in SharePoint
- Upload claims taxonomy, severity rubric, evidence checklists, fraud trigger checklist, and regulatory language guides
- Create per-claim evidence folder structure in SharePoint
- Upload catastrophe event bulletins and communication templates
- Build skills: `ins-fnol-intake`, `ins-claim-context`, `ins-claim-classifier`, `ins-coverage-gap-detection`
- Operate in AI assist mode — all outputs presented for adjuster review
- Test with one product line (e.g., homeowners property damage) and one intake team

### Wave 2 — Routing and Communications

- Build skills: `ins-claim-routing`, `ins-claim-comms`
- Promote intake to write mode (creates claim records after confirmation)
- Promote classifier and gap detection to write-back mode (updates tracker after adjuster confirmation)
- Introduce Graph Connectors for policy administration system if available
- Set up daily scheduled prompt to check for claims approaching SLA deadlines with incomplete evidence or unassigned routing

### Wave 3 — Optimization and Expansion

- Add bounded multi-document claim packet assembly for complex losses (multi-peril, bodily injury, commercial)
- Add proactive detection of repeated suspicious patterns (same provider, same location, frequent claims) via scheduled prompt
- Introduce Graph Connectors for claims platform and SIU system
- Add catastrophe event monitoring via scheduled prompt
- Expand to additional product lines (auto, commercial property, general liability)
- Measurement targets: 85%+ classification accuracy, 85%+ gap detection rate, below 5% incorrect routing rate, 75%+ draft acceptance rate, zero unauthorized claim actions, zero missing audit fields, below 10% adjuster rework rate

## Implementation Notes

- **Triage disposition is intentionally not automated** — final coverage and claim handling authority decisions carry regulatory and litigation accountability that cannot be delegated to AI
- **No skill makes coverage determinations** — this is a permanent architectural constraint; coverage interpretation, payment authority, reserve setting, and formal denial are always human-owned
- **Guardrails are regulatory requirements** — in insurance, guardrails like "never use commitment language" and "never disclose SIU status" are driven by unfair claims practices acts and bad-faith litigation exposure, not operational preferences
- **The 4-business-hour SLA** drives urgency throughout the workflow — every skill surfaces SLA status in its output
- **Jurisdiction-specific requirements are read dynamically** from SharePoint at runtime — not hard-coded into skill instructions; this ensures claims operations can update state-specific rules without modifying skills
- **SIU referral is always separated** from standard claims triage — different access controls, different routing, different communication restrictions
- **All claimant communications use approved templates** — no freeform claimant messaging is permitted
- **The Excel tracker serves dual duty** — operational state store and examination-grade audit record; every skill write includes actor, action, timestamp, and prior value
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a gap check on an existing claim, or drafting a deficiency notice without re-running the full workflow)
- **Reservation of rights letters** always require legal review before sending — this is flagged prominently in the skill output
