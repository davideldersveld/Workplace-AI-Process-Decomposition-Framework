# IT Security Alert Triage and Investigation Prep — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Security Alert Triage and Investigation Prep** workflow as a set of six Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the security operations triage process — from alert intake through review-ready case disposition — into AI-assisted capabilities within Microsoft 365.

The workflow supports alerts from SIEM, endpoint, identity, email security, and analyst intake channels, guiding each through structured normalization, entity and threat context enrichment, risk classification, severity assessment, analyst routing, and investigation summary drafting with strict containment prohibition, IOC exposure controls, insider threat handling restrictions, evidence integrity requirements, audience-scoped sensitivity controls, and examination-grade audit traceability at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **sec-alert-intake** | Normalizes inbound security alerts into structured case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **sec-enrichment-packet** | Assembles entity, asset, threat context, related alerts, and playbook references into an enrichment packet | AI act within policy | analysis | SearchSparkle |
| 3 | **sec-risk-classifier** | Classifies alert type and likely risk, recommends threat path | AI assist | analysis | Tag |
| 4 | **sec-severity-recommend** | Assesses severity, blast radius, urgency, and investigation path | AI draft + approve | analysis | Flag |
| 5 | **sec-analyst-router** | Routes cases to analyst queues per SOC routing rules | AI act within policy | communication | Mail |
| 6 | **sec-investigation-drafter** | Drafts investigation summaries, evidence requests, escalation briefs, and executive notifications | AI draft + approve | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Alert Intake     │  Normalize alert telemetry → structured case record
│     (sec-alert-      │  Validates required fields, checks duplicates/correlations
│      intake)         │  SLA: 15 minutes for high-priority triage
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Enrichment       │  Assemble entity profile, asset criticality,
│     Packet           │  related alerts (30 days), prior investigation
│     (sec-enrichment- │  history, applicable playbook reference,
│      packet)         │  threat context summary
│                      │  Output: Word enrichment packet (Confidential)
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Risk Classifier  │  Compare alert telemetry against detection
│     (sec-risk-       │  taxonomy, determine alert category,
│      classifier)     │  threat path, confidence level
│                      │  Output: Adaptive Card classification report
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Severity         │  Assess severity, blast radius, urgency,
│     Recommend        │  investigation path, escalation needs,
│     (sec-severity-   │  containment considerations (info only),
│      recommend)      │  campaign correlation
│                      │  Output: Adaptive Card severity recommendation
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Analyst Router   │  Route to Tier 1, Tier 2, incident response,
│     (sec-analyst-    │  identity security, insider threat, or SOC lead
│      router)         │  per routing rules; enforce authorization checks
│                      │  Output: Teams notifications + tracker update
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Investigation    │  Draft analyst handoff summaries, evidence
│     Drafter          │  preservation requests, escalation briefs,
│     (sec-            │  executive notifications, follow-up requests
│      investigation-  │  Output: Word docs + Outlook drafts + Teams
│      drafter)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  7. Confirm Triage   │  Confirm case is queued, escalated, or
│     Disposition      │  closed as benign. Human-only step —
│     (not automated)  │  incident declaration, containment, closure.
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Alert Intake | Structured field extraction, validation, duplicate checking — no AI judgment on security decisions |
| **AI act within policy** | Enrichment Packet, Analyst Router | Retrieves approved context from defined sources or executes routing within pre-approved rules; does not interpret threats or make severity assessments |
| **AI assist** | Risk Classifier | Surfaces classification recommendations with confidence levels; analyst reviews before any action |
| **AI draft + approve** | Severity Recommend, Investigation Drafter | AI recommends severity or drafts communications; analyst or SOC lead reviews and confirms before any action |
| **Human only** | Confirm Disposition | Incident declaration, containment authorization, case closure, and executive escalation decisions are always human-owned |

## Governance Controls

### Containment and Response Action Prohibition

- No skill may execute containment actions (block user, isolate host, disable account, revoke credentials)
- No skill may declare an incident or activate incident response procedures
- No skill may close a case as false positive or benign without analyst confirmation
- No skill may modify evidence artifacts or existing investigation documents
- Containment options are presented as information from the playbook only, with explicit notation that they require human authorization
- Final triage disposition decisions carry security accountability and remain human-only

### IOC and Data Sensitivity Controls

- Raw IOCs (IP addresses, hashes, domain names) never appear in Adaptive Card confirmations or Teams channel posts — reference by Case ID only
- Affected account names and hostnames never appear in SOC channel posts or executive communications
- Investigation hypotheses and attribution speculation never appear in any written artifact
- Raw credentials, tokens, and decrypted payloads never appear in any skill output
- PII and employee identifiers are redacted from executive-facing summaries unless explicitly confirmed
- All enrichment packet and investigation summary documents carry the sensitivity label: Confidential — Security Operations

