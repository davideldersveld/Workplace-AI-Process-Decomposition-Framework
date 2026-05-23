---
name: sales-proposal-draft
description: |
  Drafts a proposal response document or presentation deck using
  approved content, the requirements matrix, and context packet.
  Use when user asks to "draft the proposal for [account]",
  "build proposal deck for [opportunity]",
  "assemble response document for [RFP]",
  "create RFP response for [proposal ID]",
  "write the proposal for [account]",
  or "generate proposal package for [deal]".
  Do NOT use for creating a new proposal case (use sales-proposal-intake),
  gathering account context (use sales-context-packet),
  extracting RFP requirements (use sales-rfp-extraction),
  mapping requirements to approved content (use sales-content-mapping),
  or routing for approval (use sales-approval-routing).
---

## Overview

Produces a draft proposal response using approved content assets, the requirements matrix with coverage mappings, the context packet, and the organization's proposal templates. Supports two output formats: a PowerPoint executive proposal deck and a Word detailed RFP response document. Enforces strict controls on pricing placeholders, legal language placeholders, source citations, and approved-content-only usage.

This skill operates in "AI draft plus approve" mode — every proposal artifact is generated as a draft for proposal manager review. No proposal content is finalized or distributed without explicit confirmation.

## When to Use

- Requirements have been extracted and content has been mapped, and the team is ready to assemble the draft proposal
- The proposal manager has requested a PowerPoint deck for an executive presentation
- The proposal manager has requested a detailed Word response document for a formal RFP submission
- A revision to an existing draft is needed after reviewer feedback

## When NOT to Use

- Creating a new proposal case record — use sales-proposal-intake
- Gathering account history and content context — use sales-context-packet
- Extracting requirements from the RFP document — use sales-rfp-extraction
- Mapping requirements to approved content — use sales-content-mapping
- Routing the proposal package for approval — use sales-approval-routing
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read proposal inputs and templates", activeForm="Preparing proposal draft")
TaskCreate(subject="Draft proposal response", activeForm="Drafting proposal")
TaskCreate(subject="Present draft for review", activeForm="Reviewing proposal draft")
```

### Step 1: Read Draft Inputs

**Read all proposal artifacts:**
- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent` — Proposal ID, Account Name, Due Date, Complexity Rating, Status
- `SearchM365(sources=["files"], query="[proposal ID] requirements")` then `ReadFileContent` — requirements matrix with coverage mappings
- `SearchM365(sources=["files"], query="context packet [proposal ID]")` then `ReadFileContent` — account context, product fit, competitive intelligence, prior proposals
- `SearchM365(sources=["files"], query="[proposal ID] content mapping")` then `ReadFileContent` — coverage analysis with matched content assets

**Read proposal templates:**
- `SearchM365(sources=["files"], query="proposal template")` then `ReadFileContent` — standard proposal document template
- `SearchM365(sources=["files"], query="proposal deck template")` then `ReadFileContent` — standard PowerPoint presentation template

**Read matched content assets:**
- For each content asset identified in the coverage mapping, `ReadFileContent` to retrieve the approved content blocks

### Step 2: Determine Output Format

Based on the user's request and the RFP submission requirements:

| Format | When to Use |
|--------|------------|
| **PowerPoint deck** | Executive proposal presentation, initial pitch, visual-heavy requirements, or user requests "deck" or "slides" |
| **Word document** | Detailed RFP response, section-by-section answers, formal submission requirements, or user requests "document" or "response" |
| **Both** | Complex proposals requiring both an executive summary deck and a detailed response document |

### Step 3: Draft PowerPoint Proposal Deck

If a PowerPoint deck is requested, invoke the `pptx` skill to produce a presentation with:

**Slide 1 — Title**
- Proposal title, account name, date
- "DRAFT — FOR INTERNAL REVIEW ONLY" watermark

**Slide 2 — Executive Summary**
- Brief overview of the opportunity and proposed solution
- Key value proposition

**Slide 3 — Understanding Your Needs**
- Summary of customer requirements (from the requirements matrix)
- Demonstration of understanding of the customer's challenges

**Slide 4–6 — Proposed Solution**
- Product and solution overview aligned to requirements
- Architecture or approach diagram placeholder
- Key capabilities mapped to customer needs

**Slide 7 — Differentiators**
- Competitive differentiators (from context packet competitive intelligence)
- Why this solution is the best fit

