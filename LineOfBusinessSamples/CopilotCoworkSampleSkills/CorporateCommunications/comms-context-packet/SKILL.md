---
name: comms-context-packet
description: |
  Assembles a context packet of applicable communications policies, brand voice guidelines,
  audience profiles, channel taxonomy, prior announcements, and stakeholder data for a brief case.
  Use when user asks to "build context for [announcement]", "assemble brief context",
  "what policies apply to [topic]", "pull announcement history for [topic area]",
  "gather comms context", "prepare brief background",
  "find brand guidelines for", or "communications context for [case]".
  Do NOT use for creating a new case (use comms-request-intake),
  extracting message themes (use comms-message-extraction),
  drafting the brief (use comms-brief-draft),
  or routing for review (use comms-review-routing).
---

## Overview

Assembles a comprehensive context packet by searching across M365 for applicable communications policies, brand voice guidelines, channel taxonomy, audience segment definitions, prior announcements on the same or related topics, and stakeholder data. Produces a Word document that serves as the foundation for message extraction and brief drafting.

This skill operates in "AI act within policy" mode — it retrieves and assembles approved context but does not interpret policy, recommend messaging, or make editorial judgments.

## When to Use

- A brief case exists and needs communications policy and brand context assembled
- The user wants to find which policies, guidelines, and prior announcements apply to a case
- Preparing the context foundation for message extraction and brief drafting

## When NOT to Use

- Creating a new announcement case — use comms-request-intake
- Extracting message themes and risks — use comms-message-extraction
- Drafting the communications brief — use comms-brief-draft
- Routing the brief for review — use comms-review-routing

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Locate brief case and identify context needs", activeForm="Locating case details")
TaskCreate(subject="Search for policies, guidelines, and prior announcements", activeForm="Searching for context")
TaskCreate(subject="Identify stakeholders and review chain", activeForm="Identifying stakeholders")
TaskCreate(subject="Assemble context packet document", activeForm="Assembling context packet")
```

### Step 1: Locate the Case

Identify the case from the user's message:
- If a Brief ID is provided, search for the tracker: `SearchM365(sources=["files"], query="communications brief tracker")`
- If an announcement topic is given, search: `SearchM365(sources=["files"], query="<topic> announcement")`
- Read the case details from the tracker using `ReadFileContent`

Extract the communication type, target audience, sensitivity level, and department to guide context searches.

### Step 2: Search for Policies, Guidelines, and Prior Announcements

Run parallel searches across M365:

| Search | Tool | Query Strategy |
|--------|------|----------------|
| Communications policies | `SearchM365(sources=["files"])` | "communications policy", "announcement policy" |
| Brand voice guidelines | `SearchM365(sources=["files"])` | "brand voice guidelines", "brand standards", "messaging guidelines" |
| Channel taxonomy | `SearchM365(sources=["files"])` | "channel taxonomy", "communications channels", "distribution guidance" |
| Audience segment definitions | `SearchM365(sources=["files"])` | "audience segments", "employee audience", "stakeholder groups" |
| Prior announcements on same topic | `SearchM365(sources=["files"])` | "[topic] announcement", "[topic] brief", "[topic] talking points" |
| Approved messaging and talking points | `SearchM365(sources=["files"])` | "[topic] approved messaging", "[topic] talking points" |
| Review routing matrix | `SearchM365(sources=["files"])` | "review routing matrix", "approval matrix", "communications approvals" |
| Brief templates | `SearchM365(sources=["files"])` | "communications brief template", "announcement brief template" |
| Related email correspondence | `SearchM365(sources=["email"])` | "[topic] announcement" to find sponsor directives and stakeholder correspondence |

For each document found, read relevant sections using `ReadFileContent`.

Use `GetDriveChildren` to browse the communications policy library and announcement history folders.

### Step 3: Identify Stakeholders and Review Chain

Build the stakeholder roster:
- `SearchPeople(query="<sponsor name>")` to confirm sponsor
- `GetUserDetails(user_id="<sponsor>")` to get sponsor profile
- `GetManagerDetails(user_id="<sponsor>")` for executive chain
- Read the review routing matrix to determine required reviewers based on communication type and sensitivity level

Determine required reviewers based on announcement characteristics:
- **All announcements:** communications lead review + sponsor sign-off
- **Legal topics** (litigation, regulatory, compliance): legal reviewer
- **People matters** (organizational change, leadership changes): HR reviewer
- **Executive or external-facing:** executive communications reviewer
- **Media exposure:** media relations reviewer
- **Crisis or time-sensitive:** escalated review with shortened timelines

### Step 4: Assemble the Context Packet

Produce a Word document (invoke the `docx` skill) with:

1. **Announcement Request Summary** — Brief ID, announcement title, sponsor, department, communication type, target audience, requested timing, embargo status, sensitivity level
2. **Applicable Communications Policies** — Relevant policy sections with document name, version, and section number
3. **Brand Voice and Messaging Guidelines** — Applicable brand voice guidance, tone requirements, and approved language patterns
4. **Channel Taxonomy and Distribution Guidance** — Recommended channels for this audience and communication type, sequencing rules (e.g., employees before media, board before public)
5. **Prior Announcements** — History of announcements on the same or related topics with dates, channels used, and key messages
6. **Target Audience Profile** — Audience segments, communication preferences, and any known sensitivities
7. **Stakeholder and Reviewer Roster** — Sponsor, communications lead, required reviewers (legal, HR, executive, media relations as applicable), escalation chain
8. **Sensitivity Assessment** — Legal risk, employee impact, media exposure, executive visibility, embargo constraints
9. **Open Questions and Unresolved Dependencies** — Topics needing sponsor clarification, missing context, timing dependencies
10. **Gaps and Missing Context** — Policy documents not found, reviewers not identified, missing audience data

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find policies, guidelines, templates, prior announcements, routing matrix |
| SearchM365 (email) | Find sponsor directives and stakeholder correspondence |
| ReadFileContent | Read policy documents, brand guidelines, channel taxonomy |
| GetDriveChildren | Browse policy library and announcement history |
| SearchPeople / GetUserDetails | Resolve sponsor and communications team identities |
| GetManagerDetails | Map executive chain for escalation and approval routing |

## Guardrails

- **Only surface content from approved repositories** — never include draft policies, deprecated guidelines, or unapproved messaging
- **Cite source document and version** for every policy and guideline referenced
- **Flag if any referenced policy document has been updated** since the last announcement on this topic
- **Identify sensitive topic flags** — litigation, personnel actions, regulatory matters, M&A must be explicitly called out with a note that these require legal and/or HR review
- **Do not include internal-only context** (such as HR investigation details or litigation strategy) in any artifact that might be shared externally
- **Flag missing documents** — if a required policy, guideline, or template is not found, call it out in the Gaps section
- **Preserve evidence chain** — log every document accessed with timestamp in the packet
- **Note permission restrictions** — if any documents were inaccessible due to permissions, note this without attempting to bypass
