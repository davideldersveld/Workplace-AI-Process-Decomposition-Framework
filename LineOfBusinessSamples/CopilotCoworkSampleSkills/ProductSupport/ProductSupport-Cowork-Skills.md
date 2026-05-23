# Product Support Escalated Case Triage and Engineering Handoff — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Escalated Case Triage and Engineering Handoff** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert escalated support cases into a governed, evidence-backed engineering handoff workflow within Microsoft 365.

The workflow supports escalation signals from email, Teams, support platform submissions, and direct escalation, guiding each through structured normalization, evidence assembly, defect classification, impact assessment, engineering routing, and handoff drafting with strict controls for evidence integrity, severity decision control, cross-team handoff quality, customer communication review, and complete audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **ps-escalation-intake** | Normalizes escalated support cases into structured escalation records | Deterministic automation | analysis | TaskListLtr |
| 2 | **ps-evidence-packet** | Assembles case evidence, logs, telemetry, and known-issue context | AI act within policy | analysis | SearchSparkle |
| 3 | **ps-defect-classifier** | Classifies issue type, product area, and suspected defect path | AI assist | analysis | Tag |
| 4 | **ps-impact-assessment** | Assesses customer impact, severity, and incident linkage | AI draft + approve | analysis | Flag |
| 5 | **ps-engineering-routing** | Routes escalation to engineering owner or queue | AI act within policy | communication | Mail |
| 6 | **ps-handoff-drafter** | Drafts engineering handoff summary and customer status update | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Escalation       │  Normalize escalation → structured record
│     Intake           │  Validates fields, checks duplicates,
│     (ps-escalation-  │  generates Escalation ID, calculates SLA
│      intake)         │  SLA: 4 business hours to engineering-ready
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Evidence Packet  │  Assemble case data, logs, telemetry,
│     (ps-evidence-    │  customer context, prior cases, known
│      packet)         │  issues, reproduction steps, timeline
│                      │  Output: Word evidence packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Defect           │  Classify product area, issue type,
│     Classifier       │  suspected defect path; detect known
│     (ps-defect-      │  issues and duplicate escalations
│      classifier)     │  Output: Adaptive Card + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Impact           │  Assess customer impact, recommend
│     Assessment       │  severity, evaluate incident linkage,
│     (ps-impact-      │  check evidence readiness for handoff
│      assessment)     │  Output: Adaptive Card + severity update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Engineering      │  Route to engineering triage lead per
│     Routing          │  ownership map; send Teams notifications
│     (ps-engineering- │  and post to triage channel
│      routing)        │  Output: Teams messages + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Handoff Drafter  │  Draft engineering handoff summary and
│     (ps-handoff-     │  customer-facing status update email
│      drafter)        │  Output: Word handoff + Outlook draft
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Handoff  │  Confirm case is handed off, escalated
│     Disposition      │  further, or returned for more evidence.
│     (not automated)  │  Human-only step — engineering
│                      │  prioritization and customer
│                      │  commitments remain human-owned.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Escalation Intake | Structured field extraction, validation, duplicate checking, SLA calculation — no AI judgment on severity or investigation path |
| **AI act within policy** | Evidence Packet, Engineering Routing | Retrieves approved context from defined data sources or routes within defined ownership rules without exercising judgment on severity or priority |
| **AI assist** | Defect Classifier | Surfaces classification recommendations with confidence levels and known-issue matches; escalation engineer reviews before tracker updates |
| **AI draft + approve** | Impact Assessment, Handoff Drafter | AI recommends severity or drafts communications; escalation engineer reviews and confirms before actions are taken |
| **Human only** | Confirm Disposition | Final handoff confirmation, severity escalation, and customer commitment decisions are always human-owned |

## Governance Controls

### Evidence Integrity

