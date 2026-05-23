---
name: ops-classify-exception
description: |
  Classifies operational exception type and likely cause based on
  context packet data and the exception taxonomy.
  Use when user asks to "classify this exception",
  "what type of exception is [case ID]",
  "categorize the issue for [case]",
  "what caused [transaction] to fail",
  "classify exception [ID]",
  or "determine exception type for [case]".
  Do NOT use for creating a new exception case (use ops-exception-intake),
  gathering transaction context (use ops-context-packet),
  assessing impact and priority (use ops-impact-assess),
  routing to an owner or queue (use ops-route-exception),
  or drafting follow-up communications (use ops-exception-comms).
---

## Overview

Classifies an operational exception by type and likely cause based on the context packet, exception taxonomy, and prior case patterns. Presents the classification as a recommendation with supporting evidence, confidence level, and similar prior cases for the analyst to review and confirm. The analyst retains full authority to accept, modify, or override the classification.

This skill operates in "AI assist" mode — it reads and analyzes exception data but only presents classification as a recommendation. The operations analyst reviews and confirms before the classification is recorded.

## When to Use

- An exception case has context assembled and needs to be classified by type
- An analyst wants to determine the most likely cause of an exception
- A case needs reclassification after new information changes the picture
- A queue manager wants to review classification patterns across cases

## When NOT to Use

- Creating a new exception case — use ops-exception-intake
- Gathering process and transaction context — use ops-context-packet
- Assessing impact, priority, and aging risk — use ops-impact-assess
- Assigning an owner or routing to a queue — use ops-route-exception
- Drafting follow-up or handoff communications — use ops-exception-comms
- Confirming triage disposition — this is always a human decision (OPS-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and exception taxonomy", activeForm="Reading classification inputs")
TaskCreate(subject="Present classification recommendation", activeForm="Classifying exception")
```

### Step 1: Read Classification Inputs

**Read the exception case and context:**
- `SearchM365(sources=["files"], query="exception tracker")` then `ReadFileContent` — current case data
- Review the context packet data (transaction details, error state, processing history, related communications)

**Read the exception taxonomy:**
- `SearchM365(sources=["files"], query="exception taxonomy")` then `ReadFileContent` — classification categories, definitions, and decision criteria
- `SearchM365(sources=["files"], query="exception classification rules")` then `ReadFileContent` — rules for distinguishing between exception types

### Step 2: Apply Exception Taxonomy

Match the exception against the taxonomy categories:

| Exception Type | Definition | Key Indicators |
|---------------|------------|----------------|
| **Processing error** | An error in standard transaction processing | Failed validation, incorrect calculation, missing step, timeout |
| **Data mismatch** | Discrepancy between systems or data sources | Conflicting values, missing records, out-of-sync state |
| **System failure** | Technical infrastructure or application failure | System error codes, connectivity loss, application crash, timeout |
| **Manual override needed** | Standard processing cannot handle the case | Edge case, exception to standard rules, non-standard transaction |
| **Compliance exception** | Regulatory or policy violation detected | Policy threshold breach, regulatory flag, audit finding |
| **Quality defect** | Product or service quality issue | Customer complaint, inspection failure, specification deviation |

### Step 3: Determine Likely Cause

Based on the exception type and available evidence:
- Identify the processing step where the exception originated
- Determine whether the cause is data-related, system-related, process-related, or human-error-related
- Note whether the cause appears to be a one-time event or a recurring pattern

### Step 4: Assess Confidence Level

Rate the classification confidence:

| Confidence | Criteria |
|-----------|----------|
| **High** | Clear error state, matching taxonomy indicators, consistent with prior similar cases |
| **Medium** | Partial indicators, some ambiguity in error state, limited prior case pattern |
| **Low** | Ambiguous error state, multiple possible types, no clear prior pattern |

### Step 5: Find Similar Prior Cases

Search for pattern context:
- `SearchM365(sources=["files"], query="[exception type] [process type] exception resolved")` — prior cases with the same classification
- Review the exception tracker for cases with similar characteristics
- If a Graph Connector is available: `SearchM365(sources=["connectors"], connector_ids=["transaction-system-connector"])` — additional transaction history

Compile:
- Number of similar cases in the last 90 days
- Most common resolution approach
- Average resolution time
- Whether this appears to be part of a recurring pattern

### Step 6: Apply Safety-First Rule

For exception types that may involve safety implications:

**Safety-first classification rule:** When evidence is ambiguous between a safety-critical type (quality defect, compliance exception, equipment-related system failure) and a non-safety type (processing error, data mismatch), **always classify as the more severe safety-critical type.** The analyst may downgrade after explicit review with documented rationale.

Safety-critical types:
- **Quality defect** — any indication of product or service quality issues
- **Compliance exception** — any indication of regulatory or policy violation
- **System failure with safety implications** — equipment failures or system failures that could affect safety

### Step 7: Present Classification Recommendation

Present via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Transaction Reference, Process Type, Queue, Aging
- **Recommended exception type** — the classification with definition
- **Likely cause** — cause hypothesis with supporting evidence
- **Confidence level** — high, medium, or low with explanation
- **Safety flag** — prominently displayed if safety-first rule was applied, with explanation
- **Evidence basis** — the specific indicators from the context that support this classification
- **Alternative classifications** — if confidence is medium or low, list other plausible types with their indicators
- **Similar prior cases** — count, resolution patterns, recurring pattern flag
- **Recommended resolution approach** — based on exception type and SOP reference
- **Draft label** — "CLASSIFICATION RECOMMENDATION — analyst review required before recording"

### Step 8: Record Classification (After Confirmation)

After the analyst confirms or modifies the classification:
- Update the exception tracker with: Exception Type, Cause Hypothesis, Confidence Level, Safety Flag, Classification Date, Classifying Analyst
- If the analyst overrides the recommendation, log the override with the original recommendation, the analyst's classification, and their rationale

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find exception tracker, taxonomy, classification rules, prior cases |
| SearchM365 (connectors) | Pull additional transaction history via Graph Connector |
| ReadFileContent | Read taxonomy, classification rules, tracker, SOPs |

## Guardrails

- **Present classification as a recommendation** — the analyst confirms or corrects; never auto-update the tracker without explicit review
- **Apply the safety-first rule** — when evidence is ambiguous between a safety-critical type and a non-safety type, always classify as the more severe type; require explicit analyst downgrade with documented rationale
- **Include the evidence basis** for every classification so the analyst can validate the reasoning
- **Flag low-confidence classifications explicitly** — the analyst needs to know when the classification is uncertain
- **Never auto-update the exception type in the tracker** without analyst review — classification drives downstream routing and priority, so accuracy matters
- **Log every classification and override** for process improvement analysis — override patterns reveal where the taxonomy or classification rules need refinement
- **Never assess feasibility or recommend specific resolution actions** — classification identifies the type and likely cause; resolution decisions belong to the analyst and resolver
- **Never generate exception types outside the taxonomy** — if the exception does not fit any defined category, flag it as "Unclassified — requires taxonomy review" for the queue manager
- **Never downgrade a safety-flagged classification** without analyst confirmation — safety-first is a one-way escalation in the classification step
- **Preserve the context packet findings** — classification builds on context; never contradict or omit context data in the classification summary
