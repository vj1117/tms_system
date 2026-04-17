# TMS Internal Architecture Diagrams

## System Context

```mermaid
flowchart LR
    employee["Employee / Requester"]
    approver["Approver / Manager"]
    finance["Finance Team"]
    vendor["Travel Vendor / Agent"]
    admin["Admin"]

    web["Travel React SPA"]
    api["TravelAPI (.NET 6)"]
    db["MIS SQL Server"]
    files["DFS / File Share"]
    reports["Report Server"]

    aad["Azure AD / MSAL"]
    graph["Microsoft Graph"]
    travelVendor["Travel Vendor APIs"]
    concur["Concur"]
    everbridge["Everbridge"]
    fusion["Fusion / Apigee"]
    ai["Azure OpenAI / Bedrock / Textract"]

    employee --> web
    approver --> web
    finance --> web
    vendor --> web
    admin --> web

    web --> aad
    web --> api

    api --> db
    api --> files
    api --> reports
    api --> graph
    api --> travelVendor
    api --> concur
    api --> everbridge
    api --> fusion
    api --> ai
```

## Container View

```mermaid
flowchart TB
    subgraph frontend["Frontend"]
        app["App Shell"]
        routes["Route Layer"]
        workflows["Workflow Views and Components"]
        client["ParentFetch API Client"]
        session["Session Storage + Contexts"]
    end

    subgraph backend["Backend"]
        controllers["Controllers"]
        models["Models / DataModels"]
        jobs["Hosted Services"]
        docs["File/PDF/OCR Logic"]
        auth["JWT/Auth Middleware"]
    end

    subgraph infra["Infrastructure"]
        sql["SQL Stored Procedures"]
        share["Network Share"]
        ext["External Systems"]
    end

    app --> routes
    routes --> workflows
    workflows --> client
    session --> workflows
    session --> client

    client --> controllers
    controllers --> models
    controllers --> docs
    controllers --> auth
    models --> sql
    docs --> share
    controllers --> ext
    jobs --> sql
    jobs --> share
    jobs --> ext
```

## Travel Request To Booking Flow

```mermaid
sequenceDiagram
    participant User as Employee
    participant FE as React UI
    participant Req as Request APIs
    participant DB as SQL Server
    participant Apr as Approver
    participant Book as Booking APIs
    participant FS as File Share

    User->>FE: Create travel request
    FE->>Req: Submit requisition details
    Req->>DB: Persist request and workflow state
    DB-->>Req: Request ID / current state
    Req-->>FE: Request created

    Apr->>FE: Review request
    FE->>Req: Approve / reject / forward
    Req->>DB: Update approval state

    FE->>Book: Search/save options or self-book
    Book->>DB: Persist itinerary / booking records
    Book->>FS: Store generated documents and attachments
```

## Expense And Receipt Flow

```mermaid
sequenceDiagram
    participant Emp as Employee
    participant FE as React UI
    participant Exp as Expense APIs
    participant Rec as Receipt / OCR APIs
    participant DB as SQL Server
    participant AI as AI/OCR Services
    participant Fin as Finance

    Emp->>FE: Create expense or reimbursement
    FE->>Exp: Submit report lines and metadata
    Exp->>DB: Persist report

    Emp->>FE: Upload receipt
    FE->>Rec: Upload file
    Rec->>DB: Save receipt metadata
    Rec->>AI: Extract structured fields
    AI-->>Rec: Parsed receipt output
    Rec-->>FE: Receipt details

    Fin->>FE: Review settlement
    FE->>Exp: Load pending items
    Exp->>DB: Fetch settlement/report details
    Exp-->>FE: Review data
```

## In-Process Automation View

```mermaid
flowchart LR
    api["TravelAPI Host"]

    subgraph jobs["Hosted Background Services"]
        approval["ApprovalService"]
        wallet["WalletIntegration"]
        email["EmailReceiptService"]
        outlook["CreateOutlookEventService"]
        ever["EverbridgeTravelRequest"]
        exception["ExceptionReportSchedulerService"]
        journal["JournalImportBackgroundService"]
    end

    sql["SQL Server"]
    share["File Share"]
    ext["External APIs"]

    api --> jobs
    jobs --> sql
    jobs --> share
    jobs --> ext
```

## Architectural Smell Map

```mermaid
flowchart TD
    monolith["Single Deployable API"]
    largecontrollers["Large Controllers"]
    rawsql["Raw SQL / Stored Procedure Coupling"]
    sessionroles["Session Storage Role Logic"]
    bgjobs["In-Process Hosted Jobs"]
    fileshare["File Share + Impersonation"]
    secrets["Secrets Committed In Repo"]
    integrations["Many Tight Integrations"]

    monolith --> largecontrollers
    largecontrollers --> rawsql
    monolith --> bgjobs
    largecontrollers --> fileshare
    sessionroles --> largecontrollers
    integrations --> monolith
    secrets --> monolith
```
