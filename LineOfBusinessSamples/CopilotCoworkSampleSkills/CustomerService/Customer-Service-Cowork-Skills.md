# Customer Service Case Intake and Resolution Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Case Intake and Resolution Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the customer service triage process — from intake through response drafting — into AI-assisted capabilities within Microsoft 365.

The workflow supports inbound service events from email, chat, web forms, and call transcripts, guiding each through structured classification, severity assessment, routing, and response preparation with SLA awareness and customer commitment controls at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **cs-case-intake** | Normalizes inbound service events into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **cs-context-packet** | Assembles customer profile, account history, entitlements, and prior cases | AI act within policy | analysis | SearchSparkle |
| 3 | **cs-issue-classifier** | Classifies issue type and customer intent with confidence scoring | AI assist | analysis | Tag |
| 4 | **cs-severity-assessment** | Assesses severity, SLA path, and escalation conditions | AI draft + approve | analysis | Flag |
| 5 | **cs-case-routing** | Routes to the correct agent or queue based on routing rules | AI act within policy | communication | Mail |
| 6 | **cs-response-drafter** | Drafts customer responses, handoff summaries, and escalation packets | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Case Intake      │  Normalize inbound event → structured case record
│     (cs-case-intake) │  Supports: email, chat, form, call transcript
│                      │  Calculates SLA deadline (15-min routing target)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble customer profile, entitlement, prior cases,
│     (cs-context-     │  product context, known issues, account owner
│      packet)         │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Issue Classifier │  Classify issue type from taxonomy, detect customer
│     (cs-issue-       │  intent, score confidence, surface similar prior cases
│      classifier)     │  Output: Adaptive Card classification
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Severity         │  Assess severity (Critical/High/Standard/Low),
│     Assessment       │  determine SLA path, identify escalation conditions
│     (cs-severity-    │  Output: Adaptive Card severity report
│      assessment)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Case Routing     │  Route to agent/queue per routing matrix,
│     (cs-case-        │  check availability, send Teams notifications
│      routing)        │  Output: Teams messages + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Response         │  Draft customer reply, handoff summary, or
│     Drafter          │  escalation packet using knowledge articles
│     (cs-response-    │  Output: Outlook draft + Teams handoff
│      drafter)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Human            │  Confirm triage disposition — approve routing,
│     Disposition      │  authorize escalation, or return for reclassification.
│     (not automated)  │  Human-only step.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Case Intake | Structured field extraction, duplicate checking, SLA calculation — no AI judgment |
| **AI act within policy** | Context Packet, Case Routing | Retrieves approved context and routes per the routing matrix; does not interpret or override rules |
| **AI assist** | Issue Classifier | Surfaces classification as recommendation with confidence score; agent reviews before applying |
| **AI draft + approve** | Severity Assessment, Response Drafter | AI recommends severity or drafts response; agent reviews and confirms before any action |
| **Human only** | Final Disposition | Triage confirmation and escalation authorization are always human decisions |

## Governance Controls

### Customer Commitment Control
- No skill may make customer commitments — credits, refunds, timeline promises, or guarantees require supervisor approval
- All customer-facing communications are created as Outlook drafts — never sent without explicit agent confirmation
- Response templates are grounded in knowledge articles, not generated from model knowledge alone

### SLA Awareness
- SLA deadline is calculated at intake (15 minutes for standard routing) and tracked in the case tracker
- Every skill that touches the case surfaces the SLA countdown
- Severity assessment determines the SLA path based on severity level and customer entitlement tier
- Scheduled prompts can monitor for approaching SLA breaches (recommended: every 15 minutes)

### Customer Data Protection
- Financial account details (credit card numbers, billing specifics) are masked in generated documents
- Customer privacy is protected in Teams channel posts — case ID and summary only, no full customer details
- Internal routing information, severity levels, and agent names never appear in customer-facing drafts

### Classification Integrity
- Classification is always presented as a recommendation with confidence scoring, never auto-applied
- Low-confidence classifications are prominently flagged
- Classification is based on issue content, never on customer identity or account tier (prevents bias)

### Escalation Safety
- High-severity and escalation cases are presented for team lead review before routing
- Customer-reported severity is never downgraded without documented rationale
- Escalation conditions are flagged independently of overall severity assessment

## Severity Levels and SLA Targets

