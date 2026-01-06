# Automated Compliance & Audit Reporting for Bruin Atlas

## Executive Summary

Automated Compliance Reporting transforms Bruin Atlas's existing network monitoring data into audit-ready compliance reports for SOC2, ISO 27001, and enterprise security frameworks. This premium-tier feature eliminates manual evidence gathering while positioning Atlas as essential infrastructure for enterprise GRC (Governance, Risk, Compliance) workflows.

---

## Problem Statement

Enterprise customers face significant compliance burden:

- **Manual Evidence Collection**: Network teams spend 40-80 hours per audit cycle gathering device configs, uptime reports, and security posture documentation
- **Audit Fatigue**: SOC2, ISO 27001, PCI-DSS, HIPAA all require network infrastructure evidence - often redundant data formatted differently
- **Risk of Non-Compliance**: Manual processes introduce human error; missing evidence can delay certifications or trigger findings
- **Renewal Blocking**: Enterprises increasingly mandate compliance automation in vendor RFPs - lack of this feature is a competitive disadvantage
- **Point-in-Time vs Continuous**: Auditors want continuous compliance evidence, not snapshots assembled under deadline pressure

**Market Reality**: Competitors (DataDog, Splunk, ServiceNow) offer compliance modules. For enterprise renewals/expansions, this is table stakes.

---

## Proposed Solution

### Core Capabilities

1. **Compliance Framework Mapping**
   - Pre-built mappings: SOC2 Type II, ISO 27001, PCI-DSS, NIST CSF
   - Custom framework builder for internal policies
   - Control-to-evidence linking (e.g., "CC6.1 Network Security" → device firewall rules, topology isolation)

2. **Automated Report Generation**
   - Scheduled reports (daily/weekly/monthly/quarterly)
   - On-demand generation for ad-hoc audits
   - Multiple export formats: PDF, CSV, JSON, direct integrations

3. **Evidence Collection Engine**
   - Device configuration snapshots with change tracking
   - Health score trends over reporting period
   - Incident response timelines with resolution metrics
   - Data trust indicators as data quality evidence

4. **Audit Trail Dashboard**
   - Real-time compliance posture view
   - Gap identification with remediation suggestions
   - Historical trend analysis for continuous improvement

5. **Auditor Access Portal** (Premium+)
   - Read-only external access for auditors
   - Scoped views per audit engagement
   - Direct evidence download without customer intervention

---

## Leveraged Assets

This feature maximizes existing platform investments:

| Existing Asset | Compliance Use Case |
|----------------|---------------------|
| **Rule Engine** | Define compliance rules (e.g., "all edge devices must have firewall enabled"), auto-evaluate against device fleet |
| **MonitoringAPI** | Source uptime/availability metrics, health score history, incident data for SLA evidence |
| **DeviceAPI** | Pull device configs, firmware versions, security settings for configuration compliance |
| **Topology Data** | Network segmentation evidence, isolation verification, change detection |
| **Data Trust Indicators** | Data quality/completeness metrics as control evidence |
| **Incident Tracking** | Response time metrics, escalation evidence, resolution documentation |

**Implementation Efficiency**: ~70% of required data already collected; primary work is transformation layer and report templates.

---

## User Stories

### US-1: Compliance Manager - Scheduled SOC2 Reports
> As a **Compliance Manager**, I want to schedule automated monthly SOC2 evidence reports so that I can maintain continuous audit readiness without manual data gathering.

**Acceptance Criteria:**
- Configure report schedule (frequency, recipients, format)
- Map Atlas data to SOC2 control categories
- Receive automated email with attached PDF/CSV
- View report history in dashboard

### US-2: Network Engineer - Pre-Audit Gap Check
> As a **Network Engineer**, I want to run a compliance gap analysis before audit season so that I can remediate issues proactively.

**Acceptance Criteria:**
- Select target framework (SOC2, ISO, custom)
- View pass/fail status per control
- Drill down to specific devices/configs causing gaps
- Export gap report for remediation planning

### US-3: IT Director - Executive Compliance Dashboard
> As an **IT Director**, I want a real-time compliance posture dashboard so that I can report status to leadership without waiting for report generation.

**Acceptance Criteria:**
- Single-pane compliance score across frameworks
- Trend visualization (improving/degrading)
- Alert on score drops below threshold
- Shareable link for executive briefings

### US-4: External Auditor - Evidence Access
> As an **External Auditor**, I want self-service access to scoped network evidence so that I can complete audit procedures efficiently without burdening the customer team.

**Acceptance Criteria:**
- Customer creates time-limited auditor account
- Auditor sees only relevant evidence (not full platform)
- Direct evidence download (no customer middleman)
- Audit log of auditor access for customer visibility

### US-5: Security Team - Custom Framework Definition
> As a **Security Team Lead**, I want to define custom compliance frameworks based on internal policies so that I can track adherence to company-specific requirements.

**Acceptance Criteria:**
- Create named framework with custom controls
- Map Atlas data sources to each control
- Set pass/fail thresholds per control
- Include custom frameworks in scheduled reports

---

## Technical Approach

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Compliance Service                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Framework  │  │   Report    │  │    Evidence Collector   │  │
│  │   Registry  │  │  Generator  │  │   (scheduled worker)    │  │
│  └──────┬──────┘  └──────┬──────┘  └───────────┬─────────────┘  │
│         │                │                      │                │
│         └────────────────┴──────────────────────┘                │
│                          │                                       │
├──────────────────────────┼───────────────────────────────────────┤
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Compliance Data Store (PostgreSQL)              │ │
│  │  • Framework definitions    • Evidence snapshots            │ │
│  │  • Control mappings         • Report history                │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Rule Engine   │  │  MonitoringAPI  │  │    DeviceAPI    │
│  (existing)     │  │   (existing)    │  │   (existing)    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### Key Components

