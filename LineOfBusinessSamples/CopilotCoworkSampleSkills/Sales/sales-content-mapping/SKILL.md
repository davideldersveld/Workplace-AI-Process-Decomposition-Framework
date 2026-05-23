---
name: sales-content-mapping
description: |
  Maps extracted RFP requirements to approved content assets and
  identifies coverage gaps for a proposal case.
  Use when user asks to "map requirements to content for [proposal ID]",
  "find approved answers for RFP",
  "identify response gaps for [account]",
  "content coverage check for [proposal]",
  "what content do we have for these requirements",
  or "gap analysis for [RFP]".
  Do NOT use for creating a new proposal case (use sales-proposal-intake),
  gathering account context (use sales-context-packet),
  extracting RFP requirements (use sales-rfp-extraction),
  drafting the proposal response (use sales-proposal-draft),
  or routing for approval (use sales-approval-routing).
---

## Overview

Compares extracted RFP requirements against the approved content library to determine which requirements have approved responses, which have partial coverage, which have no coverage (gaps), and which have stale content that needs freshness review. Produces a coverage analysis as an Adaptive Card and updates the requirements matrix with coverage status, matched assets, and gap notes. Surfaces the gap list for content creation prioritization before drafting begins.

This skill operates in "AI assist" mode — it analyzes content coverage and presents recommendations for proposal manager review. Coverage decisions and gap prioritization are confirmed before tracker updates.

## When to Use

- Requirements have been extracted and need to be mapped to available approved content
- The proposal team needs to identify which requirements can be answered with existing content vs. which need new drafting
- A proposal manager wants to prioritize content creation effort based on coverage gaps
- Requirements have been updated (amendment, clarification) and coverage needs re-evaluation

## When NOT to Use

- Creating a new proposal case record — use sales-proposal-intake
- Gathering account history and content context — use sales-context-packet
- Extracting requirements from the RFP document — use sales-rfp-extraction
- Drafting the proposal response document or deck — use sales-proposal-draft
- Routing the proposal package for approval — use sales-approval-routing
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read requirements and search approved content", activeForm="Mapping content coverage")
TaskCreate(subject="Present coverage analysis and gap report", activeForm="Analyzing content gaps")
```

### Step 1: Read Mapping Inputs

**Read the requirements matrix:**
- `SearchM365(sources=["files"], query="[proposal ID] requirements")` then `ReadFileContent` — full requirements matrix with Requirement ID, Category, Priority, Requirement Text

**Read the proposal case data:**
- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent` — Proposal ID, Account Name, Product Areas, Due Date

**Read the context packet (if available):**
- `SearchM365(sources=["files"], query="context packet [proposal ID]")` then `ReadFileContent` — previously identified content assets and account context

### Step 2: Search Approved Content Library

For each requirement (or group of related requirements), search the approved content library:

**SharePoint content library:**
- `SearchM365(sources=["files"], query="[requirement keywords] approved")` — search for approved content matching the requirement
- `GetDriveChildren` — browse the content library by category folder (Technical, Legal, Compliance, Case Studies, Product Sheets)

**Content management connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["seismic-connector"])` — search for approved content assets indexed from the content management platform

**Prior proposals:**
- `SearchM365(sources=["files"], query="[account name] proposal [product area]")` — search for reusable content from prior winning proposals

### Step 3: Categorize Coverage for Each Requirement

| Coverage Status | Criteria | Action Needed |
|----------------|---------|---------------|
| **Covered** | An approved content asset directly addresses the requirement; content is current (updated within 6 months) | Minor tailoring for this specific proposal |
| **Partial** | Related content exists but does not fully address the requirement, or content addresses a similar but not identical requirement | Adaptation and supplementation needed |
| **Gap** | No approved content found that addresses this requirement | New content must be drafted |
| **Stale** | Content exists but is older than 6 months or is flagged as "under review" | Freshness review required before use |

### Step 4: Capture Coverage Details

For each requirement, record:

| Field | Description |
|-------|------------|
| **Coverage Status** | Covered, Partial, Gap, or Stale |
| **Matched Content Asset** | Name and SharePoint location of the matched content (if any) |
| **Content Last Updated** | Date the matched content was last modified |
| **Approval Status** | Approved, Draft, Under Review, Expired |
| **Match Confidence** | High, Medium, Low — based on how closely the content addresses the requirement |
| **Gap Notes** | What is missing, what needs adaptation, or what needs to be freshness-reviewed |

### Step 5: Apply Specialist Routing Rules

Certain requirement categories always require specialist review regardless of content coverage:

| Category | Rule |
|----------|------|
| **Pricing** | Never mark as "Covered" — always route to deal desk for pricing review |
| **Legal** | Never mark as "Covered" — always route to legal for terms review |
| **Compliance** | Route to compliance team for certification verification even if content exists |
| **Security** | Route to security team for current posture verification |

### Step 6: Calculate Coverage Summary

Produce aggregate coverage metrics:

- Total requirements count
- Requirements covered (count and percentage)
- Requirements partially covered (count and percentage)
- Requirements with gaps (count and percentage)
- Requirements with stale content (count and percentage)
- Requirements routed to specialists (count by type)
- Estimated drafting effort for gaps (based on gap count and complexity)

### Step 7: Present Coverage Analysis

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Coverage summary** — total requirements, coverage percentage, gap count
- **Coverage by category** — Technical, Commercial, Legal, Compliance breakdown
- **Gap report** — list of uncovered requirements with priority and complexity
- **Stale content flags** — list of matched content older than 6 months
- **Specialist routing** — requirements that must go to pricing, legal, compliance, or security review regardless of content availability
- **Prior proposal reuse** — content identified from prior winning proposals
- **Estimated effort** — approximate drafting effort for gaps
- **Draft label** — "COVERAGE ANALYSIS — proposal manager review required before tracker update"

### Step 8: Update Requirements Matrix (After Confirmation)

After the proposal manager confirms the coverage analysis:
- Update the requirements matrix Excel worksheet with Coverage Status, Matched Content Asset, Content Last Updated, and Gap Notes columns
- Update the proposal tracker Status to "Coverage Mapped"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find the requirements matrix, proposal tracker, approved content library, prior proposals |
| SearchM365 (connectors) | Search content management platform via Graph Connector for approved assets |
| ReadFileContent | Read requirements matrix, content assets, and reference documents |
| GetDriveChildren | Browse the approved content library folder structure by category |
| render_ui (Adaptive Card) | Present the coverage analysis and gap report |

## Guardrails

- **Only match against content explicitly tagged as approved** — never present draft, expired, or under-review assets as available responses without clearly flagging their status
- **Flag any matched content older than 6 months** for freshness review — stale content in a proposal is a quality and accuracy risk
- **Never mark a pricing or legal requirement as "Covered"** — these always route to specialist review regardless of existing content
- **Present the gap analysis for review before updating the tracker** — coverage judgment involves interpretation; the proposal manager confirms before the matrix is updated
- **Clearly distinguish between "no content found" and "content exists but does not fully address the requirement"** — partial coverage and gaps require different response strategies
- **Never fabricate content matches** — if no relevant content is found, report the gap honestly rather than stretching a tangential asset to fit
- **Do not assess content quality** — flag content age and approval status, but whether the content is "good enough" for this proposal is the proposal manager's judgment
- **Include the Proposal ID in every output** for traceability
- **Preserve the original requirement text** when showing coverage mappings — the proposal manager needs to see the requirement alongside the matched content to verify fit
