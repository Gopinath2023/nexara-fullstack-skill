# Templates, Backend Modes & Costs (reference)

Load this to fill in the "Recommended stack" lines of a recommendation once you've picked a template via the SKILL.md flowchart.

## Common foundation (applies across templates)

- **Language:** TypeScript for web/mobile/most APIs; Python where AI/data is central.
- **UI:** Tailwind CSS + shadcn/ui. No other component library alongside it (no MUI, Chakra, Ant Design).
- **Database (cloud SaaS, Mode 1):** Convex. No ORM needed — Convex TypeScript API is the data layer.
- **Database (SQL modes 3-6 or on-prem):** PostgreSQL (Neon for cloud; managed PostgreSQL for on-prem). ORM: Drizzle (lean) or Prisma (rich tooling) — pick one per project, never mix.
- **Auth (Mode 1):** Clerk + Convex ctx.auth.getUserIdentity(). Not Supabase Auth. Not NextAuth.
- **Auth (self-hosted / enterprise):** Auth.js (self-hosted Next.js) or Keycloak (enterprise SSO/SAML).
- **File storage (Mode 1):** Convex file storage. Not S3. Not Supabase Storage. Not R2.
- **File storage (Mode 2):** Cloudflare R2.
- **File storage (on-prem):** MinIO.
- **Email:** Resend. Not SendGrid. Not Mailgun. Not Nodemailer.
- **Payments:** Stripe (international) / Razorpay (India).
- **Monitoring:** Sentry + platform logs (Axiom/Grafana when budget allows).
- **Testing:** Vitest (unit/integration), Playwright (E2E), React Testing Library (components). Not Jest. Not Cypress.
- **AI:** backend-routed only — Convex actions (Mode 1) or backend API routes (other modes). Never from the frontend.

## Architecture templates

**Template A — Low-Cost Website.** Company sites, portfolios, landing pages, blogs. Next.js (SSG) + Tailwind + Cloudflare Pages (per Architecture Overview section 01). Forms via Resend/Formspree or a small Cloudflare Pages Function/Worker. Optional CMS: Contentlayer/MDX, Decap, Sanity, or headless WordPress. Next.js (SSG) is the default static stack; it grows into Template B (SSR + login) without a framework change.

**Template B — Standard Web App / SaaS.** Dashboards, portals, booking systems, SaaS MVPs, CRMs. Next.js + TypeScript + Tailwind + shadcn/ui + Convex (DB + file storage + scheduled jobs + vector search) + Clerk (auth) + Resend + Stripe/Razorpay + Sentry. Deploy on Vercel when DX matters, Cloudflare when cost/edge matters. No ORM — Convex TypeScript API is the data layer. No Supabase. No NextAuth.

**Template C — Cloudflare-First Full-Stack.** Cheapest production hosting for edge/serverless-fit workloads. React Router/TanStack Start/Next-on-CF + Hono + Cloudflare Workers + D1 (or Neon/Postgres) + R2 + KV + Queues + Durable Objects. Auth: Clerk or Auth.js. Best for low-cost apps, APIs, chatbots, file-heavy/compute-light, read-heavy multi-region. Avoid when long CPU-heavy jobs, many Node-only packages, or traditional server deployment are required.

**Template D — Mobile App.** Expo React Native + TypeScript + NativeWind/Tamagui + a shared backend from Template B (Convex) or C + Expo Notifications/FCM for push. Do not build separate native apps unless deep native features are explicitly needed.

**Template E — AI-Enabled App.** Layered on an existing web/mobile stack. Convex actions are the AI routing layer for Mode 1; FastAPI for Mode 4 Python AI systems. Convex vector search (Mode 1) or pgvector on Neon (Mode 4) for retrieval. Background jobs via Convex scheduled functions (Mode 1) or Celery (Mode 4). Rules: never call AI from the frontend; always route through backend; log model/prompt-version/cost/latency/user; per-client and per-user rate limits; monthly spend cap.

**Template F — On-Premises / Private Server.** Enterprises, government, sensitive/offline data. Docker Compose (Kubernetes only for large deployments) + Next.js frontend + NestJS or FastAPI backend + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak for SSO + Prometheus/Grafana. HARD RULES: no Convex (cloud-only), no Supabase (hosted-only), no Vercel, no Cloudflare Pages, no Clerk hosted. Default to Docker Compose first; move to Kubernetes only with real scale.

## Backend modes

1. **Convex backend (default for cloud SaaS)** — SaaS MVPs, dashboards, portals, small/medium apps. Convex provides the DB, file storage, real-time subscriptions, scheduled jobs, vector search, and serverless function runtime. Clerk provides auth. No ORM. No separate API server. Fastest path to a production-ready cloud app.
2. **Hono on Cloudflare Workers** — cheap APIs, chatbots, edge workloads, high-read traffic. Pairs with R2/KV/Queues/D1 or Neon. Auth: Clerk or Auth.js.
3. **NestJS** — enterprise, on-prem, complex integrations, long-running jobs, multiple frontends; TypeScript end-to-end. PostgreSQL + Drizzle or Prisma.
4. **FastAPI (Python)** — AI-heavy systems, data workflows, document processing, Python-library integrations. PostgreSQL + pgvector for retrieval.
5. **Spring Boot / .NET** — enterprise/government, large internal systems, existing Java/.NET teams.
6. **Laravel** — budget CRUD systems, admin panels, content-driven apps, PHP teams, traditional hosting.

## Hosting cost tiers

| Tier | Use case | Stack |
| --- | --- | --- |
| Free / ultra-low | Static sites, demos | Cloudflare Pages/Workers, Vercel Hobby where allowed |
| Low-cost production | Small apps, MVPs | Cloudflare Workers + Convex free tier or Neon free tier |
| Standard SaaS | Serious production | Vercel/Cloudflare + Convex paid or Neon paid |
| Enterprise cloud | Larger clients | AWS/GCP/Azure or managed containers |
| On-premises | Private/government/regulated | Docker Compose / Kubernetes |

Cost notes (verify before quoting; these change): Cloudflare Workers Paid from ~$5/mo min. Vercel free Hobby (non-commercial), Pro ~$20/mo + usage. Convex free tier available; paid plans start at $25/mo. Neon $0 Free with limited storage/compute.
