# Contributing

## Purpose

This repository is a documentation-first library for designing AI-enabled business workflows. Contributions should improve clarity, comparability, and implementation usefulness across the framework and the sample library.

## What Good Contributions Look Like

- Keep the repository process-centric, not chatbot-centric.
- Prefer bounded workflow examples over broad departmental overviews.
- Preserve the common structure used by the existing samples:
  - purpose and context
  - candidate process inventory
  - selected pilot process
  - process definition
  - step-level decomposition
  - signal inventory
  - automation boundary
  - translation to technical artifacts
  - governance model
  - evaluation plan
  - implementation roadmap
- Make governance, approval points, and auditability explicit.
- Keep recommendations practical for large-enterprise line-of-business work.

## Content Guidelines

- Keep changes focused. Avoid broad reformatting or rewriting unrelated sections.
- Reuse existing terminology where possible so the repository stays internally consistent.
- When adding a new sample, choose one bounded pilot workflow with a clear trigger, outcome, owner, and risk profile.
- When updating an existing sample, preserve the document shape unless there is a clear reason to improve the shared pattern.
- Prefer plain Markdown and repository-relative links.

## New Sample Checklist

When adding a new line-of-business sample, include at minimum:

1. A named pilot workflow.
2. A business objective and scope.
3. Step-level decomposition with owners, outputs, and approval requirements.
4. Human signals, system signals, and model knowledge.
5. An automation boundary by step.
6. Skills, tools, and workflow mapping.
7. Governance and evaluation guidance.

## Editing Guidance

- Do not add copyrighted third-party content that should not be redistributed.
- Do not add sensitive, proprietary, or customer-specific data.
- Keep examples generic enough to reuse, but specific enough to be actionable.
- If you add a new sample document, update [README.md](README.md) and [LineOfBusinessSamples/README.md](LineOfBusinessSamples/README.md) as needed.

## Commit Guidance

- Use concise commit messages that describe the content change clearly.
- Group related documentation updates into a single commit when practical.
- Avoid mixing structural cleanup with new content unless both are necessary for the same change.

## Licensing

By contributing to this repository, you agree that your contributions will be licensed under the Apache License 2.0 in [LICENSE](LICENSE).