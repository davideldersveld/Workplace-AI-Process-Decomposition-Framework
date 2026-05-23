# Legal Contract Intake and Clause Deviation Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Contract Intake and Clause Deviation Triage** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the contract triage process — from contract request intake through review-ready disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports contract signals from email, Teams, and direct intake, guiding each through structured normalization, context and playbook assembly, clause deviation detection, review path routing, and communication drafting with strict privilege protection, confidentiality controls, deviation-severity-based escalation, audience-scoped language rules, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **legal-contract-intake** | Normalizes inbound contract requests into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **legal-review-packet** | Assembles contract, playbook, counterparty, and approval context into a review packet | AI act within policy | analysis | SearchSparkle |
| 3 | **legal-deviation-detection** | Identifies clause deviations from the playbook and classifies severity | AI assist | analysis | Tag |
| 4 | **legal-review-routing** | Routes contracts to counsel and approvers based on deviation profile | AI draft + approve | communication | Mail |
| 5 | **legal-triage-comms** | Drafts triage summaries, follow-up requests, status updates, and escalation notices | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Contract Intake  │  Normalize contract request → structured case record
│     (legal-contract- │  Validates required fields, checks duplicates
│      intake)         │  SLA: 1 business day for standard triage completion
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Review Packet    │  Assemble contract document, clause playbook
│     (legal-review-   │  standards, counterparty history, approval
│      packet)         │  requirements, requestor context
│                      │  Output: Word review packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Deviation        │  Compare contract clauses against playbook
│     Detection        │  standards, classify deviations by severity
│     (legal-          │  (Level 1–4), cross-reference prior
│      deviation-      │  exceptions
│      detection)      │  Output: Adaptive Card + Word deviation report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Review Routing   │  Assign counsel by practice area, identify
│     (legal-review-   │  required approvers, determine escalation
│      routing)        │  path based on deviation severity
│                      │  Output: Teams notifications + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Triage Comms     │  Draft triage summaries for counsel,
│     (legal-triage-   │  missing information requests, status
│      comms)          │  updates, escalation notices, compliance
│                      │  referrals
│                      │  Output: Outlook drafts + Teams messages
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Confirm Triage   │  Confirm contract is queued for review,
│     Disposition      │  escalated, or returned for clarification.
│     (not automated)  │  Human-only step — final triage
│                      │  disposition carries legal accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Contract Intake | Structured field extraction, validation, duplicate checking — no AI judgment on contract substance or deviations |
| **AI act within policy** | Review Packet | Retrieves approved context from defined sources (playbooks, policies, directory) without exercising judgment on contract substance |
| **AI assist** | Deviation Detection | Surfaces clause deviations as factual comparisons against playbook standards; analyst reviews before recording |
| **AI draft + approve** | Review Routing, Triage Comms | AI recommends review path or drafts communications; analyst or legal operations reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final triage disposition carries legal accountability and remains human-owned |

## Governance Controls

### Privilege Protection

- No skill produces attorney work product or legal advice
- Deviation detection presents factual comparisons against playbook standards only — never recommends accepting or rejecting a clause
- All counsel-facing and compliance-facing drafts include "PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT" header
- Privileged analysis and internal legal strategy are never exposed in non-privileged channels
- Matter folders in SharePoint are access-scoped to the legal team

### Confidentiality Controls

- No contract terms, clause text, or deviation details in Teams messages or emails to non-legal recipients
- Case references in non-legal communications use Case IDs only, not substance
- Full deviation details and contract text stay in Word documents within SharePoint matter folders
- Internal routing rationale and reviewer assignment logic are never disclosed in business-facing communications

### CLM as Source of Truth

- The CLM platform (Ironclad, Icertis, Agiloft, etc.) is the canonical system of record for matter status
- The Excel contract case tracker in SharePoint is a working copy — not the source of truth
- If tracker data conflicts with CLM platform data (via Graph Connector), the platform data takes precedence
- Bidirectional state synchronization is handled by Power Automate outside of Cowork
- Skills note the dual-state reality when state consistency matters

### Deviation Severity Levels

