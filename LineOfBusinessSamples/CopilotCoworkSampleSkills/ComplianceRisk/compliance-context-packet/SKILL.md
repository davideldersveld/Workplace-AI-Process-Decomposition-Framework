---
name: compliance-context-packet
description: |
  Assembles a context packet of applicable policies, control ownership, entity context,
  prior exceptions, and evidence checklists for a compliance case.
  Use when user asks to "build compliance context for", "assemble case packet",
  "pull policy context for [case ID]", "what policies apply to [issue]",
  "prepare evidence packet", "gather compliance context",
  "find policies for [control area]", or "compliance background for [case]".
  Do NOT use for creating a new case (use compliance-case-intake),
  detecting risk indicators (use compliance-risk-detection),
  routing for review (use compliance-review-routing),
  or drafting case communications (use compliance-case-comms).
---

## Overview

Assembles a comprehensive context packet by searching across M365 for applicable policy documents, control standards, control ownership data, prior exceptions, entity context, and evidence checklists. Produces a Word document and Excel evidence checklist that serve as the foundation for risk analysis.

This skill operates in "AI act within policy" mode — it retrieves and assembles approved context but does not make risk judgments or interpret policy intent.

## When to Use

- A compliance case exists and needs policy and control context assembled
- The user wants to find which policies, controls, and prior exceptions apply to a case
- Preparing the evidence foundation for risk analysis

## When NOT to Use

- Creating a new compliance case — use compliance-case-intake
- Detecting risk indicators or evidence gaps — use compliance-risk-detection
- Routing a case for review — use compliance-review-routing
- Drafting case communications — use compliance-case-comms

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Locate case and identify affected policy areas", activeForm="Locating case details")
TaskCreate(subject="Search for applicable policies and control standards", activeForm="Searching for policies")
TaskCreate(subject="Identify control owners and accountability chain", activeForm="Identifying control owners")
TaskCreate(subject="Assemble context packet and evidence checklist", activeForm="Assembling context packet")
```

### Step 1: Locate the Case

Identify the case from the user's message:
- If a Case ID is provided, search for the tracker: `SearchM365(sources=["files"], query="compliance case tracker")`
- If a topic or policy area is given, search: `SearchM365(sources=["files"], query="<topic> compliance")`
- Read the case details from the tracker using `ReadFileContent`

### Step 2: Search for Applicable Policies and Standards

Run parallel searches across M365:

| Search | Tool | Query Strategy |
|--------|------|----------------|
| Policy documents | `SearchM365(sources=["files"])` | "[policy area] policy", "[control area] standard" |
| Control standards and frameworks | `SearchM365(sources=["files"])` | "[control area] control standard", "control framework" |
| Prior exceptions and findings | `SearchM365(sources=["files"])` | "[policy area] exception", "[policy area] finding" |
| Evidence checklists | `SearchM365(sources=["files"])` | "evidence checklist [case type]", "compliance checklist" |
| Severity matrix and routing rules | `SearchM365(sources=["files"])` | "severity matrix", "compliance routing rules" |
| Prior case history (if Graph Connector available) | `SearchM365(sources=["connectors"], connector_ids=["grc-connector"])` | "[policy area] case" |

For each document found, read relevant sections using `ReadFileContent`.

Use `GetDriveChildren` to browse the compliance policy library and evidence folders.

### Step 3: Identify Control Owners and Accountability Chain

Build the accountability roster:
- `SearchPeople(query="<control area or business unit>")` to find control owners
- `GetUserDetails(user_id="<control owner>")` to get profile
- `GetManagerDetails(user_id="<control owner>")` for escalation chain
- Cross-reference against SharePoint control ownership data if available

### Step 4: Assemble the Context Packet

Produce a Word document (invoke the `docx` skill) with:

1. **Case Summary** — Case type, reporter context (or "Anonymous"), affected area, entity, business unit
2. **Applicable Policy Extracts** — Relevant policy sections with document name, version, and section number
3. **Control Ownership and Accountability** — Control owner, business process owner, escalation chain
4. **Prior Exceptions and Findings** — History of exceptions or findings for the same policy area with dates and outcomes
5. **Evidence Checklist** — Required evidence items with completion status (submitted / missing / not applicable)
6. **Required Approvals** — Approval requirements based on case type and severity from the routing rules
7. **Gaps and Missing Context** — Policy documents not found, control owners not identified, missing evidence checklist

Also produce an Excel evidence checklist (invoke the `xlsx` skill) with:

| Column | Description |
|--------|-------------|
| Evidence Item | Required document or artifact |
| Category | Policy / Control / Financial / Operational / Legal |
| Status | Submitted / Missing / Not Applicable |
| Source Location | SharePoint path or "Not Found" |
| Date Submitted | Date if submitted |
| Notes | Any relevant context |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find policies, control standards, prior exceptions, evidence checklists |
| SearchM365 (connectors) | GRC platform case history (when Graph Connector available) |
| ReadFileContent | Read policy documents, control standards, severity matrices |
| GetDriveChildren | Browse policy library and evidence folders |
| SearchPeople / GetUserDetails | Resolve control owners and compliance analysts |
| GetManagerDetails | Map escalation chain |

## Guardrails

- **Cite every policy reference** — include document name, version, and section number for every policy extract
- **Never interpret policy intent** — present policy text as written; interpretation is the analyst's responsibility
- **Flag missing documents** — if a required policy document or control standard is not found, call it out in the Gaps section
- **Never include reporter identity** for anonymous cases — context packets must not reveal anonymous reporters
- **Preserve evidence chain of custody** — log every document accessed with timestamp in the packet
- **Restrict access awareness** — note if any documents were inaccessible due to permissions (do not attempt to bypass)
