---
name: legal-deviation-detection
description: |
  Identifies clause deviations between the contract and the clause playbook,
  classifies deviation severity, and presents findings for analyst review.
  Use when user asks to "check for deviations in [contract]",
  "clause deviation analysis for [case]",
  "what deviates from playbook in [contract]",
  "flag non-standard terms in [counterparty] agreement",
  "review contract against playbook",
  "deviation report for case [ID]",
  or "playbook comparison for [contract type]".
  Do NOT use for creating a new case (use legal-contract-intake),
  assembling review context (use legal-review-packet),
  determining review path (use legal-review-routing),
  or drafting summaries and follow-ups (use legal-triage-comms).
---

## Overview

Compares material clauses in the contract against the clause playbook's approved positions and acceptable fallback language. Classifies each deviation by severity level, cross-references against prior exception approvals for the same counterparty, and presents findings via Adaptive Card for quick review plus a detailed Word document for the case file.

This skill operates in "AI assist" mode — it reads and analyzes contract text against playbook standards but only presents deviations as factual comparisons. The analyst reviews and confirms findings before they are recorded in the case file.

## When to Use

- A contract has been received and the review packet is assembled — deviations need to be identified
- Counsel wants to see how a contract compares to the clause playbook before beginning review
- A contract has been redlined and needs deviation re-analysis against updated terms
- Legal operations needs a deviation count and severity profile for routing decisions

## When NOT to Use

