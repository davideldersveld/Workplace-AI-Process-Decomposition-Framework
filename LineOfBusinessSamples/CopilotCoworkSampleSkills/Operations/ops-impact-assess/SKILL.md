---
name: ops-impact-assess
description: |
  Assesses operational impact, priority, aging risk, and escalation need
  for a classified exception case.
  Use when user asks to "assess impact of exception [ID]",
  "how urgent is [case]", "priority recommendation for [exception]",
  "aging risk for [case]", "what's the exposure on this",
  "recommend priority for [exception]",
  or "escalation check for [case ID]".
  Do NOT use for creating a new exception case (use ops-exception-intake),
  gathering transaction context (use ops-context-packet),
  classifying exception type (use ops-classify-exception),
  routing to an owner or queue (use ops-route-exception),
  or drafting follow-up communications (use ops-exception-comms).
---

## Overview

Assesses the operational impact, priority level, aging risk, and escalation need for a classified exception case. Evaluates downstream effects on dependent transactions, customer or stakeholder exposure, SLA compliance risk, and financial thresholds against documented priority rules and escalation checklists. Presents the assessment as a recommendation for analyst approval.

This skill operates in "AI draft plus approve" mode — the impact assessment and priority recommendation are presented for analyst review. The analyst confirms or adjusts before the priority is recorded and downstream actions are taken.

## When to Use

- An exception has been classified and needs priority and impact assessment
- An analyst needs to determine urgency and escalation requirements
- A case needs reassessment after scope change, new information, or aging
- A queue manager wants to review priority distribution across the queue

## When NOT to Use

- Creating a new exception case — use ops-exception-intake
- Gathering process and transaction context — use ops-context-packet
- Classifying exception type and likely cause — use ops-classify-exception
- Assigning an owner or routing to a queue — use ops-route-exception
- Drafting follow-up or handoff communications — use ops-exception-comms
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and priority rules", activeForm="Reading impact assessment inputs")
TaskCreate(subject="Present impact and priority recommendation", activeForm="Assessing exception impact")
```

### Step 1: Read Assessment Inputs

**Read the exception case and classification:**
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent` — case data, exception type, classification, safety flag, aging, reopen count

**Read priority rules and escalation policy:**
- `SearchM365(sources=["files"], query="priority rules")` then `ReadFileContent` — priority definitions, criteria, and thresholds
- `SearchM365(sources=["files"], query="escalation checklist")` then `ReadFileContent` — escalation triggers and approval requirements
- `SearchM365(sources=["files"], query="SLA definitions")` then `ReadFileContent` — SLA targets by priority level

### Step 2: Assess Downstream Impact

Evaluate the scope of the exception's effects:

**Transaction impact:**
- How many transactions are affected (single vs. batch)?
- Are there dependent or downstream transactions that are blocked or degraded?
- Is the exception affecting a currently active process or a completed one?

**Customer or stakeholder exposure:**
- Is a customer directly affected (order delayed, service disrupted, quality issue)?
- Is an internal stakeholder blocked (dependent team waiting, report delayed)?
- What is the exposure window (how long has the impact been active)?

**Operational impact:**
- Is a queue or process line blocked?
- Are other analysts or resolvers waiting on this exception?
- Is this exception contributing to a larger pattern or outage?

