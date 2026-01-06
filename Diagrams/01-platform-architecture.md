# Bruin Atlas Platform Architecture

This document provides comprehensive architectural diagrams with detailed explanations of each component, service, and design decision powering Bruin Atlas.

---

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

### Overview Description

The high-level system overview illustrates the complete Bruin Atlas platform architecture organized into five distinct layers, each with specific responsibilities and clear boundaries.

### External Systems

| System | Purpose | Integration Pattern |
|--------|---------|---------------------|
| **Customer Networks** | Source of all device data and metrics | Polled via DeviceAPI and MonitoringAPI |
| **ITSM Tools** | ServiceNow, Jira, PagerDuty integration | Outbound webhooks from Atlas API |
| **External Auditors** | Compliance report consumers | Read-only portal access via Compliance Service |

### Presentation Layer

The presentation layer handles all user-facing interactions:

- **Web UI (React/Next.js)**: Single-page application providing topology visualization, health dashboards, and configuration interfaces. Server-side rendered for initial load performance.
- **API Gateway**: Central entry point handling authentication, rate limiting, request routing, and API versioning. All external requests flow through this gateway.

### Application Services

Core business logic resides in these stateless, horizontally-scalable services:

| Service | Responsibility | Key Operations |
|---------|----------------|----------------|
| **Atlas API Service** | Primary REST API orchestrating all client requests | CRUD operations, query coordination, response aggregation |
| **Import Service** | Handles customer network data onboarding | File parsing, validation, normalization, deduplication |
| **Health Scoring Engine** | Computes real-time network health metrics | Score aggregation, trend analysis, threshold evaluation |
| **Compliance Service** | Generates audit-ready compliance reports (Phase 2) | Evidence collection, framework mapping, report generation |
| **Simulation Engine** | Runs "what-if" change impact analysis (Phase 2) | Topology cloning, path analysis, impact calculation |

### Intelligence Layer

The intelligence layer provides the "smarts" that differentiate Atlas:

- **Rule Engine**: Evaluates JSON-defined rules against network state. Detects redundancy gaps, compliance violations, and health anomalies. Supports both real-time evaluation and batch processing.
- **Baseline Engine**: Calculates statistical baselines per device type and location (Phase 2). Enables deviation detection and proactive alerting.
- **Reference Architecture**: Stores "gold standard" network topology patterns. Used for gap analysis and compliance scoring against best practices.

### Data Layer

Purpose-built storage for different data access patterns:

| Store | Technology | Use Case |
|-------|------------|----------|
| **PostgreSQL** | Relational DB | Topology, configurations, relationships, audit logs |
| **Redis Cache** | In-memory | Hot data, session state, real-time health scores |
| **Time Series Store** | InfluxDB/TimescaleDB | Metrics history, baseline calculations, trend analysis |

### Integration Layer

Connects Atlas to external data sources:

- **MonitoringAPI**: Pulls real-time metrics and alert data from customer monitoring infrastructure
- **DeviceAPI**: Queries device status, configurations, and capabilities from network devices
- **Event Pipeline**: Async message broker (Kafka/RabbitMQ) processing high-volume device events and routing to appropriate stores

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

### Team Ownership Description

This diagram maps services to team ownership, illustrating clear boundaries and inter-team dependencies. Each team owns specific services end-to-end, from development through production operation.

### Team 1: Platform Architecture (2 Engineers)

**Mission**: Build and maintain foundational infrastructure enabling all teams to deliver value.

| Service | Description |
|---------|-------------|
| **Atlas REST API** | Central API gateway handling all client requests, authentication, and orchestration |
| **Auth Service** | JWT-based authentication, API key management, RBAC enforcement |
| **Import Wizard Backend** | Multi-step import workflow, file processing, validation pipeline |
| **PostgreSQL** | Database schema design, migrations, query optimization, backup/recovery |
| **K8s Deployment** | CI/CD pipelines, Kubernetes manifests, environment management |

**Key Responsibilities**: Security posture, data retention policies, API contracts, deployment automation.

### Team 2: Network Model (1 Engineer)

**Mission**: Encode network expertise into rules and standards that power Atlas intelligence.

| Service | Description |
|---------|-------------|
| **Rule Engine** | JSON-based rule definitions, evaluation logic, health scoring algorithms |
| **Reference Model** | Gold-standard network topology patterns, redundancy requirements |
| **DeviceAPI** | Device capability queries, status polling, vendor-specific integrations |
| **Health Scoring** | Aggregation logic combining rule outputs into actionable scores |

