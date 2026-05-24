# ADR-001: Polyglot Monorepo

## Status

Accepted

## Date

2026-05-24

## Context

inCommerce targets a multi-service architecture over time:

- TypeScript / NestJS for API and rapid feature delivery
- Java / Spring Boot for complex transactional domains
- Go for lightweight workers and read-heavy paths
- Shared contracts (proto, events) consumed by all runtimes

We need one repository that:

- Keeps contracts in sync across languages
- Enables atomic changes across API and consumers
- Supports portfolio demonstration of real-world platform design

Alternatives considered:

1. **Polyrepo** — one repo per service and per package
2. **Monolith repo, single language** — TypeScript only forever
3. **Polyglot monorepo** — multiple runtimes, one repo, shared tooling

## Decision

Adopt a **polyglot monorepo** named `incommerce-platform` with language-specific projects under `apps/`, `services/`, and `packages/`.

Shared artifacts:

- `packages/proto` — gRPC definitions
- `packages/events` — Kafka event schemas
- `packages/shared-types` — TypeScript types for web and NestJS

Language-specific services live in dedicated folders even before they contain code.

## Consequences

### Positive

- Contract changes and implementations can land in one PR
- Single CI pipeline with Nx affected graph
- Clear story for recruiters and production evolution
- Easier local development (one clone)

### Negative

- CI complexity grows as languages are added (Node, Java, Go toolchains)
- Developers need broader stack awareness
- Repository size and clone time increase over time
- Risk of tight coupling if teams skip proper module boundaries

### Mitigations

- Add languages **incrementally** (NestJS first)
- Enforce boundaries via docs, code review, and module APIs
- Use Nx to run only affected projects
- Keep generated code out of hand-edited paths where possible

## Trade-offs

| Approach | Pros | Cons |
|----------|------|------|
| Polyrepo | Independent deploy cadence per team | Contract drift, versioning pain |
| TS-only monorepo | Simplest tooling | Wrong tool for heavy reporting/workers later |
| **Polyglot monorepo (chosen)** | Unified evolution, shared contracts | Higher initial orchestration cost |

## References

- [Service boundaries](../service-boundaries.md)
- [ADR-002: Use Nx](./ADR-002-use-nx.md)
