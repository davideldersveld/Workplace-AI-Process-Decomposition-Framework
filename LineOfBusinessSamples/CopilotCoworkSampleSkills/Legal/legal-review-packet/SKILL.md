---
name: legal-review-packet
description: |
  Assembles contract context, clause playbook standards, counterparty history,
  and approval requirements into a structured review packet for counsel.
  Use when user asks to "build review packet for [case]",
  "assemble contract context", "pull playbook for [contract type]",
  "what's the standard for [clause type]",
  "prepare review materials for [counterparty]",
  "get context for legal case [ID]",
  or "contract review prep for [case]".
  Do NOT use for creating a new case (use legal-contract-intake),
  detecting clause deviations (use legal-deviation-detection),
  determining review path (use legal-review-routing),
  or drafting summaries and follow-ups (use legal-triage-comms).
---

## Overview

Assembles a comprehensive review packet for an active contract case — pulling the contract document, applicable clause playbook with approved fallback positions, standard template comparison baseline, counterparty history, requestor and business context, and required approvals. The packet is generated as a Word document saved to the SharePoint matter folder.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (playbooks, policies, service catalog, directory) without exercising judgment on contract substance or deviations.

## When to Use

- A contract case has been created and needs a complete context package before review
- Counsel needs the applicable playbook standards and fallback positions for a contract type
- Legal operations needs to assemble counterparty history and prior agreement references
- A review packet needs updating after new documents or context are added to the case

## When NOT to Use

- Creating a new case record — use legal-contract-intake
- Detecting clause deviations against the playbook — use legal-deviation-detection
- Determining the review path or required approvals — use legal-review-routing
- Drafting summaries, status updates, or follow-up requests — use legal-triage-comms
- Confirming final triage disposition — this is always a human decision (LG-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and gather source documents", activeForm="Gathering contract context")
TaskCreate(subject="Assemble review packet document", activeForm="Building review packet")
```

### Step 1: Read Case Data

Locate and read the case record:
- `SearchM365(sources=["files"], query="contract case tracker")` then `ReadFileContent` — find the case and read current status
- Identify: Case ID, Counterparty, Contract Type, Business Unit, Requestor, Urgency

### Step 2: Locate the Contract Document

Find the contract or redline:
- `SearchM365(sources=["files"], query="[counterparty] [contract type] contract")` — search for the uploaded contract
- `GetDriveChildren` — browse the matter folder for the contract document and any exhibits or amendments
- `ReadFileContent` — read the contract document to extract basic metadata (parties, type, effective date, jurisdiction)

If the contract document is not found, flag this as a gap — the review packet will be incomplete without the primary document.

### Step 3: Retrieve Clause Playbook

Find the applicable playbook for this contract type:
- `SearchM365(sources=["files"], query="[contract type] clause playbook")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[contract type] fallback positions")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="[contract type] standard template")` then `ReadFileContent`

Extract from the playbook:
- Approved positions for material clauses (indemnification, limitation of liability, IP assignment, confidentiality, termination, governing law, data protection)
- Acceptable fallback positions for each clause
- Clauses that require escalation if deviated from
- Contract-type-specific review requirements

If a required playbook document is not found, flag this prominently in the packet.

### Step 4: Gather Counterparty History

Search for prior agreements and interactions:
- `SearchM365(sources=["files"], query="[counterparty] agreement")` — prior contracts
- `SearchM365(sources=["files"], query="[counterparty] exception approval")` — prior deviation exceptions
- `SearchM365(sources=["connectors"], connector_ids=["clm-connector"])` — CLM records for this counterparty (if Graph Connector available)

Compile:
- Prior agreements with this counterparty (type, date, status)
- Prior exception approvals granted for this counterparty
- Any known negotiation patterns or special considerations

### Step 5: Gather Requestor and Business Context

- `GetUserDetails` — requestor's role, department, and reporting chain
- `GetManagerDetails` — requestor's manager for escalation awareness
- `SearchM365(sources=["files"], query="[business unit] contract authority")` — business unit contract authority limits and delegation

### Step 6: Retrieve Approval Requirements

- `SearchM365(sources=["files"], query="approval matrix")` then `ReadFileContent` — approval matrix for this contract type and value threshold
- Identify required approvers based on:
  - Contract type (NDA, MSA, SOW, etc.)
  - Contract value threshold (if known)
  - Business unit authority limits
  - Jurisdiction-specific requirements

### Step 7: Assemble Review Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Case Summary**
   - Case ID, Counterparty, Contract Type, Business Unit, Requestor
   - Urgency level and SLA target
   - Received date and current case status

2. **Contract Overview**
   - Parties, contract type, jurisdiction, effective date (if identifiable from the document)
   - Document location reference (matter folder path)

3. **Applicable Clause Playbook Standards**
   - Material clause positions from the playbook with section references
   - Approved fallback positions for each material clause
   - Escalation-required clauses flagged

4. **Standard Template Comparison Baseline**
   - Reference to the applicable standard template
   - Key structural differences from the standard form (if identifiable)

5. **Counterparty History**
   - Prior agreements (type, date, status)
   - Prior exception approvals
   - Known negotiation patterns

6. **Requestor and Business Context**
   - Requestor profile and business unit
   - Business unit contract authority limits
   - Reporting chain relevant to approvals

7. **Required Approvals**
   - Approval matrix requirements for this contract type
   - Named approvers identified from the matrix
   - Value-threshold-triggered approvals (if contract value is known)

8. **Gaps and Flags**
   - Missing documents (contract not found, playbook not found)
   - Stale data warnings (playbook older than 12 months, counterparty data older than 6 months)
   - Incomplete fields requiring follow-up

Save the packet to the SharePoint matter folder.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find case tracker, contract documents, playbooks, fallback positions, templates, approval matrix, counterparty history |
| SearchM365 (connectors) | Retrieve CLM records for counterparty history (if available) |
| ReadFileContent | Read all source documents — tracker, contract, playbooks, policies, approval matrix |
| GetDriveChildren | Browse matter folders and template libraries |
| GetUserDetails | Requestor profile and department |
| GetManagerDetails | Requestor reporting chain for escalation context |
| SearchPeople | Resolve approver and counsel names |

## Guardrails

- **Cite playbook source and version** for every clause standard referenced — traceability is non-negotiable in legal work
- **Never include attorney work product or privileged analysis** in the packet — this is factual context assembly only; legal judgment belongs to counsel
- **Never summarize or interpret contract terms** — present source text with references to the specific document and section
- **Flag missing playbook documents prominently** — a review without the applicable playbook is incomplete and potentially risky
- **Flag stale data** — playbooks older than 12 months, counterparty data older than 6 months, or approval matrices with no recent update date should be flagged for legal operations review
- **Restrict access** to the generated packet — save to the matter folder with access scoped to the legal team SharePoint group
- **Never extract or display full clause text in non-document channels** — the review packet is a Word document in SharePoint, not an email or Teams message
- **Do not assess or characterize deviations** — context assembly identifies what standards apply; deviation detection (legal-deviation-detection) determines what differs
- **Include the case SLA target** in the packet header so counsel and legal operations can track triage timeliness
