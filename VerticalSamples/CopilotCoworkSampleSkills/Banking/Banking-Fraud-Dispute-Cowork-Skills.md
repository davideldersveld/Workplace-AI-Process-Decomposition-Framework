# Banking Fraud Alert and Dispute Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Fraud Alert and Dispute Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the fraud and dispute triage process — from alert intake through review-ready case disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports fraud alert events, customer dispute submissions, and contact center reports, guiding each through structured intake, context assembly, scenario classification, evidence gap and risk assessment, analyst routing, and communication drafting with regulatory compliance, data masking, account action prohibition, AML escalation controls, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **bnk-case-intake** | Normalizes fraud alert and dispute events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **bnk-fraud-context-packet** | Assembles account, transaction, prior history, and procedure context into an evidence packet | AI act within policy | analysis | SearchSparkle |
| 3 | **bnk-scenario-classifier** | Classifies fraud scenario or dispute type and recommends handling lane | AI assist | analysis | Tag |
| 4 | **bnk-gap-risk-detection** | Assesses urgency, detects evidence gaps, identifies escalation flags, tracks regulatory timelines | AI assist | analysis | Flag |
| 5 | **bnk-case-routing** | Routes cases to analyst queues or escalation lanes per routing matrix | AI draft + approve | communication | Mail |
| 6 | **bnk-case-comms** | Drafts customer communications, analyst summaries, affidavit requests, and escalation notices | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Case Intake      │  Normalize fraud alert or dispute → structured case record
│     (bnk-case-       │  Validates required fields, checks duplicate alerts
│      intake)         │  SLA: 30-minute triage window
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Fraud Context    │  Assemble customer profile, transaction details,
│     Packet           │  prior case history, merchant context, applicable
│     (bnk-fraud-      │  procedures, evidence inventory
│      context-packet) │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Scenario         │  Compare case attributes against fraud taxonomy,
│     Classifier       │  determine scenario type, confidence level,
│     (bnk-scenario-   │  and handling lane recommendation
│      classifier)     │  Output: Adaptive Card classification report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Gap and Risk     │  Detect missing evidence, assess urgency,
│     Detection        │  identify escalation flags, track Reg E and
│     (bnk-gap-risk-   │  card network deadlines, monitor SLA
│      detection)      │  Output: Adaptive Card risk assessment
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Case Routing     │  Route to fraud ops, dispute ops, chargeback,
│     (bnk-case-       │  or AML escalation per routing matrix;
│      routing)        │  enforce segregation of duties
│                      │  Output: Teams notifications + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Case             │  Draft customer acknowledgments, affidavit
│     Communications   │  requests, evidence follow-ups, analyst
│     (bnk-case-       │  summaries, escalation notices
│      comms)          │  Output: Outlook drafts + Teams coordination
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm case is review-ready, escalated,
│     Disposition      │  or returned for more intake. Human-only
│     (not automated)  │  step — regulatory accountability.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Case Intake | Structured field extraction, date validation, duplicate checking — no AI judgment |
| **AI act within policy** | Fraud Context Packet | Retrieves approved context from defined sources; does not interpret fraud patterns or make risk judgments |
| **AI assist** | Scenario Classifier, Gap and Risk Detection | Surfaces classification and risk findings as recommendations with confidence levels; analyst reviews before any action |
| **AI draft + approve** | Case Routing, Case Comms | AI recommends routing or drafts communications; fraud analyst reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final triage disposition, account actions, and regulatory escalation decisions are always human-owned |

## Governance Controls

### Regulatory Compliance

- **Reg E provisional credit timelines** are tracked explicitly — 10-business-day and 45/90-calendar-day deadlines are flagged as regulatory requirements, not operational preferences
- **Card network chargeback windows** (Visa 120 days, Mastercard 120 days) are monitored and flagged when approaching
- **BSA/AML escalation** is separated from standard fraud triage — AML indicators route to restricted AML team channels only
- **GLBA customer data protection** is enforced through account/card number masking in all outputs
- **CFPB complaint handling** requirements are encoded in dispute procedures read dynamically from SharePoint