**Slide 8 — Customer Evidence**
- Relevant case studies and references (from approved content library)
- Quantified outcomes where available

**Slide 9 — Implementation Approach**
- High-level implementation timeline and methodology
- Key milestones and deliverables

**Slide 10 — Pricing Summary**
- "PRICING — REQUIRES DEAL DESK APPROVAL" placeholder
- Pricing structure outline (without specific numbers)

**Slide 11 — Next Steps**
- Proposed next steps and timeline
- Contact information

### Step 4: Draft Word RFP Response Document

If a Word document is requested, invoke the `docx` skill to produce a response document with:

**Document Header**
- "DRAFT — FOR INTERNAL REVIEW ONLY"
- Proposal ID, Account Name, RFP Reference, Date

**Section-by-Section Responses**
For each requirement in the requirements matrix:
- **Requirement reference** — Requirement ID and original RFP section
- **Response** — Drafted using the matched approved content asset, tailored to this specific customer and requirement
- **Source citation** — Reference to the approved content asset used
- **Coverage note** — "Approved content" for covered requirements, "Adapted from [asset]" for partial coverage, "NEW CONTENT — REQUIRES REVIEW" for gaps

**Pricing Section**
- "PRICING — REQUIRES DEAL DESK APPROVAL"
- Pricing structure placeholder with instructions for deal desk

**Legal and Terms Section**
- "LEGAL — REQUIRES LEGAL REVIEW"
- Standard terms placeholder with instructions for legal review

**Appendices**
- Company overview and qualifications
- Relevant certifications and compliance documentation references
- Team bios and qualifications (if required by RFP)

### Step 5: Content Integrity Checks

Before presenting the draft, verify:

| Check | Criteria |
|-------|---------|
| **Source traceability** | Every content block traces to an approved content asset or is clearly marked as new content |
| **Pricing placeholders** | All pricing sections contain "REQUIRES DEAL DESK APPROVAL" — no auto-populated pricing |
| **Legal placeholders** | All legal and contractual sections contain "REQUIRES LEGAL REVIEW" — no drafted legal terms |
| **No fabricated claims** | No product capabilities, customer references, or performance claims not found in approved content |
| **Template compliance** | Document follows the organization's proposal template formatting standards |
| **Requirement coverage** | All mandatory requirements have a response (even if placeholder) |

### Step 6: Present Draft for Review

Present a summary via Adaptive Card (invoke `render-ui` skill first):

- **Proposal header** — Proposal ID, Account, Due Date, Format (Deck/Document/Both)
- **Draft completeness** — sections completed, sections with placeholders, sections needing specialist input
- **Content source summary** — percentage from approved content, percentage adapted, percentage new
- **Pricing and legal status** — confirmation that placeholders are in place
- **Flagged items** — requirements with gap responses, stale content used, or low-confidence matches
- **SLA status** — time remaining on the 3 business day first draft SLA
- **Draft label** — "PROPOSAL DRAFT — proposal manager review required before distribution"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find proposal tracker, requirements matrix, context packet, content mappings, templates, approved content assets, prior proposals |
| ReadFileContent | Read all proposal artifacts, templates, and content assets |
| GetDriveChildren | Browse proposal workspace and content library for assets |

## Guardrails

- **Always create as draft** — never finalize or distribute without explicit proposal manager confirmation
- **Every claim must trace to an approved content asset** — no fabricated product capabilities, customer references, or unsupported commitments
- **Pricing sections must contain placeholder markers** ("PRICING — REQUIRES DEAL DESK APPROVAL") — never populate pricing from historical proposals or internal pricing models
- **Legal and contractual language sections must contain placeholder markers** ("LEGAL — REQUIRES LEGAL REVIEW") — never draft legal terms, indemnification language, or liability provisions
- **Include source citations for every content block** used — the proposal manager needs to verify that content is current and appropriate
- **Match proposal template formatting standards** — use the organization's approved template, not generic formatting
- **Never include internal competitive intelligence in the proposal** — differentiation language is permissible; competitive battle card content is not
- **Never include internal pricing models, discount structures, or margin data** in the proposal — the deal desk controls all pricing content
- **Flag new content prominently** — sections drafted without approved content backing must be clearly marked for review
- **Include the Proposal ID and "DRAFT" label** on every page or slide
- **Never auto-send or auto-distribute** the draft — all proposal artifacts require explicit confirmation before leaving the proposal workspace
