# IT Service Management Incident Intake and Triage — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Incident Intake and Triage** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the incident management triage process — from incident intake through review-ready disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports incident signals from email, Teams chat, phone, monitoring alerts, and analyst intake, guiding each through structured normalization, user and service context enrichment, incident classification, severity assessment, resolver group routing, and communication drafting with strict major incident controls, production change discipline, SLA accountability, audience-scoped language rules, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **itsm-incident-intake** | Normalizes inbound incident signals into structured incident records | Deterministic automation | analysis | TaskListLtr |
| 2 | **itsm-context-packet** | Assembles user, service, asset, change, and incident history into a context packet | AI act within policy | analysis | SearchSparkle |
| 3 | **itsm-classification-assist** | Classifies incident type, affected service, and initial diagnosis path | AI assist | analysis | Tag |
| 4 | **itsm-severity-recommend** | Assesses severity using impact-urgency matrix with escalation path | AI draft + approve | analysis | Flag |
| 5 | **itsm-assignment-router** | Routes incidents to resolver groups per assignment rules | AI act within policy | communication | Mail |
| 6 | **itsm-comms-drafter** | Drafts user updates, handoff summaries, escalation notices, and major incident briefs | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Incident Intake  │  Normalize incident signal → structured record
│     (itsm-incident-  │  Validates required fields, checks duplicates
│      intake)         │  SLA: 10 minutes for standard classification + routing
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble affected user profile, service and
│     (itsm-context-   │  asset details from CMDB, recent changes,
│      packet)         │  related open incidents, prior history,
│                      │  knowledge article references
│                      │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Classification   │  Compare symptoms against incident taxonomy,
│     Assist           │  determine category, subcategory, affected
│     (itsm-           │  service, confidence level
│      classification- │  Output: Adaptive Card classification report
│      assist)         │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Severity         │  Apply impact-urgency matrix, assess
│     Recommend        │  business impact, determine priority
│     (itsm-severity-  │  (P1–P4), identify escalation needs,
│      recommend)      │  check major incident triggers
│                      │  Output: Adaptive Card severity recommendation
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Assignment       │  Route to Desktop Support, Network Ops,
│     Router           │  IAM, App Support, Infrastructure,
│     (itsm-           │  Security Ops, or Major Incident Team
│      assignment-     │  per assignment rules
│      router)         │  Output: Teams notifications + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Comms Drafter    │  Draft user acknowledgments, status updates,
│     (itsm-comms-     │  resolver handoff summaries, escalation
│      drafter)        │  notices, major incident bridge summaries
│                      │  Output: Outlook drafts + Teams + Word docs
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm incident is routed, escalated,
│     Disposition      │  or returned for clarification. Human-only
│     (not automated)  │  step — major incident declaration and
│                      │  containment authorization.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Incident Intake | Structured field extraction, validation, duplicate checking — no AI judgment on incident severity or classification |
| **AI act within policy** | Context Packet, Assignment Router | Retrieves approved context from defined sources or executes routing within pre-approved assignment rules; does not classify or assess severity |
| **AI assist** | Classification Assist | Surfaces classification recommendations with confidence levels; analyst reviews before any action |
| **AI draft + approve** | Severity Recommend, Comms Drafter | AI recommends severity or drafts communications; analyst or incident manager reviews and confirms before any action |
| **Human only** | Confirm Disposition | Major incident declaration, containment authorization, and final triage disposition are always human-owned |

## Governance Controls

### Production Impact and Major Incident Controls

- No skill may declare a major incident or activate the major incident bridge
- No skill may authorize containment actions, production changes, or remediation steps
- No skill may auto-set priority for P1 or P2 incidents without incident manager confirmation
- P1 and P2 incidents always require human confirmation before routing notifications are sent
- Major incident trigger warnings are displayed prominently when severity criteria are met
- Final triage disposition carries operational accountability and remains human-only

### ITSM Platform as Source of Truth

- The ITSM platform (ServiceNow, Jira Service Management, etc.) is the canonical system of record
- The Excel incident tracker in SharePoint is a working copy — not the source of truth
- If tracker data conflicts with ITSM platform data (via Graph Connector), the platform data takes precedence
- Bidirectional state synchronization is handled by Power Automate outside of Cowork
- Skills note the dual-state reality when state consistency matters

### Audience-Scoped Language Rules

