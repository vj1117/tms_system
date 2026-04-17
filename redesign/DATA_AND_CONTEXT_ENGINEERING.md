# TMS Data And Context Engineering Design

## 1. Objective

This document defines how the redesigned TMS should represent, retrieve, and package context for agents and deterministic services.

The main design decision is:

`Treat context engineering as a multi-layer data architecture problem, not a prompt-engineering problem.`

## 2. Why The Current System Needs A Better Context Model

The current TMS mixes:

- structured workflow records
- long-form SOPs
- file attachments
- implicit approval chains
- review comments
- exception histories
- integration logs

In the current codebase, these are spread across:

- SQL stored procedures
- controller actions
- file shares
- background jobs
- spreadsheets, PDFs, and SOP documents

That is exactly the kind of environment where agents fail unless context is intentionally engineered.

## 3. Context Layers

TMS should define five context layers.

## 3.1 Live transaction context

Used for:

- current request state
- current approver
- current expense settlement state
- booking status

Source:

- deterministic operational services

Retrieval style:

- exact, typed, structured reads only

## 3.2 Relationship context

Used for:

- approval chains
- linked traveler/project/case relationships
- repeated exception patterns
- case dependency tracing

Source:

- graph projection built from events and master data

Retrieval style:

- graph traversal and neighborhood retrieval

## 3.3 Knowledge context

Used for:

- policy explanation
- SOP guidance
- procedural help
- exception precedent lookup

Source:

- chunked and embedded knowledge corpus

Retrieval style:

- metadata-filtered hybrid RAG

## 3.4 Evidence context

Used for:

- receipts
- invoices
- travel documents
- attachments

Source:

- OCR and extraction pipelines

Retrieval style:

- extracted structured fields first
- linked snippets or page evidence second

## 3.5 Operational telemetry context

Used for:

- integration triage
- agent quality review
- workflow bottleneck diagnosis

Source:

- job logs
- audit events
- metrics
- queue events

Retrieval style:

- filtered log/event retrieval
- anomaly feature lookup

## 4. Retrieval Decision Matrix

| Use case | Best retrieval mode | Why |
|---|---|---|
| “What is the status of request 123?” | Structured read | exact state, zero ambiguity |
| “Who should approve this request next?” | Structured read + graph | requires workflow state plus relationship chain |
| “Is this expense policy-compliant?” | Structured read + policy RAG | facts plus cited policy |
| “What is this receipt for?” | OCR/extraction pipeline | semantic retrieval alone is insufficient |
| “Show similar past exception cases” | Metadata filter + embeddings | analogy works only after case narrowing |
| “Why did the Everbridge sync fail?” | Log/event retrieval + graph | needs cause plus affected linkage |

## 5. Vectorization Strategy

Vector embeddings should be used only for semantic corpora and analogical retrieval.

Good candidates:

- policy documents
- SOPs
- knowledge articles
- curated exception summaries
- normalized review/comment summaries

Bad candidates:

- primary request records
- canonical approval state
- settlements
- employee entitlements
- exact booking records

## 6. Chunking Strategy

## 6.1 Policy content

Chunk by:

- document
- section
- subsection
- exception clause

Metadata required:

- policy domain
- geography
- employee class
- effective date
- superseded-by / supersedes linkage

## 6.2 SOP content

Chunk by:

- task block
- user persona
- workflow stage

Metadata required:

- persona
- channel
- business domain
- process version

## 6.3 Historical case summaries

Do not embed raw full cases by default.

Instead:

- create normalized case summaries
- strip unnecessary PII
- attach features such as policy type, exception type, route, expense category, outcome, and recency

## 7. Knowledge Graph Strategy

The graph should model relationships, not act as a dumping ground for every table.

Recommended node types:

- Employee
- Request
- ApprovalStep
- Itinerary
- BookingCase
- ExpenseReport
- ExpenseLine
- Document
- PolicyClause
- Vendor
- Incident
- ExceptionCase

Recommended edges:

- `REQUESTED_BY`
- `APPROVED_BY`
- `DELEGATED_TO`
- `LINKED_TO_PROJECT`
- `HAS_ITINERARY`
- `HAS_EXPENSE_REPORT`
- `HAS_DOCUMENT`
- `VIOLATES_POLICY`
- `SIMILAR_TO_EXCEPTION`
- `IMPACTED_BY_INCIDENT`

## 8. RAG Pipeline Design

For policy and SOP questions:

1. classify user task
2. infer retrieval filters
3. retrieve top policy/SOP chunks with metadata constraints
4. re-rank using task-aware scoring
5. answer with citations only from retrieved chunks

For exception analog retrieval:

1. filter by domain and decision type
2. retrieve similar case summaries
3. rank by policy relevance, recency, and structural similarity
4. pass only the top bounded examples

## 9. Context Packaging Format

Every agent should receive context in a structured package.

Suggested sections:

- `case_facts`
- `current_state`
- `timeline_summary`
- `policy_context`
- `relationship_context`
- `document_evidence`
- `allowed_tools`
- `guardrails`
- `required_output_schema`

This package should be built by the Context Service, not handcrafted in each agent.

## 10. Freshness And Reliability Rules

- workflow facts must come from live service reads
- policy docs must include version and effective date
- graph projections must include sync timestamp
- document extraction must include confidence and extraction timestamp
- historical analogs must show outcome date and similarity reason

## 11. Minimal Viable Context Stack

If the team wants to build incrementally, start with:

1. structured service reads
2. policy RAG corpus
3. document extraction pipeline
4. timeline summarizer

Then add:

5. graph projection
6. historical case retrieval
7. telemetry-aware triage context

## 12. Anti-Patterns To Avoid

- one global vector database containing all application data
- sending full request histories and raw OCR dumps into every prompt
- letting agents infer status from narrative text when structured state exists
- using historical examples without strong metadata filters
- allowing agents to retrieve or expose unnecessary PII

## 13. Final Design Decision

The right context engineering model for TMS is:

- service-backed structured truth
- metadata-rich RAG for policy and SOP knowledge
- OCR and extraction for evidence
- graph reasoning for relationships and linked cases
- timeline summarization for workflow continuity

That combination is necessary because TMS is not one kind of problem. It is simultaneously:

- a transactional workflow system
- a document-heavy evidence system
- a policy interpretation system
- a relationship-heavy approval network
- an operations and exception-management platform
