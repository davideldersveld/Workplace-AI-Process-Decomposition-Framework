---
name: legal-review-routing
description: |
  Determines the review path, assigns counsel, and identifies required approvers
  based on the deviation profile, contract type, and approval matrix.
  Use when user asks to "route this contract for review",
  "who reviews [contract type]", "assign reviewers for [case]",
  "determine approval path for [contract]",
  "set up review chain for case [ID]",
  "route legal case [ID]",
  or "escalate contract [ID] to senior counsel".
  Do NOT use for creating a new case (use legal-contract-intake),
  assembling review context (use legal-review-packet),
  detecting clause deviations (use legal-deviation-detection),
  or drafting summaries and follow-ups (use legal-triage-comms).
---

## Overview

Determines the correct review path for a contract case based on the deviation profile, contract type, value thresholds, and the approval matrix. Resolves counsel assignments by practice area and jurisdiction, identifies all required approvers, determines escalation needs for high-severity deviations, and presents the routing recommendation for legal operations review before execution.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for legal operations review and confirmation before any assignments are made or notifications sent.

## When to Use

- A contract case has been classified and deviations identified — it needs reviewer assignment
- Legal operations needs to determine which counsel handles a specific contract type
- A contract has high-severity deviations requiring escalation to senior counsel or business approvers
- A case needs re-routing after reclassification, new deviations, or priority change

## When NOT to Use

