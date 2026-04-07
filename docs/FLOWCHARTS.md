# FinLytics Flowcharts

## 1) End-to-End System Architecture

```mermaid
flowchart LR
    U[Borrower / Manager Browser] --> F[Next.js Frontend]
    F -->|REST /api/v1| A[FastAPI Router Layer]

    A --> D[Document Extractor Service]
    A --> S[Rule Scoring Engine]
    A --> G[GSTIN Scoring Service]
    G --> R[Fraud Detection Service]
    A --> M[Application Assignment Service]
    M --> N[Manager Narrative Service]

    D --> J[(assigned_applications.json)]
    M --> J

    G --> ML[(ML Model Artifacts)]
    G --> FEAT[(Feature Data)]

    A --> RES[Response Schemas]
    RES --> F
```

## 2) Borrower User Flow

```mermaid
flowchart TD
    B1[Open Apply Page] --> B2[Upload Documents]
    B2 --> B3[Extract Documents API]
    B3 --> B4[Normalized Payload Ready]
    B4 --> B5[Run Calculate Score API]
    B5 --> B6[Run GSTIN Explainable Score API]
    B6 --> B7[View Risk + Fraud + Recommendations]
    B7 --> B8[Submit Application]
    B8 --> B9[Application Stored + Assigned]
```

## 3) Manager User Flow

```mermaid
flowchart TD
    M1[Open Manager Portal] --> M2[Load Assigned/Pending Applications]
    M2 --> M3[Select Application]
    M3 --> M4[Review Score Summary]
    M4 --> M5[Review Fraud Topology Graph]
    M5 --> M6[Review Top Reasons + Narrative]
    M6 --> M7[Chat / Clarifications]
    M7 --> M8[Accept and Progress Decision]
```

## 4) Core Data Flow

```mermaid
flowchart LR
    P[PDF/Inputs] --> EX[Document Extraction]
    EX --> NP[Normalized Payload]

    NP --> RS[Rule Scoring]
    NP --> GS[GSTIN Scoring]

    GS --> FD[Fraud Graph Analysis]
    GS --> AM[Amnesty Runtime Adjustment]

    RS --> AGG[Backend Scoring Envelope]
    GS --> AGG
    FD --> AGG
    AM --> AGG

    AGG --> ST[(Application Store JSON)]
    AGG --> UI[Frontend Views]
```

## 5) Hybrid Scoring Pipeline

```mermaid
flowchart TD
    H1[Input Features] --> H2[Rule-Based Score]
    H1 --> H3[ML Base PD]
    H3 --> H4[Add Fraud Penalty]
    H4 --> H5[Apply Amnesty Relief if Active]
    H2 --> H6[Combine Signals]
    H5 --> H6
    H6 --> H7[Risk Score / Credit Score / Band]
    H7 --> H8[Loan Amount + Tenure Recommendation]
```

## 6) Twist 1 Fraud Ring Detection Flow

```mermaid
flowchart TD
    T1[Seed/Observed GSTIN Transactions] --> T2[Build Directed Graph]
    T2 --> T3[Find Reachable Component Around Subject GSTIN]
    T3 --> T4[Compute Strongly Connected Components]
    T4 --> T5{Component Size >= 3 and Includes Subject?}
    T5 -- No --> T6[fraud_flag = false]
    T5 -- Yes --> T7[Mark Cycle Nodes and Cycle Edges]
    T7 --> T8[Compute recirculation/concentration/round-trip severity]
    T8 --> T9[fraud_score and fraud_summary]
    T9 --> T10[fraud_network nodes/edges/cycle_count]
```

## 7) Twist 2 GST Amnesty Runtime Flow

```mermaid
flowchart TD
    A1[Read GST_AMNESTY_START_DATE / END_DATE] --> A2{Window Valid and Active Today?}
    A2 -- No --> A3[No Relief]
    A2 -- Yes --> A4[Compute Relief from late_filing_ratio and confidence]
    A4 --> A5[Reduce PD by Relief Cap]
    A5 --> A6[Attach amnesty_policy metadata]
    A3 --> A7[Return Final PD]
    A6 --> A7[Return Final PD]
```

## 8) Application Lifecycle Flow

```mermaid
flowchart LR
    S1[submitted] --> S2[pending assignment]
    S2 --> S3[accepted by manager]
    S3 --> S4[under review]
    S4 --> S5[decision stage]
    S5 --> S6[approved / rejected / negotiated]
```

## 9) API Interaction Sequence

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant API as FastAPI
    participant SRV as Services
    participant DB as JSON Store

    U->>FE: Upload docs
    FE->>API: POST /extract-documents
    API->>SRV: document_extractor
    SRV-->>API: extracted payload
    API-->>FE: extraction response

    FE->>API: POST /calculate-score
    API->>SRV: scoring_engine
    SRV-->>API: rule score result
    API-->>FE: score result

    FE->>API: POST /gstin-score
    API->>SRV: gstin_scoring + fraud_detection
    SRV-->>API: gstin explainable result
    API-->>FE: gstin result

    FE->>API: POST /applications/submit
    API->>SRV: assignment_service
    SRV->>DB: persist application
    SRV-->>API: stored application
    API-->>FE: submit response
```
