# Line Of Business Samples

This folder contains concrete examples of how to apply the Process Decomposition Framework to common enterprise functions. Each sample is centered on one bounded pilot workflow rather than a broad departmental description, so the documents stay useful for actual design and implementation work.

This library now has three practical layers for each function:

1. a narrative sample describing the workflow and decomposition
2. a Copilot Cowork plugin plan showing how the workflow maps to skills and tools
3. a Copilot Cowork skill folder containing concrete `SKILL.md` artifacts

For the core framework, see [../ProcessDecompositionFramework.md](../ProcessDecompositionFramework.md).

Current scope in this folder:

- 19 line-of-business sample documents
- 19 Copilot Cowork plugin planning documents
- 19 Copilot Cowork skill domains

## How To Use This Library

1. Start with the function closest to your current work.
2. Read the sample markdown first to understand the bounded pilot workflow and decomposition logic.
3. If you are implementing in Copilot Cowork, open the matching plugin plan under [CopilotCoworkSamplePluginPlans](CopilotCoworkSamplePluginPlans/) next.
4. Then inspect the corresponding folder under [CopilotCoworkSampleSkills](CopilotCoworkSampleSkills/) for concrete `SKILL.md` assets and domain-specific summaries.
5. Use the pilot workflow as a pattern, not a fixed template.
6. Reuse the same sequence in your own domain: process definition, decomposition, signals, automation boundary, skills and tools, governance, evaluation, rollout.
7. Treat the automation boundary and governance sections as mandatory design inputs, not optional add-ons.

## Folder Structure

```text
LineOfBusinessSamples/
|-- README.md
|-- *.md
|-- CopilotCoworkSamplePluginPlans/
|   `-- *-Cowork-Plugin-Plan.md
`-- CopilotCoworkSampleSkills/
	`-- <Function>/
		|-- <domain summary>.md
		`-- <skill-name>/SKILL.md
```

The sample markdown files explain the workflow design. The plugin plans explain how that workflow translates into Cowork concepts. The skill folders contain the most implementation-ready artifacts.

## Sample Index