- Creating a new case record — use legal-contract-intake
- Assembling contract context or playbook materials — use legal-review-packet
- Detecting clause deviations against the playbook — use legal-deviation-detection
- Drafting summaries, status updates, or follow-up requests — use legal-triage-comms
- Confirming final triage disposition — this is always a human decision (LG-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and routing inputs", activeForm="Reading routing inputs")
TaskCreate(subject="Determine review path and present recommendation", activeForm="Routing contract case")
```

### Step 1: Read Routing Inputs

**Read the case data:**
- `SearchM365(sources=["files"], query="contract case tracker")` then `ReadFileContent` — current case status, contract type, counterparty, urgency

**Read the deviation profile:**
- `SearchM365(sources=["files"], query="deviation report [Case ID]")` then `ReadFileContent` — deviation count, severity levels, flagged clauses

**Read the approval matrix and routing rules:**
- `SearchM365(sources=["files"], query="legal approval matrix")` then `ReadFileContent` — reviewer assignment rules by contract type, value, jurisdiction, and deviation severity
- `SearchM365(sources=["files"], query="legal review routing rules")` then `ReadFileContent` — practice area assignments, escalation triggers

**Resolve available counsel:**
- `SearchPeople` — resolve counsel names by practice area or specialty
- `GetUserDetails` — verify counsel availability and role
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths
- `ListCalendarView` — check counsel availability for timely review

### Step 2: Determine Counsel Assignment

Match the case to a reviewer based on:

| Assignment Factor | Source |
|-------------------|--------|
| Contract type | Case tracker (NDA, MSA, SOW, etc.) |
| Jurisdiction | Contract document or case data |
| Practice area | Approval matrix (commercial, employment, IP, regulatory) |
| Deviation severity | Deviation report (highest severity level) |
| Value threshold | Case data or contract terms (if known) |
| Counterparty tier | Counterparty history (strategic, standard, new) |

### Reviewer Categories

| Reviewer | Handles | Triggered By |
|----------|---------|-------------|
| **Associate counsel** | Standard contract reviews with Level 1-2 deviations | NDAs, standard SOWs, low-value amendments |
| **Senior counsel** | Complex contracts or Level 3 deviations | MSAs, high-value SOWs, strategic agreements, Level 3 deviations |
| **Practice area lead** | Specialty reviews (IP, employment, regulatory) | Contracts requiring domain expertise |
| **Deputy general counsel** | Level 4 deviations or contracts above authority threshold | Deviations outside policy, high-value contracts |
| **Business approver** | Business terms requiring business unit sign-off | Value commitments, service level commitments, exclusivity |
| **Compliance reviewer** | Regulatory clause deviations | Data protection, export control, anti-corruption deviations |

### Step 3: Identify Required Approvers

Based on the approval matrix, determine all required approvals:

- **Standard review** (Level 1-2 deviations only) — assigned counsel reviews and approves
- **Elevated review** (Level 3 deviations) — senior counsel or practice area lead reviews; business approver may be required for business terms
- **Escalated review** (Level 4 deviations) — deputy general counsel or legal operations manager must review; business unit head approval required for business terms
- **Compliance review** — required when data protection, export control, or anti-corruption clauses have deviations at any severity level

### Step 4: Determine Timeline

Based on urgency and deviation profile:

| Urgency | Standard (L1-L2 deviations) | Elevated (L3 deviations) | Escalated (L4 deviations) |
|---------|---------------------------|------------------------|--------------------------|
| Standard | 5 business days | 3 business days | 2 business days |
| Expedited | 3 business days | 2 business days | 1 business day |
| Urgent | 1 business day | 1 business day | Same day |

### Step 5: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Counterparty, Contract Type, Urgency
- **Deviation profile** — count by severity level, highest severity, key flagged clauses
- **Assigned counsel** — recommended reviewer with practice area rationale
- **Required approvers** — full approval chain with role and trigger reason
- **Escalation flags** — Level 3 or 4 deviations requiring elevated authority
- **Compliance referral** — if regulatory deviations are detected
- **Review timeline** — target completion date based on urgency and deviation profile
- **SLA status** — time elapsed since intake, triage SLA remaining
- **Draft label** — "ROUTING RECOMMENDATION — legal operations review required before assignment"

### Step 6: Execute Routing (After Confirmation)

**Send Teams notification to assigned counsel:**
- `PostMessage` — direct message to assigned counsel with:
  - Case ID and counterparty name
  - Contract type and urgency
  - Deviation summary (count by severity — no clause details in Teams)
  - Review timeline target
  - Link to the matter folder in SharePoint for full materials

**Send Outlook draft for formal review assignment (if required):**
- `CreateDraftMessage` — formal review assignment email for complex cases or escalated reviews

**Create review deadline calendar event:**
- `CreateEvent` — calendar hold for the assigned counsel with the review deadline

**Update case tracker:**
- Record assigned counsel, required approvers, review timeline, routing rationale, and confirming analyst in the case tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, deviation report, approval matrix, routing rules |
| ReadFileContent | Read case data, deviation report, approval matrix, routing rules |
| SearchPeople | Resolve counsel and approver names by practice area |
| GetUserDetails | Verify counsel availability and role |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths |
| ListCalendarView | Check counsel availability for timely review |
| PostMessage | Teams notification to assigned counsel (after confirmation) |
| CreateDraftMessage | Formal review assignment email (after confirmation) |
| CreateEvent | Review deadline calendar hold |

## Guardrails

- **Present the routing recommendation for legal operations review** before sending any assignments or notifications — routing decisions are not auto-executed
- **Only assign to counsel listed in the approved reviewer matrix** — ad hoc assignments outside the matrix are not permitted
- **If no matching routing rule exists**, escalate to the legal operations manager rather than guessing
- **Escalate to senior counsel or deputy general counsel** when deviation severity reaches Level 3 or 4 — standard counsel authority is insufficient for these deviations
- **Never reveal deviation details or contract terms in Teams messages** — use Case ID and a reference to the matter folder; full details are in the SharePoint documents
- **Never reveal deviation details or contract terms in emails to non-legal recipients** — business approver notifications reference the case and approval required, not the substance of the deviation
- **Include Case ID, review timeline, and matter folder reference** in every outbound notification for traceability
- **Log all routing decisions** to the case tracker with timestamp, assigned counsel, required approvers, rationale, and confirming analyst for audit
- **Never modify deviation severity or classification during routing** — routing uses the existing deviation profile; changes require re-running legal-deviation-detection
- **If compliance review is triggered**, include the compliance reviewer in the approval chain — regulatory clause deviations cannot be approved without compliance sign-off
- **Mark all counsel-facing communications with privilege headers** — "PRIVILEGED AND CONFIDENTIAL — ATTORNEY WORK PRODUCT"