### Account Action Prohibition

- No skill may block accounts, restrict cards, reverse transactions, or authorize provisional credit
- No skill may auto-clear fraud indicators or suspicious activity flags
- No skill may close, de-prioritize, or clear a case without analyst confirmation
- Final disposition decisions carry compliance accountability and remain human-only

### Data Masking and Privacy

- Account numbers are masked to last four digits in all generated outputs and communications
- Card numbers are masked to last four digits in all generated outputs and communications
- Full account and card numbers never appear in Word documents, Adaptive Cards, Outlook drafts, or Teams messages
- SAR and suspicious activity investigation status is never surfaced outside AML team channels
- Internal fraud scores and risk ratings are not included in case documentation or customer communications

### Segregation of Duties

- The analyst who performed intake is not auto-assigned to review the same case
- The analyst who classified the case is not the sole reviewer for high-risk cases
- Routing follows the documented routing matrix — no ad hoc analyst assignments
- Escalation paths follow documented procedures, not skill-invented paths

### Audit Trail

- Every case record includes actor, timestamp, and intake source for creation audit
- Every routing decision records actor, rationale, and approver
- Every document access logs source system, document name, and retrieval timestamp
- Every communication records type, recipient, Case ID, and timestamp
- The Excel case tracker serves as both operational state and examination-grade audit record

### Customer Communication Controls

- All customer-facing communications use approved templates — no freeform customer messaging
- No commitment language regarding provisional credit, reimbursement, or case outcome
- No disclosure of fraud investigation details, internal scores, SAR status, or investigation methodology
- All customer communications are Outlook drafts — nothing is sent without analyst confirmation

## Fraud Scenario Types

| Scenario Type | Description | Handling Lane |
|---------------|-------------|---------------|
| **Card-not-present fraud** | Unauthorized online or phone transaction | Fraud Operations |
| **Card-present counterfeit** | Counterfeit card used at POS terminal | Fraud Operations |
| **Account takeover** | Unauthorized access and account manipulation | Fraud Operations (elevated) |
| **Friendly fraud** | Customer disputes legitimate transaction | Dispute Operations |
| **Billing dispute** | Recurring charge, subscription, or merchant error | Dispute Operations |
| **Merchant error** | Duplicate charge, wrong amount, or processing error | Chargeback Operations |
| **Lost or stolen card** | Physical card compromised | Fraud Operations |
| **ATM dispute** | Cash not dispensed or wrong amount | Dispute Operations |
| **AML-related** | Suspicious activity indicators present | AML Escalation (immediate) |

## Evidence Gap Categories

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing customer statement | No written statement from cardholder or account holder | High |
| Missing affidavit | Fraud affidavit not submitted or unsigned | High |
| Missing transaction documentation | Supporting transaction records not provided | Medium |
| Missing merchant response | Merchant has not responded to chargeback inquiry | Medium |
| Missing identity verification | Customer identity not confirmed (account takeover cases) | Critical |
| Missing police report | Law enforcement report requested but not received | Medium |

## Escalation Flags

| Flag | Description | Required Action |
|------|-------------|-----------------|
| **AML/SAR indicators** | Suspicious activity patterns requiring SAR filing consideration | Immediate AML team escalation |
| **Reg E deadline approaching** | 10-business-day provisional credit window approaching | Regulatory compliance escalation |
| **Card network deadline** | Network-specific chargeback window approaching | Chargeback team notification |
| **SLA breach** | Case exceeding 30-minute triage window | Supervisor notification |
| **Repeat fraud pattern** | Multiple fraud events on same account within 90 days | Elevated review |
| **Internal fraud indicators** | Patterns suggesting employee involvement | Special investigations referral |

## Urgency Levels

