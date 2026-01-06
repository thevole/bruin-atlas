# Bruin Atlas

**Enterprise Network Infrastructure Monitoring & Visualization Platform**

Bruin Atlas transforms complex network infrastructure into comprehensible, actionable insights. Built for NOC teams, network engineers, and IT leadership, it provides real-time health monitoring, topology visualization, and proactive intelligence.

## Core Capabilities

| Feature | Description |
|---------|-------------|
| **Network Topology Viewer** | Interactive visualization with hierarchical location-based navigation |
| **Health Dashboard** | Executive-level network health overview with drill-down capability |
| **Import Wizard** | Guided multi-step import flow for customer network data |
| **Incident Tracking** | Real-time incident overlay on topology with affected device mapping |
| **Data Trust Indicators** | Explicit confidence markers and freshness timestamps |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Bruin Atlas Platform                     │
├──────────────┬──────────────┬──────────────┬────────────────┤
│   Platform   │ Network Model│     UI       │  Integration   │
│ Architecture │  & DeviceAPI │ Visualization│    Layer       │
├──────────────┼──────────────┼──────────────┼────────────────┤
│ PostgreSQL   │ Rule Engine  │ React/Next.js│ MonitoringAPI  │
│ REST API     │ Reference    │ Topology     │ DeviceAPI      │
│ K8s Deploy   │ Architecture │ Viewer       │ Event Pipeline │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

## Team Structure

- **Team 1 - Platform Architecture** (2 eng): Database, API, deployment
- **Team 2 - Network Model & DeviceAPI** (1 eng): Rules, standards, device logic
- **Team 3 - Visualization & UI** (2 eng): Frontend, topology, dashboards
- **Team 4 - Atlas Integration** (3 eng): Real-time data, events, incidents

## Phase 2 Feature Ideas

See the `/Ideas` folder for detailed specifications:

1. **[Automated Compliance Reporting](Ideas/01-automated-compliance-reporting.md)** - Transform network data into SOC2/ISO audit-ready reports
2. **[Network Behavior Baselines](Ideas/02-network-behavior-baselines.md)** - Statistical baselines for proactive anomaly detection
3. **[Change Impact Simulation](Ideas/03-change-impact-simulation.md)** - "What-if" analysis before network changes

## Key Differentiators

- **Real-time intelligence**: 15-30 minute device data freshness (vs industry 12hr+)
- **Data trust**: Explicit confidence markers on all data points
- **Reference architecture compliance**: Compare customer networks to gold standards
- **Proactive monitoring**: Baselines and simulation move beyond reactive alerting

## Getting Started

```bash
# Development
pnpm install
pnpm dev

# Build
pnpm build

# Tests
pnpm test
```

## Contributors

- **Michael Volerich** - Product & Engineering

---

*Built by the Bruin team*