**Key Responsibilities**: Reference model governance, rule accuracy, device knowledge documentation.

### Team 3: UI/Visualization (2 Engineers)

**Mission**: Make complex network infrastructure comprehensible through intuitive visualization.

| Service | Description |
|---------|-------------|
| **Topology Viewer** | Interactive network graph with zoom, pan, filter, and drill-down |
| **Health Dashboard** | Executive-level health overview with trend visualization |
| **Import Wizard UI** | Guided multi-step import flow with validation feedback |
| **Simulation UI** | What-if analysis interface showing change impact (Phase 2) |

**Key Responsibilities**: UX design, performance optimization, accessibility, responsive design.

### Team 4: Integration (3 Engineers)

**Mission**: Deliver real-time operational intelligence through reliable data pipelines.

| Service | Description |
|---------|-------------|
| **Data Pipeline** | ETL workflows moving data from sources to Atlas stores |
| **Event Stream** | Real-time event processing, message routing, fan-out |
| **Incident Connector** | Bi-directional sync with rule engine and external ticketing |
| **Data Trust Engine** | Confidence scoring, freshness tracking, quality validation |

**Key Responsibilities**: Data freshness (12hr→15min), incident integration, data trust indicators.

### Cross-Team Dependencies

| From | To | Dependency |
|------|----|----|
| Team 3 (UI) | Team 1 (Platform) | REST API endpoints for all data access |
| Team 1 (Platform) | Team 2 (Network Model) | Rule evaluation, device queries |
| Team 4 (Integration) | Team 1 (Platform) | Database write access for pipeline outputs |
| Team 4 (Integration) | Team 2 (Network Model) | Health signal delivery for scoring |

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

### Data Flow Description

This diagram traces how data moves through Atlas from source to consumer, highlighting transformation stages and storage decisions.

### Data Sources

| Source | Data Type | Volume | Freshness Requirement |
|--------|-----------|--------|----------------------|
| **Network Devices** | Status, config, topology | ~1000 devices/customer | 15-30 minutes |
| **Monitoring Agents** | Metrics, alerts, events | ~100K events/hour | Real-time (<5s) |
| **Manual Import** | Initial topology, bulk updates | One-time/periodic | Batch (minutes) |

### Ingestion Stage

The ingestion layer normalizes diverse data sources into a common format:

- **DeviceAPI Poller**: Scheduled jobs querying device status every 15-30 minutes. Implements backoff, retry, and rate limiting to avoid overwhelming devices.
- **MonitoringAPI**: Stream processor consuming real-time metrics. Handles burst traffic with buffering and back-pressure.
- **Import Service**: Batch processor for CSV/JSON uploads. Validates schema, detects duplicates, queues for processing.

### Processing Pipeline

Sequential processing ensures data quality before storage:

| Stage | Purpose | Key Operations |
|-------|---------|----------------|
| **Normalizer** | Convert vendor-specific formats to Atlas schema | Field mapping, unit conversion, ID normalization |
| **Validator** | Ensure data integrity | Schema validation, referential integrity, business rules |
| **Enrichment** | Add derived fields | Device classification, location inference, relationship detection |
| **Trust Scorer** | Assign confidence levels | Freshness check, source reliability, data completeness |

### Storage Strategy

Different stores optimize for different access patterns:

- **PostgreSQL**: Source of truth for topology, relationships, and configuration. Optimized for complex queries and transactional consistency.
- **TimeSeries**: Append-only metrics storage with time-based partitioning. Enables efficient range queries and downsampling.
- **Redis Cache**: Sub-millisecond access for hot data. Stores current health scores, active incidents, session state.

### Analytics Layer

Transform raw data into actionable intelligence:

- **Rule Engine**: Evaluates topology against reference model, detecting gaps and violations
- **Baseline Calculator**: Computes rolling statistics for anomaly detection
- **Health Aggregator**: Combines rule outputs and baseline deviations into unified health scores

### Output Channels

| Channel | Use Case | Latency Target |
|---------|----------|----------------|
| **REST API** | Dashboard queries, CRUD operations | <200ms p95 |
| **WebSocket** | Live topology updates, incident alerts | <1s |
| **Report Export** | Compliance reports, CSV downloads | Async (minutes) |

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

### Phase 2 Features Description

This diagram shows how the three Phase 2 features build upon existing platform services, minimizing new infrastructure while maximizing value delivery.

### Feature 1: Compliance Reporting (Blue)

**Goal**: Transform Atlas data into audit-ready compliance reports.

