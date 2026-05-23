---
name: ps-evidence-packet
description: |
  Assembles case evidence, telemetry, logs, environment context, prior
  cases, and known issues into a structured evidence packet for a
  product support escalation.
  Use when user asks to "build evidence packet for escalation [ID]",
  "pull logs and context for [case]",
  "assemble escalation context for [ID]",
  "what evidence do we have for [escalation]",
  "gather context for this defect",
  or "evidence summary for [case ID]".
  Do NOT use for creating a new escalation record (use ps-escalation-intake),
  classifying the issue or defect path (use ps-defect-classifier),
  assessing customer impact or severity (use ps-impact-assessment),
  routing to engineering (use ps-engineering-routing),
  or drafting handoff communications (use ps-handoff-drafter).
---

## Overview

Assembles a comprehensive evidence packet for an escalation case by gathering the originating support case data, customer and account context, technical environment details, log bundles and diagnostic artifacts, prior cases for the same customer or product area, known issues and existing defects, reproduction steps, and a chronological timeline of case progression. Produces a Word document that serves as the evidence foundation for all downstream skills — classification, impact assessment, routing, and engineering handoff.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined data sources without exercising judgment on severity, defect classification, or investigation path. Evidence files are never modified.

## When to Use

- An escalation record has been created via ps-escalation-intake and needs evidence assembled into a structured packet
- An escalation engineer wants to understand the full technical and customer context before classification
- New evidence (logs, screenshots, reproduction steps) has been added and the packet needs to be refreshed
- An engineering triage lead needs to understand what evidence is available before starting investigation

## When NOT to Use

- Creating a new escalation record — use ps-escalation-intake
- Classifying the issue type or suspected defect path — use ps-defect-classifier
- Assessing customer impact or recommending severity — use ps-impact-assessment
- Routing the escalation to an engineering owner or queue — use ps-engineering-routing
- Drafting the engineering handoff or customer update — use ps-handoff-drafter
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather escalation evidence and context", activeForm="Collecting evidence")
TaskCreate(subject="Build evidence packet document", activeForm="Building evidence packet")
```

### Step 1: Read the Escalation Record

- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — find the case by Escalation ID
- Extract: Escalation ID, Originating Case ID, Customer Name, Account ID, Product Area, Issue Summary, SLA Deadline

### Step 2: Gather Support Case Context

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["support-connector"])` — originating case details, case history, prior cases for same customer, agent notes, workaround attempts

**Via email and Teams:**
- `SearchM365(sources=["email"], query="[case ID] [customer name]")` — escalation email thread, customer correspondence, internal support discussion
- `SearchM365(sources=["teams"], query="[case ID] [product area]")` — support-to-engineering discussions, triage channel threads

Assemble case context:
- Original customer-reported issue description
- Support agent troubleshooting steps taken
- Workaround attempts and results
- Customer communication history

### Step 3: Gather Customer and Account Context

- `SearchPeople` — resolve customer contact and account owner
- `GetUserDetails` — pull customer contact profile, account owner profile, and customer success partner

Assemble customer context:
- Account tier (enterprise, premium, standard)
- Number of affected users (if known)
- Business impact described by customer
- Active escalations for other product areas (flag if found — may indicate broader issue)
- Prior escalation history for this customer

### Step 4: Inventory Technical Evidence

- `GetDriveChildren` — list all files in the escalation's evidence folder in SharePoint
- `ReadFileContent` — read available text-based evidence files (logs, reproduction notes, configuration exports)

Inventory and categorize:
| Evidence Type | Status | Details |
|--------------|--------|---------|
| **Log bundle** | Present / Missing | File name, upload date, size |
| **Screenshots** | Present / Missing | File names, count |
| **HAR files** | Present / Missing | File name, upload date |
| **Reproduction steps** | Present / Missing | Written steps or "not provided" |
| **Environment details** | Present / Missing | Product version, configuration, platform |
| **Crash reports** | Present / Missing | File name, upload date |
| **Configuration export** | Present / Missing | File name, upload date |

