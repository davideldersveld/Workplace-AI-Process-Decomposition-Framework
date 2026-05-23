---
name: ba-signal-extraction
description: |
  Extracts candidate requirements, needs, constraints, and assumptions from source documents.
  Reads discovery packets, workshop notes, emails, and transcripts to produce a structured signal inventory.
  Use when user asks to "extract requirements from these documents",
  "pull needs from workshop notes", "what are the stakeholder requirements",
  "extract signals from [source]", "analyze these documents for requirements",
  "what needs and constraints are in these files", "extract from discovery packet",
  or "identify requirements in [document]".
  Do NOT use for assembling background context (use ba-discovery-packet),
  clustering themes or finding conflicts (use ba-theme-synthesis),
  or drafting requirements documents (use ba-requirements-draft).
---

## Overview

Reads source artifacts — discovery packets, workshop notes, stakeholder emails, meeting transcripts, intake forms, and process maps — and extracts structured requirements signals. Each signal is categorized, attributed to its source, and flagged if ambiguous or conflicting.

This skill operates in "AI assist" mode — it presents extraction results for analyst review before any further processing. It does not make final requirements judgments.

## When to Use

- Source documents are ready and need to be analyzed for requirements signals
- The user has a discovery packet, workshop notes, or stakeholder emails to process
- Preparing input for theme synthesis by extracting structured signals

## When NOT to Use

- Assembling background context — use ba-discovery-packet
- Clustering or reconciling themes — use ba-theme-synthesis
- Drafting final requirements — use ba-requirements-draft
- Routing for review — use ba-review-routing

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Identify and read source documents", activeForm="Reading source documents")
TaskCreate(subject="Extract requirements signals by category", activeForm="Extracting signals")
TaskCreate(subject="Build signal inventory with attribution", activeForm="Building signal inventory")
```

### Step 1: Identify Source Documents

Determine which documents to process:

- If the user provides specific files, read them from `input/` or search M365
- If a case ID is given, find the discovery packet and associated documents
- Search for additional context:
  - `SearchM365(sources=["email"], query="<project> requirements")` for stakeholder emails
  - `SearchM365(sources=["teams"], query="<project> requirements")` for Teams discussions
  - `GetMeetingTranscript(join_url="...")` if meeting transcripts are available

Read each source document using `ReadFileContent` or the `Read` tool for uploaded files.

### Step 2: Extract Signals by Category

For each source document, extract signals into these categories:

| Category | Description | Example |
|----------|-------------|---------|
| **Stated Business Needs** | Explicit stakeholder requests or desired capabilities | "We need the system to support bulk uploads" |
| **Constraints** | Technical, regulatory, timeline, or budget limitations | "Must comply with SOX requirements" |
| **Assumptions** | Stated or implied assumptions | "Assumes current vendor contract will be renewed" |
| **Decisions Already Made** | Pre-existing decisions that constrain the solution | "Leadership approved the cloud-first approach" |
| **Open Questions** | Items requiring clarification | "Unclear whether mobile access is in scope" |
| **Dependencies** | Dependencies on other systems or teams | "Requires data feed from the finance system" |
| **Conflicting Statements** | Contradictory signals from different sources | "Marketing wants self-service; Compliance wants approval workflow" |

For each signal, capture:
- **Signal text** — the extracted statement
- **Category** — from the table above
- **Source document** — name and location
- **Source author** — who said or wrote it
- **Source date** — when it was created or stated
- **Confidence** — "Stated by stakeholder" vs. "Inferred from context"

### Step 3: Build Signal Inventory

Present the extracted signals as a structured summary. Use `render-ui` (invoke the `render-ui` skill first) to display the extraction summary as an Adaptive Card with:

- Count of signals per category
- Top conflicting statements requiring resolution
- Open questions requiring clarification
- Sources processed and any sources that could not be read

Then produce an Excel workbook (invoke the `xlsx` skill) with:

| Column | Description |
|--------|-------------|
| Signal ID | Auto-generated (SIG-NNN) |
| Category | From the extraction categories |
| Signal Text | The extracted statement |
| Source Document | Name and location |
| Source Author | Who stated it |
| Source Date | When it was stated |
| Confidence | Stated / Inferred |
| Status | New (for analyst review) |
| Analyst Notes | Blank (for analyst to fill) |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find source documents in SharePoint |
| SearchM365 (email) | Find stakeholder email threads |
| SearchM365 (teams) | Find relevant Teams discussions |
| ReadFileContent | Read documents from SharePoint/OneDrive |
| GetMeetingTranscript | Extract signals from meeting recordings |

## Guardrails

- **Present results for review** — this is AI assist mode; never proceed to synthesis without analyst confirmation
- **Attribute every signal** — every extracted signal must trace to a specific source document, author, and date
- **Never infer intent** — do not interpret what a stakeholder meant beyond what they explicitly stated
- **Flag ambiguity** — contradictory or ambiguous signals must be flagged for manual resolution, not resolved automatically
- **Distinguish stated from inferred** — clearly label whether a signal was directly stated or inferred from context
- **One extraction pass per source** — do not re-extract from the same document unless the user asks
