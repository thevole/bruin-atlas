# Bruin Atlas Feature Ranking Matrix

**Last Updated:** 2026-01-06
**Status:** Phase 2 Planning

---

## Executive Summary

This document ranks existing feature proposals and introduces 5 additional candidates for consideration. Features are evaluated across business impact, technical feasibility, team fit, and strategic alignment.

---

## Scoring Methodology

| Dimension | Weight | Description |
|-----------|--------|-------------|
| **Revenue Impact** | 25% | Direct monetization potential, upsell opportunity, renewal influence |
| **Customer Value** | 25% | Pain point severity, frequency of need, competitive differentiation |
| **Technical Feasibility** | 20% | Leverages existing assets, team capability match, risk level |
| **Strategic Alignment** | 15% | Fits product vision, enables future features, market positioning |
| **Time to Value** | 15% | How quickly customers see benefits post-launch |

**Score Range:** 1-5 (1=Low, 5=High)

---

## Existing Features: Ranked

### Ranking Table

| Rank | Feature | Revenue | Customer | Feasibility | Strategy | Time to Value | **Weighted Score** |
|------|---------|---------|----------|-------------|----------|---------------|-------------------|
| **1** | Automated Compliance Reporting | 5 | 4 | 4 | 5 | 4 | **4.35** |
| **2** | Network Behavior Baselines | 3 | 5 | 4 | 5 | 3 | **4.00** |
| **3** | Change Impact Simulation | 4 | 4 | 3 | 4 | 4 | **3.75** |

---

### #1: Automated Compliance Reporting

**Weighted Score: 4.35**

| Dimension | Score | Rationale |
|-----------|-------|-----------|
| Revenue Impact | 5 | Clear premium tier justification; $500K+ ARR potential in Y1 |
| Customer Value | 4 | Eliminates 40-80 hours/audit cycle; every enterprise needs this |
| Technical Feasibility | 4 | 70% data exists; transformation layer is well-understood work |
| Strategic Alignment | 5 | Table stakes for enterprise market; enables GRC positioning |
| Time to Value | 4 | First report generated within days of enablement |

**Arguments FOR:**
- Direct revenue attribution via premium tier pricing
- Competitive parity with DataDog, Splunk, ServiceNow
- Low technical risk; no new data collection required
- Auditor portal creates external user touchpoints (expansion vector)

**Arguments AGAINST:**
- Not differentiated; competitors already have this
- Framework accuracy risk could create customer liability
- Maintenance burden as compliance standards evolve

**Recommendation:** ✅ **BUILD FIRST** - Clearest ROI, lowest risk, highest urgency for enterprise deals.

---

### #2: Network Behavior Baselines

**Weighted Score: 4.00**

| Dimension | Score | Rationale |
|-----------|-------|-----------|
| Revenue Impact | 3 | Indirect; improves platform value but hard to price separately |
| Customer Value | 5 | Transforms monitoring from reactive to proactive; high NOC value |
| Technical Feasibility | 4 | Statistical approach (no ML); uses existing time-series data |
| Strategic Alignment | 5 | Core differentiator; "Atlas knows what's normal for YOUR network" |
| Time to Value | 3 | 30+ day cold start before baselines mature |

**Arguments FOR:**
- True product differentiation; competitors use static thresholds
- Reduces alert fatigue by filtering noise
- Foundation for predictive features (future ML opportunities)
- Makes health scores meaningful ("10% below YOUR normal")

**Arguments AGAINST:**
- Cold start problem delays value for new customers
- Statistical concepts may confuse non-technical users
- Risk of baseline drift masking gradual degradation

**Recommendation:** ✅ **BUILD SECOND** - High differentiation, but requires Phase 1 data accumulation.

---

### #3: Change Impact Simulation

**Weighted Score: 3.75**

| Dimension | Score | Rationale |
|-----------|-------|-----------|
| Revenue Impact | 4 | Premium feature for operations teams; chargeable |
| Customer Value | 4 | High value for NOC; prevents change-induced outages |
| Technical Feasibility | 3 | Requires graph-traversable topology; moderate complexity |
| Strategic Alignment | 4 | Differentiates from observe-only tools; "plan before act" |
| Time to Value | 4 | Immediate value once topology is loaded |

**Arguments FOR:**
- Strong competitive differentiation; most tools are observe-only
- Natural extension of topology visualization
- ITSM integration potential (change approval evidence)
- Training use case broadens user base

