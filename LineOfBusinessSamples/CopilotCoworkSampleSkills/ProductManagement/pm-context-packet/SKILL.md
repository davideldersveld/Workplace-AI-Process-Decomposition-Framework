---
name: pm-context-packet
description: |
  Assembles product, customer, telemetry, and backlog context for a
  feature request opportunity case into a structured context packet.
  Use when user asks to "build context for this request",
  "gather product context for [opportunity]",
  "what do we know about this opportunity",
  "assemble evidence for [feature request]",
  "pull context for opportunity [ID]",
  or "get background on [feature request]".
  Do NOT use for creating a new opportunity case (use pm-request-intake),
  detecting duplicates or clustering demand (use pm-demand-cluster),
  drafting the opportunity brief (use pm-opportunity-brief),
  preparing the stakeholder review packet (use pm-review-packet),
  or routing to reviewers (use pm-reviewer-router).
---

## Overview

Assembles a comprehensive context packet for an opportunity case by gathering customer profile data, product area context, related backlog items, prior decisions on similar requests, telemetry summaries, and recent correspondence. Produces a Word document that serves as the evidence foundation for all downstream skills — demand clustering, opportunity brief drafting, and review packet preparation.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined data sources and policy repositories without exercising judgment on opportunity merit or priority.

## When to Use

- An opportunity case has been created via pm-request-intake and needs product, customer, and evidence context assembled
- A product manager wants to understand the full background behind a feature request before framing it as an opportunity
- New information has arrived and the context packet needs to be refreshed
- A reviewer needs comprehensive context before starting their review

## When NOT to Use

