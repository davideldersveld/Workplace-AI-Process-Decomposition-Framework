---
name: comms-request-intake
description: |
  Normalizes incoming announcement requests — organizational changes, product announcements,
  executive communications, crisis responses, policy updates, and event announcements —
  into structured brief case records in the communications tracker.
  Use when user asks to "new announcement request", "communications brief needed for [topic]",
  "log comms request", "set up announcement case for [event]",
  "new comms intake", "register announcement for [topic]",
  "open a brief case for", or "intake communications request".
  Do NOT use for assembling context (use comms-context-packet),
  extracting message themes (use comms-message-extraction),
  drafting the brief (use comms-brief-draft),
  or routing for review (use comms-review-routing).
---

## Overview

Converts an incoming announcement request — from email, executive directive, sponsor request, or internal communications intake — into a normalized brief case record. Checks for duplicates and related active announcements, validates required fields, flags communications types that require elevated review, and writes the case to the shared brief tracker in SharePoint.

This is the entry point for the corporate communications brief workflow. Every downstream skill depends on a brief case record created here.

## When to Use

- A new announcement request has been submitted by email or conversation
- An executive has requested a communications brief
- A business sponsor needs an announcement drafted
- A crisis response or organizational change needs a communications case opened

## When NOT to Use

- Assembling business, audience, and policy context — use comms-context-packet
- Extracting message themes and risk flags — use comms-message-extraction
- Drafting the communications brief — use comms-brief-draft
- Routing the brief for review — use comms-review-routing
- Making the final brief disposition — this is a human-only step

## Core Instructions

### Progress Tracking

At the start, create tasks:

```
TaskCreate(subject="Gather announcement request details", activeForm="Gathering request details")
TaskCreate(subject="Check for duplicate or related announcements", activeForm="Checking for duplicates")
TaskCreate(subject="Create brief case record", activeForm="Creating brief case record")
```

Mark each task `in_progress` when starting and `completed` when done.

### Step 1: Gather Request Details

Collect the following from the user's message, email context, or by asking:

| Field | Required | Source |
|-------|----------|--------|
| Announcement title or topic | Yes | User input |
| Requesting sponsor name and email | Yes | User input, email sender, or SearchPeople |
| Communication type | Yes | User input (organizational change, product announcement, executive communication, crisis response, policy update, event announcement) |
| Target audience (internal, external, both) | Yes | User input |
| Requested timing or announcement date | No | User input |
| Embargo status and date | No | User input |
| Sensitivity level (standard, elevated, confidential) | No | User input (default: standard) |
| Department or business unit | Yes | User input or GetUserDetails |
| Description or summary of announcement | Yes | User input or email body |
| Supporting documentation | No | Attached files or SearchM365(sources=["files"]) |

**If the request originated from an email**, search for it:
- `SearchM365(sources=["email"], query="<announcement keywords>")` to find the request email
- `GetMessage(message_id=...)` to read the full email

**Resolve sponsor identity:**
- `SearchPeople(query="<sponsor name>")` to confirm identity
- `GetUserDetails(user_id="<email>")` to get profile and department

**Auto-flag elevated review requirements:**
- Crisis response → elevated sensitivity, shortened SLA
- Organizational change → requires HR review
- Executive communication → requires executive communications review
- Any mention of litigation, regulatory matters, or M&A → confidential sensitivity, requires legal review
- Any embargo date specified → flag embargo enforcement required

**Validate timing:** If a requested date is provided, check that it allows for the minimum review cycle time (SLA: 2 business days). If insufficient time, flag for user attention.

If critical fields are missing (announcement title, communication type, target audience, description), ask the user once, covering all gaps.

### Step 2: Check for Duplicates

Before creating a new case, check the brief tracker:

- `SearchM365(sources=["files"], query="communications brief tracker")` to locate the tracker
- `ReadFileContent(...)` to read the tracker and check for matching cases within the last 60 days (same topic area, same department, similar announcement title)

If a potential duplicate or related active announcement is found, present it to the user: "I found a related announcement logged recently — [Brief ID, title, status]. Should I create a new case, or link this to the existing one?"

### Step 3: Create Brief Case Record

Generate a case record with these fields:

| Column | Value |
|--------|-------|
| Brief ID | Auto-generated (format: CB-YYYY-NNN) |
| Announcement Title | From user input |
| Sponsor | Resolved name and email |
| Department | From user input or sponsor profile |
| Communication Type | From user input |
| Target Audience | Internal / External / Both |
| Requested Date | From user input or "TBD" |
| Embargo Status | None / Active (with date) |
| Sensitivity Level | Standard / Elevated / Confidential |
| Status | New |
| Created Date | Today's date |
| Assigned Communications Manager | Current user (from GetMyDetails) |
| Required Reviewers | Auto-populated based on communication type and sensitivity |
| Review Status | Pending |
| Audit Log | "Case created by [user] on [date] from [source]" |

**Present the case record to the user for confirmation before writing.** Show it as an Adaptive Card with all fields.

After confirmation, write the record to the tracker spreadsheet.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find the request email or executive directive |
| SearchM365 (files) | Locate the brief tracker and attached documents |
| GetMessage | Read full email content for request details |
| SearchPeople / GetUserDetails | Resolve sponsor identity and department |
| GetMyDetails | Get current user info for Assigned Communications Manager field |
| ReadFileContent | Read the existing tracker to check for duplicates |
| GetDriveChildren | Browse the brief tracker location |

## Guardrails

- **Never create duplicate cases** for the same announcement topic and timing within 60 days without explicit user confirmation
- **Validate minimum review cycle** — flag if requested timing does not allow for the 2-business-day SLA
- **Confirm details with user** before writing to the tracker — present the full record for review
- **Auto-flag elevated review requirements** — crisis response, organizational change, executive communications, and anything marked sensitive or embargoed must be flagged
- **Never expose embargo dates or sensitive topic details** in channel-wide Teams posts — use direct messages only
- **Log every action** — record case creation with timestamp, creating user, and source reference in the Audit Log column
- **Never fabricate case details** — if information is missing, mark it as "TBD" or ask the user
