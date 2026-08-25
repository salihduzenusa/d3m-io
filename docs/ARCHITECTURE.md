# Architecture notes

## High-level boxes

```text
┌────────────┐   HTTPS    ┌────────────┐   SQL/Meili   ┌────────────┐
│  Angular   │ ─────────► │  Fastify   │ ────────────► │ PostgreSQL │
│  d3m.io    │            │  api.d3m   │               └────────────┘
└────────────┘            │            │ ────────────► ┌────────────┐
                          │            │               │ Meilisearch│
                          │            │ ────────────► └────────────┘
                          │            │               ┌────────────┐
                          │            │ ────────────► │ Redis/MQ   │
                          └────────────┘               └────────────┘
```

## Listing lifecycle

```mermaid
stateDiagram-v2
  [*] --> pending: create listing
  pending --> active: admin approve
  pending --> rejected: admin reject
  active --> sold: seller marks sold
  active --> expired: TTL / unpublish
  rejected --> pending: edit & resubmit
  sold --> [*]
  expired --> [*]
```

## Credit flow

```mermaid
flowchart LR
  U[User] -->|checkout| S[Stripe]
  S -->|webhook| API[Credits service]
  API -->|FOR UPDATE ledger| DB[(users.credits_balance)]
  U -->|publish / promote| API
```
