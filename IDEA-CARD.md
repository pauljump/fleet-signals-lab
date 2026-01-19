---
id: IDEA-161
title: Fleet Signals Lab
one_liner: Internal reference system for privacy-first robotaxi fleet monitoring via authorized traffic cameras with trip data correlation
status: exploring
confidence: High
directory: fleet-signals-lab
related: []
technology: [Python, FastAPI, TypeScript, Postgres, PostGIS, Redis, Computer Vision]
domain: [Autonomous Vehicles, Fleet Management, Urban Mobility, Privacy-First Analytics]
---

# IDEA-162: Fleet Signals Lab

## One-liner
Internal reference implementation for Uber AV engineering to monitor robotaxi fleet presence via authorized traffic cameras, correlate with trip demand, and detect anomalies—all with strict privacy and data rights controls.

## Problem
**For:** Uber AV engineering teams monitoring Waymo-on-Uber and future robotaxi deployments

**Pain Points:**
1. **Limited fleet visibility** - Hard to know where robotaxis are operating vs where trips are happening
2. **Supply/demand mismatch detection** - Can't quickly identify when visible fleet presence diverges from trip demand
3. **Incident response context** - When trips drop, need to know if it's a supply issue, demand shift, or technical problem
4. **Privacy constraints** - Existing solutions either violate privacy (track individuals) or lack actionable insights
5. **Data rights complexity** - No clear framework for using external camera feeds legally and ethically
6. **Integration friction** - Adding new data sources (cameras, trip data) requires rewriting core services