- Creating a new case record — use legal-contract-intake
- Assembling contract context or playbook materials — use legal-review-packet
- Determining the review path or required approvals — use legal-review-routing
- Drafting summaries, status updates, or follow-up requests — use legal-triage-comms
- Confirming final triage disposition — this is always a human decision (LG-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read contract and playbook for comparison", activeForm="Reading contract and playbook")
TaskCreate(subject="Identify deviations and present findings", activeForm="Analyzing clause deviations")
```

### Step 1: Read Contract and Playbook

**Read the contract document:**
- `SearchM365(sources=["files"], query="[counterparty] [contract type] contract")` then `ReadFileContent`
- Or locate the contract via `GetDriveChildren` in the matter folder

**Read the applicable clause playbook:**
- `SearchM365(sources=["files"], query="[contract type] clause playbook")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[contract type] fallback positions")` then `ReadFileContent`

**Read the review packet (if available):**
- `SearchM365(sources=["files"], query="review packet [Case ID]")` then `ReadFileContent` — for playbook extracts and counterparty history already assembled

**Read prior exception approvals:**
- `SearchM365(sources=["files"], query="[counterparty] exception approval")` then `ReadFileContent` — prior deviations approved for this counterparty

### Step 2: Identify Material Clauses

Identify the material clauses present in the contract that require playbook comparison. Standard material clauses include:

| Clause Category | Examples |
|----------------|----------|
| **Indemnification** | Mutual vs. one-way indemnification, carve-outs, caps |
| **Limitation of liability** | Cap amounts, exclusions, consequential damages waiver |
| **Intellectual property** | IP ownership, license grants, IP assignment, background IP |
| **Confidentiality** | Scope, duration, permitted disclosures, return or destruction |
| **Termination** | Termination for convenience, cure periods, survival clauses |
| **Governing law and dispute resolution** | Jurisdiction, venue, arbitration vs. litigation |
| **Data protection** | Data processing terms, security obligations, breach notification |
| **Representations and warranties** | Scope, survival period, remedies for breach |
| **Insurance** | Coverage requirements, minimums, evidence of insurance |
| **Assignment** | Consent requirements, change of control provisions |
| **Force majeure** | Covered events, notice requirements, termination rights |
| **Non-solicitation and non-compete** | Scope, duration, geographic limitations |

### Step 3: Compare Against Playbook Standards

For each material clause present in the contract:

1. Identify the playbook's **approved position** for this clause type and contract type
2. Identify the playbook's **acceptable fallback position** (if defined)
3. Compare the contract's clause against both the approved and fallback positions
4. Determine whether the contract clause:
   - Matches the approved position
   - Falls within the acceptable fallback range
   - Deviates from both approved and fallback positions
   - Is missing entirely when the playbook requires it

### Step 4: Classify Deviation Severity

Assign a severity level to each deviation:

| Severity | Definition | Action Required |
|----------|------------|-----------------|
| **Level 1 — Within approved range** | Clause matches the approved position or acceptable fallback | No action — clause is within policy |
| **Level 2 — Requires counsel review** | Clause deviates from approved and fallback positions but falls within a negotiable range | Counsel review required before acceptance |
| **Level 3 — Requires senior counsel or business approval** | Clause deviates significantly from policy; acceptance requires elevated authority | Senior counsel or business approver must review |
| **Level 4 — Outside policy — must escalate** | Clause is fundamentally inconsistent with policy or creates unacceptable risk exposure | Must escalate to senior counsel or legal operations manager; cannot be approved at standard review level |

### Step 5: Cross-Reference Prior Exceptions

For each deviation at severity level 2 or above:
- Check whether a similar deviation was previously approved for this counterparty
- If a prior exception exists, note the exception Case ID, date, and approving authority
- Prior exceptions do not auto-approve current deviations — they provide context for counsel

### Step 6: Identify Missing Required Clauses

Check whether the contract is missing clauses that the playbook requires for this contract type:
- Required clauses that are absent entirely
- Required sections that are present but substantively incomplete

Flag missing required clauses as deviations at the appropriate severity level.

### Step 7: Present Deviation Findings

**Quick review via Adaptive Card** (invoke `render-ui` skill first):

- **Case header** — Case ID, Counterparty, Contract Type
- **Deviation summary** — total count by severity level
- **Top deviations** — the highest-severity deviations with clause reference, playbook standard, and contract position
- **Prior exceptions** — counterparty-specific exceptions found
- **Missing required clauses** — clauses required by the playbook but absent from the contract
- **Privilege-sensitive clauses** — indemnification, limitation of liability, IP assignment flagged with elevated visibility
- **Draft label** — "DEVIATION ANALYSIS — analyst review required before recording"

**Detailed deviation report as Word document** (invoke `docx` skill):

For each material clause analyzed:
- Clause name and contract section reference
- Playbook approved position (with playbook section reference)
- Playbook fallback position (if defined)
- Contract position (summary of what the contract says)
- Deviation severity level (1 through 4)
- Prior exception reference (if applicable)
- Analyst notes field (blank — for counsel to complete during review)

Save the deviation report to the SharePoint matter folder after analyst confirmation.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find contract document, clause playbook, fallback positions, prior exception approvals, review packet, case tracker |
| ReadFileContent | Read contract text, playbook standards, fallback language, prior exceptions |
| GetDriveChildren | Browse matter folder for contract documents and exhibits |

## Guardrails

- **Present all findings as factual comparisons against playbook standards** — never characterize findings as legal advice, risk assessments, or recommendations to accept or reject
- **Every deviation must cite the specific playbook section and the specific contract clause** — unsourced deviations are not actionable and undermine trust
- **Never recommend accepting or rejecting a clause** — present the deviation and the playbook standard; the accept or reject decision belongs to counsel
- **Flag privilege-sensitive clauses with elevated visibility** — indemnification, limitation of liability, IP assignment, and similar high-impact clauses deserve prominent treatment regardless of severity level
- **Present findings via Adaptive Card for analyst review** before writing the deviation report to the case file — this is a read-only analysis skill until confirmation
- **Never modify the contract document** — this skill analyzes only; redlining and negotiation are human activities
- **Never modify the case tracker** — deviation count and severity profile are recorded after analyst confirmation, not auto-written
- **If the playbook is not found or is outdated**, flag this prominently — deviation analysis without a current playbook is unreliable
- **Prior exception approvals provide context, not authorization** — never present a prior exception as approval for the current deviation
- **Use taxonomy-defined clause categories only** — do not invent clause categories; if a clause does not fit the playbook taxonomy, flag it for manual classification
- **Never include contract text, deviation details, or playbook excerpts in Teams messages or emails** — all deviation content stays in the Word report within the SharePoint matter folder
