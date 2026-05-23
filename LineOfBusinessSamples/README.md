# Line Of Business Samples

This folder contains concrete examples of how to apply the Process Decomposition Framework to common enterprise functions. Each sample is centered on one bounded pilot workflow rather than a broad departmental description, so the documents stay useful for actual design and implementation work.

For the core framework, see [../ProcessDecompositionFramework.md](../ProcessDecompositionFramework.md).

## How To Use This Library

1. Start with the function closest to your current work.
2. Use the pilot workflow as a pattern, not a fixed template.
3. Reuse the same sequence in your own domain: process definition, decomposition, signals, automation boundary, skills and tools, governance, evaluation, rollout.
4. Treat the automation boundary and governance sections as mandatory design inputs, not optional add-ons.

## Sample Index

| Function | Sample File | Pilot Workflow |
| --- | --- | --- |
| Finance | [Finance.md](Finance.md) | AP invoice exception handling |
| Finance Controllership | [FinanceControllership.md](FinanceControllership.md) | Manual journal entry request and approval |
| Business Analysis | [BusinessAnalysis.md](BusinessAnalysis.md) | Requirements intake and synthesis |
| HR | [HR.md](HR.md) | Employee onboarding readiness |
| Customer Service | [CustomerService.md](CustomerService.md) | Case intake and resolution triage |
| Product Support | [ProductSupport.md](ProductSupport.md) | Escalated case triage and engineering handoff |
| Procurement | [Procurement.md](Procurement.md) | Supplier onboarding and risk review |
| Sales | [Sales.md](Sales.md) | Proposal and RFP response assembly |
| Product Management | [ProductManagement.md](ProductManagement.md) | Feature request intake and opportunity framing |
| Marketing | [Marketing.md](Marketing.md) | Campaign request intake and brief synthesis |
| Corporate Communications | [CorporateCommunications.md](CorporateCommunications.md) | Announcement request intake and message brief synthesis |
| Legal | [Legal.md](Legal.md) | Contract intake and clause deviation triage |
| Compliance and Risk | [ComplianceRisk.md](ComplianceRisk.md) | Policy exception and compliance case triage |
| Internal Audit | [InternalAudit.md](InternalAudit.md) | Audit request intake and evidence collection prep |
| IT Service Management | [ITServiceManagement.md](ITServiceManagement.md) | Incident intake and triage |
| IT Security | [ITSecurity.md](ITSecurity.md) | Security alert triage and investigation prep |
| Operations | [Operations.md](Operations.md) | Operational exception intake and resolution routing |
| Supply Chain | [SupplyChain.md](SupplyChain.md) | Inventory shortage and disruption triage |
| Field Service | [FieldService.md](FieldService.md) | Work-order triage and dispatch readiness |

## Cross-Sample Comparison

Use this table to compare the samples at a glance. It is intentionally coarse; the detailed step decomposition, controls, and rollout guidance still live in the individual sample documents.

