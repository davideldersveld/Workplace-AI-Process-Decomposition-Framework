---
name: ins-claim-context
description: |
  Assembles policy details, claimant profile, loss narrative, prior claim history,
  evidence inventory, and applicable procedures into a claim context packet.
  Use when user asks to "build claim context for [claim ID]",
  "assemble claim packet for [claim]", "gather policy details for this loss",
  "what do we know about claim [number]", "pull policy and loss context",
  "prepare claim materials for [claimant]", "evidence context for [claim ID]",
  or "policy and claimant summary for [loss]".
  Do NOT use for creating a new claim case (use ins-fnol-intake),
  classifying claim type or severity (use ins-claim-classifier),
  assessing coverage path or evidence gaps (use ins-coverage-gap-detection),
  routing to adjusters or SIU (use ins-claim-routing),
  or drafting claimant or adjuster communications (use ins-claim-comms).
---

## Overview

Assembles a comprehensive claim context packet for a case by gathering policy summary and endorsements, claimant and insured profile, loss narrative, prior claim history, evidence inventory, applicable jurisdiction and regulatory notes, and claim handling guidelines. Produces a Word document for the auditable claim file.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (SharePoint policy documents, Graph Connectors for claims platform data, claims tracker) without interpreting coverage, making liability assessments, or recommending reserves.

## When to Use

- A claim case has been created and needs policy and loss context assembled
- An adjuster needs the full picture for a claim before classification or review
- Prior claim history needs to be compiled for a pattern investigation
- A context packet needs to be refreshed with updated information

## When NOT to Use

- Creating a new claim case — use ins-fnol-intake
- Classifying the claim type or severity — use ins-claim-classifier
- Assessing initial coverage path or missing evidence — use ins-coverage-gap-detection
- Routing to an adjuster, catastrophe desk, or SIU — use ins-claim-routing
- Drafting claimant, broker, or adjuster communications — use ins-claim-comms
- Confirming triage disposition — this is always a human decision (INS-CLM-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read claim data and locate source materials", activeForm="Reading claim data")
TaskCreate(subject="Assemble claim context packet", activeForm="Assembling context packet")
```

### Step 1: Read Claim Data and Locate Source Materials

**Read the claim case record:**
- `SearchM365(sources=["files"], query="claims tracker")` then `ReadFileContent` to find the case row by Claim ID or policy number

**Resolve people:**
- `GetUserDetails` — assigned adjuster (if any), intake specialist
- `SearchPeople` — agent, broker, claims supervisor contacts

**Retrieve policy data (when available):**
- `SearchM365(sources=["connectors"], connector_ids=["policy-admin-connector"])` — policy summary, coverage type, limits, deductible, effective dates, named insureds, endorsements
- If Graph Connectors are not available, search SharePoint for policy documents
- `SearchM365(sources=["files"], query="policy [policy number]")` — policy declarations, endorsement schedules

**Retrieve claims history:**
- `SearchM365(sources=["connectors"], connector_ids=["claims-platform-connector"])` — prior claims on the same policy or claimant
- If Graph Connectors are not available, search the tracker for prior entries with the same policy number or claimant name

**Locate procedures and reference materials:**
- `SearchM365(sources=["files"], query="claims handling guidelines")` — claim handling procedures for the loss type
- `SearchM365(sources=["files"], query="evidence checklist [product line]")` — evidence requirements by claim type and product line
- `SearchM365(sources=["files"], query="[jurisdiction] claims requirements")` — state-specific regulatory requirements
- `ReadFileContent` — read each located document

**Find related communications and evidence:**
- `SearchM365(sources=["email"], query="[Claim ID] OR [claimant name] OR [policy number]")` — related claimant correspondence, agent submissions, loss notification threads
- `GetDriveChildren` — list documents in the claim evidence folder in SharePoint

**Log every document and data source accessed** — record source system, document name, and retrieval timestamp for evidence traceability.

### Step 2: Assemble Context Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Policy Summary**
   - Policy number, product line, coverage type
   - Policy effective dates and status on date of loss
   - Named insureds and additional insureds
   - Coverage limits and deductible amounts
   - Key endorsements and exclusions relevant to the loss type
   - Policy document source and version cited

2. **Claimant and Insured Profile**
   - Claimant name and relationship to the policy (named insured, additional insured, third party)
   - Contact information on file
   - Agent or broker of record
   - Claimant SSN masked — do not include full SSN in the packet

3. **Loss Narrative**
   - Date, time, and location of loss
   - Loss description from the FNOL intake
   - Reporting channel and initial narrative source
   - Key details from phone transcript or email correspondence (with timestamps)

4. **Prior Claim History**
   - Prior claims on the same policy (Claim ID, date, type, status, outcome)
   - Prior claims by the same claimant across policies (if available)
   - Pattern indicators (frequency, similar loss types, same providers)

5. **Evidence Inventory**
   - Documents received (from claim evidence folder): photos, police reports, repair estimates, proof of loss forms, medical documents
   - Documents pending (from evidence checklist comparison)
   - Catastrophe event indicators (if applicable — declared catastrophe zones, weather events)

6. **Jurisdiction and Regulatory Notes**
   - State of loss and applicable jurisdiction
   - Key state-specific claims handling requirements (proof of loss timelines, mandatory disclosure language)
   - Unfair claims practices act considerations for the jurisdiction

7. **Applicable Handling Guidelines**
   - Claim handling procedures for the loss type and product line
   - Evidence requirements from the product-specific checklist
   - Routing considerations (standard adjuster, catastrophe desk, bodily injury team, SIU)

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find claims tracker, policy documents, endorsements, handling guidelines, evidence checklists, regulatory references |
| SearchM365 (email) | Find related claimant correspondence and loss notification threads |
| SearchM365 (connectors) | Retrieve policy and claims data from policy admin and claims platform via Graph Connector |
| ReadFileContent | Read policy documents, checklists, tracker, handling guidelines, and evidence documents |
| GetDriveChildren | List documents in the claim evidence folder |
| GetUserDetails / SearchPeople | Resolve adjuster, agent, broker, and claims supervisor identities |

## Guardrails

- **Never include coverage opinions or reserve recommendations** in the context packet — present factual policy and loss information only; coverage determination is adjuster-owned
- **Cite policy document source and version** for every policy extract — adjusters must be able to trace every fact to the source document
- **Flag if the policy was not active on the date of loss** — present as a factual finding ("Policy status on date of loss: inactive"), not a coverage determination
- **Mask claimant SSN, financial account numbers, and medical details** in generated documents unless explicitly required by the claim type and the user confirms inclusion
- **Flag if any required policy document is unfindable** in SharePoint — note explicitly as "document not located" with search terms used
- **Do not surface SIU or fraud investigation status** in the context packet — SIU information is restricted and must never appear in general claim documentation
- **Read-only access to all policy and claims data** — no write operations to policy admin or claims platform
- **Never interpret coverage, liability, or claim validity** — present factual context only; the adjuster makes all judgment calls
- **Preserve evidence traceability** — log every document and data source accessed with timestamp and source location
- **Restrict packet access** — note that the generated packet should be stored in the claim folder with appropriate SharePoint permissions
