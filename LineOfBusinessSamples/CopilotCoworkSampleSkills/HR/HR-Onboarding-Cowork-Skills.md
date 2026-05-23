# HR Onboarding Readiness — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Employee Onboarding Readiness** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the HR onboarding coordination process — from new-hire intake through readiness review — into AI-assisted capabilities within Microsoft 365.

The workflow supports new hire onboarding events, guiding each through structured intake, readiness context assembly, gap and risk detection, task routing, communication drafting, and readiness summary preparation with privacy controls, PII protection, policy compliance, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **hr-onboarding-intake** | Normalizes new-hire events into structured onboarding case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **hr-readiness-packet** | Assembles employee profile, policies, checklists, and contacts into a context packet | AI act within policy | analysis | SearchSparkle |
| 3 | **hr-gap-detection** | Detects missing documents, readiness blockers, and policy-sensitive conditions | AI assist | analysis | Flag |
| 4 | **hr-task-routing** | Routes onboarding tasks to owners per the responsibility matrix | AI draft + approve | communication | Mail |
| 5 | **hr-onboarding-comms** | Drafts welcome emails, missing document reminders, manager updates, and escalations | AI draft + approve | communication | Mail |
| 6 | **hr-readiness-summary** | Prepares consolidated readiness summary for review and sign-off | AI draft + approve | writing | Document |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Onboarding       │  Normalize new-hire event → structured case record
│     Intake           │  Validates required fields, checks duplicates
│     (hr-onboarding-  │  SLA: 2 business days to readiness review
│      intake)         │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Readiness        │  Assemble employee profile, role/location details,
│     Packet           │  policies, checklists, IT/facilities requirements,
│     (hr-readiness-   │  key contacts
│      packet)         │  Output: Word readiness packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Gap Detection    │  Compare readiness packet against checklist,
│     (hr-gap-         │  detect missing documents, readiness blockers,
│      detection)      │  policy-sensitive conditions, timeline risk
│                      │  Output: Adaptive Card gap report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Task Routing     │  Assign owners per responsibility matrix,
│     (hr-task-        │  resolve via org hierarchy, set deadlines
│      routing)        │  relative to start date
│                      │  Output: Teams notifications + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Onboarding       │  Draft welcome emails, missing document reminders,
│     Comms            │  manager updates, IT provisioning requests,
│     (hr-onboarding-  │  HRBP notifications, escalation notices
│      comms)          │  Output: Outlook drafts + Teams updates
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Readiness        │  Consolidate all artifacts into a reviewer-ready
│     Summary          │  summary — Adaptive Card, Word, or PowerPoint
│     (hr-readiness-   │  Output: Summary in requested format
│      summary)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm          │  Confirm case is ready, escalate, or return
│     Readiness        │  for rework. Human-only step.
│     (not automated)  │
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Onboarding Intake | Structured field extraction, date validation, duplicate checking — no AI judgment |
| **AI act within policy** | Readiness Packet | Retrieves approved context from defined sources; does not interpret policy or make HR decisions |
| **AI assist** | Gap Detection | Surfaces missing documents and readiness risks as recommendations with severity levels; HR specialist reviews before applying |
| **AI draft + approve** | Task Routing, Onboarding Comms, Readiness Summary | AI recommends routing, drafts communications, or prepares summaries; HR specialist reviews and confirms before any action |
| **Human only** | Confirm Readiness | Final readiness confirmation, exception approvals, and start-date authorization are always human decisions |

## Governance Controls

### Employee Privacy Protection
- No skill includes SSN, date of birth, salary, bank details, or background check results in any output
- Medical, disability, and accommodation details are never surfaced in gap reports or summaries — referenced only as "accommodation coordination pending" if on the checklist
- Sensitive HR content (work authorization, background check status) is confined to Outlook communications — never posted in Teams
- Information in routing messages and manager updates is limited to what the recipient needs for their task

### Policy Compliance
- Every policy extract in the readiness packet includes document reference and version date
- Checklists are read dynamically from SharePoint at runtime — policy changes take effect without skill modification
- Location-specific and employment-type-specific requirements are applied automatically based on case data
- Work authorization gaps are flagged as critical severity — these are legal compliance requirements

