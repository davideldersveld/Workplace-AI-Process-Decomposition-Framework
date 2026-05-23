# Compliance & Risk Case Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Compliance & Risk Case Triage** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the compliance case triage process — from intake through disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports policy exception requests, hotline reports, control issues, and suspected violations, guiding them through structured triage with appropriate governance controls at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **compliance-case-intake** | Normalizes incoming cases into structured records | Deterministic automation | analysis | TaskListLtr |
| 2 | **compliance-context-packet** | Assembles policy, control, and entity context | AI act within policy | analysis | SearchSparkle |
| 3 | **compliance-risk-detection** | Identifies risk indicators and evidence gaps | AI assist | analysis | Flag |
| 4 | **compliance-review-routing** | Recommends severity, reviewers, and escalation path | AI draft + approve | analysis | Edit |
| 5 | **compliance-case-comms** | Drafts audience-appropriate case communications | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Case Intake      │  Normalize incoming case → structured record in tracker
│     (compliance-     │  Supports: policy exceptions, hotline reports, control
│      case-intake)    │  issues, suspected violations, anonymous reports
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble applicable policies, control ownership,
│     (compliance-     │  prior exceptions, entity context, evidence checklist
│      context-packet) │  Output: Word context packet + Excel evidence checklist
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Risk Detection   │  Compare evidence against requirements, detect risk
│     (compliance-     │  indicator patterns, flag regulatory-reportable items
│      risk-detection) │  Output: Adaptive Card risk indicator report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Review Routing   │  Apply severity matrix, assign reviewers, determine
│     (compliance-     │  escalation path, set SLA deadlines
│      review-routing) │  Output: Routing recommendation + notifications
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Case Comms       │  Draft case summaries, evidence requests, escalation
│     (compliance-     │  notices, status updates, audit referrals
│      case-comms)     │  Output: Outlook drafts for review before sending
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Human            │  Final triage disposition — approve, escalate,
│     Disposition      │  close, or refer. Human-only step.
│     (not automated)  │
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Case Intake | Structured field extraction and duplicate checking — no AI judgment |
| **AI act within policy** | Context Packet | Retrieves and assembles approved context; does not interpret policy |
| **AI assist** | Risk Detection | Surfaces indicators for analyst review; read-only, no case modifications |
| **AI draft + approve** | Review Routing, Case Comms | AI recommends or drafts; human reviews and confirms before any action |
| **Human only** | Final Disposition | Triage decision is always made by a compliance analyst |

## Governance Controls

### Audit Trail Integrity
- Every case action is logged with timestamp, actor, and rationale
- Evidence chain of custody is preserved across all skills
- Routing decisions include full rationale for compliance records

### Anonymous Reporting Protection
- Anonymous reporters are recorded as "Anonymous" — no attempt to identify
- Context packets and communications never reveal anonymous reporter identity
- Status updates cannot be sent to anonymous reporters

### Segregation of Duties
- Review routing flags conflicts where the triaging analyst is also the control owner
- Reviewer assignments must match the approved reviewer matrix in SharePoint

### Regulatory Sensitivity
- Regulatory-reportable indicators (anti-corruption, sanctions, data breach, financial misconduct) are elevated with separate visibility
- Cases with regulatory flags are routed to compliance leadership regardless of other severity factors
- Regulatory reporting flag communications are marked URGENT

### Data Classification
- All internal compliance communications include "Confidential — Compliance Review Material" marking
- Teams messages contain case IDs and folder links only — no case substance
- Escalation notices include severity rationale without premature conclusions

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find tracker, policies, templates, evidence |
| SearchM365 (email) | Case Intake — find intake emails |
| SearchM365 (connectors) | Context Packet — GRC platform case history |
| ReadFileContent | All skills — read case materials, policies, matrices |
| GetDriveChildren | Context Packet, Risk Detection — browse evidence folders |
| SearchPeople / GetUserDetails | Intake, Context Packet, Routing, Comms — resolve identities |
| GetMyDetails | Case Intake — assigned analyst |
| GetManagerDetails / GetDirectReportsDetails | Context Packet, Routing — escalation chain |
| GetMessage | Case Intake — read intake emails |
| CreateDraftMessage | Comms, Routing — all Outlook drafts (never auto-send) |
| PostMessage | Comms, Routing — Teams notifications (after confirmation) |
| CreateEvent | Routing — SLA deadline calendar events |
| render_ui (Adaptive Card) | Risk Detection, Routing, Intake — structured data display |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Case record | Excel tracker row | Case Intake |
| Context packet | Word document | Context Packet |
| Evidence checklist | Excel workbook | Context Packet |
| Risk indicator report | Adaptive Card | Risk Detection |
| Routing recommendation | Adaptive Card | Review Routing |
| Case summary | Outlook draft | Case Comms |
| Evidence request | Outlook draft | Case Comms |
| Escalation notice | Outlook draft | Case Comms |
| Status update | Outlook draft | Case Comms |
| Audit referral | Outlook draft | Case Comms |
| Regulatory reporting flag | Outlook draft (URGENT) | Case Comms |
| SLA deadline | Calendar event | Review Routing |

## Severity Matrix

| Severity | SLA Deadline | Required Reviewers |
|----------|-------------|-------------------|
| **Low** | 5 business days | Assigned compliance analyst |
| **Medium** | 2 business days | Analyst + control owner |
| **High** | 1 business day | Analyst + control owner + compliance manager + legal reviewer |
| **Critical** | 4 hours | All above + internal audit liaison + compliance leadership |

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── compliance-case-intake/SKILL.md
├── compliance-context-packet/SKILL.md
├── compliance-risk-detection/SKILL.md
├── compliance-review-routing/SKILL.md
└── compliance-case-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Compliance case tracker** — shared Excel workbook for case records
- **Policy document library** — SharePoint document library with compliance policies
- **Compliance severity matrix** — defines severity classification criteria
- **Compliance routing rules** — defines reviewer assignments by case type and severity
- **Compliance reviewer assignments** — maps reviewers to case types and severity levels
- **Communication templates** — optional templates for case communications
- **Evidence checklist templates** — per-case-type evidence requirement lists

### Trigger Phrases

Each skill responds to natural language triggers. Examples:

| Skill | Example Triggers |
|-------|-----------------|
| Case Intake | "new compliance case", "policy exception request from Sarah", "hotline report received", "log compliance intake" |
| Context Packet | "build compliance context for CR-2026-042", "what policies apply to gift acceptance", "assemble case packet" |
| Risk Detection | "check risk indicators for CR-2026-042", "what evidence is missing", "compliance gap check" |
| Review Routing | "route this case for review", "assign compliance reviewers", "determine severity and routing" |
| Case Comms | "draft compliance case summary", "send evidence request for CR-2026-042", "draft escalation notice" |

## Implementation Notes

- **Human disposition is intentionally not automated** — the final triage decision (approve, escalate, close, refer) is always a human responsibility
- **All communications are created as Outlook drafts** — nothing is sent without explicit user confirmation
- **Risk detection operates in read-only mode** — it surfaces indicators but never modifies case status or severity
- **Routing recommendations are presented for approval** — reviewers and notifications are only sent after the analyst confirms
- **The skills are designed to be used sequentially** but can also be invoked independently for specific tasks (e.g., drafting a follow-up evidence request for an existing case)
