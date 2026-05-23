# Plan: IT Security — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the IT Security line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the Security Alert Triage and Investigation Prep pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select, Decompose, Signal Inventory, Automation Boundary, Capability Map, Translate, Architect). To evaluate how this maps to Copilot Cowork skill ideation for IT Security, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly; IT Security's low error tolerance for false negatives adds weight to the evaluation |
| **Process Decomposition** | Step records with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule is especially important in security where classification, severity, and routing must be independently reviewable |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires significant translation — the framework references "SIEM," "SOAR," "endpoint platform," and "threat intel sources" that must become Graph Connector references or SharePoint bridge data |
| **Automation Boundary** | Operating mode per step (human-only to deterministic) | **Guardrails and confirmation gates** in SKILL.md | Critical — security operations demand the strictest guardrail posture of any LOB; containment, incident declaration, and evidence handling require explicit human ownership with no exceptions |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — alert enrichment maps to Data Aggregation; risk classification maps to Decision Support; investigation summaries map to Content Generation |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but constrained — security data sensitivity means many tool calls must be permission-scoped, and outputs must never leak sensitive indicators to unauthorized channels |
| **Reference Architecture** | 9-layer runtime | **Cowork's built-in runtime** | Absorbed — but the governance and control layer (layer 8) requires extra attention for security data classification and access controls |

### Key Insight

IT Security has the most restrictive guardrail requirements of any LOB in the framework. The decomposition discipline is valuable not just for preventing mega-skills, but for ensuring that each skill's data access and output scope is precisely bounded. A classification skill should not have access to containment actions; an enrichment skill should not be able to post investigation details to general Teams channels. The framework's automation boundary assignment is not just a design guide for security — it is a safety requirement. Every skill must be designed with the assumption that security telemetry, IOCs (indicators of compromise), and investigation details are sensitive by default and must never leak beyond authorized channels.

---

## Part 2: The IT Security Plugin — Skill-by-Skill Design

The IT Security sample decomposes "Security Alert Triage and Investigation Prep" into 7 steps (SEC-001 through SEC-007), identifies 6 skills and 8 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| SEC-001: Normalize security alert | `sec-alert-intake` | Data Aggregation | Deterministic automation | Excel (alert tracker), SharePoint (list), Teams (SOC channel) |
| SEC-002: Gather entity, asset, and threat context | `sec-enrichment-packet` | Data Aggregation | AI act within policy | Excel (enrichment data), SharePoint (asset context), Word (enrichment packet) |
| SEC-003: Classify alert type and likely risk | `sec-risk-classifier` | Decision Support | AI assist | Adaptive Card (risk classification), Excel (tracker update) |
| SEC-004: Assess severity and investigation path | `sec-severity-recommend` | Decision Support | AI draft + approve | Adaptive Card (severity recommendation), Excel (tracker update) |
| SEC-005: Assign owner and next action | `sec-analyst-router` | Decision Support | AI act within policy | Teams (SOC messages), Excel (tracker update) |
| SEC-006: Draft investigation summary and follow-up requests | `sec-investigation-drafter` | Content Generation | AI draft + approve | Word (case summary), Outlook (evidence requests), Teams (SOC handoff) |
| SEC-007: Confirm triage disposition | *Not a skill — human approval step* | N/A | Human only | Teams (disposition confirmation), Calendar (review) |

### Detailed Skill Designs

#### 1. `sec-alert-intake` — Normalize Security Alert

**Framework Step:** SEC-001

**Trigger phrases:** "new security alert from", "log security case for", "intake this alert", "normalize this security event", "create security case"

**Inputs:**
- Alert source (SIEM, endpoint, identity, email security, analyst report)
- Alert description or raw telemetry summary
- Affected entity (user, host, IP, application)
- Detection rule or signature name

**M365 tools:**
- `SearchPeople` — resolve affected user identity if the alert involves a named user
- `GetUserDetails` — pull affected user profile, department, role
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — pull original alert details from Microsoft Sentinel or SIEM via Graph Connector (if available)
- `ReadFileContent` — read SharePoint-hosted alert intake template for field normalization
- `GetDriveChildren` — check existing alert tracker for duplicate or correlated alerts within the same time window

