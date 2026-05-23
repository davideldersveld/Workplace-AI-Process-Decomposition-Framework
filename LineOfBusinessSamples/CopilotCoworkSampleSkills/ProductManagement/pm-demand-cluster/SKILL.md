---
name: pm-demand-cluster
description: |
  Detects duplicate feature requests and clusters related demand signals
  into themes for a product opportunity case.
  Use when user asks to "check for duplicates on [opportunity]",
  "cluster related requests for [feature]",
  "is this a duplicate of [existing opportunity]",
  "group similar feature requests",
  "find related opportunities for [request]",
  or "demand analysis for [product area]".
  Do NOT use for creating a new opportunity case (use pm-request-intake),
  gathering product and customer context (use pm-context-packet),
  drafting the opportunity brief (use pm-opportunity-brief),
  preparing the stakeholder review packet (use pm-review-packet),
  or routing to reviewers (use pm-reviewer-router).
---

## Overview

Compares a current opportunity case against the full opportunity tracker, backlog history, and prior briefs to identify exact duplicates, near-duplicates, and thematic clusters. Surfaces demand volume, customer overlap, and suggested theme groupings for product manager review. Presents findings via Adaptive Card as recommendations only — merge, link, and theme assignment decisions are always PM-owned.

This skill operates in "AI assist" mode — it reads and analyzes opportunity data but only presents findings as recommendations. The product manager reviews and confirms before any tracker updates are made.

## When to Use

- A new opportunity case needs to be checked for duplicates against the existing tracker and backlog
- A product manager wants to identify demand patterns across multiple requests in the same product area
- New requests have accumulated and need to be grouped into themes before opportunity framing
- A product area review needs a demand volume analysis to support prioritization discussions

## When NOT to Use

- Creating a new opportunity case — use pm-request-intake
- Gathering product, customer, and telemetry context — use pm-context-packet
- Drafting the opportunity statement and problem framing — use pm-opportunity-brief
- Preparing the stakeholder review packet — use pm-review-packet
- Routing the review packet to reviewers — use pm-reviewer-router
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read opportunity data and backlog", activeForm="Analyzing demand signals")
TaskCreate(subject="Present clustering recommendations", activeForm="Clustering related demand")
```

### Step 1: Read Opportunity and Backlog Data

**Read the current opportunity case:**
- `SearchM365(sources=["files"], query="opportunity tracker")` then `ReadFileContent` — full case data including Opportunity ID, Product Area, Summary, Customer/Account, Evidence Links

**Read the full opportunity tracker:**
- `ReadFileContent` — all existing entries in the tracker for comparison

**Read backlog history:**
- `SearchM365(sources=["files"], query="backlog snapshot [product area]")` then `ReadFileContent` — active and recently closed backlog items

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])` — related backlog items and their current status

**Read prior opportunity briefs:**
- `SearchM365(sources=["files"], query="opportunity brief [product area]")` — approved, deferred, and declined briefs for similar features

**Read the feature taxonomy:**
- `SearchM365(sources=["files"], query="feature taxonomy")` then `ReadFileContent` — standard theme categories and product area hierarchy

### Step 2: Identify Exact Duplicates

Compare the current request against existing tracker entries and backlog items to find exact duplicates:

| Match Type | Criteria | Confidence |
|-----------|---------|------------|
| **Exact duplicate** | Same feature from the same customer through a different channel | High |
| **Cross-channel duplicate** | Same feature from the same customer submitted via both email and Teams | High |
| **Resubmission** | Same feature from the same customer previously submitted and closed (deferred or declined) | High |

For each exact duplicate found:
- Existing Opportunity ID or backlog item ID
- Original submission date and current status
- Disposition (if previously decided)
- Customer and channel comparison

### Step 3: Identify Near-Duplicates and Related Demand

Compare the current request against the tracker and backlog to find related signals:

| Match Type | Criteria | Confidence |
|-----------|---------|------------|
| **Near-duplicate** | Same feature with different framing from a different customer | Medium-High |
| **Related feature** | Different feature in the same product area addressing a similar user problem | Medium |
| **Adjacent demand** | Feature in a related product area that may share implementation dependencies | Low-Medium |

For each related signal:
- Opportunity ID or backlog item ID
- Summary comparison showing what overlaps and what differs
- Customer and channel source
- Confidence level with basis for the match

### Step 4: Calculate Demand Volume

Aggregate demand signals for the current request and its cluster:

- **Unique sources** — how many distinct customers, accounts, or internal teams have requested this or similar features
- **Channel distribution** — breakdown by source channel (sales, support, CAB, internal, analytics)
- **Customer tier distribution** — breakdown by customer tier (enterprise, mid-market, SMB)
- **Time span** — date range of the earliest to most recent related request
- **Trend** — increasing, stable, or decreasing request volume over time

### Step 5: Suggest Theme Grouping

Based on the clustering analysis, recommend a theme grouping:

- **Theme name** — descriptive label (e.g., "Mobile Experience Enhancements", "Reporting Customization", "API Integration Requests")
- **Theme scope** — which specific features and requests fall under this theme
- **Theme maturity** — whether this is a new theme, a growing cluster, or an established demand area
- **Alignment to taxonomy** — how the suggested theme maps to the existing feature taxonomy

If the request does not cluster with any existing signals, note it as a standalone demand signal.

### Step 6: Present Clustering Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Opportunity ID, Product Area, Summary, Customer/Account
- **Exact duplicates** — list with IDs, status, and recommended action (link, merge, or keep separate)
- **Near-duplicates and related demand** — list with confidence levels and match basis
- **Demand volume** — unique source count, channel distribution, customer tier breakdown
- **Suggested theme** — theme name, scope, and taxonomy alignment
- **Confidence summary** — overall clustering confidence with basis
- **Resubmission alert** — if this request was previously declined or deferred, show the prior decision and rationale
- **Draft label** — "CLUSTERING RECOMMENDATION — PM review required before tracker update"

### Step 7: Update Tracker (After Confirmation)

After the PM confirms the clustering findings:
- Update the Duplicate Flag field with confirmed duplicate links
- Update the Theme field with the confirmed theme assignment
- Link related opportunities in the Evidence Links field

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find opportunity tracker, backlog snapshots, prior briefs, feature taxonomy |
| SearchM365 (connectors) | Pull backlog items from Jira connector for deduplication |
| ReadFileContent | Read tracker, backlog data, taxonomy, prior briefs |

## Guardrails

- **Present clustering as recommendations only** — never auto-merge, auto-link, or auto-close opportunities without PM confirmation
- **Always show the basis for each match** — which keywords, product area, customer overlap, or feature similarity drove the clustering decision
- **Include confidence levels for every match** — high confidence (above 80%) means strong overlap; medium (50-80%) means likely related; low (below 50%) is flagged as "possible" rather than "likely"
- **Never auto-close a request as a duplicate** — only flag and recommend; the PM decides whether to merge, link, or keep as a separate opportunity
- **Preserve the original request text** even when linking to a cluster — do not overwrite or summarize away the requester's exact words
- **Flag resubmissions prominently** — if a request was previously declined or deferred, show the prior decision and rationale so the PM can decide whether circumstances have changed
- **Never fabricate demand volume or customer counts** — report only what the tracker and backlog data support; if data is incomplete, state the limitation
- **Use the existing feature taxonomy** for theme suggestions when possible — invent new theme names only when no existing category fits
- **Never assess opportunity priority or merit** based on demand volume alone — clustering provides evidence, not decisions; high volume does not automatically mean high priority
- **Distinguish between customer-sourced demand and internal demand** — these carry different signals for product prioritization
