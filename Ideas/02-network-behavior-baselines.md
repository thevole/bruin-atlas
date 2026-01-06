# Network Behavior Baselines with Deviation Alerts

## Executive Summary

Network behavior baselines establish what "normal" looks like for each customer's unique network environment, enabling Bruin Atlas to detect meaningful deviations rather than relying on arbitrary thresholds. By combining statistical baseline analysis with our existing rule engine, we transform reactive monitoring into proactive alerting—catching anomalies before they become incidents.

---

## Problem Statement

Static health scores provide a snapshot but lack contextual meaning:

- **"80% health" is meaningless without context** — Is this normal for this device? This location? This time of day?
- **One-size-fits-all thresholds miss the mark** — A 5% CPU spike might be critical for a core router but routine for an edge switch during business hours
- **Reactive monitoring fails customers** — By the time a static threshold triggers, the incident is already impacting users
- **Customer networks are snowflakes** — Each environment has unique traffic patterns, device behaviors, and operational rhythms that generic thresholds cannot capture

Current state: We alert when metrics cross fixed thresholds. Desired state: We alert when behavior deviates from established patterns—the actual signal of something wrong.

---

## Proposed Solution

Build a statistical baseline engine that:

1. **Learns normal behavior** per device type, location, and time window using simple statistical methods (no ML required)
2. **Detects deviations** from established baselines using standard deviation thresholds
3. **Integrates with rule engine** to correlate baseline deviations with existing rules for richer alerting
4. **Provides baseline context** in health scores and dashboards ("Currently 15% above typical for this time")

### Core Capabilities

| Capability | Description |
|------------|-------------|
| Baseline Profiles | Per-device-type statistical profiles (mean, std dev, percentiles) |
| Time-Aware Baselines | Separate profiles for business hours, off-hours, weekends |
| Location Segmentation | Baselines scoped to site/region for meaningful comparison |
| Deviation Detection | Configurable sigma thresholds (default: 2σ = warning, 3σ = alert) |
| Rule Engine Integration | Baseline deviations feed into existing rule conditions |
| Dashboard Context | Health scores annotated with baseline comparison |

---

## Leveraged Assets

This feature builds heavily on existing Bruin Atlas infrastructure:

### Rule Engine
- Extend condition types to include `baseline_deviation` operators
- Baseline alerts flow through existing notification pipeline
- Combine with existing rules: "IF baseline_deviation > 2σ AND device_role = 'core' THEN escalate"

### Health Scoring System
- Baseline context enriches existing health calculations
- Transform raw scores into relative scores ("80% health, 10% below baseline")
- Historical baseline trends feed into health decay algorithms

### DeviceAPI
- Device type and location metadata already available for segmentation
- Extend device endpoints to return baseline profiles
- Leverage existing metric collection infrastructure

### Data Pipeline
- Existing time-series data provides baseline training input
- No new data collection required—just new aggregation queries
- Historical data enables immediate baseline calculation for existing customers

---

## User Stories

### US-1: NOC Operator Catches Anomaly Early
> As a NOC operator, I want to receive alerts when device behavior deviates significantly from its baseline, so I can investigate potential issues before they impact users.

**Acceptance Criteria:**
- Alert triggers when metric exceeds 2σ from baseline
- Alert includes baseline context (expected vs actual)
- Alert links to device detail page with baseline visualization

### US-2: Network Engineer Understands "Normal"
> As a network engineer, I want to view baseline profiles for devices in my region, so I can understand normal operational patterns and set appropriate expectations.

**Acceptance Criteria:**
- Dashboard shows baseline bands (expected range) overlaid on metric graphs
- Can filter baselines by device type, location, time window
- Export baseline data for capacity planning

### US-3: Platform Admin Configures Sensitivity
> As a platform admin, I want to configure baseline deviation thresholds per device type, so I can tune alerting sensitivity to match our operational tolerance.

**Acceptance Criteria:**
- Configure σ thresholds (warning/critical) per device type
- Configure minimum baseline age before alerting enabled
- Configure metrics included in baseline calculations

### US-4: Incident Responder Gets Context
> As an incident responder, I want to see how current behavior compares to baseline during an incident, so I can quickly assess severity and scope.

**Acceptance Criteria:**
- Incident detail shows baseline deviation for affected devices
- Deviation history shows when anomaly started
- Compare current behavior to same time last week/month

### US-5: Customer Success Reviews Trends
> As a customer success manager, I want to show customers their network's baseline stability over time, so I can demonstrate the value of proactive monitoring.

**Acceptance Criteria:**
- Report shows baseline stability score (fewer deviations = more stable)
- Trend chart of baseline deviations over 30/60/90 days
- Comparison to similar customer profiles (anonymized)

---

## Technical Approach

### Statistical Methods (No ML)

We use straightforward statistical techniques that are transparent, debuggable, and don't require data science expertise:

```
Baseline Calculation:
- Rolling window: 30 days of metric samples
- Aggregation: hourly buckets, segmented by day-type (weekday/weekend)
- Statistics: mean, standard deviation, P5/P50/P95 percentiles
- Update frequency: Daily recalculation with sliding window
```

### Deviation Detection Algorithm

```python
# Pseudocode for deviation detection
def detect_deviation(current_value, baseline):
    z_score = (current_value - baseline.mean) / baseline.std_dev

    if abs(z_score) > 3.0:
        return CRITICAL
    elif abs(z_score) > 2.0:
        return WARNING
    else:
        return NORMAL
```

### Segmentation Strategy

| Dimension | Segments | Rationale |
|-----------|----------|-----------|
| Device Type | router, switch, firewall, access_point, etc. | Different device types have fundamentally different behavior patterns |
| Location | site_id, region | Local patterns (time zones, usage) dominate |
| Time Window | business_hours, off_hours, weekend | Traffic patterns vary dramatically by time |
| Metric Type | cpu, memory, bandwidth, latency, errors | Each metric has different characteristics |

