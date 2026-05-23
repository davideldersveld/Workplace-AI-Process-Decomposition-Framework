---
name: sc-impact-assess
description: |
  Assesses business impact, recommends priority, and identifies
  mitigation paths for a classified supply chain exception.
  Use when user asks to "assess impact of shortage [ID]",
  "how bad is the [item] disruption",
  "what's the customer impact for [case]",
  "priority recommendation for [shortage]",
  "mitigation options for [case ID]",
  or "should we escalate [shortage]".
  Do NOT use for creating a new exception case (use sc-shortage-intake),
  gathering demand and inventory context (use sc-context-packet),
  classifying the exception type (use sc-classify-exception),
  routing to an owner (use sc-route-exception),
  or drafting shortage communications (use sc-shortage-comms).
---

## Overview

Evaluates the business impact of a classified supply chain exception using the organization's severity rubric, demand exposure data, customer order impact, and mitigation option analysis. Recommends a priority level, mitigation path, and escalation decision. Presents the impact assessment for planner review and explicit confirmation before any priority is applied.

This skill operates in "AI draft plus approve" mode — priority recommendations and mitigation path suggestions are generated as drafts for planner review. Customer-impacting priorities and escalation decisions require operations manager approval.

## When to Use

- An exception has been classified and needs priority assignment and impact assessment
- A planner needs to determine the urgency and downstream exposure for a shortage case
- A priority reassessment is needed after new information (shipment update, supplier commitment, demand change)
- An exception may require escalation and needs evaluation against escalation criteria

## When NOT to Use

- Creating a new exception case record — use sc-shortage-intake
- Gathering demand, inventory, and shipment context — use sc-context-packet
- Classifying the exception type and likely cause — use sc-classify-exception
- Routing the exception to an owner — use sc-route-exception
- Drafting shortage summaries or follow-up communications — use sc-shortage-comms
- Confirming triage disposition — this is always a human decision (SC-007)

## Core Instructions

### Progress Tracking

Create tasks at the start:

```
TaskCreate(subject="Read severity rubric and assess impact", activeForm="Evaluating business impact")
TaskCreate(subject="Present impact assessment and mitigation options", activeForm="Assessing priority")
```

### Step 1: Read Impact Assessment Inputs

**Read the exception case and classification:**
- `SearchM365(sources=["files"], query="shortage tracker")` then `ReadFileContent` — Case ID, Item/SKU, Site, Exception Type, Root Cause, Customer Impact Flag, SLA Deadline

**Read context packet data:**
- Review inventory position, demand exposure, shipment status, supplier context from the assembled context

**Read the severity rubric:**
- `SearchM365(sources=["files"], query="severity rubric supply chain")` then `ReadFileContent` — priority level definitions, criteria thresholds, and escalation triggers

**Read escalation criteria:**
- `SearchM365(sources=["files"], query="escalation checklist supply chain")` then `ReadFileContent` — conditions that trigger escalation to operations manager

**Read mitigation templates:**
- `SearchM365(sources=["files"], query="mitigation template [exception type]")` then `ReadFileContent` — standard mitigation approaches for this exception type

### Step 2: Assess Customer Impact

Evaluate the downstream customer exposure:

| Dimension | Assessment |
|-----------|-----------|
| **Affected orders** | Count and total quantity of open customer orders impacted by the shortage |
| **Revenue exposure** | Approximate revenue at risk based on affected order values |
| **Customer tiers** | Which customer tiers are affected (strategic, enterprise, standard) |
| **Delivery commitments** | Orders with firm delivery dates that will be missed |
| **Service level impact** | Whether the shortage will cause service level agreement breaches |
| **Cascade risk** | Whether the shortage affects downstream production or assembly at customer sites |

**Data sources:**
- `SearchM365(sources=["connectors"], connector_ids=["erp-inventory-connector"])` — customer order details, delivery commitments, allocation data
- If connector unavailable: `SearchM365(sources=["files"], query="demand exposure [item]")` then `ReadFileContent`

### Step 3: Determine Recommended Priority

Apply the severity rubric:

| Priority | Criteria | SLA | Response Path |
|----------|---------|-----|---------------|
| **Critical** | Customer delivery commitments at risk with no workaround, revenue impact above threshold, strategic account affected, or safety/regulatory item | 1 hour | Immediate owner assignment, operations manager notification, escalation bridge consideration |
| **High** | Significant customer impact with limited workaround, multiple orders affected, or key production input shortage | 4 hours | Same-day owner assignment, elevated triage priority |
| **Medium** | Moderate impact with available workaround, limited customer exposure, or supply recovery expected within lead time | 1 business day | Standard triage queue, next planning cycle review |
| **Low** | Minimal customer impact, excess inventory at other sites available for transfer, or informational shortage below safety stock | 2 business days | Standard queue, next available planning review |

