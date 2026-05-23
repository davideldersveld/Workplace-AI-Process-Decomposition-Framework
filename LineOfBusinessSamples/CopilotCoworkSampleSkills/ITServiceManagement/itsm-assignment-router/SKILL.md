---
name: itsm-assignment-router
description: |
  Routes incidents to the correct resolver group based on assignment rules,
  classification, severity, and the support model.
  Use when user asks to "route this incident", "assign resolver group",
  "who handles this type of ticket", "route to the right team",
  "assignment for incident [ID]", "queue this ticket",
  or "escalate incident to [team]".
  Do NOT use for creating a new incident (use itsm-incident-intake),
  enriching incident context (use itsm-context-packet),
  classifying incident type (use itsm-classification-assist),
  assessing severity (use itsm-severity-recommend),
  or drafting user updates (use itsm-comms-drafter).
---

## Overview

Routes incidents to the correct resolver group or queue based on the assignment rules matrix, incident classification, severity, and support model. Resolves resolver group leads and on-call contacts, enforces assignment rule compliance, sends Teams notifications after analyst review, and creates calendar holds for SLA-driven review deadlines.

This skill operates in "AI act within policy" mode for standard P3/P4 incidents with clear assignment rule matches. For P1/P2 incidents, it operates in "AI draft plus approve" mode — the routing recommendation is presented for incident manager confirmation before execution.

## When to Use

- An incident has been classified and prioritized and needs resolver group assignment
- The service desk needs to determine which team handles a specific incident
- An incident needs escalation routing to the major incident bridge
- An incident needs re-routing after reclassification or priority change

## When NOT to Use

- Creating a new incident record — use itsm-incident-intake
- Assembling user, service, or asset context — use itsm-context-packet
- Classifying the incident type or affected service — use itsm-classification-assist
- Assessing severity or escalation path — use itsm-severity-recommend
- Drafting user updates or handoff summaries — use itsm-comms-drafter
- Confirming triage disposition — this is always a human decision (ITSM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read incident data and assignment rules", activeForm="Reading assignment inputs")
TaskCreate(subject="Route incident and send notifications", activeForm="Routing incident")
```

### Step 1: Read Assignment Inputs

**Read the incident data, classification, and severity:**
- `SearchM365(sources=["files"], query="incident tracker")` then `ReadFileContent` — current incident data with classification and priority

**Read the assignment rules and support model:**
- `SearchM365(sources=["files"], query="assignment rules matrix")` then `ReadFileContent` — resolver group mapping by category, service, and priority
- `SearchM365(sources=["files"], query="support model")` then `ReadFileContent` — team structure, on-call schedules, escalation paths

**Resolve resolver group contacts:**
- `SearchPeople` — resolve resolver group leads and on-call contacts
- `GetUserDetails` — verify resolver group lead availability and role
- `ListCalendarView` — check resolver group lead availability before assignment

### Step 2: Determine Assignment

Match the incident to the assignment rules based on:

| Assignment Factor | Source |
|-------------------|--------|
| Category and subcategory | Classification from itsm-classification-assist |
| Affected service | Classification from itsm-classification-assist |
| Priority level | Severity from itsm-severity-recommend |
| Location | User profile from context packet |
| VIP or executive flag | User details or severity assessment |

### Resolver Groups

| Group | Handles | Typical Incidents |
|-------|---------|-------------------|
| **Desktop Support** | End-user device issues, software installation, peripheral problems | Hardware, software (desktop), printing |
| **Network Operations** | Network connectivity, VPN, Wi-Fi, DNS issues | Network category incidents |
| **Identity and Access Management** | Login issues, password resets, permission requests, MFA | Identity and access category |
| **Collaboration Services** | Outlook, Teams, SharePoint, OneDrive issues | Email and collaboration category |
| **Telephony Support** | Phone system, voicemail, conference bridge | Telephony category |
| **Application Support** | Line-of-business application issues, ERP, CRM, HR systems | Business application category |
| **Infrastructure Operations** | Server, storage, database, cloud platform issues | Infrastructure category |
| **Security Operations** | Suspected phishing, malware, data loss, security events | Security category (restricted routing) |
| **Major Incident Team** | P1 incidents, confirmed major incidents, widespread outages | Any category at P1 with major incident trigger |
| **Service Operations Manager** | Escalation path when no clear rule match exists | Ambiguous routing, capacity overflow |

### Step 3: Present Routing Recommendation

**For P1 and P2 incidents**, present via Adaptive Card (invoke `render-ui` skill first) for incident manager confirmation:

- **Incident header** — Incident ID, Reported By, Category, Affected Service
- **Classification and priority** — category, subcategory, priority level
- **Recommended resolver group** — target team from assignment rules
- **Recommended assignee** — specific on-call contact or team lead
- **Assignment rationale** — which assignment rule criteria matched
- **On-call verification** — confirmed active on-call contact for P1/P2
- **SLA status** — time elapsed since intake, response and resolution deadlines
- **Escalation flags** — major incident bridge, service owner notification, executive notification
- **Draft label** — "ASSIGNMENT RECOMMENDATION — incident manager confirmation required"

**For P3 and P4 incidents** with clear assignment rule matches, proceed to execution after brief confirmation.

### Step 4: Execute Assignment (After Confirmation)

**Send Teams notification to the assigned resolver group:**
- `PostMessage` — direct message to the assigned resolver or group lead with:
  - Incident ID, category, priority level
  - Symptom summary and affected service
  - SLA response deadline
  - Link to the context packet and incident tracker
  - Required action and expected response time

**Post assignment to the incident management channel:**
- `PostChannelMessage` — post to the incident management channel with:
  - Incident ID, priority, assigned resolver group
  - Never include user PII, infrastructure details, or internal system names in channel posts

**Create SLA deadline calendar hold (P1/P2 only):**
- `CreateEvent` — calendar event for the assigned resolver lead with the SLA response deadline as a visible reminder

**Update tracker:**
- Record assigned resolver group, assignee, assignment timestamp, assignment rationale, and confirming analyst in the incident tracker

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find incident tracker, assignment rules matrix, support model |
| ReadFileContent | Read assignment rules, support model, tracker |
| SearchPeople | Resolve resolver group leads and on-call contacts |
| GetUserDetails | Verify resolver availability and role |
| ListCalendarView | Check resolver availability |
| PostMessage | Direct message to assigned resolver or group lead |
| PostChannelMessage | Incident management channel assignment notification |
| CreateEvent | SLA deadline calendar hold for P1/P2 |

## Guardrails

- **Only assign to resolver groups listed in the approved assignment rules matrix** — ad hoc assignments are not permitted
- **If no matching assignment rule exists**, escalate to the service operations manager rather than guessing
- **For P1 and P2 incidents**, verify that the assigned group has an active on-call contact before routing — present for incident manager confirmation
- **Never reassign a major incident without explicit incident manager confirmation** — major incident ownership changes are production-impacting decisions
- **Never include user PII, internal infrastructure details, or hostnames in incident management channel posts** — use Incident ID and service names only; full details go in direct messages to the assigned resolver
- **Include Incident ID and priority in every outbound Teams message** for traceability
- **Include SLA response deadline** in all assignment notifications — time-to-respond is operationally critical
- **Log the assignment decision** — record assigned group, assignee, rationale, and confirming analyst in the tracker for audit
- **Never execute remediation actions as part of routing** — routing assigns investigation ownership only; all production changes and remediation are human-owned
- **Never modify incident classification or priority during routing** — routing uses the existing classification and priority; changes require re-running the appropriate upstream skill
- **If the classification maps to security**, route to Security Operations with restricted channel visibility — security incidents require specialized handling
