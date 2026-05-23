---
name: mktg-brief-draft
description: |
  Drafts a structured campaign brief using the context packet, extracted
  signals, approved messaging, and campaign brief templates.
  Use when user asks to "draft the campaign brief",
  "create brief for [campaign]", "build the brief document",
  "assemble campaign brief for [name]",
  "write campaign brief for [ID]",
  "prepare the brief deck for [campaign]",
  or "generate campaign brief from context".
  Do NOT use for creating a new campaign case (use mktg-campaign-intake),
  assembling brand and product context (use mktg-context-packet),
  extracting goals and constraints (use mktg-signal-extraction),
  or routing for review (use mktg-review-routing).
---

## Overview

Drafts a structured campaign brief by synthesizing the context packet, extracted signals, approved brand messaging, and campaign brief templates into a cohesive document. The brief can be produced as a Word document (detailed narrative brief) or PowerPoint deck (stakeholder alignment presentation), following the approved template format. Every claim and messaging element traces to an approved source.

This skill operates in "AI draft plus approve" mode — every brief draft is presented for campaign manager review and confirmation. The draft is never finalized or distributed without explicit approval.

## When to Use

- The context packet and signal extraction are complete and the campaign needs a brief
- A campaign manager wants to generate a first draft from assembled inputs
- A brief needs revision after new signals or updated context
- A stakeholder alignment deck is needed alongside the detailed brief

## When NOT to Use

- Creating a new campaign case — use mktg-campaign-intake
- Assembling brand, product, and audience context — use mktg-context-packet
- Extracting goals, constraints, and dependencies — use mktg-signal-extraction
- Routing for brand, product, or legal review — use mktg-review-routing
- Confirming brief baseline — this is always a human decision (MK-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read inputs and campaign brief template", activeForm="Reading brief inputs")
TaskCreate(subject="Draft campaign brief for review", activeForm="Drafting campaign brief")
```

### Step 1: Read Brief Inputs

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [Campaign ID or name]")` then `ReadFileContent`

**Read the signal matrix:**
- `SearchM365(sources=["files"], query="signal matrix [Campaign ID or name]")` then `ReadFileContent`
- Or read from the campaign tracker if signals are stored there

**Read the campaign brief template:**
- `SearchM365(sources=["files"], query="campaign brief template")` then `ReadFileContent`
- Identify the correct template variant for this campaign type (product launch, event, demand gen, brand, field marketing)

**Read approved messaging sources:**
- `SearchM365(sources=["files"], query="[product or solution] approved messaging")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[product or solution] approved claims")` then `ReadFileContent`

**Read prior successful briefs (for reference):**
- `SearchM365(sources=["files"], query="[campaign type] campaign brief approved")` then `ReadFileContent` — examples of approved briefs for similar campaign types

### Step 2: Draft the Campaign Brief

Generate the brief following the approved template structure. Standard sections:

**1. Executive Summary**
- Campaign name, type, and business context
- One-paragraph overview of the campaign's purpose and expected outcome
- Source: signal matrix (objectives), context packet (business context)

**2. Campaign Objective**
- Primary objective and measurable KPIs
- Business outcome the campaign supports
- Source: signal matrix (objectives, KPIs)

**3. Target Audience**
- Primary audience definition with demographics, firmographics, and pain points
- Secondary audience (if applicable)
- Source: context packet (audience profile), signal matrix (audience signals)

**4. Key Messages**
- Primary messaging theme aligned to approved messaging pillars
- Supporting messages with approved claims and proof points
- Call to action
- Source: context packet (product messaging), approved claims documents — cite every claim

**5. Channel Strategy**
- Recommended channel mix with rationale
- Channel-specific requirements (landing pages, email templates, social assets)
- Source: context packet (channel recommendations), signal matrix (channel requirements)

**6. Deliverables and Timeline**
- Complete deliverable list with asset types and specifications
- Production timeline with key milestones
- Source: signal matrix (deliverables, timing)

**7. Budget**
- Budget range and allocation by channel or activity
- Source: signal matrix (budget parameters) — use ranges unless confirmed figures are available

**8. Success Metrics**
- How the campaign will be measured
- Benchmark targets from prior campaigns (if available)
- Source: signal matrix (KPIs), context packet (prior campaign performance)

**9. Open Questions**
- Unresolved signals, conflicting requirements, and missing inputs
- Items requiring stakeholder decision before finalization
- Source: signal matrix (unconfirmed, conflicting signals)

**10. Required Reviews**
- Brand review, product marketing review, legal review (if applicable)
- Review timeline based on campaign priority and launch date
- Source: campaign tracker (specialized review flags)

### Step 3: Choose Output Format

Based on user request or campaign needs:

**Word document** (invoke `docx` skill) — for the detailed narrative brief that becomes the campaign's governing document. Include all sections with full detail and source citations.

**PowerPoint deck** (invoke `pptx` skill) — for stakeholder alignment presentations. Include: executive summary, objective, audience visual, messaging framework, channel mix visual, timeline, and open questions. Visual and concise.

**Both** — generate the Word brief as the detailed record and the PowerPoint deck as the presentation artifact.

### Step 4: Present Draft Status

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Campaign header** — Campaign ID, Name, Type
- **Sections completed** — sections with sufficient input to draft
- **Sections needing stakeholder input** — sections with gaps or unconfirmed signals
- **Sections flagged for specialist review** — sections with claims needing product marketing validation or legal review
- **Messaging source coverage** — percentage of claims and messages traced to approved sources
- **Open questions count** — unresolved items requiring decision
- **Draft label** — "CAMPAIGN BRIEF DRAFT — campaign manager review required before distribution"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find context packet, signal matrix, campaign tracker, brief templates, approved messaging, prior briefs |
| ReadFileContent | Read all input artifacts — context packet, signals, templates, messaging documents |

## Guardrails

- **Always create as draft** — never finalize or distribute the brief without explicit campaign manager confirmation
- **Every claim and messaging element must trace to an approved brand or product messaging source** — no fabricated value propositions, statistics, or unsupported claims
- **Flag sections where approved messaging does not fully cover the campaign's needs** — these require product marketing input before the brief can be finalized
- **Budget sections must use ranges from the intake** rather than specific figures unless confirmed by the campaign manager
- **Include source citations** for every messaging block, claim, and brand element used in the brief — reviewers need to verify provenance
- **Match the approved template formatting standards** — briefs that deviate from the template structure create confusion during review
- **Flag unresolved signals and open questions prominently** — a brief with hidden gaps creates downstream rework
- **Never generate product claims or value propositions that are not backed by approved messaging** — this is especially critical for regulated industries
- **Never include internal competitive positioning** in external-facing sections of the brief — competitive context is for internal strategy only
- **Never include personally identifiable audience data** — use segment-level abstractions only
- **Never fabricate performance data, benchmarks, or statistics** — if prior campaign data is not available, state the gap
- **Never commit to timelines or budgets** that have not been confirmed by the campaign manager — use "proposed" or "estimated" language
