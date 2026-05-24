# ADR-003: Start with Modular MVP (Avoid Premature Microservices)

## Status

Accepted

## Date

2026-05-24

## Context

The target architecture includes multiple microservices (auth, organization, product, inventory, sales, report, notification, AI), Kafka, gRPC, Redis, and Kubernetes.

Building all of that on day one would:

- Delay first user value by months
- Increase operational burden (eight deployables, eight databases, network failures)
- Encourage **distributed monolith** anti-patterns (chatty services, shared DB)
- Make debugging and local development painful for a small team

The business need is an MVP for SMBs: orgs, stores, products, inventory, sales, customers, basic reports.

Alternatives considered:

1. **Full microservices from start** — match final diagram immediately
2. **Unstructured monolith** — single `app.ts` with no module boundaries
3. **Modular monolith MVP** — one deployable, NestJS modules = future service boundaries

## Decision

Implement the MVP as a **modular monolith** in `apps/api-gateway`:

- One NestJS application, one PostgreSQL database (with logical schema separation)
- Modules aligned to bounded contexts (Auth, Organization, Catalog, Inventory, Sales, Reports)
- Empty `services/*` folders document **future** extraction targets, not current deployables
- Defer Kafka, gRPC, and K8s until Phase 2–4 (see [roadmap](../../roadmap.md))

Extract a service only when the [extraction checklist](../service-boundaries.md#extraction-checklist) is satisfied.

## Consequences

### Positive

- Fast path to working product and portfolio demo
- ACID transactions across sales + inventory without sagas
- Simple local dev: one process + Postgres
- Module boundaries preserve option to split later
- Lower cloud cost during validation

### Negative

- Cannot scale one module independently until extracted
- Risk of boundary erosion if modules share repositories carelessly
- Some rework when splitting (DB migration, RPC wiring)
- `services/` folders may look "empty" to newcomers

### Mitigations

- Enforce module APIs in code review
- Document boundaries in [service-boundaries.md](../service-boundaries.md)
- Introduce `packages/events` before first extraction
- Add integration tests at module boundaries

## Trade-offs

| Strategy | Time to MVP | Ops complexity | Split cost later |
|----------|-------------|----------------|------------------|
| Microservices first | High | High | Low |
| Unstructured monolith | Low | Low | **Very high** |
| **Modular monolith (chosen)** | **Medium-low** | **Low** | **Medium** |

## Trigger conditions for first extraction

Consider splitting **auth** or **report** first when one or more apply:

- Independent scaling requirement measured in production
- Different release cadence needed (e.g. security patches for auth)
- Team grows to own a service end-to-end
- Report queries impact OLTP latency despite optimization

Do **not** split because the folder structure exists.

## References

- [System boundaries](../system-boundaries.md)
- [Domain model](../../domain-model.md)
- [Roadmap Phase 1–3](../../roadmap.md)
