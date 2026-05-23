---
name: ba-discovery-packet
description: |
  Assembles a discovery packet of background context for a requirements analysis case.
  Searches M365 for prior documents, stakeholder contacts, policy references, and related decisions.
  Use when user asks to "build discovery packet for [request]", "gather context for requirements",
  "what background do we have for [project]", "assemble discovery artifacts",
  "pull together context for [case]", "what do we know about [topic]",
  "find existing documents for [project]", or "discovery for analysis case [ID]".
  Do NOT use for extracting specific requirements from documents (use ba-signal-extraction),
  creating a new analysis case (use ba-request-intake),
  or synthesizing themes (use ba-theme-synthesis).
---

## Overview

Assembles a comprehensive discovery packet by searching across M365 for prior business requirements documents, process maps, policy documents, stakeholder contacts, and related decisions. Produces a Word document that serves as the evidence base for requirements extraction.

This skill operates in "AI act within policy" mode — it reads and assembles approved context but does not make requirements judgments.

## When to Use

- An analysis case exists and needs background context assembled
- The user wants to find what prior documents, decisions, or contacts are relevant to a request
- Starting discovery for a new or existing requirements effort

## When NOT to Use

- Creating a new analysis case — use ba-request-intake
- Extracting specific requirements from the discovery documents — use ba-signal-extraction
- Synthesizing themes from extracted signals — use ba-theme-synthesis
- Drafting requirements — use ba-requirements-draft

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Locate analysis case and request details", activeForm="Locating case details")
TaskCreate(subject="Search for prior documents and decisions", activeForm="Searching for documents")
TaskCreate(subject="Identify stakeholders and SMEs", activeForm="Identifying stakeholders")
TaskCreate(subject="Assemble discovery packet", activeForm="Assembling discovery packet")
```

### Step 1: Locate the Analysis Case

Identify the case from the user's message:
- If a case ID is provided, search for the tracker: `SearchM365(sources=["files"], query="request tracker")`
- If a project name or topic is given, search: `SearchM365(sources=["files"], query="<topic> requirements")`
- Read the case details from the tracker using `ReadFileContent`

### Step 2: Search for Prior Documents

Run parallel searches across M365:

| Search | Tool | Query Strategy |
|--------|------|----------------|
| Prior BRDs and requirements | `SearchM365(sources=["files"])` | "[domain] requirements", "[application] BRD" |
| Process maps and SOPs | `SearchM365(sources=["files"])` | "[domain] process map", "[application] SOP" |
| Policy and compliance docs | `SearchM365(sources=["files"])` | "[domain] policy", "compliance [topic]" |
| Architecture and system docs | `SearchM365(sources=["files"])` | "[application] architecture", "[system] design" |
| Prior decisions and change history | `SearchM365(sources=["email"])` | "[project] decision", "[project] approved" |
| Related discussions | `SearchM365(sources=["teams"])` | "[project] requirements", "[topic] change request" |

For each document found, read key sections using `ReadFileContent` to extract relevant context.

Use `GetDriveChildren` to browse project or domain folders if a SharePoint site is known.

### Step 3: Identify Stakeholders and SMEs

Build a stakeholder roster:
- `SearchPeople(query="<domain or application keywords>")` to find domain experts
- `GetUserDetails(user_id="<requestor>")` to get requestor profile
- `GetManagerDetails(user_id="<requestor>")` for org context
- `GetDirectReportsDetails(user_id="<manager>")` if team context is needed

### Step 4: Assemble the Discovery Packet

Produce a Word document (invoke the `docx` skill) with the following sections:

1. **Request Summary** — Business problem statement from the analysis case
2. **Document Inventory** — Table of found documents with name, type, location, date, and relevance
3. **Prior Related Decisions** — Summary of decisions found in email or documents, with source attribution
4. **System and Process Context** — Current-state information about impacted systems or processes
5. **Stakeholder Roster** — Table with name, role, department, email, and relevance to this request
6. **Policy and Compliance References** — Applicable policies or regulations found
7. **Known Constraints and Assumptions** — Items identified from source material
8. **Gaps and Missing Context** — What was expected but not found (no prior BRD, no process map, no SME identified)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find prior BRDs, process maps, policies, architecture docs |
| SearchM365 (email) | Find decision threads and stakeholder communications |
| SearchM365 (teams) | Find related discussions and informal context |
| ReadFileContent | Read specific documents for content extraction |
| GetDriveChildren | Browse project folders in SharePoint |
| SearchPeople / GetUserDetails | Resolve stakeholder identities |
| GetManagerDetails / GetDirectReportsDetails | Map org structure for stakeholder roster |

## Guardrails

- **Cite every source** — every referenced artifact must include document name, location, and retrieval date
- **Flag missing context** — if critical documents are not found (no prior BRD, no process map, no SME), call this out explicitly in the Gaps section
- **Never fabricate context** — if information is not found in M365, do not infer or invent it
- **Mark informal sources** — information from chat or email threads should be labeled as "unverified until confirmed"
- **Respect permissions** — only include documents the user has access to; do not attempt to access restricted content
