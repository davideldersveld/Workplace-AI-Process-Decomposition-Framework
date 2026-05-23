---
name: comms-brief-draft
description: |
  Drafts a structured communications brief with executive summary, key messages,
  audience and channel plan, timing, sensitivity assessment, and open questions.
  Use when user asks to "draft the communications brief", "create message brief for [announcement]",
  "build the announcement brief", "assemble comms brief",
  "write the brief for [topic]", "draft brief for [Brief ID]",
  "create the announcement brief", or "prepare comms brief for review".
  Do NOT use for assembling context (use comms-context-packet),
  extracting message themes (use comms-message-extraction),
  routing for review (use comms-review-routing),
  or creating a new case (use comms-request-intake).
---

## Overview

Produces a structured communications brief document using the context packet, extracted message themes, and approved brief templates. The brief includes executive summary, announcement objective, target audience and channels, key messages with source citations, timing and sequencing plan, sensitivity assessment, required reviews, and open questions.

This skill operates in "AI draft plus approve" mode — the brief is created as a draft document for communications manager review. Nothing is finalized, distributed, or shared with reviewers without explicit user confirmation.

## When to Use

- Message themes have been extracted and the communications manager is ready to draft the brief
- The user wants to produce a review-ready communications brief from assembled context
- A brief needs to be drafted or redrafted based on reviewer feedback

## When NOT to Use

- Assembling business and policy context — use comms-context-packet
- Extracting message themes and risks — use comms-message-extraction
- Routing the brief for review — use comms-review-routing
- Creating a new case — use comms-request-intake
- Making the final brief disposition — this is a human-only step

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read context packet, message themes, and brief template", activeForm="Reading brief inputs")
TaskCreate(subject="Draft communications brief document", activeForm="Drafting communications brief")
TaskCreate(subject="Present draft summary for review", activeForm="Preparing draft summary")
```

### Step 1: Read Brief Inputs

Locate and read all required inputs:

- **Brief case data** — from the brief tracker: `SearchM365(sources=["files"], query="communications brief tracker")`
- **Context packet** — from comms-context-packet output: `SearchM365(sources=["files"], query="context packet [Brief ID]")`
- **Message themes and risk flags** — from comms-message-extraction output: `SearchM365(sources=["files"], query="message themes [Brief ID]")`
- **Brief template** — `SearchM365(sources=["files"], query="communications brief template")`
- **Brand voice guidelines** — `SearchM365(sources=["files"], query="brand voice guidelines")`
- **Prior approved briefs on similar topics** — `SearchM365(sources=["files"], query="[topic] approved brief")`

Read each document using `ReadFileContent`.

### Step 2: Draft the Communications Brief

Produce a Word document (invoke the `docx` skill) with these sections:

**1. Executive Summary**
- One-paragraph overview of the announcement, its purpose, and its significance
- Communication type and sensitivity level
- Target announcement date and any embargo constraints

**2. Announcement Objective**
- What the organization intends to communicate and why
- Business context driving the announcement
- Desired outcome from the communication

**3. Target Audience and Channels**
- Primary and secondary audience segments
- Recommended channels for each audience (aligned with channel taxonomy)
- Sequencing plan — who hears first, and why
- Any audience-specific messaging variations

**4. Key Messages**
- Numbered key messages with source citations (sponsor directive, approved messaging, policy reference)
- Executive-provided language marked as verbatim — "Executive-provided — do not paraphrase"
- Supporting talking points for each key message
- Anticipated questions and suggested responses

**5. Timing and Sequencing Plan**
- Planned announcement date and time
- Embargo dates and conditions (if applicable)
- Sequencing across audiences and channels
- Dependencies on other announcements or events
- Coordination requirements with other teams

**6. Sensitivity Assessment and Risk Mitigation**
- Sensitivity level with rationale
- Identified risk flags from message extraction (with severity)
- Recommended mitigation actions for each risk
- Topics requiring elevated review (legal, HR, executive)

**7. Required Reviews**
- List of required reviewers by role and name
- Review scope for each reviewer (what they are specifically reviewing for)
- Review deadline (based on SLA)
- Current review status (all "Pending" for new briefs)

**8. Open Questions**
- Unresolved items requiring sponsor or stakeholder input
- Missing information that could affect the brief
- Decision points that must be resolved before review routing

**9. Appendix**
- Prior related announcements (title, date, channel, key messages)
- Source document references with version numbers
- Glossary of terms (if the announcement involves technical or specialized language)

### Step 3: Present Draft Summary

After generating the Word document, present a summary via Adaptive Card (invoke `render-ui` skill first):

- Brief ID and announcement title
- Sections completed vs. sections needing sponsor input
- Sensitive sections flagged for elevated review
- Unresolved open questions count
- Recommended next step (resolve open questions → route for review)

Tell the user: "I've created a draft communications brief for [announcement title]. Review and confirm before routing for review."

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find brief tracker, context packet, message themes, templates, brand guidelines, prior briefs |
| ReadFileContent | Read all input artifacts from SharePoint |

## Guardrails

- **Always create as draft** — never finalize, distribute, or share with reviewers without explicit user confirmation
- **Every key message must trace to a source** — sponsor-provided directive, approved messaging document, or policy reference; no fabricated organizational positions or unsupported claims
- **Sensitive topic sections must be clearly marked** and require explicit acknowledgment before inclusion
- **Channel recommendations must align with the channel taxonomy** — no ad hoc channel suggestions outside the approved taxonomy
- **Timing recommendations must respect embargo dates and sequencing rules** — employees before media, board before public, internal before external
- **Include source citations** for every message element and policy reference used in the brief
- **Match template formatting** from the approved brief template library
- **Never include specific personnel names** in organizational change briefs without sponsor confirmation
- **Preserve executive-provided language exactly** — mark it clearly and do not paraphrase
- **Never present the brief as complete** if open questions remain unresolved — always surface the gap count prominently
