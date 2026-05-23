---
name: compliance-review-routing
description: |
  Recommends the review path, severity classification, and reviewer assignments for a compliance case
  based on the severity matrix, routing rules, and risk indicators.
  Use when user asks to "route this case for review", "who reviews [case type]",
  "assign compliance reviewers", "determine severity and routing for",
  "set up review chain for case", "escalate this compliance case",
  "who needs to review [case ID]", or "compliance routing for [case]".
  Do NOT use for assembling context (use compliance-context-packet),
  detecting risk indicators (use compliance-risk-detection),
  drafting case communications (use compliance-case-comms),
  or creating a new case (use compliance-case-intake).
---

## Overview

Reads the severity matrix and routing rules from SharePoint, cross-references against case data and risk indicators, and recommends the review path — assigned analyst, required reviewers, severity classification, escalation path, and SLA deadline. All recommendations are presented for compliance operations review before any messages are sent.

This skill operates in "AI draft plus approve" mode — the routing recommendation is presented for user review and confirmation before any notifications or assignments are sent.

## When to Use

- A compliance case has risk indicators and needs to be routed to the correct reviewers
- The user wants to determine severity and assign reviewers based on routing rules
- Escalating a case to compliance leadership, legal, or internal audit

## When NOT to Use

- Assembling policy context — use compliance-context-packet
- Detecting risk indicators — use compliance-risk-detection
- Drafting case communications — use compliance-case-comms
- Creating a new case — use compliance-case-intake
- Making the final triage disposition — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data, risk indicators, and routing rules", activeForm="Reading routing inputs")
TaskCreate(subject="Determine severity and review path", activeForm="Determining review path")
TaskCreate(subject="Present routing recommendation for approval", activeForm="Preparing recommendation")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Case data** — from the compliance case tracker: `SearchM365(sources=["files"], query="compliance case tracker")`
- **Risk indicator report** — from compliance-risk-detection output or uploaded file
- **Severity matrix** — from SharePoint: `SearchM365(sources=["files"], query="compliance severity matrix")`
- **Routing rules** — from SharePoint: `SearchM365(sources=["files"], query="compliance routing rules")`
- **Reviewer assignments** — from SharePoint: `SearchM365(sources=["files"], query="compliance reviewer assignments")`

Read each document using `ReadFileContent`.

### Step 2: Determine Severity and Review Path

Apply the severity matrix to classify the case:

| Factor | How It Affects Severity |
|--------|------------------------|
| Case type | Policy exceptions vs. suspected violations have different base severity |
| Risk indicators | High or critical indicators elevate severity |
| Regulatory flags | Any regulatory-reportable indicator sets minimum severity to High |
| Multi-entity scope | Cross-entity cases elevate severity |
| Repeat pattern | Repeat exceptions in same area elevate severity |
| Evidence gaps | Critical missing evidence may elevate severity |

**Determine required reviewers** based on routing rules:

| Severity | Required Reviewers |
|----------|-------------------|
| Low | Assigned compliance analyst |
| Medium | Compliance analyst + control owner |
| High | Compliance analyst + control owner + compliance manager + legal reviewer |
| Critical | All of the above + internal audit liaison + compliance leadership |

**Determine escalation path:**
- High severity: compliance manager must be notified
- Critical severity: compliance leadership and legal must be notified immediately
- Control failure indicators: internal audit liaison must be included
- Regulatory-reportable indicators: compliance leadership must be notified with regulatory flag

**Determine SLA deadline** based on case type and severity:
- Critical: 4 hours
- High: 1 business day
- Medium: 2 business days
- Low: 5 business days

### Step 3: Present Routing Recommendation

Present the recommendation via Adaptive Card (invoke `render-ui` skill first):

- **Severity classification** with supporting rationale
- **Assigned analyst** (from routing rules or current assignment)
- **Required reviewers** with names and roles
- **Escalation notifications** (if severity is High or Critical)
- **SLA deadline** with date and time
- **Segregation of duties check** — flag if the assigned analyst is also the control owner

After user confirms the recommendation:

**Send notifications:**
- `PostMessage` — Teams notification to assigned reviewers with case ID and link to case folder (no case substance in the message)
- `CreateDraftMessage` — Outlook draft for formal escalation notices (if severity is High or Critical)
- `CreateEvent` — SLA deadline calendar event for the assigned analyst

### Step 4: Log Routing Decision

Note the routing decision for updating the case tracker:
- Severity classification
- Assigned reviewers
- SLA deadline
- Routing rationale
- Timestamp and actor

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, severity matrix, routing rules, reviewer assignments |
| ReadFileContent | Read routing rules, severity matrix, case data |
| SearchPeople / GetUserDetails | Resolve reviewer identities |
| GetManagerDetails / GetDirectReportsDetails | Map escalation chain |
| PostMessage | Send Teams notifications to assigned reviewers (after approval) |
| CreateDraftMessage | Create Outlook drafts for formal escalation notices (after approval) |
| CreateEvent | Create SLA deadline calendar events |

## Guardrails

- **Present recommendation before sending** — all routing recommendations must be reviewed and confirmed by the user before any messages or assignments are sent
- **Never auto-assign outside the approved reviewer matrix** — reviewers must match the routing rules in SharePoint
- **Enforce escalation rules** — High and Critical cases must include compliance manager; Critical must include legal and leadership
- **Enforce segregation of duties** — flag if the triaging analyst is also the control owner or a required approver
- **Never reveal case substance in Teams messages** — use case IDs and links to the SharePoint case folder only
- **Log all routing decisions** — every routing action must include timestamp, actor, and rationale
- **Escalate regulatory flags** — cases with regulatory-reportable indicators must be routed to compliance leadership regardless of other severity factors