| Function | Sample File | Pilot Workflow | Plugin Plan | Skill Assets |
| --- | --- | --- | --- | --- |
| Finance | [Finance.md](Finance.md) | AP invoice exception handling | [Finance-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Finance-Cowork-Plugin-Plan.md) | [Finance/](CopilotCoworkSampleSkills/Finance/) |
| Finance Controllership | [FinanceControllership.md](FinanceControllership.md) | Manual journal entry request and approval | [Finance-Controllership-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Finance-Controllership-Cowork-Plugin-Plan.md) | [FinanceControllership/](CopilotCoworkSampleSkills/FinanceControllership/) |
| Business Analysis | [BusinessAnalysis.md](BusinessAnalysis.md) | Requirements intake and synthesis | [Business-Analysis-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Business-Analysis-Cowork-Plugin-Plan.md) | [BusinessAnalysis/](CopilotCoworkSampleSkills/BusinessAnalysis/) |
| HR | [HR.md](HR.md) | Employee onboarding readiness | [HR-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/HR-Cowork-Plugin-Plan.md) | [HR/](CopilotCoworkSampleSkills/HR/) |
| Customer Service | [CustomerService.md](CustomerService.md) | Case intake and resolution triage | [Customer-Service-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Customer-Service-Cowork-Plugin-Plan.md) | [CustomerService/](CopilotCoworkSampleSkills/CustomerService/) |
| Product Support | [ProductSupport.md](ProductSupport.md) | Escalated case triage and engineering handoff | [Product-Support-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Product-Support-Cowork-Plugin-Plan.md) | [ProductSupport/](CopilotCoworkSampleSkills/ProductSupport/) |
| Procurement | [Procurement.md](Procurement.md) | Supplier onboarding and risk review | [Procurement-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Procurement-Cowork-Plugin-Plan.md) | [Procurement/](CopilotCoworkSampleSkills/Procurement/) |
| Sales | [Sales.md](Sales.md) | Proposal and RFP response assembly | [Sales-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Sales-Cowork-Plugin-Plan.md) | [Sales/](CopilotCoworkSampleSkills/Sales/) |
| Product Management | [ProductManagement.md](ProductManagement.md) | Feature request intake and opportunity framing | [Product-Management-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Product-Management-Cowork-Plugin-Plan.md) | [ProductManagement/](CopilotCoworkSampleSkills/ProductManagement/) |
| Marketing | [Marketing.md](Marketing.md) | Campaign request intake and brief synthesis | [Marketing-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Marketing-Cowork-Plugin-Plan.md) | [Marketing/](CopilotCoworkSampleSkills/Marketing/) |
| Corporate Communications | [CorporateCommunications.md](CorporateCommunications.md) | Announcement request intake and message brief synthesis | [Corporate-Communications-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Corporate-Communications-Cowork-Plugin-Plan.md) | [CorporateCommunications/](CopilotCoworkSampleSkills/CorporateCommunications/) |
| Legal | [Legal.md](Legal.md) | Contract intake and clause deviation triage | [Legal-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Legal-Cowork-Plugin-Plan.md) | [Legal/](CopilotCoworkSampleSkills/Legal/) |
| Compliance and Risk | [ComplianceRisk.md](ComplianceRisk.md) | Policy exception and compliance case triage | [Compliance-Risk-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Compliance-Risk-Cowork-Plugin-Plan.md) | [ComplianceRisk/](CopilotCoworkSampleSkills/ComplianceRisk/) |
| Internal Audit | [InternalAudit.md](InternalAudit.md) | Audit request intake and evidence collection prep | [Internal-Audit-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Internal-Audit-Cowork-Plugin-Plan.md) | [InternalAudit/](CopilotCoworkSampleSkills/InternalAudit/) |
| IT Service Management | [ITServiceManagement.md](ITServiceManagement.md) | Incident intake and triage | [IT-Service-Management-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/IT-Service-Management-Cowork-Plugin-Plan.md) | [ITServiceManagement/](CopilotCoworkSampleSkills/ITServiceManagement/) |
| IT Security | [ITSecurity.md](ITSecurity.md) | Security alert triage and investigation prep | [IT-Security-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/IT-Security-Cowork-Plugin-Plan.md) | [ITSecurity/](CopilotCoworkSampleSkills/ITSecurity/) |
| Operations | [Operations.md](Operations.md) | Operational exception intake and resolution routing | [Operations-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Operations-Cowork-Plugin-Plan.md) | [Operations/](CopilotCoworkSampleSkills/Operations/) |
| Supply Chain | [SupplyChain.md](SupplyChain.md) | Inventory shortage and disruption triage | [Supply-Chain-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Supply-Chain-Cowork-Plugin-Plan.md) | [SupplyChain/](CopilotCoworkSampleSkills/SupplyChain/) |
| Field Service | [FieldService.md](FieldService.md) | Work-order triage and dispatch readiness | [Field-Service-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Field-Service-Cowork-Plugin-Plan.md) | [FieldService/](CopilotCoworkSampleSkills/FieldService/) |

## Copilot Cowork Assets

The Cowork assets under this folder are useful for two different audiences:

- `CopilotCoworkSamplePluginPlans` is best for architects and process designers who want to see how the framework translates into skill suites, operating modes, tool choices, and governance.
- `CopilotCoworkSampleSkills` is best for implementers who want concrete `SKILL.md` artifacts, domain summaries, and folder structures they can adapt.

Typical reading order for one function:

1. open the sample markdown to understand the workflow and decomposition
2. open the matching plugin plan to see the proposed Cowork design
3. open the matching skill folder to inspect concrete skill artifacts
4. adapt the same pattern to your own process, data sources, and governance rules

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
