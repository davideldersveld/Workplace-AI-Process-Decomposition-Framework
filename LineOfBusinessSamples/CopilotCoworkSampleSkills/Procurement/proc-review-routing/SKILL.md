---
name: proc-review-routing
description: |
  Determines the required review path and routes the supplier onboarding
  case to the appropriate risk, legal, tax, AP, and category reviewers.
  Use when user asks to "route supplier for review",
  "who needs to review [supplier]",
  "assign reviewers for supplier case [ID]",
  "determine review path for [supplier]",
  "submit [supplier] for onboarding review",
  or "set up review chain for [supplier case]".
  Do NOT use for creating a new supplier case (use proc-supplier-intake),
  gathering supplier and policy context (use proc-context-packet),
  detecting missing items or risk indicators (use proc-gap-risk-detect),
  drafting outreach communications (use proc-supplier-comms),
  or summarizing the reviewer packet (use proc-reviewer-packet).
---

## Overview

Determines the required review path for a supplier onboarding case based on the approval matrix, spend tier, risk flags, geography, and category requirements. Resolves reviewer identities, presents the routing recommendation for analyst approval, and then sends Teams notifications to assigned reviewers and creates calendar holds for review deadlines.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for procurement analyst review. Notifications are sent only after explicit confirmation.

## When to Use

- A supplier onboarding case has passed gap and risk detection and is ready for reviewer assignment
- A procurement analyst needs to determine which reviewers are required for a case
- A case needs re-routing after changes to risk tier, spend category, or geography
- A review path needs to be updated after new risk flags are identified

## When NOT to Use

- Creating a new supplier case — use proc-supplier-intake
- Gathering supplier and policy context — use proc-context-packet
- Detecting missing documents or risk indicators — use proc-gap-risk-detect
- Drafting outreach or follow-up communications — use proc-supplier-comms
- Summarizing the case for reviewer approval — use proc-reviewer-packet
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and approval matrix", activeForm="Determining review path")
TaskCreate(subject="Route case and send reviewer notifications", activeForm="Routing supplier case")
```

### Step 1: Read Routing Inputs

**Read the case data:**
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` — Case ID, Supplier Name, Spend Category, Geography, Risk Tier, Risk Flags, Estimated Spend, Priority

**Read the gap and risk report:**
- Review the risk flags, screening status, and checklist completion from the tracker or context packet

**Read the approval matrix:**
- `SearchM365(sources=["files"], query="approval matrix")` then `ReadFileContent` — reviewer requirements by spend tier, risk level, category, and geography
- `SearchM365(sources=["files"], query="onboarding routing rules")` then `ReadFileContent` — routing policies and escalation rules

### Step 2: Determine Required Reviewers

Match the case to required reviewers based on the approval matrix:

| Reviewer | Required When | Review Scope |
|----------|--------------|-------------|
| **Category manager** | All supplier onboarding cases | Category fit, business need, supplier qualification |
| **Risk reviewer** | Cases with risk flags, sanctions-sensitive geography, or high-risk tier | Sanctions screening, debarment, compliance risk, due diligence adequacy |
| **Legal reviewer** | Cases involving contracts, IP, data processing, or regulated services | Contract terms, liability, IP protection, data processing agreements |
| **Tax reviewer** | International suppliers, or suppliers with tax classification questions | W-8/W-9 validation, withholding requirements, tax treaty applicability |
| **AP onboarding specialist** | All cases (final step before activation) | Banking details validation, payment terms setup, vendor master entry |
| **Procurement manager** | Spend above elevated threshold, or cases escalated by category manager | Spend authority, strategic alignment, exception approval |

### Step 3: Determine Spend Tier and Authority

Based on estimated annual spend:

| Spend Tier | Threshold | Additional Requirements |
|-----------|-----------|----------------------|
| **Tier 1** | Below standard threshold | Category manager approval sufficient |
| **Tier 2** | Above standard, below elevated threshold | Category manager + procurement manager approval |
| **Tier 3** | Above elevated threshold | Category manager + procurement manager + VP/director approval |

If estimated spend is not available, flag for the analyst to confirm before routing.

### Step 4: Determine Review Timeline

Based on case priority and SLA:

| Priority | Review Deadline |
|----------|----------------|
| **Expedited** | 1 business day per reviewer |
| **Standard** | 2 business days per reviewer |

If the review timeline would extend past the SLA target date, flag as a timeline risk.

### Step 5: Resolve Reviewer Identities

- `SearchPeople` — resolve each required reviewer by role and specialty
- `GetUserDetails` — verify reviewer availability and role
- `GetManagerDetails` / `GetDirectReportsDetails` — resolve escalation paths for elevated spend tiers

If no clear reviewer is found for a required review type, escalate to the procurement operations manager for manual assignment.

### Step 6: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Supplier Name, Spend Category, Geography, Risk Tier, Estimated Spend
- **Required reviewers** — each reviewer with role, name, review scope, and routing rationale
- **Spend tier** — tier level with authority requirements
- **Review timeline** — target completion date per reviewer based on priority
- **Timeline risk** — flag if review completion would exceed SLA target
- **Review sequence** — any ordering requirements (e.g., risk review before legal, legal before AP)
- **High-risk flags** — if sanctions or compliance flags are present, require category manager acknowledgment before routing
- **Draft label** — "ROUTING RECOMMENDATION — analyst confirmation required before sending notifications"

### Step 7: Execute Routing (After Confirmation)

**Send Teams notifications to each reviewer:**
- `PostMessage` — direct message to each reviewer with:
  - Case ID and Supplier Name
  - Review scope (what they are reviewing)
  - Review deadline
  - Link to the supplier onboarding folder in SharePoint
  - Risk flags summary (if applicable)

**Create review deadline calendar holds:**
- `CreateEvent` — calendar reminder for each reviewer with the review deadline

**Update the onboarding tracker:**
- Record: Assigned Reviewers, Review Path, Routing Date, Review Deadlines, Status updated to "In Review"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, approval matrix, routing rules |
| ReadFileContent | Read approval matrix, routing rules, tracker |
| SearchPeople | Resolve reviewer identities by role and specialty |
| GetUserDetails | Verify reviewer availability and role |
| GetManagerDetails / GetDirectReportsDetails | Resolve escalation paths for elevated spend tiers |
| PostMessage | Send Teams notifications to reviewers (after confirmation) |
| CreateEvent | Create review deadline calendar holds |

## Guardrails

- **Present routing recommendations for analyst review** before sending any notifications — routing decisions are confirmed before execution
- **Never auto-assign reviewers outside the documented approval matrix** — reviewer assignment must follow the defined rules
- **For high-risk suppliers (sanctions flags, high-spend categories)**, require explicit category manager approval before routing — high-risk cases need acknowledgment before reviewer notification
- **Include spend authority threshold context** — if estimated spend exceeds a tier boundary, flag for elevated review path; do not let a high-spend supplier go through a low-tier review
- **Escalate to procurement operations manager** if no clear reviewer is found for a required review type — never leave a required review unassigned
- **Never skip a required reviewer** even if the case appears straightforward — the approval matrix defines minimum review requirements
- **Flag timeline risks prominently** — if the review timeline would extend past the SLA target, the analyst needs to know immediately
- **Include risk flags in every reviewer notification** so reviewers can prioritize effectively — a reviewer needs to know if there are sanctions concerns before starting their review
- **Never include bank account details or tax identifiers in Teams notifications** — reference the onboarding folder for sensitive documents
- **Log every routing decision** with rationale, assigned reviewers, deadlines, and confirming analyst for audit trail
- **Track review sequence requirements** — if risk review must complete before legal review, route in order and note the dependency
