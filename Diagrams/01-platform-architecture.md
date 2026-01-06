# Bruin Atlas Platform Architecture

## High-Level System Overview

```mermaid
graph TB
    subgraph "External Systems"
        CUST[Customer Networks]
        ITSM[ITSM Tools]
        AUDITORS[External Auditors]
    end

    subgraph "Bruin Atlas Platform"
        subgraph "Presentation Layer"
            UI[Web UI<br/>React/Next.js]
            API_GW[API Gateway]
        end

        subgraph "Application Services"
            ATLAS_API[Atlas API Service]
            IMPORT[Import Service]
            HEALTH[Health Scoring Engine]
            COMPLIANCE[Compliance Service]
            SIMULATION[Simulation Engine]
        end

        subgraph "Intelligence Layer"
            RULES[Rule Engine]
            BASELINE[Baseline Engine]
            REF_ARCH[Reference Architecture]
        end

        subgraph "Data Layer"
            PG[(PostgreSQL<br/>Primary Store)]
            CACHE[(Redis Cache)]
            TS[(Time Series<br/>Metrics Store)]
        end

        subgraph "Integration Layer"
            MON_API[MonitoringAPI]
            DEV_API[DeviceAPI]
            EVENT[Event Pipeline]
        end
    end

    CUST -->|Device Data| DEV_API
    CUST -->|Metrics| MON_API

    UI --> API_GW
    API_GW --> ATLAS_API

    ATLAS_API --> IMPORT
    ATLAS_API --> HEALTH
    ATLAS_API --> COMPLIANCE
    ATLAS_API --> SIMULATION

    IMPORT --> PG
    HEALTH --> RULES
    HEALTH --> BASELINE
    COMPLIANCE --> PG
    SIMULATION --> REF_ARCH

    RULES --> PG
    BASELINE --> TS
    REF_ARCH --> PG

    MON_API --> EVENT
    DEV_API --> EVENT
    EVENT --> PG
    EVENT --> TS

    COMPLIANCE -->|Reports| AUDITORS
    ATLAS_API -->|Webhooks| ITSM

    style UI fill:#6f42c1,color:#fff
    style RULES fill:#28a745,color:#fff
    style PG fill:#0066cc,color:#fff
    style EVENT fill:#dc3545,color:#fff
```

---

## Service Architecture by Team

```mermaid
graph LR
    subgraph "Team 1: Platform Architecture"
        direction TB
        T1_API[Atlas REST API]
        T1_AUTH[Auth Service]
        T1_IMPORT[Import Wizard Backend]
        T1_DB[(PostgreSQL)]
        T1_DEPLOY[K8s Deployment]

        T1_API --> T1_AUTH
        T1_API --> T1_DB
        T1_IMPORT --> T1_DB
    end

    subgraph "Team 2: Network Model"
        direction TB
        T2_RULES[Rule Engine]
        T2_REF[Reference Model]
        T2_DEVICE[DeviceAPI]
        T2_HEALTH[Health Scoring]

        T2_RULES --> T2_REF
        T2_DEVICE --> T2_HEALTH
    end

    subgraph "Team 3: UI/Visualization"
        direction TB
        T3_TOPO[Topology Viewer]
        T3_DASH[Health Dashboard]
        T3_WIZARD[Import Wizard UI]
        T3_SIM[Simulation UI]

        T3_TOPO --> T3_DASH
    end

    subgraph "Team 4: Integration"
        direction TB
        T4_PIPE[Data Pipeline]
        T4_EVENT[Event Stream]
        T4_INCIDENT[Incident Connector]
        T4_TRUST[Data Trust Engine]

        T4_PIPE --> T4_EVENT
        T4_EVENT --> T4_INCIDENT
        T4_PIPE --> T4_TRUST
    end

    T3_TOPO -->|REST| T1_API
    T3_DASH -->|REST| T1_API
    T1_API -->|Rules| T2_RULES
    T1_API -->|Device Data| T2_DEVICE
    T4_PIPE -->|Store| T1_DB
    T4_EVENT -->|Health Signals| T2_HEALTH

    style T1_API fill:#0066cc,color:#fff
    style T2_RULES fill:#28a745,color:#fff
    style T3_TOPO fill:#6f42c1,color:#fff
    style T4_PIPE fill:#dc3545,color:#fff
```

---

## Data Flow Architecture

