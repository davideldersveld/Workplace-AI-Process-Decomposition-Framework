# Plan: HR Onboarding Readiness — Copilot Cowork Plugin

## Overview

This document defines a plan for building a sample Copilot Cowork plugin for the HR line of business, based on the Workplace AI Process Decomposition Framework. It covers the evaluation process used to map the framework to Cowork skill ideation, detailed skill designs for the HR Onboarding Readiness pilot, and analysis on how to generalize the framework for creating Copilot Cowork Skills in the Microsoft 365 ecosystem.

---

## Part 1: Evaluation Process — Mapping the Framework to Skill Ideation

The Process Decomposition Framework defines a seven-phase methodology (Select → Decompose → Signal Inventory → Automation Boundary → Capability Map → Translate → Architect). To evaluate how this maps to Copilot Cowork skill ideation, each phase was assessed for where framework concepts have direct Cowork equivalents, where they require adaptation, and where they surface gaps that need bridging.

### Phase-by-Phase Mapping Evaluation

| Framework Phase | What It Produces | Cowork Equivalent | Translation Quality |
|---|---|---|---|
| **Process Selection** | Prioritized candidate process | Choice of which skill *suite* to build | Direct — the scoring criteria (volume, data readiness, error tolerance) map cleanly to "will this skill get used and work well?" |
| **Process Decomposition** | Step records (YAML) with goal, trigger, inputs, outputs, owner | Individual **SKILL.md files** — each step with one dominant goal becomes one skill | Strong — the "one dominant goal, one decision type" rule maps perfectly to Cowork's principle that each skill should have narrow scope |
| **Signal Inventory** | Human signals, system signals, model knowledge | **MCP tool selection** — which Outlook, Teams, Graph, and SharePoint tools to call, plus which M365 data sources to read | Requires translation — the framework uses generic terms ("CRM", "ticketing system") that must become specific M365 tool names |
| **Automation Boundary** | Operating mode per step (human-only → deterministic) | **Guardrails and confirmation gates** in SKILL.md — Cowork's "present draft before sending" pattern maps to "AI draft plus approve" | Strong — but Cowork enforces this through instructions and approval dialogs, not a runtime policy engine |
| **Capability Mapping** | AI task pattern per step (extract, classify, summarize, generate, etc.) | **Skill template selection** — Data Aggregation, Content Generation, or Decision Support | Direct — the 10 task patterns map to 3 Cowork skill templates with some combination |
| **Technical Translation** | Skill contracts, tool contracts, workflow definitions | **SKILL.md** (skill), **MCP tool calls** (tools), **multi-skill orchestration** (workflow) | Good but simplified — Cowork does not have formal "tool contracts" or a workflow engine; orchestration is implicit in skill instructions |
| **Reference Architecture** | 9-layer runtime (intake, process model, registry, orchestrator, etc.) | **Cowork's built-in runtime** — the session, MCP servers, skill routing, memory, and tool execution ARE the runtime | Absorbed — Cowork provides layers 1–5 and 7–9 natively; what the skill author controls is capability definition (layer 3) and decision logic (layer 6) |

### Key Insight

The framework is designed for custom-built systems where you control orchestration, state management, and runtime. Cowork provides all of that as platform infrastructure. The skill author's job narrows to three things:

1. **Define the capability** — what the skill does, what it reads, what it produces
2. **Set the boundaries** — when to trigger, when NOT to trigger, what requires human review
3. **Specify the M365 grounding** — which tools to call, which data sources to read, which artifact formats to produce

Everything else — signal intake, context assembly, policy enforcement, observability — is handled by the Cowork platform.

---

## Part 2: The HR Onboarding Readiness Plugin — Skill-by-Skill Design

The HR sample decomposes "Employee Onboarding Readiness" into 7 steps (HR-ONB-001 through HR-ONB-007), identifies 6 skills and 7 tools, and assigns automation boundaries. Below is how each maps to a concrete Cowork skill, including which M365 artifacts and tools it touches.

### Skill Suite Overview

