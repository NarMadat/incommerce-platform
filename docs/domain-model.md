# Domain Model

This document describes the **core domain** for the MVP. It is intentionally pragmatic: enough structure to guide module boundaries without premature DDD ceremony.

## Ubiquitous language

| Term | Meaning |
|------|---------|
| **Organization** | A business account (tenant). All data is scoped to an organization. |
| **Store** | A physical or logical location belonging to an organization. Inventory and sales are store-scoped. |
| **User** | A person who accesses the platform. Belongs to one or more organizations with a role. |
| **Product** | Something the business sells. Has SKU, name, price, and optional category. |
| **Category** | Hierarchical grouping for products within an organization. |
| **Customer** | A buyer; may be walk-in (anonymous) or registered for repeat sales. |
| **Sale** | A completed transaction at a store: line items, quantities, totals, timestamp. |
| **Inventory movement** | A change to stock: purchase/receipt, sale deduction, adjustment, transfer. |

## Entity relationship (MVP)

```
Organization
 ├── Store (1..*)
 ├── User (membership + role)
 ├── Category (tree, org-scoped)
 ├── Product (org-scoped, optional category)
 ├── Customer (org-scoped)
 └── Sale (store-scoped)
      └── SaleLineItem → Product

Store
 └── InventoryBalance (product + quantity on hand)

InventoryMovement
 ├── store
 ├── product
 ├── quantity (+/-)
 ├── type: IN | OUT | ADJUSTMENT | TRANSFER
 └── reference (optional: saleId, note)
```

## Aggregates and consistency rules

### Organization (aggregate root)

- Owns stores, membership, and org-wide catalog settings.
- **Invariant:** slug/code unique per platform (or per region, TBD at implementation).

### Store

- Belongs to exactly one organization.
- **Invariant:** cannot exist without organization.

### Product

- Belongs to organization; price and SKU unique within org.
- **Invariant:** soft-delete preferred over hard-delete if referenced by sales.

### Inventory

- **InventoryBalance** is derived from movements (or maintained synchronously in MVP).
- **Invariant:** quantity on hand must not go negative unless org setting allows backorders (default: disallow).

### Sale

- Created at a store; lines reference products and quantities at time of sale.
- **Invariant:** completing a sale must create corresponding inventory OUT movements (same transaction in MVP).
- **Invariant:** line prices snapshot at sale time (product price changes do not rewrite history).

### Customer

- Optional on a sale (walk-in allowed).
- **Invariant:** email/phone unique within org when provided.

## Bounded contexts (logical)

| Context | Entities | Notes |
|---------|----------|-------|
| **Identity & access** | User, Role, Session | Auth module; future `auth-service` |
| **Organization** | Organization, Store, Membership | Tenant boundary |
| **Catalog** | Product, Category | Read-heavy; shared by sales and inventory |
| **Inventory** | Balance, Movement | Write-heavy; strong consistency with sales |
| **Sales** | Sale, SaleLineItem, Customer | Orchestrates with inventory |
| **Reporting** | Read models, aggregates | No writes; queries across contexts |

## MVP simplifications

- Single currency per organization (multi-currency later).
- No tax engine — store tax as a simple rate or manual line (export-friendly formatting only).
- No payment capture — sales represent recorded transactions, not card processing.
- Users belong to organizations via a simple join table (no complex RBAC tree yet).

## Events (future)

When Kafka is introduced, candidate domain events:

| Event | Publisher | Consumers |
|-------|-----------|-----------|
| `OrganizationCreated` | Organization | Audit, analytics |
| `ProductCreated` / `ProductUpdated` | Catalog | Search index (future) |
| `StockAdjusted` | Inventory | Reports, alerts |
| `SaleCompleted` | Sales | Inventory (if async), reports, notifications |

Schemas will live in `packages/events`.

## Open questions: questions for Phase 1

- Organization slug vs UUID as public identifier?
- Store required on every sale, or org-level default store?
- Product variants (size/color) in MVP or Phase 2?

Record answers in a new ADR when decided.
