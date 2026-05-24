# Quality Attributes

Non-functional requirements that guide architectural choices. Priorities are ordered for the **MVP phase**; later phases may rebalance.

## Priority ranking (MVP)

1. **Correctness** — inventory and sales must stay consistent.
2. **Security & tenant isolation** — no cross-organization data leaks.
3. **Maintainability** — clear modules, testable domain logic.
4. **Availability** — reasonable uptime for SMB use (not five-nines yet).
5. **Performance** — responsive admin UI; sub-second API for common operations.
6. **Scalability** — design for growth; do not over-provision infra early.

## Attribute details

### Correctness & consistency

- Sale completion and inventory deduction occur in **one database transaction** in the modular monolith.
- Monetary values use decimal types (not floating point).
- Idempotency keys for critical write APIs (Phase 1+ where retries matter).

**Trade-off:** Strong consistency in one DB is simpler than sagas across microservices — acceptable for MVP.

### Security

- JWT or session-based auth; passwords hashed (bcrypt/argon2).
- RBAC: at minimum `OWNER`, `STAFF`, `READ_ONLY`.
- All repository queries filter by tenant context from authenticated user.
- Secrets via environment variables locally; AWS Secrets Manager in production (later).

### Maintainability

- NestJS modules aligned to bounded contexts (see [service-boundaries](./service-boundaries.md)).
- Business rules in application/domain services, not controllers.
- Shared types in `packages/shared-types`; avoid copy-paste DTOs.

### Performance

- Target: p95 < 300ms for CRUD on warm DB (single region, modest data).
- Reports may use read-optimized queries; heavy analytics deferred to `report-service` (Go) later.
- Redis for caching hot catalog data when measured need exists — not by default on day one.

### Scalability

- Stateless API instances behind load balancer (when horizontally scaled).
- Kafka for async decoupling before splitting services.
- PostgreSQL vertical scale first; read replicas when report load demands.

### Observability

- Structured JSON logs with `correlationId`, `organizationId`.
- Health checks: `/health`, `/ready`.
- Metrics and distributed tracing in Phase 4 (Prometheus/OpenTelemetry).

### Reliability

- Database backups (daily minimum in production).
- Graceful shutdown for in-flight requests.
- Retry with backoff for external integrations only (not for internal DB writes).

## Quality scenarios

| Scenario | Stimulus | Response |
|----------|----------|----------|
| Concurrent sales | Two staff sell last unit | One succeeds; other gets clear out-of-stock error |
| Tenant isolation | User A queries org B id | 404 or 403; no data leakage |
| Report generation | 10k sales/month org | Summary report < 5s (MVP); optimize in Phase 3 |
| Deploy new version | Rolling deploy | Zero-downtime goal for Phase 4; acceptable brief restart in MVP |

## Explicit non-goals (MVP)

- Sub-100ms global latency
- Infinite horizontal scale on day one
- 99.99% SLA
- Real-time analytics on every keystroke

## Review cadence

Revisit this document when:

- Extracting the first microservice
- Adding Kafka consumers
- Onboarding > 100 concurrent tenants or > 1M transactions/month