| Framework Step | Cowork Skill Name | Template Type | Operating Mode | Primary M365 Artifacts |
|---|---|---|---|---|
| HR-ONB-001: Normalize onboarding event | `hr-onboarding-intake` | Data Aggregation | Deterministic automation | Excel (tracker), SharePoint (list), Outlook (notifications) |
| HR-ONB-002: Gather readiness context | `hr-readiness-packet` | Data Aggregation + Content Generation | AI act within policy | Word (packet), SharePoint (policies), Graph API (people) |
| HR-ONB-003: Identify missing items and risks | `hr-gap-detection` | Decision Support | AI assist | Excel (checklist), SharePoint (documents), Adaptive Card (report) |
| HR-ONB-004: Assign owners and next actions | `hr-task-routing` | Decision Support | AI draft + approve | Teams (messages), Graph API (org hierarchy), Planner (tasks) |
| HR-ONB-005: Draft outreach and reminders | `hr-onboarding-comms` | Content Generation | AI draft + approve | Outlook (drafts), Teams (messages), Word (templates) |
| HR-ONB-006: Prepare readiness summary | `hr-readiness-summary` | Data Aggregation + Content Generation | AI draft + approve | PowerPoint (review deck), Word (summary), Adaptive Card |
| HR-ONB-007: Confirm onboarding readiness | *Not a skill — human approval step* | N/A | Human only | Calendar (review meeting), Outlook (sign-off email) |

### Detailed Skill Designs

#### 1. `hr-onboarding-intake` — Normalize Onboarding Event

**Framework Step:** HR-ONB-001

**Trigger phrases:** "new hire starting", "onboarding case for [name]", "set up onboarding for"

**Inputs:**
- New hire name or email
- Start date
- Role and location
- Hiring manager

**M365 tools:**
- `SearchPeople` — resolve new hire and manager identities
- `GetUserDetails` — pull profile data
- `SearchM365(sources=["email"])` — find offer letter thread for context
- SharePoint list read — check existing tracker for duplicates

**Output:** Structured case record written to Excel onboarding tracker in SharePoint; confirmation via Adaptive Card

**Artifact:** Excel workbook with columns: Case ID, Employee Name, Start Date, Role, Location, Manager, Status, Created Date, Assigned To

**Guardrails:**
- Never create duplicate cases for the same employee and start date
- Validate that start date is in the future
- Confirm details with user before writing to tracker

---

#### 2. `hr-readiness-packet` — Gather Readiness Context

**Framework Step:** HR-ONB-002

**Trigger phrases:** "build readiness packet", "assemble onboarding context for", "what do we need for [name]'s onboarding"

**Inputs:**
- Onboarding case ID or employee name

**M365 tools:**
- `GetUserDetails` — employee profile
- `GetManagerDetails` — reporting chain
- `SearchM365(sources=["files"])` — policies, checklists, location-specific guides
- `ReadFileContent` — SharePoint policy documents

**Output:** Word document containing:
- Employee profile summary
- Role and location details
- Applicable policy extracts
- Required document checklist
- Manager and HRBP contacts

**Artifact:** Word (.docx) saved to SharePoint onboarding folder; the `docx` skill handles generation

**Guardrails:**
- Mask SSN and other sensitive PII in generated documents
- Cite policy source and version for every extract
- Flag if any required policy document is unfindable in SharePoint

---

#### 3. `hr-gap-detection` — Identify Missing Items and Risks

**Framework Step:** HR-ONB-003

**Trigger phrases:** "check onboarding gaps", "what's missing for [name]", "onboarding readiness check", "audit onboarding case"

**Inputs:**
- Readiness packet (Word doc or case data from Excel tracker)

**M365 tools:**
- `ReadFileContent` — checklist from SharePoint
- `SearchM365(sources=["files"])` — uploaded employee documents
- `GetDriveChildren` — onboarding folder contents to check what has been uploaded

**Output:** Gap report as Adaptive Card (for quick review) plus Excel worksheet update (for tracking)

**Logic:** Compare required checklist items against uploaded documents in the employee's onboarding folder. Flag missing items, approaching deadlines, and policy-sensitive conditions such as work authorization gaps or privileged access requests.

**Guardrails:**
- Never mark an item as complete without document evidence in the SharePoint folder
- Flag privacy-sensitive gaps (work authorization, background checks) separately with elevated visibility
- Present findings for user review before updating the Excel tracker

---

#### 4. `hr-task-routing` — Assign Owners and Next Actions

**Framework Step:** HR-ONB-004

**Trigger phrases:** "assign onboarding tasks", "route onboarding work", "who handles [task] for this hire"

**Inputs:**
- Gap report output
- Onboarding case data
- Responsibility matrix (SharePoint document)

