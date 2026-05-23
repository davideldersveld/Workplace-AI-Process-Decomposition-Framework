---
name: compliance-risk-detection
description: |
  Identifies risk indicators, evidence gaps, and elevated-risk patterns for a compliance case.
  Compares submitted evidence against required checklists and surfaces factual gaps.
  Use when user asks to "check risk indicators for", "what evidence is missing",
  "compliance gap check for [case ID]", "risk assessment for case",
  "evidence status for", "analyze compliance risk for",
  "what's missing from this case", or "risk flags for [case]".
  Do NOT use for assembling policy context (use compliance-context-packet),
  routing for review (use compliance-review-routing),
  drafting communications (use compliance-case-comms),
  or creating a new case (use compliance-case-intake).
---

## Overview

Compares required evidence against submitted materials, detects risk indicator patterns, and surfaces factual gaps. Presents all findings as indicators — never as conclusions, determinations, or risk judgments. Flags regulatory-reportable indicators with elevated visibility.

This skill operates in "AI assist" mode — it presents analysis for user review via Adaptive Card. It does not modify case status or disposition.

## When to Use

- A context packet has been assembled and the analyst wants to check for risk indicators
- The user wants to know what evidence is missing from a case
- Preparing risk analysis input before routing a case for review

## When NOT to Use

- Assembling policy and control context — use compliance-context-packet
- Routing a case for review — use compliance-review-routing
- Drafting case communications — use compliance-case-comms
- Creating a new case — use compliance-case-intake

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case context and evidence checklist", activeForm="Reading case materials")
TaskCreate(subject="Compare evidence against requirements", activeForm="Checking evidence gaps")
TaskCreate(subject="Detect risk indicator patterns", activeForm="Detecting risk indicators")
TaskCreate(subject="Present risk indicator report", activeForm="Preparing risk report")
```

### Step 1: Read Case Materials

Locate and read required inputs:

- **Context packet** — from compliance-context-packet. Check `input/` first, then: `SearchM365(sources=["files"], query="context packet [case ID]")`
- **Evidence checklist** — the Excel checklist from the context packet: `SearchM365(sources=["files"], query="evidence checklist [case ID]")`
- **Case evidence folder** — use `GetDriveChildren` to list what has been submitted to the case folder
- **Policy documents** — for cross-reference: `ReadFileContent` on the applicable policies identified in the context packet

### Step 2: Evidence Gap Analysis

Compare the evidence checklist against the case evidence folder contents:

For each required evidence item:
1. Check if a matching document exists in the case folder
2. Mark as: **Submitted** (found), **Missing** (not found), or **Partial** (found but incomplete)
3. Note the gap severity: **Critical** (required for case decision), **Important** (needed for thorough review), **Supplementary** (helpful but not blocking)

Present the evidence gap summary showing:
- Total required items vs. submitted
- Critical missing items (blocking)
- Important missing items (should be requested)

### Step 3: Risk Indicator Detection

Analyze the case materials for these risk indicator categories:

| Indicator Category | What to Look For | Severity Signal |
|-------------------|------------------|-----------------|
| **Missing mandatory evidence** | Required evidence items not submitted | Elevated if critical items missing |
| **Inconsistency flags** | Reported facts contradict submitted evidence | Elevated |
| **Repeat pattern** | Same policy area has prior exceptions or findings | Elevated if 3+ in 12 months |
| **Multi-entity or multi-jurisdiction** | Case involves multiple legal entities or jurisdictions | Elevated — may require coordinated review |
| **Regulatory reporting trigger** | Anti-corruption, sanctions, data breach, or financial misconduct indicators | Critical — requires separate escalation |
| **Control failure pattern** | Evidence suggests a systemic control weakness, not an isolated exception | Elevated — may require internal audit referral |
| **Approval bypass indicators** | Evidence of action taken before required approval obtained | Elevated |

For each indicator found:
- State the specific evidence gap or pattern that triggered it
- Cite the source document or data point
- Classify as: Low / Medium / High / Critical

### Step 4: Present Risk Indicator Report

Present findings via Adaptive Card (invoke `render-ui` skill first):

**Summary section:**
- Evidence completeness percentage
- Number of risk indicators by severity
- Regulatory-reportable flags (highlighted separately)

**Detail section:**
- Evidence gap table (item, status, severity)
- Risk indicator list (indicator, evidence, severity, source)

**Regulatory flags section (if any):**
- Separate, prominently displayed section for any regulatory-reportable indicators
- Clear callout: "Regulatory reporting may be required — consult compliance leadership"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find context packet, evidence checklist, prior findings |
| ReadFileContent | Read case documents, policy text, and evidence items |
| GetDriveChildren | List submitted evidence in the case folder |

## Guardrails

- **Indicators only, never conclusions** — present all findings as "indicators" or "flags"; never characterize findings as conclusions, determinations, fault assessments, or risk judgments
- **Cite every indicator** — each risk indicator must reference the specific evidence gap or pattern that triggered it
- **Never assess intent or culpability** — surface factual gaps and patterns only
- **Elevate regulatory flags** — anti-corruption, sanctions, data breach, and financial misconduct indicators must be presented with separate, prominent visibility
- **Read-only mode** — this skill does not modify case status, severity, or disposition; it only surfaces analysis for the analyst
- **Present before any action** — all findings must be reviewed by the user via Adaptive Card before any downstream action
