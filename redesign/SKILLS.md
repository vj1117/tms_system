# TMS Agent Skills

This file defines the recommended skill packs for the TMS agentic redesign.

## Shared Skills

### `workflow-state-reasoning`

Use when an agent must interpret where a request, booking, report, or settlement is in its lifecycle.

Must do:

- identify current state from canonical state models only
- cite the exact state evidence used
- avoid inventing state transitions

Must not do:

- infer hidden approvals or settlements without evidence
- mutate state directly

### `policy-interpretation`

Use when evaluating travel, booking, or expense compliance.

Must do:

- cite policy rule identifiers and effective dates
- distinguish policy violation from recommendation
- identify when a case requires exception handling

Must not do:

- claim policy authority from model intuition alone

### `human-handoff`

Use when confidence is low, a case is high-risk, or a write is disallowed.

Must do:

- state why the handoff is needed
- include recommended reviewer role
- include minimal decision-ready summary

### `secure-tool-usage`

Use for every agent with tool access.

Must do:

- call only least-privilege tools
- separate reads from writes
- log tool intent and reason

Must not do:

- use free-form DB access
- invoke write tools outside allowed autonomy level

### `pii-handling`

Use for any agent touching employee, travel, receipt, or finance data.

Must do:

- minimize returned PII
- redact unnecessary personal data in summaries
- respect region and retention rules

## Agent-Specific Skills

### `request-intake-playbook`

For Request Intake Agent.

Must do:

- map unstructured travel intent into draft fields
- ask for only missing critical fields
- prefer draft save over direct submit

### `policy-advisor-playbook`

For Policy Advisor Agent.

Must do:

- explain compliant and non-compliant options
- show exception path when relevant
- provide plain-language guidance for employees and approvers

### `document-intelligence-playbook`

For Document Intelligence Agent.

Must do:

- classify document type first
- extract normalized fields with confidence
- route low-confidence results to review

### `expense-audit-playbook`

For Expense Auditor Agent.

Must do:

- compare claimed amounts, categories, dates, and trip context
- flag duplicates and unsupported claims
- generate reviewer-ready summaries

### `approver-copilot-playbook`

For Approver Copilot Agent.

Must do:

- summarize request intent and risk
- distinguish facts, policy signals, and recommendations
- draft comments without making the decision

### `travel-ops-playbook`

For Travel Operations Agent.

Must do:

- prioritize traveler-impacting cases
- identify missing prerequisites before booking action
- prepare disruption and recovery actions

### `reporting-analyst-playbook`

For Reporting Analyst Agent.

Must do:

- summarize trends and outliers
- link findings to source metrics
- separate insight from speculation

### `integration-triage-playbook`

For Integration Triage Agent.

Must do:

- classify integration failures
- enrich incidents with affected business cases
- recommend retry, fallback, or escalation path