**M365 tools:**
- `SearchPeople` — resolve role owners by name or function
- `GetManagerDetails` / `GetDirectReportsDetails` — org structure
- `PostMessage` — Teams notification to assignees
- `CreateEvent` — optional deadline reminders on calendars

**Output:** Routed work items:
- Teams messages to responsible parties with task details
- Calendar holds for key deadlines
- Updated Excel tracker with owner assignments per task

**Guardrails:**
- Present routing recommendations for HR specialist review before sending any messages
- Never auto-assign to someone outside the onboarding responsibility matrix
- Escalate to the HR operations manager if no clear owner is found for a task

---

#### 5. `hr-onboarding-comms` — Draft Outreach and Reminders

**Framework Step:** HR-ONB-005

**Trigger phrases:** "draft onboarding email for", "remind [manager] about onboarding", "send welcome message to new hire", "onboarding follow-up"

**Inputs:**
- Case data from Excel tracker
- Readiness packet (Word document)
- Gap report
- Target audience (employee, manager, HRBP, IT)

**M365 tools:**
- `CreateDraftMessage` — Outlook drafts (never auto-send)
- `PostMessage` — Teams coordination messages
- `SearchM365(sources=["files"])` — email templates from SharePoint

**Communication templates:**
- Welcome email to new hire
- Missing documents reminder to employee
- Readiness update to hiring manager
- IT provisioning request
- HRBP notification

**Guardrails:**
- Always create as Outlook draft — never send without explicit user confirmation
- Match tone to audience: formal for employee-facing, operational for IT requests
- Include onboarding case reference number in every communication
- Respect the framework's "AI draft plus approve" boundary — every output is reviewable

---

#### 6. `hr-readiness-summary` — Prepare Readiness Summary

**Framework Step:** HR-ONB-006

**Trigger phrases:** "summarize onboarding status", "readiness review for [name]", "onboarding deck for manager review", "prepare readiness report"

**Inputs:**
- All prior artifacts: case record (Excel), readiness packet (Word), gap report, task assignments, communication history

**M365 tools:**
- `ReadFileContent` — all case artifacts from SharePoint
- `SearchM365(sources=["email"])` — recent correspondence thread
- Document generation skills: `docx`, `pptx`, `xlsx`

**Output options:**
- **Adaptive Card** — Quick status view for chat-based review
- **Word document** — Formal readiness summary for sign-off
- **PowerPoint deck** — Manager-facing review presentation with status, blockers, required approvals, and timeline
- **Excel update** — Tracker status set to "Ready for Review" or "Blocked"

**Guardrails:**
- Every finding must trace to a source artifact — no fabricated status information
- Clearly distinguish complete vs. incomplete vs. blocked items in all output formats
- Flag any items requiring exception approval with explicit callouts
- Present summary for review before finalizing

---

#### Step 7: Confirm Onboarding Readiness (Human Only)

**Framework Step:** HR-ONB-007

This is not a Cowork skill. The framework correctly identifies final readiness confirmation as a human-only step. In Cowork, it is supported by:

- The `schedule-meeting` skill — book the readiness review meeting
- The readiness summary artifacts — provide the evidence package for the reviewer
- The `hr-onboarding-comms` skill — send the sign-off confirmation email after the human decision is made

---

## Part 3: Analysis — Bridging the General Framework to Copilot Cowork in M365

The core challenge is that the framework is **platform-agnostic and architecture-heavy**, while Cowork is **M365-native and runtime-provided**. This section analyzes how to approach that translation systematically.

### 3.1 What the Framework Provides That Cowork Needs

The framework's greatest value to Cowork skill authors is **pre-implementation discipline**:

**Process decomposition prevents mega-skills.** The common Cowork anti-pattern is building one broad skill that tries to handle an entire domain. The framework's rule — "keep breaking down until each step has one dominant goal" — directly produces well-scoped skills that score high on the quality rubric's Scope Boundaries dimension (0–25 scale).

**Automation boundary assignment maps to guardrails architecture.** The five operating modes translate directly:

| Framework Mode | Cowork Implementation |
|---|---|
| Human only | Do not build a skill; support with `meeting-intel` or `daily-briefing` |
| AI assist | Skill reads and analyzes but only presents via Adaptive Card; no write actions |
| AI draft + approve | Skill produces artifacts but uses `CreateDraftMessage`, shows output before writing, requires explicit confirmation |
| AI act within policy | Skill can execute bounded write actions (update tracker, post to channel) within defined rules |
| Deterministic automation | Scheduled prompt or rule-based skill with no model judgment needed |