| Severity | Description | Response SLA | Resolution Path SLA |
|----------|-------------|-------------|-------------------|
| **Critical** | Business-stopping, data loss, safety, production outage | 1 hour | 4 hours |
| **High** | Significant impact, degraded workaround, premium customer impacted | 4 hours | 1 business day |
| **Standard** | Normal operational issue, common request, inquiry | 15-minute routing | 1 business day first response |
| **Low** | Informational, feature request, non-urgent enhancement | 1 business day | 3 business days |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (email) | Case Intake, Context Packet — find customer emails |
| SearchM365 (files) | All skills — find tracker, taxonomy, policies, knowledge articles, templates |
| SearchM365 (teams) | Context Packet — find internal discussions about the customer |
| SearchM365 (connectors) | Intake, Context Packet, Classifier, Severity — CRM data via Graph Connector |
| ReadFileContent | All skills — read tracker, taxonomy, SLA policy, routing rules, knowledge articles |
| GetDriveChildren | Context Packet — browse case folders and account documents |
| GetMessage | Case Intake, Classifier — read original inbound email |
| SearchPeople / GetUserDetails | Intake, Context Packet, Routing — resolve identities |
| GetMyDetails | Case Intake — current user for tracking |
| GetManagerDetails / GetDirectReportsDetails | Context Packet, Routing — escalation chain |
| ListCalendarView | Case Routing — check agent availability |
| CreateDraftMessage | Response Drafter — customer reply drafts (never auto-send) |
| PostMessage | Routing, Response Drafter — Teams notifications to agents |
| PostChannelMessage | Case Routing — case card to triage channel |
| render_ui (Adaptive Card) | Classifier, Severity, Intake — real-time decision support |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Case record | Excel tracker row | Case Intake |
| Context packet | Word document | Context Packet |
| Classification report | Adaptive Card | Issue Classifier |
| Severity assessment | Adaptive Card | Severity Assessment |
| Routing notification (agent) | Teams direct message | Case Routing |
| Triage channel card | Teams channel post | Case Routing |
| Customer acknowledgment | Outlook draft | Response Drafter |
| Customer first response | Outlook draft | Response Drafter |
| Follow-up request | Outlook draft | Response Drafter |
| Resolution confirmation | Outlook draft | Response Drafter |
| Internal handoff summary | Teams message | Response Drafter |
| Escalation packet | Word document | Response Drafter |

## Federated Data Access

Customer Service depends on external systems (CRM, case management, telephony). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | CRM account profiles, case history, entitlement data indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Customer lookup tables, knowledge article index, routing rules synced via Power Automate |
| **Tier 3** | Manual Input | Telephony transcripts, chat logs, systems with no integration path |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge) for CRM data and Tier 3 for telephony. Introduce Graph Connectors in Wave 2.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── cs-case-intake/SKILL.md
├── cs-context-packet/SKILL.md
├── cs-issue-classifier/SKILL.md
├── cs-severity-assessment/SKILL.md
├── cs-case-routing/SKILL.md
└── cs-response-drafter/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Service case tracker** — shared Excel workbook for case records (Case ID, Customer, Account, Channel, Category, Priority, Status, SLA Deadline, Assigned To, Resolution)
- **Issue taxonomy** — document defining issue categories, intents, and category-to-queue mappings
- **SLA policy** — document defining severity levels, SLA targets, and escalation criteria
- **Routing rules matrix** — defines which agents and queues handle which issue categories and severity levels
- **Response templates** — approved templates for acknowledgments, first responses, follow-ups, and escalation packets
- **Knowledge article library** — resolution guidance documents indexed by issue category

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Case Intake | "new service case", "log case for Acme Corp", "customer email case", "intake from chat" |
| Context Packet | "build context for CS-2026-00142", "what do we know about this customer", "pull customer history" |
| Issue Classifier | "classify this case", "what type of issue is this", "triage this ticket", "categorize case CS-2026-00142" |
| Severity Assessment | "assess severity for CS-2026-00142", "what priority is this", "is this an escalation", "check SLA path" |
| Case Routing | "route case CS-2026-00142", "assign this case", "send to the right queue", "who handles this" |
| Response Drafter | "draft response for CS-2026-00142", "write customer reply", "prepare handoff summary", "escalation summary" |

## Implementation Roadmap

### Wave 1 — Foundation
- Create the shared Excel case tracker in SharePoint
- Upload SLA policies, routing rules, issue taxonomy, and response templates
- Configure SharePoint bridge data: customer entitlement lookup, knowledge article index
- Build skills: `cs-case-intake`, `cs-context-packet`, `cs-issue-classifier`, `cs-response-drafter`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 10-15 real cases across multiple issue types and channels

### Wave 2 — Routing and Severity
- Build skills: `cs-severity-assessment`, `cs-case-routing`
- Promote intake to write mode (creates records after confirmation)
- Promote classifier to write-back mode (updates tracker after confirmation)
- Set up scheduled prompt (every 15 minutes) for SLA breach monitoring
- Set up daily summary prompt for case volume, triage time, and SLA compliance

### Wave 3 — Optimization and Proactive Support
- Introduce Graph Connectors for CRM data if available
- Add escalation packet assembly (multi-source Word document)
- Add proactive reopen detection (analyze recently closed cases for reopen patterns)
- Add knowledge gap detection (identify cases with no matching knowledge article)
- Measurement targets: 85%+ classification accuracy, 90%+ routing accuracy, under 10 minutes first response, under 5% SLA breach rate, zero unauthorized commitment incidents

## Implementation Notes

- **Human disposition is intentionally not automated** — final triage confirmation and escalation authorization are human-only decisions
- **All customer-facing communications are Outlook drafts** — nothing is sent without explicit agent confirmation
- **Classification operates in recommendation mode** — confidence scores and evidence are always visible so agents make informed decisions
- **SLA is a cross-cutting concern** — every skill surfaces time constraints; scheduled prompts provide breach monitoring
- **Adaptive Cards are the primary decision surface** — service agents work in real time and need inline recommendations, not long documents
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., drafting a response for an existing case, or reclassifying after new information)