- Skills never modify or delete original evidence files — the evidence packet is a read-only synthesis with source citations
- Every data point in the evidence packet cites its source document and retrieval timestamp
- Log bundles, screenshots, HAR files, and reproduction notes remain untouched in the SharePoint evidence folder
- Evidence completeness is assessed before routing — incomplete packets are flagged, not silently forwarded

### Severity Decision Control

- Severity recommendations require explicit escalation engineer confirmation before applying to the tracker
- Sev 1 and Sev 2 declarations require product support manager approval before routing proceeds
- No skill downgrades a severity set by a previous human decision without documented rationale
- Escalations meeting incident criteria are always flagged, even if overall severity appears low
- Severity classification and defect classification are separate skills with separate approval gates

### Cross-Team Handoff Quality

- Engineering handoff documents must include: problem statement, evidence inventory, reproduction steps (or explicit gap note), customer impact, and suggested investigation path
- Skills flag if any required handoff section is empty — incomplete handoffs waste engineering time
- Evidence readiness is checked before routing — escalations with insufficient evidence are held for additional collection
- The evidence packet is designed to be self-contained — engineering should not need to ask support for clarification

### Customer Communication Control

- All customer-facing communications are created as Outlook drafts — never sent without explicit escalation engineer confirmation
- Customer update drafts never include internal severity classifications, engineering queue names, or triage notes
- No timeline commitments in customer communications — use "actively investigating" language, not specific dates
- Internal investigation hypotheses are never revealed in customer-facing drafts

### Audit Trail

- Every escalation records the originating case ID, escalating agent, creation timestamp, and Escalation ID
- Every classification records the product area, issue type, confidence level, and confirming engineer
- Every severity decision records the recommended level, rubric justification, and confirming user (plus manager for Sev 1/2)
- Every routing decision records the assigned engineer, queue, rationale, and timestamp
- Every communication records the type, audience, channel, and confirming user
- The Excel escalation tracker serves as the Cowork-accessible audit record

### Data Sensitivity Summary

| Data Type | Handling Rule |
|-----------|--------------|
| Customer financial data (contract value, ARR) | Account tier indicators only — never exact figures |
| Internal severity classifications | Never in customer-facing communications |
| Engineering queue names and triage notes | Internal only — never in customer communications |
| Investigation hypotheses | Internal only — customer sees "actively investigating" |
| Log bundles and diagnostic data | Permission-scoped; never modified by skills |

## Severity Levels

| Severity | Criteria | Response Path |
|----------|---------|--------------|
| **Sev 1 — Critical** | Complete loss of service, data at risk, or critical business function blocked for enterprise customer with no workaround | Immediate engineering engagement, incident bridge consideration, product support manager approval required |
| **Sev 2 — High** | Major feature impaired, significant business impact, or enterprise customer blocked with limited workaround | Same-day engineering engagement, elevated priority routing |
| **Sev 3 — Standard** | Feature issue with available workaround, moderate business impact, or non-critical function affected | Standard SLA engineering engagement |
| **Sev 4 — Low** | Minor issue, cosmetic problem, documentation question, or feature request with minimal business impact | Standard queue, next available cycle |

## Issue Type Classification

| Issue Type | Criteria |
|-----------|---------|
| **Defect** | Product behavior differs from documented specification or expected behavior |
| **Configuration** | Issue caused by customer-specific configuration, environment, or integration setup |
| **Documentation** | Product works as designed but documentation is missing, incorrect, or misleading |
| **Feature gap** | Customer needs functionality that does not currently exist |
| **Performance** | Product functions correctly but response time, throughput, or resource usage is unacceptable |
| **Regression** | Functionality that previously worked has broken in a recent release |

## Evidence Completeness Levels