**Signal inventory forces explicit M365 tool selection.** Instead of vague instructions like "gather relevant context," the framework requires naming every input source. This translates to specific MCP tool calls in the SKILL.md instructions — a direct quality improvement.

### 3.2 What Cowork Provides That the Framework Assumes You Build

The framework describes a 9-layer reference architecture. In Cowork, most of these layers are platform-provided:

| Framework Layer | Cowork Provides It As |
|---|---|
| Signal intake and normalization | Built-in — email, Teams, calendar, and files are all accessible via MCP tools |
| Process model | Implicit — the skill's trigger phrases and instructions define which "process step" is active |
| Capability registry | Built-in — `/mnt/user-config/.claude/skills/` IS the registry; skills are discovered and versioned |
| Runtime orchestrator | Built-in — the Cowork session manages tool selection, context assembly, and execution |
| Knowledge and context assembly | Built-in — `SearchM365`, `ReadFileContent`, and Graph API tools provide grounded context |
| Memory and state | Partial — session memory persists within inline scheduled tasks; durable state needs M365 artifacts |
| Decision and approval plane | Partial — `AskUserQuestion` and draft tools provide human-in-the-loop; no formal approval routing engine |
| Governance and control | Partial — skill instructions encode policies; audit logging is platform-level; no custom policy versioning |
| Evaluation and observability | Limited — no built-in skill-level metrics; evaluation happens through the quality rubric and manual testing |

**The key gap:** Cowork does not have a durable workflow state engine. The framework's "Process State" concept (pending approvals, prior decisions, case history) must be externalized to M365 artifacts. This is where first-class artifact support becomes essential.

### 3.3 M365 Artifacts as First-Class Process State

This is the most important architectural insight for making the framework useful in Cowork. Each M365 artifact type serves a specific role in the process:

| Artifact | Role in the Framework | How Skills Use It |
|---|---|---|
| **Excel** | Process state store (case tracker, checklist status, metrics) | The onboarding tracker workbook IS the process state — skills read current status, write updates, score completeness |
| **Word** | Evidence artifacts (readiness packets, summaries, policy extracts) | Skills generate documents that become the auditable record of what was assembled and reviewed |
| **PowerPoint** | Decision-support artifacts (review decks, status presentations) | Skills create decks that support human approval steps — the manager reviews a deck, not raw data |
| **SharePoint** | Source of truth (policies, checklists, templates, document library) | Skills read policies and checklists from SharePoint; uploaded employee documents live here |
| **Outlook** | Communication channel and signal source | Skills read incoming email for context; draft outgoing communications as reviewable drafts |
| **Teams** | Coordination channel and real-time routing | Skills post task assignments, status updates, and escalation notices to channels or chats |
| **Graph API** | People and org data (profiles, hierarchy, managers) | Skills resolve people, org structure, and reporting chains for routing decisions |
| **Calendar** | Time-bound process events (deadlines, review meetings) | Skills create calendar events for review milestones and deadline reminders |

**The design pattern:** Instead of a database-backed workflow engine, the Cowork HR plugin uses a SharePoint-hosted Excel workbook as the canonical case tracker, with Word, PowerPoint, and Outlook artifacts as the evidence trail. Each skill reads from and writes to this shared state through M365 tools. This is less formally rigorous than a purpose-built state engine, but it works within the M365 ecosystem and gives HR teams artifacts they already know how to work with.

### 3.4 Federated Connectors for Third-Party Systems

The framework references systems like "HRIS", "ATS", "ticketing system", and "identity workflow" — these do not exist natively in M365. The approach for Copilot Cowork follows a tiered model:

**Tier 1 — Graph Connectors (indexed into M365 Search)**

For HRIS platforms (Workday, SuccessFactors, BambooHR) and ATS platforms (Greenhouse, iCIMS), Graph Connectors index external records into M365 Search. Skills access them via `SearchM365(sources=["connectors"], connector_ids=["workday-connector"])`. This provides read access to employee records, job requisitions, and onboarding status from external systems without custom integration code.

**Tier 2 — SharePoint as Bridge**