**Arguments AGAINST:**
- Full simulation is complex; must tightly scope MVP
- Risk of false confidence if simulation misses real-world factors
- Requires topology model to be highly accurate
- Scope creep risk is high

**Recommendation:** ✅ **BUILD THIRD** - High differentiation but scope carefully; start with device removal only.

---

## New Feature Proposals

### Proposed Features: Quick Comparison

| Rank | Feature | Revenue | Customer | Feasibility | Strategy | Time to Value | **Weighted Score** |
|------|---------|---------|----------|-------------|----------|---------------|-------------------|
| **4** | Capacity Forecasting | 4 | 4 | 3 | 4 | 3 | **3.65** |
| **5** | Multi-Tenant Comparison Insights | 5 | 3 | 4 | 4 | 4 | **3.95** |
| **6** | Automated Runbook Integration | 3 | 5 | 3 | 4 | 3 | **3.60** |
| **7** | Network Change Changelog | 3 | 4 | 5 | 3 | 5 | **3.90** |
| **8** | AI-Powered Root Cause Analysis | 4 | 5 | 2 | 5 | 2 | **3.55** |

---

### #4: Capacity Forecasting

**Proposed Score: 3.65**

#### Executive Summary
Predict when network components will hit capacity limits based on historical trends, enabling proactive expansion planning before performance degrades.

#### Problem Statement
- Network teams discover capacity issues reactively when performance degrades
- Budgeting for network expansion happens without data-driven justification
- "We need more bandwidth" requests lack evidence for finance approval
- Seasonal patterns (holiday traffic, quarterly close) catch teams off guard

#### Proposed Solution
```
Capacity Forecasting Engine:
1. Analyze historical utilization trends per device/link
2. Apply statistical forecasting (linear regression, seasonal decomposition)
3. Project "days until X% capacity" thresholds
4. Alert when projected exhaustion falls within planning horizon
5. Generate capacity planning reports for budget requests
```

#### Leveraged Assets
| Asset | Usage |
|-------|-------|
| Time Series DB | Historical utilization data for trend analysis |
| Baseline Engine | Seasonal patterns already computed |
| Health Scoring | Capacity as input to health calculations |
| Report Generator | Capacity planning report templates |

#### Arguments FOR
- **Budget Justification**: "We'll hit 90% on core links in 4 months" gets budget approved
- **Natural Extension**: Baselines already compute trends; forecasting is next step
- **Upsell Opportunity**: Premium capacity planning module
- **Executive Appeal**: CIO-level visibility into infrastructure runway

#### Arguments AGAINST
- **Accuracy Risk**: Forecasting is inherently uncertain; wrong predictions erode trust
- **Scope Creep**: Full capacity planning is complex (procurement, vendor lead times)
- **Baseline Dependency**: Requires mature baselines; compounds cold start problem
- **Competition**: Many monitoring tools offer basic forecasting

#### Technical Considerations
- Start with simple linear projection; avoid complex ML
- Confidence intervals are critical; never show point estimates alone
- Seasonal decomposition needed for networks with cyclical patterns

#### Recommendation
**BUILD IN PHASE 3** - Natural evolution of baselines, but wait until baseline adoption proves value.

---

### #5: Multi-Tenant Comparison Insights (Anonymized Benchmarking)

**Proposed Score: 3.95**

#### Executive Summary
Enable customers to compare their network health, redundancy posture, and operational metrics against anonymized peer benchmarks, answering "How do we compare to similar organizations?"

#### Problem Statement
- Customers have no external reference for whether their metrics are "good"
- "Our availability is 99.5%" - is that above or below industry norm?
- Security and compliance teams want to benchmark against peers
- Executives ask "How do we compare to competitors?" with no data to answer

#### Proposed Solution
```
Benchmarking Service:
1. Aggregate anonymized metrics across customer base
2. Segment by industry, company size, network complexity
3. Compute percentile rankings for key metrics
4. Surface comparisons in dashboards: "Your redundancy score is 85th percentile"
5. Optional: Anonymized best practices from top performers
```

#### Leveraged Assets
| Asset | Usage |
|-------|-------|
| Health Scoring | Standardized metrics enable cross-customer comparison |
| Customer Metadata | Industry, size segmentation for meaningful cohorts |
| PostgreSQL | Aggregate queries across tenant data |
| Compliance Reports | Benchmark compliance posture |

#### Arguments FOR
- **Unique Value**: Only possible for platforms with multiple customers (SaaS advantage)
- **Executive Appeal**: Answers "How do we compare?" at board level
- **Retention Driver**: Customers become invested in improving their ranking
- **Upsell Trigger**: "Top 10% have these features enabled" drives adoption

