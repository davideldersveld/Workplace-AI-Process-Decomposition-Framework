# Finance Controllership Journal Entry Review — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Manual Journal Entry Request and Approval** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the controllership journal review process — from intake through approval routing — into AI-assisted capabilities within Microsoft 365.

The workflow supports manual journal entry requests submitted during close cycles, guiding each through structured intake, evidence assembly, gap and risk detection, approval routing, and communication drafting with SOX-compliant controls, segregation of duties enforcement, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **fc-journal-intake** | Normalizes journal entry requests into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **fc-journal-context** | Assembles ledger context, policies, support documents, and approval thresholds | AI act within policy | analysis | SearchSparkle |
| 3 | **fc-gap-risk-detection** | Detects missing evidence, control risks, and policy-sensitive conditions | AI assist | analysis | Flag |
| 4 | **fc-approval-routing** | Routes to the correct approver with segregation of duties enforcement | AI draft + approve | communication | Mail |
| 5 | **fc-journal-comms** | Drafts reviewer summaries, evidence follow-ups, and escalation notices | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Journal Intake   │  Normalize request → structured case record
│     (fc-journal-     │  Validates debit-credit balance, checks duplicates
│      intake)         │  SLA: 1 business day to review-ready state
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Journal Context  │  Assemble policies, ledger context, support inventory,
│     (fc-journal-     │  approval thresholds, historical comparisons
│      context)        │  Output: Word evidence packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Gap and Risk     │  Compare evidence packet against control checklist,
│     Detection        │  detect missing documents, flag control risks,
│     (fc-gap-risk-    │  assess audit readiness
│      detection)      │  Output: Adaptive Card risk report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Approval         │  Determine required approval level, resolve named
│     Routing          │  approver, enforce segregation of duties,
│     (fc-approval-    │  send routing notifications
│      routing)        │  Output: Teams + Outlook draft + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Journal Comms    │  Draft reviewer summaries, missing evidence requests,
│     (fc-journal-     │  close coordination updates, escalation notices,
│      comms)          │  rework notifications
│                      │  Output: Outlook drafts + Teams updates
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Review-Readiness │  Confirm entry is ready for approval, return for
│     Disposition      │  rework, or escalate for additional review.
│     (not automated)  │  Human-only step.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Journal Intake | Structured field extraction, balance validation, duplicate checking — no AI judgment |
| **AI act within policy** | Journal Context | Retrieves approved context from defined sources; does not interpret policy or make accounting judgments |
| **AI assist** | Gap and Risk Detection | Surfaces missing evidence and control risks as recommendations with confidence levels; accountant reviews before applying |
| **AI draft + approve** | Approval Routing, Journal Comms | AI recommends approval path or drafts communication; accountant reviews and confirms before any action |
| **Human only** | Review-Readiness Disposition | Final approval, posting authorization, and review-readiness confirmation are always human decisions |

## Governance Controls

### SOX Compliance
- No skill can post a journal entry to the ERP — all skills produce reviewable drafts and recommendations
- Every skill action is logged with actor, timestamp, and journal case linkage for audit traceability
- Evidence citation is mandatory — every assertion in a reviewer summary traces to a source document
- Approval authority follows the matrix — no skill can bypass or substitute approval thresholds

### Segregation of Duties
- The approval routing skill enforces that the assigned approver is not the same person who requested or prepared the journal entry
- If the natural approver per the matrix is the requestor's direct manager who also prepared the entry, the skill escalates to the next level
- Segregation check results are logged in the journal tracker for audit review

### Evidence Trail
- The journal evidence packet (Word document) preserves the complete assembly of what was gathered, from which sources, and when
- The gap and risk report documents what was checked, what was found, and what the confidence level was for each finding
- Every policy citation includes the document reference and version date
- Missing context is flagged rather than silently omitted

### Approval Matrix Integrity
- Approval routing is determined strictly by the approval matrix and threshold table — not by convenience or availability
- Amount thresholds, entry types, entity requirements, and risk flags all influence the required approval level
- The matrix is read dynamically from SharePoint at runtime — policy changes take effect without skill modification

### Financial Data Protection
- Account balances and entry amounts are never included in Teams channel posts — only in the Word packet and direct Outlook communications
- Draft or unposted ledger data is clearly labeled as preliminary when included in the evidence packet
- Sensitive financial details are confined to the evidence packet and approval communications

## Gap and Risk Detection Categories

### Missing Evidence

| Gap Type | Description |
|----------|-------------|
| Missing supporting schedule | Required schedule not in case folder |
| Missing reconciliation | Reconciliation required by policy but not uploaded |
| Missing rationale memo | Entry has no written rationale or rationale is too vague |
| Missing prior-period reference | Reversal or reclassification without reference to original |

### Control Risks

| Risk Type | Description |
|-----------|-------------|
| Above materiality threshold | Amount exceeds threshold without enhanced approval documentation |
| Unusual account combination | Debit-credit pair not seen in prior periods for this entity |
| Duplicate entry candidate | Similar amount, accounts, and period to an existing case |
| Reversal without original | Reversal entry with no linked original entry reference |
| Cross-entity entry | Entry spans multiple legal entities requiring additional sign-off |
| Late close adjustment | Entry submitted after the standard close cutoff date |
| Segregation concern | Requestor is likely to be the approver |

## Approval Levels

