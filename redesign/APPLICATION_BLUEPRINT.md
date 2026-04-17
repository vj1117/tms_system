# TMS Internal Agentic Application Blueprint

## 1. Scope And Positioning

This document translates the agentic redesign into an actual target application blueprint.

It does not assume that a single giant “AI app” will replace TMS. Instead, it defines a production-ready platform composed of:

- deterministic workflow services
- an agent orchestration runtime
- a governed tool layer
- a context engineering layer
- secure operational and analytical data access patterns
- user-facing applications for employees, approvers, finance, travel ops, and admins

The objective is to build a system that is:

- more intelligent than the current TMS
- safer than the current TMS
- easier to operate than the current TMS
- explainable and auditable

## 2. Recommended Build Stack

The redesigned application should be polyglot by responsibility.

### 2.1 Frontend

Recommended:

- `Next.js` or `React` with a domain-based frontend architecture
- TypeScript
- a typed API client layer
- server-side session integration with enterprise identity

Why:

- the current React experience can be evolved rather than reinvented
- domain-level composition is easier with typed contracts
- agent/copilot surfaces can be embedded as sidebar, inbox, timeline, and case assistant modules

### 2.2 Deterministic business services

Recommended:

- `.NET` for domain services if the organization wants stack continuity
- separate services or modular monolith boundaries for:
  - requests
  - approvals
  - booking
  - expenses
  - documents
  - reporting
  - policy

Why:

- the existing team likely has .NET and enterprise integration familiarity
- the major problem today is coupling, not the .NET runtime itself

### 2.3 Agent runtime

Recommended:

- Python-based agent runtime for orchestration, context assembly, ranking, document pipelines, and AI model integrations
- all tool access exposed through internal typed service APIs, not direct DB calls

Why:

- Python has stronger ecosystem support for orchestration, retrieval, document pipelines, evaluation, and agent tooling
- this isolates agent logic from transactional domain logic

### 2.4 Data and retrieval infrastructure

Recommended:

- Postgres or equivalent for new application state where feasible
- object storage for document artifacts
- event bus / queue for workflow events
- graph store only for relationship-intensive workflows
- vector index only for semantic retrieval use cases
- warehouse or analytical store for reporting and long-horizon analysis

## 3. Application Modules

## 3.1 User-facing applications

### Employee portal

Capabilities:

- request creation and tracking
- guided travel copilot
- booking status
- expense submission
- reimbursement status
- receipt upload

### Approver workspace

Capabilities:

- approval inbox
- approver copilot summary
- decision context, policy signals, and history
- exception review

### Travel operations workspace

Capabilities:

- booking-ready queue
- disruption management
- vendor coordination tools
- travel operations agent recommendations

### Finance workspace

Capabilities:

- expense audit queue
- settlement review
- anomaly summaries
- policy and evidence support

### Admin and policy workspace

Capabilities:

- policy management
- workflow configuration
- agent behavior configuration
- quality and incident dashboards

## 3.2 Core backend services

### Request Service

Owns:

- request drafts
- request submission
- request state
- traveler and trip structure
- linked artifacts and references

### Approval Service

Owns:

- approval routing
- delegates
- approval state transitions
- SLA timers
- escalation rules
- decision audit trail

### Booking Service

Owns:

- booking cases
- option packages
- itinerary records
- booking readiness
- deterministic booking actions

### Expense Service

Owns:

- expense reports
- report lines
- settlements
- reimbursement states
- finance review states

### Document Service

Owns:

- document metadata
- storage references
- OCR outputs
- extraction outputs
- review status

### Policy Service

Owns:

- policy rules
- decisionable constraints
- policy references
- exception pathways

### Reporting Service

Owns:

- analytical queries
- operational metrics
- scheduled exports
- dashboard backing data

## 3.3 Agent platform modules

### Orchestrator

Owns:

- workflow-to-agent routing
- context packing
- policy enforcement before tool usage
- response shaping
- confidence-aware escalation

### Tool Gateway

Owns:

- typed tool registration
- access control
- rate limiting
- audit logs
- side-effect approval hooks

### Context Service

Owns:

- retrieval planning
- case context hydration
- structured summaries
- graph traversal where needed
- vector retrieval where needed

### Agent Evaluation Service

Owns:

- offline evals
- online quality scoring
- hallucination and safety checks
- tool-usage compliance metrics

## 4. Data Model Strategy

The redesigned system must treat data by category. The context strategy depends on the nature of the data, not on a generic “RAG everywhere” pattern.

