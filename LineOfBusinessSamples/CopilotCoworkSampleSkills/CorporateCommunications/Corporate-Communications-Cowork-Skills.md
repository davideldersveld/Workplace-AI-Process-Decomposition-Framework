# Corporate Communications Announcement Brief — Copilot Cowork Skill Suite

## Overview

This skill suite implements the **Announcement Request Intake and Message Brief Synthesis** workflow as a set of five Copilot Cowork skills. The skills follow the Process Decomposition Framework to convert the corporate communications brief process — from intake through review routing — into AI-assisted capabilities within Microsoft 365.

The workflow supports organizational changes, product announcements, executive communications, crisis responses, policy updates, and event announcements, guiding each through structured brief creation with brand, legal, HR, and executive review controls at every step.

## Skill Suite

| # | Skill | Purpose | Automation Mode | Category | Icon |
|---|-------|---------|----------------|----------|------|
| 1 | **comms-request-intake** | Normalizes incoming announcement requests into structured brief case records | Deterministic automation | analysis | TaskListLtr |
| 2 | **comms-context-packet** | Assembles communications policies, brand guidelines, audience profiles, and prior announcements | AI act within policy | analysis | SearchSparkle |
| 3 | **comms-message-extraction** | Extracts key message themes, audience implications, and sensitivity indicators | AI assist | analysis | Flag |
| 4 | **comms-brief-draft** | Drafts a structured communications brief with key messages, channels, and timing | AI draft + approve | writing | Document |
| 5 | **comms-review-routing** | Routes the brief to legal, HR, executive, and sponsor reviewers | AI act within policy | communication | Mail |

## Workflow Sequence

```
┌─────────────────────┐
│  1. Request Intake   │  Normalize announcement request → structured brief case
│     (comms-request-  │  Supports: org changes, product announcements, executive
│      intake)         │  comms, crisis response, policy updates, events
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  2. Context Packet   │  Assemble policies, brand voice, channel taxonomy,
│     (comms-context-  │  audience profiles, prior announcements, stakeholders
│      packet)         │  Output: Word context packet
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  3. Message          │  Extract key messages, audience implications, timing,
│     Extraction       │  sensitivities, dependencies, conflicts, open questions
│     (comms-message-  │  Output: Adaptive Card + Excel message themes
│      extraction)     │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  4. Brief Draft      │  Draft structured brief: executive summary, key messages,
│     (comms-brief-    │  audience/channels, timing, sensitivity, reviews needed
│      draft)          │  Output: Word communications brief
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  5. Review Routing   │  Route to legal, HR, executive, media relations, and
│     (comms-review-   │  sponsor reviewers per the approval matrix
│      routing)        │  Output: Teams DMs + Outlook drafts + calendar holds
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  6. Human            │  Final brief disposition — approve, revise, or return
│     Disposition      │  for rework. Human-only step.
│     (not automated)  │
└─────────────────────┘
```

## Automation Boundary Summary

| Mode | Skills | What It Means |
|------|--------|---------------|
| **Deterministic automation** | Request Intake | Structured field extraction and duplicate checking — no AI judgment |
| **AI act within policy** | Context Packet, Review Routing | Retrieves approved context and routes per the approval matrix; does not interpret policy or override routing rules |
| **AI assist** | Message Extraction | Surfaces themes and risks for analyst review; read-only, no case modifications |
| **AI draft + approve** | Brief Draft | AI produces the brief; communications manager reviews and confirms before any distribution |
| **Human only** | Final Disposition | Brief approval or rejection is always a human decision |

## Governance Controls

### Premature Release Prevention
- Every output is a draft until human disposition — skills never finalize, distribute, or share content without explicit confirmation
- Brief drafts are not routed to reviewers until the communications manager approves the routing recommendation
- No announcement content is posted to Teams channels or email distribution lists without completing the full review cycle

### Embargo Enforcement
- Skills track and respect embargo dates throughout the workflow
- Embargoed content never appears in channel-wide Teams posts — only direct messages to approved reviewers
- Timing validation checks that requested dates allow for the minimum review cycle (2 business days)

### Executive Language Fidelity
- When executives provide specific language, skills preserve it exactly
- Executive-provided messaging is marked "Executive-provided — do not paraphrase" in the brief
- Original sponsor language is preserved alongside any summarized version

### Sensitive Topic Handling
- Announcements involving litigation, personnel actions, regulatory matters, or M&A are flagged for elevated review
- Sensitive topics require legal and/or HR review regardless of other characteristics
- Restricted distribution is enforced — sensitive content only shared with approved reviewers via direct message

### Channel Discipline
- Channel recommendations align with the approved communications policy channel taxonomy
- Internal announcements are not shared through external channels
- Employee-first announcements sequence internal distribution before any external release

### Multi-Stakeholder Review Integrity
- Skills never present a brief as "review complete" until all required reviewers have responded
- Partial review status is always clearly visible
- Required reviewers are never skipped, even for routine announcements

## M365 Tool Coverage

