# Internal Audit Evidence Prep — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Audit Request Intake and Evidence Collection Prep** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the internal audit evidence preparation process — from audit request intake through evidence-ready packet review — into AI-assisted capabilities within Microsoft 365.

The workflow supports audit engagement events, guiding each through structured intake, evidence context assembly, gap and scope analysis, evidence request routing, and communication drafting with audit independence, workpaper integrity, evidence chain of custody, IIA Standards compliance, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **audit-request-intake** | Normalizes audit request events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **audit-evidence-packet** | Assembles scope, control ownership, prior findings, and evidence requirements into a context packet | AI act within policy | analysis | SearchSparkle |
| 3 | **audit-gap-detection** | Detects missing evidence, scope gaps, traceability issues, and timeline risks | AI assist | analysis | Flag |
| 4 | **audit-request-routing** | Routes evidence requests to control owners per the reviewer matrix | AI draft + approve | communication | Mail |
| 5 | **audit-case-comms** | Drafts audit summaries, evidence requests, follow-ups, and escalation notices | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Audit Request    │  Normalize audit request → structured case record
│     Intake           │  Validates required fields, checks duplicate engagements
│     (audit-request-  │  SLA: 2 business days to review-ready packet
│      intake)         │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Evidence         │  Assemble scope definition, control ownership,
│     Packet           │  prior findings, evidence requirements, methodology
│     (audit-evidence- │  references, key contacts
│      packet)         │  Output: Word audit packet + Excel evidence checklist
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Gap Detection    │  Compare evidence requirements against submitted
│     (audit-gap-      │  materials, detect missing evidence, wrong-period
│      detection)      │  evidence, uncontacted owners, remediation gaps
│                      │  Output: Adaptive Card gap report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Request          │  Route evidence requests to control owners per
│     Routing          │  reviewer matrix, resolve via org hierarchy,
│     (audit-request-  │  enforce segregation of duties, set deadlines
│      routing)        │  Output: Teams notifications + calendar reminders
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Case             │  Draft audit manager summaries, evidence requests,
│     Communications   │  overdue follow-ups, status updates, escalation
│     (audit-case-     │  notices, methodology-cited packet summaries
│      comms)          │  Output: Outlook drafts + Teams coordination
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Confirm Audit    │  Confirm packet is review-ready, escalated, or
│     Packet           │  returned for additional scope clarification.
│     Disposition      │  Human-only step.
│     (not automated)  │
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Audit Request Intake | Structured field extraction, date validation, duplicate checking — no AI judgment |
| **AI act within policy** | Evidence Packet | Retrieves approved context from defined sources; does not interpret evidence or make audit judgments |
| **AI assist** | Gap Detection | Surfaces missing evidence and scope gaps as observations with severity levels; auditor reviews before any action |
| **AI draft + approve** | Request Routing, Case Comms | AI recommends routing or drafts communications; lead auditor reviews and confirms before any action |
| **Human only** | Confirm Disposition | Final audit packet readiness, scope decisions, and testing authorization are always human decisions |

## Governance Controls

### Audit Independence

- No skill draws audit conclusions, assesses control effectiveness, or rates risk — all outputs present factual assembly and gap identification only
- Skills do not influence audit scope, testing strategy, or engagement findings
- Routing guardrails prevent assigning evidence requests to individuals under audit for the same engagement
- Communications to control owners contain only evidence request specifics and engagement references — never preliminary observations or audit deliberations

### Workpaper Integrity

- All generated documents (audit packets, summaries, evidence checklists) are designated for storage in versioned SharePoint engagement folders
- Skills never modify submitted evidence documents — they only read and reference
- Every document in the evidence packet includes source reference, version, and retrieval timestamp
- Methodology-cited packet summaries reference specific audit program sections and IIA Standards

### Evidence Chain of Custody

- Skills log every evidence document accessed with timestamp, source location, and document version
- Evidence references include document name, version, location, and access timestamp
- The gap report documents what was checked, what was found, and the specific checklist item reference for every gap
- All routing decisions are logged with timestamp, actor, and rationale

### Segregation of Duties