### Audit Trail
- Every skill action is logged with actor, timestamp, and case linkage
- The readiness packet preserves what was gathered, from which sources, and when
- The gap report documents what was checked, what was found, and the severity assessment
- Communication history is recorded in the tracker for audit review

### Ownership and Accountability
- Task routing follows the responsibility matrix — no skill bypasses or substitutes role assignments
- Escalation paths are defined in the responsibility matrix, not invented by the skill
- Final readiness confirmation remains a human decision (HR-ONB-007)
- All communications are Outlook drafts — nothing is sent without HR specialist confirmation

## Missing Document Categories

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing identification documents | Government-issued ID not uploaded | High |
| Missing tax forms | W-4 or state withholding not submitted | High |
| Missing policy acknowledgments | Handbook, code of conduct sign-offs incomplete | Medium |
| Missing work authorization | I-9 documentation incomplete or expired | Critical |
| Missing IT provisioning request | Equipment/account setup not submitted | High |
| Missing manager confirmation | Start date or team details unconfirmed | Medium |

## Readiness Blockers

| Blocker Type | Description | Severity |
|-------------|-------------|----------|
| IT provisioning not started | No setup request logged; start date approaching | High |
| Background check pending | Initiated but not cleared | High |
| Facilities not confirmed | Badge, workspace, or access not arranged | Medium |
| Benefits enrollment window | Enrollment deadline approaching | Medium |
| Offer letter unsigned | Sent but not returned signed | High |

## Policy-Sensitive Conditions

| Condition | Description | Severity |
|-----------|-------------|----------|
| Work authorization gap | I-9 incomplete, expired, or flagged | Critical |
| Privileged access request | Role requires elevated system access | High |
| International hire | Located outside US or visa sponsorship required | High |
| Rehire | Previously employed; prior records may need review | Medium |
| Contractor conversion | Converting from contractor to employee | Medium |

## Readiness Dispositions

| Disposition | Criteria |
|------------|----------|
| **Ready for Review** | All checklist items satisfied, no high-severity gaps or blockers |
| **Needs Documents** | Missing items that can be resolved by uploading documents or completing forms |
| **Needs Action** | Blockers requiring action from another team (IT, facilities, hiring manager) |
| **Needs Escalation** | Policy-sensitive conditions requiring HR operations manager review |
| **Critical** | Work authorization or legal compliance issues that must be resolved before start date |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Onboarding Intake, Readiness Packet, Readiness Summary — find offer letters, request emails, correspondence |
| SearchM365 (files) | All skills — find tracker, policies, checklists, location guides, templates, readiness packets |
| SearchM365 (connectors) | Onboarding Intake, Readiness Packet — HRIS data via Graph Connector |
| ReadFileContent | All skills — read tracker, policies, checklists, readiness packets |
| GetDriveChildren | Readiness Packet, Gap Detection, Readiness Summary — verify uploaded documents in onboarding folders |
| GetMessage | Onboarding Intake — read full request email content |
| SearchPeople / GetUserDetails | All skills — resolve employee, manager, HRBP, IT coordinator identities |
| GetManagerDetails / GetDirectReportsDetails | Readiness Packet, Task Routing — reporting chain and escalation paths |
| CreateDraftMessage | Onboarding Comms — all outreach and follow-up drafts (never auto-send) |
| PostMessage | Task Routing, Onboarding Comms — Teams notifications to task owners |
| CreateEvent | Task Routing — calendar deadline reminders for critical items |
| render_ui (Adaptive Card) | Onboarding Intake, Gap Detection, Readiness Summary — confirmations, reports, and scorecards |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Onboarding case record | Excel tracker row | Onboarding Intake |
| Readiness packet | Word document | Readiness Packet |
| Gap and risk report | Adaptive Card | Gap Detection |
| Task assignment notifications | Teams direct messages | Task Routing |
| Deadline reminders | Calendar events | Task Routing |
| Welcome email | Outlook draft | Onboarding Comms |
| Missing document reminder | Outlook draft | Onboarding Comms |
| Manager readiness update | Outlook draft / Teams message | Onboarding Comms |
| IT provisioning request | Outlook draft | Onboarding Comms |
| HRBP notification | Outlook draft | Onboarding Comms |
| Escalation notice | Outlook draft | Onboarding Comms |
| Readiness scorecard | Adaptive Card | Readiness Summary |
| Readiness review document | Word document | Readiness Summary |
| Manager review presentation | PowerPoint deck | Readiness Summary |

