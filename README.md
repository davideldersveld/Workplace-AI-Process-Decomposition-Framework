# Workplace AI Process Decomposition Library

This repository captures a practical framework for turning line-of-business and industry workflows into AI-enabled operating capabilities. It is designed for business leaders, process owners, architects, and engineering teams who need a structured way to move from workflow analysis to governed implementation.

The repository is organized around three major areas:

- the core framework for process decomposition, automation boundaries, skills and tools mapping, governance, evaluation, and rollout
- a line-of-business sample library that applies the framework to enterprise functions
- a vertical sample library that applies the framework to industry-specific workflows and Copilot Cowork assets

## Quick Navigation

- [Core Framework](ProcessDecompositionFramework.md)
- [Line Of Business Index](LineOfBusinessSamples/README.md)
- [Vertical Index](VerticalSamples/README.md)
- [Line Of Business Plugin Plans](LineOfBusinessSamples/CopilotCoworkSamplePluginPlans)
- [Line Of Business Skill Assets](LineOfBusinessSamples/CopilotCoworkSampleSkills)
- [Vertical Plugin Plans](VerticalSamples/CopilotCoworkSamplePluginPlans)
- [Vertical Skill Assets](VerticalSamples/CopilotCoworkSampleSkills)

## Repository Structure

```text
MainPlan/
|-- ProcessDecompositionFramework.md
|-- LineOfBusinessSamples/
|   |-- README.md
|   |-- *.md
|   |-- CopilotCoworkSamplePluginPlans/
|   `-- CopilotCoworkSampleSkills/
|-- VerticalSamples/
|   |-- README.md
|   |-- Insurance.md
|   |-- Banking.md
|   |-- CapitalMarkets.md
|   |-- CopilotCoworkSamplePluginPlans/
|   `-- CopilotCoworkSampleSkills/
|-- LICENSE
`-- CONTRIBUTING.md
```

## Repository Contents

| Path | Purpose |
| --- | --- |
| [ProcessDecompositionFramework.md](ProcessDecompositionFramework.md) | Core framework for process decomposition, reference architecture, governance, evaluation, and implementation guidance |
| [LineOfBusinessSamples/README.md](LineOfBusinessSamples/README.md) | Index of the line-of-business sample library, comparison matrix, and Copilot Cowork assets |
| [LineOfBusinessSamples](LineOfBusinessSamples) | Function-specific examples showing how to apply the framework to bounded pilot workflows |
| [LineOfBusinessSamples/CopilotCoworkSamplePluginPlans](LineOfBusinessSamples/CopilotCoworkSamplePluginPlans) | Copilot Cowork plugin planning documents for line-of-business workflows |
| [LineOfBusinessSamples/CopilotCoworkSampleSkills](LineOfBusinessSamples/CopilotCoworkSampleSkills) | Copilot Cowork sample skill assets for line-of-business workflows |
| [VerticalSamples/README.md](VerticalSamples/README.md) | Index of the vertical sample library, comparison view, and Copilot Cowork assets |
| [VerticalSamples](VerticalSamples) | Industry-specific examples showing how to apply the framework at the vertical level |
| [VerticalSamples/CopilotCoworkSamplePluginPlans](VerticalSamples/CopilotCoworkSamplePluginPlans) | Copilot Cowork plugin planning documents for industry vertical workflows |
| [VerticalSamples/CopilotCoworkSampleSkills](VerticalSamples/CopilotCoworkSampleSkills) | Copilot Cowork sample skill assets for industry vertical workflows |
| [LICENSE](LICENSE) | Repository license under Apache License 2.0 |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Contribution expectations for framework and sample-library changes |

## What This Is For

Use this repository when you need to:

- break a business workflow into steps, decisions, inputs, tools, and approval points
- identify where AI should assist, draft, act within policy, or stay out entirely
- translate process steps into skills, tools, workflows, agents, policies, and human gates
- design governance, auditability, and evaluation into the process from the start
- compare how the same framework applies across different business functions and industries
- derive Copilot Cowork-oriented plugin and skill designs from process decomposition outputs

## How To Read It

1. Start with [ProcessDecompositionFramework.md](ProcessDecompositionFramework.md).
2. Move to [LineOfBusinessSamples/README.md](LineOfBusinessSamples/README.md) to browse the function-specific sample library.
3. Use [VerticalSamples/README.md](VerticalSamples/README.md) when you want an industry-specific overlay across multiple business functions.
4. If you are implementing in Microsoft 365 Copilot Cowork, review the plugin planning folders under the relevant sample library.
5. Use the skill folders when you want concrete `SKILL.md` artifacts derived from those plans.
6. Pair a vertical sample with the closest line-of-business sample when you need both workflow shape and domain-specific controls.
7. Adapt the decomposition, automation boundary, governance, and evaluation sections to your own process.

## Repository Governance

- License: [Apache License 2.0](LICENSE)
- Contribution guidance: [CONTRIBUTING.md](CONTRIBUTING.md)

## Current Sample Coverage

The line-of-business sample library currently includes examples for:

- Finance
- Finance Controllership
- Business Analysis
- HR
- Customer Service
- Product Support
- Procurement
- Sales
- Product Management
- Marketing
- Corporate Communications
- Legal
- Compliance and Risk
- Internal Audit
- IT Service Management
- IT Security
- Operations
- Supply Chain
- Field Service

The vertical sample library currently includes examples for:

- Insurance
- Banking
- Capital Markets

## Copilot Cowork Assets

The repository now includes Copilot Cowork planning and skill artifacts in both sample libraries.

- `CopilotCoworkSamplePluginPlans` contains planning documents that translate decomposed workflows into Cowork-oriented skill suites.
- `CopilotCoworkSampleSkills` contains imported and normalized sample skill assets.
- Each skill lives in its own subfolder with a `SKILL.md` file.
- Summary markdown files remain at the root of each domain folder for easier browsing and comparison.

Current repo footprint:

- 22 Copilot Cowork plugin planning documents
- 22 Copilot Cowork skill domains
- 19 function-oriented samples
- 3 industry vertical samples

## Recent Updates

- Added a vertical industry sample library for Insurance, Banking, and Capital Markets.
- Added dedicated index files for the line-of-business and vertical sample libraries.
- Added Copilot Cowork plugin planning assets for both line-of-business and vertical sample sets.
- Added Copilot Cowork skill assets and normalized them into per-skill folders with `SKILL.md` files.

## Design Principle

The recurring design pattern across the repository is simple:

1. Choose one bounded, high-value workflow.
2. Decompose it until each step has one dominant goal and one dominant decision type.
3. Separate human signals, system signals, and model knowledge.
4. Assign the correct operating mode to each step.
5. Translate only the right steps into skills, tools, workflows, or approvals.
6. Build governance, observability, and rollout into the design from the start.
