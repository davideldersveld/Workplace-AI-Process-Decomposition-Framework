---
name: comms-message-extraction
description: |
  Extracts key message themes, audience implications, timing requirements, sensitivity indicators,
  and risk flags from announcement request artifacts for a communications brief case.
  Use when user asks to "extract message themes from [request]", "what are the key messages for [announcement]",
  "identify risks for [communication]", "parse announcement signals",
  "message analysis for [brief]", "what are the comms risks",
  "extract talking points from", or "analyze announcement request for [topic]".
  Do NOT use for assembling context (use comms-context-packet),
  drafting the brief (use comms-brief-draft),
  routing for review (use comms-review-routing),
  or creating a new case (use comms-request-intake).
---

## Overview

Parses request artifacts — email threads, attached talking points, executive directives, sponsor briefings — to extract key message themes, audience implications, timing and sequencing requirements, sensitivity indicators, dependencies, and conflicting stakeholder expectations. Presents all findings for communications manager review via Adaptive Card and Excel worksheet.

This skill operates in "AI assist" mode — it presents analysis for user review. It does not modify case status, draft messages, or make editorial decisions.

## When to Use

- A context packet has been assembled and the communications manager wants to identify the key messages and risks
- Request artifacts need to be parsed for message themes before brief drafting
- The user wants to understand sensitivities, dependencies, and conflicts in the announcement request

## When NOT to Use

- Assembling business and policy context — use comms-context-packet
- Drafting the communications brief — use comms-brief-draft
- Routing the brief for review — use comms-review-routing
- Creating a new case — use comms-request-intake

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read request artifacts and case context", activeForm="Reading request materials")
TaskCreate(subject="Extract message themes and audience implications", activeForm="Extracting message themes")
TaskCreate(subject="Identify sensitivity indicators and risk flags", activeForm="Detecting risk flags")
TaskCreate(subject="Present message analysis for review", activeForm="Preparing analysis")
```

### Step 1: Read Request Artifacts

Locate and read required inputs:

- **Brief case data** — from the communications brief tracker: `SearchM365(sources=["files"], query="communications brief tracker")`
- **Context packet** — from comms-context-packet output: `SearchM365(sources=["files"], query="context packet [Brief ID]")`
- **Request emails and directives** — `SearchM365(sources=["email"], query="[announcement topic]")`
- **Attached talking points and briefings** — from the brief workspace folder: `GetDriveChildren` to list all artifacts
- **Related Teams discussions** — `SearchM365(sources=["teams"], query="[announcement topic]")`

Read each document using `ReadFileContent`.

### Step 2: Extract Message Themes

Parse the request artifacts to extract structured themes across these categories:

| Category | What to Extract |
|----------|----------------|
| **Key Messages** | Core messages the sponsor wants communicated, including exact executive-provided language |
| **Audience Segments** | Which audiences need to receive this, and any audience-specific messaging variations |
| **Channel Implications** | Which channels are appropriate or required for each audience segment |
| **Timing and Sequencing** | Announcement date, embargo dates, sequencing requirements (who hears first), coordination with other events |
| **Dependencies** | Other announcements, events, approvals, or decisions this announcement depends on |
| **Stakeholder Expectations** | What different stakeholders (sponsor, executives, legal, HR) expect from this announcement |
| **Conflicts** | Conflicting messages from different stakeholders, conflicting timing requirements, or messaging that contradicts prior announcements |
| **Open Questions** | Items that need sponsor clarification before the brief can be drafted |

For each extracted theme:
1. Record the theme text
2. Note the source document or conversation
3. Mark as Confirmed (sponsor-stated), Unconfirmed (inferred), or Conflicting (contradicts another source)
4. Flag if it involves a sensitive topic

### Step 3: Identify Sensitivity Indicators and Risk Flags

Analyze the request materials for these risk indicator categories:

| Indicator Category | What to Look For | Sensitivity Signal |
|-------------------|------------------|-------------------|
| **Sensitive topic reference** | Litigation, personnel actions, regulatory matters, M&A, layoffs | Elevated — requires legal and/or HR review |
| **Executive language fidelity** | Specific language provided by an executive with instruction to use verbatim | Flag as "executive-provided — do not paraphrase" |
| **Embargo constraint** | Dates or conditions before which the announcement cannot be released | Critical — requires embargo enforcement tracking |
| **Audience sequencing risk** | Internal audiences who must hear before external release (employees before media, board before public) | Elevated — misordering causes trust damage |
| **Timing conflict** | Requested date conflicts with other active announcements, company events, or known media cycles | Flag for communications manager decision |
| **Contradictory messaging** | Different stakeholders providing conflicting key messages or positioning | Elevated — cannot be auto-resolved |
| **Prior announcement inconsistency** | Key messages that contradict or significantly shift from prior announcements on the same topic | Flag with reference to prior announcement |
| **Missing required context** | Key information gaps that would prevent a complete brief (e.g., no audience defined, no key messages from sponsor) | Blocking — brief cannot proceed |

For each indicator found:
- State the specific evidence that triggered it
- Cite the source document or conversation
- Classify as: Low / Medium / High / Critical

### Step 4: Present Message Analysis

Present findings via Adaptive Card (invoke `render-ui` skill first):

**Summary section:**
- Total message themes extracted by category
- Number of sensitivity indicators by severity
- Confirmed vs. unconfirmed vs. conflicting themes
- Blocking items (if any)

**Detail section:**
- Message theme table (ID, category, theme text, source, status, sensitivity flag)
- Risk indicator list (indicator, evidence, severity, source)

Also produce an Excel worksheet with columns:

| Column | Description |
|--------|-------------|
| Theme ID | Sequential identifier |
| Category | Key Message / Audience / Timing / Risk / Dependency / Conflict / Open Question |
| Theme Text | Extracted content |
| Source | Document or conversation reference |
| Sensitivity Flag | Yes / No |
| Status | Confirmed / Unconfirmed / Conflicting |
| Assigned To | Person who needs to resolve (if applicable) |
| Notes | Additional context |

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find brief tracker, context packet, attached documents |
| SearchM365 (email) | Find sponsor emails, executive directives, stakeholder correspondence |
| SearchM365 (teams) | Find related Teams discussions about the announcement |
| ReadFileContent | Read request documents, talking points, context packet |
| GetDriveChildren | List all artifacts in the brief workspace folder |

## Guardrails

- **Present extracted themes for user review** before writing to the tracker — message interpretation in communications is high-judgment work
- **Flag sensitive topics with elevated visibility** — litigation, personnel changes, regulatory matters must be prominently surfaced
- **Never auto-resolve conflicting messages** from different stakeholders — present conflicts for communications manager decision
- **Preserve original sponsor or executive language** alongside any summarized version — in communications, exact wording matters
- **Flag timing conflicts** with known embargo dates or other active announcements
- **Read-only mode** — this skill does not modify case status, draft messages, or make editorial decisions; it only surfaces analysis
- **Cite every extraction** — each theme and indicator must reference the specific source document or conversation
