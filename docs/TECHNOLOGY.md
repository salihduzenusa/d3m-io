# Technology deep dive

## Frontend (`d3msource`)

- **Angular 21** standalone components
- Design system in SCSS (`assets/scss`)
- **ngx-translate** with broad locale coverage
- Mobile-first chrome: header bar, bottom nav, listing detail overlays
- Guards for auth / admin / credits flows

## Backend (`d3mapi`)

- **Fastify 5** TypeScript API under `/api/v1`
- **Zod** request schemas on write paths
- Modules: auth, listings, ads, messages, credits, admin, media, geo, search
- **PostgreSQL** with migrations; category tables for listings scale
- **Meilisearch** optional browse acceleration + BullMQ sync worker
- **Redis** cache / queues
- **Stripe** Checkout + webhooks when keys are real (not placeholders)

## Security controls (current)

| Control | Status |
|---------|--------|
| Parameterized SQL | ✅ |
| bcrypt cost 12 | ✅ |
| Helmet + CORS allowlist | ✅ |
| Stripe webhook signatures | ✅ |
| Participant checks on messages | ✅ |
| Server-side OTP password reset | ⚠️ planned hardening |
| Production JWT secret rotation | ⚠️ planned hardening |

## Deploy

```text
npm run build:prod   (d3msource)
  └─ rsync → d3m.io + d3mmanager

npm run build && pm2 restart   (d3mapi)
```

Scripts live under `scripts/` (`d3m-build-deploy.sh`, `d3m-api-build-deploy.sh`, `d3m-github-backup.sh`).
