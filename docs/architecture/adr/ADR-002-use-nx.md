# ADR-002: Use Nx for Monorepo Orchestration

## Status

Accepted

## Date

2026-05-24

## Context

The platform will contain multiple applications, libraries, and (later) non-Node projects. We need:

- A **project graph** and dependency awareness
- **Affected** commands for CI (only build/test what changed)
- Consistent task running (`build`, `test`, `lint`)
- Optional remote caching for CI speed

Alternatives considered:

1. **npm/pnpm workspaces alone** — minimal, no graph intelligence
2. **Turborepo** — strong for JS/TS pipelines
3. **Bazel** — powerful, steep learning curve
4. **Nx** — graph, plugins, polyglot path, strong CI story

## Decision

Use **Nx** (v22+) as the monorepo orchestrator. The workspace is already initialized at the repository root.

Initial scope:

- TypeScript projects in `apps/` and `packages/`
- `@nx/js` plugin for libraries and inferred tasks
- GitHub Actions workflow using `nx affected`

Future:

- `@nx/nest` for NestJS applications
- `@nx/react` or similar for `web-admin`
- Custom executors or plugins for Java/Go as needed

## Consequences

### Positive

- Built-in project graph visualization (`nx graph`)
- Affected detection reduces CI time as repo grows
- Nx Cloud remote cache available (workspace already linked)
- Large ecosystem of generators for NestJS, React, etc.
- Industry-recognized tool for portfolio projects

### Negative

- Learning curve for `project.json`, plugins, and inferred tasks
- Configuration can feel heavy for a tiny repo early on
- Non-JS projects require explicit integration (not zero-config)

### Mitigations

- Start with few projects; add generators when scaffolding apps
- Document common commands in README
- Defer Java/Go Nx integration until those folders have code

## Trade-offs

| Tool | Best for | Why not chosen (or chosen) |
|------|----------|----------------------------|
| Workspaces only | 1–2 packages | No affected graph; CI runs everything |
| Turborepo | JS-heavy monorepos | Less mature polyglot story for our roadmap |
| Bazel | Huge multi-language repos | Overkill for MVP; slower team onboarding |
| **Nx** | Evolving monorepo + CI | **Chosen** — balance of power and approachability |

## Implementation notes

- Root `package.json` workspaces: `packages/*` (extend when apps use workspace protocol)
- Run `npx nx sync:check` in CI once multiple TS projects exist
- Prefer Nx generators over manual folder creation for apps/libs going forward

## References

- [CI/CD strategy](../ci-cd-strategy.md)
- [Nx documentation](https://nx.dev)
