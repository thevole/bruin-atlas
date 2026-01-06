# Foundation Implementation Exercises

**Purpose:** Productive work the team can execute immediately while business decisions on feature priorities are finalized. Each exercise builds infrastructure required by ALL proposed features.

**Criteria for inclusion:**
- Valuable regardless of which Phase 2 features ship
- Demonstrates team capability quickly
- No business decision dependencies
- Clear definition of done

---

## Exercise 1: Core Data Model & Schema

**Duration:** 1-2 weeks
**Team:** Platform (2 engineers)
**Dependencies:** None

### Why This Matters (For Any Feature)

| Feature | Schema Dependency |
|---------|-------------------|
| Compliance Reporting | CUSTOMER, NETWORK, DEVICE, COMPLIANCE_REPORT, FRAMEWORK |
| Behavior Baselines | DEVICE, DEVICE_TYPE, HEALTH_SIGNAL, BASELINE_PROFILE |
| Change Simulation | NETWORK topology_data, DEVICE relationships |
| Import Wizard | All core entities |
| Health Dashboard | HEALTH_SIGNAL, INCIDENT |

### Deliverables

1. **PostgreSQL schema migration scripts**
   - Core entities: CUSTOMER, NETWORK, LOCATION, DEVICE, DEVICE_TYPE
   - Health entities: HEALTH_SIGNAL, INCIDENT
   - Extensible JSON columns for topology_data, config_snapshot
   - Proper indexes for common query patterns

2. **Database seed scripts**
   - Sample customer with realistic network topology
   - 50-100 devices across 5-10 locations
   - Historical health signals (30 days)
   - Sample incidents (open and resolved)

3. **Documentation**
   - ERD diagram (update Diagrams/01-platform-architecture.md)
   - Column-level documentation
   - Query examples for common access patterns

### Definition of Done
- [ ] Migrations run cleanly on fresh PostgreSQL instance
- [ ] Seed data creates realistic test environment
- [ ] Query performance <100ms for topology retrieval
- [ ] Schema review completed with Tech Lead

---

## Exercise 2: Rule Engine Foundation

**Duration:** 2-3 weeks
**Team:** Network Model (1 engineer) + Platform (1 engineer)
**Dependencies:** Exercise 1 (schema)

### Why This Matters (For Any Feature)

| Feature | Rule Engine Usage |
|---------|-------------------|
| Compliance Reporting | Evaluate compliance controls against network state |
| Behavior Baselines | Contextualize deviations with rule conditions |
| Change Simulation | Run rules against hypothetical topology state |
| Health Scoring | Aggregate rule outputs into health scores |

### Deliverables

1. **Rule definition schema (JSON)**
   ```json
   {
     "rule_id": "redundancy_core_router",
     "name": "Core Router Redundancy",
     "conditions": [...],
     "severity": "critical",
     "remediation": "Add redundant core router"
   }
   ```

2. **Rule evaluation engine**
   - Parse rule definitions
   - Evaluate conditions against device/network state
   - Support operators: equals, gt, lt, contains, exists, baseline_deviation
   - Return structured results with affected entities

3. **Initial rule library (10-15 rules)**
   - Redundancy rules (single point of failure detection)
   - Basic health rules (device reachability, response time)
   - Configuration rules (firmware version, security settings)

4. **Rule testing framework**
   - Unit tests for each rule
   - Test fixtures with pass/fail scenarios

### Definition of Done
- [ ] Engine evaluates rules against seed data correctly
- [ ] All 10-15 rules have passing tests
- [ ] Evaluation completes <500ms for full network scan
- [ ] Rule schema documented with examples

---

## Exercise 3: REST API Scaffold

**Duration:** 1-2 weeks
**Team:** Platform (2 engineers)
**Dependencies:** Exercise 1 (schema)

### Why This Matters (For Any Feature)

Every feature requires API endpoints. Building the scaffold now means features plug in cleanly later.

### Deliverables