**Output:** Structured alert case record written to Excel alert tracker in SharePoint; confirmation via Adaptive Card showing normalized fields

**Artifact:** Excel workbook with columns: Case ID, Alert Source, Detection Rule, Affected Entity, Entity Type, Alert Category (pending), Severity (pending), Status, Created Date, Assigned Analyst (pending), Investigation Path (pending), SLA Target

**Guardrails:**
- Never create duplicate cases for the same alert signature and entity within a 1-hour window; instead, flag as correlated and link to the existing case
- Validate that required fields (alert source, affected entity, detection rule) are present before writing
- Auto-generate Case ID using date and source prefix (e.g., SEC-20260523-SIEM-001)
- Confirm details with analyst via Adaptive Card before writing to tracker
- Never include raw IOCs (IP addresses, hashes, domain names) in the Adaptive Card confirmation — only reference them by alert ID
- Scheduled prompt variant: monitor the SOC Teams channel for unprocessed alert reports every 10 minutes

---

#### 2. `sec-enrichment-packet` — Gather Entity, Asset, and Threat Context

**Framework Step:** SEC-002

**Trigger phrases:** "enrich this alert", "build context for security case", "what do we know about this entity", "gather threat context for"

**Inputs:**
- Case ID or affected entity from the alert record

**M365 tools:**
- `GetUserDetails` — affected user profile, department, role, location
- `GetManagerDetails` — reporting chain for escalation context
- `SearchM365(sources=["files"])` — asset criticality documents, environment classification guides, prior investigation reports in SharePoint
- `ReadFileContent` — SharePoint-hosted asset inventory, service criticality ratings, identity role definitions, playbook references
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — related alerts for the same entity in the last 30 days
- `SearchM365(sources=["connectors"], connector_ids=["defender-connector"])` — endpoint context, device health, recent detections on the affected host
- `SearchM365(sources=["email"])` — recent emails from or to the affected user if the alert involves phishing or email-based threats (permission-scoped)
- `SearchM365(sources=["teams"])` — recent SOC channel discussions mentioning the affected entity or detection rule

**Output:** Word document containing:
- Affected entity profile (user, host, or application)
- Asset criticality and environment classification
- Related alerts from the last 30 days
- Prior investigation history for this entity
- Applicable playbook reference
- Threat context summary (detection rule description, known attack patterns)

**Artifact:** Word (.docx) saved to SharePoint security cases folder; linked from the Excel tracker row. The document is classified as internal/confidential.

**Guardrails:**
- Only retrieve context for entities and assets the requesting analyst has permission to view
- Never include raw credentials, tokens, or decrypted payloads in the enrichment packet
- Cite source system and retrieval timestamp for every data point
- Flag if any critical enrichment source (SIEM, endpoint, identity) returned no data or is unavailable
- Mark the output document with a sensitivity label: "Confidential - Security Operations"
- Never post enrichment packet contents to general Teams channels — only to the designated SOC channel or direct messages to authorized analysts
- If the alert involves executive accounts or privileged identities, flag for elevated handling and restrict distribution

---

#### 3. `sec-risk-classifier` — Classify Alert Type and Likely Risk

**Framework Step:** SEC-003

**Trigger phrases:** "classify this alert", "what type of threat is this", "risk assessment for this alert", "categorize this security event"

**Inputs:**
- Enrichment packet (Word document or data from Excel tracker)
- Detection taxonomy (SharePoint document)
- Playbook index (SharePoint document)

**M365 tools:**
- `ReadFileContent` — detection taxonomy, threat classification matrix, and playbook index from SharePoint
- `SearchM365(sources=["files"])` — similar past cases and their final classifications
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — historical alert classification patterns and false-positive rates for this detection rule

