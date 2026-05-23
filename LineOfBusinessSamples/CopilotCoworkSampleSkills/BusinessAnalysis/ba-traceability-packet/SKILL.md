---
name: ba-traceability-packet
description: |
  Builds the traceability matrix and review summary package that links requirements
  to source signals and prepares the package for stakeholder review.
  Use when user asks to "prepare review packet", "build traceability matrix",
  "link requirements to sources", "review readiness check",
  "prepare for sign-off", "traceability for [case]", "create review summary",
  "package requirements for review", or "build the review deck".
  Do NOT use for drafting requirements (use ba-requirements-draft),
  extracting signals (use ba-signal-extraction),
  or routing the package to reviewers (use ba-review-routing).
---

## Overview

Creates the complete review and traceability package: a traceability matrix linking each requirement to its source signals, a review summary with completeness assessment, and an executive review presentation. This package is what reviewers receive before sign-off.

This skill operates in "AI draft plus approve" mode — the traceability packet is presented for analyst review before routing to stakeholders.

## When to Use

- A draft requirements package is complete and needs to be prepared for review
- The user wants to verify that requirements trace back to source evidence
- Preparing the materials that reviewers will use for sign-off

## When NOT to Use

- Drafting the requirements themselves — use ba-requirements-draft
- Extracting signals from documents — use ba-signal-extraction
- Synthesizing themes — use ba-theme-synthesis
- Routing the package to reviewers — use ba-review-routing

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read requirements package and signal inventory", activeForm="Reading inputs")
TaskCreate(subject="Build traceability matrix", activeForm="Building traceability matrix")
TaskCreate(subject="Assess review readiness", activeForm="Assessing readiness")
TaskCreate(subject="Create review summary and deck", activeForm="Creating review materials")
```

### Step 1: Read Inputs

Locate and read required inputs:

- **Draft requirements package** — from ba-requirements-draft. Check `input/`, then: `SearchM365(sources=["files"], query="requirements [case]")`
- **Signal inventory** — from ba-signal-extraction: `SearchM365(sources=["files"], query="signal inventory [case]")`
- **Stakeholder roster** — from the discovery packet or tracker: `SearchM365(sources=["files"], query="stakeholder roster [case]")`
- **Review checklist** — if one exists: `SearchM365(sources=["files"], query="requirements review checklist")`
- **Traceability standard** — if defined: `SearchM365(sources=["files"], query="traceability standard")`

Verify all supporting documents exist using `GetDriveChildren` if a case folder is known.

### Step 2: Build the Traceability Matrix

Produce an Excel workbook (invoke the `xlsx` skill) with:

| Column | Description |
|--------|-------------|
| Req ID | From requirements package |
| Requirement Statement | The requirement text |
| Source Signal ID | Link to signal inventory |
| Source Document | Name of the source document |
| Source Author | Who stated the original need |
| Source Date | When it was stated |
| Traceability Status | Linked / Analyst Judgment / Unlinked |

**Traceability rules:**
- **Linked** — requirement traces to one or more signals with clear source attribution
- **Analyst Judgment** — requirement was added by analyst direction, not from a source signal. Flag as "Requires explicit confirmation"
- **Unlinked** — requirement has no source support. Flag for review.

Calculate **traceability coverage**: (Linked requirements / Total requirements) x 100%

### Step 3: Assess Review Readiness

Check the package against the review checklist (if available) or standard criteria:

| Check | Status |
|-------|--------|
| All requirements have Req IDs | Pass / Fail |
| Traceability coverage meets minimum threshold | Pass / Fail (show percentage) |
| Open questions are documented | Pass / Fail |
| Conflicts are documented with resolution status | Pass / Fail |
| Stakeholder roster is complete | Pass / Fail |
| All reviewers are identified | Pass / Fail |
| Template structure is followed | Pass / Fail |

If traceability coverage is below threshold (typically 90%+), flag this — the package should not be routed for review until coverage is addressed.

### Step 4: Create Review Materials

Produce three outputs:

**1. Review Summary (Word)** — invoke `docx` skill:
- Package completeness assessment (from Step 3)
- Traceability coverage percentage
- Unresolved questions list
- Unresolved conflicts list
- Reviewer assignments (from stakeholder roster)
- Recommended review timeline

**2. Executive Review Deck (PowerPoint)** — invoke `pptx` skill:
- Slide 1: Request summary and business objective
- Slide 2: Key requirement themes (from theme synthesis)
- Slide 3: Requirement highlights — count by type, priority distribution
- Slide 4: Open items — unresolved questions, conflicts, and gaps
- Slide 5: Approval request — what reviewers are being asked to decide

**3. Summary Card** — invoke `render-ui` skill to show:
- Traceability coverage percentage
- Requirements count by type and priority
- Open items count
- Review readiness verdict (Ready / Needs Attention / Not Ready)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find requirements package, signal inventory, review checklist, stakeholder roster |
| ReadFileContent | Read all input documents |
| GetDriveChildren | Verify case folder completeness |
| SearchPeople | Resolve reviewer contacts from stakeholder roster |

## Guardrails

- **Present for review** — the full traceability packet must be reviewed by the analyst before routing
- **Flag unlinked requirements** — any requirement without source support must be visibly flagged as "Analyst judgment — requires explicit confirmation"
- **Calculate and display coverage** — always show the traceability coverage percentage
- **Include all open items** — never suppress unresolved questions or conflicts in the review summary
- **Verify reviewers** — every listed reviewer should match the stakeholder roster
- **Do not mark as review-ready** if traceability coverage is below the defined minimum threshold
