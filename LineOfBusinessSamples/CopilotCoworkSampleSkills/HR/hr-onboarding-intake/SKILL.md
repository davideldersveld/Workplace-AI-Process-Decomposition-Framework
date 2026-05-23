---
name: hr-onboarding-intake
description: |
  Normalizes inbound new-hire onboarding events into structured case records
  in the onboarding tracker, validates required fields, and checks for duplicate cases.
  Use when user asks to "new hire starting", "onboarding case for [name]",
  "set up onboarding for [employee]", "create onboarding case",
  "log new hire onboarding", "intake onboarding request",
  "onboarding for [name] starting [date]", or "new employee onboarding".
  Do NOT use for assembling readiness context (use hr-readiness-packet),
  detecting missing items or risks (use hr-gap-detection),
  assigning owners and routing tasks (use hr-task-routing),
  drafting outreach or reminders (use hr-onboarding-comms),
  or preparing readiness summaries (use hr-readiness-summary).
---

## Overview

Normalizes inbound new-hire onboarding events — from HRIS records, recruiter email, hiring manager requests, or direct intake — into structured case records in the shared Excel onboarding tracker. Validates that required fields are present, confirms the start date is in the future, checks for duplicate cases, and logs creation metadata for audit traceability.

This skill operates in "deterministic automation" mode — structured field extraction, data validation, duplicate checking, and case creation follow fixed rules with no AI judgment required.

## When to Use

- A new hire record has been created and needs an onboarding case set up
- A recruiter or hiring manager has submitted an onboarding request
- The user wants to create a structured onboarding case for a new employee

## When NOT to Use

- Assembling readiness context, policies, and checklists — use hr-readiness-packet
- Detecting missing documents or readiness risks — use hr-gap-detection
- Assigning task owners and routing work — use hr-task-routing
- Drafting outreach emails or reminders — use hr-onboarding-comms
- Preparing readiness summaries for review — use hr-readiness-summary
- Final readiness confirmation — this is always a human decision (HR-ONB-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read inbound onboarding event and extract case fields", activeForm="Reading onboarding request")
TaskCreate(subject="Validate, check for duplicates, and write case record", activeForm="Creating onboarding case")
```

### Step 1: Read Inbound Onboarding Event

Identify the source of the onboarding request:

- **HRIS system**: `SearchM365(sources=["connectors"], connector_ids=["hris-connector"])` if Graph Connector is configured
- **ATS / recruiter email**: `SearchM365(sources=["email"])` with new hire name or requisition number, then `GetMessage(message_id=...)` to read the full email
- **Hiring manager request**: `SearchM365(sources=["email"])` or `ListChatMessages(person=...)` to find the request
- **SharePoint upload**: `SearchM365(sources=["files"])` to locate onboarding request forms or offer packets

Extract the following fields from the inbound event:

| Field | Description | Required |
|-------|-------------|----------|
| Employee Name | Full name of the new hire | Yes |
| Employee Email | Work email if provisioned, or personal email | If available |
| Start Date | First day of employment | Yes |
| Role / Job Title | Position the new hire is filling | Yes |
| Location | Office location, region, or remote designation | Yes |
| Department / Business Unit | Cost center or organizational unit | Yes |
| Hiring Manager | Name or email of the direct manager | Yes |
| Recruiter | Name or email of the assigned recruiter | If available |
| HR Business Partner | HRBP assigned to the business unit | If available |
| Employment Type | Full-time, part-time, contractor, intern | If available |

**Resolve identities:**
- `SearchPeople(query="<hiring manager name>")` to resolve the manager
- `GetUserDetails(user_id="<manager>")` to pull the manager's profile
- `SearchPeople(query="<new hire name>")` to check if the employee profile already exists in the directory
- `GetMyDetails` to record who created the onboarding case

### Step 2: Validate and Create Record

**Validation checks:**
- **Required fields**: Reject if employee name, start date, role, location, department, or hiring manager is missing. Present the missing fields and ask the user to provide them.
- **Start date validation**: Verify that the start date is in the future. Flag past start dates for correction or confirmation.
- **Date reasonableness**: Flag start dates more than 90 days in the future as potentially needing confirmation.

**Duplicate check:**
Search the onboarding tracker for existing cases matching the same employee name and start date:
- `SearchM365(sources=["files"], query="onboarding tracker")` to locate the tracker
- `ReadFileContent` to read existing records
- If a case already exists for this employee and start date, present it to the user and ask whether to proceed with a new case or update the existing one

**Generate Onboarding Case ID:**
Format: ONB-YYYY-NNNNN (e.g., ONB-2026-00034), incrementing from the last ID in the tracker.

**Write the onboarding case record** to the tracker with all extracted fields and:
- Status: "New"
- Created Date: current timestamp
- Assigned To: blank (assigned later by hr-task-routing)
- Checklist Completion: 0%
- SLA target: 2 business days to readiness review

**Present confirmation** via Adaptive Card (invoke `render-ui` skill first):
- Onboarding Case ID, Employee Name, Start Date
- Role, Location, Department
- Hiring Manager
- Status: New
- SLA: 2 business days to readiness review

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find recruiter or hiring manager request email |
| SearchM365 (files) | Find onboarding tracker, locate offer packets or request forms |
| SearchM365 (connectors) | Pull new hire data from HRIS if Graph Connector available |
| GetMessage | Read full request email content |
| ReadFileContent | Read onboarding tracker for duplicate check |
| SearchPeople / GetUserDetails | Resolve hiring manager, recruiter, new hire identity |
| GetMyDetails | Current user for audit trail |

## Guardrails

- **Never create duplicate cases** for the same employee name and start date — check the tracker first
- **Validate start date** — reject past start dates; flag dates more than 90 days out for confirmation
- **Reject entries missing required fields** — employee name, start date, role, location, department, and hiring manager are mandatory
- **Confirm details with user** before writing to the onboarding tracker
- **Log creation metadata** — timestamp, created-by user, source channel, and request reference for audit traceability
- **Protect employee PII** — never include SSN, date of birth, salary, or bank details in case records or communications
- **Never make employment decisions** — this skill creates onboarding case records; employment eligibility and benefits decisions are always human actions
