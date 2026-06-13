---
name: nexara-stack-selection
description: Selects the recommended Nexara technology stack, architecture template (A-F), and backend mode (1-6) for a software project, with a hosting cost estimate. Use this skill whenever a new Nexara project is being scoped, planned, or quoted; whenever someone asks what stack to use or how to build something; or whenever someone describes something to be built. Also use when a hosting or monthly-cost estimate is requested.
---

# Nexara Stack Selection

Nexara uses one engineering system with several architecture templates, tweaked by client budget, scale, compliance, and team skill. The default backend is Convex (reactive DB + serverless functions) with Clerk for auth. Use PostgreSQL only for SQL-heavy workloads, on-prem, or enterprise.

## How to use this skill

1. Read the project description. Ask only for missing facts that change the template branch (login required? on-prem? AI core? expected scale? budget?).
2. Walk the decision procedure below in order, stop at the first match.
3. Produce the recommendation using the output format.
4. Pull full detail from `references/decision-matrix.md` and `references/templates-and-costs.md` when needed.

## Decision procedure

**Q1 - Is it a mobile app?**
- Mobile, no deep native features -> Template D (Expo React Native + Convex + Clerk). Convex React Native SDK works natively.
- Mobile, deep native features -> Native Kotlin + Swift. Custom scoping.
- Not mobile -> Q2.

**Q2 - Is it primarily a content/marketing site with no user login?**
- Yes, devs manage content -> Template A / Next.js (SSG) + Cloudflare Pages. $0-$5/mo.
- Yes, client self-edits -> Template A / WordPress or Webflow. $5-$50/mo.
- Yes, may need login later -> Template A / Next.js (grows into B without rewrite).
- No -> Q3.

**Q3 - Is it an ecommerce store?**
- Standard store -> Shopify or WooCommerce.
- Custom marketplace -> Template B (Next.js + Convex) or Laravel + PostgreSQL. $50-$200/mo.
- No -> Q4.

**Q4 - Does the client require on-premises or private-server deployment?**
- Yes -> Template F (Docker Compose + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak) + Mode 3 (NestJS) or Mode 4 (FastAPI). Convex cannot be used on-prem.
- No -> Q5.

**Q5 - Is AI a core feature, not just a minor add-on?**
- Yes -> Template E on B or C + Convex actions for AI routing + Convex vector search (or pgvector on Neon for large-scale retrieval). Base hosting + AI usage.
- No -> Q6.

**Q6 - What matters most: lowest cost or developer experience?**
- Lowest cost + edge -> Template C (Cloudflare Workers + Hono + D1/R2) + Mode 2. $5-$30/mo.
- Standard SaaS/portal -> Template B (Next.js + Convex + Clerk) + Mode 1. $25-$75/mo.
- Enterprise/complex -> Template B frontend + Mode 3 (NestJS), Mode 5 (Spring Boot/.NET), or Mode 6 (Laravel). $50+/mo.

## Cross-cutting rules

- Default language: TypeScript for web/mobile/APIs; Python where AI/data is central.
- Default database: Convex for new apps; PostgreSQL for SQL-heavy or on-prem workloads.
- Default auth: Clerk + Convex integration. NextAuth.js for self-hosted. Keycloak for enterprise SSO.
- AI is always backend-routed through Convex actions or backend routes - never from the frontend.
- Multi-tenant with Convex: orgId/tenantId field on every tenant-owned table; all queries filtered by orgId; indexes on orgId.
- Cheapest-good default for small clients: Convex free tier + Vercel/Cloudflare + Clerk free + Resend + Sentry free.

## Output format

```
## Recommendation: [Project name]

Template: [A-F] - [one-line name]
Backend mode: [1-6]
Recommended stack:
- Frontend: ...
- Backend/DB: ...
- Auth: ...
- Hosting: ...
- Storage / Email / Payments / AI: [relevant ones only]

Starting cost target: $X-$Y/month

Why this fits: [2-4 sentences]

Watch-outs / when to revisit: [e.g. switch to Mode 3 if background jobs grow complex]
```

See `references/decision-matrix.md` and `references/templates-and-costs.md` for full detail.