### Audience-Scoped Sensitivity Matrix

| Audience | IOCs | Account Names | Hostnames | Classification | Investigation Details |
|----------|------|---------------|-----------|----------------|----------------------|
| Investigating analyst | Allowed (secure document) | Allowed | Allowed | Allowed | Allowed |
| SOC lead / IR team | Allowed | Allowed | Allowed | Allowed | Allowed |
| IT operations / data custodian | Not allowed | Not allowed | Allowed (evidence scope) | Not allowed | Not allowed |
| Executive leadership | Not allowed | Not allowed | Not allowed | Summary only | Not allowed |
| Platform / network teams | Not allowed | Not allowed | Allowed (log scope) | Not allowed | Not allowed |

### False-Negative Safety

- No skill may auto-classify an alert as false positive or benign — false-negative risk is the primary safety concern
- Low-confidence classifications are explicitly flagged for senior analyst review
- Active intrusion and data exfiltration classifications trigger urgent escalation banners
- Severity can never be auto-downgraded — new evidence suggesting lower severity requires analyst review

### Insider Threat Controls

- Cases flagged as potential insider threat receive restricted routing — only designated insider threat analysts can view full details
- General SOC channel visibility is suppressed for insider threat cases
- Prior insider threat investigation history is referenced without disclosing investigation details
- Insider threat classification triggers automatic routing restriction in downstream skills

### Evidence Integrity

- Investigation artifacts stored in SharePoint with sensitivity labels and version history
- Skills never modify existing evidence documents — only create new versions
- Every data point in enrichment packets cites source system and retrieval timestamp
- Missing or unavailable enrichment sources are flagged explicitly
- Cases with legal hold advisory prevent modification of evidence

### Audit Trail

- Every case record includes actor, timestamp, alert source, and reporting analyst for creation audit
- Every classification decision records recommended category, confidence, and analyst final decision
- Every severity assessment records recommended severity, analyst final decision, and override rationale if applicable
- Every routing decision records assigned analyst, queue, rationale, and confirming reviewer
- Every communication records type, recipient, Case ID, and timestamp
- The Excel alert tracker serves as both operational state and examination-grade audit record

## Alert Categories

| Alert Category | Description | Typical Threat Path |
|----------------|-------------|---------------------|
| **Phishing** | Email-based social engineering, credential harvesting, malicious attachments | Credential theft, malware delivery, BEC |
| **Malware** | Malicious software detection on endpoint, server, or network | Ransomware, trojan, cryptominer, worm |
| **Identity compromise** | Unauthorized access, credential misuse, account takeover | Brute force, stolen credentials, MFA bypass |
| **Data exfiltration** | Unusual data transfer, unauthorized file access, data staging | Insider threat, external attacker data theft |
| **Policy violation** | Violation of security policy, unauthorized software, shadow IT | Non-compliant behavior, unapproved access |
| **Lateral movement** | Internal network traversal, privilege escalation, service abuse | Post-compromise activity, APT progression |
| **Denial of service** | Service disruption, resource exhaustion, availability impact | DDoS, application-layer attacks |
| **Insider threat** | Suspicious activity by an authorized user, data hoarding, access anomalies | Malicious insider, compromised insider |
| **Configuration drift** | Security control misconfiguration, policy deviation, exposure | Unintentional exposure, compliance gap |

## Severity Levels

| Severity | Criteria | SLA Target |
|----------|----------|------------|
| **Critical** | Active intrusion confirmed or likely; data exfiltration in progress; ransomware execution; privileged account compromise on production systems; correlated campaign indicators | 15 min triage, immediate IR activation |
| **High** | Credential compromise with misuse evidence; malware on production/critical systems; data staging; privileged identity anomaly; confirmed credential submission to phishing | 15 min triage, 1 hr initial investigation |
| **Medium** | Endpoint malware on non-critical systems; policy violation with security implications; phishing without confirmed compromise; identity anomaly without confirmed misuse | 1 hr triage, 4 hr initial investigation |
| **Low** | Known false-positive patterns; compliance drift without active threat; informational alerts; blocked attempts with no success evidence | 4 hr triage, next business day investigation |

## Routing Queues

| Queue | Handles | Typical Cases |
|-------|---------|---------------|
| **Tier 1 — SOC Analyst** | Standard triage, monitoring, low-complexity | Policy violations, blocked phishing, informational alerts |
| **Tier 2 — Senior SOC Analyst** | Deep investigation, complex triage, multi-entity | Credential compromise, malware analysis, data staging |
| **Incident Response Team** | Active incidents, confirmed breaches, critical severity | Active intrusion, ransomware, data exfiltration |
| **Identity Security Team** | Identity-focused alerts, privilege escalation | Account takeover, MFA bypass, privileged identity compromise |
| **Insider Threat Team** | Cases flagged as potential insider threat | Data hoarding, unauthorized access patterns, policy evasion |
| **SOC Lead** | Escalation when no clear routing rule matches | Ambiguous cases, routing conflicts, capacity overflow |

