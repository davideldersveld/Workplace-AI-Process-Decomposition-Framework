---
name: comms-review-routing
description: |
  Routes a communications brief to the required reviewers — legal, HR, executive communications,
  media relations, and business sponsor — based on the review routing matrix.
  Use when user asks to "route brief for review", "send for legal review",
  "submit brief to executive review", "request HR review of announcement",
  "route [Brief ID] for approval", "send comms brief for review",
  "who needs to review [announcement]", or "set up review chain for [brief]".
  Do NOT use for assembling context (use comms-context-packet),
  extracting message themes (use comms-message-extraction),
  drafting the brief (use comms-brief-draft),
  or creating a new case (use comms-request-intake).
---

## Overview

Reads the review routing matrix and approval rules from SharePoint, cross-references against the brief case data (communication type, sensitivity level, audience, channels), and routes the brief to the correct set of reviewers. Creates Teams direct messages for review notifications, Outlook draft emails for formal review requests, and calendar holds for review deadlines.

This skill operates in "AI act within policy" mode — routing is determined strictly by the approved review routing matrix. The communications manager reviews and confirms the routing recommendation before any messages are sent.

## When to Use

- A communications brief has been drafted and is ready for review routing
- The user wants to determine which reviewers are required for a specific announcement
- The brief needs to be sent to specific reviewers (legal, HR, executive, sponsor)
- Review deadlines need to be set and communicated

## When NOT to Use

- Assembling business and policy context — use comms-context-packet
- Extracting message themes and risks — use comms-message-extraction
- Drafting the communications brief — use comms-brief-draft
- Creating a new case — use comms-request-intake
- Making the final brief disposition — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read brief case data and routing matrix", activeForm="Reading routing inputs")
TaskCreate(subject="Determine required reviewers and deadlines", activeForm="Determining review chain")
TaskCreate(subject="Present routing recommendation for approval", activeForm="Preparing recommendation")
TaskCreate(subject="Send review notifications", activeForm="Sending review notifications")
```

### Step 1: Read Routing Inputs

Locate and read required inputs:

- **Brief case data** — from the brief tracker: `SearchM365(sources=["files"], query="communications brief tracker")`
- **Draft brief** — from comms-brief-draft output: `SearchM365(sources=["files"], query="brief [Brief ID]")`
- **Review routing matrix** — `SearchM365(sources=["files"], query="review routing matrix")` or `SearchM365(sources=["files"], query="communications approval matrix")`
- **Message themes and risk flags** — from comms-message-extraction output (for sensitivity-based routing)

Read each document using `ReadFileContent`.

### Step 2: Determine Required Reviewers and Deadlines

Apply the routing matrix to determine the review chain based on announcement characteristics:

| Announcement Characteristic | Required Reviewers |
|----------------------------|-------------------|
| All announcements | Communications lead + business sponsor |
| Legal topics (litigation, regulatory, compliance) | + Legal reviewer |
| People matters (organizational change, leadership changes, layoffs) | + HR reviewer |
| Executive-authored or representing company externally | + Executive communications reviewer |
| Announcements with media exposure | + Media relations reviewer |
| Crisis or time-sensitive | All applicable reviewers + shortened deadlines |
| Confidential sensitivity level | All applicable reviewers + restricted distribution |

**Determine review deadlines** based on announcement date and SLA:
- Standard announcements: review due 2 business days before announcement date
- Time-sensitive or crisis: review due within 4 hours of routing
- If no announcement date set: review due within 3 business days of routing

**Determine review scope** — what each reviewer is specifically reviewing for:
- **Legal reviewer:** claims accuracy, regulatory compliance, litigation risk, liability exposure
- **HR reviewer:** employee impact, fair treatment, labor relations implications, privacy concerns
- **Executive communications:** tone, strategic alignment, executive positioning, external perception
- **Media relations:** media readiness, potential media questions, external messaging consistency
- **Business sponsor:** factual accuracy, business context, key message alignment, timing confirmation

### Step 3: Present Routing Recommendation

Present the recommendation via Adaptive Card (invoke `render-ui` skill first):

- **Brief summary** — Brief ID, announcement title, communication type, sensitivity level
- **Required reviewers** — name, role, review scope, deadline
- **Routing rationale** — why each reviewer is required (linked to announcement characteristics)
- **Sensitivity handling** — whether Teams direct messages (for sensitive) or channel coordination (for standard) will be used
- **Escalation note** — if the routing matrix produces no clear path for an announcement characteristic, flag for communications leadership

After user confirms the recommendation, proceed to Step 4.

### Step 4: Send Review Notifications

After user confirmation, send notifications:

**Teams direct messages** (for sensitive announcements — use `PostMessage`):
- Direct message to each reviewer with: Brief ID, announcement title (not full content), review scope, deadline, link to brief in SharePoint
- Never post sensitive announcement details in channel-wide Teams messages

**Outlook draft emails** (for formal review chain — use `CreateDraftMessage`):
- Draft email to each reviewer with: brief attached, review scope, deadline, response instructions
- For legal and HR reviews: include sensitivity classification and relevant risk flags

**Calendar holds** (for review deadlines — use `CreateEvent`):
- Create review deadline event for each reviewer with brief link and review scope
- For crisis/time-sensitive: create shorter-window holds with explicit urgency

**Update the brief tracker:**
- Note that the brief has been routed, with reviewer list, deadlines, and routing timestamp

### Review Summary Generation

When reviewers have responded (via email or Teams), this skill can also synthesize reviewer feedback:

- Search for review responses: `SearchM365(sources=["email"], query="[Brief ID] review")` and `SearchM365(sources=["teams"], query="[Brief ID]")`
- Consolidate feedback into a structured revision list
- Present via Adaptive Card: approvals received, revisions requested, outstanding reviews
- Flag any reviewer who has not responded as the deadline approaches

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find brief tracker, routing matrix, draft brief, message themes |
| ReadFileContent | Read routing matrix, approval rules, brief data |
| SearchPeople / GetUserDetails | Resolve reviewer identities by role |
| GetManagerDetails / GetDirectReportsDetails | Executive chain for escalation paths |
| SearchM365 (email, teams) | Find reviewer responses for review summary |
| PostMessage | Teams direct messages to reviewers (after approval) |
| CreateDraftMessage | Outlook draft emails for formal review requests (after approval) |
| CreateEvent | Calendar holds for review deadlines |

## Guardrails

- **Present routing recommendation for review** before sending any messages — communications manager must confirm
- **Route based strictly on the approval matrix** — communication type, sensitivity level, and audience determine which reviewers are required
- **Never skip a required reviewer** even if the announcement appears routine
- **Use Teams direct messages for sensitive announcements** — never channel posts for embargoed or confidential content
- **Escalate to communications leadership** if the routing matrix produces no clear path for an announcement characteristic
- **Include sensitivity level, audience scope, and timing** in every review request so reviewers can prioritize
- **Never disclose embargoed content** to anyone not on the approved reviewer list
- **Never present the brief as "review complete"** until all required reviewers have responded — partial status must be clearly visible
- **Log all routing actions** — record reviewer notifications with timestamp, actor, and rationale
