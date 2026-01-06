# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bruin Atlas is an enterprise network infrastructure monitoring and visualization platform. This repository currently contains **planning and design documentation** for the platform, not implementation code.

## Repository Structure

```
/Ideas/              Feature specifications and proposals
  00-feature-ranking-matrix.md   Ranked feature backlog with scoring
  01-automated-compliance-reporting.md
  02-network-behavior-baselines.md
  03-change-impact-simulation.md

/Diagrams/           Architecture diagrams (Mermaid format)
  01-platform-architecture.md    System overview, team ownership, data flow

/Discussions/        Decision records and deliberation logs
```

## Architecture (Planned)

**Five-Layer Architecture:**
1. **Presentation**: React/Next.js UI, API Gateway
2. **Application Services**: Atlas API, Import Service, Health Scoring, Compliance, Simulation
3. **Intelligence**: Rule Engine (JSON rules), Baseline Engine, Reference Architecture
4. **Data**: PostgreSQL (topology), Redis (cache), TimeSeries DB (metrics)
5. **Integration**: MonitoringAPI, DeviceAPI, Event Pipeline (Kafka/RabbitMQ)

**Team Ownership:**
- Team 1 (Platform): API, Auth, Import, PostgreSQL, K8s deployment
- Team 2 (Network Model): Rule Engine, Reference Model, DeviceAPI, Health Scoring
- Team 3 (UI): Topology Viewer, Health Dashboard, Import Wizard
- Team 4 (Integration): Data Pipeline, Event Stream, Incident Connector, Data Trust

## Key Concepts

- **Health Scoring**: Aggregated network health metrics with confidence levels
- **Data Trust Indicators**: Every data point has freshness timestamp and confidence marker (verified/inferred/stale/unknown)
- **Reference Architecture**: Gold-standard network patterns for gap analysis
- **Rule Engine**: JSON-defined rules for redundancy detection, compliance, health anomalies

## Commands (When Implementation Exists)

```bash
pnpm install    # Install dependencies
pnpm dev        # Development server
pnpm build      # Production build
pnpm test       # Run tests
```

## Working with This Repository

- Feature specs in `/Ideas/` follow a standard template: Executive Summary, Problem Statement, Proposed Solution, User Stories, Technical Approach, Success Metrics, Risks
- Architecture diagrams use Mermaid syntax (renders in GitHub/VSCode)
- Feature ranking uses weighted scoring across: Revenue Impact, Customer Value, Technical Feasibility, Strategic Alignment, Time to Value
