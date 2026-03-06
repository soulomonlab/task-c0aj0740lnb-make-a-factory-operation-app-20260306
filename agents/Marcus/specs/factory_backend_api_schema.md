Title: Factory Operation App — Backend API & DB Schema Draft

Scope
- Deliver initial API routes and DB schema draft for MVP.
- Focus: facilities, machines, operations (work orders), shifts, alerts, user SSO integration.
- Out of scope: PLC/OT integration, ML, predictive maintenance.

Assumptions
- Primary DB: PostgreSQL (single region).
- Auth: OAuth2 / OIDC via Okta/Google (SSO). Backend maintains local user records mapped to external IDs.
- Frontend: React+TS (vetted by Product).
- Performance targets: p95 < 100ms for list/get; DB queries < 20ms.

API Routes (initial)
| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET    | /api/v1/facilities | List facilities (pagination) | Bearer OIDC | 
| POST   | /api/v1/facilities | Create facility | Bearer, role:admin |
| GET    | /api/v1/facilities/{id} | Get facility | Bearer |
| GET    | /api/v1/machines | List machines (filter by facility, status) | Bearer |
| POST   | /api/v1/machines | Register machine | Bearer, role:admin |
| GET    | /api/v1/machines/{id} | Machine detail + latest status | Bearer |
| GET    | /api/v1/operations | List operations (work orders) | Bearer |
| POST   | /api/v1/operations | Create operation | Bearer |
| PATCH  | /api/v1/operations/{id}/status | Update op status (idempotent) | Bearer |
| GET    | /api/v1/alerts | List alerts | Bearer |
| POST   | /api/v1/alerts/{id}/ack | Acknowledge alert | Bearer |
| POST   | /api/v1/auth/callback | OAuth callback (exchange code) | public |

Error format
- {"error": {"code": "string", "message": "string", "details": {}}}

DB Schema (draft) — core tables
1) users
- id UUID PK
- external_id VARCHAR UNIQUE (Okta/Google sub)
- email VARCHAR UNIQUE
- name VARCHAR
- role VARCHAR (enum: admin, manager, operator)
- created_at TIMESTAMP, updated_at TIMESTAMP
Indexes: external_id, email

2) facilities
- id UUID PK
- name TEXT
- location JSONB (optional)
- metadata JSONB
- created_at, updated_at

3) machines
- id UUID PK
- facility_id UUID FK -> facilities.id
- name TEXT
- model TEXT
- serial VARCHAR
- status VARCHAR (enum: online, offline, maintenance)
- metadata JSONB
- created_at, updated_at
Indexes: facility_id, status

4) operations
- id UUID PK
- facility_id UUID FK
- created_by UUID FK -> users.id
- machine_id UUID FK -> machines.id (nullable)
- type VARCHAR
- status VARCHAR (enum: pending, running, completed, failed)
- payload JSONB
- started_at TIMESTAMP, finished_at TIMESTAMP
- created_at, updated_at
Indexes: facility_id, status

5) shifts
- id UUID PK
- facility_id
- name
- start_time, end_time
- created_at

6) alerts
- id UUID PK
- facility_id
- machine_id (nullable)
- severity VARCHAR (info, warn, critical)
- status VARCHAR (open, ack, closed)
- message TEXT
- metadata JSONB
- created_at, acknowledged_at

7) readings (time-series lightweight)
- id BIGSERIAL PK
- machine_id UUID
- ts TIMESTAMP WITH TIME ZONE
- metrics JSONB
Indexes: (machine_id, ts DESC) BRIN or btree depending on volume

Audit & logs
- audit_logs(id UUID, entity_type, entity_id, action, actor_id, payload JSONB, created_at)

Key constraints & indexes noted inline; further index recommendations after profiling.

Auth model
- Use OIDC for primary authentication.
- Backend: /auth/callback exchanges code -> verifies id_token, upserts users table by external_id.
- Sessions: issue short-lived access JWTs (15m) and refresh tokens (rotate) OR rely on frontend to hold OIDC tokens. (Decision: implement JWT access/refresh for internal APIs; map scopes -> roles.)

Migrations
- Use Alembic (or equivalent) migrations to create tables above. Provide initial migration script placeholder.

Acceptance criteria
- CRUD endpoints implemented for facilities, machines, operations with OpenAPI v3 spec.
- OAuth callback established; local user upserted on first login.
- Unit tests covering business logic + DB layer (target >80% coverage)
- Perf: p95 <100ms for list endpoints under baseline dataset (10k machines)

Estimated effort (engineering)
- DB schema + migrations: 1.0 day
- OpenAPI v3 spec (complete): 0.5 day
- Auth OIDC callback + user upsert + token handling (stub): 1.0 day
- CRUD endpoints + service layer for core entities: 1.5 day
- Unit tests + perf test plan: 1.0 day
Total: ~5.0 engineering days (1 BE engineer)

Risks & next steps
- Risk: High cardinality readings table — may need dedicated TSDB later; consider using PostgreSQL partitioning.
- Next: I'll produce OpenAPI v3 file and initial migration + auth stub. Then hand off API contract to #ai-frontend (Kevin) for integration.

Files produced
- output/specs/factory_backend_api_schema.md
- output/docs/openapi_factory_v1.yaml (minimal stub)

Document created by: Marcus (Backend)
