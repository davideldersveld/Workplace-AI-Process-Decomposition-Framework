---
name: audit-request-routing
description: |
  Routes evidence requests and review assignments to the appropriate control owners
  and reviewers based on the reviewer matrix, control ownership records, and org hierarchy.
  Use when user asks to "route evidence requests for [case]", "who owns [control area]",
  "assign evidence requests for [case ID]", "set up review chain for audit",
  "determine who needs to provide evidence", "send evidence requests",
  "notify control owners for [engagement]", or "escalate overdue evidence".
  Do NOT use for creating a new audit case (use audit-request-intake),
  assembling scope and evidence context (use audit-evidence-packet),
  detecting evidence gaps (use audit-gap-detection),
  or drafting audit summaries (use audit-case-comms).
---

## Overview

Determines the correct evidence request recipients and review chain for an audit engagement based on the reviewer matrix, control ownership records, and organizational hierarchy. Resolves control owners, assigns evidence requests, identifies required reviewers, defines escalation paths, and sets timeline recommendations. Sends Teams notifications and creates calendar holds after lead auditor approval.

This skill operates in "AI draft plus approve" mode — routing recommendations are presented for lead auditor review before any messages are sent or assignments made.

## When to Use

- Evidence gaps have been identified and requests need to be routed to control owners
- The lead auditor needs to know who owns each control area for evidence collection
- Evidence requests need to be sent to the right people with appropriate deadlines
- Overdue evidence requests need escalation to management
- The review chain needs to be established for an engagement

## When NOT to Use

- Creating a new audit case — use audit-request-intake
- Assembling scope and evidence context — use audit-evidence-packet
- Detecting evidence gaps — use audit-gap-detection
- Drafting detailed audit communications — use audit-case-comms
- Confirming audit packet disposition — this is always a human decision (IA-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read gap report and resolve owners", activeForm="Resolving evidence request owners")
TaskCreate(subject="Present routing recommendation", activeForm="Preparing routing recommendation")
```

### Step 1: Read Gap Report and Engagement Context

**Read inputs:**
- **Gap report** — from audit-gap-detection output or the evidence checklist with current status
- **Engagement data** — Case ID, process area, audit period, lead auditor, audit manager from the case tracker
- `SearchM365(sources=["files"], query="audit case tracker")` then `ReadFileContent`

**Read routing references:**
- `SearchM365(sources=["files"], query="reviewer matrix")` then `ReadFileContent` — who reviews what by engagement type and severity
- `SearchM365(sources=["files"], query="control ownership [process area]")` then `ReadFileContent` — who owns each control in the scope area
- `SearchM365(sources=["files"], query="audit escalation path")` then `ReadFileContent` — escalation rules for overdue or unresponsive owners

### Step 2: Resolve Owners and Reviewers

**Resolve control owners:**
- For each evidence gap or request item, identify the control owner from the control ownership records
- `SearchPeople` — resolve each control owner by name or role
- `GetUserDetails` — confirm identity and email
- `GetManagerDetails` / `GetDirectReportsDetails` — determine escalation path if control owner is unresponsive

**Determine required reviewers:**
- Based on engagement type and gap severity from the reviewer matrix:
  - Standard engagements: lead auditor reviews evidence, audit manager reviews packet
  - High-severity gaps: audit manager reviews evidence requests before sending
  - Critical-severity gaps: audit director or chief audit executive notified

**Check for conflicts:**
- Verify that no evidence request is routed to an individual who is also under audit for the same engagement
- Verify that the evidence requestor is not also the evidence approver (segregation of duties)
- Flag any conflicts for lead auditor resolution

### Step 3: Build Routing Recommendation

For each evidence request, prepare:

- **Control owner** — name, email, role, department
- **Evidence items requested** — specific checklist items with descriptions
- **Submission deadline** — based on the audit timeline (typically 5 business days from request, adjusted for testing start date)
- **Required format** — document types expected
- **Review chain** — who reviews the submitted evidence after receipt
- **Escalation trigger** — when and to whom to escalate if no response (typically 3 business days)

**Timeline recommendations:**
- Set initial deadlines relative to the testing start date
- Allow at least 2 business days between evidence submission deadline and testing start
- Flag requests where the timeline is compressed (less than 5 business days total)

### Step 4: Present for Approval

Present the full routing recommendation via Adaptive Card (invoke `render-ui` skill first):

- **Engagement header** — Case ID, Process Area, Audit Period
- **Routing table** — each evidence request with: control owner, items, deadline, review chain
- **Conflicts or concerns** — any segregation-of-duties flags or unresolved ownership
- **Timeline summary** — compressed timelines, escalation triggers
- **Recommended actions** — "Send evidence requests via Teams" / "Create calendar deadline reminders" / "Escalate to [manager]"

### Step 5: Execute After Approval

After the lead auditor confirms the routing:

**Send Teams notifications:**
- `PostMessage` — notify each control owner with their specific evidence request items, deadlines, and submission instructions
- Keep messages factual and concise — include engagement reference (Case ID) and evidence folder location, but no audit observations or preliminary findings

**Create calendar deadline reminders:**
- `CreateEvent` — for each critical-path evidence submission deadline, create a calendar reminder for the lead auditor

**Log routing decisions:**
- Record each routing assignment in the case tracker with timestamp, control owner, items assigned, deadline, and the auditor who approved the routing

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find reviewer matrix, control ownership records, escalation paths, case tracker |
| ReadFileContent | Read routing references and engagement data |
| SearchPeople / GetUserDetails | Resolve control owners, process owners, business liaisons |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths and control accountability |
| PostMessage | Teams notifications to control owners (after approval) |
| CreateEvent | Calendar deadline reminders for critical evidence items |

## Guardrails

- **Present routing recommendation for lead auditor review** before sending any messages or creating any assignments
- **Never auto-assign outside the approved reviewer matrix** and control ownership records — routing must follow documented ownership
- **Enforce segregation of duties** — flag and block routing where the evidence requestor would also approve, or where a control owner is asked to provide evidence for their own control effectiveness assessment
- **Preserve audit independence** — never route evidence requests through individuals who are also under audit for the same engagement
- **Never reveal audit findings or observations** in routing messages — use engagement IDs, evidence checklist item references, and pointers to the SharePoint engagement folder only
- **Escalate to audit manager** if critical evidence gaps are identified or control owners cannot be determined from the control ownership records
- **Log all routing decisions** to the case tracker with timestamp, actor, and rationale for audit trail purposes
- **Respect data classification** — evidence request notifications go only to identified control owners and audit team members; no routing messages to non-audit recipients