| Severity | Definition | Action Required |
|----------|------------|-----------------|
| **Level 1 — Within approved range** | Clause matches the approved position or acceptable fallback | No action — clause is within policy |
| **Level 2 — Requires counsel review** | Clause deviates from approved and fallback positions but falls within a negotiable range | Counsel review required before acceptance |
| **Level 3 — Requires senior counsel or business approval** | Clause deviates significantly from policy; acceptance requires elevated authority | Senior counsel or business approver must review |
| **Level 4 — Outside policy — must escalate** | Clause is fundamentally inconsistent with policy or creates unacceptable risk exposure | Must escalate; cannot be approved at standard review level |

### Audience-Scoped Language Rules

| Audience | Contract Terms | Deviation Details | Internal Routing | Legal Analysis | Privilege Header |
|----------|---------------|-------------------|-----------------|----------------|-----------------|
| Counsel | Allowed (reference) | Allowed (summary) | Allowed | Not included (counsel's role) | Required |
| Business requestor | Not allowed | Not allowed | Not allowed | Not allowed | Not required |
| Business stakeholder | Not allowed | Not allowed | Not allowed | Not allowed | Not required |
| Senior counsel / escalation | Allowed (reference) | Allowed (severity summary) | Allowed | Not included | Required |
| Compliance reviewer | Clause area reference | Severity per clause area | Allowed | Not included | Required |

### SLA Accountability

- Standard contract requests must be triaged within 1 business day
- Review timelines are calculated based on urgency and deviation severity profile
- Approaching SLA deadlines are flagged by scheduled prompt monitoring
- SLA enforcement remains the responsibility of the CLM platform — skills track and surface, not enforce

### Data Sensitivity

- No full clause text in email summaries or Teams messages
- No deviation details in communications to non-legal recipients
- No internal routing rationale in business-facing communications
- No other cases' details disclosed in any communication
- Stale playbook data (older than 12 months) flagged in review packets
- Stale counterparty data (older than 6 months) flagged in review packets

### Audit Trail

- Every case record includes requestor, intake source, timestamp, and Case ID for creation audit
- Every deviation finding records the playbook section, contract clause, severity level, and analyst confirmation
- Every routing decision records assigned counsel, required approvers, rationale, and confirming analyst
- Every communication records type, recipient, Case ID, and timestamp
- The Excel contract case tracker serves as the Cowork-accessible audit record
- SharePoint document versioning preserves all changes to case artifacts

## Contract Types

| Type | Description | Typical Complexity |
|------|-------------|-------------------|
| **NDA** | Non-disclosure agreement | Low — standard form, limited deviation surface |
| **MSA** | Master services agreement | High — broad clause coverage, significant negotiation |
| **SOW** | Statement of work | Medium — scoped to specific deliverables and terms |
| **Amendment** | Modification to existing agreement | Medium — depends on scope of changes |
| **License Agreement** | Software or IP license | High — IP clauses, usage restrictions |
| **Services Agreement** | Professional or managed services | Medium to High — liability, SLA, IP clauses |
| **Data Processing Agreement** | Data protection and processing terms | High — regulatory compliance requirements |
| **Other** | Non-standard or specialized agreements | Varies — manual classification recommended |

## Material Clause Categories

| Category | Examples | Privilege Sensitivity |
|----------|----------|----------------------|
| **Indemnification** | Mutual vs. one-way, carve-outs, caps | Elevated |
| **Limitation of liability** | Cap amounts, exclusions, consequential damages | Elevated |
| **Intellectual property** | Ownership, license grants, assignment, background IP | Elevated |
| **Confidentiality** | Scope, duration, permitted disclosures | Standard |
| **Termination** | For convenience, cure periods, survival | Standard |
| **Governing law** | Jurisdiction, venue, arbitration | Standard |
| **Data protection** | Processing terms, security, breach notification | Elevated (regulatory) |
| **Representations and warranties** | Scope, survival, remedies | Standard |
| **Insurance** | Coverage requirements, minimums | Standard |
| **Assignment** | Consent, change of control | Standard |
| **Force majeure** | Covered events, notice, termination rights | Standard |
| **Non-solicitation / non-compete** | Scope, duration, geography | Elevated |

## Reviewer Categories

| Reviewer | Handles | Triggered By |
|----------|---------|-------------|
| **Associate counsel** | Standard contract reviews with Level 1-2 deviations | NDAs, standard SOWs, low-value amendments |
| **Senior counsel** | Complex contracts or Level 3 deviations | MSAs, high-value SOWs, strategic agreements |
| **Practice area lead** | Specialty reviews (IP, employment, regulatory) | Domain expertise required |
| **Deputy general counsel** | Level 4 deviations or authority threshold exceeded | Deviations outside policy, high-value contracts |
| **Business approver** | Business terms requiring business unit sign-off | Value commitments, exclusivity, SLAs |
| **Compliance reviewer** | Regulatory clause deviations | Data protection, export control, anti-corruption |

## Communication Types

| Communication | Audience | Channel | Tone | Privilege Header |
|---------------|----------|---------|------|-----------------|
| Triage summary | Counsel | Outlook draft | Formal, precise, structured | Required |
| Missing information request | Business requestor | Outlook draft | Clear, helpful, specific | Not required |
| Status update | Business stakeholder | Outlook draft | Professional, non-technical | Not required |
| Escalation notice | Senior counsel / legal ops | Teams DM or Outlook draft | Concise, action-oriented | Required |
| Compliance referral | Compliance reviewer | Outlook draft | Formal, factual, precise | Required |
| Disposition confirmation | Business requestor | Outlook draft | Professional, reassuring | Not required |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find case tracker, playbooks, fallback positions, templates, approval matrix, deviation reports, prior exceptions |
| SearchM365 (connectors) | Review Packet, Deviation Detection — retrieve CLM records, counterparty history, prior agreements via Graph Connector |
| SearchM365 (email) | Contract Intake — find original contract request email threads |
| SearchM365 (teams) | Contract Intake — find contract request messages in Teams |
| ReadFileContent | All skills — read tracker, playbooks, contracts, policies, deviation reports, templates |
| GetDriveChildren | Review Packet, Deviation Detection — browse matter folders and template libraries |
| SearchPeople / GetUserDetails | All skills — resolve requestors, counsel, approvers |
| GetManagerDetails / GetDirectReportsDetails | Review Packet, Review Routing — reporting chain and escalation paths |
| ListCalendarView | Review Routing — check counsel availability |
| CreateDraftMessage | Review Routing, Triage Comms — Outlook drafts for all external and counsel-facing communications |
| PostMessage | Review Routing, Triage Comms — Teams messages for internal legal coordination |
| CreateEvent | Review Routing — review deadline calendar holds |
| render_ui (Adaptive Card) | Contract Intake, Deviation Detection, Review Routing — confirmations, deviation summaries, routing recommendations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Case record | Excel tracker row | Contract Intake |
| Review packet | Word document | Review Packet |
| Deviation report (quick) | Adaptive Card | Deviation Detection |
| Deviation report (detailed) | Word document | Deviation Detection |
| Routing recommendation | Adaptive Card | Review Routing |
| Counsel assignment notification | Teams direct message | Review Routing |
| Review deadline calendar hold | Calendar event | Review Routing |
| Triage summary for counsel | Outlook draft | Triage Comms |
| Missing information request | Outlook draft | Triage Comms |
| Status update | Outlook draft | Triage Comms |
| Escalation notice | Teams DM or Outlook draft | Triage Comms |
| Compliance referral | Outlook draft | Triage Comms |
| Disposition confirmation | Outlook draft | Triage Comms |

## Federated Data Access

Contract triage depends on the CLM platform (Ironclad, Icertis, Agiloft, DocuSign CLM), playbook repositories, and approval systems. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | CLM platform contract metadata, matter records, counterparty history, and prior agreement data indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Clause playbooks, approved fallback language, standard templates, approval matrices, counterparty history, and case tracker maintained in SharePoint via Power Automate sync from CLM |
| **Tier 3** | Manual Input | Negotiation context, verbal instructions from counsel, and data points not available through integration, captured via structured prompts with "manual entry" source tagging |

**Recommended pilot approach:** Start with Tier 2 (SharePoint for playbooks, approval matrices, templates, and counterparty history) and Tier 3 for request context from direct intake. Introduce Graph Connectors for CLM integration in Wave 2 after skill workflows are proven and tenant admin has configured connectors.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── legal-contract-intake/SKILL.md
├── legal-review-packet/SKILL.md
├── legal-deviation-detection/SKILL.md
├── legal-review-routing/SKILL.md
└── legal-triage-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Contract case tracker** — shared Excel workbook (Case ID, Counterparty, Contract Type, Business Unit, Requestor, Urgency, Status, Received Date, Assigned Counsel, Deviation Count, Disposition)
- **Clause playbooks** — Word documents defining approved positions and acceptable fallback language per contract type and clause category
- **Approved fallback positions** — companion documents to playbooks with negotiable ranges per clause
- **Standard contract templates** — baseline templates per contract type for structural comparison
- **Approval matrix** — reviewer assignment rules by contract type, value threshold, jurisdiction, and deviation severity
- **Counterparty history** — prior agreements, exception approvals, and negotiation patterns (synced from CLM via Power Automate)
- **Communication templates** — triage summary, missing information request, status update, escalation notice, compliance referral, and disposition confirmation templates
- **Matter folder structure** — per-case SharePoint folders with access scoped to the legal team

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Contract Intake | "new contract request", "contract intake for [counterparty]", "set up legal case for", "log contract request from [requestor]" |
| Review Packet | "build review packet for [case]", "assemble contract context", "pull playbook for [contract type]", "prepare review materials" |
| Deviation Detection | "check for deviations in [contract]", "clause deviation analysis for [case]", "flag non-standard terms", "review contract against playbook" |
| Review Routing | "route this contract for review", "who reviews [contract type]", "assign reviewers for [case]", "determine approval path" |
| Triage Comms | "draft contract summary for [case]", "send missing info request", "legal case update for [ID]", "escalation notice for contract" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel contract case tracker in SharePoint with standard columns
- Upload clause playbooks, approved fallback language, and standard templates to SharePoint
- Create SharePoint folder structure for per-matter case artifacts
- Upload the approval matrix as a SharePoint-hosted Excel workbook
- Build skills: `legal-contract-intake`, `legal-review-packet`, `legal-deviation-detection`
- Operate in AI assist mode — all outputs presented for analyst review
- Test with 5–10 real contract cases over 2 weeks

### Wave 2 — Routing and Communication

- Build skills: `legal-review-routing`, `legal-triage-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote deviation detection to write-back mode (saves deviation report to matter folder after confirmation)
- Set up daily scheduled prompt: check for cases with approaching SLA deadlines
- Introduce Graph Connectors for CLM data if available at the tenant level

### Wave 3 — Optimization and Expansion

- Add bounded multi-document analysis (compare deviations across related agreements for the same counterparty)
- Add proactive detection of high-risk deviation patterns across the contract portfolio via scheduled prompt
- Refine deviation detection thresholds based on counsel override patterns from Waves 1-2
- Measurement targets: 80%+ deviation detection recall, below 5% incorrect routing rate, above 70% summary acceptance rate, zero privileged-access incidents

## Implementation Notes

- **Triage disposition is intentionally not automated** — final legal triage decisions carry legal accountability that cannot be delegated to AI
- **No skill produces legal advice or attorney work product** — this is a permanent architectural constraint; legal judgment remains exclusively human-owned
- **The CLM platform is the source of truth** — the Excel case tracker is a Cowork-accessible working copy; state drift is mitigated by Power Automate sync
- **Privilege protection is the strictest guardrail** — every counsel-facing communication carries a privilege header; no deviation details or contract terms in non-legal channels
- **Deviation detection presents factual comparisons only** — the skill never recommends accepting or rejecting a clause; that judgment belongs to counsel
- **Prior exception approvals provide context, not authorization** — a prior exception for the same counterparty does not auto-approve a current deviation
- **The 1 business day SLA** drives urgency for standard contract triage — every skill surfaces SLA status
- **Stale data is flagged, not silently used** — playbooks older than 12 months and counterparty data older than 6 months are flagged prominently
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running deviation detection on an existing case, or drafting a status update without re-running the full workflow)
- **Regulatory clause deviations trigger compliance referral** — data protection, export control, and anti-corruption deviations are never approved without compliance review
