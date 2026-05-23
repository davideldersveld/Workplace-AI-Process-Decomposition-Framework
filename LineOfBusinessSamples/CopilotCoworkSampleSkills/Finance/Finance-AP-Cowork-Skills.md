# Finance AP Exception Handling — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **AP Invoice Exception Handling** workflow as a set of seven Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the AP exception resolution process — from invoice intake through status tracking — into AI-assisted capabilities within Microsoft 365.

The workflow supports invoice exceptions triggered by three-way match failures, missing POs, vendor mismatches, and other validation issues, guiding each through structured intake, context assembly, classification, risk assessment, approval routing, communication drafting, and status management with SOX-compliant controls, segregation of duties enforcement, duplicate payment prevention, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **fin-invoice-intake** | Normalizes invoice exception events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **fin-match-context** | Assembles PO, receipt, vendor, policy, and history into a context packet | AI act within policy | analysis | SearchSparkle |
| 3 | **fin-exception-classify** | Classifies exception type with confidence scoring | AI assist | analysis | Tag |
| 4 | **fin-control-path** | Assesses risk and determines the control path per approval matrix | AI draft + approve | analysis | Flag |
| 5 | **fin-exception-routing** | Routes to the correct action owner with segregation enforcement | AI draft + approve | communication | Mail |
| 6 | **fin-ap-comms** | Drafts outreach, approval packets, follow-ups, and escalation notices | AI draft + approve | communication | Mail |
| 7 | **fin-status-update** | Updates tracker status with transition validation and aging alerts | AI act within policy | analysis | Edit |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Invoice Intake   │  Normalize exception event → structured case record
│     (fin-invoice-    │  Validates amount, checks duplicates by Invoice ID
│      intake)         │  SLA: 2 business days to resolution
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Match Context    │  Assemble invoice, PO, receipt, vendor, policy,
│     (fin-match-      │  prior exception history, key contacts
│      context)        │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Exception        │  Compare invoice vs. PO/receipt/vendor data,
│     Classification   │  assign dominant type, confidence score,
│     (fin-exception-  │  flag multi-issue cases
│      classify)       │  Output: Adaptive Card classification report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Control Path     │  Apply approval matrix and thresholds,
│     Assessment       │  detect risk flags, determine control path
│     (fin-control-    │  (auto-resolve / standard / escalated / investigate)
│      path)           │  Output: Adaptive Card risk + control recommendation
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Exception        │  Resolve named owner per matrix and org hierarchy,
│     Routing          │  enforce segregation of duties,
│     (fin-exception-  │  send routing notifications
│      routing)        │  Output: Teams + Outlook draft + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. AP Comms         │  Draft supplier outreach, buyer follow-ups,
│     (fin-ap-comms)   │  approval summaries, escalation notices,
│                      │  resolution confirmations
│                      │  Output: Outlook drafts + Teams updates
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Status Update    │  Update tracker with validated status transitions,
│     (fin-status-     │  action taken, actor, timestamp, aging alerts
│      update)         │  Output: Updated Excel tracker
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  8. Confirm          │  Confirm invoice corrected, approved, rejected,
│     Disposition      │  or escalated with full audit trail.
│     (not automated)  │  Human-only step.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Invoice Intake | Structured field extraction, amount validation, duplicate checking — no AI judgment |
| **AI act within policy** | Match Context, Status Update | Retrieves approved context from defined sources or updates tracker per validated rules; does not interpret policy or make accounting judgments |
| **AI assist** | Exception Classify | Surfaces classification with confidence scores as recommendations; AP analyst reviews before applying |
| **AI draft + approve** | Control Path, Exception Routing, AP Comms | AI recommends control path, resolves approvers, or drafts communication; AP analyst reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final disposition confirmation, payment release authorization, and case closure are always human decisions |

## Governance Controls

### SOX Compliance
- No skill can release payments, modify vendor bank details, or bypass the approval matrix
- Every skill action is logged with actor, timestamp, and exception case linkage for audit traceability
- Evidence citation is mandatory — every assertion in an approval summary traces to a source document
- Approval authority follows the matrix — no skill can substitute or override approval thresholds
- The ERP remains the authoritative source of truth for invoice status and disposition

### Segregation of Duties
- The exception routing skill enforces that the assigned approver is not the same person who created the PO on the invoice
- If the natural approver per the matrix is the PO creator, the skill escalates to the next level
- Segregation check results are logged in the exception tracker for audit review

### Duplicate Payment Prevention
- The invoice intake skill checks for existing cases by Invoice ID before creating a new case
- The exception classify skill flags duplicate invoice candidates as critical severity
- The control path skill requires manual investigation for any duplicate payment risk