**Output:** Risk classification recommendation as Adaptive Card showing:
- Recommended alert category (phishing, malware, identity compromise, data exfiltration, policy violation, lateral movement, etc.)
- Likely threat path description
- Confidence level (high, medium, low) with basis
- Historical false-positive rate for this detection rule
- Applicable playbook reference
- Top 3 similar past cases with their outcomes

**Logic:** Compare alert telemetry and enrichment context against the detection taxonomy. Cross-reference with historical cases for this detection rule. Factor in the entity's risk profile (privileged user, critical asset, internet-facing system). Present ranked classification candidates.

**Guardrails:**
- Present classification as a recommendation only — never auto-update the alert category without analyst confirmation
- Always show confidence level and the reasoning chain
- If confidence is low (below 50%), explicitly flag for senior analyst review
- Never classify an alert as "false positive" or "benign" without analyst confirmation — false-negative risk is the primary safety concern
- Do not include specific IOC values in the Adaptive Card; reference them by alert ID and enrichment packet section
- If the classification suggests an active intrusion or data exfiltration, add an urgent escalation banner to the Adaptive Card

---

#### 4. `sec-severity-recommend` — Assess Severity and Investigation Path

**Framework Step:** SEC-004

**Trigger phrases:** "assess severity for this alert", "what priority should this case be", "investigation path for this alert", "severity recommendation for security case"

**Inputs:**
- Classification output
- Enrichment packet
- Severity matrix (SharePoint document)
- Environment context (asset criticality, data classification)

**M365 tools:**
- `ReadFileContent` — severity matrix, escalation policy, investigation SLA targets, containment playbooks from SharePoint
- `SearchM365(sources=["files"])` — asset criticality ratings, data classification records, business impact assessments
- `GetUserDetails` — affected entity's role, privilege level, and organizational position
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — current active incidents that might be related; recent high-severity cases in the same environment

**Output:** Severity recommendation as Adaptive Card showing:
- Recommended severity level (Critical, High, Medium, Low) with rationale
- Blast radius assessment (affected users, systems, data types)
- Urgency assessment (active threat vs. historical detection, dwell time indicators)
- Investigation path recommendation (standard triage, deep investigation, incident response, containment required)
- Escalation recommendation (SOC lead, incident response team, CISO notification)
- SLA target for the recommended severity
- Containment considerations (if applicable — presented as information only, never auto-executed)

**Guardrails:**
- Never auto-set severity — always present as a recommendation for analyst review
- If the recommendation is Critical or High, add an explicit incident declaration advisory with instructions for the analyst
- Require the analyst to confirm or override before severity is written to the tracker
- Log the recommended severity and the analyst's final decision for override tracking and model tuning
- If the alert involves privileged accounts, production systems, or regulated data (PCI, HIPAA, PII), auto-escalate the visibility to the SOC lead regardless of calculated severity
- Never recommend containment actions (block user, isolate host, disable account) — only describe what containment options exist in the playbook and note that they require human authorization
- Cross-reference against open incidents to identify potential campaign correlation

---

#### 5. `sec-analyst-router` — Assign Owner and Next Action

**Framework Step:** SEC-005

**Trigger phrases:** "route this case", "assign analyst for this alert", "who handles this type of alert", "queue this for investigation"

**Inputs:**
- Classification and severity output
- Analyst routing rules (SharePoint document)
- SOC operating model and queue structure (SharePoint document)

**M365 tools:**
- `ReadFileContent` — analyst routing rules, SOC operating model, queue structure, on-call schedule from SharePoint
- `SearchPeople` — resolve analyst queue leads and on-call contacts
- `GetUserDetails` — verify assigned analyst availability and current role
- `PostMessage` — Teams notification to the assigned analyst with case summary
- `PostChannelMessage` — post assignment summary to the SOC coordination channel
- `ListCalendarView` — check analyst availability before assignment

**Output:** Routed security case:
- Teams direct message to assigned analyst with case summary (excluding raw IOCs — reference by case ID)
- Channel post in the SOC coordination channel for visibility
- Updated Excel tracker with assigned analyst, queue, and SLA target

