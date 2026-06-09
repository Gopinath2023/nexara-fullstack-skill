# Templates, Backend Modes & Costs (reference)

Load this to fill in the "Recommended stack" lines of a recommendation once you've picked a template via the SKILL.md flowchart.

## Common foundation (applies across templates)

- **Language:** TypeScript for web/mobile/most APIs; Python where AI/data is central.
- **UI:** Tailwind CSS + shadcn/ui, or a client-approved design system.
- **Database:** PostgreSQL — Supabase for speed, Neon for database-only hosting.
- **ORM:** Drizzle for simple type-safe SQL; Prisma when richer tooling is needed.
- **Auth:** Supabase Auth, Clerk, Auth.js, or Keycloak by client type.
- **File storage:** Cloudflare R2 or Supabase Storage (MinIO on-prem).
- **Email:** Resend. **Payments:** Stripe (international) / Razorpay (India).
- **Monitoring:** Sentry + platform logs (Axiom/Grafana when budget allows).
- **AI:** backend-routed provider abstraction with model routing, prompt logging, rate limits, retrieval where needed.

## Architecture templates

**Template A — Low-Cost Website.** Company sites, portfolios, landing pages, blogs. Astro or Next.js + Tailwind + Cloudflare Pages/Workers. Forms via Resend/Formspree/Supabase/a small Worker. Optional CMS: Decap, Sanity, or headless WordPress. Use Astro when mostly content; Next.js when login/app features may come later.

**Template B — Standard Web App / SaaS.** Dashboards, portals, booking systems, SaaS MVPs, CRMs. Next.js + TypeScript + Tailwind + shadcn/ui + PostgreSQL + Drizzle + Supabase Auth/Auth.js + R2/Supabase Storage + Resend + Stripe/Razorpay + Sentry. Deploy on Vercel when DX matters, Cloudflare when cost/edge matters.

**Template C — Cloudflare-First Full-Stack.** Cheapest production hosting for edge/serverless-fit workloads. React Router/TanStack Start/Next-on-CF + Hono + Cloudflare Workers + D1 (or Neon/Supabase Postgres) + R2 + KV + Queues + Durable Objects. Best for low-cost apps, APIs, chatbots, file-heavy/compute-light, read-heavy multi-region. Avoid when long CPU-heavy jobs, many Node-only packages, or traditional server deployment are required.

**Template D — Mobile App.** Expo React Native + TypeScript + NativeWind/Tamagui + a shared backend from Template B or C + Expo Notifications/FCM for push. Don't build separate native apps unless deep native features are explicitly needed.

**Template E — AI-Enabled App.** Layered on an existing web/mobile stack. AI provider abstraction layer, prompt templates in code, conversation/request logs in PostgreSQL, vector search only when retrieval is needed, background jobs for long work, human review for sensitive actions. Providers: OpenAI primary; Anthropic/Gemini secondary; Cloudflare AI Gateway or custom gateway for tracking/switching; pgvector for small/medium retrieval. Rules: never call AI from the frontend; always route through backend; log model/prompt-version/cost/latency/user; per-client and per-user rate limits.

**Template F — On-Premises / Private Server.** Enterprises, government, sensitive/offline data. Docker Compose (Kubernetes only for large deployments) + Next.js frontend + NestJS or FastAPI backend + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak for SSO + Prometheus/Grafana or simpler logs. Default to Docker Compose first; move to Kubernetes only with real scale and infra maturity.

## Backend modes

1. **Next.js backend** — SaaS MVPs, dashboards, portals, small/medium apps. Fastest path.
2. **Hono on Cloudflare Workers** — cheap APIs, chatbots, edge workloads, high-read traffic. Pairs with R2/KV/Queues/D1.
3. **NestJS** — enterprise, on-prem, complex integrations, long-running jobs, multiple frontends; TypeScript end-to-end.
4. **FastAPI (Python)** — AI-heavy systems, data workflows, document processing, Python-library integrations.
5. **Spring Boot / .NET** — enterprise/government, large internal systems, existing Java/.NET teams.
6. **Laravel** — budget CRUD systems, admin panels, content-driven apps, PHP teams, traditional hosting.

## Hosting cost tiers

| Tier | Use case | Stack |
| --- | --- | --- |
| Free / ultra-low | Static sites, demos | Cloudflare Pages/Workers, Vercel Hobby where allowed |
| Low-cost production | Small apps, MVPs | Cloudflare Workers + Supabase/Neon |
| Standard SaaS | Serious production | Vercel/Cloudflare + Supabase Pro/Neon paid |
| Enterprise cloud | Larger clients | AWS/GCP/Azure or managed containers |
| On-premises | Private/government/regulated | Docker Compose / Kubernetes |

Cost notes (verify before quoting; these change): Cloudflare Workers Paid from ~$5/mo min (Workers, Pages Functions, KV, Hyperdrive, Durable Objects; no Workers egress charges). Vercel free Hobby (non-commercial), Pro ~$20/mo + usage. Supabase Pro ~$25/mo. Neon $0 Free with limited storage/compute.
