---
name: sales-context-packet
description: |
  Assembles account history, product context, competitive intelligence,
  and approved content assets into a structured context packet for a
  proposal case.
  Use when user asks to "build context packet for [account]",
  "assemble proposal context for [opportunity]",
  "what do we have on [account]",
  "pull account history for proposal",
  "gather context for [proposal ID]",
  or "account research for [deal]".
  Do NOT use for creating a new proposal case (use sales-proposal-intake),
  extracting RFP requirements (use sales-rfp-extraction),
  mapping requirements to approved content (use sales-content-mapping),
  drafting the proposal response (use sales-proposal-draft),
  or routing for approval (use sales-approval-routing).
---

## Overview

Assembles a comprehensive context packet for a proposal case by gathering account history, opportunity data, product and solution fit analysis, prior proposals, approved content assets, competitive context, and key contacts from across CRM, SharePoint, and the Microsoft 365 ecosystem. Produces a Word document that gives the proposal team everything they need to understand the account and available resources before drafting begins.

This skill operates in "AI act within policy" mode — it retrieves context from approved data sources without exercising judgment on deal strategy or customer commitments.

## When to Use

- A proposal case has been created and the team needs account and content context before drafting
- An account executive needs a quick overview of what assets are available for a proposal
- A proposal manager is preparing for a kickoff meeting and needs consolidated account intelligence
- A deal team needs to identify reusable content from prior proposals

## When NOT to Use

- Creating a new proposal case record — use sales-proposal-intake
- Extracting requirements from the RFP document — use sales-rfp-extraction
- Mapping requirements to approved content or identifying gaps — use sales-content-mapping
- Drafting the proposal response document or deck — use sales-proposal-draft
- Routing the proposal package for approval — use sales-approval-routing
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Gather account and opportunity data", activeForm="Researching account context")
TaskCreate(subject="Identify approved content assets", activeForm="Assembling content inventory")
TaskCreate(subject="Produce context packet document", activeForm="Writing context packet")
```

### Step 1: Read Proposal Case Data

- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent` — retrieve the proposal case record including Account Name, Opportunity ID, Product Areas, Deal Size, Complexity Rating

### Step 2: Gather Account Context

**CRM opportunity data:**
- `SearchM365(sources=["connectors"], connector_ids=["dynamics-connector"])` — opportunity stage, deal size, timeline, competitive context, decision makers, buying committee

**Account history:**
- `SearchM365(sources=["files"], query="account plan [account name]")` — account plans, strategic summaries, relationship history
- `SearchM365(sources=["files"], query="[account name] proposal")` — prior proposals for this account
- `SearchM365(sources=["files"], query="[account name] win loss")` — win/loss analysis and post-mortem notes

**Account team:**
- `SearchPeople` — resolve the account team members (AE, SE, CSM, executive sponsor)
- `GetUserDetails` — verify profiles and roles
- `GetManagerDetails` — reporting chain for escalation context

### Step 3: Gather Product and Solution Context

**Product collateral:**
- `SearchM365(sources=["files"], query="[product area] product sheet")` — product data sheets and solution briefs
- `SearchM365(sources=["files"], query="[product area] architecture")` — architecture diagrams and technical specifications

**Competitive positioning:**
- `SearchM365(sources=["files"], query="competitive positioning [competitor]")` — competitive battle cards and differentiation materials
- `SearchM365(sources=["files"], query="[product area] competitive")` — product-area-specific competitive intelligence

### Step 4: Inventory Approved Content Assets

**Browse the approved content library:**
- `SearchM365(sources=["files"], query="approved content library")` then `GetDriveChildren` — browse by category (Technical, Legal, Compliance, Case Studies, Product Sheets, Competitive Positioning)

**Search for relevant approved content:**
- `SearchM365(sources=["files"], query="case study [industry or product area]")` — relevant case studies
- `SearchM365(sources=["files"], query="reference architecture [product area]")` — technical reference materials
- `SearchM365(sources=["files"], query="compliance certification [requirement]")` — compliance and certification documentation

