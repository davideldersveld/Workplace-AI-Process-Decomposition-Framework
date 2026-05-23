---
name: proc-context-packet
description: |
  Assembles supplier profile, onboarding policies, required document
  checklists, screening requirements, and reviewer contacts into a
  structured context packet for supplier onboarding.
  Use when user asks to "build supplier packet for [supplier]",
  "assemble onboarding context for [supplier]",
  "what do we need for [supplier] onboarding",
  "gather supplier context for [case]",
  "prepare onboarding packet for [supplier]",
  or "pull requirements for [supplier] onboarding".
  Do NOT use for creating a new supplier case (use proc-supplier-intake),
  detecting missing items or risk indicators (use proc-gap-risk-detect),
  determining review path (use proc-review-routing),
  drafting outreach communications (use proc-supplier-comms),
  or summarizing the reviewer packet (use proc-reviewer-packet).
---

## Overview

Assembles a comprehensive context packet for an active supplier onboarding case — pulling the supplier profile, applicable onboarding policies by spend category and geography, required document checklists, screening requirements, and assigned reviewer contacts. The packet is generated as a Word document saved to the supplier's SharePoint onboarding folder.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (policy repositories, checklists, vendor master, people directory) without exercising judgment on risk level or approval requirements.

## When to Use

- A supplier onboarding case has been created and needs context assembled before gap detection
- A procurement analyst needs the applicable policies and checklists for a supplier's category and geography
- An onboarding packet needs updating after a category or geography change
- A new analyst is assigned and needs the full onboarding context for a case

## When NOT to Use

