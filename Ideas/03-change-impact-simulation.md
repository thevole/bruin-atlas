# Change Impact Simulation (Scoped MVP)

## 1. Executive Summary

Change Impact Simulation enables NOC engineers to model infrastructure changes before execution by running "what-if" analysis against the live topology. Starting with device removal scenarios, this feature leverages existing topology visualization and health rules to predict redundancy impacts, affected paths, and downstream health effects—differentiating Atlas from observe-only competitors.

## 2. Problem Statement

**Current Pain Points:**

- NOC engineers execute maintenance changes (device removals, link cuts, firmware upgrades) without visibility into downstream impact
- Post-change incidents often stem from unrecognized dependencies or single points of failure
- Teams rely on tribal knowledge and manual topology tracing, which doesn't scale
- Rollback decisions happen reactively after outages, not proactively before changes

**User Need:** Before executing a change, engineers need a confident answer to: *"If I remove/disable device X, what breaks?"*

## 3. Proposed Solution

A scoped "what-if" analysis engine that:

1. Accepts a hypothetical change (MVP: device removal)
2. Creates a transient modified topology state
3. Runs existing health rules against that hypothetical state
4. Visualizes affected paths, redundancy gaps, and health score deltas
5. Does NOT execute any actual changes

**Key Insight:** Simulation = applying existing health/rules logic to a modified graph state. We're not building a network simulator; we're reusing what we have.

## 4. Leveraged Assets

| Asset | How It's Used |
|-------|---------------|
| **Topology Model** | Graph structure enables path traversal and dependency analysis |
| **Health Rules Engine** | Existing scoring logic runs against hypothetical state |
| **Reference Architecture** | Defines expected redundancy patterns to check against |
| **Device Registry** | Provides device metadata for impact categorization |
| **Path Visualization** | Existing rendering code shows affected routes |

## 5. User Stories

### US-1: Pre-Maintenance Impact Check
> As a NOC engineer, I want to simulate removing a core switch before maintenance, so I can identify which services lose redundancy and notify affected teams.

**Acceptance Criteria:**
- Select device from topology or search
- Click "Simulate Removal"
- See list of affected paths with criticality
- See health score delta for affected segments

### US-2: Redundancy Validation
> As a network architect, I want to verify that removing any single device doesn't create a single point of failure, so I can validate our redundancy design.

**Acceptance Criteria:**
- Batch simulation mode for "remove each device, one at a time"
- Report showing which removals cause critical path failures
- Export results for architecture review

### US-3: Change Approval Evidence
> As a change manager, I want simulation results attached to change requests, so I have evidence that impact was analyzed before approval.

**Acceptance Criteria:**
- Simulation results exportable as PDF/JSON
- Timestamp and topology snapshot version included
- Linkable from external ITSM tools

### US-4: Quick Path Check
> As a NOC engineer during an incident, I want to quickly see if disabling a misbehaving device will isolate critical paths, so I can make fast containment decisions.

**Acceptance Criteria:**
- Simulation completes in <3 seconds for single device
- Clear red/yellow/green impact summary
- One-click from device context menu

### US-5: Training Scenarios
> As a new team member, I want to run "what-if" scenarios on production topology (read-only), so I can learn network dependencies without risk.

**Acceptance Criteria:**
- Simulation mode clearly marked as "no changes made"
- Sharable simulation URLs for team discussion

## 6. Technical Approach

### Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   UI: Simulate  │────>│ Simulation Engine │────>│ Impact Renderer │
│   Remove Device │     │                  │     │                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              ┌─────▼─────┐        ┌──────▼──────┐
              │ Topology  │        │ Health Rules │
              │   Clone   │        │   Engine     │
              └───────────┘        └──────────────┘