When Graph Connectors are not available, the pragmatic approach is to maintain synchronized data in SharePoint lists or Excel workbooks that are populated by Power Automate flows from the third-party system. Skills interact with the SharePoint copy. Bidirectional sync is handled by Power Automate outside of Cowork.

**Tier 3 — Manual Input with Templates**

For systems with no integration path, skills provide structured intake via `AskUserQuestion` that captures data from manual lookups, writing it into the shared Excel tracker. The framework's Signal Inventory phase identifies exactly which data points are needed, so the skill can prompt for only what is missing rather than asking broad questions.

**Practical recommendation for a first pilot:** Start with Tier 2 (SharePoint as bridge) and Tier 3 (manual input). Graph Connectors require tenant admin setup and are better introduced in Wave 2 after the skill workflows are proven.

### 3.5 Governance in Cowork

The framework's governance model (ownership, access, data classification, audit, release management) maps to Cowork as follows:

| Governance Domain | Cowork Implementation |
|---|---|
| **Ownership** | Each skill has an author; personal instructions document team structure and escalation paths |
| **Access** | M365 permissions govern what data the skill can reach; Graph API respects tenant RBAC |
| **Data classification** | Skill guardrails enforce PII handling rules (e.g., "mask SSN", "never include salary data in Teams messages") |
| **Audit** | The platform logs tool invocations; artifacts in SharePoint and Outlook provide a document trail |
| **Release management** | Skills are versioned in OneDrive; the skill quality rubric (0–100 scoring) provides a pre-deployment gate |
| **Policy enforcement** | Encoded in skill instructions ("always draft, never auto-send", "require confirmation before updating tracker") |

**The main governance gap** is formal policy versioning. If the onboarding checklist changes, the skill author needs to update the skill instructions manually. The recommended mitigation is to keep policy content in SharePoint documents that the skill reads dynamically at runtime, rather than hard-coding policy rules into the SKILL.md body. This way, HR operations can update the checklist in SharePoint without modifying the skill.

### 3.6 Evaluation Approach

Following the framework's two-level evaluation model, adapted for Cowork:

**Component-level evaluation (per skill):**
- Trigger accuracy — does the skill activate on the right prompts and stay silent on wrong ones? Assessed via the quality rubric's trigger coverage analysis (8–10 should-trigger and 8–10 should-not-trigger phrases per skill)
- Output quality — do generated documents contain accurate, cited information? Assessed via manual review of 10+ outputs
- Tool success rate — do M365 tool calls return expected results? Assessed via dry-run testing

**Process-level evaluation (end-to-end):**
- Readiness cycle time — time from case creation to "Ready for Review" status
- Missing item detection rate — percentage of actual gaps identified by `hr-gap-detection`
- Draft acceptance rate — percentage of `hr-onboarding-comms` drafts sent without major edits
- Reroute rate — how often `hr-task-routing` assignments need to be changed
- Overdue case rate — percentage of cases not reaching readiness by start date minus 2 days
- Reviewer trust and override rate — how often HR specialists override skill recommendations

---

## Part 4: Implementation Roadmap

Following the framework's wave structure, adapted for Cowork:

### Wave 1 — Foundation

**Infrastructure setup:**
- Create the shared Excel onboarding tracker workbook in SharePoint with standard columns (Case ID, Employee Name, Start Date, Role, Location, Manager, Status, Created Date, Assigned To, Checklist Completion %)
- Upload onboarding policies, checklists, and location-specific guides to a dedicated SharePoint document library
- Create a SharePoint folder structure for per-employee onboarding evidence (uploaded documents)

**Skills to build:**
- `hr-onboarding-intake`
- `hr-readiness-packet`
- `hr-gap-detection`

**Operating posture:** AI assist mode only. Skills read and analyze but do not write to systems. All outputs are presented via Adaptive Card or generated documents for manual review. Test with 5–10 real onboarding cases.

### Wave 2 — Communication and Routing

**Skills to build:**
- `hr-onboarding-comms`
- `hr-task-routing`

**Promotions:**
- Promote `hr-gap-detection` to write-back mode (updates Excel tracker after user confirmation)
- Promote `hr-onboarding-intake` to write mode (creates case records after confirmation)

**Automation:**
- Set up a daily scheduled prompt that checks for onboarding cases with approaching start dates and surfaces any with incomplete status

**Operating posture:** AI draft plus approve for all communication and routing skills. Every output reviewed before action.

### Wave 3 — Review and Optimization

