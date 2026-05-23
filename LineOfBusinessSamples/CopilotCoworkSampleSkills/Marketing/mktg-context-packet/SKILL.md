---
name: mktg-context-packet
description: |
  Assembles brand guidelines, product messaging, audience context,
  prior campaign history, and channel recommendations into a structured
  context packet for campaign brief creation.
  Use when user asks to "build campaign context for [campaign]",
  "assemble brief context", "what brand assets apply to [campaign]",
  "pull product messaging for [product]",
  "prepare context for campaign [ID]",
  "get brand guidelines for [campaign type]",
  or "campaign background materials for [name]".
  Do NOT use for creating a new campaign case (use mktg-campaign-intake),
  extracting goals and constraints (use mktg-signal-extraction),
  drafting the campaign brief (use mktg-brief-draft),
  or routing for review (use mktg-review-routing).
---

## Overview

Assembles a comprehensive context packet for an active campaign case — pulling applicable brand guidelines, approved product messaging, target audience profiles, prior campaign history and performance insights, channel recommendations, and open dependencies. The packet is generated as a Word document saved to the SharePoint campaign workspace folder.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (brand repositories, messaging frameworks, audience data, campaign history) without exercising judgment on campaign strategy or messaging choices.

## When to Use

- A campaign case has been created and needs context assembled before brief drafting
- A campaign manager needs the applicable brand guidelines and messaging for a product area
- Marketing operations needs to pull audience data and prior campaign performance for planning
- A context packet needs updating after new brand assets or audience research become available

## When NOT to Use

