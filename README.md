# inCommerce Platform

inCommerce is a SaaS CRM/ERP platform for small and medium businesses. It helps business owners manage products, inventory, sales, customers, reports, tax-ready exports, and (later) AI-powered business insights.

This repository is a **polyglot monorepo** orchestrated with [Nx](https://nx.dev). We start with a **modular monolith MVP** and evolve toward bounded microservices only when complexity and team scale justify the operational cost.

## Repository layout

```
incommerce-platform/
├── apps/                 # User-facing applications
│   ├── api-gateway/      # Public HTTP API entry point (NestJS)
│   └── web-admin/        # Admin dashboard (future)
├── services/             # Future bounded contexts (not all active in MVP)
├── packages/             # Shared contracts and types
│   ├── proto/            # gRPC / Protobuf definitions
│   ├── events/           # Kafka event schemas
│   └── shared-types/     # Cross-cutting TypeScript types
├── infra/                # Docker, NGINX, K8s, Helm, Terraform
├── docs/                 # Architecture and domain documentation
└── scripts/              # Dev, CI, and database utilities
```

## Architecture principles

- **Start simple** — one deployable MVP before splitting services.
- **Clear boundaries** — folders and contracts reflect future service ownership.
- **Contracts first** — shared types, events, and proto definitions live in `packages/`.
- **Thin controllers** — HTTP/gRPC adapters delegate to application/domain layers.
- **Data ownership** — each future service owns its schema; no shared database tables across contexts.
- **Explicit decisions** — major choices are recorded as ADRs under `docs/architecture/adr/`.

## MVP scope

**In scope:** organizations, stores, users, products, categories, inventory movements, sales, customers, basic reports.

**Out of scope (for now):** tax authority integration, AI assistant, AWS production deployment, Kubernetes, full microservices split, real payments, mobile app.

## Documentation

| Document | Purpose |
|----------|---------|
| [Roadmap](docs/roadmap.md) | Phased delivery plan |
| [Domain model](docs/domain-model.md) | Core entities and relationships |
| [System context](docs/architecture/context.md) | Actors, external systems, high-level view |
| [Quality attributes](docs/architecture/quality-attributes.md) | Non-functional requirements |
| [System boundaries](docs/architecture/system-boundaries.md) | What is inside vs outside the platform |
| [Service boundaries](docs/architecture/service-boundaries.md) | Bounded contexts and ownership |
| [CI/CD strategy](docs/architecture/ci-cd-strategy.md) | Build, test, and deploy approach |
| [ADRs](docs/architecture/adr/) | Architecture decision records |

## Tech stack (target)

| Layer | Technology |
|-------|------------|
| Monorepo | Nx |
| API gateway / early services | NestJS (TypeScript) |
| Complex transactional services | Java / Spring Boot (later) |
| Workers / reports / notifications | Go (later) |
| Database | PostgreSQL |
| Cache / locks / realtime | Redis |
| Async events | Kafka |
| Internal RPC | gRPC |
| Realtime UI | WebSocket |
| Edge / reverse proxy | NGINX |
| Local infra | Docker Compose |
| Production (later) | Kubernetes on AWS |
| CI/CD | GitHub Actions |

## Getting started

The Nx workspace is initialized. Application and service projects will be added incrementally.

```bash
npm install
npx nx graph    # explore project graph (as projects are added)
```

## Current status

**Phase 0 — Foundation:** repository structure and architecture documentation. No business logic yet.

Next step: scaffold the first NestJS modular monolith inside `apps/api-gateway` with organization and auth modules.

## License

MIT
