# Council of Five: Bruin Atlas Phase 2 Feature Ideas

**Date:** 2026-01-06
**Subject:** Deliberation on 3 high-impact features for Phase 2

---

## Today's Council (Randomly Selected)

1. **The Operator** (Architecture) - Production reality expert
2. **The Revenue Guardian** (Business) - Monetization focus
3. **The Power User** (UX) - Enterprise efficiency advocate
4. **The Skeptic** (Wildcard) - Assumption challenger
5. **The Data Scientist** (Business) - Measurement mindset

---

## The Matter Before the Council

Identify 3 high-impact features for Bruin Atlas Phase 2 that:
- Deliver clear customer value
- Leverage existing assets (rule engine, MonitoringAPI, DeviceAPI)
- Are feasible for an 8-engineer team

---

## Phase 2: Opening Statements

### The Operator
**Position:** Focus on **predictive failure detection** - correlate health signals with historical incident patterns to warn before outages
**Primary concern:** Current 15-30min refresh is reactive; customers need proactive warnings
**Recommendation:** Build anomaly detection that surfaces "this looks like what happened before failure X"

### The Revenue Guardian
**Position:** Invest in **automated compliance reporting** - turn Atlas data into SOC2/audit-ready exports
**Primary concern:** Phase 1 is cost center; need clear upsell/premium tier justification
**Recommendation:** Compliance reports command premium pricing; enterprise buyers need this for renewals

### The Power User
**Position:** Build **change impact simulation** - "what if" analysis before network modifications
**Primary concern:** NOC engineers need to model changes before executing; current tools are blind
**Recommendation:** Let users simulate adding/removing devices and see projected health impact

### The Skeptic
**Position:** Challenge all three - are we solving problems customers actually have, or building features we think are cool?
**Primary concern:** 8 engineers is thin; betting on wrong feature could burn a quarter
**Recommendation:** Validate with 3-5 customer interviews before committing to any of these

### The Data Scientist
**Position:** Build **network behavior baselines** - establish "normal" for each customer's network, alert on deviation
**Primary concern:** Health scores without baseline are arbitrary; "80%" means nothing without context
**Recommendation:** ML-light approach: statistical baselines per device type/location, no fancy models needed

---

## Phase 3: Deliberation

### Round 1

**The Skeptic → The Operator:** "Predictive failure sounds great, but do we have enough historical data to correlate? How much incident history exists?"

**The Operator:** "Fair point. We'd need 6+ months of incident data per customer. But we DO have the rule engine history - that's our unfair advantage. We're not starting from zero."

---

**The Revenue Guardian → The Skeptic:** "You want customer validation? Compliance reporting doesn't need validation - every enterprise customer asks for this during procurement. It's table stakes for renewals."

**The Skeptic:** "Then why isn't everyone already doing it? If it's so obvious, what's the barrier?"

**The Revenue Guardian:** "Execution complexity - mapping raw data to compliance frameworks is tedious. That's exactly why it commands premium pricing."

---

### Round 2

**The Power User → The Data Scientist:** "Baselines are nice, but operators need action. How does 'this is abnormal' translate to 'here's what to do'?"

**The Data Scientist:** "Combine baseline deviation with the rule engine's gap detection. Baseline says 'something changed,' rules say 'here's what's wrong.' They complement."

---

**The Operator → The Power User:** "Change simulation is ambitious for 8 engineers. The topology model would need to be fully graph-traversable. Are we ready for that complexity?"

**The Power User:** "We're building topology visualization anyway. Simulation is a natural extension - just run health rules against hypothetical state instead of actual state."

---

## Phase 4: Council Verdict

### Consensus Points
- All agree features must leverage existing rule engine, not build new ML systems
- Compliance reporting has clearest ROI and least technical risk
- Predictive/baseline features need historical data - feasibility depends on data availability

### Dissenting Views
- **The Skeptic** remains unconvinced on change simulation complexity for current team size
- **The Power User** thinks compliance is boring and won't differentiate

### Recommended Features (Ranked)

| Rank | Feature | Rationale |
|------|---------|-----------|
| 1 | **Automated Compliance & Audit Reporting** | Clear revenue justification, low technical risk, leverages existing data |
| 2 | **Network Behavior Baselines with Deviation Alerts** | Statistical approach (no ML), builds on health scoring, validates "normal" per customer |
| 3 | **Change Impact Simulation (Scoped)** | Start with simple "remove device X, show affected paths" - not full simulation |

### Acknowledged Tradeoffs
- Prioritizing compliance over innovation (safe but not exciting)
- Baselines require 30+ days data accumulation before value
- Simulation scope deliberately limited to reduce risk

### Minority Report
**The Skeptic** notes: "We're making decisions without customer validation. I recommend a lightweight validation sprint before committing engineering resources to any of these."

---

## Action Items

1. Write detailed specs for all three features → `/Ideas/` folder
2. Validate compliance feature with 3 enterprise customers (PM)
3. Audit existing incident history to assess baseline feasibility (Integration team)
4. Confirm topology model supports graph traversal (Platform team)

---

*Council adjourned.*