## Federated Data Access

HR onboarding depends on external systems (HRIS, ATS, ticketing system, identity workflow). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | HRIS employee records, ATS requisition data indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Onboarding policies, checklists, location guides, responsibility matrices maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Data points not available through integration, captured via structured prompts |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (policies, checklists, responsibility matrices) and Tier 3 for HRIS data requiring manual lookup. Introduce HRIS Graph Connectors in Wave 2.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── hr-onboarding-intake/SKILL.md
├── hr-readiness-packet/SKILL.md
├── hr-gap-detection/SKILL.md
├── hr-task-routing/SKILL.md
├── hr-onboarding-comms/SKILL.md
└── hr-readiness-summary/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Onboarding tracker** — shared Excel workbook (Case ID, Employee Name, Start Date, Role, Location, Department, Manager, Status, Created Date, Assigned To, Checklist Completion %)
- **Onboarding checklist** — standard checklist of required documents, forms, provisioning items, and policy acknowledgments
- **Onboarding policies** — company onboarding standards, location-specific requirements, employment-type requirements
- **Responsibility matrix** — defines who owns which onboarding tasks by category (IT, facilities, HR, manager)
- **Communication templates** — approved templates for welcome emails, missing document reminders, IT requests, manager updates, and escalation notices
- **Employee onboarding folder structure** — per-employee folders in SharePoint for document uploads and evidence

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Onboarding Intake | "new hire starting", "onboarding case for [name]", "set up onboarding for [employee]" |
| Readiness Packet | "build readiness packet", "assemble onboarding context for [name]", "what do we need for onboarding" |
| Gap Detection | "check onboarding gaps", "what's missing for [name]", "onboarding readiness check" |
| Task Routing | "assign onboarding tasks", "route onboarding work", "who handles [task] for this hire" |
| Onboarding Comms | "draft onboarding email", "remind manager about onboarding", "send welcome message to new hire" |
| Readiness Summary | "summarize onboarding status", "readiness review for [name]", "onboarding deck for manager review" |

## Implementation Roadmap

### Wave 1 — Foundation and Gap Detection
- Create the shared Excel onboarding tracker in SharePoint
- Upload onboarding policies, checklists, and location-specific guides
- Create per-employee folder structure in SharePoint for document uploads
- Build skills: `hr-onboarding-intake`, `hr-readiness-packet`, `hr-gap-detection`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 5-10 real onboarding cases from one region or business unit

### Wave 2 — Communication and Routing
- Build skills: `hr-onboarding-comms`, `hr-task-routing`
- Promote intake to write mode (creates case records after confirmation)
- Promote gap detection to write-back mode (updates tracker after specialist confirmation)
- Set up daily scheduled prompt to check for onboarding cases with approaching start dates and incomplete status
- Test with full onboarding cycle from intake through task completion

### Wave 3 — Review and Optimization
- Build skill: `hr-readiness-summary`
- Introduce Graph Connectors for HRIS data if available
- Add proactive monitoring: flag overdue cases, highlight recurring blockers, suggest process improvements
- Measurement targets: 85%+ gap detection rate, 70%+ draft acceptance rate, below 10% overdue case rate, below 15% reroute rate, zero privacy incidents

## Implementation Notes

- **Readiness confirmation is intentionally not automated** — final readiness decisions, exception approvals, and start-date authorization are always human actions
- **No skill makes employment decisions** — this is a permanent architectural constraint; eligibility, benefits, and offer decisions are always human-owned
- **PII protection is enforced at every skill boundary** — SSN, salary, medical data, and background check details are never included in outputs
- **Work authorization is the strictest governance control** — I-9 gaps are flagged as critical and require HR operations manager review
- **Multi-format readiness summary** — Adaptive Card for quick review, Word for formal sign-off, PowerPoint for manager-facing presentations; the HR specialist chooses the format
- **All external and formal communications are Outlook drafts** — nothing is sent without explicit HR specialist confirmation
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a gap check on an existing case, or drafting a follow-up without re-running the full workflow)