## 4.1 Data categories

### A. Transactional workflow data

Examples:

- requests
- approval states
- itinerary records
- expense reports
- settlements
- delegates
- assignments

Characteristics:

- highly structured
- strongly time-sensitive
- authoritative
- auditable
- low tolerance for ambiguity

Use:

- relational storage
- deterministic APIs
- no embedding-first retrieval

### B. Reference master data

Examples:

- employees
- project codes
- cost centers
- vendors
- locations
- policy catalogs
- expense categories

Characteristics:

- structured or semi-structured
- high lookup frequency
- low ambiguity

Use:

- relational/document lookup
- exact-match and faceted retrieval
- optional alias index for fuzzy search

### C. Policy and SOP content

Examples:

- travel policy
- reimbursement rules
- finance process docs
- travel SOPs
- exception handling guides

Characteristics:

- long-form unstructured text
- versioned
- citation-sensitive
- frequently referenced by copilots

Use:

- chunking + embeddings + RAG
- plus metadata filters by policy domain, geography, user type, and effective date

### D. Documents and attachments

Examples:

- receipts
- invoices
- tickets
- visa documents
- invitation letters
- approval attachments

Characteristics:

- file-based
- multimodal
- often noisy
- require extraction and confidence scoring

Use:

- OCR + parser pipeline first
- store extracted structured fields
- optionally embed normalized text for semantic search
- never rely only on vector retrieval for factual extraction

### E. Relationship-heavy operational context

Examples:

- who approves whom
- which traveler is linked to which request, project, and trip
- exception chains
- repeated vendor issues
- linked finance and booking cases

Characteristics:

- graph-shaped
- relationship-rich
- useful for case reasoning and escalation

Use:

- knowledge graph or graph projection layer

### F. Historical narratives and prior decisions

Examples:

- previous exception cases
- approver comments
- finance review notes
- incident histories

Characteristics:

- semi-structured
- useful for analogical reasoning
- high risk of misleading retrieval if not filtered carefully

Use:

- hybrid retrieval:
  - metadata filters first
  - then embeddings
  - then deterministic ranking by similarity, recency, and policy relevance

## 5. Context Engineering Strategy

## 5.1 Core principle

Different TMS workflows need different context assembly methods.

The correct strategy is not:

- “use a vector DB for everything”

The correct strategy is:

- structured retrieval for facts
- graph retrieval for relationships
- vector retrieval for semantic policy/help content
- event/timeline retrieval for recent workflow state
- document extraction pipelines for file evidence

## 5.2 Where embeddings and RAG are needed

Use embeddings + RAG for:

- policy Q&A
- SOP retrieval
- help articles
- explanation of process steps
- prior exception analogs
- semantic search across review comments and knowledge articles

Do not use embeddings as the system of record for:

- request status
- approver identity
- whether an expense was settled
- whether a booking is confirmed
- who owns the current queue item

## 5.3 Chunking strategy

For policy and SOP content:

- chunk by semantic section, not fixed token size only
- preserve document title, section title, effective date, geography, employee type, and workflow domain as metadata
- link chunks back to canonical document versions

Recommended chunk targets:

- 300 to 800 tokens per chunk for policy text
- include overlap only across section boundaries where policy meaning spans clauses
- maintain “citation windows” so answers can point back to the exact clause

For long procedural guides:

- chunk by step group or task block
- create parallel “step summaries” for fast retrieval

## 5.4 Knowledge graph requirements

Yes, a knowledge graph is needed, but only for the relationship-heavy parts of TMS.

Recommended graph use cases:

- employee -> manager -> delegate -> approver chain
- request -> itinerary -> booking -> expense -> settlement linkage
- document -> report -> traveler -> project linkage
- repeated exceptions by traveler / project / route / vendor
- incident and integration failure propagation analysis

The graph should not replace transactional storage. It should be a read-optimized relationship layer generated from authoritative services and events.

## 5.5 Hybrid retrieval planner

The orchestrator should use a retrieval planner:

1. classify task type
2. decide required context sources
3. fetch structured facts first
4. fetch policy/help/exception analogs second
5. fetch graph neighbors if relationship context matters
6. build a bounded context package for the agent

Example:

For an approver copilot task:

- structured facts:
  request details, traveler, dates, cost, current approver, state
- policy context:
  relevant travel policy clauses
- historical analogs:
  similar exception cases in same policy area
- graph context:
  approval chain, linked project, related previous requests