| Audience | Internal System Names | Infrastructure Details | Queue Names | PII | Technical Jargon |
|----------|----------------------|----------------------|-------------|-----|------------------|
| End user | Not allowed | Not allowed | Not allowed | Own details only | Avoid |
| Resolver group | Allowed | Allowed | Allowed | Allowed (scoped) | Appropriate |
| Incident manager | Allowed | Allowed | Allowed | Allowed (scoped) | Appropriate |
| Executive / major incident | Not allowed | Not allowed | Not allowed | Minimal | Avoid |

### SLA Accountability

- Standard incidents must be classified and routed within 10 minutes
- SLA targets are calculated per priority level and tracked in every skill output
- Approaching SLA breaches are flagged by scheduled prompt monitoring
- SLA enforcement remains the responsibility of the ITSM platform — skills track and surface, not enforce

### Data Sensitivity

- No raw infrastructure details (IP addresses, hostnames, server names) in user-facing or executive communications
- No internal queue names or system names in user-facing updates
- No security logs, identity credentials, or infrastructure secrets in context packets
- CMDB sync data staleness (older than 24 hours) is flagged in context packets
- Security-classified incidents route to Security Operations with restricted channel visibility

### Audit Trail

- Every incident record includes intake analyst, timestamp, report channel, and source tag for creation audit
- Every classification decision records recommended category, confidence, and analyst final decision
- Every severity assessment records recommended priority, analyst final decision, and override rationale if applicable
- Every routing decision records assigned group, assignee, rationale, and confirming analyst
- Every communication records type, recipient, Incident ID, and timestamp
- The Excel incident tracker serves as the Cowork-accessible audit record

## Incident Categories

| Category | Subcategory Examples | Description |
|----------|---------------------|-------------|
| **Hardware** | Laptop, desktop, peripheral, mobile device | Physical device issues |
| **Software** | Application error, crash, performance, compatibility | Software malfunction or behavior |
| **Network** | Connectivity, VPN, Wi-Fi, DNS, latency | Network access or performance |
| **Identity and access** | Login failure, password reset, MFA issue | Authentication and authorization |
| **Email and collaboration** | Outlook, Teams, SharePoint, OneDrive | Collaboration tool issues |
| **Printing** | Printer, print queue, print driver | Print service issues |
| **Telephony** | Phone system, voicemail, conference bridge | Voice communication issues |
| **Security** | Suspected phishing, malware, data loss concern | Security-related (specialized routing) |
| **Infrastructure** | Server, storage, database, cloud service | Backend infrastructure issues |
| **Business application** | ERP, CRM, HR system, finance system | Line-of-business application issues |

## Priority Levels

| Priority | Description | Response SLA | Resolution SLA | Escalation Path |
|----------|-------------|-------------|----------------|-----------------|
| **P1 — Critical** | Major business impact; service down for large population; no workaround | 15 min | 4 hours | Incident manager, service owner, major incident bridge |
| **P2 — High** | Significant impact; service severely degraded; limited workaround | 30 min | 8 hours | Incident manager, resolver group lead |
| **P3 — Standard** | Moderate impact; service impaired but functional; workaround available | 2 hours | 24 hours | Resolver group |
| **P4 — Low** | Minor impact; inconvenience; full workaround available | 4 hours | 72 hours | Resolver group |

## Impact-Urgency Matrix

| | Critical Urgency | High Urgency | Medium Urgency | Low Urgency |
|---|---|---|---|---|
| **Enterprise-wide** | P1 | P1 | P2 | P3 |
| **Department/site** | P1 | P2 | P2 | P3 |
| **Multiple users** | P2 | P2 | P3 | P4 |
| **Single user** | P2 | P3 | P3 | P4 |

## Resolver Groups

| Group | Handles | Typical Incidents |
|-------|---------|-------------------|
| **Desktop Support** | End-user devices, software, peripherals | Hardware, software (desktop), printing |
| **Network Operations** | Connectivity, VPN, Wi-Fi, DNS | Network category |
| **Identity and Access Management** | Login, password, permissions, MFA | Identity and access category |
| **Collaboration Services** | Outlook, Teams, SharePoint, OneDrive | Email and collaboration category |
| **Telephony Support** | Phone system, voicemail, conferencing | Telephony category |
| **Application Support** | LOB applications, ERP, CRM, HR systems | Business application category |
| **Infrastructure Operations** | Servers, storage, databases, cloud | Infrastructure category |
| **Security Operations** | Phishing, malware, data loss, security events | Security category (restricted routing) |
| **Major Incident Team** | P1 incidents, confirmed major incidents | Any category at P1 |
| **Service Operations Manager** | Escalation when no clear rule match | Ambiguous routing, capacity overflow |