### Evidence Trail
- The case context packet (Word document) preserves the complete assembly of what was gathered, from which sources, and when
- The classification report documents the exception type, confidence score, and evidence supporting the classification
- Every policy citation includes the document reference and version date
- Missing context is flagged rather than silently omitted

### Approval Matrix Integrity
- Control path and routing are determined strictly by the approval matrix and threshold table
- Amount, exception type, business unit, risk flags, and vendor profile all influence the required approval level
- The matrix is read dynamically from SharePoint at runtime — policy changes take effect without skill modification

### Financial Data Protection
- Bank account details and sensitive vendor financial data are never included in Teams messages
- Sensitive financial content is confined to the Word context packet and direct Outlook communications
- Draft or unposted data is clearly labeled as preliminary when included in the context packet

## Exception Categories

| Category | Description |
|----------|-------------|
| Amount mismatch | Invoice amount differs from PO amount beyond tolerance |
| Missing goods receipt | Goods receipt not posted or partial |
| Missing or invalid PO | Invoice references no PO or an invalid PO number |
| Duplicate invoice candidate | Invoice matches existing invoice by amount, vendor, and date |
| Vendor master mismatch | Invoice vendor details differ from vendor master record |
| Incomplete supporting documentation | Required documents (contract, delivery note, service acceptance) missing |
| Tax or withholding discrepancy | Tax calculation or withholding amount does not match expected values |

## Risk Flags

| Risk Flag | Severity |
|-----------|----------|
| Above materiality threshold | High |
| Duplicate payment risk | Critical |
| Vendor master mismatch | High |
| Segregation of duties conflict | High |
| Low classification confidence | Medium |
| Multi-issue exception | Medium |
| Recurring vendor exception | Medium |
| Late payment risk | Medium |

## Control Paths

| Path | Criteria |
|------|----------|
| **Auto-resolve** | Below auto-resolve threshold, single clear type with high confidence, no risk flags |
| **Standard approval** | Below materiality threshold, clear type, no critical risk flags |
| **Escalated review** | Above materiality threshold, critical risk flags, or low confidence |
| **Manual investigation** | Duplicate payment risk, vendor mismatch, segregation conflict, or multi-issue with low confidence |

## Approval Levels

| Level | Typical Criteria |
|-------|-----------------|
| **AP Analyst** | Auto-resolve cases below the first threshold |
| **Cost Center Owner** | Standard exceptions requiring business confirmation |
| **AP Manager** | Exceptions above the first threshold or with medium risk flags |
| **Controller** | Exceptions above materiality threshold, critical risk flags, or SOX-sensitive conditions |
| **CFO / VP Finance** | Exceptions above the second threshold or with fraud indicators |

## Status Lifecycle

```
New → Classified → Assessed → Routed → Awaiting Approval → Approved → Resolved
                                    → Awaiting Information → Routed (re-routed)
                                    → Escalated → Awaiting Approval
                                                → Manual Investigation → Resolved
                    Awaiting Approval → Rejected → Resolved
                    Any → Rework
```

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Invoice Intake — find supplier email threads |
| SearchM365 (files) | All skills — find tracker, policies, approval matrix, taxonomy, templates, context packets |
| SearchM365 (connectors) | Invoice Intake, Match Context — ERP, invoice capture, vendor master via Graph Connectors |
| ReadFileContent | All skills — read tracker, policies, threshold tables, context packets, taxonomy |
| GetDriveChildren | Match Context, Status Update — verify uploaded evidence in case folders |
| GetMessage | Invoice Intake — read full supplier email content |
| SearchPeople / GetUserDetails | Intake, Match Context, Routing — resolve identities |
| GetMyDetails | Invoice Intake, Status Update — current user for audit trail |
| GetManagerDetails / GetDirectReportsDetails | Match Context, Exception Routing — reporting chain and segregation check |
| CreateDraftMessage | Exception Routing, AP Comms — approval requests and follow-ups (never auto-send) |
| PostMessage | Exception Routing, AP Comms — Teams notifications to action owners and coordinators |
| render_ui (Adaptive Card) | Exception Classify, Control Path, Invoice Intake — reports and confirmations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Exception case record | Excel tracker row | Invoice Intake |
| Case context packet | Word document | Match Context |
| Classification report | Adaptive Card | Exception Classify |
| Control path recommendation | Adaptive Card | Control Path |
| Action owner notification | Teams direct message | Exception Routing |
| Formal routing request | Outlook draft | Exception Routing |
| Supplier outreach | Outlook draft | AP Comms |
| Buyer follow-up | Outlook draft | AP Comms |
| Approval summary | Outlook draft | AP Comms |
| Escalation notice | Outlook draft | AP Comms |
| Resolution confirmation | Outlook draft + Teams message | AP Comms |
| Status update | Excel tracker row | Status Update |