## 5.6 Long-context handling

Do not push raw full request history, full policies, and all attachments into a single giant prompt.

Instead:

- summarize previous workflow events into a timeline capsule
- include only current-state-critical structured fields
- include citations for policy excerpts
- attach document extraction summaries, not raw OCR dumps by default
- allow drill-down retrieval when the agent asks for deeper evidence

## 6. Data Source Blueprint

## 6.1 Operational data plane

Purpose:

- authoritative current-state facts

Sources:

- Request Service DB
- Approval Service DB
- Booking Service DB
- Expense Service DB
- Document metadata store

Access pattern:

- typed internal APIs
- read models optimized by persona and use case

## 6.2 Knowledge plane

Purpose:

- policy, SOP, help, and historical explanatory retrieval

Sources:

- cleaned policy docs
- SOP docs
- approved knowledge articles
- curated historical exception corpus

Access pattern:

- vector index + metadata filters + citation store

## 6.3 Relationship plane

Purpose:

- linked-case reasoning and approval/incident traversal

Sources:

- event projections from operational services

Access pattern:

- graph queries for neighbors, paths, ownership chains, repeated patterns

## 6.4 Evidence plane

Purpose:

- document content understanding

Sources:

- receipts
- tickets
- invoices
- attachments

Access pattern:

- OCR/extraction pipeline first
- extracted field store second
- semantic document retrieval only as a supplement

## 7. Agent-to-Context Mapping

## 7.1 Request Intake Agent

Needs:

- structured employee/profile/project data
- limited policy retrieval
- no graph required by default
- no vector retrieval except for help/policy wording

## 7.2 Policy Advisor Agent

Needs:

- vectorized policy corpus
- metadata filters by region, business unit, travel type, role
- structured inputs about trip/expense context

## 7.3 Approver Copilot Agent

Needs:

- structured case facts
- timeline summary
- policy RAG
- graph lookup for delegation and related requests

## 7.4 Expense Auditor Agent

Needs:

- structured expense lines
- extracted receipt fields
- trip context
- policy RAG
- anomaly features

## 7.5 Document Intelligence Agent

Needs:

- document bytes
- OCR output
- extraction schema
- optional supplier/merchant normalization dictionary

No default need for large vector context beyond document classification aids.

## 7.6 Reporting Analyst Agent

Needs:

- warehouse/metric datasets
- anomaly summaries
- prior reporting templates

This should be analytical retrieval, not classic RAG first.

## 7.7 Integration Triage Agent

Needs:

- job logs
- error payloads
- case links
- graph traversal for impacted downstream records

## 8. Guardrails For Context

- never give agents full unrestricted DB access
- redact PII fields unless directly needed
- bound context by task and user role
- separate real-time facts from semantic background knowledge
- attach confidence to extracted document fields
- enforce freshness windows for stateful data
- record which retrieval sources were used in each answer

## 9. Build Sequence For The Agentic Application

## Phase 1: deterministic foundations

- design canonical domain schemas
- build Request, Approval, Expense, Document, Policy, and Booking services
- expose typed read models
- create event bus

## Phase 2: context foundation

- create policy corpus pipeline
- chunk and embed policy/SOP content
- create graph projection for approval and case relationships
- build document extraction pipeline and extracted-field store

## Phase 3: tool gateway and orchestrator

- implement tool registry
- implement autonomy enforcement
- implement retrieval planner
- implement trace and audit framework

## Phase 4: first safe agents

- Employee Travel Copilot
- Request Intake Agent
- Document Intelligence Agent
- Reporting Analyst Agent

## Phase 5: higher-leverage review agents

- Policy Advisor Agent
- Expense Auditor Agent
- Approver Copilot Agent
- Integration Triage Agent

## Phase 6: operations maturity

- Travel Operations Agent
- disruption handling playbooks
- exception routing automation
- multi-agent coordination for high-friction queues

## 10. Final Recommendation

The redesigned TMS should be built as an intelligent workflow platform, not as a pure LLM application.

The correct context engineering approach is hybrid:

- structured retrieval for operational truth
- RAG for policies and help content
- document extraction for evidence
- graph retrieval for relationship reasoning
- event timeline summarization for workflow context

Yes, embeddings are needed.
Yes, chunking is needed.
Yes, RAG is needed.
Yes, a knowledge graph is needed.

But each is needed for different parts of the problem, and forcing one retrieval pattern across all TMS workflows would recreate the same design mistakes in a new form.