## Communication Types

| Communication | Audience | Channel | Tone |
|---------------|----------|---------|------|
| User acknowledgment | End user | Outlook draft | Empathetic, clear, reassuring |
| User status update | End user | Outlook draft | Clear, helpful, progress-focused |
| Resolver handoff summary | Resolver group | Word doc + Teams DM | Operational, structured, factual |
| Escalation notice | Incident manager | Teams DM or Outlook draft | Concise, structured, action-oriented |
| Major incident bridge summary | Executive / service owners | Outlook draft | Professional, high-level, impact-focused |
| Follow-up information request | End user | Outlook draft | Helpful, specific, action-oriented |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find incident tracker, taxonomy, severity matrix, assignment rules, CMDB sync, service catalog, templates, knowledge articles |
| SearchM365 (connectors) | Context Packet, Classification Assist, Severity Recommend — retrieve CI data, change records, open incidents, known errors from ITSM platform via Graph Connector |
| SearchM365 (email) | Incident Intake, Context Packet — find original report threads and user emails about the service |
| SearchM365 (teams) | Incident Intake, Context Packet — find service desk escalations and IT channel discussions |
| ReadFileContent | All skills — read tracker, taxonomy, matrices, rules, CMDB data, templates, knowledge articles |
| SearchPeople / GetUserDetails | All skills — resolve affected users, resolver group leads, on-call contacts |
| GetManagerDetails | Context Packet — reporting chain for escalation context |
| ListCalendarView | Assignment Router — check resolver availability |
| CreateDraftMessage | Comms Drafter — Outlook drafts for user-facing updates and escalation notices |
| PostMessage | Assignment Router, Comms Drafter — Teams direct messages to resolvers and incident managers |
| PostChannelMessage | Assignment Router — incident management channel assignment posts |
| CreateEvent | Assignment Router — SLA deadline calendar holds for P1/P2 |
| render_ui (Adaptive Card) | Incident Intake, Classification Assist, Severity Recommend — confirmations, classifications, severity recommendations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Incident record | Excel tracker row | Incident Intake |
| Context packet | Word document | Context Packet |
| Classification report | Adaptive Card | Classification Assist |
| Severity recommendation | Adaptive Card | Severity Recommend |
| Assignment notification | Teams direct messages + channel posts | Assignment Router |
| SLA deadline calendar holds | Calendar events (P1/P2) | Assignment Router |
| User acknowledgment | Outlook draft | Comms Drafter |
| User status update | Outlook draft | Comms Drafter |
| Resolver handoff summary | Word document + Teams DM | Comms Drafter |
| Escalation notice | Teams DM or Outlook draft | Comms Drafter |
| Major incident bridge summary | Outlook draft | Comms Drafter |
| Follow-up information request | Outlook draft | Comms Drafter |

## Federated Data Access

Incident management depends on the ITSM platform (ServiceNow, Jira Service Management, BMC), CMDB, monitoring tools, and identity systems. The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | ITSM platform ticket records, CI data, known errors, change records, and knowledge articles indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | CMDB sync data, service catalog, asset inventory, recent change log, incident taxonomy, severity matrix, assignment rules, and communication templates maintained in SharePoint via Power Automate sync |
| **Tier 3** | Manual Input | Monitoring alert details, telemetry context, and data points not available through integration, captured via structured prompts with "manual entry" source tagging |