### Step 5: Gather Known Issue and Prior Defect Context

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["issue-tracker-connector"])` — known issues and existing defects for the suspected product area

**Via SharePoint (if manual sync):**
- `SearchM365(sources=["files"], query="known issues [product area]")` then `ReadFileContent` — known issues list synced from issue tracker

Assemble known issue context:
- Known issues matching this product area and symptom pattern
- Existing defects that may be related (with status: open, in progress, resolved)
- Prior escalations with similar symptoms (with resolution if available)
- Active incidents that may be related

### Step 6: Build the Chronological Timeline

Assemble a timeline of case progression from available data:

- Case created date and original report
- Escalation date and escalating agent
- Key support interactions (troubleshooting steps, workaround attempts)
- Evidence uploads (when logs, screenshots, etc. were added)
- Any severity changes or status updates

### Step 7: Calculate Evidence Completeness

Assess the completeness of the evidence packet:

| Completeness Level | Criteria |
|-------------------|---------|
| **Complete** | Log bundle present, reproduction steps documented, environment details captured, customer impact described |
| **Partial** | Some evidence present but critical items missing (no logs, no repro steps, or no environment details) |
| **Insufficient** | Critical evidence missing that would block engineering investigation |

Flag specific gaps with recommendations for what the escalation engineer should collect.

### Step 8: Assemble the Evidence Packet

Produce a Word document (invoke `docx` skill) with the following sections:

1. **Escalation Summary** — Escalation ID, Originating Case ID, Customer, Product Area, Issue Description, SLA Deadline
2. **Customer Impact** — Account tier, affected users, business impact, prior escalation history
3. **Technical Environment** — Product version, configuration, platform details (or "Not provided" for missing items)
4. **Evidence Inventory** — complete list of all artifacts in the evidence folder with SharePoint links, upload dates, and file types
5. **Reproduction Steps** — documented steps to reproduce (or "Not provided — evidence gap flagged")
6. **Case Timeline** — chronological progression from initial report through escalation
7. **Known Issue Cross-Reference** — matching known issues, prior defects, and related escalations
8. **Prior Cases** — previous escalations from this customer or product area with resolutions
9. **Key Contacts** — escalation engineer, customer success partner, engineering triage lead (if known)
10. **Evidence Completeness Assessment** — completeness level, specific gaps, and collection recommendations
11. **Data Source Inventory** — which sources were consulted and which returned no data

Save to the SharePoint escalation evidence folder and link from the tracker row.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find escalation tracker, known issues list, evidence documents |
| SearchM365 (connectors) | Pull support case data, known issues, prior defects via Graph Connectors |
| SearchM365 (email) | Find escalation email threads and customer correspondence |
| SearchM365 (teams) | Find support-to-engineering discussions in triage channels |
| ReadFileContent | Read tracker, evidence files, known issues list |
| GetDriveChildren | Inventory evidence files in escalation folder |
| SearchPeople | Resolve customer contacts and account owners |
| GetUserDetails | Pull contact profiles |

## Guardrails

- **Never modify or delete original evidence files** — the evidence packet is a read-only synthesis; original logs, screenshots, and artifacts remain untouched in the evidence folder
- **Cite source and retrieval date for every data point** — system-sourced data (support platform, issue tracker) is labeled with source and timestamp; email and chat-sourced context is labeled as correspondence-based
- **Flag critical evidence gaps prominently** — missing logs, missing reproduction steps, or missing environment details are called out explicitly with collection recommendations
- **Flag if the customer has active escalations for other product areas** — this may indicate a broader issue affecting the account
- **Never assess severity or classify the defect** in the evidence packet — evidence assembly is separate from judgment; classification and severity are handled by downstream skills
- **Never fabricate evidence or reproduction steps** — if information is unavailable, report it as missing rather than inferring
- **Never include customer financial data** (contract value, ARR, renewal dates) — use account tier and general business impact indicators only
- **Operate within read-only boundaries** — never update the support case, issue tracker, or any source system; the evidence packet is a synthesis, not a state change
- **Include the SLA deadline and remaining time** — the escalation engineer needs urgency context for evidence collection prioritization
- **Timestamp all evidence citations** — if the packet is refreshed after new evidence arrives, stale citations are identifiable