**Guardrails:**
- Only assign to analysts listed in the approved SOC routing rules
- If no matching routing rule exists, escalate to the SOC lead rather than guessing
- For Critical or High severity cases, verify that the assigned analyst has incident response authorization before routing
- Present assignment recommendation for analyst review before sending Teams messages (standard severity cases may proceed under "act within policy" mode)
- Never route a case marked as potential incident to a Tier 1 analyst — these must go to Tier 2 or incident response
- Include the case ID and severity in every outbound Teams message, but never include raw IOCs, affected account names, or hostnames in channel posts — only in direct messages to the assigned analyst
- If the case involves a potential insider threat, route only to designated insider threat analysts and suppress general SOC channel visibility

---

#### 6. `sec-investigation-drafter` — Draft Investigation Summary and Follow-Up Requests

**Framework Step:** SEC-006

**Trigger phrases:** "draft investigation summary", "prepare case handoff", "write evidence request for", "summarize this security case", "create analyst briefing for"

**Inputs:**
- Alert record from Excel tracker
- Enrichment packet (Word document)
- Classification and severity data
- Assignment details
- Target audience (investigating analyst, SOC lead, incident response team, evidence custodian)

**M365 tools:**
- `ReadFileContent` — investigation summary templates, evidence request templates from SharePoint
- `CreateDraftMessage` — Outlook drafts for evidence requests to external teams (never auto-send)
- `PostMessage` — Teams coordination messages to SOC analysts (direct messages only for sensitive details)
- `SearchM365(sources=["files"])` — prior investigation summaries for similar cases as exemplars

**Communication templates:**
- Analyst handoff summary (case context, classification, severity, recommended next steps)
- Evidence preservation request (to IT, identity team, or data custodian)
- Escalation brief for SOC lead or incident response team
- Executive notification draft (for Critical severity only — highly summarized, no technical IOCs)
- Follow-up request for additional telemetry or log access

**Output options:**
- **Word document** — Formal investigation summary for case file
- **Adaptive Card** — Quick case status for SOC channel review
- **Outlook draft** — Evidence request or follow-up email

**Guardrails:**
- Always create external-facing communications as Outlook drafts — never send without explicit analyst confirmation
- Match detail level to audience: full technical detail for investigating analysts, operational summary for SOC leads, executive summary (no IOCs, no hostnames, no account names) for leadership
- Never include raw IOCs, IP addresses, hashes, domain names, or affected account credentials in any communication to channels outside the SOC
- Never include investigation hypotheses or attribution speculation in any written artifact — only state confirmed findings and open questions
- Redact all PII, employee names, and account identifiers from executive-facing summaries unless the analyst explicitly confirms inclusion
- For cases involving potential data breach, add a legal hold advisory to the investigation summary
- Include case ID and reference links in every communication for traceability
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable before delivery

---

#### Step 7: Confirm Triage Disposition (Human Only)

**Framework Step:** SEC-007

This is not a Cowork skill. The framework correctly identifies final triage disposition as a human-only step, especially for incident declaration, containment authorization, and case closure decisions. In Cowork, it is supported by:

- The severity recommendation Adaptive Card from `sec-severity-recommend` — provides the evidence for the disposition decision
- The investigation summary from `sec-investigation-drafter` — provides the case file for SOC lead review
- The `sec-analyst-router` assignment — confirms the case is in the right queue
- Calendar events for SOC review checkpoints

For cases requiring incident declaration, the human disposition step includes: confirming the incident classification, authorizing containment actions (block user, isolate host, revoke credentials), initiating the incident response process, and determining notification obligations (legal, compliance, executive, regulatory). These are consequential security decisions that must remain human-owned with full accountability.

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. For IT Security, this challenge is compounded by the sensitivity of security data, the dependency on specialized security platforms (SIEM, SOAR, endpoint), and the strict access controls required for investigation artifacts. This section analyzes how to approach that translation systematically.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors in IT Security is **pre-implementation discipline with safety implications**:

