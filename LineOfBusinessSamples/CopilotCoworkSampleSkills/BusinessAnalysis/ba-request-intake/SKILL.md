---
name: ba-request-intake
description: |
  Normalizes incoming business requests into structured analysis cases for requirements work.
  Creates a case record with standard fields in the request tracker spreadsheet.
  Use when user asks to "new requirements request", "intake request for [project]",
  "set up analysis case for", "new change request from [stakeholder]",
  "business request for [topic]", "log a new request", "create an analysis case",
  "intake a new enhancement request", or "register a business request".
  Do NOT use for gathering context on an existing case (use ba-discovery-packet),
  extracting requirements from documents (use ba-signal-extraction),
  or drafting requirements (use ba-requirements-draft).
---

## Overview

Converts an incoming business request — from email, a conversation, or manual description — into a normalized analysis case record. Checks for duplicates, validates required fields, and writes the case to the shared request tracker spreadsheet in SharePoint.

This is the entry point for the BA requirements workflow. Every downstream skill depends on a case record created here.

## When to Use

- A new business request, change request, or enhancement request needs to be logged
- A stakeholder has submitted a request via email, Teams, or verbally and it needs to be formalized
- The user wants to set up a new analysis case for requirements work

## When NOT to Use

- Gathering context or building a discovery packet for an existing case — use ba-discovery-packet
- Extracting requirements from documents — use ba-signal-extraction
- Synthesizing themes or drafting requirements — use ba-theme-synthesis or ba-requirements-draft
- Routing a completed package for review — use ba-review-routing

## Core Instructions

### Progress Tracking

At the start, create tasks:

```
TaskCreate(subject="Gather request details", activeForm="Gathering request details")
TaskCreate(subject="Check for duplicate cases", activeForm="Checking for duplicates")
TaskCreate(subject="Create analysis case record", activeForm="Creating case record")
```

Mark each task `in_progress` when starting and `completed` when done.

### Step 1: Gather Request Details

Collect the following from the user's message, email context, or by asking:

| Field | Required | Source |
|-------|----------|--------|
| Request description / business problem | Yes | User input or email body |
| Requestor name or email | Yes | User input, email sender, or SearchPeople |
| Business capability or domain | Yes | User input |
| Impacted application or process | No | User input or SearchM365(sources=["files"]) |
| Priority | No | User input (default: Medium) |
| Target timeline | No | User input |
| Supporting documents | No | Attached files or SearchM365(sources=["files"]) |

**If the request originated from an email**, search for it:
- `SearchM365(sources=["email"], query="<request keywords>")` to find the thread
- `GetMessage(message_id=...)` to read the full email

**Resolve the requestor identity:**
- `SearchPeople(query="<requestor name>")` to confirm identity
- `GetUserDetails(user_id="<email>")` to get full profile

If critical fields are missing (description, requestor, or domain), ask the user — but ask only once, covering all gaps in a single question.

### Step 2: Check for Duplicates

Before creating a new case, check the request tracker for existing cases:

- `SearchM365(sources=["files"], query="request tracker requirements")` to locate the tracker
- `ReadFileContent(...)` to read the tracker and check for matching descriptions, requestors, or domains

If a potential duplicate is found, present it to the user: "I found an existing case that looks similar — [case ID, description]. Should I create a new case anyway, or update the existing one?"

### Step 3: Create Analysis Case Record

Generate a case record with these fields:

| Column | Value |
|--------|-------|
| Analysis Case ID | Auto-generated (format: AC-YYYY-NNN) |
| Request ID | From source system if available |
| Description | Business problem statement |
| Requestor | Resolved name and email |
| Business Domain | From user input |
| Impacted Application | From user input or "TBD" |
| Priority | From user input or "Medium" |
| Status | New |
| Created Date | Today's date |
| Discovery Status | Not Started |
| Synthesis Status | Not Started |
| Review Status | Not Started |
| Assigned Analyst | Current user (from GetMyDetails) |
| Target Date | From user input or blank |

**Present the case record to the user for confirmation before writing.** Show it as an Adaptive Card with all fields.

After confirmation, write the record to the tracker spreadsheet. If no tracker exists yet, note this to the user and offer to create the case record as a standalone summary.

### Step 4: Confirm and Notify

After writing the case:
- Confirm the case ID and summary to the user
- If a requestor email is available, offer to send a confirmation message acknowledging receipt

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 | Find request emails, existing tracker, related files |
| GetMessage | Read full email content for request details |
| SearchPeople / GetUserDetails | Resolve requestor and stakeholder identities |
| GetMyDetails | Get current user info for Assigned Analyst field |
| ReadFileContent | Read existing tracker to check duplicates |

## Guardrails

- **Never create duplicate cases** for the same request — always check first
- **Validate required fields** before creating a record (description, requestor, domain)
- **Confirm with user** before writing to the tracker — present the full record for review
- **Log the source** — record where the request came from (email ID, conversation, manual entry)
- **Never fabricate request details** — if information is missing, mark it as "TBD" or ask the user
