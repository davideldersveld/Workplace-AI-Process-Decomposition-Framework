---
name: mktg-signal-extraction
description: |
  Extracts campaign objectives, audience definitions, deliverable requirements,
  timing constraints, budget parameters, dependencies, and blockers from
  request artifacts.
  Use when user asks to "extract campaign requirements",
  "what are the goals for [campaign]",
  "parse campaign request signals",
  "identify campaign dependencies for [name]",
  "what does this campaign request need",
  "pull requirements from [campaign] request",
  or "extract constraints for campaign [ID]".
  Do NOT use for creating a new campaign case (use mktg-campaign-intake),
  assembling brand and product context (use mktg-context-packet),
  drafting the campaign brief (use mktg-brief-draft),
  or routing for review (use mktg-review-routing).
---

## Overview

Parses campaign request artifacts — emails, attached briefs, intake forms, Teams discussions, and stakeholder clarifications — to extract structured signals: campaign objectives, KPIs, target audience definitions, required deliverables, timing and milestone constraints, budget parameters, channel requirements, dependencies on other teams, known blockers, and stakeholder expectations. Presents findings for campaign manager review via Adaptive Card and records confirmed signals in the signal matrix.

This skill operates in "AI assist" mode — it reads and analyzes request artifacts but only presents extracted signals as recommendations. The campaign manager reviews and confirms findings before they are recorded.

## When to Use

- A campaign request has been logged and needs structured requirement extraction
- A campaign manager wants to identify all goals, constraints, and dependencies before drafting
- New stakeholder input has been received and needs to be incorporated into the signal matrix
- A campaign scope change needs to be captured and flagged

## When NOT to Use