```mermaid
flowchart LR
    subgraph "Data Sources"
        DS1[Network Devices]
        DS2[Monitoring Agents]
        DS3[Manual Import]
    end

    subgraph "Ingestion"
        ING1[DeviceAPI Poller<br/>15-30 min cycle]
        ING2[MonitoringAPI<br/>Real-time]
        ING3[Import Service<br/>Batch]
    end

    subgraph "Processing"
        NORM[Normalizer]
        VALID[Validator]
        ENRICH[Enrichment]
        TRUST[Trust Scorer]
    end

    subgraph "Storage"
        PG[(PostgreSQL<br/>Topology & Config)]
        TS[(TimeSeries<br/>Metrics)]
        CACHE[(Redis<br/>Hot Data)]
    end

    subgraph "Analytics"
        RULES[Rule Engine]
        BASELINE[Baseline Calc]
        HEALTH[Health Aggregator]
    end

    subgraph "Output"
        API[REST API]
        WS[WebSocket<br/>Live Updates]
        EXPORT[Report Export]
    end

    DS1 --> ING1
    DS2 --> ING2
    DS3 --> ING3

    ING1 --> NORM
    ING2 --> NORM
    ING3 --> VALID

    NORM --> VALID
    VALID --> ENRICH
    ENRICH --> TRUST

    TRUST --> PG
    TRUST --> TS
    TRUST --> CACHE

    PG --> RULES
    TS --> BASELINE
    RULES --> HEALTH
    BASELINE --> HEALTH

    HEALTH --> CACHE
    CACHE --> API
    CACHE --> WS
    PG --> EXPORT

    style ING1 fill:#dc3545,color:#fff
    style TRUST fill:#ffc107,color:#000
    style HEALTH fill:#28a745,color:#fff
```

---

## Phase 2 Features: Service Dependencies

```mermaid
graph TB
    subgraph "Feature 1: Compliance Reporting"
        C1[Compliance Service]
        C2[Evidence Collector]
        C3[Report Generator]
        C4[Auditor Portal]

        C1 --> C2
        C2 --> C3
        C1 --> C4
    end

    subgraph "Feature 2: Behavior Baselines"
        B1[Baseline Calculator]
        B2[Deviation Detector]
        B3[Alert Enricher]

        B1 --> B2
        B2 --> B3
    end

    subgraph "Feature 3: Change Simulation"
        S1[Topology Cloner]
        S2[Path Analyzer]
        S3[Impact Calculator]
        S4[Simulation API]

        S1 --> S2
        S2 --> S3
        S3 --> S4
    end

    subgraph "Existing Services"
        EX1[Rule Engine]
        EX2[Health Scoring]
        EX3[DeviceAPI]
        EX4[MonitoringAPI]
        EX5[(PostgreSQL)]
        EX6[(TimeSeries)]
    end

    C2 -->|Query| EX3
    C2 -->|Query| EX4
    C2 -->|Store| EX5
    C3 -->|Rules| EX1

    B1 -->|Metrics| EX6
    B2 -->|Enrich| EX1
    B3 -->|Update| EX2

    S1 -->|Clone| EX5
    S2 -->|Evaluate| EX1
    S3 -->|Score| EX2

    style C1 fill:#0066cc,color:#fff
    style B1 fill:#28a745,color:#fff
    style S1 fill:#6f42c1,color:#fff
    style EX1 fill:#ffc107,color:#000
```

---

## Infrastructure Topology

```mermaid
graph TB
    subgraph "Production Cluster (K8s)"
        subgraph "Ingress"
            LB[Load Balancer]
            ING[Ingress Controller]
        end

        subgraph "Frontend Pods"
            FE1[UI Pod 1]
            FE2[UI Pod 2]
        end

        subgraph "API Pods"
            API1[API Pod 1]
            API2[API Pod 2]
            API3[API Pod 3]
        end

        subgraph "Worker Pods"
            W1[Import Worker]
            W2[Event Processor]
            W3[Baseline Calculator]
            W4[Report Generator]
        end

        subgraph "Data Pods"
            PG1[(PostgreSQL Primary)]
            PG2[(PostgreSQL Replica)]
            REDIS[(Redis Cluster)]
            TS1[(TimeSeries DB)]
        end
    end

    subgraph "External Services"
        CDN[CDN / Static Assets]
        MON[Monitoring Stack]
        LOG[Log Aggregator]
    end

    LB --> ING
    ING --> FE1
    ING --> FE2
    ING --> API1
    ING --> API2
    ING --> API3

    API1 --> PG1
    API2 --> PG1
    API3 --> PG1
    API1 --> REDIS
    API2 --> REDIS
    API3 --> REDIS

    W1 --> PG1
    W2 --> PG1
    W2 --> TS1
    W3 --> TS1
    W4 --> PG1

    PG1 --> PG2

    FE1 --> CDN
    API1 --> MON
    W1 --> LOG

    style LB fill:#0066cc,color:#fff
    style PG1 fill:#28a745,color:#fff
    style REDIS fill:#dc3545,color:#fff
```

---