## Communication Types

| Communication | Audience | Channel | Sensitivity |
|---------------|----------|---------|-------------|
| Analyst handoff summary | Investigating analyst | Word doc + Teams DM | Full technical detail |
| Evidence preservation request | IT ops / data custodian | Outlook draft | No classification, IOCs, or investigation details |
| Escalation brief | SOC lead / IR team | Word doc + Teams DM | Full security context |
| Executive notification | CISO / leadership | Outlook draft | No IOCs, hostnames, account names, or hypotheses |
| Follow-up telemetry request | Platform / network teams | Outlook draft or Teams DM | No investigation hypotheses |

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find alert tracker, detection taxonomy, severity matrix, playbooks, asset data, templates, prior cases |
| SearchM365 (connectors) | Enrichment Packet, Risk Classifier, Severity Recommend — retrieve alert data, endpoint context, identity anomalies from Sentinel and Defender via Graph Connector |
| SearchM365 (email) | Enrichment Packet — find emails related to phishing or email-based alerts (permission-scoped) |
| SearchM365 (teams) | Enrichment Packet — find SOC channel discussions about affected entities |
| ReadFileContent | All skills — read tracker, taxonomy, matrices, playbooks, enrichment packets, templates |
| GetDriveChildren | Alert Intake — check existing tracker for duplicates |
| SearchPeople / GetUserDetails | All skills — resolve affected users, analysts, SOC leads |
| GetManagerDetails | Enrichment Packet — reporting chain for escalation context |
| ListCalendarView | Analyst Router — check analyst availability |
| CreateDraftMessage | Investigation Drafter — Outlook drafts for evidence requests and executive notifications |
| PostMessage | Analyst Router, Investigation Drafter — Teams direct messages to assigned analysts |
| PostChannelMessage | Analyst Router — SOC coordination channel assignment posts |
| CreateEvent | Analyst Router — SLA deadline calendar holds |
| render_ui (Adaptive Card) | Alert Intake, Risk Classifier, Severity Recommend — confirmations, classifications, severity recommendations |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Alert case record | Excel tracker row | Alert Intake |
| Enrichment packet | Word document (Confidential) | Enrichment Packet |
| Classification report | Adaptive Card | Risk Classifier |
| Severity recommendation | Adaptive Card | Severity Recommend |
| Routing recommendation | Adaptive Card | Analyst Router |
| Analyst assignment notifications | Teams direct messages + channel posts | Analyst Router |
| SLA deadline calendar holds | Calendar events | Analyst Router |
| Investigation summary | Word document (Confidential) | Investigation Drafter |
| Evidence preservation request | Outlook draft | Investigation Drafter |
| Escalation brief | Word document + Teams DM | Investigation Drafter |
| Executive notification | Outlook draft | Investigation Drafter |
| Follow-up telemetry request | Outlook draft or Teams DM | Investigation Drafter |

## Federated Data Access

Security alert triage depends on specialized platforms (SIEM, SOAR, endpoint, identity, threat intel). The skills use a tiered access approach:

| Tier | Method | Use Case |
|------|--------|----------|
| **Tier 1** | Graph Connectors | Microsoft Sentinel alert data, incident correlations, detection rule metadata; Microsoft Defender endpoint context, device health, detection events indexed into M365 Search |
| **Tier 2** | SharePoint Bridge | Detection taxonomy, severity matrix, playbook library, analyst routing rules, asset inventory, service criticality, threat intelligence briefs, and investigation templates maintained in SharePoint via Power Automate sync |
| **Tier 3** | Manual Input | SIEM console alert details, raw telemetry, and data points not available through integration, captured via structured prompts with "manual entry" source tagging |

**Recommended pilot approach:** Start with Tier 2 (SharePoint sync for asset inventory, playbooks, severity matrices, and routing rules) and Tier 3 for SIEM alert details from manual lookups. Introduce Graph Connectors for Microsoft Sentinel and Defender in Wave 2 after skill workflows are proven and connector availability is confirmed.

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── sec-alert-intake/SKILL.md
├── sec-enrichment-packet/SKILL.md
├── sec-risk-classifier/SKILL.md
├── sec-severity-recommend/SKILL.md
├── sec-analyst-router/SKILL.md
└── sec-investigation-drafter/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Alert tracker** — shared Excel workbook (Case ID, Alert Source, Detection Rule, Affected Entity, Entity Type, Alert Category, Severity, Status, Created Date, Assigned Analyst, Investigation Path, SLA Target, Disposition)
- **Detection taxonomy** — alert category definitions, threat path descriptions, classification criteria
- **Severity matrix** — severity level definitions, criteria, thresholds, SLA targets
- **Playbook library** — detection playbooks, response procedures, investigation steps, containment options
- **Analyst routing rules** — queue structure, specialization mapping, on-call schedule, escalation paths
- **SOC operating model** — team structure, authorization levels, incident response procedures
- **Asset inventory** — asset criticality ratings, environment classification, service ownership, data classification
- **Escalation policy** — notification triggers by severity level, CISO and legal notification criteria
- **Communication templates** — investigation summary, evidence request, escalation brief, executive notification, follow-up request templates
- **Threat intelligence briefs** — curated threat intel summaries searchable by detection rule or attack technique