- Creating a new campaign case — use mktg-campaign-intake
- Assembling brand, product, and audience context — use mktg-context-packet
- Drafting the campaign brief — use mktg-brief-draft
- Routing for brand, product, or legal review — use mktg-review-routing
- Confirming brief baseline — this is always a human decision (MK-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read request artifacts and extract signals", activeForm="Reading campaign request artifacts")
TaskCreate(subject="Present extracted signals for review", activeForm="Extracting campaign signals")
```

### Step 1: Read Request Artifacts

Gather all available request materials:

- **Request emails** — `SearchM365(sources=["email"], query="[campaign name] campaign request")` — original request, stakeholder clarifications, scope changes
- **Teams discussions** — `SearchM365(sources=["teams"], query="[campaign name] campaign")` — coordination threads, stakeholder comments
- **Attached documents** — `SearchM365(sources=["files"], query="[campaign name] brief")` then `ReadFileContent` — draft briefs, request forms, stakeholder decks
- **Campaign workspace** — `GetDriveChildren` — browse the campaign folder for all request artifacts
- **Campaign tracker** — `SearchM365(sources=["files"], query="campaign tracker")` then `ReadFileContent` — current case data

### Step 2: Extract Signal Categories

Parse request artifacts to extract signals in each category:

| Signal Category | What to Extract | Examples |
|----------------|-----------------|----------|
| **Objectives** | Campaign goals, business outcomes, KPIs | "Drive 500 MQLs", "Increase brand awareness by 20%", "Support Q3 product launch" |
| **Target audience** | Who the campaign targets | "Enterprise IT decision-makers", "Mid-market CFOs", "Existing customers in healthcare" |
| **Deliverables** | Required assets and content types | "Landing page, 3 emails, social kit, event collateral, sales enablement deck" |
| **Timing** | Dates, deadlines, milestones | "Launch by June 15", "Asset delivery 2 weeks before event", "Email sequence starts July 1" |
| **Budget** | Budget range, spend constraints | "$50K total budget", "Use existing agency retainer", "No paid media budget" |
| **Channels** | Distribution and promotion channels | "Email, LinkedIn, webinar, partner co-marketing" |
| **Dependencies** | Requirements on other teams or campaigns | "Needs product marketing approval on claims", "Depends on website redesign completion" |
| **Blockers** | Known obstacles or risks | "Legal review pending on testimonial", "No approved photography for new product" |
| **Stakeholder expectations** | Specific requests or constraints from stakeholders | "CEO wants to review before launch", "Sales team needs enablement materials by June 1" |

### Step 3: Assess Signal Status

For each extracted signal, assess its status:

| Status | Definition |
|--------|------------|
| **Confirmed** | Explicitly stated in a verifiable source (email, document, form) |
| **Unconfirmed** | Implied or partially stated; needs stakeholder validation |
| **Conflicting** | Contradicted by another signal from a different source |

### Step 4: Identify Conflicts and Scope Risks

Flag signals that indicate potential problems:

- **Timeline vs. deliverables conflict** — aggressive launch date with extensive deliverable list
- **Budget vs. scope conflict** — narrow budget with broad channel requirements
- **Audience vs. messaging gap** — target audience not covered by existing approved messaging
- **Dependency risk** — critical dependency on a team or asset with no confirmed timeline
- **Scope creep indicators** — new deliverables or audiences added in later communications that were not in the original request
- **Missing critical signals** — no clear objective, no defined audience, or no launch date

### Step 5: Present Extraction Summary

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Campaign header** — Campaign ID, Name, Type, Requesting Stakeholder
- **Signal summary** — count by category (objectives, audience, deliverables, timing, budget, channels, dependencies, blockers)
- **Confirmed signals** — signals with clear source attribution
- **Unconfirmed signals** — signals needing stakeholder validation
- **Conflicting signals** — signals that contradict each other with source references for both
- **Scope risk flags** — timeline vs. deliverables, budget vs. scope, missing critical signals
- **Scope creep indicators** — new requirements that appeared after the original request
- **Draft label** — "SIGNAL EXTRACTION — campaign manager review required before recording"

### Step 6: Record Confirmed Signals (After Confirmation)

After campaign manager review, record confirmed signals to the Excel signal matrix worksheet.

Columns: Signal ID, Campaign ID, Category (Objective / Constraint / Dependency / Risk / Deliverable / Audience / Channel), Signal Text, Source (email / Teams / document / verbal), Status (Confirmed / Unconfirmed / Conflicting), Assigned To, Notes

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (email) | Find request emails, stakeholder clarifications, scope changes |
| SearchM365 (teams) | Find campaign coordination threads and stakeholder comments |
| SearchM365 (files) | Find attached briefs, request forms, campaign tracker |
| ReadFileContent | Read request documents, attached briefs, intake forms, tracker |
| GetDriveChildren | Browse campaign workspace for all request artifacts |

## Guardrails

- **Present extracted signals for campaign manager review** before writing to the signal matrix — interpretation of stakeholder intent is judgment-dependent
- **Flag conflicting signals prominently** with source references for both sides — never auto-resolve conflicts; present for campaign manager decision
- **Preserve original stakeholder language** alongside any summarized version — the original wording is evidence; the summary is interpretation
- **Flag scope creep indicators** when new requirements appear in later communications that expand beyond the original request — the campaign manager needs visibility into scope changes
- **Flag missing critical signals** — if no clear objective, audience, or launch date has been identified, this is a significant gap that must be addressed before brief drafting
- **Never generate campaign objectives, audience definitions, or deliverable lists** from assumptions — extract only what is present in the artifacts; flag gaps for stakeholder input
- **Never assess the quality or feasibility of stated objectives** — signal extraction identifies what was requested; feasibility assessment belongs to the campaign manager
- **Never modify the campaign tracker** — this skill presents analysis only; tracker updates happen after campaign manager confirmation
- **Tag the source** for every extracted signal — email, Teams, document reference, or verbal — for audit trail
- **Never include internal stakeholder politics or interpersonal context** in the signal matrix — capture requirements and constraints only