**Process decomposition prevents dangerous mega-skills.** In security, a single "triage this alert" skill that classifies, assesses severity, routes, and drafts communications in one invocation creates an unacceptable blast radius. If the skill makes an error in classification, it cascades through severity, routing, and communication. The framework's decomposition ensures each decision is independently reviewable.

**Automation boundary assignment is a safety requirement, not a design preference.** The five operating modes translate with heightened strictness:

| Framework Mode | Cowork Implementation for Security |
|---|---|
| Human only | Do not build a skill; support disposition, containment, and incident declaration with evidence artifacts only |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions; analyst must explicitly act on every recommendation (used for risk classification) |
| AI draft + approve | Skill produces recommendations and drafts but uses `CreateDraftMessage`, shows all output before any write; requires explicit confirmation with logged rationale (used for severity and investigation summaries) |
| AI act within policy | Skill can execute bounded actions (update tracker, route to analyst queue) only within pre-approved rules and only for standard-severity cases |
| Deterministic automation | Scheduled prompt for stable alert intake normalization only; no model judgment on security decisions |

**Signal inventory forces data access scoping.** The framework requires naming every input source, which in security translates to explicit data access boundaries. A classification skill should not have access to raw endpoint telemetry; an enrichment skill should not be able to read unrelated users' email.

### 3.2 What Cowork Provides That the Framework Assumes You Build

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are accessible via MCP tools; security telemetry requires connector bridging |
| Process model | Implicit — the skill's trigger phrases and instructions define which triage phase is active |
| Capability registry | Built-in — `/mnt/user-config/.claude/skills/` IS the registry |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection and execution |
| Knowledge and context assembly | Built-in for M365 data; security-specific data (SIEM, endpoint, threat intel) requires Graph Connectors or SharePoint bridge |
| Memory and state | Partial — session memory persists; durable case state needs M365 artifacts |
| Decision and approval plane | Partial — draft tools and Adaptive Cards provide human-in-the-loop; no formal incident response workflow engine |
| Governance and control | Partial — skill instructions encode policies; security classification and access scoping require extra discipline |
| Evaluation and observability | Limited — no built-in skill-level metrics; security evaluation requires false-negative tracking |

**The key gap for IT Security:** Cowork does not natively enforce data classification labels or access-scoped tool calls. If a skill can call `SearchM365`, it can potentially access any data the user has permission to view. The guardrails must be encoded in skill instructions (e.g., "never search for emails from users not named in the alert") and reinforced by M365 sensitivity labels on SharePoint documents. This is a softer boundary than a purpose-built security platform would provide.

### 3.3 M365 Artifacts as Process State

For IT Security, the primary M365 artifacts are **Excel** and **SharePoint**, reflecting the evidence-heavy, document-centric nature of security investigations.

| Artifact | Role in IT Security | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (alert tracker, case status, severity log, SLA dashboard) | The alert tracker workbook IS the Cowork-accessible case state — skills read current status, write updates, track SLA compliance, and log classification and severity decisions |
| **SharePoint** | Source of truth for policies and evidence (detection taxonomy, severity matrix, playbooks, asset inventory, investigation reports, enrichment data) | Skills read classification rules, severity policies, and playbook references from SharePoint; investigation artifacts are stored here with sensitivity labels |
| **Word** | Evidence artifacts (enrichment packets, investigation summaries, case reports) | Skills generate case documents that become the auditable investigation record; these are the primary output for analyst review and handoff |
| **Teams** | SOC coordination channel (analyst routing, case updates, escalation notices) | Skills post case assignments and status updates to the SOC channel; direct messages for sensitive case details; channel posts use case IDs only, never raw IOCs |
| **Outlook** | Communication channel for evidence requests and external notifications | Skills draft evidence preservation requests, follow-up queries, and executive notifications as reviewable Outlook drafts |
| **Graph API** | People and org data (affected user profiles, reporting chains, analyst directories) | Skills resolve affected entities, managers, and analyst queue leads for enrichment and routing |
| **Calendar** | SLA deadline reminders and SOC review checkpoints | Skills create calendar events for investigation SLA warnings and case review meetings |
| **Adaptive Card** | In-flow decision support (risk classification, severity assessment, case status) | Skills present triage recommendations as structured cards for quick analyst review; sensitive details are omitted from cards in favor of case ID references |

