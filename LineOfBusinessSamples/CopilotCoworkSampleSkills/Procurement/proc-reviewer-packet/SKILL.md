---
name: proc-reviewer-packet
description: |
  Summarizes the supplier onboarding case into a review-ready packet
  with case status, risk findings, checklist completion, and evidence
  inventory for reviewer approval.
  Use when user asks to "prepare review packet for [supplier]",
  "summarize supplier case for review",
  "onboarding status for [supplier]",
  "build approval packet for [supplier case]",
  "reviewer summary for [supplier]",
  or "compile onboarding evidence for [case]".
  Do NOT use for creating a new supplier case (use proc-supplier-intake),
  gathering supplier and policy context (use proc-context-packet),
  detecting missing items or risk indicators (use proc-gap-risk-detect),
  determining review path (use proc-review-routing),
  or drafting outreach communications (use proc-supplier-comms).
---

## Overview

Synthesizes all onboarding case artifacts — the case record, context packet, gap and risk report, screening results, uploaded evidence documents, reviewer assignments, and communication history — into a consolidated reviewer packet. The packet provides reviewers with everything they need to make an informed approval, rejection, or further-information decision. Available as a Word document (formal reviewer packet), Adaptive Card (quick status view), or tracker update.

This skill operates in "AI draft plus approve" mode — the reviewer packet is presented for analyst review and confirmation before it is finalized and shared with reviewers.

## When to Use

- A supplier onboarding case is ready for formal review and needs a consolidated evidence packet
- A reviewer needs a structured summary of the case before starting their review
- A procurement manager needs a quick status view of a case before an approval meeting
- A case needs an updated summary after new documents or risk information arrives

## When NOT to Use

- Creating a new supplier case — use proc-supplier-intake
- Gathering supplier and policy context — use proc-context-packet
- Detecting missing documents or risk indicators — use proc-gap-risk-detect
- Determining the review path and assigning reviewers — use proc-review-routing
- Drafting outreach or follow-up communications — use proc-supplier-comms
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read all case artifacts", activeForm="Reading onboarding case data")
TaskCreate(subject="Assemble reviewer packet", activeForm="Building reviewer packet")
```

### Step 1: Read All Case Artifacts

**Read the case record:**
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent` — full case data including status, risk tier, checklist completion, assigned reviewers

**Read the context packet:**
- `SearchM365(sources=["files"], query="context packet [supplier name]")` then `ReadFileContent` — supplier profile, policy requirements, screening requirements

**Read the gap and risk report data:**
- Review risk flags, screening status, missing items, and completeness percentage from the tracker

**Inventory evidence documents:**
- `GetDriveChildren` — full inventory of uploaded documents in the supplier onboarding folder (W-9, insurance certs, banking forms, contracts, screening results)

**Read related communications:**
- `SearchM365(sources=["email"], query="[supplier name] onboarding")` — recent correspondence thread including missing document responses, reviewer comments

### Step 2: Assess Case Readiness

Evaluate whether the case is ready for review:

| Readiness Level | Criteria |
|----------------|---------|
| **Ready for review** | All required documents received, all screenings completed, no unresolved critical risk flags |
| **Conditionally ready** | Most items complete but minor gaps remain (non-critical missing documents, pending non-critical screening) |
| **Blocked** | Critical documents missing, screening not completed, unresolved critical risk flags |

### Step 3: Choose Output Format

Based on the user's request or case needs:

**Adaptive Card** (invoke `render-ui` skill first) — for quick status review:
- Case header with key fields
- Readiness level with reasoning
- Checklist completion summary
- Risk flags summary with disposition status
- Reviewer assignments and deadlines
- Open items and blockers

**Word document** (invoke `docx` skill) — for the formal reviewer packet:

1. **Case Summary**
   - Case ID, Supplier Name, Requester, Spend Category, Geography
   - Priority, SLA target date, days elapsed
   - Assigned procurement analyst
   - Estimated annual spend and spend tier

2. **Supplier Profile**
   - Legal entity details
   - Vendor master status (new, reactivation, expansion)
   - Prior relationship history
   - Diversity classification

3. **Checklist Status**
   - Complete inventory of required items with status (received / pending / expired / insufficient / not required)
   - Completion percentage
   - List of each received document with upload date and file reference

4. **Screening and Due Diligence**
   - Sanctions screening status and findings
   - Debarment check status
   - Conflict of interest screening status
   - Financial health check (if applicable)
   - Cybersecurity assessment (if applicable)
   - Source and date for each screening result

5. **Risk Assessment Summary**
   - Risk tier with justification
   - Active risk flags with severity and evidence
   - Risk flags cleared with disposition and reviewer
   - Residual risks and mitigations

6. **Review Assignments**
   - Each assigned reviewer with role, scope, deadline, and current status
   - Review sequence dependencies
   - Any overdue reviews flagged

7. **Open Items and Recommendations**
   - Items requiring reviewer decision
   - Missing information that could affect the review
   - Recommended disposition (approve, conditional approve, reject, return for more information)
   - Readiness level assessment

8. **Evidence Index**
   - Complete list of documents in the onboarding folder with file names, types, and upload dates
   - Mapping of each document to the checklist item it satisfies

**Excel tracker update:**
- Update the Status field to "Ready for Review" or "Blocked — [reason]"
- Update the Checklist Completion percentage

### Step 4: Present Packet for Review

Present the draft packet summary for analyst confirmation before finalizing:

- **Readiness assessment** — ready, conditionally ready, or blocked
- **Completeness** — checklist percentage and count
- **Risk summary** — active flags and dispositions
- **Reviewer status** — assignments, deadlines, and progress
- **Recommendation** — suggested disposition path
- **Draft label** — "REVIEWER PACKET DRAFT — analyst review required before sharing with reviewers"

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, context packet, policies, screening results |
| SearchM365 (email) | Find related correspondence and reviewer comments |
| ReadFileContent | Read all case artifacts from SharePoint |
| GetDriveChildren | Full inventory of uploaded evidence documents |

## Guardrails

- **Every finding must trace to a source artifact** — no fabricated status information, screening results, or risk dispositions
- **Clearly distinguish complete versus incomplete versus blocked items** in all output formats — reviewers need to know exactly what is and is not ready
- **Flag items requiring exception approval** with explicit callouts — exceptions to standard policy need documented approval
- **Mask sensitive financial details** (bank routing numbers, tax IDs, EIN/SSN) in the reviewer packet — reference the document by name and location ("W-9 received, filed in onboarding folder") without reproducing sensitive content
- **Present the packet for analyst review** before sharing with reviewers — the analyst confirms accuracy before distribution
- **Never fabricate readiness assessments** — if the data is insufficient to determine readiness, state the gap
- **Include the evidence index** mapping each document to its checklist item — reviewers need to locate supporting evidence quickly
- **Never recommend approval when critical risk flags are unresolved** — unresolved critical flags always result in a "blocked" or "conditional" readiness assessment
- **Include all reviewer assignments and their status** — a reviewer packet without clear accountability creates review gaps
- **Never modify risk flags or screening results** in the reviewer packet — the packet summarizes findings as-is; changes require re-running proc-gap-risk-detect
- **Include the SLA target date and elapsed time** — reviewers need urgency context to prioritize their review
