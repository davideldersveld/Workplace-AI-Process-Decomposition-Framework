---
name: hr-readiness-packet
description: |
  Assembles employee profile, role and location details, applicable policies,
  required document checklists, and key contacts into a reviewer readiness packet.
  Use when user asks to "build readiness packet", "assemble onboarding context",
  "what do we need for [name]'s onboarding", "gather onboarding context",
  "onboarding packet for [employee]", "pull readiness data for [case]",
  "evidence packet for onboarding [ID]", or "context for new hire [name]".
  Do NOT use for creating a new onboarding case (use hr-onboarding-intake),
  detecting missing items or risks (use hr-gap-detection),
  assigning owners and routing tasks (use hr-task-routing),
  drafting outreach or reminders (use hr-onboarding-comms),
  or preparing readiness summaries (use hr-readiness-summary).
---

## Overview

Assembles all context needed for onboarding readiness review into a single Word document — the readiness packet. Gathers employee profile data, role and location details, applicable HR policy extracts, required document checklists, manager and HRBP contacts, and IT provisioning requirements from M365 and federated data sources.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources and assembles it without interpreting policy or making HR decisions. It does not modify any source records.

## When to Use

- An onboarding case has been created and needs context assembled before readiness review
- The user wants to understand what documents and information are required for a new hire
- An HR specialist needs a complete readiness packet before gap detection or routing

## When NOT to Use

- Creating a new onboarding case — use hr-onboarding-intake
- Detecting missing documents or readiness risks — use hr-gap-detection
- Assigning task owners and routing work — use hr-task-routing
- Drafting outreach emails or reminders — use hr-onboarding-comms
- Preparing readiness summaries — use hr-readiness-summary

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read onboarding case and locate context sources", activeForm="Reading onboarding case data")
TaskCreate(subject="Assemble readiness packet", activeForm="Assembling readiness packet")
```

### Step 1: Read Onboarding Case Record

Locate and read the onboarding case:

- `SearchM365(sources=["files"], query="onboarding tracker")` to find the tracker
- `ReadFileContent` to read the specific onboarding case row
- Extract: Case ID, Employee Name, Start Date, Role, Location, Department, Hiring Manager, Employment Type

### Step 2: Gather Context from All Sources

Retrieve context from each source in parallel where possible:

**Employee profile:**
- `SearchPeople(query="<employee name>")` to check if the employee exists in the directory
- `GetUserDetails(user_id="<employee>")` if the profile is already provisioned
- `SearchM365(sources=["connectors"], connector_ids=["hris-connector"])` for HRIS data if available
- `SearchM365(sources=["email"], query="offer [employee name]")` for offer letter details
- Collect: name, contact information, employment type, offer details, prior employment start dates if rehire

**Role and location details:**
- `SearchM365(sources=["files"], query="[role] job description")` or `SearchM365(sources=["files"], query="[job code] role profile")`
- `SearchM365(sources=["files"], query="[location] office guide")` or `SearchM365(sources=["files"], query="[location] onboarding guide")`
- Collect: job description, reporting structure, office address, floor or seat assignment, remote work policy applicability

**Manager and key contacts:**
- `SearchPeople(query="<hiring manager>")` and `GetUserDetails(user_id="<manager>")` for manager profile
- `GetManagerDetails(user_id="<manager>")` for the manager's reporting chain
- `SearchPeople(query="HR business partner [department]")` for the assigned HRBP
- `SearchPeople(query="IT onboarding coordinator")` for IT provisioning contact
- Collect: hiring manager, skip-level manager, HRBP, IT coordinator, recruiter

**HR policies and checklists:**
- `SearchM365(sources=["files"], query="onboarding checklist")` for the standard onboarding checklist
- `SearchM365(sources=["files"], query="onboarding policy")` for onboarding policies
- `SearchM365(sources=["files"], query="[location] onboarding requirements")` for location-specific requirements
- `SearchM365(sources=["files"], query="[employment type] onboarding")` for type-specific checklists (contractor, intern, etc.)
- `SearchM365(sources=["files"], query="work authorization policy")` for I-9 and work authorization requirements
- Collect: standard checklist items, location-specific requirements, employment-type requirements, policy extracts with version dates

**IT and facilities requirements:**
- `SearchM365(sources=["files"], query="IT provisioning checklist")` for equipment, access, and account setup requirements
- `SearchM365(sources=["files"], query="facilities onboarding")` for badge, parking, and workspace setup
- Collect: required IT setup items (laptop, accounts, VPN, software licenses), facilities items (badge, workspace, parking)

### Step 3: Produce Readiness Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Onboarding Case Summary** — Case ID, Employee Name, Start Date, Role, Location, Department, Employment Type, Status
2. **Employee Profile** — contact information, offer details, employment type; note if profile is not yet provisioned in directory
3. **Role and Location Details** — job description summary, reporting structure, office location, remote work applicability
4. **Required Document Checklist** — all required items by category (identification, tax forms, policy acknowledgments, IT setup, facilities), with status indicators (required / received / not applicable)
5. **HR Policy Context** — applicable policy extracts with document reference and version date; location-specific and employment-type-specific requirements
6. **IT and Facilities Requirements** — equipment, accounts, access, workspace, badge, parking — with responsible team for each
7. **Key Contacts** — hiring manager, HRBP, recruiter, IT coordinator, facilities coordinator
8. **Missing Context Flags** — any critical data that could not be retrieved, with source and reason

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, policies, checklists, location guides, IT requirements |
| SearchM365 (email) | Find offer letter thread for context |
| SearchM365 (connectors) | Pull new hire data from HRIS if Graph Connector available |
| ReadFileContent | Read tracker, policies, checklists, location guides |
| GetDriveChildren | List files in the employee's onboarding folder |
| SearchPeople / GetUserDetails | Resolve employee, manager, HRBP, IT coordinator |
| GetManagerDetails | Reporting chain for the hiring manager |

## Guardrails

- **Mask sensitive PII** — never include SSN, date of birth, salary, bank details, or background check results in the readiness packet
- **Cite source and version** for every policy extract — include document reference and version date
- **Flag missing critical context** — if required policies, checklists, or role data cannot be found, flag prominently
- **Mark data freshness** — flag context sourced from systems not updated in the last 5 business days
- **Operate read-only** — never update the HRIS, onboarding tracker, or any source system from this skill
- **Record assembly metadata** — note which sources were queried, what was found, and what was missing
- **Protect employee privacy** — limit information in the packet to what is needed for onboarding readiness; exclude medical, disability, or accommodation details unless explicitly part of a checklist item