If a Graph Connector is available:
- `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — downstream impact data, affected processes, dependent transactions

### Step 3: Determine Priority Level

Apply the documented priority rules:

| Priority | Criteria | SLA Deadline |
|----------|---------|-------------|
| **Critical** | Safety exception, compliance violation, major customer impact, process line blocked, financial exposure above threshold | 30 minutes |
| **High** | Multiple transactions affected, customer-facing impact, approaching regulatory deadline, dependent team blocked | 2 hours |
| **Medium** | Single transaction affected, internal process delay, no immediate customer impact, workaround available | 4 hours |
| **Low** | Minor discrepancy, no downstream impact, informational exception, can be batched | 1 business day |

### Step 4: Assess Aging Risk

Calculate aging status:
- **Time since creation** — hours elapsed since Case Created Date
- **SLA percentage consumed** — time elapsed as percentage of SLA deadline for the recommended priority
- **Aging risk level:**
  - Green: below 50% of SLA consumed
  - Yellow: 50-80% of SLA consumed
  - Red: above 80% of SLA consumed — requires explicit urgency callout
  - Breached: past SLA deadline — requires immediate escalation recommendation

### Step 5: Determine Handling Path

Based on priority and exception type:

| Handling Path | When to Recommend |
|--------------|-------------------|
| **Immediate resolution** | Critical priority, clear resolution path, assigned resolver available |
| **Standard queue** | Medium or low priority, standard exception type, no special expertise needed |
| **Specialist referral** | Complex exception requiring domain expertise, non-standard resolution |
| **Escalation** | Safety exception, SLA breach, financial threshold exceeded, circular routing detected, reopened exception with prior failed resolution |

### Step 6: Apply Safety and Financial Threshold Rules

**Safety-critical exceptions:** Regardless of calculated priority, safety exceptions (quality defects, compliance violations, equipment failures) always receive an escalation recommendation. Safety exceptions bypass standard priority logic.

**Financial threshold:** For exceptions involving financial transactions above the defined organizational threshold, flag for team lead review regardless of calculated priority.

**Reopened exceptions:** If the reopen count is greater than zero, elevate the handling path by one level (e.g., standard queue becomes specialist referral) since a prior resolution attempt failed.

### Step 7: Present Impact Assessment

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Transaction Reference, Exception Type, Classification Confidence, Aging
- **Priority recommendation** — Critical, High, Medium, or Low with policy citation
- **Aging risk indicator** — Green, Yellow, Red, or Breached with time remaining
- **Downstream impact summary** — affected transactions, customer exposure, operational impact
- **Recommended handling path** — immediate resolution, standard queue, specialist referral, or escalation
- **Escalation recommendation** — if aging, safety, financial threshold, or reopened status triggers escalation
- **Safety flag** — prominently displayed if safety-first rule applies
- **Financial threshold flag** — if transaction amounts exceed the review threshold
- **Priority rules criteria** — the specific criteria used so the analyst can validate
- **Draft label** — "IMPACT ASSESSMENT — analyst review required before priority is recorded"

### Step 8: Record Priority (After Confirmation)

After analyst confirmation:
- Update the exception tracker with: Priority, SLA Deadline (calculated from priority level), Handling Path, Escalation Flag, Assessment Date, Assessing Analyst
- If the analyst overrides the recommendation, log the override with original recommendation, analyst's decision, and rationale

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, priority rules, escalation checklist, SLA definitions |
| SearchM365 (connectors) | Pull downstream impact data via Graph Connector |
| ReadFileContent | Read priority rules, escalation checklist, SLA definitions, tracker |

## Guardrails

- **Present priority and impact as a recommendation** requiring analyst approval — never auto-set priority in the tracker
- **Never auto-close, auto-resolve, or auto-reroute** based on impact assessment alone — these are irreversible disposition changes requiring human decision
- **For safety-critical exceptions, always recommend escalation** regardless of calculated priority level — safety exceptions bypass standard priority logic
- **Flag any exception aging beyond 80% of its SLA window** with explicit urgency callout — the analyst must know when time is running out
- **Include the priority rules criteria used** so the analyst can validate the recommendation against documented policy
- **For exceptions involving financial transactions above the defined threshold**, flag for team lead review regardless of calculated priority
- **For reopened exceptions**, elevate the handling path since a prior resolution attempt failed — do not treat reopened cases the same as new ones
- **Never fabricate downstream impact data** — if impact data is unavailable from source systems, state the gap rather than estimating
- **Log every priority assessment and override** for process improvement — override patterns reveal where priority rules need refinement
- **Never assert that an exception is low-risk without evidence** — absence of impact data does not equal low impact
