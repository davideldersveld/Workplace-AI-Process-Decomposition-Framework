---
name: sales-rfp-extraction
description: |
  Extracts structured requirements, questions, deadlines, and
  constraints from an RFP document for a proposal case.
  Use when user asks to "extract requirements from RFP",
  "parse customer questions from [RFP]",
  "what does the RFP ask for",
  "break down proposal requirements for [account]",
  "analyze RFP for [proposal ID]",
  or "structure the RFP requirements".
  Do NOT use for creating a new proposal case (use sales-proposal-intake),
  gathering account context (use sales-context-packet),
  mapping requirements to approved content (use sales-content-mapping),
  drafting the proposal response (use sales-proposal-draft),
  or routing for approval (use sales-approval-routing).
---

## Overview

Parses an RFP document to extract structured requirements, customer questions, technical specifications, compliance requirements, submission format requirements, evaluation criteria, deadlines, and terms and conditions references. Produces a requirements matrix as an Adaptive Card for quick review and an Excel worksheet for downstream tracking and coverage mapping. Preserves original RFP language alongside summarized versions.

This skill operates in "AI assist" mode — it reads and analyzes the RFP document but only presents extracted requirements for escalation engineer review. The proposal manager confirms the requirements matrix before it is used for content mapping and drafting.

## When to Use

- An RFP document has been received and needs to be broken into structured requirements
- A proposal manager needs to understand the scope and complexity of an RFP before kickoff
- The proposal team needs a requirements matrix to assign response owners and track progress
- An RFP amendment has been received and requirements need to be updated

## When NOT to Use

