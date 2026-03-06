# Feature: Factory Operation App (MVP)

**Goal:**
Provide factory floor operators and supervisors an intuitive web app to monitor production, log events, and manage work orders to improve throughput and reduce downtime.

**North Star Impact:**
Reduce mean time to detect/resolve production issues by 30% for pilot plants.

**Users:**
- Floor Operator: quick dashboard for assigned work orders, log issues.
- Supervisor: monitor KPIs (OEE, uptime), assign/track work orders.
- Maintenance Engineer: receive alerts and view asset history.

**RICE Score:**
- Reach = 2,000 users/quarter
- Impact = 2 (meaningful improvement for operations)
- Confidence = 70%
- Effort = 8 person-weeks
RICE = (2000 × 2 × 0.7) / 8 = 350

**Kano Category:** Performance

**Acceptance Criteria:**
- [ ] Authentication: Users can sign in with SSO (OAuth) and roles (operator/supervisor/maintenance).
- [ ] Dashboard: Real-time KPI tiles (OEE, uptime, throughput) refresh <5s for pilot-scale data.
- [ ] Work Orders: Create/assign/update work orders; operators can mark progress and close.
- [ ] Event logging: Operators can log incidents with timestamp, severity, and attach photos.
- [ ] Alerts: Rules-based alerts (threshold + manual) that notify maintenance via in-app and email.
- [ ] API: REST endpoints for KPIs, work orders, events; paginated responses; rate limit 200 req/min.
- [ ] Data durability: Use PostgreSQL for relational data, retention policy documented.
- [ ] Performance: 99th percentile page load <1.5s under pilot load (100 concurrent users).

**Out of Scope:**
- PLC connectivity and OT protocol adapters (will integrate via middleware in a later phase).
- Advanced analytics / ML for anomaly detection (future roadmap).

**Success Metrics:**
- Adoption: >60% active use among pilot operators within 8 weeks.
- Reliability: Error rate <1% in production pilot.
- Ops impact: 30% reduction in MTTD for pilot sites.

**Key Decisions (documented):**
- Stack: React + TypeScript frontend; Node.js + Express for API; PostgreSQL for primary DB. (Reason: full-stack JS improves developer velocity; reversible to Python if needed.)
- Auth: Start with OAuth SSO (Okta/Google) for enterprise readiness; fallback local accounts for pilots.
- Data model: Relational for deterministic queries (work orders, events) — easier to audit.
- Deployment: Containerized services (Docker) with Helm/K8s for scalability; start with single-cluster pilot.

**GitHub Issue:** TBD
