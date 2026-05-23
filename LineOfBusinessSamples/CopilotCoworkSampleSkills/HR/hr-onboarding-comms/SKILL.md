---
name: hr-onboarding-comms
description: |
  Drafts onboarding outreach emails, missing document reminders, manager updates,
  IT provisioning requests, and welcome messages for onboarding cases.
  Use when user asks to "draft onboarding email", "remind [manager] about onboarding",
  "send welcome message to new hire", "onboarding follow-up",
  "draft missing documents reminder", "IT provisioning request for [employee]",
  "HRBP notification for new hire", "draft onboarding update for [manager]",
  or "onboarding communication for [case]".
  Do NOT use for creating a new onboarding case (use hr-onboarding-intake),
  assembling readiness context (use hr-readiness-packet),
  detecting missing items or risks (use hr-gap-detection),
  assigning owners and routing tasks (use hr-task-routing),
  or preparing readiness summaries (use hr-readiness-summary).
---

## Overview

Drafts audience-appropriate communications for onboarding coordination — welcome emails to new hires, missing document reminders, readiness updates to hiring managers, IT provisioning requests, HRBP notifications, and escalation notices. All communications are created as Outlook drafts or Teams messages for HR specialist review before sending. Draws on the readiness packet, gap report, and case data to produce accurate, context-specific content.

This skill operates in "AI draft plus approve" mode — every draft is presented for HR specialist review and confirmation before any message is sent.

## When to Use

- A new hire needs a welcome email or pre-boarding information
- Missing documents need to be requested from the employee or hiring manager
- A hiring manager needs an update on onboarding status
- IT or facilities needs a provisioning request
- An HRBP needs to be notified about a new hire in their business unit
- An escalation notice needs to be sent for a blocked onboarding case

## When NOT to Use

- Creating a new onboarding case — use hr-onboarding-intake
- Assembling readiness context — use hr-readiness-packet
- Detecting missing documents or risks — use hr-gap-detection
- Assigning task owners — use hr-task-routing
- Preparing readiness summaries — use hr-readiness-summary

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and determine communication type", activeForm="Reading case context")
TaskCreate(subject="Draft communication for review", activeForm="Drafting communication")
```

### Step 1: Read Case Context

Locate and read required inputs:

- **Onboarding case data** — from the tracker: `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent`
- **Readiness packet** — `SearchM365(sources=["files"], query="readiness packet [Case ID]")` then `ReadFileContent`
- **Gap report** — from hr-gap-detection output (missing items, blockers, timeline risk)
- **Communication templates** — `SearchM365(sources=["files"], query="onboarding email template")` or `SearchM365(sources=["files"], query="HR communication template")` then `ReadFileContent`
- **Tone guides** — `SearchM365(sources=["files"], query="HR communication guidelines")` if available

Determine which communication type is needed based on the gap report, case status, and user request.

### Step 2: Draft Communication

**Communication type 1: Welcome email to new hire**
- To: new hire (personal email or provisioned work email)
- Tone: warm, professional, welcoming
- Content: congratulations, start date confirmation, first-day logistics (time, location, who to ask for), what to bring, links to pre-boarding portal or document submission forms, key contact (hiring manager, HR specialist)
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 2: Missing documents reminder to employee**
- To: new hire (via hiring manager if employee email not available)
- Tone: helpful, clear, not punitive
- Content: Onboarding Case ID, specific documents still needed, how to submit (SharePoint upload link or email), deadline based on start date, contact for questions
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 3: Readiness update to hiring manager**
- To: hiring manager
- Tone: concise, operational
- Content: Onboarding Case ID, Employee Name, Start Date, current readiness status, items complete vs. pending, any blockers requiring manager action (confirm seat assignment, confirm team introduction plan), timeline risk if applicable
- Channel: Outlook draft (use `CreateDraftMessage`) or Teams message (use `PostMessage` for quick updates)

**Communication type 4: IT provisioning request**
- To: IT onboarding coordinator
- Tone: structured, operational
- Content: Onboarding Case ID, Employee Name, Start Date, Role, Location, required equipment (laptop type, monitors), required accounts and access (email, VPN, software licenses, system access), any privileged access requirements flagged in gap report
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 5: HRBP notification**
- To: HR business partner assigned to the department
- Tone: informational, concise
- Content: New hire summary (name, role, department, start date, hiring manager), any policy-sensitive conditions flagged (international hire, contractor conversion, rehire), readiness status overview
- Channel: Outlook draft (use `CreateDraftMessage`)

**Communication type 6: Escalation notice**
- To: HR operations manager
- Tone: factual, urgent where appropriate
- Content: Onboarding Case ID, Employee Name, Start Date, what is blocked, how long it has been blocked, prior actions taken, recommended resolution, timeline risk assessment
- Channel: Outlook draft (use `CreateDraftMessage`)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, readiness packet, communication templates, tone guides |
| ReadFileContent | Read case data, readiness packet, templates |
| CreateDraftMessage | Outlook drafts for all formal communications |
| PostMessage | Teams messages for quick internal coordination updates |
| SearchPeople / GetUserDetails | Resolve recipient identities and email addresses |

## Guardrails

- **Always create as Outlook draft** — never send without explicit HR specialist confirmation
- **Match tone to audience** — warm and welcoming for new hire communications; concise and operational for internal requests; factual for escalations
- **Include Onboarding Case ID** in every communication for audit traceability
- **Never include sensitive PII** in any communication — no SSN, date of birth, salary, background check details, or medical information
- **Never include salary or compensation details** in manager updates or IT requests — these are confidential between HR and the employee
- **Never send work authorization or background check details** via Teams — use Outlook only for sensitive HR content
- **Respect the 2-business-day SLA** — note urgency in communications when the start date is approaching
- **Never make employment commitments** in draft communications — avoid language that could be construed as employment guarantees or benefit promises beyond what is in the offer letter
- **Log the communication** — record communication type, recipient, timestamp, and case reference in the tracker after sending
