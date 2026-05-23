---
name: bnk-case-routing
description: |
  Routes fraud and dispute cases to the correct analyst queue or escalation lane
  based on the routing matrix, classification, and risk assessment.
  Use when user asks to "route this fraud case", "assign case to analyst",
  "who handles this dispute type", "recommend routing for [case ID]",
  "escalate this case", "assign [case] to the right queue",
  "route dispute to chargeback", or "fraud case assignment for [case ID]".
  Do NOT use for creating a new case (use bnk-case-intake),
  assembling account and transaction context (use bnk-fraud-context-packet),
  classifying fraud scenario or dispute type (use bnk-scenario-classifier),
  assessing urgency or evidence gaps (use bnk-gap-risk-detection),
  or drafting customer communications (use bnk-case-comms).
---

## Overview

Determines the correct analyst queue, review lane, or escalation path for a fraud or dispute case based on the routing matrix, scenario classification, risk assessment, and urgency level. Resolves analyst identities, verifies queue capacity considerations, enforces segregation of duties, and sends routing notifications after fraud operations approval.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for fraud operations review before any messages are sent or assignments made.

## When to Use

- A fraud or dispute case has been classified and assessed, and needs analyst assignment
- A case needs to be routed to a specific handling lane (fraud ops, dispute ops, chargeback, AML)
- An escalation path needs to be activated for a high-risk or regulatory-sensitive case
- A case needs reassignment after reclassification or updated risk assessment

## When NOT to Use

- Creating a new case — use bnk-case-intake
- Assembling account and transaction context — use bnk-fraud-context-packet
- Classifying the fraud scenario — use bnk-scenario-classifier
- Assessing urgency or evidence gaps — use bnk-gap-risk-detection
- Drafting customer or analyst communications — use bnk-case-comms
- Confirming triage disposition — this is always a human decision (BNK-FRD-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read classification and risk data", activeForm="Reading routing inputs")
TaskCreate(subject="Present routing recommendation", activeForm="Preparing routing recommendation")
```

### Step 1: Read Routing Inputs

**Read case data:**
- `SearchM365(sources=["files"], query="fraud dispute case tracker")` then `ReadFileContent` — case record with classification and risk assessment
- Classification output — scenario type, confidence, handling lane recommendation
- Risk assessment output — urgency, escalation flags, SLA status

**Read the routing matrix:**
- `SearchM365(sources=["files"], query="fraud dispute routing matrix")` then `ReadFileContent` — who handles what by scenario type, urgency, and product

**Read escalation rules:**
- `SearchM365(sources=["files"], query="fraud escalation procedures")` then `ReadFileContent` — escalation criteria and paths

### Step 2: Determine Routing

**Handling lane assignment:**

| Handling Lane | Routes To | Criteria |
|---------------|-----------|----------|
| **Fraud Operations** | Fraud analyst queue | Card fraud, account takeover, unauthorized transactions |
| **Dispute Operations** | Dispute specialist queue | Billing disputes, friendly fraud, merchant errors |
| **Chargeback Operations** | Chargeback team | Cases requiring card network chargeback initiation |
| **AML Escalation** | AML team (restricted) | Cases with suspicious activity indicators |
| **Special Investigations** | Internal fraud team (restricted) | Cases with internal fraud indicators |
| **Supervisor Review** | Fraud operations supervisor | High-value, complex, or multi-pattern cases |

**Analyst assignment:**
- `SearchPeople` — resolve analysts in the target queue by name or function
- `GetManagerDetails` / `GetDirectReportsDetails` — fraud operations org structure for escalation
- `ReadFileContent` — routing matrix for analyst assignment criteria (product expertise, case load considerations)
- `ListCalendarView` — check analyst availability for urgent cases

**Segregation of duties check:**
- The analyst who performed intake must not be auto-assigned to review the same case
- The analyst who classified the case should not be the sole reviewer for high-risk cases
- If a conflict exists, flag it and recommend an alternative assignment

### Step 3: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Scenario Type, Urgency, SLA Status
- **Recommended handling lane** — with routing matrix criteria cited
- **Recommended analyst or queue** — name, role, rationale
- **Escalation path** — if applicable, with escalation criteria cited
- **Segregation of duties status** — any conflicts flagged
- **SLA impact** — time remaining, whether routing adds risk to SLA compliance
- **Routing actions** — "Send Teams notification to [analyst]" / "Escalate to [supervisor]" / "Route to AML team"

### Step 4: Execute After Approval

After fraud operations confirms the routing:

**Send routing notifications:**
- `PostMessage` — Teams message to the assigned analyst or queue with Case ID, scenario type, urgency, and case folder location
- For AML escalation: `CreateDraftMessage` — Outlook draft to AML team (not Teams, due to restricted access requirements)

**Check analyst availability for urgent cases:**
- `ListCalendarView` — if the assigned analyst is unavailable and the case is urgent, recommend a backup assignment

**Update the tracker:**
- Record analyst assignment, handling lane, routing timestamp, routing approver, and rationale in the case tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find routing matrix, escalation procedures, case tracker |
| ReadFileContent | Read routing rules and case data |
| SearchPeople / GetUserDetails | Resolve analyst and supervisor identities |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths |
| PostMessage | Teams notifications to assigned analysts or queues (after approval) |
| CreateDraftMessage | Outlook draft for AML escalation or email-based routing |
| ListCalendarView | Check analyst availability for urgent cases |

## Guardrails

- **Present routing recommendation for fraud operations review** before sending any messages or making assignments
- **Never auto-route to AML or suspicious activity review lanes** — AML escalation requires explicit supervisor approval and carries regulatory accountability
- **Never auto-route to Special Investigations** — internal fraud referrals require supervisor authorization
- **Never assign a case to an analyst outside the approved routing matrix** — assignments must follow documented routing rules
- **Enforce segregation of duties** — the analyst who performed intake should not be auto-assigned to review; flag and resolve any conflicts
- **Escalate to fraud operations supervisor** if no clear routing path exists in the matrix — do not guess or create ad hoc assignments
- **Log routing decision, rationale, and approver** in the case tracker for audit trail — every routing action must be traceable
- **Respect access boundaries** — AML routing notifications go via Outlook draft only (not Teams) because AML investigation status is restricted
- **Include SLA status** in every routing notification — the assigned analyst needs to know the urgency and remaining time
