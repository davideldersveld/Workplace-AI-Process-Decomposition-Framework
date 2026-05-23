---
name: pm-opportunity-brief
description: |
  Drafts an evidence-backed opportunity brief with problem statement,
  business impact, open questions, and source traceability for a
  feature request opportunity case.
  Use when user asks to "draft opportunity brief for [opportunity]",
  "write the opportunity statement for [request]",
  "frame this as an opportunity",
  "create the problem statement for [feature]",
  "draft the brief for [feature request]",
  or "write up the opportunity for [case ID]".
  Do NOT use for creating a new opportunity case (use pm-request-intake),
  gathering product and customer context (use pm-context-packet),
  detecting duplicates or clustering demand (use pm-demand-cluster),
  preparing the stakeholder review packet (use pm-review-packet),
  or routing to reviewers (use pm-reviewer-router).
---

## Overview

Drafts a structured opportunity brief following the organization's template — including a user-centric problem statement, business impact assessment, evidence summary, open questions, and full traceability to source signals. The brief is the core synthesis artifact that transforms fragmented demand signals into a clear, review-ready framing of a product opportunity. Produced as a Word document with explicit "DRAFT — NOT COMMITTED" labeling.

This skill operates in "AI draft plus approve" mode — the brief is generated as a draft for product manager review, editing, and approval. It is never published or distributed without explicit PM confirmation.

## When to Use

- A context packet and demand clustering analysis are available and the opportunity needs to be framed as a structured brief
- A product manager wants a first draft of the opportunity statement to refine and edit
- An existing brief needs to be refreshed after new context or demand signals have been added
- A product area review needs a standardized brief for each opportunity under consideration

## When NOT to Use

- Creating a new opportunity case — use pm-request-intake
- Gathering product, customer, and telemetry context — use pm-context-packet
- Detecting duplicates or clustering related demand — use pm-demand-cluster
- Preparing the stakeholder review packet — use pm-review-packet
- Routing the review packet to reviewers — use pm-reviewer-router
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read context packet and brief template", activeForm="Preparing opportunity brief inputs")
TaskCreate(subject="Draft opportunity brief", activeForm="Drafting opportunity brief")
```

### Step 1: Read Brief Inputs

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [opportunity ID or supplier name]")` then `ReadFileContent` — full customer, product, backlog, and telemetry context

**Read the demand cluster output:**
- Review the clustering findings from the opportunity tracker — demand volume, related signals, theme assignment

**Read the opportunity brief template:**
- `SearchM365(sources=["files"], query="opportunity brief template")` then `ReadFileContent` — the organization's standard template structure

**Read product strategy principles:**
- `SearchM365(sources=["files"], query="product strategy principles")` then `ReadFileContent` — strategic priorities that inform how the brief should position the opportunity

**Read exemplar briefs:**
- `SearchM365(sources=["files"], query="approved opportunity brief")` then `ReadFileContent` — 2-3 previously approved briefs for tone, structure, and depth calibration

**Read the prioritization rubric:**
- `SearchM365(sources=["files"], query="prioritization rubric")` then `ReadFileContent` — criteria that reviewers will use to evaluate the brief

### Step 2: Draft the Opportunity Brief

Produce a Word document (invoke `docx` skill) following the organization's template structure:

**1. Header**
- Opportunity ID
- Product Area
- Draft date
- Status: "DRAFT — NOT COMMITTED"
- Assigned PM (if known)

**2. Opportunity Title**
- Clear, descriptive title that captures the user problem and product area

**3. Problem Statement**
- User-centric framing: who is affected, what they are trying to do, and why the current experience falls short
- Evidence-backed: every assertion traces to a source in the context packet
- Scope-bounded: what is included and what is explicitly excluded

**4. Business Impact**
- Customer reach: how many customers, accounts, or users are affected (from demand cluster data)
- Revenue context: customer tier distribution, churn risk indicators (tier-level, not exact figures)
- Strategic alignment: how this opportunity maps to stated product strategy principles
- Competitive context: relevant competitive pressure (if available from context packet)

**5. Evidence Summary**
- Demand volume: number of unique request sources, channel distribution
- Customer signals: key themes from customer requests (paraphrased, not attributed to individuals without verification)
- Telemetry indicators: usage data, support volume, adoption metrics (if available)
- Backlog relationship: how this relates to existing backlog items, planned work, or prior decisions

**6. Open Questions**
- Unknowns that need discovery before the opportunity can be prioritized
- Dependencies on other teams, systems, or initiatives
- Risks that could affect feasibility or impact
- Areas where evidence is thin and additional data gathering is recommended

**7. Traceability**
- Links to the original request sources
- Link to the context packet document
- Link to the demand cluster analysis
- Links to related backlog items and prior decisions

Save to the SharePoint product opportunities folder and link from the tracker row.

### Step 3: Update Tracker Status

Update the opportunity tracker:
- Status: "Brief Drafted"
- Link the brief document in the Evidence Links field

### Step 4: Present Draft for Review

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Opportunity title** — the proposed title
- **Problem statement preview** — first 2-3 sentences
- **Evidence strength** — summary of demand volume and source count
- **Open questions count** — number of unresolved questions
- **Template compliance** — confirmation that all required sections are present
- **Draft label** — "OPPORTUNITY BRIEF DRAFT — PM review and approval required before advancing"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find context packet, brief template, strategy principles, exemplar briefs, prioritization rubric |
| ReadFileContent | Read all input documents from SharePoint |

## Guardrails

- **Always generate as a draft for PM review** — never publish, distribute, or advance the brief without explicit PM confirmation
- **Follow the organization's brief template structure exactly** — do not invent new sections or omit required sections; if no template is found, use the standard structure defined in Step 2
- **Every claim in the brief must trace to a source** in the context packet or demand cluster output — unsourced assertions are not permitted
- **Clearly separate confirmed facts from hypotheses and open questions** — use explicit labels ("Confirmed:", "Hypothesis:", "Open Question:")
- **Never include delivery estimates, timelines, or commitment language** — the brief is a problem framing artifact, not a project plan or roadmap commitment
- **Never state that a feature "will be built" or "is planned"** — use language like "is being evaluated", "is under consideration", or "has been identified as an opportunity"
- **Include a prominent "DRAFT — NOT COMMITTED" label** in the document header — this prevents the brief from being mistaken for an approved roadmap item
- **Never include exact revenue figures or financial projections** — use tier-level indicators and general business impact language
- **Include all open questions and evidence gaps** rather than presenting false certainty — an honest brief with acknowledged unknowns is more useful than a confident brief with hidden gaps
- **Never fabricate customer quotes, demand figures, or telemetry data** — if evidence is unavailable, note the gap in the Evidence Summary and Open Questions sections
- **Respect the prioritization rubric criteria** — structure the brief so reviewers can easily evaluate it against the organization's prioritization framework
- **If the context packet has been modified since the brief was generated**, flag the brief as potentially stale and recommend a refresh