| Level | Typical Criteria |
|-------|-----------------|
| **Accounting Manager** | Standard entries below the first materiality threshold |
| **Controller** | Entries above materiality threshold, intercompany, late adjustments, high-risk flagged |
| **CFO / VP Finance** | Entries above the second threshold, equity adjustments, extraordinary items |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Journal Intake — find inbound request emails |
| SearchM365 (files) | All skills — find tracker, policies, approval matrix, checklists, templates, evidence packets |
| SearchM365 (connectors) | Journal Intake, Journal Context — ERP ledger data via Graph Connector |
| ReadFileContent | All skills — read tracker, policies, threshold tables, evidence packets, checklists |
| GetDriveChildren | Journal Context, Gap Detection — verify uploaded evidence in case folders |
| GetMessage | Journal Intake — read full request email content |
| SearchPeople / GetUserDetails | Intake, Context, Routing — resolve identities |
| GetMyDetails | Journal Intake — current user for audit trail |
| GetManagerDetails / GetDirectReportsDetails | Journal Context, Approval Routing — reporting chain and segregation check |
| CreateDraftMessage | Approval Routing, Journal Comms — approval requests and follow-ups (never auto-send) |
| PostMessage | Approval Routing, Journal Comms — Teams notifications to reviewers and close coordinators |
| render_ui (Adaptive Card) | Gap Detection, Journal Intake — risk reports and case confirmations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Journal case record | Excel tracker row | Journal Intake |
| Journal evidence packet | Word document | Journal Context |
| Gap and risk report | Adaptive Card | Gap and Risk Detection |
| Approval path recommendation | Adaptive Card | Approval Routing |
| Reviewer notification | Teams direct message | Approval Routing |
| Formal approval request | Outlook draft | Approval Routing |
| Reviewer summary | Outlook draft | Journal Comms |
| Missing evidence request | Outlook draft | Journal Comms |
| Close coordination update | Teams message / Outlook draft | Journal Comms |
| Escalation notice | Outlook draft | Journal Comms |
| Rework notification | Outlook draft | Journal Comms |

## Federated Data Access

Controllership depends on external systems (ERP, close management system, document repository). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | ERP ledger balances, account metadata, journal posting status indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Close management data, approval matrices, threshold tables, control checklists maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Ledger data points not available through integration, captured via structured prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (policies, matrices, checklists) and Tier 3 for ledger data requiring manual lookup. Introduce ERP Graph Connectors in Wave 2.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── fc-journal-intake/SKILL.md
├── fc-journal-context/SKILL.md
├── fc-gap-risk-detection/SKILL.md
├── fc-approval-routing/SKILL.md
└── fc-journal-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Journal tracker** — shared Excel workbook (Journal Case ID, Description, Requestor, Entity, Business Unit, Debit Account, Credit Account, Amount, Currency, Close Period, Status, Support Status, Approval Path, Assigned Reviewer, Disposition)
- **Accounting policies** — journal entry standards, entry-type-specific policies, materiality thresholds
- **Approval matrix and threshold table** — defines which approval level is required based on amount, entry type, and entity
- **Control checklist** — defines required supporting evidence by entry type and materiality level
- **Communication templates** — approved templates for reviewer summaries, evidence requests, escalation notices, and rework notifications
- **Journal case folder structure** — per-case folders in SharePoint for supporting schedules, reconciliations, and memos

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Journal Intake | "new journal entry request", "manual JE for Q2 accrual", "close adjustment request", "journal case for intercompany" |
| Journal Context | "build journal packet for JE-2026-00087", "gather context for this journal", "what do we need for this entry" |
| Gap and Risk Detection | "check journal for gaps", "audit readiness for JE-2026-00087", "what's missing for this entry", "control review" |
| Approval Routing | "route journal for approval", "who approves this entry", "approval path for JE-2026-00087" |
| Journal Comms | "draft journal summary", "write follow-up for missing support", "escalation notice for JE-2026-00087", "rework notification" |

## Implementation Roadmap

### Wave 1 — Foundation and Evidence Assembly
- Create the shared Excel journal tracker in SharePoint
- Upload accounting policies, journal standards, approval matrices, threshold tables, and control checklists
- Create per-case folder structure in SharePoint for evidence documents
- Populate ledger reference data in SharePoint (Tier 2 bridge) if feasible
- Build skills: `fc-journal-intake`, `fc-journal-context`, `fc-gap-risk-detection`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 10-15 real journal entry cases from one close period

### Wave 2 — Routing and Communication
- Build skills: `fc-approval-routing`, `fc-journal-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote gap detection to write-back mode (updates tracker after accountant confirmation)
- Set up close-period scheduled prompt to check for cases approaching close deadline with incomplete evidence or missing approvals
- Enforce segregation of duties in routing guardrails

### Wave 3 — Optimization and Extended Coverage
- Introduce Graph Connectors for ERP ledger data if available
- Add proactive detection of likely duplicate entries or reversal mismatches
- Add close checklist summary generation (aggregate view of all cases by status for close coordinator)
- Add policy-cited reviewer packet support with enhanced evidence formatting
- Measurement targets: 85%+ missing-support detection rate, below 5% incorrect approval-path rate, above 75% draft summary acceptance rate, below 15% rework rate, zero segregation violations, zero missing audit fields

## Implementation Notes

- **Review-readiness disposition is intentionally not automated** — final approval, posting authorization, and review-readiness confirmation are always human decisions
- **No skill can post a journal entry** — this is a permanent architectural constraint reflecting SOX requirements, not a pilot limitation
- **Dual-primary artifact pattern** — Excel tracker for status trail, Word evidence packet for audit trail; both are mandatory outputs of the preparation workflow
- **Segregation of duties is enforced in code** — the approval routing skill refuses to route if the check fails; this is the strictest governance control in the suite
- **All communications are Outlook drafts** — nothing is sent without explicit accountant confirmation
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a gap check on an existing case, or drafting a follow-up for missing evidence without re-running the full workflow)