- Creating a new opportunity case — use pm-request-intake
- Detecting duplicates or clustering related demand — use pm-demand-cluster
- Drafting the opportunity statement and problem framing — use pm-opportunity-brief
- Preparing the stakeholder review packet — use pm-review-packet
- Routing the review packet to reviewers — use pm-reviewer-router
- Confirming disposition (accept, defer, decline) — this is always a human decision (PM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read opportunity case and gather context", activeForm="Gathering product context")
TaskCreate(subject="Build context packet document", activeForm="Building context packet")
```

### Step 1: Read the Opportunity Case

- `SearchM365(sources=["files"], query="opportunity tracker")` then `ReadFileContent` — find the case record by Opportunity ID or supplier name
- Extract: Opportunity ID, Request Source, Requester, Customer/Account, Product Area, Summary, Evidence Links

### Step 2: Gather Customer Context

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` — customer account details including tier, ARR, segment, churn risk, prior requests

**Via SharePoint (if manual sync):**
- `SearchM365(sources=["files"], query="customer account summary [customer name]")` then `ReadFileContent` — Power Automate-synced customer data

**Via people tools:**
- `SearchPeople` — resolve customer contacts and internal account owners
- `GetUserDetails` — pull requester profile and organizational context

Assemble customer profile:
- Account name and tier
- Customer segment
- ARR or contract context (tier-level only, not exact financial figures)
- Churn risk indicators (if available)
- Prior feature requests from this customer
- Relationship history summary

### Step 3: Gather Product Area Context

- `SearchM365(sources=["files"], query="[product area] roadmap")` then `ReadFileContent` — current roadmap position for the relevant product area
- `SearchM365(sources=["files"], query="[product area] product strategy")` then `ReadFileContent` — strategic priorities and principles
- `SearchM365(sources=["files"], query="product glossary")` then `ReadFileContent` — terminology and feature taxonomy
- `SearchM365(sources=["files"], query="[product area] recent launches")` then `ReadFileContent` — recent releases and known gaps

Assemble product area context:
- Current roadmap position (in-flight initiatives, planned work, known gaps)
- Strategic alignment indicators (does this request align with stated strategy?)
- Recent launches in the area
- Known limitations or technical constraints

### Step 4: Gather Backlog and Prior Decision Context

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["jira-connector"])` — related backlog items, epics, and their current status

**Via SharePoint (if manual sync):**
- `SearchM365(sources=["files"], query="backlog snapshot [product area]")` then `ReadFileContent` — active and recently closed backlog items

Assemble backlog context:
- Existing backlog items related to this request (with status: open, in progress, completed, declined)
- Prior requests for the same or similar features (with disposition: approved, deferred, declined, and rationale)
- Planned work that may overlap or conflict

### Step 5: Gather Telemetry and Support Context

**Via SharePoint (published reports):**
- `SearchM365(sources=["files"], query="[product area] usage report")` then `ReadFileContent` — product analytics summaries
- `SearchM365(sources=["files"], query="[product area] support volume")` then `ReadFileContent` — support ticket trends

Assemble telemetry context:
- Usage data for the relevant product area (adoption, engagement, feature utilization)
- Support ticket volume and common issues related to the request
- Churn or retention indicators if relevant

### Step 6: Gather Correspondence Context

- `SearchM365(sources=["email"], query="[customer name] [feature keywords]")` — recent email threads mentioning the customer or feature area
- `SearchM365(sources=["teams"], query="[feature keywords] [product area]")` — related discussions in product, sales, and support Teams channels

Summarize relevant correspondence:
- Key stakeholder sentiments
- Additional context not captured in the original request
- Urgency signals or escalation history

### Step 7: Assemble the Context Packet

Produce a Word document (invoke `docx` skill) with the following sections:

1. **Request Summary** — Opportunity ID, source, requester, original request text
2. **Customer Profile** — account tier, segment, prior requests, relationship summary
3. **Product Area Context** — roadmap position, strategic alignment, recent launches, known gaps
4. **Related Backlog Items** — existing features, planned work, declined requests with rationale
5. **Telemetry Summary** — usage data, support volume, adoption metrics (or "Not available" if data sources are missing)
6. **Prior Decisions** — past decisions on similar requests with documented rationale
7. **Correspondence Summary** — key themes from email and Teams discussions
8. **Data Source Inventory** — which sources were consulted and which returned no data or were unavailable
9. **Evidence Gaps** — specific context that is missing and where it might be found

Save to the SharePoint product opportunities folder and link from the tracker row.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find opportunity tracker, product strategy docs, roadmap, glossary, backlog snapshots, telemetry reports |
| SearchM365 (connectors) | Pull customer data from CRM connector, backlog items from Jira connector |
| SearchM365 (email) | Find related customer and stakeholder correspondence |
| SearchM365 (teams) | Find related discussions in product channels |
| ReadFileContent | Read all context source documents from SharePoint |
| SearchPeople | Resolve customer contacts and account owners |
| GetUserDetails | Pull requester and stakeholder profiles |

## Guardrails

- **Cite source document and retrieval date for every data point** — the context packet is an evidence document; every claim must be traceable
- **Clearly distinguish confirmed data from inferred context** — system-sourced data (CRM, backlog, telemetry) is labeled as confirmed; email and chat-sourced context is labeled as inferred
- **Flag if any critical context source returned no data or is unavailable** — missing CRM data, missing backlog history, or missing telemetry are explicitly noted in the Data Source Inventory section
- **Never include exact revenue figures, contract values, or financial projections** — use tier-level data (enterprise, mid-market, SMB) and general indicators; exact financials belong in restricted CRM records
- **If customer data is sensitive** (enterprise contract, NDA-protected feedback), note the sensitivity and recommend restricted distribution to the assigned PM only
- **Never fabricate context** — if a data source is unavailable, report it as unavailable rather than guessing
- **Never assess opportunity merit or priority** — the context packet assembles evidence; the PM evaluates merit in the opportunity brief step
- **Include the Evidence Gaps section** — reviewers need to know what context is missing, not just what was found
- **Timestamp all evidence citations** — if the context packet is refreshed later, stale citations need to be identifiable
- **Never include customer quotes attributed to specific individuals** without verifying the source — paraphrase or cite the document rather than attributing statements
