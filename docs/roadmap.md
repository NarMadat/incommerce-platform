# inCommerce Roadmap

This roadmap describes **phased delivery**. Each phase adds user-visible value while keeping operational complexity manageable.

## Phase 0 — Foundation (current)

**Goal:** Establish monorepo structure, architecture docs, and team conventions.

- [x] Nx workspace initialized
- [x] Folder structure for apps, services, packages, infra, docs
- [x] Architecture documentation and ADRs
- [ ] Root ESLint / Prettier / commit conventions
- [ ] Docker Compose for PostgreSQL and Redis (local only)

**Exit criteria:** A new developer can clone the repo, read the docs, and understand where code will live.

---

## Phase 1 — Modular monolith MVP

**Goal:** One deployable backend + minimal admin UI covering core SMB workflows.

### Backend (NestJS modular monolith in `apps/api-gateway`)

| Module | Capabilities |
|--------|--------------|
| Auth | Register, login, JWT, roles (owner, staff) |
| Organization | CRUD organizations, multi-store support |
| Catalog | Products, categories |
| Inventory | Stock levels, movements (in/out/adjustment) |
| Sales | Orders, line items, basic totals |
| Customers | Customer records linked to sales |
| Reports | Sales summary, inventory snapshot (read-only queries) |

### Frontend

- `web-admin`: authentication, org/store switcher, CRUD screens for MVP entities, basic report views.

### Data & infra

- PostgreSQL (single database, schema per bounded context where practical)
- Redis for session/cache (optional in early MVP)
- REST API only (gRPC/Kafka deferred)

**Exit criteria:** A business owner can sign up, create a store, add products, record a sale, and view a simple sales report.

---

## Phase 2 — Contracts and async foundations

**Goal:** Prepare for scale without splitting deployables yet.

- Define event schemas in `packages/events`
- Define proto files in `packages/proto` (even if unused initially)
- Introduce Kafka for domain events (e.g. `SaleCompleted`, `StockAdjusted`)
- Add outbox pattern or transactional messaging for critical events
- NGINX reverse proxy in local Docker stack

**Exit criteria:** Sale completion publishes an event; a worker can consume it (even if the worker is still in-process).

---

## Phase 3 — Extract first services

**Goal:** Split only where boundaries are clear and load justifies it.

Candidate extractions (order may change):

1. **auth-service** — identity, tokens, permissions
2. **notification-service** — email/SMS (Go worker)
3. **report-service** — read-heavy aggregations (Go)

Keep catalog, inventory, and sales in a core service until transactional complexity demands Spring Boot.

**Exit criteria:** At least one service runs independently with its own database schema and gRPC/HTTP contract.

---

## Phase 4 — Production readiness

**Goal:** Operate reliably in a cloud environment.

- Kubernetes manifests / Helm charts in `infra/`
- Terraform for AWS (VPC, RDS, ElastiCache, MSK or managed Kafka alternative)
- GitHub Actions deploy pipeline
- Observability: structured logs, metrics, tracing
- Rate limiting and security hardening at NGINX + API layer

**Out of scope until business validation:** full tax authority integration, AI insights, mobile apps, payment gateways.

---

## Phase 5 — Advanced capabilities (future)

- AI insights service (aggregated analytics, recommendations)
- Tax-ready export formats per jurisdiction
- Real payment provider integration
- Mobile companion app
- Multi-region deployment

---

## Decision gate before each phase

Ask:

1. Does the current architecture block a concrete user story?
2. Is the operational cost of the next step justified by team size or load?
3. Are boundaries and contracts documented?

If any answer is "no," stay in the current phase and deepen the MVP instead.