- Creating a new supplier case — use proc-supplier-intake
- Detecting missing documents or risk indicators — use proc-gap-risk-detect
- Determining the review path and assigning reviewers — use proc-review-routing
- Drafting outreach or follow-up communications — use proc-supplier-comms
- Summarizing the case for reviewer approval — use proc-reviewer-packet
- Confirming onboarding disposition — this is always a human decision (PR-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and gather policy requirements", activeForm="Gathering supplier context")
TaskCreate(subject="Assemble context packet document", activeForm="Building onboarding packet")
```

### Step 1: Read Onboarding Case Data

Locate and read the case:
- `SearchM365(sources=["files"], query="onboarding tracker")` then `ReadFileContent`
- Identify: Case ID, Supplier Name, Requester, Spend Category, Geography, Priority, Estimated Spend, Status

### Step 2: Retrieve Supplier Profile

Pull existing supplier data:

**Via Graph Connector (if available):**
- `SearchM365(sources=["connectors"], connector_ids=["vendor-master-connector"])` — existing vendor master record, prior contracts, payment history, supplier classification

**Via SharePoint (if Graph Connector unavailable):**
- `SearchM365(sources=["files"], query="[supplier name] vendor")` then `ReadFileContent` — vendor master exports, prior onboarding records

Extract:
- Legal entity name, DBA names, and registered address
- Existing vendor master status (new, reactivation, or expansion)
- Prior contracts or purchase orders (if any)
- Supplier diversity classification (if applicable)

### Step 3: Retrieve Onboarding Policies

Find applicable policy documents by category and geography:
- `SearchM365(sources=["files"], query="[spend category] onboarding policy")` then `ReadFileContent` — category-specific requirements
- `SearchM365(sources=["files"], query="[geography] supplier requirements")` then `ReadFileContent` — geography-specific regulatory requirements
- `SearchM365(sources=["files"], query="supplier onboarding checklist")` then `ReadFileContent` — master onboarding checklist
- `SearchM365(sources=["files"], query="supplier insurance requirements")` then `ReadFileContent` — insurance minimums by category

Extract:
- Required onboarding documents by category (W-9/W-8, insurance certificates, banking forms, contracts)
- Geography-specific requirements (tax registration, local compliance, import/export controls)
- Insurance minimums (general liability, professional liability, workers' compensation)
- Contractual requirements (MSA, NDA, SOW, data processing agreement)

### Step 4: Retrieve Screening Requirements

Identify required screening and due diligence:
- `SearchM365(sources=["files"], query="supplier risk screening requirements")` then `ReadFileContent`
- `SearchM365(sources=["files"], query="sanctions screening policy")` then `ReadFileContent`

Extract:
- Sanctions and debarment screening requirements (OFAC, EU sanctions, debarment lists)
- Conflict of interest screening requirements
- Financial health check requirements (for high-spend categories)
- Cybersecurity assessment requirements (for IT and data-handling suppliers)
- Environmental and social governance screening (if applicable)

### Step 5: Resolve Reviewer Contacts

Identify the team who will review this case:
- `SearchPeople` — resolve category manager for this spend category
- `SearchPeople` — resolve risk reviewer by geography or risk tier
- `GetUserDetails` — verify reviewer roles and availability
- `SearchM365(sources=["files"], query="approval matrix")` then `ReadFileContent` — identify required reviewers by spend tier and risk level

Compile reviewer roster:
- Category manager (required for all cases)
- Risk reviewer (required if risk flags are present)
- Legal reviewer (required if contracts or IP are involved)
- Tax reviewer (required for international suppliers)
- AP onboarding specialist (required for banking and payment setup)

### Step 6: Check Existing Documents

Inventory what has already been uploaded:
- `GetDriveChildren` — list documents in the supplier's onboarding folder in SharePoint
- Note which required documents are already present and which are missing

### Step 7: Assemble Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Case Summary**
   - Case ID, Supplier Name, Requester, Spend Category, Geography
   - Priority and SLA target date
   - Assigned procurement analyst

2. **Supplier Profile**
   - Legal entity details and registered address
   - Vendor master status (new, reactivation, expansion)
   - Prior relationship history (if any)
   - Diversity classification (if applicable)

3. **Required Documents Checklist**
   - Category-specific document requirements with status (received / pending / not required)
   - Geography-specific regulatory documents
   - Insurance requirements with minimum coverage amounts
   - Contractual documents required

4. **Screening Requirements**
   - Sanctions and debarment screening status
   - Conflict of interest screening requirement
   - Financial health check requirement
   - Cybersecurity assessment requirement (if applicable)
   - Source document references for each requirement

5. **Reviewer Contacts**
   - Assigned reviewers by role with names and contact information
   - Spend tier and approval authority context

6. **Dependencies and Open Questions**
   - Missing policy documents flagged
   - Unclear category or geography assignments
   - Questions requiring requester input

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find onboarding tracker, policies, checklists, insurance requirements, approval matrix |
| SearchM365 (connectors) | Pull vendor master data via Graph Connector |
| ReadFileContent | Read all policy documents, checklists, tracker, approval matrix |
| GetDriveChildren | Inventory uploaded supplier documents in onboarding folder |
| SearchPeople | Resolve category manager, risk reviewer, and other reviewer contacts |
| GetUserDetails | Verify reviewer roles and availability |

## Guardrails

- **Cite policy source and version** for every requirement listed — the analyst must know which policy document drives each requirement
- **Flag if any required policy document is missing from SharePoint** — a missing policy creates a gap in the onboarding checklist
- **Never include bank account details, tax identifiers (EIN/SSN), or sensitive financial data** in the context packet — these are collected through secure channels
- **Mark the packet as draft** until reviewed by the assigned analyst — the packet is a working document, not a final record
- **Never generate onboarding requirements from assumptions** — extract only from documented policies; flag gaps for procurement operations review
- **Never assess supplier risk or recommend a risk tier** — context assembly presents facts; risk assessment belongs to proc-gap-risk-detect
- **Flag if the supplier geography triggers sanctions-sensitive screening requirements** — this is a factual flag based on policy, not a risk judgment
- **Include spend tier context** when available — estimated annual spend determines which approval authority chain applies
- **Never fabricate supplier history, screening status, or compliance data** — if data is unavailable, state the gap rather than estimating
- **Restrict the generated packet** to the supplier's onboarding folder in SharePoint with appropriate access controls