1. **API framework setup**
   - Express/Fastify/NestJS (team choice)
   - OpenAPI/Swagger documentation
   - Request validation middleware
   - Error handling standardization
   - Logging and request tracing

2. **Core CRUD endpoints**
   ```
   GET/POST   /api/v1/customers
   GET/PUT    /api/v1/customers/:id
   GET        /api/v1/customers/:id/networks
   GET        /api/v1/networks/:id
   GET        /api/v1/networks/:id/devices
   GET        /api/v1/devices/:id
   GET        /api/v1/devices/:id/health
   ```

3. **Authentication scaffold**
   - JWT token validation middleware
   - API key authentication option
   - Role extraction (for future RBAC)

4. **Integration test suite**
   - Tests for each endpoint
   - Test database setup/teardown
   - CI pipeline integration

### Definition of Done
- [ ] All endpoints return correct data from seed database
- [ ] OpenAPI spec generates valid documentation
- [ ] Auth middleware blocks unauthenticated requests
- [ ] Test coverage >80% for API routes

---

## Exercise 4: Topology Graph Model

**Duration:** 2 weeks
**Team:** Platform (1 engineer) + UI (1 engineer)
**Dependencies:** Exercise 1 (schema)

### Why This Matters (For Any Feature)

| Feature | Topology Usage |
|---------|----------------|
| Change Simulation | Graph traversal for impact analysis |
| Health Dashboard | Hierarchical navigation (customer→network→location→device) |
| Compliance | Network segmentation evidence |
| Incident Tracking | Affected device identification |

### Deliverables

1. **Graph data structure**
   - Nodes: locations, devices
   - Edges: physical connections, logical relationships
   - Support for hierarchical queries (parent/child)
   - Efficient path finding between nodes

2. **Graph query API**
   ```typescript
   getChildren(nodeId): Node[]
   getPath(fromId, toId): Node[]
   getAffectedNodes(nodeId): Node[]  // downstream impact
   getRedundantPaths(nodeId): Path[]
   ```

3. **Topology diff utility**
   - Compare two topology states
   - Identify added/removed/modified nodes and edges
   - Foundation for change detection

4. **Visualization data format**
   - JSON structure optimized for D3.js/Cytoscape rendering
   - Support for hierarchical and force-directed layouts

### Definition of Done
- [ ] Graph loads from PostgreSQL topology_data
- [ ] Path queries return correct results
- [ ] Diff utility detects changes in test scenarios
- [ ] Visualization JSON renders in sample D3 component

---

## Exercise 5: Health Signal Pipeline

**Duration:** 2 weeks
**Team:** Integration (2 engineers)
**Dependencies:** Exercise 1 (schema)

### Why This Matters (For Any Feature)

| Feature | Health Signal Usage |
|---------|---------------------|
| Behavior Baselines | Historical signals for baseline calculation |
| Health Dashboard | Real-time health display |
| Compliance | SLA/uptime evidence |
| Incident Detection | Threshold-based alerting |

### Deliverables

1. **Signal ingestion service**
   - Accept health signals via REST API
   - Validate signal schema
   - Write to PostgreSQL + Time Series DB
   - Add data trust metadata (timestamp, source, confidence)

2. **Signal aggregation queries**
   ```sql
   -- Device health over time
   -- Location-level rollup
   -- Network-level summary
   -- Trend calculation
   ```

3. **Mock data generator**
   - Simulate realistic health signals
   - Configurable patterns (normal, degraded, incident)
   - Support continuous generation for testing

4. **Time Series integration**
   - TimescaleDB or InfluxDB setup
   - Retention policies (raw: 7 days, aggregated: 90 days)
   - Downsampling queries

### Definition of Done
- [ ] Pipeline ingests 1000 signals/second without backpressure
- [ ] Aggregation queries return in <200ms
- [ ] Mock generator produces realistic patterns
- [ ] Data automatically rolls up and ages out

---

## Exercise 6: UI Component Library & Topology Viewer Prototype

