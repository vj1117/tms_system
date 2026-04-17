# TMS Internal Design Document

## Executive Summary

TMS Internal is an internal Travel Management System that has expanded into a broader travel, approval, booking, expense, reimbursement, and reporting platform. The repository contains:

- a React frontend in `Travel/`
- a .NET 6 API in `TravelAPI/`
- business SOPs, training videos, and operational artifacts in `docs/`

The codebase supports the end-to-end lifecycle of:

- travel request creation
- approval routing
- vendor and self-booking workflows
- expense and non-travel reimbursements
- receipt ingestion and extraction
- reporting, settlements, and finance operations

The platform is functionally rich but structurally fragile. Domain logic, infrastructure logic, integrations, secrets, and operational workflows are tightly mixed.

## Repository Overview

### Frontend

The React app is route- and workflow-oriented.

Important observed areas:

- `src/App.js`: token handling, MSAL integration, session cleanup, assist launcher visibility
- `src/routes/`: public vs authenticated route configuration
- `src/views/`: top-level experience entrypoints such as approvals, requests, dashboard, login, and assist
- `src/components/`: domain groupings for booking, requisition, expense details, dashboards, reports, configurations, receipts, and self-booking
- `src/libs/endpoints/ParentFetch.js`: central fetch utility

The frontend relies heavily on:

- session storage for user context and role IDs
- route state for workflow transitions
- access-token exchange via backend APIs

### Backend

The API is a single ASP.NET Core host configured in `TravelAPI/TravelAPI/Program.cs`.

Observed structure:

- 18 controllers
- 29 data model classes
- 118 entity classes
- 9 middleware/background/infrastructure classes
- small NUnit test coverage for selected controllers/models

The backend owns almost every system concern:

- requests
- approvals
- travel options
- self-booking
- expenses
- non-travel reports
- receipts
- reports
- configurations
- employee data
- mobile endpoints
- AI assist and OCR flows

## Technology Stack

### Frontend stack

- React 18
- React Router 6
- Material UI 5
- Azure MSAL
- Elastic APM RUM
- Google Maps integrations
- Power BI client
- file/PDF helper libraries

### Backend stack

- ASP.NET Core Web API on .NET 6
- raw `System.Data.SqlClient`
- Swagger
- JWT and cookie authentication
- Elastic APM
- PDF/Excel/document processing libraries
- Azure OpenAI and Form Recognizer
- AWS Bedrock and Textract
- Microsoft Graph

### External dependencies inferred from config/code

- SQL Server MIS database
- DFS/network shares
- Microsoft Graph / Outlook
- Concur
- Everbridge
- travel vendor APIs
- Google Maps
- Fusion / Apigee services

## Business Capabilities

### Travel requests

The request flow captures:

- requester context
- project/billable metadata
- travel plan and trip segments
- passenger and visa details
- accommodation requirements
- meeting details
- approvals

The `TravelRequisition.jsx` workflow strongly indicates a multi-step form spanning these domains.

### Approvals

Approvals include:

- manager/approver inbox flows
- status transitions
- discussion/comment threads
- booking and QC-related approval variants

The behavior is spread across request, approval, and dashboard layers.

### Booking

TMS supports both:

- vendor-driven booking
- self-booking for flights and hotels

The system persists itineraries, options, generated documents, and related actions. `SelfBookingController` and booking-related components suggest a complex booking lifecycle that extends beyond basic search and confirmation.

### Expenses and reimbursements

The expense domain includes:

- travel expenses
- non-travel reimbursements
- project party reimbursements
- advances and settlements
- receipt upload/download
- receipt extraction support
- finance review workflows

### Reporting and operations

The reporting layer supports:

- travel dump
- expense dump
- non-travel dump
- concur dump
- budget reports
- SLA monitoring
- leadership and operational dashboards

## Current Architecture

### Frontend architecture

The frontend is organized around views and component islands rather than strong domain modules.

Observed patterns:

