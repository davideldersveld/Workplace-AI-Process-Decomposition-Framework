---
name: cs-context-packet
description: |
  Assembles customer profile, account history, entitlement details, prior cases,
  and product context for a service case.
  Use when user asks to "build context for case [ID]", "pull customer history for",
  "assemble case context", "what do we know about this customer",
  "customer background for [case]", "gather account context",
  "prior cases for [customer]", or "case context packet".
  Do NOT use for creating a new case (use cs-case-intake),
  classifying the issue (use cs-issue-classifier),
  assessing severity (use cs-severity-assessment),
  routing the case (use cs-case-routing),
  or drafting a response (use cs-response-drafter).
---

## Overview

Assembles a comprehensive context packet by searching across M365 for customer profile data, account history, entitlement and service tier details, prior case history, product or service context, and known issues or active incidents. Produces a Word document that serves as the foundation for classification, severity assessment, and response drafting.

This skill operates in "AI act within policy" mode — it retrieves and assembles approved context but does not classify, prioritize, or recommend actions.

## When to Use

- A case exists and needs customer and account context assembled before triage
- The user wants to understand the customer's history and entitlement before responding
- Preparing the context foundation for classification, severity assessment, and response drafting

## When NOT to Use

- Creating a new case — use cs-case-intake
- Classifying the issue type — use cs-issue-classifier
- Assessing severity and SLA — use cs-severity-assessment
- Routing the case — use cs-case-routing
- Drafting a response — use cs-response-drafter

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Locate case and identify customer", activeForm="Locating case details")
TaskCreate(subject="Search for customer account and history", activeForm="Searching for customer data")
TaskCreate(subject="Assemble context packet document", activeForm="Assembling context packet")
```

### Step 1: Locate the Case

Identify the case from the user's message:
- If a Case ID is provided, search for the tracker: `SearchM365(sources=["files"], query="service case tracker")`
- If a customer name is given, search: `SearchM365(sources=["email"], query="<customer name>")`
- Read the case details from the tracker using `ReadFileContent`

Extract the customer name, account ID, channel, and issue summary to guide context searches.

### Step 2: Search for Customer Account and History

Run parallel searches across M365:

| Search | Tool | Query Strategy |
|--------|------|----------------|
| CRM account profile | `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` | "[customer name]" or "[account ID]" |
| Prior correspondence | `SearchM365(sources=["email"])` | "from:[customer email]" or "[customer name]" |
| Account documents | `SearchM365(sources=["files"])` | "[customer name] account", "[account ID] contract" |
| Prior cases | `SearchM365(sources=["files"])` | "[customer name] case", "[account ID]" in case tracker |
| Product or service docs | `SearchM365(sources=["files"])` | "[product name] documentation", "[service] SOP" |
| Known issues or incidents | `SearchM365(sources=["files"])` | "known issue", "active incident", "[product] outage" |
| Internal discussions | `SearchM365(sources=["teams"])` | "[customer name]" or "[account ID]" |
| Account owner profile | `GetUserDetails(user_id="<account owner>")` | Resolve internal account owner |

For each document found, read relevant sections using `ReadFileContent`.

### Step 3: Assemble the Context Packet

Produce a Word document (invoke the `docx` skill) with:

1. **Customer Profile** — Name, email, account ID, company, contact details
2. **Account and Entitlement Summary** — Service tier, contract status, entitlement details, account health indicators
3. **Prior Case History** — Last 5 cases with dates, issue types, resolutions, and outcomes; flag any repeat patterns
4. **Product or Service Context** — Products owned, services subscribed, relevant version or configuration details
5. **Known Issues and Active Incidents** — Any active incidents affecting this customer's product or service area
6. **Prior Correspondence Summary** — Recent email and Teams threads involving this customer (last 30 days)
7. **Account Owner and Escalation Contacts** — Internal account owner, escalation manager, executive sponsor (if applicable)
8. **Data Gaps** — Information not found (e.g., CRM unavailable, no prior cases, entitlement data stale)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, account documents, known issues |
| SearchM365 (email) | Find prior customer correspondence |
| SearchM365 (teams) | Find internal discussions about this customer |
| SearchM365 (connectors) | Pull CRM account data if Graph Connector available |
| ReadFileContent | Read case tracker, account docs, product documentation |
| GetDriveChildren | Browse case folders and account document libraries |
| SearchPeople / GetUserDetails | Resolve account owner and service contacts |
| GetManagerDetails | Map escalation chain for the account |

## Guardrails

- **Mask sensitive financial details** — credit card numbers, billing specifics, and payment information must not appear in generated documents
- **Cite source and retrieval date** for every data point from CRM or prior cases
- **Flag stale data** — if customer entitlement data is older than 30 days, call it out prominently
- **Operate in read-only mode** — never update CRM, account records, or case status
- **Flag data gaps** — if CRM is unavailable, prior cases not found, or entitlement data missing, surface it in the Data Gaps section
- **Note permission restrictions** — if any data sources were inaccessible due to permissions, note this
- **Preserve customer privacy** — do not include customer data in any artifact beyond what is needed for the case