- Creating a new campaign case — use mktg-campaign-intake
- Extracting goals, constraints, and dependencies from request artifacts — use mktg-signal-extraction
- Drafting the campaign brief — use mktg-brief-draft
- Routing for brand, product, or legal review — use mktg-review-routing
- Confirming brief baseline — this is always a human decision (MK-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read campaign data and gather source materials", activeForm="Gathering campaign context")
TaskCreate(subject="Assemble context packet document", activeForm="Building context packet")
```

### Step 1: Read Campaign Data

Locate and read the campaign case:
- `SearchM365(sources=["files"], query="campaign tracker")` then `ReadFileContent` — find the case and read current status
- Identify: Campaign ID, Campaign Name, Campaign Type, Product or Solution Area, Target Audience, Requested Launch Date

### Step 2: Retrieve Brand Guidelines

Find the applicable brand assets:
- `SearchM365(sources=["files"], query="brand guidelines")` then `ReadFileContent` — master brand book, visual identity standards
- `SearchM365(sources=["files"], query="[campaign type] brand standards")` then `ReadFileContent` — campaign-type-specific guidelines (digital, print, event, etc.)
- `SearchM365(sources=["files"], query="messaging framework")` then `ReadFileContent` — corporate messaging architecture

Extract:
- Brand voice and tone guidelines
- Visual identity requirements (logo usage, color palette, typography)
- Campaign-type-specific design and messaging standards
- Brand do's and don'ts relevant to this campaign type

### Step 3: Retrieve Product Messaging

Find approved product and solution messaging:
- `SearchM365(sources=["files"], query="[product or solution] messaging")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[product or solution] approved claims")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[product or solution] value proposition")` then `ReadFileContent`

Extract:
- Product messaging pillars and key themes
- Approved claims and proof points
- Approved value propositions and differentiators
- Competitive positioning guidance (for internal use only — never in external-facing content)

### Step 4: Retrieve Audience Data

Find target audience profiles and segmentation:
- `SearchM365(sources=["files"], query="[audience segment] profile")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="audience segmentation [product or industry]")` then `ReadFileContent`
- `SearchM365(sources=["connectors"], connector_ids=["marketo-connector"])` — audience segment data and engagement history from marketing automation (if Graph Connector available)

Extract:
- Target audience demographics, firmographics, and psychographics
- Audience pain points and buying triggers
- Preferred channels and content consumption patterns
- Audience segment size and engagement benchmarks

### Step 5: Retrieve Prior Campaign History

Search for relevant prior campaigns:
- `SearchM365(sources=["files"], query="[product or solution] campaign brief")` — prior briefs
- `SearchM365(sources=["files"], query="[product or solution] campaign performance")` — performance reports
- `SearchM365(sources=["files"], query="[campaign type] campaign results")` — type-specific historical results

Compile:
- Prior campaigns for the same product or audience (name, date, type, channels)
- Performance highlights and lessons learned (if available)
- Messaging that performed well vs. messaging that underperformed
- Assets from prior campaigns available for reuse

### Step 6: Determine Channel Recommendations

Based on campaign type, audience, and prior performance:
- Map campaign type to typical channel mix (email, social, web, events, paid media, content syndication)
- Cross-reference with audience channel preferences
- Note channel-specific requirements (landing pages, email templates, social assets, event collateral)

### Step 7: Identify Open Dependencies and Questions

Flag:
- Missing brand assets for this campaign type
- Product messaging gaps (no approved claims for a feature or benefit area)
- Audience data staleness (segment data older than one quarter)
- Brand guidelines updated since the last campaign in this product area
- Competitive positioning materials under review or expired
- Dependencies on other teams (product marketing, design, web, events)

### Step 8: Assemble Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Campaign Summary**
   - Campaign ID, Name, Type, Product/Solution Area
   - Requesting stakeholder and department
   - Requested launch date and SLA target

2. **Brand Guidelines**
   - Applicable brand voice and tone
   - Visual identity requirements
   - Campaign-type-specific standards
   - Source document references

3. **Product Messaging**
   - Messaging pillars and key themes
   - Approved claims and proof points
   - Value propositions and differentiators
   - Source document references

4. **Target Audience Profile**
   - Demographics, firmographics, pain points
   - Preferred channels and content patterns
   - Engagement benchmarks
   - Data freshness indicator

5. **Prior Campaign History**
   - Relevant prior campaigns with performance highlights
   - Reusable assets identified
   - Messaging performance insights

6. **Channel Recommendations**
   - Recommended channel mix based on type, audience, and history
   - Channel-specific asset requirements

7. **Dependencies and Open Questions**
   - Missing materials, stale data, team dependencies
   - Questions requiring stakeholder input

Save the packet to the SharePoint campaign workspace folder.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find campaign tracker, brand guidelines, messaging documents, audience profiles, prior campaigns, templates |
| SearchM365 (connectors) | Retrieve audience data and campaign history from marketing automation (if available) |
| ReadFileContent | Read all source documents — tracker, brand guidelines, messaging, audience data, prior briefs |
| GetDriveChildren | Browse brand asset library, campaign workspace, and template repositories |
| GetUserDetails | Requesting stakeholder profile and department context |

## Guardrails

- **Only surface content from approved brand and messaging repositories** — never include draft, expired, under-review, or deprecated assets
- **Cite source document and version** for every messaging element, claim, and brand guideline referenced — traceability is required for review
- **Flag if brand guidelines have been updated** since the last campaign in this product area — the campaign manager needs to review changes
- **Flag audience data staleness** — segment data older than one quarter should be flagged prominently for marketing operations review
- **Do not include internal competitive positioning** in any section that could appear in external-facing content — competitive insights are for internal planning only
- **Do not include personally identifiable audience data** — use segment-level abstractions only (demographics, firmographics, behavioral segments)
- **Never generate campaign strategy, messaging, or creative direction** — this is factual context assembly only; strategy belongs to the campaign manager and brief draft step
- **Never fabricate claims, statistics, or performance data** — if prior campaign performance data is not available, state the gap rather than estimating
- **Restrict access** to the generated packet — save to the campaign workspace folder with appropriate SharePoint permissions
- **Include the campaign SLA target** in the packet header so the team can track triage timeliness
