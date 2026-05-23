# Industry Vertical Samples

This folder contains industry-oriented examples of how to apply the Workplace AI Process Decomposition Framework in regulated and operationally complex environments. Each vertical sample translates the same core framework into an industry context with its own controls, data landscape, operating risks, and implementation constraints.

This library has three practical layers for each vertical:

1. a narrative sample describing the industry context, candidate processes, and selected pilot workflow
2. a Copilot Cowork plugin plan showing how the workflow maps to skills, tools, guardrails, and rollout waves
3. a Copilot Cowork skill folder containing concrete `SKILL.md` artifacts and domain summaries

For the core framework, see [../ProcessDecompositionFramework.md](../ProcessDecompositionFramework.md).

Current scope in this folder:

- 3 vertical sample documents
- 3 Copilot Cowork plugin planning documents
- 3 Copilot Cowork skill domains

## How To Use This Library

1. Start with the industry closest to your operating environment.
2. Read the vertical sample markdown first to understand the domain context, candidate process inventory, and selected pilot workflow.
3. If you are implementing in Copilot Cowork, open the matching plugin plan under [CopilotCoworkSamplePluginPlans](CopilotCoworkSamplePluginPlans/) next.
4. Then inspect the corresponding folder under [CopilotCoworkSampleSkills](CopilotCoworkSampleSkills/) for concrete `SKILL.md` artifacts and domain-specific summaries.
5. Use the vertical sample as an overlay on top of the core framework and any relevant line-of-business sample.
6. Reuse the same sequence in your own environment: process selection, decomposition, signal inventory, automation boundary, capability mapping, governance, evaluation, and rollout.
7. Treat the regulatory and governance sections as mandatory design inputs, not optional commentary.

## Folder Structure

```text
VerticalSamples/
|-- README.md
|-- Banking.md
|-- Insurance.md
|-- CapitalMarkets.md
|-- CopilotCoworkSamplePluginPlans/
|   `-- *-Cowork-Plugin-Plan.md
`-- CopilotCoworkSampleSkills/
    `-- <Vertical>/
        |-- <domain summary>.md
        `-- <skill-name>/SKILL.md
```

The sample markdown files explain the industry context and pilot workflow choice. The plugin plans explain how that workflow translates into Cowork concepts. The skill folders contain the most implementation-ready artifacts.

## Sample Index

| Vertical | Sample File | Selected Pilot Workflow | Plugin Plan | Skill Assets |
| --- | --- | --- | --- | --- |
| Banking | [Banking.md](Banking.md) | Fraud alert and dispute triage | [Banking-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Banking-Cowork-Plugin-Plan.md) | [Banking/](CopilotCoworkSampleSkills/Banking/) |
| Insurance | [Insurance.md](Insurance.md) | FNOL and coverage triage | [Insurance-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Insurance-Cowork-Plugin-Plan.md) | [Insurance/](CopilotCoworkSampleSkills/Insurance/) |
| Capital Markets | [CapitalMarkets.md](CapitalMarkets.md) | Trade exception and settlement break triage | [Capital-Markets-Cowork-Plugin-Plan.md](CopilotCoworkSamplePluginPlans/Capital-Markets-Cowork-Plugin-Plan.md) | [CapitalMarkets/](CopilotCoworkSampleSkills/CapitalMarkets/) |

## Copilot Cowork Assets

The Cowork assets under this folder are useful for two different audiences:

- `CopilotCoworkSamplePluginPlans` is best for architects, operating model owners, and governance teams who want to see how the framework translates into skill suites, operating modes, tool choices, and rollout plans for a regulated vertical.
- `CopilotCoworkSampleSkills` is best for implementers who want concrete `SKILL.md` artifacts, domain summaries, and folder structures they can adapt.

Typical reading order for one vertical:

1. open the vertical sample markdown to understand the industry context and chosen pilot process
2. open the matching plugin plan to see the proposed Cowork design and phased rollout
3. open the matching skill folder to inspect concrete skill artifacts
4. adapt the same pattern to your own systems, data sources, controls, and regulatory obligations

## Cross-Vertical Comparison

Use this table to compare the verticals at a glance. It is intentionally coarse; the detailed controls, system assumptions, and implementation guidance live in the individual sample documents and plugin plans.

| Vertical | Pilot Workflow | Dominant Signal Mix | Best First AI Value | Recommended Starting Boundary | Main Human Gate | Primary Governance Concern |
| --- | --- | --- | --- | --- | --- | --- |
| Banking | Fraud alert and dispute triage | transactions, fraud alerts, customer reports, evidence checklists, regulatory timelines | normalize, enrich, classify, route, draft | assist plus draft/approve | fraud or dispute disposition and account actions | Reg E compliance, AML escalation, customer data protection |
| Insurance | FNOL and coverage triage | loss narratives, policy docs, claimant evidence, prior claims, jurisdiction rules | normalize, assemble evidence, classify, draft | assist plus draft/approve | coverage determination and claim authority | unfair claims practices compliance, claimant privacy, SIU controls |
| Capital Markets | Trade exception and settlement break triage | trade records, settlement status, SSI data, counterparty notices, cutoff schedules | enrich, classify, risk assess, route | assist plus draft/approve | booking or settlement action and final break disposition | T+1 settlement compliance, information barriers, MNPI protection |

## Suggested Reading Paths

### Highly Regulated Customer Casework

- [Banking.md](Banking.md)
- [Insurance.md](Insurance.md)

### Time-Critical Market And Operations Control

- [CapitalMarkets.md](CapitalMarkets.md)

### For M365 Copilot Cowork Implementation

1. read the vertical sample for domain context and process choice
2. read the plugin plan to understand the proposed skills, tools, guardrails, and rollout waves
3. inspect the skill folders for the concrete `SKILL.md` implementations
4. pair the vertical sample with the closest line-of-business sample if you need more function-level detail

## What The Vertical Samples Add

The line-of-business samples explain workflow design within a function. The vertical samples add the industry constraints that often matter more than the workflow itself:

- regulatory timelines and prohibited actions
- external system dependencies and connector strategy
- data sensitivity and masking rules
- audit and evidence requirements
- role segregation and approval authority
- vertical-specific risk patterns and escalation paths

In practice, the vertical sample is the domain-control overlay you apply to the more general line-of-business workflow pattern.

## Design Pattern To Reuse

Across these verticals, the recurring pattern is:

1. choose one bounded, high-value workflow with strong operational pressure and clear controls
2. decompose it into steps with one dominant goal and decision type
3. separate human signals, system signals, and model knowledge
4. assign the correct operating mode to each step
5. convert durable process state into M365 artifacts such as Excel trackers, SharePoint folders, and Word evidence packets
6. encode prohibitions explicitly so AI prepares and recommends, but humans decide and authorize
7. build governance, evaluation, and rollout into the design from the start

## When To Prefer A Vertical Sample

Use a vertical sample before a function sample when your main design risk is driven by industry constraints such as:

- regulatory deadlines or statutory notices
- sensitive financial, policy, or trading data
- information barrier or segregation-of-duties requirements
- external counterparties, claimants, or banking customers
- auditable case files and examination-grade traceability

If your main design risk is workflow shape rather than industry control, start with the line-of-business sample first and then layer the vertical pattern on top.