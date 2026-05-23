---
name: ps-handoff-drafter
description: |
  Drafts the engineering handoff summary and customer-facing status
  update for an escalated support case.
  Use when user asks to "draft engineering handoff for escalation [ID]",
  "write handoff summary for [case]",
  "prepare customer update for [escalation]",
  "escalation handoff to engineering for [ID]",
  "draft customer email for [escalation]",
  or "write the engineering summary for [case]".
  Do NOT use for creating a new escalation record (use ps-escalation-intake),
  gathering evidence and context (use ps-evidence-packet),
  classifying the issue or defect path (use ps-defect-classifier),
  assessing customer impact or severity (use ps-impact-assessment),
  or routing to engineering (use ps-engineering-routing).
---

## Overview

Drafts two communication artifacts for an escalated support case: (1) an engineering handoff summary — a structured Word document providing engineering with everything needed to begin investigation without asking support for clarification, and (2) a customer-facing status update email — an empathetic, professional acknowledgment that the issue is being investigated. Matches tone to audience and enforces strict separation between internal and external content.

This skill operates in "AI draft plus approve" mode — every communication is generated as a draft for escalation engineer review. Customer-facing emails are created as Outlook drafts; engineering summaries are presented for review before distribution.

## When to Use

- An escalation has been routed to engineering and needs a formal handoff summary document
- A customer needs a status update acknowledging that their escalation is being investigated
- An engineering triage lead has requested additional context beyond the evidence packet
- A customer success partner needs to be notified about a high-severity escalation for their account
- A follow-up update is needed after the initial handoff

## When NOT to Use