### Data Model

```sql
-- Baseline profile storage
CREATE TABLE baseline_profiles (
    id UUID PRIMARY KEY,
    device_type VARCHAR(50),
    location_id UUID,
    metric_name VARCHAR(100),
    time_segment VARCHAR(20),  -- 'business_hours', 'off_hours', 'weekend'

    -- Statistical values
    mean DECIMAL,
    std_dev DECIMAL,
    p5 DECIMAL,
    p50 DECIMAL,
    p95 DECIMAL,
    sample_count INTEGER,

    -- Metadata
    window_start TIMESTAMP,
    window_end TIMESTAMP,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

### Rule Engine Integration

Extend rule condition DSL:

```yaml
# Example rule using baseline deviation
rule:
  name: "Core Router Baseline Deviation"
  conditions:
    - metric: cpu_utilization
      operator: baseline_deviation_exceeds
      threshold: 2.5  # sigma
    - device_role: core
  actions:
    - alert:
        severity: warning
        message: "Core router CPU {deviation}σ above baseline"
```

---

## Success Metrics

### Primary Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Mean Time to Detection (MTTD) | 40% reduction | Compare detection time for incidents with/without baselines |
| False Positive Rate | < 5% of baseline alerts | Alerts that don't require action within 24h |
| Baseline Coverage | > 80% of monitored devices | Devices with mature (30+ day) baselines |

### Secondary Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Customer Adoption | 60% enable within 90 days | Customers with baseline alerting configured |
| Baseline Stability | > 70% of devices "stable" | Devices with < 3 deviations/week |
| Rule Enhancement | 30% of rules use baselines | Rules incorporating baseline_deviation conditions |

### Qualitative Success

- NOC teams report earlier incident awareness
- Customers cite baselines in renewal conversations
- Reduction in "why didn't we catch this?" post-mortems

---

## Risks & Mitigations

### Risk 1: Cold Start Problem
**Risk:** New customers receive no baseline value for 30+ days.
**Impact:** High — poor initial experience.
**Mitigation:**
- Use device-type defaults from anonymized aggregate baselines
- Clearly communicate "baseline learning" status in UI
- Provide value through rule engine immediately; baselines enhance over time

### Risk 2: Baseline Drift
**Risk:** Gradual changes become the new normal, masking degradation.
**Impact:** Medium — miss slow-burn issues.
**Mitigation:**
- Implement "baseline of baselines" — alert if baseline itself shifts significantly
- Configurable drift detection thresholds
- Periodic baseline reset option for known-good states

### Risk 3: Alert Fatigue
**Risk:** Too many baseline deviation alerts overwhelm operators.
**Impact:** High — users disable feature.
**Mitigation:**
- Conservative default thresholds (start at 3σ)
- Correlation with rule engine suppresses redundant alerts
- Anomaly grouping — single alert for related deviations

### Risk 4: Storage/Compute Costs
**Risk:** Historical data and calculations strain infrastructure.
**Impact:** Medium — cost overruns.
**Mitigation:**
- Aggregate storage (hourly buckets, not raw samples)
- Off-peak baseline recalculation
- Tiered retention (detailed recent, summarized historical)

### Risk 5: Customer Misinterpretation
**Risk:** Customers don't understand statistical concepts.
**Impact:** Medium — support burden.
**Mitigation:**
- Plain language in UI ("unusually high" vs "2.3σ deviation")
- Guided onboarding explaining baseline concepts
- Documentation and training materials

---

## Team Allocation Recommendation

**Total effort estimate:** 6-8 weeks with parallel workstreams

### Phase 1: Foundation (Weeks 1-3)

| Team | Allocation | Focus |
|------|------------|-------|
| Platform | 1.5 engineers | Baseline calculation pipeline, data model, storage |
| Network Model | 0.5 engineer | Device type segmentation logic, metric selection |
| Integration | 1 engineer | DeviceAPI extensions for baseline data |

### Phase 2: Detection & Rules (Weeks 3-5)

| Team | Allocation | Focus |
|------|------------|-------|
| Platform | 1 engineer | Deviation detection engine, alerting integration |
| Network Model | 0.5 engineer | Rule engine baseline condition extensions |
| Integration | 1 engineer | Notification pipeline enhancements |

### Phase 3: UI & Polish (Weeks 5-7)

| Team | Allocation | Focus |
|------|------------|-------|
| UI | 2 engineers | Baseline visualizations, dashboard context, configuration UI |
| Integration | 1 engineer | API documentation, customer-facing endpoints |

### Phase 4: Rollout (Week 7-8)

| Team | Allocation | Focus |
|------|------------|-------|
| Platform | 0.5 engineer | Performance tuning, monitoring |
| UI | 0.5 engineer | Onboarding flow, help content |
| Integration | 1 engineer | Beta customer support, feedback integration |

### Resource Summary

| Team | Total Allocation | Notes |
|------|-----------------|-------|
| Platform (2) | ~3 engineer-weeks | Core baseline infrastructure |
| Network Model (1) | ~1 engineer-week | Domain expertise for segmentation |
| UI (2) | ~2.5 engineer-weeks | Visualization and configuration |
| Integration (3) | ~3.5 engineer-weeks | API, notifications, rollout support |

---

## Appendix: Key Design Decisions

1. **Statistical over ML** — Transparency, debuggability, no specialized expertise required
2. **Device-type segmentation** — Balance granularity with sample size for statistical validity
3. **30-day minimum window** — Statistical significance without excessive cold start
4. **Rule engine integration** — Leverage existing investment, enable powerful combinations
5. **Conservative defaults** — Prefer false negatives to alert fatigue during rollout
