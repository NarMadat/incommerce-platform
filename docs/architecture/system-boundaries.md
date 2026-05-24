# System Boundaries

Defines what is **inside** the inCommerce platform versus **outside**, and how data and control flow across the boundary.

## Platform boundary (inside)

Everything required to deliver MVP value to SMB users:

| Capability | Inside boundary | MVP delivery |
|------------|-----------------|--------------|
| User authentication & sessions | Yes | NestJS auth module |
| Organization & store management | Yes | NestJS org module |
| Product catalog | Yes | Nest capability |
| Inventory tracking | Yes | Inventory module |
| Sales & customers | Yes | Sales module |
| Basic reports | Yes | In-process queries |
| Admin web UI | Yes | `web-admin` app |
| Public HTTP API | Yes | `api-gateway` (modular monolith initially) |

## Outside the platform (MVP)

| System | Relationship | When |
|--------|--------------|------|
| Tax authority APIs | Future export consumer | Phase 5+ |
| Payment gateways (Stripe, etc.) | Future integration | Phase 5+ |
| Email/SMS providers | Future notification service | Phase 3+ |
| Identity providers (Google SSO) | Optional future | Phase 2+ |
| Mobile app stores | Separate client | Phase 5+ |

## Infrastructure boundary

### Local development (Phase 0–1)

```
Developer machine
 └── Docker Compose
      ├── PostgreSQL
      └── Redis (optional early)
 └── npm / Nx
      └── apps/api-gateway (NestJS)
      └── apps/web-admin (future)
```

### Staging / production (Phase 4+)

```
Internet
 └── NGINX (TLS termination, routing, rate limits)
      └── API pods (Kubernetes)
           └── Services (monolith or split)
      └── Static admin assets (CDN or NGINX)
 └── Managed PostgreSQL (RDS)
 └── Redis (ElastiCache)
 └── Kafka (MSK or alternative)
```

## Data boundaries

| Data type | Stored where | Notes |
|-----------|--------------|-------|
| Tenant business data | PostgreSQL | Org-scoped rows |
| Sessions / cache | Redis | Ephemeral |
| Domain events | Kafka | Durable log (Phase 2+) |
| User-uploaded files (future) | S3 | Not in MVP |
| Secrets | Env / Secrets Manager | Never in repo |

## API boundary

**Northbound (clients → platform):**

- REST JSON over HTTPS (MVP)
- WebSocket for live dashboard (Phase 2+)
- OpenAPI spec published from `api-gateway`

**Southbound (platform → external):**

- None in MVP
- Future: webhooks and provider SDKs from worker/notification services only

**East/West (internal, future):**

- gRPC between services using contracts in `packages/proto`
- Events via Kafka using schemas in `packages/events`

## Security boundary

- **Trust zone 1:** Public clients (browser, future mobile) — untrusted input, strict validation.
- **Trust zone 2:** API layer — authentication, authorization, rate limiting.
- **Trust zone 3:** Domain/application services — business invariants.
- **Trust zone 4:** Data stores — credentials only from application layer.

Cross-zone rule: controllers never access the database directly; they call application services.

## What we deliberately defer

Splitting deployables, Kubernetes, and cloud IaC are **outside the MVP boundary** but **inside the repository** as placeholders (`infra/`, `services/`) so the target architecture stays visible without incurring operational cost today.

## Related documents

- [Context](./context.md)
- [Service boundaries](./service-boundaries.md)
- [ADR-003: Start with modular MVP](./adr/ADR-003-start-with-modular-mvp.md)