- route definitions directly wire page-level features
- many screens depend on `sessionStorage` values such as `RoleId`, `EmployeeId`, and `UserName`
- the API wrapper is thin and loosely typed
- role-based behavior is scattered in UI logic

Examples from code:

- `LandingPage.js` selects dashboard tabs based on role IDs like `189`, `190`, and `191`
- `App.js` manages authentication refresh and logout behavior directly in the app shell
- `ParentFetch.js` sends the API token using a custom `token` header

### Backend architecture

The backend is a monolith with many business and infrastructure responsibilities.

`Program.cs` wires:

- controllers
- JWT auth
- CORS
- Swagger
- Elastic APM
- SQL connection services
- hosted background services

Hosted services include:

- `ApprovalService`
- `WalletIntegration`
- `EmailReceiptService`
- `CreateOutlookEventService`
- `EverbridgeTravelRequest`
- `ExceptionReportSchedulerService`
- `JournalImportBackgroundService`

This means the web API host is also the job runner and integration orchestrator.

### Persistence model

The backend is strongly database-centric.

Observed patterns:

- controllers instantiate model/data model classes directly
- data model code opens raw SQL connections
- stored procedures drive major read/write operations
- DTO/entity mapping happens in multiple layers

The application logic is therefore split across:

- controllers
- models
- data models
- stored procedures not included in the repo

This makes the true domain behavior harder to reason about and test.

### File and document handling

The backend also handles:

- attachment storage
- receipt file uploads
- PDF generation/manipulation
- network-share writes using Windows impersonation

This infrastructure concern is embedded directly in request-processing logic.

## Integration Landscape

### Identity

- frontend uses Azure AD/MSAL
- backend issues or validates app-level JWT tokens

### Enterprise systems

- SQL Server
- report server
- DFS/file share
- Elastic APM

### Travel/finance systems

- booking/vendor APIs
- Concur
- Fusion / Apigee
- Everbridge

### AI/automation systems

- Azure OpenAI
- Azure Form Recognizer
- AWS Bedrock
- AWS Textract

## Key Flaws

### Security flaws

The repository contains severe security issues grounded in the codebase:

- `TravelAPI/TravelAPI/appsettings.Development.json` includes database credentials, client secrets, API keys, JWT secrets, report server passwords, and third-party credentials
- frontend environment files exist in source control
- `TravelAssistController.AssistDetails` exposes credential-like configuration data

These are immediate remediation items.

### Architecture flaws

- one API host owns many unrelated business domains
- controllers mix transport, orchestration, SQL access, file handling, and integration logic
- background automation runs inside the API process
- raw SQL/stored procedure dependence prevents clear service boundaries
- file-share and impersonation logic leaks into business request flows

### Engineering flaws

- weak layering and inconsistent naming across `Controllers`, `Models`, `DataModels`, and `Entities`
- limited automated test coverage for a large workflow surface
- generated coverage artifacts are committed
- `Travel/README.md` is still placeholder content
- build pipeline content appears stale and partially templated

### Product/process flaws

- role IDs are magic numbers
- workflow state is not represented as a clear typed state machine
- behavior depends on `ViewName` and action strings in multiple flows
- ownership boundaries between request, booking, approval, expense, and reporting are blurred

### Operational flaws

- difficult local setup due to internal dependencies
- background services appear tightly coupled to connectivity and environment state
- retry/idempotency behavior is not clear for long-running external workflows

## Why Reimagining Is Necessary

This codebase has reached the point where improvements are constrained by structure:

- every new workflow expands monolithic complexity
- integration risk grows with each feature
- security posture is currently unacceptable
- debugging likely requires knowledge spread across UI, API, SQL procedures, and operational conventions

A redesign is justified not only for modernization, but for safety, maintainability, and long-term delivery speed.

## Assumptions

- a large portion of business rules live in SQL stored procedures not present in the repo
- some operational behavior is defined outside source control through environment/config and process conventions
- the `docs/` SOPs are broadly representative of actual production workflows
