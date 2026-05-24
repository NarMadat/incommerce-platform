# System Context

C4 Level 1 — who uses inCommerce and what external systems interact with it.

## Purpose

inCommerce helps SMB owners run day-to-day operations: catalog, stock, sales, customers, and basic reporting — with a path toward tax exports and AI insights.

## Context diagram

```
                    ┌─────────────────────────────────────────┐
                    │           inCommerce Platform            │
                    │  (API, admin UI, domain services)        │
                    └─────────────────────────────────────────┘
                           ▲              ▲           ▲
                           │              │           │
              ┌────────────┘              │           └────────────┐
              │                           │                        │
     ┌────────┴────────┐        ┌─────────┴─────────┐    ┌─────────┴─────────┐
     │ Business Owner  │        │   Store Staff     │    │  Accountant       │
     │ (admin user)    │        │   (staff user)    │    │  (read-only export)│
     └─────────────────┘        └───────────────────┘    └───────────────────┘

Future external systems (out of MVP scope):

     ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
     │ Tax authority│   │ Payment      │   │ Email / SMS    │
     │ APIs         │   │ providers    │   │ providers      │
     └──────────────┘   └──────────────┘   └──────────────┘
            ▲                  ▲                  ▲
            └──────────────────┴──────────────────┘
                          (Phase 4+)
```

## Actors

| Actor | Goal | MVP access |
|-------|------|------------|
| **Business owner** | Configure org, stores, catalog; view reports | Full admin |
| **Store staff** | Record sales, adjust stock, manage customers | Limited write |
| **Accountant** (future) | Export tax-ready data | Read-only exports |

## System responsibilities

**In scope (platform):**

- Multi-tenant organization and store management
- Product catalog and inventory tracking
- Sales recording and customer history
- Basic operational reports
- Authentication and authorization

**Out of scope (MVP):**

- Legal tax filing to government APIs
- Payment card processing
- AI recommendations
- Native mobile apps

## Deployment context (evolution)

| Stage | Runtime | Audience |
|-------|---------|----------|
| MVP | Single NestJS process + PostgreSQL; local Docker | Developers, pilot users |
| Phase 2–3 | Modular monolith + Kafka + Redis; optional service split | Staging |
| Phase 4+ | Kubernetes on AWS, NGINX edge | Production SMB tenants |

## Trust boundaries

- **Public internet → NGINX → API:** authenticated REST (later WebSocket for dashboard).
- **Internal services → gRPC:** mTLS or mesh policy (when services split).
- **Tenant isolation:** every query scoped by `organizationId` (and often `storeId`).

## Related documents

- [System boundaries](./system-boundaries.md)
- [Service boundaries](./service-boundaries.md)
- [Quality attributes](./quality-attributes.md)