### Trigger Phrases

| Skill | Example Triggers |
|-------|-----------------|
| Alert Intake | "new security alert from [source]", "log security case for [entity]", "intake this alert", "create security case" |
| Enrichment Packet | "enrich this alert", "build context for security case [ID]", "what do we know about this entity", "gather threat context" |
| Risk Classifier | "classify this alert", "what type of threat is this", "risk assessment for this alert", "categorize this security event" |
| Severity Recommend | "assess severity for this alert", "what priority should this case be", "investigation path for this alert", "severity recommendation" |
| Analyst Router | "route this case", "assign analyst for this alert", "who handles this type of alert", "queue this for investigation" |
| Investigation Drafter | "draft investigation summary", "prepare case handoff", "write evidence request", "create analyst briefing", "executive notification" |

## Implementation Roadmap

### Wave 1 — Foundation and Read-Only Assist

- Create the shared Excel alert tracker workbook in SharePoint with standard columns
- Upload detection taxonomy, severity matrix, playbook library, and asset inventory
- Configure SharePoint sensitivity labels on the security cases folder (Confidential — Security Operations)
- Set up the SOC coordination Teams channel with restricted membership
- Create Power Automate flows for asset inventory and service criticality sync
- Build skills: `sec-alert-intake`, `sec-enrichment-packet`, `sec-risk-classifier`, `sec-investigation-drafter`
- Operate in AI assist mode — all outputs presented for analyst review
- Test with 30–50 real alerts over 3 weeks with dedicated sensitivity compliance audit

### Wave 2 — Routing and Severity

- Build skills: `sec-severity-recommend`, `sec-analyst-router`
- Promote intake to write mode (creates tracker records after confirmation)
- Promote classifier to write-back mode (updates tracker category after analyst confirmation)
- Promote router to "act within policy" for Medium and Low severity with clear rule matches
- Set up scheduled prompt: check for new unprocessed alerts in SOC channel every 10 minutes
- Set up daily scheduled prompt: flag cases approaching SLA breach and stale cases (no update in 24 hours)
- Introduce Graph Connectors for Microsoft Sentinel and Defender if available

### Wave 3 — Optimization and Proactive Detection

- Add playbook-cited investigation guidance in enrichment packets
- Add campaign correlation: cross-reference new alerts against recent high-severity cases
- Add proactive monitoring: identify repeat alert clusters, flag potential false-negative patterns, surface analyst queue imbalances
- Introduce additional Graph Connectors for third-party SIEM or endpoint platforms if available
- Measurement targets: 75%+ classification agreement rate, 85%+ correct routing rate, under 15-minute triage for high-priority alerts, 95%+ SLA compliance for Critical/High, 65%+ draft acceptance rate, zero false-negative escapes, zero sensitivity violations, zero unauthorized actions

## Implementation Notes

- **Triage disposition is intentionally not automated** — incident declaration, containment authorization, and case closure decisions carry security accountability that cannot be delegated to AI
- **No skill executes containment actions** — this is a permanent architectural constraint; blocking users, isolating hosts, disabling accounts, and revoking credentials are always human-authorized through the SIEM/SOAR platform
- **False-negative safety is the primary design concern** — no skill may auto-close alerts as benign or false positive; every classification recommendation is reviewable, and low-confidence results are explicitly flagged
- **IOC exposure controls are security requirements** — raw IOCs, account names, and investigation hypotheses in unauthorized channels represent real security risk, not just operational preferences
- **The 15-minute SLA** drives urgency for high-priority alerts — every skill surfaces SLA status in its output
- **Insider threat cases receive restricted handling** throughout the workflow — routing, communication, and channel visibility are all scoped to designated insider threat analysts only
- **Evidence integrity is an investigation requirement** — all investigation artifacts are stored in SharePoint with sensitivity labels and version history; skills never modify existing evidence documents
- **Guardrails are defense-in-depth** — encoded in skill instructions, reinforced by M365 sensitivity labels, restricted by Teams channel membership, and monitored via pilot sensitivity audits
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., enriching an existing case, reclassifying after new evidence, or drafting an escalation brief without re-running the full workflow)
