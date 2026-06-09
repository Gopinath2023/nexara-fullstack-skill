---
name: nexara-stack-selection
description: Selects the recommended Nexara technology stack, architecture template (A-F), and backend mode (1-6) for a software project, with a hosting cost estimate, using the Nexara Standard Tech Stack Playbook. Use this skill whenever a new Nexara project is being scoped, planned, or quoted; whenever someone asks "what stack should we use", "which template", or "how should we build this"; or whenever someone describes something to be built. Also use it when a hosting or monthly-cost estimate for a project is requested.
---

# Nexara Stack Selection

Nexara does not sell one tech stack. It sells one engineering system with several architecture templates, then tweaks based on client budget, scale, compliance, and team skill. This skill turns a project description into a concrete recommendation: a template, a backend mode, a default stack, and a starting cost target.

## How to use this skill

1. Read the project description the user gives you. If key facts are missing (login required? on-prem? AI core? expected scale? budget?), ask only for the ones that actually change the branch you land on.
2. Walk the decision procedure below in order, stopping at the first match.
3. Produce the recommendation using the output format at the end.
4. Pull stack details from `references/decision-matrix.md` and `references/templates-and-costs.md` when needed.

## Decision procedure

**Q1 - Is it a mobile app?**
- Mobile, no deep native features -> Template D (Expo React Native) + shared backend from B or C.
- Mobile, deep native features -> Native Kotlin + Swift. Custom scoping.
- Not mobile -> Q2.

**Q2 - Is it primarily a content/marketing site with no user login?**
- Yes, devs manage content -> Template A / Next.js (SSG) + Cloudflare Pages. $0-$5/mo.
- Yes, client self-edits -> Template A / WordPress or Webflow. $5-$50/mo.
- Yes, may need login later -> Template A / Next.js (grows into B without rewrite).
- No -> Q3.

**Q3 - Is it an ecommerce store?**
- Standard store -> Shopify or WooCommerce.
- Custom marketplace -> Template B or Laravel + PostgreSQL. $50-$200/mo.
- No -> Q4.

**Q4 - Does the client require on-premises or private-server deployment?**
- Yes -> Template F (Docker Compose + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak) + Backend Mode 3 (NestJS) or Mode 4 (FastAPI) if AI-heavy.
- No -> Q5.

**Q5 - Is AI a core feature, not just a minor add-on?**
- Yes -> Template E on B or C + Backend Mode 4 (FastAPI) + pgvector. Base hosting + AI usage.
- No -> Q6.

**Q6 - What matters most: lowest cost or developer experience?**
- Lowest cost + edge -> Template C (Cloudflare Workers + Hono) + Backend Mode 2. $5-$30/mo.
- Standard SaaS/portal -> Template B (Next.js + PostgreSQL + Supabase) + Backend Mode 1. $25-$75/mo.
- Enterprise/complex -> Template B frontend + Mode 3 (NestJS), Mode 5 (Spring Boot/.NET), or Mode 6 (Laravel). $50+/mo.

## Cross-cutting rules

- Default language: TypeScript for web/mobile/APIs; Python where AI/data is central.
- Default database: PostgreSQL.
- AI is always backend-routed - never from the frontend.
- Multi-tenant: shared DB with tenant_id on every tenant-owned table.
- Cheapest-good default: Cloudflare + Supabase/Neon + R2 + Resend + Sentry free tier.

## Output format

```
## Recommendation: [Project name]

**Template:** [A-F] - [one-line name]
**Backend mode:** [1-6]
**Recommended stack:**
- Frontend: ...
- Backend/API: ...
- Database: ...
- Auth: ...
- Hosting: ...
- Storage / Email / Payments / AI: [relevant ones only]

**Starting cost target:** $X-$Y/month

**Why this fits:** [2-4 sentences]

**Watch-outs / when to revisit:** [e.g. move to Mode 3 if background jobs appear]
```

See `references/decision-matrix.md` and `references/templates-and-costs.md` for full detail.