| Tool | Used By |
|------|---------|
| SearchM365 (files) | All skills — find tracker, policies, guidelines, templates, prior announcements |
| SearchM365 (email) | Intake, Context Packet, Message Extraction — find request emails, sponsor directives |
| SearchM365 (teams) | Message Extraction, Review Routing — find discussions, reviewer responses |
| ReadFileContent | All skills — read policies, guidelines, templates, case materials |
| GetDriveChildren | Intake, Context Packet, Message Extraction — browse folders |
| SearchPeople / GetUserDetails | Intake, Context Packet, Review Routing — resolve identities |
| GetMyDetails | Request Intake — assigned communications manager |
| GetManagerDetails / GetDirectReportsDetails | Context Packet, Review Routing — executive chain |
| GetMessage | Request Intake — read intake emails |
| CreateDraftMessage | Review Routing — formal review request emails (never auto-send) |
| PostMessage | Review Routing — Teams direct messages to reviewers (after confirmation) |
| CreateEvent | Review Routing — review deadline calendar holds |
| render_ui (Adaptive Card) | Message Extraction, Brief Draft, Review Routing — structured data display |

## Artifact Patterns

| Artifact | Format | Produced By |
|----------|--------|-------------|
| Brief case record | Excel tracker row | Request Intake |
| Context packet | Word document | Context Packet |
| Message themes matrix | Excel worksheet | Message Extraction |
| Risk indicator summary | Adaptive Card | Message Extraction |
| Communications brief | Word document | Brief Draft |
| Draft summary | Adaptive Card | Brief Draft |
| Review routing recommendation | Adaptive Card | Review Routing |
| Review request (formal) | Outlook draft email | Review Routing |
| Review notification (sensitive) | Teams direct message | Review Routing |
| Review deadline | Calendar event | Review Routing |
| Review status summary | Adaptive Card | Review Routing |

## Communication Types and Required Reviewers

| Communication Type | Required Reviewers |
|-------------------|-------------------|
| **All announcements** | Communications lead + business sponsor |
| **Legal topics** (litigation, regulatory, compliance) | + Legal reviewer |
| **People matters** (organizational change, leadership changes, layoffs) | + HR reviewer |
| **Executive or external-facing** | + Executive communications reviewer |
| **Media exposure** | + Media relations reviewer |
| **Crisis or time-sensitive** | All applicable + shortened deadlines |
| **Confidential sensitivity** | All applicable + restricted distribution |

## Installation

Each skill is installed as a `SKILL.md` file in the Copilot Cowork skills directory:

```
/mnt/user-config/.claude/skills/
├── comms-request-intake/SKILL.md
├── comms-context-packet/SKILL.md
├── comms-message-extraction/SKILL.md
├── comms-brief-draft/SKILL.md
└── comms-review-routing/SKILL.md
```

### Prerequisites

The skills expect these SharePoint resources to be available via M365 search:

- **Communications brief tracker** — shared Excel workbook for brief case records
- **Communications policy library** — SharePoint document library with communications policies and brand voice guidelines
- **Channel taxonomy** — document defining approved channels, audience segments, and distribution rules
- **Review routing matrix** — defines which reviewers are required based on communication type, sensitivity level, and audience
- **Brief templates** — Word templates for the communications brief format
- **Announcement history archive** — SharePoint library of prior announcements indexed by topic and date

### Trigger Phrases

Each skill responds to natural language triggers. Examples:

| Skill | Example Triggers |
|-------|-----------------|
| Request Intake | "new announcement request", "communications brief needed for the merger", "log comms request", "set up announcement case for Q3 earnings" |
| Context Packet | "build context for CB-2026-015", "what policies apply to executive departures", "pull announcement history for product launches" |
| Message Extraction | "extract message themes from the request", "what are the key messages for the reorg", "identify risks for the layoff announcement" |
| Brief Draft | "draft the communications brief", "create message brief for CB-2026-015", "build the announcement brief" |
| Review Routing | "route brief for review", "send for legal review", "submit brief to executive review", "who needs to review this announcement" |

## Implementation Roadmap

### Wave 1 — Foundation
- Set up the shared Excel brief tracker in SharePoint
- Organize the communications policy library with versioned policies, brand voice guidelines, and channel taxonomy
- Create brief workspace folder template in SharePoint
- Build skills: `comms-request-intake`, `comms-context-packet`, `comms-message-extraction`, `comms-brief-draft`
- Operate in AI assist mode — all outputs presented for manual review
- Test with 5-10 real announcement requests including at least 2 sensitive-topic cases

### Wave 2 — Review Routing and Policy Checks
- Build `comms-review-routing`
- Promote intake to write mode (creates case records after confirmation)
- Promote context packet to full document generation
- Add review summary generation (synthesize reviewer feedback)
- Add policy-cited claim and channel checks
- Set up daily scheduled prompt for briefs with approaching deadlines
- Heightened caution for sensitive-topic briefs — remain in AI assist mode until trust is established

### Wave 3 — Optimization and Proactive Intelligence
- Introduce Graph Connectors for media monitoring if available
- Add multi-source announcement packet assembly (talking points, FAQ, channel-specific variants)
- Add proactive detection of review blockers and timing conflicts
- Add missing stakeholder detection
- Measurement targets: 35% reduction in time to first review-ready brief, 70%+ brief acceptance rate, 95%+ routing accuracy, zero embargo compliance incidents

## Implementation Notes

- **Human disposition is intentionally not automated** — the final brief approval decision is always a communications manager or lead responsibility
- **All communications are created as drafts** — nothing is sent, posted, or distributed without explicit user confirmation
- **Message extraction operates in read-only mode** — it surfaces themes and risks but never modifies case status or edits messages
- **Review routing follows the approval matrix strictly** — the skills do not skip required reviewers or override routing rules
- **Sensitive announcements use Teams direct messages only** — never channel posts for embargoed, confidential, or personnel-related content
- **The skills are designed to be used sequentially** but can also be invoked independently (e.g., routing a revised brief for re-review, or extracting themes from a new executive directive for an existing case)