### Step 4: Evaluate Mitigation Options

Based on the exception type and available supply, recommend mitigation paths:

| Mitigation Path | When Applicable | Considerations |
|----------------|----------------|---------------|
| **Expedite** | Supplier can accelerate delivery; carrier can expedite transit | Cost premium; supplier willingness; transit time reduction |
| **Reallocate** | Inventory available at other network sites | Transfer time; customer priority trade-offs; allocation authority required |
| **Substitute** | Alternative item or supplier available | Customer acceptance; specification compatibility; qualification status |
| **Escalate to supplier** | Root cause is supplier-side; need commitment update or recovery plan | Relationship impact; contractual leverage; supplier capacity |
| **Adjust demand** | Demand can be deferred, reduced, or fulfilled from safety stock | Customer agreement required; revenue impact; SLA implications |
| **Accept and communicate** | No mitigation available; shortage must be absorbed | Customer communication required; service level impact acknowledged |

For each recommended mitigation path, assess:
- Feasibility (high, medium, low)
- Estimated time to implement
- Cost or trade-off implications
- Required approvals (allocation changes, customer communication, expedite spend)

### Step 5: Evaluate Escalation Need

Check against escalation criteria:

| Escalation Trigger | Action |
|-------------------|--------|
| **Critical priority** | Automatic escalation to operations manager |
| **Revenue exposure above threshold** | Operations manager and sales leadership notification |
| **Strategic account affected** | Account team notification in addition to supply chain |
| **Repeat pattern (3+ cases in 90 days)** | Escalation for systemic issue review |
| **No viable mitigation path** | Escalation for management decision on customer communication |
| **Regulated item at risk** | Compliance team notification |

### Step 6: Present Impact Assessment

Present findings via Adaptive Card (invoke `render-ui` skill first):

- **Case header** — Case ID, Item/SKU, Site, Exception Type, SLA Deadline
- **Customer impact summary** — affected orders, revenue exposure, customer tiers, delivery commitments at risk
- **Recommended priority** — Critical/High/Medium/Low with rubric-based justification
- **Mitigation options** — ranked list with feasibility, timeline, cost implications, and required approvals
- **Escalation recommendation** — escalate/do not escalate with criteria citation
- **SLA status** — time remaining on the SLA deadline; flag if SLA adjustment is recommended based on new priority
- **Approval requirement** — "Requires operations manager approval" for critical priority or escalation decisions
- **Assessment label** — "IMPACT ASSESSMENT — planner confirmation required before priority is applied"

### Step 7: Apply Priority (After Confirmation)

After the planner confirms (and operations manager approves for critical priority):
- Update Priority field in the shortage tracker
- Update SLA Deadline if priority change warrants adjustment
- Update Status to "Assessed — Ready for Routing"
- Record priority rationale and confirming user
- If escalation approved, flag for operations manager review

### Available MCP Tools

| Tool | Purpose |
|------|---------|
| SearchM365 (files) | Find shortage tracker, severity rubric, escalation checklist, mitigation templates |
| SearchM365 (connectors) | Pull customer order exposure, allocation data, and demand details via Graph Connector |
| ReadFileContent | Read severity rubric, escalation criteria, mitigation templates, tracker data |
| render_ui (Adaptive Card) | Present the impact assessment for planner review |

## Guardrails

- **Present priority as a draft recommendation** — require explicit planner confirmation before applying to the tracker
- **Require operations manager approval for critical priority** — critical priorities affect resource allocation and customer commitments
- **Never recommend allocation changes without explicit human approval** — reallocation trades off one customer against another and requires documented authority
- **Never auto-modify customer delivery commitments** — any change to customer-facing dates requires human approval and customer communication
- **Flag any customer-impacting shortage** as requiring operations manager visibility regardless of calculated priority
- **Revenue exposure estimates must be clearly labeled as approximate** — they are based on available order data and may not reflect all affected demand
- **Include the severity rubric criteria used** so the planner can validate the recommendation against the documented policy
- **Never downgrade a priority** set by a previous human decision without documenting the rationale
- **Include the SLA countdown** in every assessment output — urgency context affects routing and mitigation decisions
- **Never fabricate customer impact data** — if affected order count or revenue exposure is unknown, report it as unknown rather than estimating
- **Distinguish between planner-assessed priority and system-indicated priority** — if the context data suggests a different priority than the rubric, note both perspectives
