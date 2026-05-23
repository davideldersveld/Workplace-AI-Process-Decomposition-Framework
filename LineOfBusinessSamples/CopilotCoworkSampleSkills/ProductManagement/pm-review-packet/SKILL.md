---
name: pm-review-packet
description: |
  Prepares a stakeholder review packet — presentation deck and evidence
  summary — for a product opportunity ready for cross-functional review.
  Use when user asks to "prepare review packet for [opportunity]",
  "build the review deck for [brief]",
  "package this for review",
  "create the stakeholder packet for [opportunity]",
  "get this ready for the review meeting",
  or "review deck for [opportunity ID]".
  Do NOT use for creating a new opportunity case (use pm-request-intake),
  gathering product and customer context (use pm-context-packet),
  detecting duplicates or clustering demand (use pm-demand-cluster),
  drafting the opportunity brief (use pm-opportunity-brief),
  or routing to reviewers (use pm-reviewer-router).
---

## Overview

Packages an approved opportunity brief, context packet, demand evidence, and open questions into a stakeholder-ready review packet. Produces a PowerPoint presentation formatted for a 15-minute review discussion and an optional Word evidence summary for detail-oriented reviewers. Updates the opportunity tracker status to "Review Ready" and records reviewer assignments.

This skill operates in "AI draft plus approve" mode — the review packet is generated as a draft for product manager review and approval. It is never distributed to reviewers without explicit PM confirmation.

## When to Use

- An opportunity brief has been approved by the PM and is ready for cross-functional stakeholder review
- A review meeting is scheduled and the PM needs a presentation deck and evidence summary
- A previously prepared review packet needs to be refreshed after brief updates or new evidence
- Multiple opportunities need standardized review packets for a batch review session

## When NOT to Use

- Creating a new opportunity case — use pm-request-intake
- Gathering product, customer, and telemetry context — use pm-context-packet
- Detecting duplicates or clustering related demand — use pm-demand-cluster
- Drafting the opportunity statement and problem framing — use pm-opportunity-brief
- Routing the review packet to reviewers — use pm-reviewer-router
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read opportunity brief and evidence", activeForm="Gathering review packet inputs")
TaskCreate(subject="Build review packet", activeForm="Building review packet")
```

### Step 1: Read Review Packet Inputs

**Read the opportunity brief:**
- `SearchM365(sources=["files"], query="opportunity brief [opportunity ID]")` then `ReadFileContent` — the approved brief with problem statement, impact, evidence, and open questions

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [opportunity ID]")` then `ReadFileContent` — full evidence foundation

**Read the demand cluster data:**
- Review clustering findings from the tracker — demand volume, related signals, theme

**Read the review packet template:**
- `SearchM365(sources=["files"], query="review packet template")` then `ReadFileContent` — the organization's standard review deck template
- `SearchM365(sources=["files"], query="review checklist")` then `ReadFileContent` — reviewer checklist for completeness validation

**Read supporting evidence:**
- `SearchM365(sources=["files"], query="[product area] customer feedback")` then `ReadFileContent` — customer feedback exports relevant to this opportunity
- `SearchM365(sources=["files"], query="[product area] competitor analysis")` then `ReadFileContent` — competitive context if available

**Read the reviewer list:**
- `SearchM365(sources=["files"], query="reviewer matrix")` then `ReadFileContent` — which roles review which product areas

### Step 2: Build the PowerPoint Review Deck

Produce a PowerPoint presentation (invoke `pptx` skill) formatted for a 15-minute review discussion:

**Slide 1 — Title**
- Opportunity title
- Opportunity ID
- Product Area
- Date
- "DRAFT — For Review" label

**Slide 2 — Problem Statement**
- User-centric problem framing (2-3 bullet points from the brief)
- Who is affected and why it matters

**Slide 3 — Evidence Highlights**
- Demand volume (unique source count, channel distribution)
- Customer tier breakdown
- Key customer signals (paraphrased themes, not individual quotes)
- Telemetry indicators (if available)

**Slide 4 — Business Impact**
- Customer reach and strategic alignment
- Competitive context (if available)
- Revenue risk or growth opportunity indicators (tier-level)

**Slide 5 — Related Demand and Backlog**
- Theme cluster summary
- Existing backlog items and their status
- Prior decisions on similar requests

**Slide 6 — Open Questions**
- Key unknowns requiring discovery
- Dependencies and risks
- Evidence gaps

**Slide 7 — Recommended Next Steps**
- Suggested disposition options (proceed to prioritization, request more discovery, defer, decline)
- What the PM recommends and why
- Review timeline and decision deadline

**Slide 8 — Traceability**
- Links to source documents (context packet, brief, original requests)
- Evidence source inventory

### Step 3: Build the Word Evidence Summary (Optional)

If the user requests a detailed review document or the opportunity is high-complexity, produce a Word document (invoke `docx` skill) containing:

1. **Executive Summary** — 1-paragraph opportunity overview
2. **Full Problem Statement** — expanded from the brief
3. **Complete Evidence Appendix** — all demand signals, customer context, telemetry data, and backlog relationships with source citations
4. **Competitive Analysis** — detailed competitive context (if available)
5. **Risk Assessment** — feasibility risks, dependency risks, and evidence quality assessment
6. **Open Questions Deep Dive** — expanded context for each open question
7. **Traceability Index** — complete mapping of every evidence claim to its source document

### Step 4: Update Tracker

Update the opportunity tracker:
- Status: "Review Ready"
- Reviewer Assignments: list of reviewers (from reviewer matrix)
- Review Deadline: target date based on review timeline
- Link the review deck and evidence summary in Evidence Links

### Step 5: Present Packet for PM Review

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Opportunity title** — from the brief
- **Packet contents** — list of generated artifacts (deck, evidence summary, tracker update)
- **Slide count and estimated review time** — targeting 15 minutes
- **Evidence completeness** — which data sources contributed and which are missing
- **Reviewer list** — who will receive the packet (from reviewer matrix)
- **Open questions count** — how many unknowns are surfaced
- **Template compliance** — confirmation that the deck follows the organization's template
- **Draft label** — "REVIEW PACKET DRAFT — PM approval required before distributing to reviewers"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find opportunity brief, context packet, review template, reviewer checklist, customer feedback, competitor analysis, reviewer matrix |
| ReadFileContent | Read all input documents from SharePoint |

## Guardrails

- **Always generate as a draft for PM review** — never distribute to reviewers without explicit PM confirmation
- **Every evidence claim in the deck must trace to a source document** — unsourced claims on slides undermine reviewer trust
- **Include the Opportunity ID and "DRAFT" label on every slide** — prevents the deck from being treated as an approved commitment
- **Clearly distinguish confirmed evidence from hypotheses** — slides must visually separate facts from open questions
- **Never include delivery estimates, cost projections, or commitment language** — the review packet supports a prioritization decision, not a delivery plan
- **Match the organization's review deck template** if one exists in SharePoint — do not invent a non-standard layout
- **Include the Open Questions slide** prominently — hiding unknowns from reviewers is worse than surfacing them; honest gaps invite better discussion
- **Never include exact customer revenue figures or contract values** — use tier-level indicators consistent with the brief
- **Never fabricate competitive intelligence or market data** — if competitive context is unavailable, omit the section rather than speculating
- **Target a 15-minute review discussion** — the deck should be concise enough for a focused review; detailed evidence goes in the optional Word summary
- **For sensitive opportunities** (competitive intelligence, unreleased strategy), flag the packet for restricted distribution — only designated reviewers should have access
- **Flag if the opportunity brief has been modified since the review packet was generated** — stale packets may not reflect current evidence