| Level | Criteria |
|-------|---------|
| **Complete** | Log bundle present, reproduction steps documented, environment details captured, customer impact quantified |
| **Partial** | Some evidence present but critical items missing (no logs, no repro steps, or no environment details) |
| **Insufficient** | Critical evidence missing that would block engineering investigation — flag for additional collection |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find escalation tracker, evidence documents, taxonomy, severity rubric, ownership map, templates, known issues list |
| SearchM365 (connectors) | Evidence Packet, Defect Classifier, Impact Assessment — pull support case data, known issues, prior defects, customer entitlement via Graph Connectors |
| SearchM365 (email) | Escalation Intake, Evidence Packet, Handoff Drafter — find escalation threads, customer correspondence |
| SearchM365 (teams) | Escalation Intake, Evidence Packet — find support-to-engineering discussions |
| ReadFileContent | All skills — read tracker, evidence files, policies, templates, known issues |
| GetDriveChildren | Evidence Packet, Handoff Drafter — inventory evidence folder contents |
| SearchPeople / GetUserDetails | Escalation Intake, Engineering Routing — resolve agents, owners, triage leads |
| GetManagerDetails / GetDirectReportsDetails | Engineering Routing — resolve escalation paths for high-severity cases |
| ListCalendarView | Engineering Routing — check engineering lead availability |
| PostMessage | Engineering Routing — Teams notifications to engineering triage leads |
| PostChannelMessage | Engineering Routing — escalation summaries to engineering triage channels |
| CreateDraftMessage | Handoff Drafter — customer status update emails as Outlook drafts |
| render_ui (Adaptive Card) | Escalation Intake, Defect Classifier, Impact Assessment, Engineering Routing, Handoff Drafter — decision surfaces and status summaries |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Escalation case record | Excel tracker row | Escalation Intake |
| Evidence packet | Word document | Evidence Packet |
| Classification report | Adaptive Card + Excel tracker update | Defect Classifier |
| Impact assessment | Adaptive Card + Excel severity update | Impact Assessment |
| Engineering routing notifications | Teams direct messages + channel posts | Engineering Routing |
| Engineering handoff summary | Word document | Handoff Drafter |
| Customer status update | Outlook draft email | Handoff Drafter |
| Customer success notification | Teams message or Outlook draft | Handoff Drafter |

## Federated Data Access

Product Support depends on support platforms, issue trackers, logging platforms, and telemetry systems. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Support platforms (Zendesk, ServiceNow) for case records and customer data; issue trackers (Jira, Azure DevOps) for known issues, defect history, and incident status |
| **Tier 2** | SharePoint Bridge | Known issues list synced via Power Automate; engineering ownership map maintained in Excel; product area taxonomy and defect classification guide; severity rubric and escalation criteria |
| **Tier 3** | Manual Input | Log bundles, crash reports, and telemetry uploaded by support agents to SharePoint evidence folders; reproduction steps captured via structured intake |

**Recommended pilot approach:** Start with Tier 2 (SharePoint bridge for ownership maps, known issues, and classification references) and Tier 3 (manual evidence upload for logs and telemetry). Introduce Graph Connectors for the support platform and issue tracker in Wave 2 after skill workflows are proven.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── ps-escalation-intake/SKILL.md
├── ps-evidence-packet/SKILL.md
├── ps-defect-classifier/SKILL.md
├── ps-impact-assessment/SKILL.md
├── ps-engineering-routing/SKILL.md
└── ps-handoff-drafter/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Escalation tracker** — shared Excel workbook (Escalation ID, Originating Case ID, Customer Name, Account ID, Product Area, Issue Summary, Suspected Defect Path, Severity, Status, Created Date, SLA Deadline, Assigned To, Engineering Queue, Evidence Completeness, Resolution, Last Updated, Updated By)
- **Severity rubric** — severity level definitions with criteria thresholds and escalation triggers
- **Product area taxonomy** — standard product area hierarchy with component definitions
- **Defect classification guide** — issue type definitions (defect, configuration, documentation, feature gap, performance, regression)
- **Engineering ownership map** — product area to engineering team and triage lead mapping
- **Escalation routing rules** — routing policies, severity-based escalation paths, queue assignments
- **Incident criteria** — conditions that trigger incident declaration
- **Known issues list** — synced from issue tracker (or maintained manually in SharePoint)
- **Engineering handoff template** — standard handoff document structure
- **Customer update template** — customer communication templates for escalation acknowledgment and status updates
- **Per-escalation evidence folder template** — SharePoint folder structure for logs, screenshots, HAR files, and reproduction notes

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Escalation Intake | "new escalation for [case]", "escalated case from [agent]", "product issue for [customer]", "suspected defect from support" |
| Evidence Packet | "build evidence packet for [ID]", "pull logs and context for [case]", "what evidence do we have for this defect" |
| Defect Classifier | "classify this escalation", "what product area is this", "is this a known issue", "check if this is a duplicate" |
| Impact Assessment | "assess impact for [ID]", "what severity is this", "check urgency", "is this a critical escalation" |
| Engineering Routing | "route escalation to engineering", "assign engineering owner", "send to [area] queue", "who handles this" |
| Handoff Drafter | "draft engineering handoff", "write handoff summary", "prepare customer update", "escalation handoff to engineering" |