| Component | Purpose | Existing Service Dependency |
|-----------|---------|----------------------------|
| **Compliance Service** | Orchestrates report lifecycle | None (new service) |
| **Evidence Collector** | Gathers compliance evidence | DeviceAPI, MonitoringAPI for raw data |
| **Report Generator** | Produces formatted reports | Rule Engine for compliance rule evaluation |
| **Auditor Portal** | External auditor access | PostgreSQL for evidence storage |

**Key Insight**: 70% of required data already exists; primary work is transformation and templating.

### Feature 2: Behavior Baselines (Green)

**Goal**: Establish "normal" behavior patterns for proactive anomaly detection.

| Component | Purpose | Existing Service Dependency |
|-----------|---------|----------------------------|
| **Baseline Calculator** | Computes statistical profiles | TimeSeries DB for historical metrics |
| **Deviation Detector** | Identifies anomalies from baseline | Rule Engine for contextual evaluation |
| **Alert Enricher** | Adds baseline context to alerts | Health Scoring for unified output |

**Key Insight**: Statistical approach (no ML) using existing time-series data; rule engine provides correlation.

### Feature 3: Change Simulation (Purple)

**Goal**: Enable "what-if" analysis before network changes.

| Component | Purpose | Existing Service Dependency |
|-----------|---------|----------------------------|
| **Topology Cloner** | Creates in-memory topology copy | PostgreSQL for topology data |
| **Path Analyzer** | Traverses graph for impact | Rule Engine for redundancy evaluation |
| **Impact Calculator** | Computes health deltas | Health Scoring for before/after comparison |
| **Simulation API** | Exposes results to UI | None (new API endpoint) |

**Key Insight**: Simulation = running existing rules against modified topology state, not building a network simulator.

### Shared Foundation (Yellow)

All three features heavily leverage the **Rule Engine**, making it the central integration point:

- Compliance uses rules to evaluate compliance controls
- Baselines use rules to contextualize deviations
- Simulation uses rules to calculate hypothetical impact

**Implication**: Rule Engine must support "dry run" mode for simulation without side effects.

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

### Infrastructure Description

This diagram details the Kubernetes deployment topology for production Atlas, showing pod allocation, scaling strategy, and external service integration.

### Ingress Layer

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Load Balancer** | AWS ALB / GCP LB | SSL termination, DDoS protection, geographic routing |
| **Ingress Controller** | NGINX Ingress | Path-based routing, rate limiting, request logging |

**Scaling**: Load balancer scales automatically; ingress controller runs 2+ replicas for HA.

### Frontend Pods

| Component | Replicas | Resources | Scaling Trigger |
|-----------|----------|-----------|-----------------|
| **UI Pod** | 2-4 | 512MB RAM, 0.5 CPU | CPU > 70% |

**Details**: Serves static React bundle and handles SSR for initial page loads. Stateless; scales horizontally based on request volume.

### API Pods

| Component | Replicas | Resources | Scaling Trigger |
|-----------|----------|-----------|-----------------|
| **API Pod** | 3-10 | 1GB RAM, 1 CPU | CPU > 60%, latency > 200ms |

**Details**: Handles all REST API requests. Maintains connection pools to PostgreSQL and Redis. Health checks enable graceful failover.

### Worker Pods

Background processors handling async workloads:

| Worker | Purpose | Scaling Strategy |
|--------|---------|------------------|
| **Import Worker** | Process file uploads | Queue depth-based (autoscale on backlog) |
| **Event Processor** | Handle real-time events | Fixed replicas (high throughput, low latency) |
| **Baseline Calculator** | Compute statistical baselines | Scheduled (runs during off-peak hours) |
| **Report Generator** | Generate compliance reports | Queue depth-based (burst during audit season) |

### Data Pods

| Component | Topology | Backup Strategy |
|-----------|----------|-----------------|
| **PostgreSQL Primary** | Single leader | Daily snapshots, WAL streaming |
| **PostgreSQL Replica** | Read replica | Async replication (<1s lag) |
| **Redis Cluster** | 3-node cluster | RDB snapshots hourly |
| **TimeSeries DB** | Single instance | Daily retention-based archival |

### External Services

| Service | Provider | Purpose |
|---------|----------|---------|
| **CDN** | CloudFront/Fastly | Static asset delivery, edge caching |
| **Monitoring Stack** | Datadog/Prometheus | Metrics, alerting, dashboards |
| **Log Aggregator** | ELK/Splunk | Centralized logging, search, retention |

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