| Level | Criteria |
|-------|----------|
| **Immediate** | High-value transaction, account takeover indicators, AML/SAR flags, internal fraud indicators |
| **Standard** | Standard fraud or dispute within normal parameters and SLA timeline |
| **Low Priority** | Minor disputes, merchant errors, cases with complete evidence and no escalation flags |

## Regulatory Timelines

| Regulation | Timeline | Applicability |
|------------|----------|---------------|
| **Reg E — Provisional credit** | 10 business days from dispute receipt | Electronic fund transfers, debit card disputes |
| **Reg E — Investigation (standard)** | 45 calendar days | Standard electronic fund transfer disputes |
| **Reg E — Investigation (extended)** | 90 calendar days | New accounts, POS transactions, foreign transactions |
| **Visa chargeback** | 120 calendar days from transaction date | Visa card disputes |
| **Mastercard chargeback** | 120 calendar days from transaction date | Mastercard disputes |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Case Intake, Context Packet, Gap Detection — find alerts, customer correspondence, affidavit submissions |
| SearchM365 (files) | All skills — find case tracker, fraud taxonomy, dispute procedures, evidence checklists, routing matrix, templates |
| SearchM365 (connectors) | Context Packet — retrieve account and transaction data from core banking via Graph Connector |
| ReadFileContent | All skills — read tracker, procedures, taxonomy, checklists, templates, context packets |
| GetDriveChildren | Context Packet, Gap Detection, Scenario Classifier — browse case evidence folders |
| SearchPeople / GetUserDetails | All skills — resolve analysts, supervisors, branch contacts, customer service representatives |
| GetManagerDetails / GetDirectReportsDetails | Case Routing — fraud operations org structure for escalation paths |
| CreateDraftMessage | Case Routing, Case Comms — Outlook drafts for customer communications and AML escalation (never auto-send) |
| PostMessage | Case Routing, Case Comms — Teams notifications to analyst queues and coordination channels |
| ListCalendarView | Case Routing — check analyst availability for urgent cases |
| render_ui (Adaptive Card) | Case Intake, Scenario Classifier, Gap Detection, Case Routing — confirmations, classifications, risk reports |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Fraud/dispute case record | Excel tracker row | Case Intake |
| Fraud context packet | Word document | Context Packet |
| Classification report | Adaptive Card | Scenario Classifier |
| Risk and gap assessment | Adaptive Card | Gap and Risk Detection |
| Routing recommendation | Adaptive Card | Case Routing |
| Analyst assignment notifications | Teams direct messages | Case Routing |
| Customer dispute acknowledgment | Outlook draft | Case Comms |
| Affidavit or statement request | Outlook draft | Case Comms |
| Missing evidence follow-up | Outlook draft | Case Comms |
| Analyst case summary / handoff note | Outlook draft or Teams message | Case Comms |
| Supervisor escalation notice | Outlook draft | Case Comms |
| Branch coordination message | Outlook draft or Teams message | Case Comms |

## Federated Data Access