- Routing guardrails enforce that evidence requestors are not also evidence approvers
- Skills flag conflicts when a control owner is asked to provide evidence for their own control effectiveness assessment
- All routing follows the documented reviewer matrix and control ownership records — no skill bypasses or substitutes role assignments

### Data Classification

- No audit observations, preliminary findings, or testing strategies appear in any outgoing communication
- All internal audit communications carry appropriate confidentiality markings
- Sensitive audit content is confined to Outlook drafts — never posted in Teams messages to non-audit recipients
- Information in routing messages is limited to engagement IDs, evidence checklist item references, and SharePoint folder locations

### Professional Standards

- Skills reference IIA Standards and organizational audit methodology in evidence packets
- Methodology documents are read dynamically from SharePoint at runtime — audit operations can update methodology without modifying skill definitions
- Skills flag when required methodology steps have no corresponding evidence

## Evidence Gap Categories

| Gap Type | Description | Severity |
|----------|-------------|----------|
| Missing mandatory evidence | Required evidence item with no submission in engagement folder | High / Critical |
| Wrong-period evidence | Evidence submitted but covering a different timeframe than the audit period | Medium / High |
| Uncontacted control owner | Control owner identified but no evidence request sent or response received | Medium / High |
| Prior finding remediation not evidenced | Prior audit finding flagged for follow-up but no remediation evidence provided | Medium / High |
| Unassigned scope area | Process area or control within scope has no identified control owner | High |
| Traceability gap | Evidence submitted but no clear link to the specific control or checklist item | Low |

## Timeline Risk Levels

| Risk Level | Criteria |
|------------|----------|
| **On Track** | 5+ business days to next milestone with no high-severity gaps |
| **At Risk** | Fewer than 5 business days to next milestone OR high-severity gaps requiring action |
| **Critical** | Fewer than 2 business days to next milestone OR critical gaps unresolved |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Audit Request Intake — find original audit request or scheduling email |
| SearchM365 (files) | All skills — find case tracker, methodology, control library, evidence checklists, prior findings, templates |
| SearchM365 (connectors) | Evidence Packet — audit management platform data via Graph Connector |
| ReadFileContent | All skills — read tracker, methodology, control library, evidence checklists, prior workpapers |
| GetDriveChildren | Evidence Packet, Gap Detection — browse engagement evidence folders and prior workpapers |
| GetMessage | Audit Request Intake — read full request email content |
| SearchPeople / GetUserDetails | All skills — resolve auditors, control owners, process owners, business liaisons |
| GetManagerDetails / GetDirectReportsDetails | Evidence Packet, Request Routing — reporting chain and escalation paths |
| CreateDraftMessage | Case Comms — all formal evidence requests and audit communications (never auto-send) |
| PostMessage | Request Routing, Case Comms — Teams notifications to control owners and audit team coordination |
| CreateEvent | Request Routing — calendar deadline reminders for critical evidence items |
| render_ui (Adaptive Card) | Audit Request Intake, Gap Detection, Request Routing — confirmations, gap reports, routing recommendations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Audit case record | Excel tracker row | Audit Request Intake |
| Audit evidence packet | Word document | Evidence Packet |
| Evidence requirements checklist | Excel workbook | Evidence Packet |
| Evidence gap report | Adaptive Card | Gap Detection |
| Evidence request notifications | Teams direct messages | Request Routing |
| Deadline reminders | Calendar events | Request Routing |
| Audit manager summary | Outlook draft | Case Comms |
| Evidence request to control owner | Outlook draft | Case Comms |
| Overdue evidence follow-up | Outlook draft | Case Comms |
| Status update to process owner | Outlook draft | Case Comms |
| Escalation notice | Outlook draft | Case Comms |
| Methodology-cited packet summary | Word document | Case Comms |

## Federated Data Access