**For each content asset found, capture:**
- Asset name and location
- Category (Technical, Legal, Compliance, Case Study, Product Sheet)
- Last updated date
- Approval status (Approved, Draft, Under Review, Expired)
- Relevance to this proposal

### Step 5: Check for Content Library Connector

If a content management connector is available:
- `SearchM365(sources=["connectors"], connector_ids=["seismic-connector"])` — search for approved content assets indexed from the content management platform

### Step 6: Gather Recent Correspondence

- `SearchM365(sources=["email"], query="[account name] [opportunity context]")` — recent email threads with the customer or about the deal
- `SearchM365(sources=["teams"], query="[account name] proposal")` — deal team discussions in Teams

### Step 7: Produce Context Packet Document

Invoke the `docx` skill to produce a Word document with the following sections:

**1. Account Overview**
- Account name, industry, tier, relationship tenure
- Key contacts and decision makers
- Account health and sentiment (if available from CRM)

**2. Opportunity Summary**
- Opportunity stage, deal size, expected close date
- Product areas and solution components
- Competitive landscape (known competitors in the deal)
- Win probability factors

**3. Prior Proposal History**
- List of prior proposals for this account with outcomes
- Key themes, objections, and lessons from prior engagements
- Reusable content identified from winning proposals

**4. Product and Solution Fit**
- Product capabilities relevant to the opportunity
- Architecture fit and integration considerations
- Known limitations or gaps relevant to the customer's environment

**5. Approved Content Asset Inventory**
- Table of identified content assets with: Asset Name, Category, Last Updated, Approval Status, Relevance
- Flagged assets that are stale (older than 6 months) or under review

**6. Competitive Intelligence Summary**
- Key differentiators vs. known competitors
- Competitive positioning guidance
- Prior win/loss insights for this account or industry

**7. Key Contacts**
- Account team members (AE, SE, CSM, executive sponsor) with roles and contact information
- Customer contacts and decision makers (from CRM data)

**8. Deal Risks and Considerations**
- Timeline risks (tight RFP deadline, complex requirements)
- Competitive risks (incumbent advantage, pricing pressure)
- Content gaps (areas where no approved content exists)

**9. Data Source Inventory**
- Table listing every data source consulted and its retrieval timestamp
- Flag any data that is stale (account data older than 90 days, CRM opportunity in a stale stage)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find proposal tracker, account plans, prior proposals, product collateral, competitive materials, approved content library |
| SearchM365 (connectors) | Pull CRM opportunity data, content management assets via Graph Connectors |
| SearchM365 (email) | Find recent customer correspondence and deal threads |
| SearchM365 (teams) | Find deal team discussions in Teams channels or chats |
| ReadFileContent | Read all account artifacts, content assets, and reference documents |
| GetDriveChildren | Browse approved content library folder structure |
| SearchPeople | Resolve account team members and contacts |
| GetUserDetails | Verify account team profiles and roles |
| GetManagerDetails | Resolve reporting chain for escalation context |

## Guardrails

- **Only surface content from the approved content library** — never include draft, expired, or under-review assets as available content without clearly flagging their status
- **Cite source and last-updated date for every content asset** referenced in the inventory — stale content is a proposal risk
- **Flag if account data is older than 90 days** or if the CRM opportunity is in a stale stage — outdated context leads to misaligned proposals
- **Do not include internal competitive intelligence in customer-facing sections** — the context packet is an internal document, but downstream skills may extract from it; label internal-only sections clearly
- **Do not include customer financial data** (contract value, exact ARR) — account tier and deal size range are sufficient context
- **Never make strategic recommendations** about deal approach or pricing strategy — the context packet presents facts and assets, not advice
- **Flag content gaps prominently** — areas with no approved content need attention before drafting begins
- **Include the Proposal ID in every output** for traceability across the proposal lifecycle
- **Never fabricate account history or opportunity data** — if CRM data is unavailable, report it as unavailable rather than inferring
