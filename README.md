# Workplace AI Process Decomposition Library

This repository captures a practical framework for turning line-of-business workflows into AI-enabled operating capabilities. It is designed for business leaders, process owners, architects, and engineering teams who need a structured way to move from workflow analysis to governed implementation.

The repository is organized around two layers:

- the core framework for process decomposition, automation boundaries, skills and tools mapping, governance, evaluation, and rollout
- a growing sample library that applies the framework to specific enterprise functions

## Repository Contents

| Path | Purpose |
| --- | --- |
| [ProcessDecompositionFramework.md](ProcessDecompositionFramework.md) | Core framework for process decomposition, reference architecture, governance, evaluation, and implementation guidance |
| [LineOfBusinessSamples/README.md](LineOfBusinessSamples/README.md) | Index of the sample library and cross-sample comparison matrix |
| [LineOfBusinessSamples](LineOfBusinessSamples) | Function-specific examples showing how to apply the framework to bounded pilot workflows |

## What This Is For

Use this repository when you need to:

- break a business workflow into steps, decisions, inputs, tools, and approval points
- identify where AI should assist, draft, act within policy, or stay out entirely
- translate process steps into skills, tools, workflows, agents, policies, and human gates
- design governance, auditability, and evaluation into the process from the start
- compare how the same framework applies across different business functions

## How To Read It

1. Start with [ProcessDecompositionFramework.md](ProcessDecompositionFramework.md).
2. Move to [LineOfBusinessSamples/README.md](LineOfBusinessSamples/README.md) to browse the available function-specific examples.
3. Open the sample closest to your domain and use its pilot workflow as a pattern.
4. Adapt the decomposition, automation boundary, governance, and evaluation sections to your own process.

## Current Sample Coverage

The sample library currently includes examples for:

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

## Design Principle

The recurring design pattern across the repository is simple:

1. Choose one bounded, high-value workflow.
2. Decompose it until each step has one dominant goal and one dominant decision type.
3. Separate human signals, system signals, and model knowledge.
4. Assign the correct operating mode to each step.
5. Translate only the right steps into skills, tools, workflows, or approvals.
6. Build governance, observability, and rollout into the design from the start.

## Suggested Next Additions

Natural next additions for the repository would be reusable templates, scoring sheets for pilot selection, and additional function-specific samples such as Facilities, Revenue Operations, Corporate Strategy, or Learning and Development.