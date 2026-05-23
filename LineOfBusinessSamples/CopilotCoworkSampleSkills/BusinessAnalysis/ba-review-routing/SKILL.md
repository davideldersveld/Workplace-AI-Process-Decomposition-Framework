---
name: ba-review-routing
description: |
  Routes the requirements review packet to stakeholders for feedback and sign-off.
  Sends Teams notifications and email review requests to assigned reviewers.
  Use when user asks to "send for review", "route requirements to reviewers",
  "request sign-off", "send package to [stakeholder]",
  "distribute review packet", "notify reviewers", "send review requests",
  "route for approval", or "send requirements for feedback".
  Do NOT use for building the review packet (use ba-traceability-packet),
  drafting requirements (use ba-requirements-draft),
  or creating a new case (use ba-request-intake).
---

## Overview

Routes the completed review and traceability packet to assigned stakeholders for feedback and sign-off. Sends Teams messages for routine notifications and creates Outlook email drafts for formal review requests. Updates the request tracker with review status.

This skill operates in "AI act within policy" mode — routing is executed when it follows the approved reviewer registry and workflow rules. All communications include the case ID and request reference.

## When to Use

- The review and traceability packet is complete and approved by the analyst
- The user wants to send the requirements package to reviewers
- Distributing review materials and setting review deadlines

## When NOT to Use

- Building the review packet — use ba-traceability-packet
- Drafting requirements — use ba-requirements-draft
- Synthesizing themes — use ba-theme-synthesis
- Making the final baseline approval decision — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Verify review readiness and resolve reviewers", activeForm="Verifying readiness")
TaskCreate(subject="Send review notifications", activeForm="Sending notifications")
TaskCreate(subject="Update tracker with review status", activeForm="Updating tracker")
```

### Step 1: Verify Review Readiness

Before routing, confirm:

1. **Traceability packet exists** — search for it: `SearchM365(sources=["files"], query="traceability [case]")`
2. **No blocking items** — read the review summary to check for unresolved blocking items
3. **Reviewers are identified** — read the stakeholder roster from the review summary

If blocking items exist (unresolved critical conflicts, traceability coverage below threshold), stop and inform the user: "The review packet has unresolved blocking items. These should be addressed before routing."

### Step 2: Resolve Reviewers

For each reviewer in the stakeholder roster:
- `SearchPeople(query="<reviewer name>")` to resolve identity
- `GetUserDetails(user_id="<email>")` to confirm contact details

Verify all reviewers are in the approved stakeholder roster. If a reviewer cannot be resolved, flag it to the user.

### Step 3: Send Review Communications

**Teams Notifications (routine updates):**

For each reviewer, send a Teams message using `PostMessage`:

```
Subject: Requirements Review Request — [Case ID]

A requirements package is ready for your review.

Case: [Case ID] — [Request description]
Documents: [Link to review packet location]
Review deadline: [Date — minimum 2 business days from today]

Please review the requirements package and traceability matrix and provide your feedback by the deadline.
```

**Outlook Drafts (formal review requests):**

For formal review requests, create email drafts using `CreateDraftMessage`:

- To: Reviewer email
- Subject: "Requirements Review Request — [Case ID]: [Request title]"
- Body: Formal review request with case summary, document references, review deadline, and expected feedback format

Present the draft to the user: "I've created an email draft for [reviewer]. Review and send when ready."

**Calendar Holds (optional):**

If the user requests it, create review deadline calendar events using `CreateEvent`:
- Subject: "Review Deadline: [Case ID] Requirements"
- Date: Review deadline date
- Attendees: Reviewer email

### Step 4: Update Tracker

After routing is complete:
- Note the review status, reviewer names, and sent date for updating the request tracker
- Present a summary to the user of what was sent and to whom

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find traceability packet and review summary |
| ReadFileContent | Read review summary to check for blocking items |
| SearchPeople / GetUserDetails | Resolve reviewer identities and contact details |
| PostMessage | Send Teams review notifications |
| CreateDraftMessage | Create formal email review requests as drafts |
| CreateEvent | Create review deadline calendar holds |

## Guardrails

- **Verify reviewers** — all reviewers must be in the approved stakeholder roster before sending
- **Include case reference** — every communication must include the Analysis Case ID and request description
- **Minimum review period** — review deadlines must be at least 2 business days from the send date
- **Do not route with blocking items** — if the traceability packet flags unresolved blocking items, do not proceed
- **Formal requests as drafts** — create Outlook review requests as drafts so the analyst can review before sending; Teams status updates may be sent directly
- **Never approve on behalf of reviewers** — this skill routes for review; it does not confirm sign-off