**The design pattern:** The IT Security plugin uses a SharePoint-hosted Excel workbook as the Cowork-accessible alert tracker, with SharePoint document libraries as the evidence repository. Word documents serve as the primary investigation artifacts. Teams is the coordination channel, but its use is carefully scoped — raw IOCs, account names, and investigation hypotheses never appear in channel posts, only in direct messages to authorized analysts or in the SharePoint-stored case documents. This is a security-by-design pattern where every artifact's distribution scope is explicitly controlled.

### 3.4 Federated Connectors for Third-Party Systems

IT Security is heavily dependent on specialized platforms. The framework references "SIEM," "SOAR," "endpoint platform," "identity platform," and "threat intel sources" — none of which exist natively in M365 (though Microsoft Sentinel and Defender are M365-adjacent). The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For Microsoft Sentinel (SIEM), Graph Connectors can index alert records, incident correlations, and detection rule metadata into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])`. For Microsoft Defender for Endpoint, similar connectors can index device health, detection events, and investigation results.

For third-party SIEMs (Splunk, CrowdStrike, Elastic), Graph Connectors are available from some vendors or can be custom-built. These index alert summaries and case metadata (not raw telemetry) into M365 Search.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint:
- **Asset inventory**: A Power Automate flow exports asset criticality, environment classification, and service ownership data to SharePoint lists on a daily basis.
- **Alert sync**: Active alerts from the SIEM are synced to the SharePoint Excel tracker. This is a one-way read sync — the SIEM remains the source of truth.
- **Playbook library**: Detection playbooks, response procedures, and severity matrices are maintained as SharePoint documents that skills read at runtime.
- **Threat intel summaries**: Curated threat intelligence briefs exported to SharePoint, searchable by detection rule or attack technique.

**Tier 3 — Manual Input with Templates**

For data that cannot be bridged:
- The `sec-alert-intake` skill prompts the analyst for fields from the SIEM console (alert source, detection rule, affected entities)
- The `sec-enrichment-packet` skill notes which enrichment sources are unavailable and prompts for manual context
- All manual inputs are written to the tracker with a "manual entry" source tag

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint sync for asset inventory, playbooks, and severity matrices) and Tier 3 (manual intake for SIEM alert details). If the organization uses Microsoft Sentinel and Defender, Tier 1 Graph Connectors should be prioritized in Wave 2 as these provide the highest-value enrichment data.

### 3.5 Governance in Cowork

IT Security governance has the most stringent requirements of any LOB, with specific concerns around evidence integrity, access control, and data sensitivity:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; the security operations manager owns the skill suite; SOC leads own routing rules and severity matrices |
| **Access** | M365 permissions govern data access; security investigation data must be scoped to authorized SOC personnel only; skills must never expose investigation details to non-SOC Teams channels |
| **Data classification** | Skill guardrails enforce strict sensitivity rules: no IOCs in channel posts, no affected account names in executive summaries, no investigation hypotheses in written artifacts, no raw telemetry in general-access documents |
| **Evidence integrity** | Investigation artifacts stored in SharePoint with sensitivity labels and version history; skills never modify existing evidence documents — only create new versions |
| **Audit** | Every skill invocation produces artifacts (Excel updates, Teams messages, Outlook drafts) that form a traceable record; the framework requires actor, timestamp, prior value, and case linkage for every write |
| **Change control** | Detection taxonomy, severity matrix, playbooks, and routing rules live in SharePoint documents with version control; changes follow SecOps change management processes |
| **Containment authority** | Skills never execute containment actions (block user, isolate host, disable account, revoke credentials); these are always human-authorized through the SIEM/SOAR platform directly |
| **Insider threat handling** | Cases flagged as potential insider threats receive restricted routing — only designated insider threat analysts can view full case details; general SOC channel visibility is suppressed |
| **Legal hold** | For cases involving potential data breach, the investigation summary skill includes a legal hold advisory; evidence preservation requests are always Outlook drafts requiring analyst review |

**The main governance gap** is that Cowork's guardrails are instruction-based, not enforced by a runtime policy engine. A skill instruction that says "never post IOCs to the general channel" depends on the model following the instruction reliably. The recommended mitigation is defense in depth: (1) encode the rule in skill instructions, (2) use M365 sensitivity labels on SharePoint documents, (3) restrict the SOC channel membership in Teams, and (4) monitor skill outputs during the pilot for any boundary violations.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for IT Security:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis
- Classification quality — does `sec-risk-classifier` recommend the correct alert category? Assessed via comparison against analyst final classification
- Severity agreement rate — does `sec-severity-recommend` match the analyst's final severity assessment? Measured as percentage of recommendations accepted without override
- Enrichment completeness — does `sec-enrichment-packet` include all available context? Assessed by analyst review of 20+ enrichment packets
- Sensitivity compliance — do any skill outputs contain IOCs, account names, or investigation details in unauthorized locations? Assessed via output audit (must be zero violations)

**Process-level evaluation (end-to-end):**
- Alert classification quality — percentage of alerts correctly classified on first attempt
- Correct routing rate — percentage of cases routed to the right analyst queue without reassignment
- Time to triage high-priority alerts — elapsed time from alert intake to classified, prioritized, and assigned state (target: under 15 minutes for high-priority)
- Severity recommendation agreement rate — percentage of severity recommendations accepted by analysts
- False-negative escape rate — percentage of actual security incidents that were not flagged as high severity by the skill (critical safety metric — target: zero)
- Override rate — how often analysts override skill recommendations
- Unauthorized action incidents — any cases where the skill took an action outside its permitted boundary (must be zero)
- Sensitivity violation rate — any cases where investigation details leaked to unauthorized channels (must be zero)

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for IT Security in Cowork:

### Wave 1 — Foundation and Read-Only Assist

**Infrastructure setup:**
- Create the shared Excel alert tracker workbook in SharePoint with standard columns (Case ID, Alert Source, Detection Rule, Affected Entity, Entity Type, Alert Category, Severity, Status, Created Date, Assigned Analyst, Investigation Path, SLA Target, Disposition)
- Upload detection taxonomy, severity matrix, playbook index, analyst routing rules, and asset criticality data to a dedicated, access-controlled SharePoint document library
- Configure SharePoint sensitivity labels on the security cases folder (Confidential - Security Operations)
- Set up the SOC coordination Teams channel with restricted membership (authorized SOC analysts only)
- Create a Power Automate flow to sync asset inventory and service criticality data from the security asset management system to SharePoint (daily)
- Establish the evidence storage folder structure in SharePoint with per-case subfolders

**Skills to build:**
- `sec-alert-intake` — normalize incoming alerts into the tracker
- `sec-enrichment-packet` — assemble entity, asset, and threat context
- `sec-risk-classifier` — present risk classification recommendations
- `sec-investigation-drafter` — draft investigation summaries and evidence requests

**Operating posture:** AI assist mode for classification; deterministic for intake; AI draft plus approve for investigation drafts. All outputs are presented via Adaptive Card or generated documents for analyst review. No automated writes to external systems. No containment recommendations. Test with 30-50 real alerts over 3 weeks with a dedicated analyst reviewing every output for sensitivity compliance.

### Wave 2 — Routing and Severity

**Skills to build:**
- `sec-severity-recommend` — present severity and investigation path recommendations
- `sec-analyst-router` — route cases to analyst queues via Teams

**Promotions:**
- Promote `sec-alert-intake` to write mode (creates tracker records after confirmation)
- Promote `sec-risk-classifier` to write-back mode (updates Excel tracker category after analyst confirmation)
- Promote `sec-analyst-router` to "act within policy" mode for Medium and Low severity cases with clear routing rule matches

**Automation:**
- Set up a scheduled prompt every 10 minutes to check for new unprocessed alerts in the SOC Teams channel
- Set up a daily scheduled prompt to flag cases approaching SLA breach and cases with stale status (no update in 24 hours)

**Graph Connector introduction:**
- If Microsoft Sentinel is available, configure the Graph Connector for alert metadata and incident correlation data
- If Microsoft Defender for Endpoint is available, configure the connector for device context and detection events

**Operating posture:** AI draft plus approve for severity and Critical/High routing. AI act within policy for Medium/Low routing with clear rule matches. All incident declaration, containment, and escalation decisions remain fully human-controlled.

### Wave 3 — Optimization and Proactive Detection

**Enhancements:**
- Introduce additional Graph Connectors for third-party SIEM or endpoint platforms if available
- Add proactive monitoring via scheduled prompt: identify repeat alert clusters for the same entity, flag potential false-negative patterns (alerts closed as benign that recur), surface analyst queue imbalances
- Add playbook-cited investigation guidance in enrichment packets (link specific playbook steps based on the alert classification)
- Add campaign correlation: cross-reference new alerts against recent high-severity cases to identify related activity
- Refine all skills based on override patterns, sensitivity audit results, and analyst feedback from Waves 1-2

**Measurement:**
- Time-to-triage reduction for high-priority alerts: target under 15 minutes (vs. pre-pilot baseline)
- Classification quality target: above 75% agreement with analyst final classification
- Correct routing rate target: above 85% first-time correct assignment
- SLA compliance target: above 95% for Critical and High severity cases
- False-negative escape rate: target zero
- Sensitivity violation rate: target zero
- Draft acceptance rate for investigation summaries: above 65%

---

## Part 5: Generalizing the Approach — IT Security Artifact Pattern

IT Security's primary artifact pattern is **Excel + SharePoint** — structured case tracking in Excel combined with evidence-grade document storage in SharePoint. This reflects the fundamental nature of security operations: it is evidence-heavy, audit-sensitive, and requires both structured case management (classification, severity, assignment, SLA) and document-centric investigation artifacts (enrichment packets, investigation summaries, evidence requests).

This pattern fits the cross-LOB method as follows:

| Dimension | IT Security Pattern |
|---|---|
| Primary tracking artifact | Excel (alert tracker with structured case columns) |
| Primary evidence artifact | Word (enrichment packets, investigation summaries) stored in SharePoint with sensitivity labels |
| Primary policy artifact | SharePoint documents (detection taxonomy, severity matrix, playbooks, routing rules) |
| Primary coordination artifact | Teams (SOC channel — restricted membership, case ID references only in channel posts) |
| Primary communication artifact | Outlook (evidence requests, executive notifications — always drafts) |
| State management approach | Excel tracker as Cowork-accessible state, SIEM/SOAR as source of truth, Power Automate for sync |
| Guardrail posture | Maximum strictness — no auto-containment, no IOC exposure in channels, no investigation hypotheses in artifacts, no false-positive auto-closure, sensitivity labels on all evidence documents |

The decomposition and translation method used for HR onboarding applies directly to IT Security. The key differences are: (1) the guardrail posture is the strictest of any LOB due to data sensitivity and the consequence of false negatives, (2) the federated connector dependency is high because security telemetry lives in specialized platforms, and (3) evidence integrity requirements mean investigation artifacts must be versioned and sensitivity-labeled.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references; security platforms bridged via Graph Connectors |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint; defense-in-depth with sensitivity labels |
| Process State | SharePoint-hosted Excel workbook with sensitivity labels | Durable state externalized to M365 artifacts; SIEM/SOAR is the canonical source of truth |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) + Graph Connectors for SIEM/endpoint/identity | Security telemetry requires connector bridging |
| Approval | Draft tools + Adaptive Card confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns; incident declaration and containment always human-owned |
| Evaluation | Quality rubric scoring + process-level metrics + sensitivity audit | Component eval via trigger analysis; process eval via classification accuracy, triage time, SLA compliance, and false-negative rate; mandatory sensitivity compliance audit |