**Skills to build:**
- `hr-readiness-summary`

**Enhancements:**
- Introduce Graph Connectors for HRIS data if available at the tenant level
- Add proactive monitoring via scheduled prompt: flag overdue cases, highlight recurring blockers, suggest process improvements
- Refine all skills based on override patterns and reviewer feedback from Waves 1–2

**Measurement:**
- Cycle time reduction vs. pre-pilot baseline
- Draft acceptance rate target: above 70%
- Gap detection accuracy target: above 85%
- Overdue case rate target: below 10%

---

## Part 5: Generalizing the Approach — Framework-to-Cowork Translation Method

For any line of business applying this framework to Cowork, the repeatable method is:

1. **Decompose** using the framework's rules until each step has one goal and one decision type
2. **Assign automation boundaries** — this directly determines each skill's guardrail posture
3. **Map signals to M365 tools** — replace every generic "CRM" or "ticketing system" reference with specific MCP tool names (Graph API, Outlook, Teams, SharePoint)
4. **Identify the process state artifact** — which Excel workbook or SharePoint list will serve as the canonical tracker
5. **Select skill templates** — each step maps to Data Aggregation, Content Generation, or Decision Support
6. **Define first-class outputs** — which steps produce Excel, Word, PowerPoint, or Adaptive Card artifacts
7. **Encode governance in guardrails** — translate the framework's policy and approval layer into SKILL.md guardrails sections
8. **Plan federated access** — classify third-party system needs as Graph Connector, SharePoint Bridge, or Manual Input
9. **Build Wave 1 as read-only** — start with skills that analyze and present; add write actions after trust is established
10. **Evaluate at the process level** — measure cycle time and acceptance rate, not just model output quality

### Cross-LOB Artifact Patterns

This method works across all 19 LOB samples in the framework repository. Each domain has a natural primary artifact:

| Line of Business | Primary M365 Artifact | Why |
|---|---|---|
| Finance | Excel (journal entries, variance analysis, reconciliation workbooks) | Financial data is inherently tabular and auditable |
| Legal | Word (contract review, clause comparison, legal memos) | Legal work products are document-centric |
| Customer Service | Outlook + Teams (case communication, escalation routing) | Service workflows center on real-time communication |
| Procurement | Excel + SharePoint (vendor scorecards, approval tracking) | Procurement tracks structured data with document evidence |
| Sales | PowerPoint + Outlook (deal review decks, prospect outreach) | Sales needs presentation artifacts and communication |
| HR | Excel + Word + SharePoint (case tracking, policy packets, evidence) | HR combines structured tracking with document-heavy evidence |
| Marketing | PowerPoint + Word (campaign briefs, content drafts) | Marketing produces presentation and written deliverables |
| IT Service Management | Teams + Excel (ticket routing, SLA tracking) | ITSM needs real-time coordination and metric tracking |

The framework's decomposition is universal. The M365 artifact mapping is what makes it concrete for Copilot Cowork.

---

## Appendix: Framework Concept to Cowork Concept Reference

| Framework Concept | Cowork Equivalent | Notes |
|---|---|---|
| Process | Skill collection or plugin suite | A process maps to a set of related skills sharing a common tracker |
| Step | Individual SKILL.md | Each step with one dominant goal becomes one skill |
| Skill (framework) | Cowork Skill (SKILL.md) | Direct mapping — reusable business capability |
| Tool or Plugin | MCP tools (Graph, Outlook, Teams, SharePoint) | Native M365 tools replace generic system references |
| Workflow | Multi-skill orchestration | Cowork handles via sequential skill invocation within a session |
| Agent | Subagent (general-purpose or deep-research) | Used sparingly per framework guidance — prefer skills and tools |
| Policy or Guardrail | Guardrails section in SKILL.md | Embedded in skill instructions; dynamic policy read from SharePoint |
| Process State | SharePoint-hosted Excel workbook or SharePoint list | Durable state externalized to M365 artifacts |
| Signal Intake | M365 MCP tools (Outlook, Teams, Calendar, SharePoint) | Email arrival, Teams message, form submission, calendar event |
| Approval | AskUserQuestion + CreateDraftMessage + confirmation gates | Human-in-the-loop via Cowork's review-before-action patterns |
| Evaluation | Quality rubric scoring + process-level metrics | Component eval via trigger analysis; process eval via cycle time and acceptance rate |
