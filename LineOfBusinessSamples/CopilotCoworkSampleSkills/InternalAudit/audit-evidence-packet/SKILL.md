---
name: audit-evidence-packet
description: |
  Assembles scope, control ownership, prior findings, evidence requirements,
  and methodology references into a structured audit evidence packet.
  Use when user asks to "build audit packet for [case]", "assemble evidence context",
  "pull prior findings for [control area]", "prepare audit scope materials",
  "what evidence do we need for [engagement]", "audit packet for [case ID]",
  "scope context for [process area]", or "evidence requirements for [engagement]".
  Do NOT use for creating a new audit case (use audit-request-intake),
  detecting evidence gaps (use audit-gap-detection),
  routing evidence requests (use audit-request-routing),
  or drafting audit communications (use audit-case-comms).
---

## Overview

Assembles a comprehensive audit evidence packet for an engagement by gathering scope definition, control environment details, control ownership, prior findings and remediation status, evidence requirements, applicable methodology standards, and key contacts. Produces a Word document for the workpaper trail and an Excel evidence requirements checklist for tracking.

This skill operates in "AI act within policy" mode — it retrieves approved context from defined sources (SharePoint methodology library, control library, prior audit reports, engagement folders) without interpreting policy, drawing conclusions, or making audit judgments.

## When to Use

- An audit engagement has been created and needs scope and evidence context assembled
- An auditor needs prior findings and control ownership for a process area
- Evidence requirements need to be compiled for an upcoming walkthrough
- An audit packet needs to be refreshed for a follow-up engagement

## When NOT to Use

- Creating a new audit case — use audit-request-intake
- Detecting missing evidence or scope gaps — use audit-gap-detection
- Routing evidence requests to control owners — use audit-request-routing
- Drafting audit summaries or communications — use audit-case-comms
- Confirming audit packet disposition — this is always a human decision (IA-006)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read case data and locate source materials", activeForm="Reading case data")
TaskCreate(subject="Assemble audit evidence packet", activeForm="Assembling audit packet")
```

### Step 1: Read Case Data and Locate Source Materials

**Read the audit case record:**
- `SearchM365(sources=["files"], query="audit case tracker")` then `ReadFileContent` to find the case row by Case ID or process area

**Resolve people:**
- `GetUserDetails` — lead auditor, audit manager
- `SearchPeople` — control owners and process owners for the process area under audit
- `GetManagerDetails` — reporting chain for control ownership accountability

**Locate methodology and reference materials:**
- `SearchM365(sources=["files"], query="audit methodology")` — audit program, methodology standards
- `SearchM365(sources=["files"], query="evidence checklist [process area]")` — evidence requirements for the control area
- `SearchM365(sources=["files"], query="control library [process area]")` — control descriptions, control owners, process narratives
- `SearchM365(sources=["files"], query="prior findings [process area]")` — prior audit reports and remediation tracking
- `ReadFileContent` — read each located document

**Browse evidence library:**
- `GetDriveChildren` — browse the audit evidence library and prior engagement folders for the same process area

**Log every document accessed** — record document name, version, source location, and access timestamp for evidence traceability.

### Step 2: Assemble Audit Evidence Packet

Generate a Word document (invoke `docx` skill) containing:

1. **Engagement Summary**
   - Case ID, Engagement Type, Process Area, Business Unit, Legal Entity
   - Audit Period (start and end dates)
   - Lead Auditor, Audit Manager
   - Engagement objective and scope statement

2. **Control Environment Overview**
   - Control owners identified for the process area (name, role, department)
   - Process narratives — summary of the business process under audit
   - Key controls — list of controls relevant to the scope, with control IDs, descriptions, and owners
   - Control testing approach references from the methodology

3. **Prior Findings and Remediation Status**
   - Prior audit findings for the same process area (finding ID, description, severity, date)
   - Remediation status for each finding (open, in progress, closed, overdue)
   - Prior evidence requests and responses (summary only — no raw evidence content)
   - Flag any findings with overdue remediation

4. **Evidence Requirements Checklist**
   - Each required evidence item with: Item ID, Description, Source (who provides it), Format (document type), Period (what timeframe it covers), Status (not yet requested)
   - Derived from the evidence checklist for this control area
   - Include methodology-specific requirements (e.g., sampling documentation, walkthrough evidence, management representation)

5. **Applicable Methodology Standards**
   - Relevant sections from the audit methodology document
   - Testing approach guidance for the engagement type
   - IIA Standards references applicable to the engagement
   - Document reference and version date for each cited standard

6. **Key Contacts**
   - Lead Auditor, Audit Manager, Control Owners, Process Owners, Business Unit Liaison
   - Include name, role, email, and relationship to the engagement

### Step 3: Generate Evidence Requirements Checklist

Also produce a standalone Excel evidence requirements checklist with columns:
- Item ID, Description, Control Reference, Source Owner, Format, Period, Submission Deadline, Status, Notes

This checklist will be used by audit-gap-detection to track evidence collection progress.

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find audit methodology, evidence checklists, control library, prior findings, prior workpapers |
| ReadFileContent | Read control library, prior audit reports, evidence requirements, audit program |
| GetDriveChildren | Browse audit evidence library and prior engagement folders |
| SearchM365 (connectors) | Audit management platform data via Graph Connector (when available) |
| SearchPeople / GetUserDetails | Resolve control owners, process owners, audit team members |
| GetManagerDetails | Reporting chain for control ownership accountability |

## Guardrails

- **Cite source and date** for every prior finding, control narrative, methodology reference, or evidence requirement included in the packet
- **Never interpret evidence sufficiency or control effectiveness** — present factual context only; the auditor makes all assessment judgments
- **Never include draft audit conclusions or opinions** — this is factual assembly, not analysis
- **Flag missing materials** — if any required methodology document, control narrative, or prior finding is not found in SharePoint, note it explicitly as "not located" with the search terms used
- **Preserve evidence traceability** — log every document accessed with timestamp, source location, document version, and engagement reference
- **Restrict packet access** — note that the generated packet should be stored in the audit team's engagement folder with appropriate SharePoint permissions
- **Never modify submitted evidence** — read and reference only; evidence documents are never altered by this skill
- **Reference IIA Standards** where applicable — cite specific standard numbers when methodology sections align with professional standards
