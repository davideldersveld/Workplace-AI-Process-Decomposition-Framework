---
name: cs-issue-classifier
description: |
  Classifies a service case by issue type and customer intent using the service taxonomy,
  and surfaces similar prior cases to support the classification rationale.
  Use when user asks to "classify this case", "what type of issue is this",
  "categorize case [ID]", "triage this ticket",
  "what's the issue category", "classify issue for [customer]",
  "issue type for case [ID]", or "triage classification for [case]".
  Do NOT use for assembling context (use cs-context-packet),
  assessing severity (use cs-severity-assessment),
  routing the case (use cs-case-routing),
  drafting a response (use cs-response-drafter),
  or creating a new case (use cs-case-intake).
---

## Overview

Compares the inbound issue description against the service taxonomy, cross-references with prior case patterns, and presents a classification recommendation with confidence score and supporting evidence. The classification includes issue category, detected customer intent, suggested queue, and similar prior cases.

This skill operates in "AI assist" mode — it presents classification as a recommendation for agent review via Adaptive Card. It does not write the classification to the tracker without user confirmation.

## When to Use

- A case has been created and context assembled, and needs to be classified before routing
- The user wants to determine what type of issue a case represents
- A case needs reclassification after new information has surfaced

## When NOT to Use

- Assembling customer and account context — use cs-context-packet
- Assessing severity and SLA path — use cs-severity-assessment
- Routing the case to an agent or queue — use cs-case-routing
- Drafting a customer response — use cs-response-drafter
- Creating a new case — use cs-case-intake

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and service taxonomy", activeForm="Reading classification inputs")
TaskCreate(subject="Classify issue and detect intent", activeForm="Classifying issue")
TaskCreate(subject="Present classification for review", activeForm="Preparing classification")
```

### Step 1: Read Classification Inputs

Locate and read required inputs:

- **Case data** — from the case tracker: `SearchM365(sources=["files"], query="service case tracker")`
- **Context packet** — from cs-context-packet output: `SearchM365(sources=["files"], query="context packet [Case ID]")`
- **Service taxonomy** — `SearchM365(sources=["files"], query="issue taxonomy")` or `SearchM365(sources=["files"], query="service classification guide")`
- **Inbound message content** — from the original email or intake: `GetMessage(message_id=...)` or `ReadFileContent`
- **Prior case patterns** — `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])` for prior case classifications, or read from the case tracker history

Read each document using `ReadFileContent`.

### Step 2: Classify Issue and Detect Intent

**Issue Classification:**
Compare the inbound issue description against the service taxonomy categories. For each potential category:
1. Score the match based on keywords, phrases, and context alignment
2. Check for supporting evidence in the context packet (product owned, prior cases in same category)
3. Identify the best-fit category and any secondary categories

**Customer Intent Detection:**
Determine what the customer is trying to accomplish:
- **Request** — asking for something to be done (password reset, feature activation, account change)
- **Complaint** — expressing dissatisfaction with a product, service, or experience
- **Inquiry** — asking for information or clarification
- **Escalation** — demanding elevated attention, referencing prior unresolved issues
- **Urgent/Critical** — reporting an outage, data loss, or business-stopping issue

**Similar Prior Cases:**
Search for prior cases with similar issue descriptions and the same category:
- From the case tracker: filter by same category, last 90 days
- From CRM if available: `SearchM365(sources=["connectors"], connector_ids=["crm-connector"])`
- Note resolutions and outcomes to inform the classification rationale

**Confidence Assessment:**
- **High** — clear keyword match to taxonomy, prior cases confirm the pattern, unambiguous intent
- **Medium** — reasonable match but could fit multiple categories, or limited prior case evidence
- **Low** — no clear taxonomy match, ambiguous language, or conflicting signals

### Step 3: Present Classification

Present the classification via Adaptive Card (invoke `render-ui` skill first):

- **Recommended category** from the taxonomy with supporting rationale
- **Customer intent** (request, complaint, inquiry, escalation, urgent/critical)
- **Confidence level** (high, medium, low) with explanation
- **Suggested queue** based on the category-to-queue mapping in the taxonomy
- **Similar prior cases** (up to 3) with case ID, issue summary, resolution, and outcome
- **Alternative categories** if confidence is medium or low

After user confirms the classification, update the case tracker with the assigned category.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, service taxonomy, context packet |
| SearchM365 (connectors) | Pull prior case classifications from CRM |
| ReadFileContent | Read taxonomy, context packet, case data |
| GetMessage | Read original inbound message |

## Guardrails

- **Present classification as recommendation only** — never auto-assign category without user review
- **Always show confidence level** — flag low-confidence classifications prominently with explanation
- **Surface similar prior cases** to support the classification rationale
- **Never classify based on customer identity alone** — prevent bias toward account tier; classify on issue content
- **Flag ambiguous cases** — when the issue fits multiple categories equally, present all options
- **Preserve original issue language** — show the customer's own words alongside the classification
- **Log classification decision** — record category, confidence, and rationale in the case tracker after confirmation