- Creating a new escalation record — use ps-escalation-intake
- Gathering case evidence, logs, and telemetry context — use ps-evidence-packet
- Classifying the issue type or suspected defect path — use ps-defect-classifier
- Assessing customer impact or recommending severity — use ps-impact-assessment
- Routing the escalation to an engineering owner or queue — use ps-engineering-routing
- Confirming handoff disposition — this is always a human decision (PS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and draft templates", activeForm="Preparing handoff drafts")
TaskCreate(subject="Draft communications", activeForm="Drafting handoff and customer update")
```

### Step 1: Read Handoff Inputs

**Read all case artifacts:**
- `SearchM365(sources=["files"], query="escalation tracker")` then `ReadFileContent` — full escalation record including classification, severity, routing, SLA
- `SearchM365(sources=["files"], query="evidence packet [escalation ID]")` then `ReadFileContent` — complete evidence packet
- `GetDriveChildren` — evidence folder inventory for document references

**Read communication templates:**
- `SearchM365(sources=["files"], query="engineering handoff template")` then `ReadFileContent` — standard handoff document structure
- `SearchM365(sources=["files"], query="customer update template")` then `ReadFileContent` — customer communication template

**Read recent correspondence:**
- `SearchM365(sources=["email"], query="[case ID] [customer name]")` — recent email thread for context and tone

### Step 2: Identify Required Communications

Determine which communications are needed based on the user's request and case state:

| Communication Type | Audience | Channel | When Needed |
|-------------------|----------|---------|-------------|
| **Engineering handoff summary** | Engineering triage lead | Word document + Teams message | Always — the primary handoff artifact |
| **Customer status update** | Customer contact | Outlook draft | When customer needs acknowledgment of escalation |
| **Customer success notification** | Customer success partner | Teams message or Outlook draft | For Sev 1/2 or enterprise accounts |
| **Follow-up update** | Customer contact | Outlook draft | When providing progress or resolution information |

### Step 3: Draft Engineering Handoff Summary

Produce a Word document (invoke `docx` skill) with the following sections:

**1. Escalation Header**
- Escalation ID, Originating Case ID
- Customer Name, Account Tier
- Product Area, Component
- Severity Level
- Assigned Engineering Triage Lead
- SLA Deadline and Time Remaining

**2. Problem Statement**
- Clear, technical description of the reported issue
- Affected feature, service, or component
- Conditions under which the issue occurs

**3. Evidence Inventory**
- Complete list of available evidence artifacts with SharePoint links:
  - Log bundles (with file names and timestamps)
  - Screenshots
  - HAR files
  - Reproduction notes
  - Configuration exports
- Evidence gaps flagged (what is missing)

**4. Reproduction Steps**
- Step-by-step reproduction if available
- If not available: "Reproduction steps not yet documented — customer reports [summary of conditions]"

**5. Customer Impact**
- Affected users and scope
- Business function impact
- Workaround availability and effectiveness
- Customer sentiment and urgency context

**6. Suggested Investigation Path**
- Suspected defect path from classification
- Recommended starting point for investigation
- Related known issues or prior defects
- Confidence level for the classification

**7. Case Timeline**
- Chronological progression from initial report through escalation and routing

**8. Key Contacts**
- Escalation engineer (name, email)
- Customer success partner (name, email)
- Product support manager (for Sev 1/2)

Save to the SharePoint escalation evidence folder.

### Step 4: Draft Customer Status Update

Create an Outlook draft email (`CreateDraftMessage`) with:

- **To:** Customer contact email
- **Subject:** "Update: [Issue summary] — Case [Originating Case ID]"
- **Body:**
  - Acknowledgment that the issue has been escalated to the engineering team
  - Brief summary of what is being investigated (in customer-accessible language)
  - Workaround information (if available)
  - Expected next update timeline (general, not specific)
  - Contact information for the escalation engineer
  - Professional, empathetic tone

### Step 5: Draft Customer Success Notification (If Needed)

For Sev 1/2 or enterprise accounts, draft a notification to the customer success partner:

- `PostMessage` (after confirmation) or `CreateDraftMessage` — notification with:
  - Escalation summary (severity, product area, customer impact)
  - Current status and response path
  - What customer success should know for account management

### Step 6: Present Drafts for Review

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Engineering handoff** — document location, completeness check (all required sections present)
- **Customer update** — draft preview with sensitive content check
- **Customer success notification** — if applicable
- **Content separation check** — confirmation that no internal details appear in customer-facing drafts
- **Tone check** — technical for engineering, empathetic for customer
- **Draft label** — "HANDOFF DRAFTS — escalation engineer review required before distribution"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find escalation tracker, evidence packet, handoff templates, customer update templates |
| SearchM365 (email) | Find recent correspondence thread for context |
| ReadFileContent | Read all case artifacts and templates |
| GetDriveChildren | Inventory evidence folder for document references |
| CreateDraftMessage | Create Outlook draft for customer status update |
| PostMessage | Send Teams notification to customer success partner (after confirmation) |

## Guardrails

- **Always create customer-facing communications as Outlook drafts** — never send without explicit escalation engineer confirmation
- **Never include internal severity classifications, engineering queue names, or triage notes** in customer-facing drafts — these are internal operational details
- **Never make timeline commitments** in customer update drafts — use language like "we are actively investigating" and "we will provide an update by [general timeframe]" rather than specific dates or resolution promises
- **Engineering handoff must include all required sections** — problem statement, evidence inventory, reproduction steps (or explicit gap note), customer impact, and suggested investigation path; flag if any section is empty
- **Cite the evidence packet and all attached artifacts** — engineering should not need to ask support for clarification; the handoff document must be self-contained
- **Match tone to audience** — technical and precise for engineering; empathetic and professional for customer; operational and direct for customer success
- **Never reveal internal investigation hypotheses to the customer** — customer communications acknowledge the issue and describe next steps without speculating on root cause
- **Never include customer financial data** (contract value, ARR) in any communication — account tier is sufficient context
- **Include the Escalation ID in every communication** for traceability
- **Never auto-send any communication** — all outputs are drafts or require explicit confirmation before posting
- **Flag if the evidence packet has been updated since the handoff was drafted** — stale handoffs may not reflect the latest evidence