```

### Core Components

**1. Topology Clone Service**
- Creates in-memory copy of relevant topology subgraph
- Applies hypothetical modification (device removal)
- Immutable—never touches production state

**2. Path Impact Analyzer**
- Traverses graph from removed node
- Identifies all paths that included removed device
- Checks for alternative paths (redundancy validation)
- Outputs: affected paths, now-critical paths, isolated segments

**3. Health Delta Calculator**
- Runs health rules against modified topology
- Compares scores: before vs. after
- Flags rules that transition from pass to fail

**4. Impact Visualization**
- Highlights affected paths on existing topology view
- Color-codes by severity (red = no redundancy, yellow = degraded, green = redundant path exists)
- Summary panel with metrics

### Data Flow

1. User selects device, clicks "Simulate Removal"
2. Frontend sends `POST /api/simulate` with `{deviceId, action: "remove"}`
3. Backend clones topology subgraph (2-hop radius from device)
4. Removes device from clone
5. Runs path analysis on clone
6. Runs health rules on clone
7. Returns `{affectedPaths[], healthDeltas[], redundancyStatus}`
8. Frontend renders overlay on topology

### Performance Targets

| Metric | Target |
|--------|--------|
| Single device simulation | <3 seconds |
| Batch simulation (10 devices) | <30 seconds |
| Memory overhead per simulation | <50MB |

## 7. MVP Scope

### In Scope (MVP)

- **Single action type:** Device removal only
- **Single device:** One device per simulation
- **Path impact:** Show all paths affected by removal
- **Redundancy check:** Binary—alternative path exists or not
- **Health delta:** Show rules that would fail post-change
- **Visualization:** Overlay on existing topology view
- **Export:** JSON format for change documentation

### Explicitly Out of Scope (Future)

| Feature | Why Deferred |
|---------|--------------|
| Link removal/degradation | Adds complexity; device removal covers 80% of use cases |
| Multi-device simulation | Combinatorial explosion; need to prove single-device first |
| Traffic simulation | Requires flow data we don't have; different problem domain |
| Automated remediation suggestions | AI/ML scope creep |
| Real-time simulation during changes | Requires event streaming infrastructure |
| Configuration change simulation | Needs config parsing; device removal is topology-only |
| Historical simulation ("what if we had...") | Requires topology versioning; future feature |

### MVP Success Definition

A NOC engineer can:
1. Right-click any device in topology
2. Select "Simulate Removal"
3. See affected paths highlighted within 3 seconds
4. See clear indication if any path loses all redundancy
5. Export results to attach to change ticket

## 8. Success Metrics

### Primary Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Adoption | 50% of change tickets include simulation | ITSM integration tracking |
| Pre-change incidents | 30% reduction | Incident correlation with changes |
| Simulation usage | 100+ simulations/week | Platform analytics |

### Secondary Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to simulate | <10 seconds end-to-end | Frontend timing |
| User satisfaction | >4.0/5.0 rating | In-app feedback |
| False positives | <10% of flagged impacts | User feedback/override tracking |

### Validation Approach

1. **Alpha:** 2-week pilot with 3 NOC engineers, daily feedback
2. **Beta:** 4-week rollout to full NOC team, weekly metrics review
3. **GA:** Measure against targets, iterate based on data

## 9. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Topology model incomplete** | Medium | High | Audit graph traversability before starting; block on topology completeness |
| **Scope creep to full simulation** | High | High | Strict MVP definition; PM gatekeeping; "not now" list visible |
| **Performance at scale** | Medium | Medium | Subgraph cloning (not full topology); lazy loading; caching |
| **False confidence from simulation** | Medium | High | Clear disclaimers; show confidence level; highlight what's NOT modeled |
| **Health rules not designed for hypotheticals** | Low | Medium | Audit rules for side effects; sandbox rule execution |
| **User expects real network simulator** | Medium | Medium | Clear UX messaging; "Impact Analysis" not "Simulation" in UI |

### Technical Debt Considerations

- Topology model must be fully graph-traversable (prerequisite)
- Health rules must be pure functions (no side effects)
- Clone mechanism must be memory-efficient for large topologies

## 10. Team Allocation Recommendation

### Proposed Allocation (8 engineers total)

| Team | Engineers | Role in Feature |
|------|-----------|-----------------|
| **Platform** | 1 of 2 | Simulation engine, topology clone service |
| **Network Model** | 1 of 1 | Path analysis, redundancy detection |
| **UI** | 1 of 2 | Simulation UX, impact visualization overlay |
| **Integration** | 1 of 3 | Export format, ITSM linkage |

**Total dedicated:** 4 engineers (50% of team)

### Phase Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| **Phase 1: Foundation** | 2 weeks | Topology clone service, basic path traversal |
| **Phase 2: Core** | 3 weeks | Impact analysis, health delta, API |
| **Phase 3: UX** | 2 weeks | Visualization, export, polish |
| **Phase 4: Validation** | 1 week | Alpha testing, bug fixes |

**Total MVP timeline:** 8 weeks

### Dependencies

- Topology model graph-traversability: Must be complete before Phase 1
- Health rules audit: Complete during Phase 1
- Design review: Complete before Phase 3

---

## Appendix: Competitive Differentiation

Most network monitoring tools are **observe-only**:
- SolarWinds: Monitors, alerts, visualizes—no simulation
- Datadog Network Monitoring: Traffic analysis—no topology simulation
- Kentik: Flow analysis—no change impact modeling

**Atlas differentiation:** Move from reactive monitoring to **proactive change planning**. This positions Atlas as a tool for **operations confidence**, not just visibility.

---

*Document Version: 1.0*
*Last Updated: 2026-01-06*
*Status: Draft for Review*