## API Service Map

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              ATLAS API GATEWAY                                │
├──────────────────────────────────────────────────────────────────────────────┤
│  /api/v1                                                                     │
│  ├── /auth                    Authentication & Authorization                 │
│  │   ├── POST /login          │  ├── POST /refresh                          │
│  │   └── POST /logout         │  └── GET  /me                               │
│  │                                                                           │
│  ├── /customers               Customer Management                            │
│  │   ├── GET  /               │  ├── POST /                                 │
│  │   ├── GET  /:id            │  └── PUT  /:id                              │
│  │                                                                           │
│  ├── /networks                Network Topology                               │
│  │   ├── GET  /:customerId    │  ├── GET  /:id/topology                     │
│  │   └── GET  /:id/health     │  └── GET  /:id/devices                      │
│  │                                                                           │
│  ├── /devices                 Device Operations                              │
│  │   ├── GET  /               │  ├── GET  /:id                              │
│  │   ├── GET  /:id/status     │  └── GET  /:id/baseline                     │
│  │                                                                           │
│  ├── /health                  Health & Scoring                               │
│  │   ├── GET  /dashboard      │  ├── GET  /scores/:networkId                │
│  │   └── GET  /trends         │  └── GET  /deviations                       │
│  │                                                                           │
│  ├── /incidents               Incident Management                            │
│  │   ├── GET  /               │  ├── GET  /:id                              │
│  │   └── GET  /active         │  └── PUT  /:id/resolve                      │
│  │                                                                           │
│  ├── /import                  Import Wizard                                  │
│  │   ├── POST /start          │  ├── POST /:id/upload                       │
│  │   ├── GET  /:id/status     │  └── POST /:id/complete                     │
│  │                                                                           │
│  ├── /compliance   [PHASE 2]  Compliance Reporting                          │
│  │   ├── GET  /frameworks     │  ├── POST /reports/generate                 │
│  │   ├── GET  /reports        │  └── GET  /reports/:id                      │
│  │                                                                           │
│  ├── /baselines    [PHASE 2]  Behavior Baselines                            │
│  │   ├── GET  /profiles       │  ├── GET  /deviations                       │
│  │   └── PUT  /config         │  └── GET  /alerts                           │
│  │                                                                           │
│  └── /simulate     [PHASE 2]  Change Simulation                             │
│      ├── POST /remove-device  │  ├── GET  /:id/results                      │
│      └── POST /batch          │  └── GET  /:id/export                       │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Database Schema Overview

```mermaid
erDiagram
    CUSTOMER ||--o{ NETWORK : owns
    NETWORK ||--o{ LOCATION : contains
    LOCATION ||--o{ DEVICE : hosts
    DEVICE ||--o{ HEALTH_SIGNAL : generates
    DEVICE ||--o{ INCIDENT : affects
    NETWORK ||--o{ INCIDENT : has

    DEVICE }|--|| DEVICE_TYPE : is
    DEVICE_TYPE ||--o{ BASELINE_PROFILE : has

    NETWORK ||--o{ COMPLIANCE_REPORT : generates
    COMPLIANCE_REPORT }|--|| FRAMEWORK : uses

    CUSTOMER {
        uuid id PK
        string name
        string tier
        timestamp created_at
    }

    NETWORK {
        uuid id PK
        uuid customer_id FK
        string name
        json topology_data
        float health_score
        timestamp last_sync
    }

    LOCATION {
        uuid id PK
        uuid network_id FK
        string name
        string type
        json coordinates
    }

    DEVICE {
        uuid id PK
        uuid location_id FK
        uuid device_type_id FK
        string name
        string vendor
        string model
        string role
        json config_snapshot
        string trust_level
        timestamp last_seen
    }

    DEVICE_TYPE {
        uuid id PK
        string name
        string category
        json default_thresholds
    }

    HEALTH_SIGNAL {
        uuid id PK
        uuid device_id FK
        string metric_name
        float value
        string confidence
        timestamp recorded_at
    }

    BASELINE_PROFILE {
        uuid id PK
        uuid device_type_id FK
        uuid location_id FK
        string metric_name
        string time_segment
        float mean
        float std_dev
        timestamp window_end
    }

    INCIDENT {
        uuid id PK
        uuid network_id FK
        string severity
        string status
        json affected_devices
        timestamp detected_at
        timestamp resolved_at
    }

    COMPLIANCE_REPORT {
        uuid id PK
        uuid network_id FK
        uuid framework_id FK
        json evidence_snapshot
        float compliance_score
        timestamp generated_at
    }

    FRAMEWORK {
        uuid id PK
        string name
        string version
        json control_mappings
    }
```

---

*Diagrams created: 2026-01-06*
*Format: Mermaid (renders in GitHub, VSCode, most markdown viewers)*