1. **Framework Registry**: JSON schema definitions for SOC2, ISO, etc. with control→data mappings
2. **Evidence Collector**: Scheduled job aggregating data from existing APIs into compliance snapshots
3. **Report Generator**: Template engine producing PDF/CSV/JSON from evidence + framework rules
4. **Compliance API**: REST endpoints for dashboard, gap analysis, report scheduling
5. **Auditor Portal**: Scoped read-only UI with separate auth flow

### Data Flow

1. Evidence Collector runs on schedule (configurable, default: nightly)
2. Pulls current state from MonitoringAPI, DeviceAPI, Rule Engine
3. Evaluates against Framework Registry control definitions
4. Stores snapshots in Compliance Data Store
5. Report Generator consumes snapshots to produce formatted reports
6. Dashboard queries latest snapshot for real-time view

### Integration Points

- **Existing Auth**: Extend RBAC for compliance roles (Compliance Admin, Auditor)
- **Existing Notifications**: Use alert infrastructure for compliance score drops
- **Export**: Integrate with existing export functionality (PDF generation, email delivery)

---

## Success Metrics

### Business Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Premium tier conversion | 15% of enterprise customers in Y1 | Stripe/billing data |
| Enterprise renewal rate | +5% improvement | CRM renewal tracking |
| New logo win rate (enterprise) | +10% in RFP success | Sales pipeline data |
| Upsell revenue | $500K ARR in Y1 | Revenue attribution |

### Product Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Reports generated/month | 1000+ across customers | Internal telemetry |
| Avg time-to-first-report | <30 min from feature enable | Onboarding funnel |
| Compliance score improvement | +15% avg over 6 months | Trend analysis |
| Auditor portal sessions | 50+ external users in Y1 | Auth logs |

### Operational Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Report generation latency | <5 min for standard report | Job duration logs |
| Evidence collector success rate | >99.5% | Job monitoring |
| Support tickets (compliance) | <10/month after launch | Zendesk tagging |

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Framework accuracy**: Incorrect control mappings create audit risk | Medium | High | Partner with GRC consultant for validation; customer-editable mappings as escape hatch |
| **Scope creep**: Compliance is infinitely complex | High | Medium | MVP = SOC2 + ISO only; custom frameworks as fast-follow; explicit "not supported" list |
| **Performance**: Large evidence queries slow platform | Medium | Medium | Separate read replica for compliance queries; aggressive caching; async report generation |
| **Security**: Auditor portal access misconfigured | Low | High | Default deny; explicit scoping required; comprehensive audit logging; auto-expire credentials |
| **Adoption**: Customers don't activate feature | Medium | High | In-app prompts during audit season; CSM enablement playbook; free trial period |
| **Maintenance burden**: Framework updates (e.g., SOC2 2025) | Medium | Medium | Abstract framework definitions; partner with compliance content provider for updates |

---

## Team Allocation Recommendation

**Total Estimate**: 8-10 weeks to MVP with current team

### Phase 1: Foundation (Weeks 1-4)

| Team | Allocation | Deliverables |
|------|------------|--------------|
| **Platform** (2 eng) | 100% | Compliance Service skeleton, Evidence Collector, data store schema |
| **Network Model** (1 eng) | 50% | Control→data mapping definitions, Rule Engine extensions |
| **Integration** (3 eng) | 33% (1 eng) | MonitoringAPI/DeviceAPI query optimization for bulk evidence pulls |

### Phase 2: Core Features (Weeks 5-7)

| Team | Allocation | Deliverables |
|------|------------|--------------|
| **Platform** (2 eng) | 100% | Report Generator, scheduling infrastructure, Compliance API |
| **UI** (2 eng) | 100% | Compliance Dashboard, gap analysis view, report configuration |
| **Integration** (3 eng) | 33% (1 eng) | PDF/CSV export, email delivery integration |

### Phase 3: Polish & Launch (Weeks 8-10)

| Team | Allocation | Deliverables |
|------|------------|--------------|
| **Platform** (2 eng) | 50% | Auditor Portal backend, performance optimization |
| **UI** (2 eng) | 50% | Auditor Portal UI, onboarding flow |
| **Network Model** (1 eng) | 25% | Framework validation with test customers |
| **Integration** (3 eng) | 33% (1 eng) | Production hardening, monitoring dashboards |

### Staffing Notes

- **No new hires required**: Feature is achievable with current team
- **Integration team flex**: Only 1 of 3 needed; others continue roadmap work
- **UI dependency**: Blocked on Platform completing APIs (Week 4)
- **Risk buffer**: Weeks 9-10 include buffer for iteration based on beta feedback

---

## Appendix: Framework Mapping Example (SOC2)

```yaml
framework: soc2_type2
version: "2024"
controls:
  - id: CC6.1
    name: "Logical and Physical Access Controls"
    evidence_sources:
      - type: device_config
        query: "firewall_enabled = true"
        devices: ["edge", "core"]
      - type: health_score
        metric: "security_score"
        threshold: ">= 80"

  - id: CC7.2
    name: "System Monitoring"
    evidence_sources:
      - type: monitoring
        metric: "uptime_percentage"
        period: "30d"
        threshold: ">= 99.9"
      - type: incident
        metric: "mean_time_to_detect"
        threshold: "<= 15m"
```

---

*Document Version: 1.0*
*Last Updated: 2026-01-06*
*Author: Product & Engineering*
