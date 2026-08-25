<p align="center">
  <img src="docs/assets/d3m-logo-light.png" alt="D3M" width="160"/>
</p>

<h1 align="center">D3M.io</h1>

<p align="center">
  <strong>Global classifieds marketplace</strong> — vehicles, real estate, jobs, services, community & more.<br/>
  Multi-country · multi-language · credit-based publishing · real-time search.
</p>

<p align="center">
  <a href="https://d3m.io"><img src="https://img.shields.io/badge/Live-d3m.io-4f46e5?style=for-the-badge" alt="Live"/></a>
  <a href="https://api.d3m.io"><img src="https://img.shields.io/badge/API-api.d3m.io-0ea5e9?style=for-the-badge" alt="API"/></a>
  <img src="https://img.shields.io/badge/Platform-Angular_21_%7C_Fastify_5-111827?style=for-the-badge" alt="Stack"/>
  <img src="https://img.shields.io/badge/License-Proprietary-64748b?style=for-the-badge" alt="License"/>
</p>

<p align="center">
  <img src="docs/assets/hero-banner.png" alt="D3M hero" width="920"/>
</p>

---

## Story

**D3M** started as a modern alternative to fragmented local classifieds boards — one product that can run in many markets without rewriting the stack each time.

The platform is built for:

- **Sellers** who need fast listing, promotion credits, and messaging
- **Buyers** who need filters, map/geo search, and trustworthy detail pages
- **Operators** who need country-scoped admin, moderation, and billing controls

Today the live product serves **d3m.io** with a Fastify API at **api.d3m.io**, Angular SPA, PostgreSQL, Meilisearch, Redis, and Stripe-ready credits.

> This public repository is the **product & architecture showcase**.  
> Full source disaster-recovery backup lives in a **private** repo (`d3m-backup`).

---

## Product surface

| Area | What users get |
|------|----------------|
| Marketplace | Goods for sale / rent with rich filters |
| Vehicles | Cars, boats, and catalog-assisted attributes |
| Real estate | Property listings with geo hierarchy |
| Jobs & services | Hiring and local professional services |
| Community | Local notices, lost & found, meetups |
| Messages | Listing-scoped buyer ↔ seller chat |
| Credits & ads | Publish / promote with wallet + Stripe checkout |
| Admin | Country-scoped staff, moderation, users, ads |

<p align="center">
  <img src="docs/assets/icons/marketplace.svg" width="48" alt="Marketplace"/>
  &nbsp;&nbsp;
  <img src="docs/assets/icons/vehicles.svg" width="48" alt="Vehicles"/>
  &nbsp;&nbsp;
  <img src="docs/assets/icons/real-estate.svg" width="48" alt="Real estate"/>
  &nbsp;&nbsp;
  <img src="docs/assets/icons/jobs.svg" width="48" alt="Jobs"/>
</p>

---

## Architecture

```mermaid
flowchart TB
  subgraph Clients
    WEB["Angular SPA<br/>d3m.io / d3mmanager"]
    MOB["Mobile browsers<br/>bottom nav + PWA-ready UX"]
  end

  subgraph Edge
    CF["Cloudflare / TLS"]
  end

  subgraph API["classifieds-api · Fastify 5"]
    REST["REST /api/v1"]
    AUTH["JWT auth · roles · country admin"]
    MEDIA["Multipart media uploads"]
    WH["Stripe webhooks"]
  end

  subgraph Data
    PG[(PostgreSQL)]
    MEILI[(Meilisearch)]
    REDIS[(Redis · BullMQ)]
  end

  subgraph Workers
    SYNC["Meili sync worker"]
  end

  WEB --> CF --> REST
  MOB --> CF
  REST --> AUTH
  REST --> MEDIA
  REST --> PG
  REST --> MEILI
  REST --> REDIS
  WH --> REST
  REDIS --> SYNC --> MEILI
  SYNC --> PG
```

### Request path (browse)

```mermaid
sequenceDiagram
  participant U as Browser
  participant A as Angular SPA
  participant F as Fastify API
  participant M as Meilisearch
  participant P as PostgreSQL

  U->>A: Open category / filters
  A->>F: GET /listings?...
  alt Meili browse enabled
    F->>M: Search + filters
    M-->>F: Hits
  else SQL fallback
    F->>P: Parameterized browse query
    P-->>F: Rows
  end
  F-->>A: Cards + facets
  A-->>U: Grid / map UI
```

---

## Technology stack

```mermaid
mindmap
  root((D3M.io))
    Frontend
      Angular 21
      Angular Material / CDK
      ngx-translate · 60+ locales
      SCSS design system
    Backend
      Fastify 5
      Zod validation
      JWT · bcrypt
      Helmet · CORS · rate limit
    Data
      PostgreSQL
      Meilisearch
      Redis · BullMQ
    Payments
      Stripe Checkout
      Credits ledger
    Ops
      PM2
      Apache / .htaccess
      Cloudflare
```

| Layer | Choices |
|-------|---------|
| Frontend | Angular 21, standalone components, i18n |
| API | Node.js, Fastify 5, TypeScript, Zod |
| DB | PostgreSQL (category-partitioned listings) |
| Search | Meilisearch + SQL fallback |
| Queue | Redis + BullMQ (index sync) |
| Auth | JWT access/refresh, role + country admin |
| Media | Local uploads → `/api/v1/media/files` |
| Billing | Credits wallet, Stripe when configured |
| Deploy | `d3msource` → `d3m.io` / `d3mmanager`, PM2 API |

---

## Monorepo layout (production tree)

```text
public_html/
├── d3msource/     # Angular source of truth
├── d3mapi/        # Fastify API + workers + migrations
├── d3m.io/        # Production static deploy
├── d3mmanager/    # Staging / manager deploy
└── scripts/       # build, deploy, checkpoint, GitHub backup
```

---

## Status & links

| Resource | URL |
|----------|-----|
| Production | https://d3m.io |
| API | https://api.d3m.io |
| This showcase | https://github.com/salihduzenusa/d3m-io |
| Private DR backup | https://github.com/salihduzenusa/d3m-backup *(private)* |
| Owner | [@salihduzenusa](https://github.com/salihduzenusa) |

---

## Roadmap highlights

- [x] Multi-category classifieds + geo hierarchy
- [x] Meilisearch browse path
- [x] Credits + Stripe checkout path
- [x] Mobile bottom nav / messaging UX
- [ ] Hardened auth (server OTP reset, rotated JWT secrets)
- [ ] Stronger upload MIME sniffing & listing image allowlist
- [ ] Public developer docs / OpenAPI export

---

## License

**Proprietary** — © D3M / salihduzenusa.  
Source and trademarks are not open source. Contact the owner for partnership or licensing.