#### Arguments AGAINST
- **Privacy Concerns**: Customers may resist data aggregation even if anonymized
- **Small N Problem**: Need sufficient customers per segment for meaningful benchmarks
- **Gaming Risk**: Customers may optimize for benchmarks, not real improvement
- **Legal Review**: Requires careful terms of service and consent design

#### Technical Considerations
- Differential privacy techniques for true anonymization
- Minimum cohort size (N≥20) before showing percentiles
- Opt-in model with clear value proposition

#### Recommendation
**BUILD IN PHASE 3** - High strategic value but requires customer scale and legal groundwork.

---

### #6: Automated Runbook Integration

**Proposed Score: 3.60**

#### Executive Summary
Connect Atlas incidents to automated remediation runbooks, enabling one-click or auto-triggered responses to common issues (restart service, clear cache, failover).

#### Problem Statement
- Same incidents occur repeatedly; operators perform identical manual steps each time
- Mean Time to Resolve (MTTR) is dominated by human response latency
- Runbooks exist in wikis but aren't connected to alerting systems
- Night/weekend incidents wait for on-call human response

#### Proposed Solution
```
Runbook Integration Layer:
1. Define runbook templates (restart, failover, escalate)
2. Associate runbooks with incident types or rule triggers
3. Enable one-click execution from incident view
4. Optional: Auto-execute with approval thresholds
5. Audit trail of all runbook executions
```

#### Leveraged Assets
| Asset | Usage |
|-------|-------|
| Incident System | Trigger point for runbook execution |
| DeviceAPI | Execute device operations (restart, config push) |
| Rule Engine | Conditions that trigger runbook suggestions |
| Audit Logging | Track all automated actions for compliance |

#### Arguments FOR
- **MTTR Reduction**: Automated response cuts resolution time by 10-50x
- **NOC Efficiency**: Operators handle exceptions, not routine responses
- **Competitive Parity**: PagerDuty, Rundeck offer runbook automation
- **Compliance Appeal**: Documented, auditable response procedures

#### Arguments AGAINST
- **Blast Radius Risk**: Automated actions can cause cascading failures
- **Trust Barrier**: Customers hesitant to let tool take actions
- **Integration Complexity**: Must support diverse device types and vendors
- **Liability Concerns**: Who's responsible when runbook causes outage?

#### Technical Considerations
- Start with "suggest and confirm" before auto-execute
- Sandbox/dry-run mode for runbook testing
- Rollback capability for every automated action
- Strict RBAC on who can enable automation

#### Recommendation
**BUILD IN PHASE 4** - High value but high risk; requires mature platform trust and careful rollout.

---

### #7: Network Change Changelog

**Proposed Score: 3.90**

#### Executive Summary
Automatically detect and log all network configuration changes, creating a searchable audit trail that answers "What changed and when?" for troubleshooting and compliance.

#### Problem Statement
- "Something changed" is the most common incident trigger
- Correlating incidents to changes requires manual investigation
- Configuration drift goes undetected until it causes problems
- Compliance requires change documentation that's often incomplete

#### Proposed Solution
```
Change Detection Service:
1. Poll device configs on regular interval (hourly/daily)
2. Diff against previous snapshot; detect changes
3. Classify changes (security, routing, interface, etc.)
4. Correlate changes with incidents (time proximity)
5. Surface in UI: "3 changes detected near incident start"
```

#### Leveraged Assets
| Asset | Usage |
|-------|-------|
| DeviceAPI | Pull device configurations |
| PostgreSQL | Store config snapshots and diffs |
| Incident System | Correlate changes with incidents |
| Compliance Service | Change logs as audit evidence |

#### Arguments FOR
- **High Feasibility**: Simple diff algorithm; no complex logic
- **Immediate Value**: Changelog populated from day one
- **Universal Need**: Every NOC asks "what changed?"
- **Compliance Bonus**: Change documentation for SOC2/ISO

#### Arguments AGAINST
- **Noise Risk**: Many changes are routine; need smart filtering
- **Storage Cost**: Config snapshots can be large
- **Attribution Gap**: Detects what changed, not who changed it
- **Vendor Variation**: Config formats vary widely across vendors

#### Technical Considerations
- Semantic diff (not line-by-line) for meaningful comparisons
- Change classification using rule-based heuristics
- Retention policy to manage storage growth
- Integration with ITSM for change ticket correlation