Internal Audit workflows depend on external systems (audit management platform, GRC platform, control repository, workflow system). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Audit management platform engagement metadata, finding records, and evidence status indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Audit methodology, evidence checklists, control library, control ownership records, reviewer matrices, prior findings maintained in SharePoint via Power Automate |
| **Tier 3** | Manual Input | Walkthrough observations, verbal explanations from control owners, ad hoc audit requests captured via structured intake |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for all reference data (methodology, control library, prior findings, reviewer matrices) and Tier 3 for walkthrough observations and ad hoc requests. Introduce audit management platform Graph Connectors in Wave 2 after skill workflows are proven.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── audit-request-intake/SKILL.md
├── audit-evidence-packet/SKILL.md
├── audit-gap-detection/SKILL.md
├── audit-request-routing/SKILL.md
└── audit-case-comms/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Audit case tracker** — shared Excel workbook (Case ID, Engagement Type, Process Area, Business Unit, Legal Entity, Audit Period, Lead Auditor, Status, Created Date, Evidence Status, Prior Findings Count, Disposition, Audit Log)
- **Audit methodology documents** — organizational audit standards, testing approach guidance, IIA Standards references
- **Evidence checklists** — per-control-area checklists of required evidence items with sources, formats, and owners
- **Control library** — control descriptions, control IDs, control owners, process narratives
- **Control ownership records** — who owns which controls by process area and business unit
- **Reviewer matrix** — defines who reviews what by engagement type, gap severity, and escalation level
- **Prior findings summaries** — historical audit findings with remediation status by process area
- **Communication templates** — approved templates for evidence requests, follow-ups, summaries, and escalation notices
- **Engagement folder structure** — per-engagement folders in SharePoint for evidence uploads, workpapers, and generated documents

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Audit Request Intake | "new audit request", "audit intake for [process area]", "log audit case", "walkthrough scheduled" |
| Evidence Packet | "build audit packet for [case]", "assemble evidence context", "pull prior findings", "what evidence do we need" |
| Gap Detection | "check evidence gaps", "what's missing for audit [case ID]", "audit readiness check", "scope gap analysis" |
| Request Routing | "route evidence requests", "who owns [control area]", "assign evidence requests", "set up review chain" |
| Case Comms | "draft audit summary", "prepare evidence request for [control owner]", "remind [owner] about evidence", "escalate overdue evidence" |

## Implementation Roadmap

### Wave 1 — Foundation and Gap Detection

- Create the shared Excel audit case tracker in SharePoint
- Upload audit methodology, evidence checklists, control library, and prior findings to SharePoint
- Create per-engagement folder structure in SharePoint for evidence and workpapers
- Upload reviewer matrix and control ownership records
- Build skills: `audit-request-intake`, `audit-evidence-packet`, `audit-gap-detection`
- Operate in AI assist mode — all outputs presented for auditor review
- Test with 5-10 real audit engagements across different engagement types

### Wave 2 — Routing and Communication

- Build skills: `audit-request-routing`, `audit-case-comms`
- Promote intake to write mode (creates case records after confirmation)
- Promote gap detection to write-back mode (updates evidence checklist after user confirmation)
- Set up daily scheduled prompt to check for engagements with approaching evidence deadlines
- Set up weekly scheduled prompt to identify overdue evidence requests and unresponsive control owners
- Introduce Graph Connectors for audit management platform data if available

### Wave 3 — Optimization and Expansion

- Add bounded multi-source evidence packet assembly (email, SharePoint, Teams, control library)
- Add proactive detection of recurring evidence gaps and unresponsive control owner patterns
- Refine gap detection thresholds based on auditor override patterns from Waves 1-2
- Measurement targets: 85%+ gap detection rate, 70%+ draft acceptance rate, below 5% incorrect routing rate, above 90% traceability coverage, below 20% override rate

## Implementation Notes

- **Audit packet disposition is intentionally not automated** — final readiness decisions, scope judgments, and testing authorization are always human actions
- **No skill draws audit conclusions** — this is a permanent architectural constraint; control effectiveness assessments, risk ratings, and finding severity are always auditor-owned
- **Evidence chain of custody is enforced at every skill boundary** — every document access is logged with timestamp, source, and version
- **Segregation of duties is enforced in routing** — skills flag conflicts where requestors would approve their own evidence or control owners would assess their own controls
- **The triple-backbone artifact pattern** (Excel + Word + SharePoint) provides structured tracking, narrative workpapers, and evidence storage with versioning and access control
- **All formal communications are Outlook drafts** — nothing is sent without explicit lead auditor confirmation
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., running a gap check on an existing engagement, or drafting a follow-up without re-running the full workflow)
