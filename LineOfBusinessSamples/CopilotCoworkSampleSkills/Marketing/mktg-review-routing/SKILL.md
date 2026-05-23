---
name: mktg-review-routing
description: |
  Routes campaign briefs to the required brand, product marketing, legal,
  and leadership reviewers based on the review routing matrix.
  Use when user asks to "route brief for review",
  "send for brand review", "request legal review of campaign",
  "submit brief to reviewers", "who needs to review [campaign]",
  "route campaign [ID] for approval",
  or "set up review chain for [campaign]".
  Do NOT use for creating a new campaign case (use mktg-campaign-intake),
  assembling brand and product context (use mktg-context-packet),
  extracting goals and constraints (use mktg-signal-extraction),
  or drafting the campaign brief (use mktg-brief-draft).
---

## Overview

Routes campaign briefs to the required reviewers based on the review routing matrix, campaign characteristics (type, audience, claims, channels, budget), and organizational escalation paths. Resolves reviewer identities, sends Teams notifications and Outlook review requests after confirmation, creates review deadline calendar holds, and updates the campaign tracker with review status.

This skill operates in "AI act within policy" mode — it executes routing within pre-approved review matrix rules. The campaign manager confirms the routing recommendation before notifications are sent.

## When to Use

- A campaign brief draft is complete and needs to be routed for review
- A campaign manager needs to determine which reviewers are required
- A revised brief needs re-routing after changes to scope, claims, or audience
- A review status update is needed for the campaign tracker

## When NOT to Use

- Creating a new campaign case — use mktg-campaign-intake
- Assembling brand, product, and audience context — use mktg-context-packet
- Extracting goals, constraints, and dependencies — use mktg-signal-extraction
- Drafting the campaign brief — use mktg-brief-draft
- Confirming brief baseline — this is always a human decision (MK-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read campaign data and review routing matrix", activeForm="Reading routing inputs")
TaskCreate(subject="Route brief and send review notifications", activeForm="Routing campaign brief")
```

### Step 1: Read Routing Inputs

**Read the campaign case data:**
- `SearchM365(sources=["files"], query="campaign tracker")` then `ReadFileContent` — campaign type, audience, priority, specialized review flags

**Read the draft brief:**
- `SearchM365(sources=["files"], query="campaign brief [Campaign ID or name]")` then `ReadFileContent` — identify claims, channels, and content requiring review

**Read the review routing matrix:**
- `SearchM365(sources=["files"], query="review routing matrix")` then `ReadFileContent` — reviewer assignment rules by campaign type, audience, claims, channels, and budget

**Resolve reviewers:**
- `SearchPeople` — resolve reviewer identities by role and specialty
- `GetUserDetails` — verify reviewer availability and role
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure for escalation paths

### Step 2: Determine Required Reviewers

Match the campaign to required reviewers based on the routing matrix:

| Reviewer | Required When | Review Scope |
|----------|--------------|-------------|
| **Brand reviewer** | All campaigns | Brand voice, visual identity, messaging consistency |
| **Product marketing reviewer** | Campaigns with product claims or value propositions | Claim accuracy, messaging alignment, competitive positioning |
| **Legal reviewer** | Regulated industry claims, testimonials, sweepstakes, comparative claims | Legal compliance, risk, regulatory requirements |
| **Communications reviewer** | External-facing campaigns, press-adjacent content | External messaging, PR alignment, corporate narrative |
| **Marketing leadership** | Campaigns above budget threshold, executive-sponsored campaigns | Strategic alignment, resource allocation, priority |
| **Regional marketing reviewer** | Cross-regional or international campaigns | Regional messaging adaptation, local compliance |
| **Privacy reviewer** | Campaigns using customer data, personalization, or targeting | Data usage compliance, consent requirements |

### Step 3: Determine Review Timeline

Based on campaign priority and launch date:

| Priority | Standard Review | Expedited Review |
|----------|----------------|-----------------|
| **High** | 2 business days | 1 business day |
| **Standard** | 3 business days | 2 business days |
| **Low** | 5 business days | 3 business days |

If the review timeline would extend past the requested launch date, flag this as a timeline risk.

### Step 4: Present Routing Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Campaign header** — Campaign ID, Name, Type, Priority, Requested Launch Date
- **Required reviewers** — each reviewer with role, review scope, and routing rationale
- **Review timeline** — target completion date per reviewer based on priority
- **Timeline risk** — flag if review completion would exceed requested launch date
- **Review sequence** — any sequencing requirements (e.g., legal review after product marketing confirms claims)
- **SLA status** — time elapsed since intake, triage SLA remaining
- **Draft label** — "ROUTING RECOMMENDATION — campaign manager confirmation required"

### Step 5: Execute Routing (After Confirmation)

**Send Teams notifications to each reviewer:**
- `PostMessage` — direct message to each reviewer with:
  - Campaign ID and name
  - Review scope (what they are reviewing)
  - Review deadline
  - Link to the campaign brief in the SharePoint workspace
  - Link to the campaign workspace folder for supporting materials

**Send Outlook review request (for formal review chains):**
- `CreateDraftMessage` — formal review request email with the brief attached for reviewers who require email-based workflow

**Create review deadline calendar holds:**
- `CreateEvent` — calendar event for each reviewer with the review deadline as a visible reminder

**Update campaign tracker:**
- Record review status per reviewer (Pending, In Review, Approved, Revision Requested), reviewer names, routing timestamp, and review deadlines

### Step 6: Post-Review Status Tracking

When asked about review status:
- Read the campaign tracker for current review status per reviewer
- Identify overdue reviews (past deadline with no response)
- Flag reviews blocking the brief baseline
- Recommend follow-up actions for overdue reviews

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find campaign tracker, draft brief, review routing matrix |
| ReadFileContent | Read campaign data, brief content, routing rules |
| SearchPeople | Resolve reviewer identities by role and specialty |
| GetUserDetails | Verify reviewer availability and role |
| GetManagerDetails / GetDirectReportsDetails | Org structure for escalation paths |
| PostMessage | Teams notifications to reviewers (after confirmation) |
| CreateDraftMessage | Formal review request emails (after confirmation) |
| CreateEvent | Review deadline calendar holds |

## Guardrails

- **Present routing recommendations for campaign manager review** before sending any notifications — routing decisions are confirmed before execution
- **Route based strictly on the review routing matrix** — campaign type, audience, claims, channels, and budget determine which reviewers are required
- **Never skip a required reviewer** even if the campaign appears straightforward — the routing matrix defines minimum review requirements
- **Escalate to marketing operations** if the review matrix produces no clear routing for a campaign characteristic — do not guess at reviewer assignments
- **Include campaign type, audience, and key claims** in every review request so reviewers can prioritize effectively
- **Include the review deadline and scope** in every notification — reviewers need to know what they are reviewing and when it is due
- **Never include full brief content in Teams messages** — reference the SharePoint workspace and provide a link; full materials stay in SharePoint
- **Flag timeline risks prominently** — if the review timeline would extend past the requested launch date, the campaign manager needs to know immediately
- **Never modify the campaign brief** during routing — routing assigns review ownership; changes to the brief require re-running mktg-brief-draft
- **Log all routing decisions** to the campaign tracker with timestamp, reviewer assignments, deadlines, and confirming campaign manager for audit
- **Never bypass legal review** for campaigns flagged with regulated claims, testimonials, sweepstakes, or comparative claims — legal review is non-negotiable for these campaign types
- **Track review sequence requirements** — if legal review depends on product marketing confirming claims first, route product marketing first and note the dependency
