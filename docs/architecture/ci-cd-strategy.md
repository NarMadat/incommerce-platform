# CI/CD Strategy

How we build, verify, and deploy inCommerce across monorepo phases.

## Goals

- **Fast feedback** on pull requests (lint, typecheck, unit tests).
- **Affected-only execution** via Nx to avoid rebuilding the entire polyglot repo.
- **Reproducible builds** from locked dependencies.
- **Progressive rollout** — local → CI → staging → production (production in Phase 4).

## Pipeline overview

```
PR opened / push
      │
      ▼
┌─────────────┐
│   Install   │  npm ci (Node workspaces)
└──────┬──────┘
       ▼
┌─────────────┐
│  Nx affected │  lint, typecheck, test, build
└──────┬──────┘
       ▼
┌─────────────┐
│   Quality   │  format check, nx sync:check (when TS projects exist)
└──────┬──────┘
       ▼
┌─────────────┐     main branch only (later)
│   Deploy    │  Docker image → registry → K8s
└─────────────┘
```

## GitHub Actions (`.github/workflows/`)

### Phase 0–1: CI only

| Workflow | Trigger | Jobs |
|----------|---------|------|
| `ci.yml` | PR + push to `main` | install, nx affected lint/test/build |

Existing Nx-generated `ci.yml` is a starting point; extend as apps are added.

### Phase 2+: Extended CI

- Integration tests with Docker Compose (PostgreSQL service container).
- Contract tests for `packages/events` and `packages/proto` (breaking change detection).
- Optional Nx Cloud remote cache (already configured via `nxCloudId`).

### Phase 4: CD

- Build and push Docker images per app (`apps/api-gateway`, etc.).
- Deploy to staging on merge to `main`; production on tagged release.
- Terraform plan/apply in guarded workflow with manual approval.

## Nx commands (conventions)

```bash
# Local — same as CI intent
npx nx affected -t lint test build --base=origin/main

# Verify TS project references
npx nx sync:check
```

When Java and Go projects are added, register them as Nx projects (via plugins or `project.json`) so `affected` includes them.

## Branch strategy

| Branch | Purpose |
|--------|---------|
| `main` | Always deployable; protected |
| `feature/*` | Short-lived feature work |
| `release/*` (later) | Stabilization before tag |

## Environments

| Environment | Purpose | Infra |
|-------------|---------|-------|
| **local** | Developer machines | Docker Compose |
| **ci** | Ephemeral PR validation | GitHub Actions runners |
| **staging** (Phase 4) | Pre-production integration | AWS / K8s |
| **production** (Phase 4) | Customer-facing | AWS / K8s |

## Secrets and configuration

- **Never** commit secrets; use `.env.example` templates.
- CI: GitHub Actions secrets for registry, DB URLs (staging).
- Runtime: environment variables; AWS Secrets Manager in production.

## Docker strategy

| Image | When |
|-------|------|
| `api-gateway` | Phase 1 — single Dockerfile in `apps/api-gateway` |
| `web-admin` | Phase 1 — static build served by NGINX |
| Workers (Go) | Phase 3 |

Base images: official Node LTS, Alpine where appropriate; scan in CI (Trivy) in Phase 4.

## Deployment strategy (Phase 4+)

- **Rolling updates** for stateless API.
- **Database migrations** run as init job or dedicated migration step before traffic shift (Flyway/Liquibase/Prisma — choice at implementation).
- **Rollback:** previous image tag; backward-compatible migrations only.

## Quality gates

PR cannot merge if:

- CI workflow fails
- Required reviewers approve (team policy)
- No unresolved blocking comments

Future: minimum coverage threshold per critical module (inventory, sales).

## Observability in pipeline

- Publish test results and build artifacts
- Slack/email on `main` failure (Phase 2+)
- Deployment annotations linked to git SHA

## Related documents

- [ADR-002: Use Nx](./adr/ADR-002-use-nx.md)
- [Roadmap](../roadmap.md) — Phase 4 production readiness