Banking fraud and dispute workflows depend on external systems (core banking, fraud platform, card processor, case management). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Core banking account profiles, transaction summaries, and prior case history indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Fraud taxonomy, dispute procedures, Reg E guidance, routing matrices, evidence checklists maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Fraud scores, card processor details, and data points not available through integration, captured via structured prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (taxonomy, procedures, routing matrix, checklists) and Tier 3 for fraud score context and card processor details. Introduce Graph Connectors for core banking data in Wave 2 after skill workflows are proven and data access is approved by the banking data owner.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── bnk-case-intake/SKILL.md
├── bnk-fraud-context-packet/SKILL.md
├── bnk-scenario-classifier/SKILL.md
├── bnk-gap-risk-detection/SKILL.md
├── bnk-case-routing/SKILL.md
└── bnk-case-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Fraud and dispute case tracker** — shared Excel workbook (Case ID, Alert ID, Customer Name, Account Number masked, Transaction Reference, Dispute Type, Fraud Indicator, Channel, Status, Created Date, Assigned To, SLA Deadline, Priority, Evidence Status, Classification, Routing, Disposition, Audit Log)
- **Fraud taxonomy** — scenario type definitions, pattern indicators, handling lane criteria, confidence thresholds
- **Dispute type definitions** — dispute categories, handling rules, evidence requirements per type
- **Evidence checklists** — required evidence items by dispute type and fraud scenario
- **Reg E handling guidance** — provisional credit timelines, investigation deadlines, compliance requirements
- **Card network chargeback rules** — Visa, Mastercard deadline and reason code reference
- **Routing matrix** — who handles what by scenario type, urgency, product type, and escalation level
- **Communication templates** — approved templates for customer acknowledgment, affidavit request, evidence follow-up, analyst handoff, escalation notice, branch coordination
- **Case evidence folder structure** — per-case folders in SharePoint for document uploads and evidence storage

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Case Intake | "new fraud alert", "dispute case for [customer]", "intake fraud report", "new dispute submission" |
| Context Packet | "build fraud context packet", "gather transaction history", "what do we know about this fraud case" |
| Scenario Classifier | "classify this fraud case", "what type of dispute is this", "triage classification for [case]" |
| Gap and Risk Detection | "check evidence gaps", "assess fraud case urgency", "Reg E deadline check", "SLA status" |
| Case Routing | "route this fraud case", "assign case to analyst", "who handles this dispute type", "escalate this case" |
| Case Comms | "draft customer message", "prepare affidavit request", "send analyst summary", "fraud case follow-up" |

## Implementation Roadmap

### Wave 1 — Foundation (Intake, Context, Classification, Risk)

- Create the shared Excel fraud and dispute case tracker in SharePoint
- Upload fraud taxonomy, dispute definitions, evidence checklists, Reg E guidance, and routing matrix
- Create per-case evidence folder structure in SharePoint
- Establish approved communication templates
- Build skills: `bnk-case-intake`, `bnk-fraud-context-packet`, `bnk-scenario-classifier`, `bnk-gap-risk-detection`
- Operate in AI assist mode — all outputs presented for analyst review
- Test with one fraud or dispute queue and one product set

### Wave 2 — Routing and Communications

- Build skills: `bnk-case-routing`, `bnk-case-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote scenario classifier and gap detection to write-back mode (updates tracker after analyst confirmation)
- Set up daily scheduled prompt to check for cases approaching SLA deadlines with incomplete status
- Set up weekly scheduled prompt to summarize case volume, classification distribution, and evidence gap trends

### Wave 3 — Optimization and Pattern Detection

- Introduce Graph Connectors for core banking and fraud platform data
- Add proactive monitoring: flag repeated merchant patterns, recurring intake deficiencies, regulatory deadline risk
- Add bounded multi-document case packet assembly for complex or repeat-pattern cases
- Measurement targets: 85%+ classification accuracy, 85%+ gap detection rate, below 5% incorrect routing rate, 75%+ customer draft acceptance rate, zero unauthorized account actions, zero missing audit fields

## Implementation Notes

- **Triage disposition is intentionally not automated** — final fraud or dispute disposition decisions carry compliance accountability that cannot be delegated to AI
- **No skill performs account actions** — this is a permanent architectural constraint; account blocking, card restriction, transaction reversal, and provisional credit authorization are always human-owned
- **Guardrails are regulatory requirements** — in banking, guardrails like "never disclose SAR status" are federal legal requirements (BSA/AML), not operational preferences
- **The 30-minute SLA** drives urgency throughout the workflow — every skill surfaces SLA status in its output
- **AML escalation is always separated** from standard fraud triage — different access controls, different routing, different communication channels
- **All customer communications use approved templates** — no freeform customer messaging is permitted
- **The Excel tracker serves dual duty** — operational state store and examination-grade audit record; every skill write includes actor, action, timestamp, and prior value
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a gap check on an existing case, or drafting a follow-up without re-running the full workflow)