#### Recommendation
**BUILD IN PHASE 2** - Low risk, high feasibility, immediate value. Consider bundling with compliance feature.

---

### #8: AI-Powered Root Cause Analysis

**Proposed Score: 3.55**

#### Executive Summary
Apply machine learning to automatically identify root causes of incidents by correlating symptoms across devices, time, and topology, reducing diagnosis time from hours to minutes.

#### Problem Statement
- Root cause analysis is manual, time-consuming, and expertise-dependent
- Incident symptoms span multiple devices; humans miss correlations
- Post-mortems reveal obvious patterns that weren't seen in the moment
- Expert knowledge walks out the door when senior engineers leave

#### Proposed Solution
```
RCA Engine:
1. Collect symptom data across all devices during incident window
2. Apply correlation analysis (statistical and graph-based)
3. Use historical incident patterns to suggest likely causes
4. Present ranked hypotheses with supporting evidence
5. Learn from operator feedback to improve accuracy
```

#### Leveraged Assets
| Asset | Usage |
|-------|-------|
| Incident History | Training data for pattern recognition |
| Topology Model | Graph analysis for impact propagation |
| Health Signals | Symptom data across devices |
| Baseline Engine | Deviation data as input signals |

#### Arguments FOR
- **High Differentiation**: Few competitors offer genuine AI-powered RCA
- **Expert Capture**: Encodes senior engineer knowledge in the system
- **Scalability**: Works at 3am when experts are sleeping
- **Future Platform**: Foundation for broader AI/ML features

#### Arguments AGAINST
- **Technical Complexity**: Genuine ML requires expertise team may lack
- **Data Requirements**: Needs significant incident history to train
- **Accuracy Risk**: Wrong suggestions erode trust quickly
- **Cold Start Squared**: New customers have no incident history

#### Technical Considerations
- Start with rule-based heuristics before ML
- Explainability is critical; black-box suggestions won't be trusted
- Human-in-the-loop validation before suggestions go live
- A/B testing against human RCA for accuracy measurement

#### Recommendation
**BUILD IN PHASE 4+** - High strategic value but technically challenging; defer until platform matures.

---

## Consolidated Ranking: All 8 Features

| Rank | Feature | Score | Phase | Rationale |
|------|---------|-------|-------|-----------|
| 1 | **Automated Compliance Reporting** | 4.35 | Phase 2 | Clearest ROI, enterprise table stakes |
| 2 | **Network Behavior Baselines** | 4.00 | Phase 2 | High differentiation, enables future features |
| 3 | **Multi-Tenant Comparison Insights** | 3.95 | Phase 3 | Unique SaaS advantage, executive appeal |
| 4 | **Network Change Changelog** | 3.90 | Phase 2 | Low risk, high feasibility, immediate value |
| 5 | **Change Impact Simulation** | 3.75 | Phase 2-3 | Differentiation, but scope carefully |
| 6 | **Capacity Forecasting** | 3.65 | Phase 3 | Natural baseline evolution |
| 7 | **Automated Runbook Integration** | 3.60 | Phase 4 | High value but high risk |
| 8 | **AI-Powered Root Cause Analysis** | 3.55 | Phase 4+ | Strategic but technically challenging |

---

## Recommended Phase 2 Portfolio

Based on analysis, recommended Phase 2 feature set:

| Priority | Feature | Effort | Risk | Revenue |
|----------|---------|--------|------|---------|
| P0 | Automated Compliance Reporting | 8-10 weeks | Low | High |
| P1 | Network Behavior Baselines | 6-8 weeks | Medium | Medium |
| P2 | Network Change Changelog | 3-4 weeks | Low | Low |
| P3 | Change Impact Simulation (MVP) | 8 weeks | Medium | Medium |

**Total Phase 2 Effort**: ~25-30 weeks (with parallel workstreams)

**Team Allocation**:
- Compliance + Changelog: Platform team + 1 Integration
- Baselines: Integration team + Network Model
- Simulation: Platform + UI teams

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-01-06 | Initial ranking completed | Council of Five deliberation |
| 2026-01-06 | Added 5 new feature proposals | Extended roadmap planning |
| - | Compliance prioritized over Baselines | Revenue urgency for enterprise deals |
| - | Changelog added to Phase 2 | Low effort, high synergy with compliance |
| - | RCA deferred to Phase 4+ | Technical complexity, data requirements |

---

*Document maintained by Product & Engineering*