**Recommended pilot approach:** Start with Tier 2 (SharePoint sync for CMDB, service catalog, and policy documents) and Tier 3 for monitoring context from manual lookups. Introduce Graph Connectors for ServiceNow or Jira in Wave 2 after skill workflows are proven and tenant admin has configured connectors.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── itsm-incident-intake/SKILL.md
├── itsm-context-packet/SKILL.md
├── itsm-classification-assist/SKILL.md
├── itsm-severity-recommend/SKILL.md
├── itsm-assignment-router/SKILL.md
└── itsm-comms-drafter/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Incident tracker** — shared Excel workbook (Incident ID, Reported By, Report Channel, Summary, Affected Service, Category, Priority, Status, Created Date, Assigned To, SLA Target, Resolution Notes)
- **Incident taxonomy** — category and subcategory definitions, service-to-category mappings
- **Severity matrix** — impact-urgency matrix, priority level definitions, SLA targets
- **Escalation policy** — escalation triggers by priority level, notification requirements
- **Assignment rules matrix** — resolver group mapping by category, service, and priority
- **Support model** — team structure, on-call schedules, resolver group directory
- **Service catalog** — service definitions, criticality ratings, business owners, support contacts
- **CMDB sync data** — configuration items, asset inventory, service dependencies (synced from ITSM platform via Power Automate)
- **Recent change log** — changes deployed to services in the last 7 days
- **Communication templates** — user acknowledgment, status update, handoff summary, escalation notice, major incident bridge summary templates
- **Knowledge articles** — troubleshooting guides and runbooks (synced from knowledge base)

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Incident Intake | "new incident from [source]", "log incident for [user]", "ticket came in for [service]", "create incident case" |
| Context Packet | "build context for incident [ID]", "enrich this incident", "get context for ticket [number]", "what do we know about this incident" |
| Classification Assist | "classify this incident", "what type of incident is this", "categorize ticket [ID]", "triage classification" |
| Severity Recommend | "assess severity for [incident]", "what priority should this be", "escalation check", "major incident check" |
| Assignment Router | "route this incident", "assign resolver group", "who handles this type of ticket", "route to the right team" |
| Comms Drafter | "draft user update for [incident]", "write handoff summary", "prepare incident update email", "major incident summary" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel incident tracker workbook in SharePoint with standard columns
- Upload incident taxonomy, severity matrix, assignment rules, and service catalog
- Set up SharePoint folder for CMDB sync data (service ownership, asset inventory, recent changes)
- Create Power Automate flow to sync active incident data from ITSM platform
- Establish the incident management Teams channel for routing and coordination
- Build skills: `itsm-incident-intake`, `itsm-context-packet`, `itsm-classification-assist`, `itsm-comms-drafter`
- Operate in AI assist mode — all outputs presented for analyst review
- Test with 20–30 real incidents over 2 weeks

### Wave 2 — Routing and Severity

- Build skills: `itsm-severity-recommend`, `itsm-assignment-router`
- Promote intake to write mode (creates tracker records after confirmation)
- Promote classification to write-back mode (updates tracker category after analyst confirmation)
- Promote router to "act within policy" for P3/P4 incidents with clear rule matches
- Set up scheduled prompt: check for new unprocessed incidents every 15 minutes
- Set up daily scheduled prompt: flag incidents approaching SLA breach
- Introduce Graph Connectors for ServiceNow or Jira if available

### Wave 3 — Optimization and Proactive Detection

- Add knowledge-article citation in classification and context packets
- Add proactive monitoring: identify repeat incident patterns, flag likely major incidents from volume spikes, surface resolver group queue imbalances
- Introduce additional Graph Connectors for monitoring platforms if available
- Measurement targets: 80%+ classification accuracy, 85%+ correct routing rate, 40% time-to-triage improvement, below 5% SLA breach rate for P1/P2, 70%+ draft acceptance rate, zero major incident misses, zero unauthorized actions

## Implementation Notes

- **Triage disposition is intentionally not automated** — major incident declaration, containment authorization, and final disposition carry operational accountability that cannot be delegated to AI
- **No skill executes remediation actions** — this is a permanent architectural constraint; production changes, workaround deployment, and service restarts are always human-authorized
- **The ITSM platform is the source of truth** — the Excel tracker is a Cowork-accessible working copy; state drift between the tracker and the ITSM platform is mitigated by Power Automate sync on a 15-minute cycle
- **Major incident controls are the strictest guardrails** — P1/P2 severity never auto-set, major incident bridge never auto-activated, containment never auto-authorized
- **The 10-minute SLA** drives urgency for standard incident classification and routing — every skill surfaces SLA status
- **Recent changes are flagged but never blamed** — changes within 48 hours of an incident are flagged as potential contributing factors, but causation is for the resolver group to determine
- **Security-classified incidents receive restricted routing** — security category incidents route to Security Operations with suppressed general channel visibility
- **Audience language rules are enforced across all communications** — internal system names, queue names, and infrastructure details never appear in user-facing or executive communications
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., enriching an existing incident, reclassifying after new symptoms, or drafting a status update without re-running the full workflow)
- **Knowledge article recommendations must come from confirmed articles** — unverified workarounds are never included in user communications as they may cause additional damage