| Function | Pilot Workflow | Dominant Signal Mix | Best First AI Value | Recommended Starting Boundary | Main Human Gate | Primary Governance Concern |
| --- | --- | --- | --- | --- | --- | --- |
| Finance | AP invoice exception handling | invoices, ERP, approvals, email | extract, classify, draft, route | assist plus draft/approve | exception disposition and approvals | payment accuracy and approval control |
| Finance Controllership | Manual journal entry request and approval | support docs, ledger, policy, approvals | gap detection and summary | assist plus draft/approve | journal approval and posting | accounting policy, SoD, auditability |
| Business Analysis | Requirements intake and synthesis | meetings, docs, comments, backlog | extract, synthesize, draft | assist plus draft/approve | baseline and sign-off | traceability and scope control |
| HR | Employee onboarding readiness | employee records, checklists, email, docs | gap detection, routing, draft | assist plus draft/approve | readiness and exception approval | privacy and employee data handling |
| Customer Service | Case intake and resolution triage | customer messages, CRM, case history | classify, route, draft | assist plus draft/approve | escalation and customer commitments | SLA compliance and customer-impact risk |
| Product Support | Escalated case triage and engineering handoff | cases, logs, telemetry, comments | classify, summarize, route | assist plus draft/approve | severity and engineering handoff | evidence quality and customer-impact accuracy |
| Procurement | Supplier onboarding and risk review | supplier docs, vendor master, policy | gap detection, route, draft | assist plus draft/approve | supplier approval path | due diligence, bank data, approval integrity |
| Sales | Proposal and RFP response assembly | RFP docs, CRM, approved content | extract, map, draft | assist plus draft/approve | final submission and pricing approval | approved claims and pricing control |
| Product Management | Feature request intake and opportunity framing | feedback, telemetry, backlog, customer context | clustering, synthesis, draft | assist plus draft/approve | prioritization readiness | evidence traceability and no silent commitments |
| Marketing | Campaign request intake and brief synthesis | requests, brand assets, reviews | extract, draft, route | assist plus draft/approve | brief approval and launch review | approved claims and brand consistency |
| Corporate Communications | Announcement request intake and message brief synthesis | requests, executive input, policy, reviews | synthesize, draft, route | assist plus draft/approve | release approval | message accuracy and release control |
| Legal | Contract intake and clause deviation triage | contracts, playbooks, redlines | deviation detection and summary | assist plus draft/approve | legal triage and approval | privilege, legal judgment, clause policy |
| Compliance and Risk | Policy exception and compliance case triage | policies, evidence, GRC cases | risk indication and summary | assist plus draft/approve | risk disposition and approval | policy interpretation and audit trail |
| Internal Audit | Audit request intake and evidence collection prep | workpapers, evidence, control records | gap detection and summary | assist plus draft/approve | audit review readiness | evidence traceability and independence |
| IT Service Management | Incident intake and triage | tickets, CMDB, monitoring, chat | classify, prioritize, route | assist plus controlled routing | major incident and escalation decisions | production-impacting actions and service risk |
| IT Security | Security alert triage and investigation prep | alerts, telemetry, asset context | enrich, classify, summarize | assist plus draft/approve | containment and incident declaration | evidence integrity and unauthorized response risk |
| Operations | Operational exception intake and resolution routing | work queues, transactions, comments | classify, prioritize, route | assist plus controlled routing | escalation and closure decisions | routing accuracy and audit trail |
| Supply Chain | Inventory shortage and disruption triage | planning data, supplier signals, logistics events | classify, impact, route | assist plus draft/approve | allocation and escalation decisions | customer impact and unsupported allocations |
| Field Service | Work-order triage and dispatch readiness | work orders, asset history, schedule, parts | classify, prep, route | assist plus draft/approve | dispatch-readiness decision | schedule commitments and safety controls |

## Suggested Reading Paths

### Business Workflow And Knowledge Work

- [BusinessAnalysis.md](BusinessAnalysis.md)
- [ProductManagement.md](ProductManagement.md)
- [Marketing.md](Marketing.md)
- [CorporateCommunications.md](CorporateCommunications.md)

### Revenue And Customer Operations

- [Sales.md](Sales.md)
- [CustomerService.md](CustomerService.md)
- [ProductSupport.md](ProductSupport.md)

### Finance, Governance, And Control Functions

- [Finance.md](Finance.md)
- [FinanceControllership.md](FinanceControllership.md)
- [Procurement.md](Procurement.md)
- [Legal.md](Legal.md)
- [ComplianceRisk.md](ComplianceRisk.md)
- [InternalAudit.md](InternalAudit.md)

### Workforce And Operational Execution

- [HR.md](HR.md)
- [Operations.md](Operations.md)
- [SupplyChain.md](SupplyChain.md)
- [FieldService.md](FieldService.md)
- [ITServiceManagement.md](ITServiceManagement.md)
- [ITSecurity.md](ITSecurity.md)

## Design Pattern To Reuse

Across these samples, the recurring pattern is:

1. Choose one bounded, high-value workflow.
2. Decompose it into steps with one dominant goal and decision type.
3. Separate human signals, system signals, and model knowledge.
4. Assign an operating mode to each step.
5. Translate only the right steps into skills, tools, workflows, or approvals.
6. Keep governance, evaluation, and rollout in the design from the start.
