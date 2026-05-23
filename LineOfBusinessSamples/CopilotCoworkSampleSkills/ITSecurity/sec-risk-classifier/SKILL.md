---
name: sec-risk-classifier
description: |
  Classifies alert type and likely risk based on the detection taxonomy,
  enrichment context, and historical case patterns.
  Use when user asks to "classify this alert", "what type of threat is this",
  "risk assessment for this alert", "categorize this security event",
  "alert classification for [case ID]", "what kind of attack is this",
  or "threat classification for [entity]".
  Do NOT use for creating a new case (use sec-alert-intake),
  enriching alert context (use sec-enrichment-packet),
  assessing severity (use sec-severity-recommend),
  routing to analysts (use sec-analyst-router),
  or drafting investigation summaries (use sec-investigation-drafter).
---

## Overview

Compares alert telemetry and enrichment context against the detection taxonomy to recommend an alert category, likely threat path, and confidence level. Cross-references historical cases for the same detection rule and factors in the entity's risk profile. Presents classification with supporting evidence and similar past cases for analyst review.

This skill operates in "AI assist" mode — it reads and analyzes case data but only presents classification as a recommendation via Adaptive Card. The analyst confirms or overrides the classification before it is recorded.

## When to Use

- An alert case has been enriched and needs type and risk classification
- An analyst wants a recommended alert category and threat path before review
- A case needs reclassification after new evidence or updated enrichment
- Triage needs to determine whether an alert is phishing, malware, identity compromise, data exfiltration, or another category

## When NOT to Use

- Creating a new alert case — use sec-alert-intake
- Assembling entity, asset, or threat context — use sec-enrichment-packet
- Assessing severity or investigation path — use sec-severity-recommend
- Routing to an analyst queue or SOC team — use sec-analyst-router
- Drafting investigation summaries or evidence requests — use sec-investigation-drafter
- Confirming triage disposition — this is always a human decision (SEC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and classification inputs", activeForm="Reading classification inputs")
TaskCreate(subject="Classify alert type and present recommendation", activeForm="Classifying alert type")
```

### Step 1: Read Classification Inputs

**Read the enrichment packet:**
- `SearchM365(sources=["files"], query="enrichment packet [Case ID]")` then `ReadFileContent`
- If no enrichment packet exists, note this as a prerequisite gap — recommend running sec-enrichment-packet first

**Read the detection taxonomy:**
- `SearchM365(sources=["files"], query="detection taxonomy")` then `ReadFileContent` — alert category definitions, threat path descriptions, classification criteria

**Read the playbook index:**
- `SearchM365(sources=["files"], query="security playbook index")` then `ReadFileContent` — playbook-to-category mappings

**Read the alert tracker:**
- `SearchM365(sources=["files"], query="security alert tracker")` then `ReadFileContent` — current case data

**Check historical patterns:**
- `SearchM365(sources=["files"], query="[detection rule] classification")` — prior cases with the same detection rule and their final classifications
- `SearchM365(sources=["connectors"], connector_ids=["sentinel-connector"])` — historical alert classification patterns and false-positive rates for this detection rule (if available)

### Step 2: Classify Alert Type

Compare case attributes against the detection taxonomy. Consider:

**Alert attributes:**
- Detection rule name, type, and known attack technique
- Alert description and telemetry indicators
- Affected entity type and risk profile
- Time of detection and activity pattern

**Entity risk factors:**
- Privileged user or service account
- Critical asset or production system
- Internet-facing system or externally accessible application
- High-value data access (PCI, HIPAA, PII environments)

**Historical context:**
- False-positive rate for this detection rule
- Prior classifications for similar alerts
- Prior investigation outcomes for this entity

### Alert Categories

Classify into one of the taxonomy-defined categories:

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

### Step 3: Assess Confidence

Rate classification confidence:

- **High confidence** — detection rule, alert telemetry, and enrichment context clearly match a single alert category; historical cases with this rule consistently map to the same category
- **Medium confidence** — primary classification is supported but some indicators point to alternatives; the detection rule has moderate false-positive rates or maps to multiple categories historically
- **Low confidence** — alert attributes are ambiguous, detection rule is new or has high false-positive rates, or enrichment data is incomplete; senior analyst review recommended

### Step 4: Identify Similar Past Cases

Search for the top 3 most similar past cases:
- Same detection rule with confirmed final classification
- Same affected entity type with similar alert patterns
- Same environment or asset classification

For each similar case, note: Case ID, date, final classification, and outcome (incident, false positive, benign).

### Step 5: Present Classification

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Alert Source, Detection Rule, Affected Entity
- **Recommended alert category** — primary classification with description
- **Likely threat path** — description of the expected attack progression
- **Confidence level** — High / Medium / Low with reasoning
- **Historical false-positive rate** — percentage for this detection rule if available
- **Applicable playbook** — playbook reference linked to the recommended category
- **Top 3 similar past cases** — Case ID, date, classification, and outcome
- **Alternative classifications** — if confidence is medium or low, list alternatives with supporting indicators
- **Entity risk factors** — privileged user, critical asset, or high-value data flags
- **Urgent escalation banner** — if the classification suggests an active intrusion, data exfiltration, or lateral movement, add an urgent escalation banner
- **Draft label** — "CLASSIFICATION RECOMMENDATION — analyst review required before recording"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find detection taxonomy, playbook index, alert tracker, prior case classifications |
| SearchM365 (connectors) | Retrieve historical classification patterns and false-positive rates from Sentinel via Graph Connector |
| ReadFileContent | Read taxonomy, playbook index, enrichment packet, tracker |

## Guardrails

- **Present classification as a recommendation only** — never auto-update the alert category in the tracker without analyst confirmation
- **Always display confidence level and reasoning** — the analyst must see why the classification was recommended
- **If confidence is low, explicitly flag for senior analyst review** — do not present uncertain classifications as definitive
- **Never classify an alert as false positive or benign** without analyst confirmation — false-negative risk is the primary safety concern in security operations
- **Do not include specific IOC values** (IP addresses, hashes, domain names) in the Adaptive Card — reference them by Case ID and enrichment packet section
- **If the classification suggests active intrusion or data exfiltration**, add an urgent escalation banner — time-sensitive threats must be visually prominent
- **Never assess severity, recommend containment, or prescribe response actions** — classification determines the category only; severity and response are handled by downstream skills
- **Do not modify the alert tracker** — this skill presents analysis only; tracker updates happen after analyst confirmation
- **If the classification maps to insider threat**, flag for restricted handling — insider threat cases require specialized routing and restricted visibility
- **Never include investigation hypotheses or threat attribution** in the classification output — state what category the evidence supports, not who is responsible