### API Service Map Description

This ASCII diagram documents all REST API endpoints, organized by domain. Each endpoint group serves a specific functional area of the Atlas platform.

### Core API Domains (Phase 1)

| Domain | Purpose | Key Consumers |
|--------|---------|---------------|
| **/auth** | Authentication and session management | All clients (UI, integrations) |
| **/customers** | Multi-tenant customer management | Admin UI, provisioning scripts |
| **/networks** | Network topology and health access | Topology Viewer, Health Dashboard |
| **/devices** | Individual device operations | Device detail views, search |
| **/health** | Health scores and trends | Executive dashboard, alerting |
| **/incidents** | Incident lifecycle management | NOC console, ITSM webhooks |
| **/import** | Data onboarding workflow | Import Wizard UI |

### Phase 2 API Additions

| Domain | Purpose | New Capabilities |
|--------|---------|------------------|
| **/compliance** | Compliance reporting | Framework listing, report generation, evidence access |
| **/baselines** | Behavior baseline management | Profile viewing, deviation queries, threshold config |
| **/simulate** | Change impact simulation | Device removal simulation, batch analysis, result export |

### API Design Principles

1. **RESTful conventions**: Resources as nouns, HTTP verbs for actions
2. **Versioned**: `/api/v1` prefix enables future breaking changes
3. **Consistent responses**: Standardized error format, pagination, filtering
4. **Auth required**: All endpoints except `/auth/login` require valid JWT
5. **Rate limited**: Per-customer limits prevent abuse (1000 req/min default)

### Authentication Flow

```
1. POST /auth/login → { email, password }
2. Response → { access_token, refresh_token, expires_in }
3. All subsequent requests: Authorization: Bearer <access_token>
4. Token refresh: POST /auth/refresh → { refresh_token }
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

### Database Schema Description

This entity-relationship diagram shows the core data model powering Atlas, with tables organized by domain.

### Core Entities

| Entity | Purpose | Key Relationships |
|--------|---------|-------------------|
| **CUSTOMER** | Multi-tenant root entity | Owns multiple networks |
| **NETWORK** | Customer's network infrastructure | Contains locations, has incidents |
| **LOCATION** | Physical or logical site | Hosts devices (HQ, branch, DC, cloud) |
| **DEVICE** | Network device instance | Belongs to location, generates signals |

### Device Classification

| Entity | Purpose | Usage |
|--------|---------|-------|
| **DEVICE_TYPE** | Canonical device categories | Router, switch, firewall, AP, etc. |
| **DEVICE.role** | Functional role in topology | Core, edge, access, WAN |
| **DEVICE.trust_level** | Data confidence indicator | Verified, inferred, stale, unknown |

### Health & Monitoring

| Entity | Purpose | Volume Considerations |
|--------|---------|----------------------|
| **HEALTH_SIGNAL** | Point-in-time device metrics | High volume; partitioned by time |
| **BASELINE_PROFILE** | Statistical profiles for baselines | Computed daily; moderate volume |
| **INCIDENT** | Active and historical incidents | Moderate volume; indexed by status |

### Compliance (Phase 2)

| Entity | Purpose | Key Fields |
|--------|---------|------------|
| **FRAMEWORK** | Compliance framework definitions | SOC2, ISO, PCI-DSS control mappings |
| **COMPLIANCE_REPORT** | Generated compliance reports | Evidence snapshot, score, timestamp |

### Schema Design Principles

1. **UUID primary keys**: Enable distributed ID generation, prevent enumeration
2. **JSON columns for flexibility**: `topology_data`, `config_snapshot`, `control_mappings` accommodate varying structures
3. **Timestamp tracking**: `created_at`, `last_seen`, `recorded_at` enable audit and freshness
4. **Soft references**: `affected_devices` as JSON array avoids complex junction tables
5. **Tenant isolation**: All queries filter by `customer_id` via application layer

### Index Strategy

| Table | Indexed Columns | Purpose |
|-------|-----------------|---------|
| DEVICE | `location_id`, `device_type_id`, `last_seen` | Topology queries, freshness checks |
| HEALTH_SIGNAL | `device_id`, `recorded_at` | Time-range metric queries |
| INCIDENT | `network_id`, `status`, `detected_at` | Active incident lookups |
| BASELINE_PROFILE | `device_type_id`, `location_id`, `metric_name` | Baseline retrieval |

---

*Diagrams and documentation created: 2026-01-06*
*Format: Mermaid (renders in GitHub, VSCode, most markdown viewers)*
