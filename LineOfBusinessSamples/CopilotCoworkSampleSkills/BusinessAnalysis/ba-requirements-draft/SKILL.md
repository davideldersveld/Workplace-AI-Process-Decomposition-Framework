---
name: ba-requirements-draft
description: |
  Drafts business requirements documents, user stories, and acceptance criteria from
  synthesized requirement themes.
  Use when user asks to "draft requirements for [request]", "generate BRD",
  "write user stories", "draft acceptance criteria", "prepare requirements package",
  "create a requirements document", "write business requirements",
  "draft the BRD for [project]", or "generate requirements from themes".
  Do NOT use for extracting signals (use ba-signal-extraction),
  synthesizing themes (use ba-theme-synthesis),
  building the traceability matrix (use ba-traceability-packet),
  or routing for review (use ba-review-routing).
---

## Overview

Generates a structured requirements package — business requirements document (BRD), user stories with acceptance criteria, and requirements matrix — from synthesized themes. Uses approved templates from SharePoint where available and follows enterprise glossary conventions.

This skill operates in "AI draft plus approve" mode — every output is created as a draft for analyst review. Nothing is finalized without explicit confirmation.

## When to Use

- Synthesized themes are ready and the user wants to draft the requirements package
- The user needs a BRD, user stories, acceptance criteria, or requirements matrix
- Converting theme analysis into formal requirements deliverables

## When NOT to Use

- Extracting signals from documents — use ba-signal-extraction
- Synthesizing themes from signals — use ba-theme-synthesis
- Building the traceability matrix — use ba-traceability-packet
- Routing for review — use ba-review-routing
- Assembling discovery context — use ba-discovery-packet

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read themes and locate templates", activeForm="Reading themes and templates")
TaskCreate(subject="Draft requirements package", activeForm="Drafting requirements")
TaskCreate(subject="Generate requirements matrix", activeForm="Generating requirements matrix")
```

### Step 1: Read Inputs

Locate and read required inputs:

- **Synthesized themes** — the theme report from ba-theme-synthesis. Check `input/` first, then search: `SearchM365(sources=["files"], query="theme synthesis [case]")`
- **Requirements templates** — search for BRD template, user story format, acceptance criteria template: `SearchM365(sources=["files"], query="BRD template")`, `SearchM365(sources=["files"], query="requirements template")`
- **Domain glossary** — if available: `SearchM365(sources=["files"], query="[domain] glossary")`
- **Prior approved packages** — for style reference: `SearchM365(sources=["files"], query="approved requirements [domain]")`
- **Extracted signal set** — for traceability: `SearchM365(sources=["files"], query="signal inventory [case]")`

### Step 2: Draft the Requirements Package

Produce the deliverables the user requests. The default package includes all three; the user may request specific outputs.

#### Option A: Business Requirements Document (BRD)

Produce a Word document (invoke the `docx` skill) following the approved template structure. Standard sections:

1. **Executive Summary** — Business problem, objective, and expected outcome
2. **Business Objectives** — Measurable goals this requirements package supports
3. **Scope** — In-scope and out-of-scope items
4. **Functional Requirements** — Organized by theme, each with Req ID, statement, priority, and source reference
5. **Non-Functional Requirements** — Performance, security, compliance, usability
6. **Business Rules** — Rules that constrain or govern behavior
7. **Assumptions** — Stated assumptions from the theme analysis
8. **Constraints** — Technical, regulatory, timeline, budget constraints
9. **Dependencies** — Cross-system and cross-team dependencies
10. **Open Questions** — Items requiring stakeholder clarification before finalization

#### Option B: User Stories with Acceptance Criteria

Format each requirement as:

> **As a** [role], **I want** [capability], **so that** [benefit]
>
> **Acceptance Criteria:**
> - Given [context], when [action], then [expected result]
> - Given [context], when [action], then [expected result]

Group stories by theme. Include a Story ID and source signal reference for each.

#### Option C: Requirements Matrix

Produce an Excel workbook (invoke the `xlsx` skill) with:

| Column | Description |
|--------|-------------|
| Req ID | Auto-generated (REQ-NNN) |
| Theme | From theme synthesis |
| Requirement Statement | Clear, testable requirement |
| Type | Functional / Non-Functional / Business Rule |
| Priority | High / Medium / Low (from priority signals) |
| Source Signal ID | Link to the signal inventory |
| Status | Draft |
| Owner | Business owner or "TBD" |

### Step 3: Quality Checks

Before presenting the draft:

- Verify every requirement traces to at least one signal or is marked "Analyst judgment"
- Check terminology against the domain glossary — flag new terms not in the glossary
- Verify the template structure matches the approved template (if found)
- Include the Open Questions section with any unresolved items from theme synthesis
- Flag requirements generated from single-source or low-confidence signals

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find theme report, templates, glossary, prior packages, signal inventory |
| ReadFileContent | Read themes, templates, and reference documents |

## Guardrails

- **Always create as draft** — never finalize without explicit analyst confirmation
- **Trace every requirement** — each requirement must link to at least one extracted signal or be explicitly marked "Analyst judgment — requires confirmation"
- **Use approved templates** — if a template exists in SharePoint, follow its structure; do not invent new sections
- **Flag low-confidence requirements** — requirements from single-source or low-confidence signals get a visible flag
- **No scope additions** — do not add requirements not present in the synthesized themes without explicit analyst direction
- **Match the glossary** — use domain terminology; flag any new terms introduced
- **Include Open Questions** — never suppress unresolved items; they must be visible in the draft
