---
name: compliance-case-intake
description: |
  Normalizes incoming compliance cases — policy exceptions, hotline reports, control issues,
  and suspected violations — into structured case records in the compliance tracker.
  Use when user asks to "new compliance case", "policy exception request from [name]",
  "log compliance intake", "hotline report received", "open compliance case for",
  "new control issue from", "register a compliance matter",
  "intake a policy exception", or "create compliance case for [topic]".
  Do NOT use for assembling policy context (use compliance-context-packet),
  detecting risk indicators (use compliance-risk-detection),
  routing for review (use compliance-review-routing),
  or drafting case communications (use compliance-case-comms).
---

## Overview

Converts an incoming compliance event — policy exception request, hotline report, control issue, or suspected violation — into a normalized case record. Checks for duplicates, validates required fields, supports anonymous reporting, and writes the case to the shared compliance case tracker in SharePoint.

This is the entry point for the compliance case triage workflow. Every downstream skill depends on a case record created here.

## When to Use

- A policy exception request has been submitted
- A hotline or intake report has been received
- A control issue has been identified and needs to be logged
- A suspected compliance violation needs to be formalized as a case

## When NOT to Use

- Assembling policy and control context for an existing case — use compliance-context-packet
- Detecting risk indicators or evidence gaps — use compliance-risk-detection
- Routing a case for review — use compliance-review-routing
- Drafting case communications — use compliance-case-comms

## Core Instructions

### Progress Tracking

At the start, create tasks:

```
TaskCreate(subject="Gather case details", activeForm="Gathering case details")
TaskCreate(subject="Check for duplicate or related cases", activeForm="Checking for duplicates")
TaskCreate(subject="Create compliance case record", activeForm="Creating case record")
```

Mark each task `in_progress` when starting and `completed` when done.

### Step 1: Gather Case Details

Collect the following from the user's message, email context, or by asking:

| Field | Required | Source |
|-------|----------|--------|
| Case type (policy exception, suspected violation, control issue, third-party risk) | Yes | User input |
| Reporter or requestor name/email | Yes (may be "Anonymous") | User input, email sender, or SearchPeople |
| Affected policy or control area | Yes | User input |
| Business unit | Yes | User input or GetUserDetails |
| Legal entity | No | User input |
| Urgency or initial severity | No | User input (default: Medium) |
| Supporting documentation | No | Attached files or SearchM365(sources=["files"]) |
| Description of the issue | Yes | User input or email body |

**If the case originated from an email**, search for it:
- `SearchM365(sources=["email"], query="<case keywords>")` to find the intake email
- `GetMessage(message_id=...)` to read the full email

**Resolve reporter identity (when not anonymous):**
- `SearchPeople(query="<reporter name>")` to confirm identity
- `GetUserDetails(user_id="<email>")` to get profile and department

**For anonymous reports:** Record Reporter as "Anonymous" — do not attempt to identify the reporter through any means.

If critical fields are missing (case type, affected policy, description), ask the user once, covering all gaps.

### Step 2: Check for Duplicates

Before creating a new case, check the compliance case tracker:

- `SearchM365(sources=["files"], query="compliance case tracker")` to locate the tracker
- `ReadFileContent(...)` to read the tracker and check for matching cases within the last 30 days (same policy area, same business unit, similar description)

If a potential duplicate is found, present it to the user: "I found a similar case logged recently — [Case ID, type, affected policy]. Should I create a new case, or link this to the existing one?"

### Step 3: Create Compliance Case Record

Generate a case record with these fields:

| Column | Value |
|--------|-------|
| Case ID | Auto-generated (format: CR-YYYY-NNN) |
| Case Type | Policy exception / Suspected violation / Control issue / Third-party risk |
| Reporter | Resolved name and email, or "Anonymous" |
| Affected Policy | From user input |
| Business Unit | From user input or reporter profile |
| Legal Entity | From user input or "TBD" |
| Severity | From user input or "Medium" |
| Status | New |
| Received Date | Today's date |
| Assigned Analyst | Current user (from GetMyDetails) |
| Evidence Status | Pending |
| Disposition | Open |
| Audit Log | "Case created by [user] on [date] from [source]" |

**Present the case record to the user for confirmation before writing.** Show it as an Adaptive Card with all fields.

After confirmation, write the record to the tracker spreadsheet.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 | Find intake emails, existing tracker, related cases |
| GetMessage | Read full email content for case details |
| SearchPeople / GetUserDetails | Resolve reporter and control owner identities |
| GetMyDetails | Get current user info for Assigned Analyst field |
| ReadFileContent | Read existing tracker to check duplicates |

## Guardrails

- **Support anonymous reporting** — never attempt to identify anonymous reporters through any means
- **Never create duplicate cases** for the same issue within 30 days without explicit user confirmation
- **Validate required fields** before writing (case type, affected policy, description, reporter or "Anonymous")
- **Confirm with user** before writing to the tracker — present the full record for review
- **Log every action** — record case creation with timestamp, creating user, and source reference in the Audit Log column
- **Never fabricate case details** — if information is missing, mark it as "TBD" or ask the user
