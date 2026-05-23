---
name: ba-theme-synthesis
description: |
  Clusters extracted requirements signals into coherent themes, detects conflicts,
  and identifies gaps in stakeholder input.
  Use when user asks to "synthesize requirements themes", "cluster these needs",
  "reconcile stakeholder input", "what are the main themes",
  "find conflicts in requirements", "group these signals", "identify requirement gaps",
  "analyze conflicts across stakeholders", or "theme analysis for [case]".
  Do NOT use for extracting signals from documents (use ba-signal-extraction),
  drafting the final requirements package (use ba-requirements-draft),
  or building the traceability matrix (use ba-traceability-packet).
---

## Overview

Takes extracted requirements signals and groups them into coherent themes, detects conflicts between stakeholders, identifies gaps where expected requirements are missing, and surfaces open questions. Produces a theme report that becomes the input for requirements drafting.

This skill operates in "AI assist" mode — synthesis results are presented for analyst review before further processing. Conflicts are never resolved automatically.

## When to Use

- An extracted signal set is available and needs to be organized into themes
- The user wants to identify conflicts, gaps, or patterns across stakeholder input
- Preparing the input for requirements drafting

## When NOT to Use

- Extracting signals from source documents — use ba-signal-extraction
- Drafting the requirements document — use ba-requirements-draft
- Building the traceability matrix — use ba-traceability-packet
- Routing for review — use ba-review-routing

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read signal inventory and reference materials", activeForm="Reading signals")
TaskCreate(subject="Cluster signals into requirement themes", activeForm="Clustering themes")
TaskCreate(subject="Detect conflicts and identify gaps", activeForm="Detecting conflicts")
TaskCreate(subject="Produce theme synthesis report", activeForm="Producing theme report")
```

### Step 1: Read Inputs

Locate and read the required inputs:

- **Signal inventory** — the Excel workbook or structured data from ba-signal-extraction. Check `input/` first for uploaded files, then search M365: `SearchM365(sources=["files"], query="signal inventory [case]")`
- **Requirements taxonomy** — if one exists in SharePoint: `SearchM365(sources=["files"], query="requirements taxonomy")`. This provides standard categories for grouping.
- **Discovery packet** — for additional context: `SearchM365(sources=["files"], query="discovery packet [case]")`
- **Prior requirements packages** — for pattern reference: `SearchM365(sources=["files"], query="requirements [domain] approved")`

### Step 2: Cluster Signals into Themes

Group related signals into coherent requirement themes:

1. **Identify natural groupings** — signals that address the same business need or capability area
2. **Apply taxonomy labels** — if a requirements taxonomy exists, use its categories; otherwise create descriptive theme labels
3. **Assign each signal to a theme** — a signal may belong to multiple themes if it spans concerns
4. **Label each theme** with:
   - Theme name (descriptive label)
   - Theme description (one sentence)
   - Supporting signals (list of Signal IDs)
   - Signal count
   - Source diversity (number of unique sources/stakeholders)

### Step 3: Detect Conflicts and Gaps

**Conflicts:** Identify signals within or across themes where stakeholders have stated contradictory needs. For each conflict:
- State both sides with source attribution
- Note which stakeholders are on each side
- Mark as "Requires analyst resolution" — do not pick a side

**Gaps:** Identify expected requirement areas with no stakeholder input. Compare the themes found against:
- The requirements taxonomy (if available)
- Common requirement categories for this type of request (functional, non-functional, business rules, data, integration, security, compliance)
- Prior requirements packages for similar work

**Open Questions:** Surface items that need stakeholder clarification before drafting.

**Priority Signals:** Note any relative importance indicators from the source material (urgency language, escalation context, executive sponsorship).

### Step 4: Produce the Synthesis Report

Present key findings via Adaptive Card (invoke `render-ui` skill first):
- Number of themes identified
- Number of conflicts detected
- Number of gaps identified
- Top conflicts requiring resolution

Then produce a Word document (invoke the `docx` skill) with:

1. **Theme Summary** — Table of themes with name, description, signal count, and source diversity
2. **Detailed Themes** — For each theme: description, supporting signals with attribution, and confidence notes
3. **Conflict Report** — Each conflict with both sides, source attribution, and resolution status ("Unresolved — requires analyst decision")
4. **Gap Identification** — Expected requirement areas with no coverage, labeled "Analyst should verify"
5. **Open Questions** — Items needing stakeholder clarification
6. **Priority Signals** — Relative importance indicators from source material

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find signal inventory, taxonomy, discovery packet, prior packages |
| ReadFileContent | Read signal inventory and reference documents |

## Guardrails

- **Present for review** — this is AI assist mode; never proceed to drafting without analyst confirmation of themes
- **Preserve attribution** — every theme must trace to one or more extracted signals with source references
- **Never resolve conflicts** — present both sides with attribution; the analyst decides
- **Flag single-source themes** — themes supported by only one source may be incomplete
- **Label gaps as advisory** — gap identification is "analyst should verify", not an assertion of missing requirements
- **Do not add scope** — do not introduce themes or requirements not supported by the signal inventory