**Current Solutions:**
- Manual observation (doesn't scale)
- Trip data alone (no supply visibility)
- Third-party fleet tracking (privacy violations, data rights issues)
- Custom one-off scripts (not production-grade, hard to maintain)

## Solution
**Fleet Signals Lab** - A production-grade reference system that:

### Core Capabilities
1. **Authorized Image Ingestion** - Plugin system for traffic camera still images (explicit permission only)
2. **Privacy-First Detection** - Coarse "robotaxi-like vehicle" detection (NO identity, tracking, or re-identification)
3. **Zone Aggregation** - Events rolled up to zones/time buckets (not individual vehicles)
4. **Anomaly Detection** - Baseline comparison (hour-of-week) with spike explanations
5. **Trip Data Join** - Correlate camera signals (supply proxy) with trip data (demand proxy)
6. **Alerts & Dashboard** - Slack/email alerts + web UI (map + charts)

### Privacy & Governance Built-In
- **No storage of raw video** - Only detection events and optional debug thumbnails (short TTL, config flag)
- **Coarse labels only** - "robotaxi_like", "vehicle" (NOT license plates, faces, person tracking)
- **RBAC + Audit Logging** - Who accessed what, when
- **Data Rights Checklist** - Must verify permission before adding camera sources
- **Retention Policies** - Configurable TTL for events (30d), thumbnails (24h), rollups (180d)

### Extensibility
- **Camera Source Plugins** - Easy to add new authorized feeds
- **Trip Data Adapters** - Accept any format (CSV, Parquet, DB, API) and normalize
- **Model Interface** - Swap detection models without changing pipeline
- **Deployment Agnostic** - Docker Compose for dev, K8s manifests for prod

## Target User
**Primary:** Uber AV engineering teams (operations, analytics, incident response)
**Secondary:** Fleet managers, city partnership teams (if extended)

## Why Now
1. **Waymo-on-Uber partnership** - Need to monitor mixed fleet (Waymo + future robotaxi partners)
2. **Expansion to new cities** - Need scalable way to monitor rollouts
3. **Privacy regulations tightening** - Must build privacy-first from day one
4. **Internal demand** - AV teams asking for better visibility tools

## Core Value Proposition
Unlike manual observation or privacy-violating tracking:
- **Legal & Ethical** - Only uses authorized sources, no PII tracking
- **Actionable** - Correlates supply proxy (cameras) with demand (trips) to detect mismatches
- **Extensible** - Plugin architecture lets you add sources without rewriting core
- **Production-Ready** - RBAC, audit logs, retention policies, observability built-in

## Technology Approach

### System Architecture
```
[Authorized Camera Sources] → Ingest Service → Queue (Redis)
                                                    ↓
                              Infer Service ← [Detection Model]
                                    ↓
                              Events API (FastAPI + Postgres/PostGIS)
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                               ↓
            Analytics Service              [Trip Data Source]
          (rollups, anomalies)                      ↓
                    ↓                        Trip Rollups
                    ↓                               ↓
            Alerting Service         ←──────────────┘
          (Slack, Email)
                    ↓
            Web Dashboard
         (Map + Charts + Insights)
```

### Tech Stack
- **Backend:** Python (FastAPI), Postgres/PostGIS, Redis
- **Web:** TypeScript/Node, Mapbox GL JS, ECharts
- **ML:** Pluggable detector interface (baseline: lightweight vehicle detector or mock)
- **Infra:** Docker Compose (dev), Kubernetes (prod reference)
- **Auth:** JWT + RBAC stub (SSO integration documented)
- **Observability:** Structured logs, metrics endpoints

### Services (Monorepo)
1. **Ingest** - Fetch images from camera plugins, rate-limited
2. **Infer** - Run detection model, output events
3. **Events API** - Store/query events, RBAC enforcement
4. **Analytics** - Rollups, baselines, anomaly detection
5. **Alerting** - Slack/email on anomalies
6. **Web** - Dashboard UI

### Data Model
- **Cameras** - id, name, lat/lon, zone, source config
- **Zones** - PostGIS polygons (example: Austin neighborhoods)
- **Events** - detection events (camera, timestamp, label, confidence, bbox optional)
- **Rollups** - counts per zone/time (5min, 1h)
- **Anomalies** - detected spikes with explanations
- **Trip Rollups** - trips per zone/time (if trip data integrated)
- **Audit Logs** - access logs for compliance

## MVP Scope

### In Scope (v1)
- ✅ Docker Compose local dev environment
- ✅ All 6 services running end-to-end
- ✅ Camera plugin interface + Austin CCTV example (configurable)
- ✅ Detection model interface + mock detector (no weights needed for demo)
- ✅ Zone assignment (point-in-polygon)
- ✅ Rollups + hour-of-week baseline
- ✅ Simple anomaly detection (z-score)
- ✅ Web dashboard (map + time series charts + anomaly timeline)
- ✅ Slack webhook alerting
- ✅ RBAC (viewer, analyst, admin roles)
- ✅ Audit logging
- ✅ Retention policies (configurable)
- ✅ Trip data plugin stub (accepts CSV/Parquet, maps to canonical schema)
- ✅ Supply vs demand correlation view
- ✅ Synthetic data generator for demo
- ✅ Comprehensive docs (data rights, privacy threat model, ops runbook)
- ✅ K8s deployment manifests (reference, not production-tuned)
- ✅ Integration test (end-to-end on synthetic data)

### Out of Scope (v1)
- ❌ Real camera credentials (user provides)
- ❌ Real trip data (user provides via plugin)
- ❌ Fine-tuned robotaxi detection model (provide interface only)
- ❌ Multi-city configs (Austin only for demo)
- ❌ Advanced anomaly methods (LSTM, Prophet, etc.)
- ❌ Mobile app
- ❌ Public API
- ❌ SSO integration (document how, don't implement)
- ❌ Video streaming (still images only)
- ❌ Real-time latency <1min (batch processing fine for v1)

### Deferred (v2+)
- Advanced analytics (demand forecasting, route optimization insights)
- ML model training pipeline (transfer learning for robotaxi detection)
- Multi-city zone library
- Comparison with other data sources (weather, events, traffic APIs)
- Cost estimation tools
- A/B testing framework

## Technical Decisions

### Critical Constraints (Enforced in Code)
1. **Data Rights Checklist** - Must verify before adding camera source
2. **Privacy-First Detection** - Coarse labels only, no identity tracking
3. **No Raw Video Storage** - Events + optional debug thumbs (short TTL)
4. **RBAC + Audit Logs** - Who accessed what
5. **Retention Policies** - Configurable, documented

### Platform Requirements
- Postgres 14+ with PostGIS extension
- Redis 6+ (or Postgres-based job queue alternative)
- Python 3.10+
- Node 18+
- Docker + Docker Compose
- Kubernetes 1.24+ (optional, for prod reference)

### Integration Points
- **Camera Sources:** HTTP still image endpoints (plugin interface)
- **Trip Data:** CSV/Parquet/DB/API (plugin interface, canonical schema)
- **Alerts:** Slack webhook, email SMTP (stub)
- **Auth:** JWT tokens (SSO integration documented, not implemented)
- **Monitoring:** Logs (stdout) + metrics endpoint (Prometheus format)

## Key Assumptions & Risks

### Assumptions
1. **Permission Available** - User has authorized camera sources to use
2. **Coarse Detection Sufficient** - Don't need per-vehicle tracking for insights
3. **Zone Aggregation Acceptable** - Loss of granularity is acceptable for privacy
4. **Trip Data Available** - User can provide trip data in some format
5. **Internal Use Only** - Not a public product (lighter compliance requirements)

### Risks & Mitigations

**Risk:** Camera sources unauthorized or lose permission
**Mitigation:** Data rights checklist, regular audits, kill switch per source

**Risk:** Detection model too inaccurate (high false positives/negatives)
**Mitigation:** Pluggable model interface, threshold tuning, confidence filtering

**Risk:** Privacy violation via re-identification
**Mitigation:** No storage of raw frames, coarse labels only, audit logs, threat model doc

**Risk:** Data retention compliance
**Mitigation:** Configurable TTLs, automated deletion, retention policy docs

**Risk:** Low adoption (teams don't use it)
**Mitigation:** Demo mode with synthetic data, clear docs, integrate with existing workflows

**Risk:** Performance issues at scale
**Mitigation:** Batch processing, rollups, indexing strategy, horizontal scaling (K8s)

**Risk:** Trip data format changes break ingestion
**Mitigation:** Plugin interface with schema mapping, version the canonical schema

## Success Metrics

### MVP Success (v1)
- ✅ System runs end-to-end in demo mode with synthetic data
- ✅ Can add a new camera source via plugin in <1 hour
- ✅ Can ingest trip data CSV and see correlation chart
- ✅ Anomaly detected and alert sent on synthetic spike
- ✅ 3+ AV engineers can run locally and understand code

### Production Success (v2)
- 5+ real camera sources ingesting
- Trip data integrated (Waymo-on-Uber trips visible)
- 1+ actionable insight per week (supply/demand mismatch, incident context)
- <5min to investigate anomaly via dashboard
- 0 privacy violations or data rights issues

## Open Questions

### Technical
1. **Detection Model:** Use pre-trained COCO vehicle detector or fine-tune on robotaxi images?
2. **Queue Choice:** Redis Streams vs Celery vs Postgres-based (pg_job)?
3. **Map Library:** Mapbox (requires key) vs Leaflet (open) vs deck.gl?
4. **Anomaly Method:** Z-score sufficient or use EWMA/Prophet?
5. **Real-time Requirements:** Batch ok or need <1min latency?

### Product
6. **Uber Trip Data Access:** What's the approval process? Can we get sample data?
7. **Camera Permissions:** Which cities have authorized public camera APIs?
8. **Privacy Review:** Does this need legal/privacy team sign-off?
9. **Deployment:** Self-hosted or managed service?
10. **Ownership:** Which team owns and maintains this long-term?

### Business
11. **Budget:** Compute/storage costs at scale?
12. **Licensing:** Are detection model weights open-source compatible?
13. **Data Sharing:** Can insights be shared with city partners?

## Repository Structure
```
fleet-signals-lab/
├── README.md
├── docs/
│   ├── 00_overview.md
│   ├── 01_data_rights_checklist.md
│   ├── 02_privacy_threat_model.md
│   ├── 03_retention_and_access.md
│   ├── 04_ops_runbook.md
│   └── 05_uber_trip_data_integration.md
├── infra/
│   ├── docker-compose.yml
│   └── k8s/
├── services/
│   ├── ingest/
│   ├── infer/
│   ├── events_api/
│   ├── analytics/
│   ├── alerting/
│   └── web/
├── packages/
│   ├── shared_schemas/
│   └── source_plugins/
├── db/
│   ├── migrations/
│   └── seed/
├── ml/
│   ├── model_interface.md
│   ├── eval/
│   └── training_stub/
└── scripts/
    ├── dev_bootstrap.sh
    ├── load_synthetic_data.py
    └── demo_run.sh
```

## Next Steps

### Immediate (Pre-MVP)
1. Validate with AV engineering team - is this useful?
2. Get sample trip data (anonymized/synthetic)
3. Identify 1-2 authorized camera sources for pilot
4. Privacy/legal review of data rights checklist
5. Decide on detection model approach (pre-trained vs fine-tune)

### MVP Development
1. Set up monorepo structure
2. Build services in order: ingest → infer → events_api → analytics → web → alerting
3. Implement camera plugin interface + Austin example
4. Implement trip data plugin interface + CSV adapter
5. Create synthetic data generators
6. Build web dashboard
7. Write comprehensive docs
8. Integration tests
9. Demo with synthetic data

### Post-MVP
1. Onboard real camera sources (1-2 cities)
2. Integrate real trip data
3. Tune detection model
4. Deploy to staging environment
5. User testing with AV team
6. Iterate based on feedback
7. Plan v2 features

## Related Ideas
- IDEA-007: CityCel (civic data infrastructure - could share camera source plugins)
- IDEA-013: EasyEdgar (SEC data aggregation - similar plugin architecture pattern)
- IDEA-024: Shovelsale Retail Intelligence (similar analytics/anomaly detection)

## Notes
- **This is a reference implementation** - not production-deployed initially
- **Privacy-first by design** - easier to add features than remove privacy violations
- **Plugin architecture** - makes it useful beyond initial use case
- **Documentation as code** - compliance via docs + code structure
- **Demo mode critical** - must run without real credentials for development

---

**Status:** Exploring → Ready to validate with stakeholders
**Confidence:** High (well-defined problem, clear technical approach, strong constraints)
**Risk Level:** Medium (privacy/legal review needed, depends on data access approvals)

🤖 Generated with Idea Factory