- Creating a new proposal case record — use sales-proposal-intake
- Gathering account history and approved content assets — use sales-context-packet
- Mapping requirements to approved content or identifying gaps — use sales-content-mapping
- Drafting the proposal response document or deck — use sales-proposal-draft
- Routing the proposal package for approval — use sales-approval-routing
- Confirming submission readiness — this is always a human decision (SL-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read RFP document and extract requirements", activeForm="Analyzing RFP")
TaskCreate(subject="Present requirements matrix for review", activeForm="Structuring requirements")
```

### Step 1: Locate and Read the RFP Document

**Find the RFP document:**
- `SearchM365(sources=["files"], query="RFP [account name]")` — locate the RFP document in SharePoint or OneDrive
- `GetDriveChildren` — check the proposal workspace folder for RFP-related documents
- `SearchM365(sources=["files"], query="[proposal ID] RFP")` — search by Proposal ID

**Read the RFP:**
- `ReadFileContent` — read the primary RFP document
- Check for supplementary attachments (addenda, amendments, Q&A documents, technical specifications)
- If the RFP is a PDF, read the full document; if it is a Word document, read all sections

**Read the proposal case data:**
- `SearchM365(sources=["files"], query="proposal tracker")` then `ReadFileContent` — Proposal ID, Account Name, Due Date, Product Areas

### Step 2: Extract Requirements by Category

Parse the RFP document to identify and categorize each requirement:

| Category | What to Extract |
|----------|----------------|
| **Technical requirements** | Product capabilities, integration needs, architecture constraints, performance specifications, scalability requirements |
| **Commercial requirements** | Pricing format, payment terms, licensing model, volume commitments, SLA expectations |
| **Legal requirements** | Contract terms, liability provisions, indemnification, IP ownership, data handling, confidentiality |
| **Compliance requirements** | Regulatory certifications, audit requirements, data residency, security standards (SOC 2, ISO 27001, GDPR) |
| **Submission format** | Page limits, font requirements, section ordering, file format, delivery method, number of copies |
| **Evaluation criteria** | Scoring methodology, weighted categories, mandatory pass/fail criteria, demonstration requirements |
| **Deadlines and milestones** | Proposal due date, Q&A deadline, presentation date, decision timeline, implementation start |
| **Terms and conditions** | Referenced master agreements, insurance requirements, subcontracting rules, conflict of interest declarations |

### Step 3: Structure Each Requirement

For each extracted requirement, capture:

| Field | Description |
|-------|------------|
| **Requirement ID** | Sequential identifier (REQ-001, REQ-002, etc.) |
| **Section** | RFP section or page reference where the requirement appears |
| **Requirement Text** | Original language from the RFP (verbatim) |
| **Summary** | Concise summary of the requirement in plain language |
| **Category** | Technical, Commercial, Legal, Compliance, Submission, Evaluation |
| **Priority** | Must (mandatory), Should (preferred), Nice (optional) — based on RFP language |
| **Complexity** | High, Medium, Low — based on response effort required |
| **Response Status** | "Pending" for all newly extracted requirements |
| **Assigned To** | Blank — to be assigned by the proposal manager |
| **Notes** | Ambiguities, cross-references to other requirements, or clarification needs |

### Step 4: Identify Ambiguous and Cross-Referencing Requirements

Flag requirements that need attention:

| Flag Type | Criteria |
|-----------|---------|
| **Ambiguous** | Requirement language is vague, contradictory, or open to multiple interpretations |
| **Cross-reference** | Requirement references another section, external document, or prior agreement |
| **Conflicting** | Two requirements appear to contradict each other |
| **Missing context** | Requirement references information not provided in the RFP |
| **Unusual** | Requirement is non-standard for this type of engagement and may need legal or commercial review |

### Step 5: Summarize RFP Metadata

Extract high-level RFP metadata:

- Issuing organization and contact information
- RFP reference number
- Total number of requirements extracted
- Requirements breakdown by category
- Key deadlines (Q&A, submission, presentation, decision)
- Evaluation methodology summary
- Mandatory pass/fail criteria (if any)
- Page or word limits for the response

### Step 6: Present Requirements Matrix

Present the extracted requirements via Adaptive Card (invoke `render-ui` skill first):

- **RFP Summary** — title, issuing org, total requirements, due date, evaluation methodology
- **Requirements by category** — count per category with complexity distribution
- **Flagged requirements** — ambiguous, conflicting, cross-referencing, or unusual items
- **Key deadlines** — all milestones extracted from the RFP
- **Mandatory pass/fail criteria** — items that must be addressed to remain eligible
- **Complexity assessment** — overall RFP complexity based on requirement count, category mix, and flagged items
- **Draft label** — "REQUIREMENTS EXTRACTION — proposal manager review required before tracker update"

### Step 7: Update Tracker (After Confirmation)

After the proposal manager confirms the requirements matrix:
- Write the requirements to the Excel requirements worksheet (add a sheet to the proposal tracker or create a separate requirements workbook)
- Update the proposal tracker Status to "Requirements Extracted"
- Record the requirement count and complexity rating

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find the RFP document, supplementary attachments, proposal tracker |
| ReadFileContent | Read the RFP document, addenda, amendments, and tracker |
| GetDriveChildren | Browse the proposal workspace folder for all RFP-related documents |
| render_ui (Adaptive Card) | Present the requirements matrix for review |

## Guardrails

- **Present extracted requirements for user review before writing to the tracker** — requirement interpretation involves judgment; the proposal manager must confirm the extraction is accurate
- **Flag ambiguous requirements prominently** — requirements that need customer clarification should be surfaced early to avoid delays
- **Never auto-classify a requirement as "not applicable"** — all requirements must be human-reviewed; even apparently irrelevant requirements may have contractual implications
- **Preserve original RFP language** alongside any summarized version — the verbatim text is the contractual basis for the response
- **Never modify the original RFP document** — the extraction produces a separate requirements matrix; the RFP remains untouched
- **Flag conflicting requirements** — contradictions between requirements can create contractual risk if not resolved before response
- **Do not interpret legal or contractual language** — flag legal requirements for legal review rather than summarizing their implications
- **Include the Proposal ID and RFP reference number** in every output for traceability
- **Flag if the RFP document appears incomplete** — missing sections, truncated content, or references to attachments not found should be surfaced
- **Never fabricate requirements** — if a section of the RFP is unreadable or ambiguous, report it as such rather than inferring intent