**Duration:** 2-3 weeks
**Team:** UI (2 engineers)
**Dependencies:** Exercise 4 (topology model)

### Why This Matters (For Any Feature)

| Feature | UI Component Usage |
|---------|---------------------|
| Health Dashboard | Health score cards, trend charts, status indicators |
| Topology Viewer | Network graph, device details, location hierarchy |
| Import Wizard | Form components, progress indicators, validation |
| Simulation | Topology with hypothetical state overlay |

### Deliverables

1. **Design system foundation**
   - Color palette (health states: green/yellow/red)
   - Typography scale
   - Spacing system
   - Component variants (default, loading, error, empty)

2. **Core UI components**
   - HealthScoreCard (score, trend, confidence indicator)
   - DeviceStatusBadge (online/offline/degraded/unknown)
   - DataFreshnessIndicator (last updated timestamp)
   - TrustLevelBadge (verified/inferred/stale)
   - SearchableDropdown
   - DataTable with sorting/filtering

3. **Topology viewer prototype**
   - Render network graph from Exercise 4 JSON
   - Zoom, pan, fit-to-screen
   - Click to select node
   - Highlight connected nodes on selection
   - Basic search/filter

4. **Storybook documentation**
   - All components documented with examples
   - Interactive props playground
   - Usage guidelines

### Definition of Done
- [ ] Components render correctly across browsers
- [ ] Topology viewer handles 500+ nodes smoothly
- [ ] Storybook documents all components
- [ ] Accessibility audit passes (keyboard nav, screen reader)

---

## Exercise Dependencies & Sequencing

```
Week 1-2:     [Exercise 1: Schema] ────────────────────────────────┐
                      │                                            │
Week 2-3:     [Exercise 3: API Scaffold]                           │
                      │                                            │
Week 2-4:     [Exercise 2: Rule Engine] ───────────────────────────┤
                      │                                            │
Week 3-4:     [Exercise 4: Topology Graph] ────────────────────────┤
                      │                                            │
Week 3-5:     [Exercise 5: Health Pipeline] ───────────────────────┤
                      │                                            │
Week 4-6:     [Exercise 6: UI Components] ─────────────────────────┘
                                                                   │
Week 6+:      ──────────── PHASE 2 FEATURES BEGIN ─────────────────┘
```

**Parallel Tracks:**
- **Track A (Platform):** Schema → API → Topology Graph
- **Track B (Intelligence):** Schema → Rule Engine
- **Track C (Integration):** Schema → Health Pipeline
- **Track D (UI):** Topology Graph → UI Components

---

## Team Allocation

| Exercise | Platform (2) | Network Model (1) | UI (2) | Integration (3) |
|----------|--------------|-------------------|--------|-----------------|
| 1. Schema | **Lead** | Consult | - | Support |
| 2. Rule Engine | Support | **Lead** | - | - |
| 3. API Scaffold | **Lead** | - | - | Support |
| 4. Topology Graph | **Lead** | Consult | Support | - |
| 5. Health Pipeline | - | - | - | **Lead** |
| 6. UI Components | - | - | **Lead** | - |

---

## Success Metrics

At the end of 6 weeks, the team should demonstrate:

1. **Working API** returning real data from PostgreSQL
2. **Rule Engine** evaluating 10+ rules against sample network
3. **Topology Viewer** rendering sample network with interaction
4. **Health Pipeline** ingesting and aggregating signals
5. **Component Library** documented in Storybook

**Business Value:** When Phase 2 feature decisions are made, implementation begins immediately on solid foundation rather than starting from scratch.

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Schema changes when features finalize | Use migrations; design for extensibility with JSON columns |
| Rule format changes | Version rule schema; maintain backwards compatibility |
| API contract changes | Use OpenAPI; version endpoints (/v1/) |
| UI component scope creep | Strict Storybook-first development; defer feature-specific components |

---

*Document created: 2026-01-06*
*Status: Ready for team kickoff*
