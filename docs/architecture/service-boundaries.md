# Service Boundaries

Logical **bounded contexts** and their mapping to repository folders. In the MVP, these are **NestJS modules inside one deployable** (`apps/api-gateway`), not separate processes.

## Boundary map

```
┌─────────────────────────────────────────────────────────────────┐
│                     apps/api-gateway (MVP)                       │
│  ┌──────────┐ ┌──────────────┐ ┌─────────┐ ┌───────────┐        │
│  │   Auth   │ │ Organization │ │ Catalog │ │ Inventory │        │
│  └──────────┘ └──────────────┘ └─────────┘ └───────────┘        │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐                            │
│  │  Sales   │ │ Reports  │ │ (shared)│                            │
│  └──────────┘ └──────────┘ └─────────┘                            │
└─────────────────────────────────────────────────────────────────┘
         │ uses                    │ uses
         ▼                         ▼
   packages/shared-types     packages/events (Phase 2)
                             packages/proto (Phase 3)
```

## Context definitions

### Auth (`services/auth-service/` — future)

| Owns | Does not own |
|------|--------------|
| Users, credentials, sessions, JWT issuance | Organization business data |
| Role assignments (reference by userId) | Product or sales data |

**MVP:** `AuthModule` in api-gateway.  
**Extract when:** Dedicated security team, SSO complexity, or independent scaling need.

---

### Organization (`services/organization-service/` — future)

| Owns | Does not own |
|------|--------------|
| Organization, Store, membership | Products, inventory quantities |
| Tenant settings (currency, timezone) | Sales transactions |

**MVP:** `OrganizationModule`.

---

### Catalog (`services/product-service/` — future)

| Owns | Does not own |
|------|--------------|
| Product, Category, pricing (current) | Stock levels |
| SKU uniqueness within org | Customer records |

**MVP:** `CatalogModule` (or `ProductModule`).

---

### Inventory (`services/inventory-service/` — future)

| Owns | Does not own |
|------|--------------|
| InventoryBalance, InventoryMovement | Product definitions |
| Stock reservations (future) | Payment state |

**MVP:** `InventoryModule`. Strong coupling to Sales for synchronous deduction.

---

### Sales (`services/sales-service/` — future)

| Owns | Does not own |
|------|--------------|
| Sale, SaleLineItem, Customer | Product master data |
| Sale lifecycle (draft → completed) | User authentication |

**MVP:** `SalesModule`. Calls inventory application service on completion.

---

### Reports (`services/report-service/` — future)

| Owns | Does not own |
|------|--------------|
| Read models, aggregations, export jobs | Source-of-truth writes |

**MVP:** `ReportsModule` with read-only SQL.  
**Extract to Go when:** Query cost or isolation requirements grow.

---

### Notifications (`services/notification-service/` — future)

| Owns | Does not own |
|------|--------------|
| Delivery attempts, templates, channels | Business decisions to notify |

**MVP:** Not implemented. Folder reserved.

---

### AI Insights (`services/ai-insights-service/` — future)

| Owns | Does not own |
|------|--------------|
| Analytics features, model orchestration | Transactional writes |

**MVP:** Out of scope.

---

## Communication rules

### MVP (in-process)

- Modules communicate via **application service interfaces** injected through NestJS DI.
- No direct cross-module repository access — call the owning module's application service.
- Shared kernel limited to `packages/shared-types` (IDs, money type, pagination).

### Future (distributed)

| From → To | Mechanism | Contract location |
|-----------|-----------|----------|-------------------|
| Sync read | gRPC | `packages/proto` |
| Async notify | Kafka | `packages/events` |
| External clients | REST/WS | OpenAPI at gateway |

## Data ownership (future state)

Each extracted service gets **its own PostgreSQL schema or database**. Until extraction:

- Single database with **table naming or schema prefixes** per context (e.g. `catalog.products`, `sales.sales`).
- Foreign keys across contexts allowed in monolith; replace with IDs + eventual consistency when split.

## Anti-patterns to avoid

- **God module** — one `BusinessService` for everything.
- **Shared repository** — Inventory module querying `products` table via Sales repository.
- **Distributed monolith early** — eight empty microservices with network calls and no team scale.
- **Logic in controllers** — HTTP adapters should only map DTOs and call application layer.

## Extraction checklist

Before moving a context to `services/<name>/`:

1. Module has clear public application API (not leaking entities).
2. Proto/event contracts defined in `packages/`.
3. Data access confined to module; migration path for DB split documented.
4. Load or team ownership justifies deploy and ops overhead.

## Related documents

- [Domain model](../domain-model.md)
- [ADR-003: Start with modular MVP](./adr/ADR-003-start-with-modular-mvp.md)