## Implementation Roadmap

### Wave 1 — Foundation

- Create the shared Excel escalation tracker in SharePoint with standard columns
- Upload severity rubric, product area taxonomy, engineering ownership map, and handoff templates to SharePoint
- Create per-escalation evidence folder template in SharePoint
- Configure SharePoint bridge data: known issues list, engineering ownership map
- Build skills: `ps-escalation-intake`, `ps-evidence-packet`, `ps-defect-classifier`, `ps-handoff-drafter`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 10-15 real escalations across multiple product areas and severity levels

### Wave 2 — Routing and Impact Assessment

- Build skills: `ps-impact-assessment`, `ps-engineering-routing`
- Promote intake to write mode (creates escalation records after confirmation)
- Promote classifier to write-back mode (updates tracker classification after confirmation)
- Promote routing to bounded write mode (posts to Teams engineering channels within ownership map)
- Set up scheduled prompt (every 30 minutes): check for cases approaching 4-hour SLA breach
- Set up daily scheduled prompt: summarize escalation volume, average time to engineering-ready, and severity distribution
- Introduce Graph Connectors for support platform and issue tracker
- Measurement targets: above 80% severity agreement rate, above 85% correct routing rate

### Wave 3 — Optimization and Proactive Detection

- Add bounded multi-source escalation packet assembly for complex cross-system evidence
- Add proactive defect pattern detection: scheduled prompt analyzing recent escalations for recurring product areas and increasing severity trends
- Add evidence gap monitoring: flag in-progress escalations with incomplete evidence before SLA deadline
- Measurement targets: above 90% handoff completeness, under 3 hours time to engineering-ready, above 60% duplicate detection rate, above 70% draft acceptance rate

## Implementation Notes

- **Handoff disposition is intentionally not automated** — final escalation confirmation, engineering prioritization, and customer commitments carry product support accountability that cannot be delegated to AI
- **The evidence packet is the most critical single output** — it must be comprehensive enough that engineering can begin investigation without asking support for clarification
- **Evidence files are never modified by any skill** — the evidence packet is a read-only synthesis with source citations; original logs, screenshots, and artifacts remain untouched
- **Severity declarations require explicit human confirmation** — Sev 1 and Sev 2 additionally require product support manager approval because they affect engineering prioritization and customer expectations
- **Classification and severity are separate skills** — defect classification (what) is independent of impact assessment (how bad), preventing premature severity inflation and supporting clearer decision-making
- **The 4-hour SLA** drives urgency across all skills — every output surfaces SLA countdown and time remaining
- **Cross-system state synchronization is manual in Wave 1** — the escalation tracker in SharePoint is the Cowork-accessible state; Power Automate flows bridge to the issue tracker in Wave 2
- **Known-issue detection accelerates resolution** — surfacing matching known issues during classification can resolve escalations faster than a full engineering investigation