## Federated Data Access

AP exception handling depends on external systems (ERP, invoice capture platform, vendor master, PO system). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | ERP invoice records, PO data, vendor master profiles, goods receipt status indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | AP policies, approval matrices, exception taxonomy, threshold tables maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Data points not available through integration, captured via structured prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (policies, matrices, taxonomy) and Tier 3 for ERP data requiring manual lookup. Introduce ERP Graph Connectors in Wave 2.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── fin-invoice-intake/SKILL.md
├── fin-match-context/SKILL.md
├── fin-exception-classify/SKILL.md
├── fin-control-path/SKILL.md
├── fin-exception-routing/SKILL.md
├── fin-ap-comms/SKILL.md
└── fin-status-update/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **AP exception tracker** — shared Excel workbook (Exception Case ID, Invoice ID, Vendor ID, Vendor Name, PO ID, Amount, Currency, Exception Source, Business Unit, Status, Created Date, Priority, Classification, Confidence Score, Assigned To, Control Path, Routing Date, Resolution Date, Disposition)
- **AP policies** — invoice exception handling standards, three-way match tolerance rules
- **Approval matrix and threshold table** — defines which approval level is required based on amount, exception type, and business unit
- **Exception taxonomy** — defines classification categories and rules for exception type assignment
- **Communication templates** — approved templates for supplier outreach, buyer follow-ups, approval summaries, escalation notices, and resolution confirmations
- **Exception case folder structure** — per-case folders in SharePoint for context packets and supporting documents

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Invoice Intake | "new invoice exception", "AP exception for invoice 12345", "invoice failed matching", "set up exception case" |
| Match Context | "build context packet for invoice", "gather matching data for exception", "assemble AP case packet" |
| Exception Classify | "classify this exception", "what type of exception is this", "triage invoice exception" |
| Control Path | "assess risk for this exception", "what's the control path", "what approvals are needed" |
| Exception Routing | "route this exception", "assign to the right owner", "who handles this exception" |
| AP Comms | "draft supplier email", "prepare approval packet", "write follow-up for missing receipt", "escalation notice" |
| Status Update | "update exception status", "mark case as resolved", "log resolution", "update AP tracker" |

## Implementation Roadmap

### Wave 1 — Foundation and Triage
- Create the shared Excel AP exception tracker in SharePoint
- Upload AP policies, approval matrices, exception taxonomy, and threshold tables
- Create per-case folder structure in SharePoint for evidence documents
- Populate PO, receipt, and vendor reference data in SharePoint (Tier 2 bridge) via Power Automate
- Build skills: `fin-invoice-intake`, `fin-match-context`, `fin-exception-classify`
- Operate in AI assist mode — all outputs presented for manual review
- Test with one AP queue or business unit, 10-20 real exception cases

### Wave 2 — Communication and Routing
- Build skills: `fin-control-path`, `fin-exception-routing`, `fin-ap-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote classification to write-back mode (updates tracker after analyst confirmation)
- Set up daily scheduled prompt to check for exception cases approaching the 2-business-day SLA
- Enforce segregation of duties in routing guardrails

### Wave 3 — Status Management and Optimization
- Build skill: `fin-status-update`
- Introduce Graph Connectors for ERP data if available
- Add proactive monitoring: flag aging exceptions, recurring vendor issues, exception patterns
- Add approval summary generation for controller-level review
- Measurement targets: 90%+ classification accuracy, 75%+ draft acceptance rate, below 5% incorrect routing rate, zero unauthorized write actions, zero missing audit fields, zero duplicate payment incidents

## Implementation Notes

- **Confirm disposition is intentionally not automated** — final disposition confirmation, payment release, and case closure are always human decisions
- **No skill can release payments or modify vendor bank details** — this is a permanent architectural constraint reflecting SOX requirements
- **Excel-centric artifact pattern** — the exception tracker workbook serves as process state store, audit log, and operational dashboard; the Word context packet provides the evidence trail
- **Segregation of duties is enforced in code** — the exception routing skill refuses to route if the check fails
- **All external communications are Outlook drafts** — nothing is sent without explicit AP analyst confirmation
- **The ERP is the authoritative state source** — the Excel tracker is the coordination layer; Power Automate pushes final disposition back to ERP
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., classifying an existing case, drafting a follow-up without re-running the full workflow)
