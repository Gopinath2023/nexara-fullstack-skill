# Templates, Backend Modes & Costs (reference)

Load this to fill in the "Recommended stack" lines of a recommendation once you have picked a template via the SKILL.md flowchart.

## Common foundation (applies across templates)

- **Language:** TypeScript for web/mobile/most APIs; Python where AI/data is central.
- **UI:** Tailwind CSS + shadcn/ui, or a client-approved design system.
- **Database:** Convex (default for reactive/real-time apps with TypeScript backend logic); PostgreSQL via Neon for SQL-heavy or reporting workloads. On-prem: PostgreSQL + Docker.
- **Backend logic:** Convex queries/mutations/actions (no ORM needed); Drizzle or Prisma when using PostgreSQL directly.
- **Auth:** Clerk (primary, integrates natively with Convex); Auth.js/NextAuth for self-hosted; Keycloak for enterprise SSO.
- **File storage:** Convex file storage (default); Cloudflare R2 for edge/Cloudflare-first stacks; MinIO on-prem.
- **Email:** Resend. **Payments:** Stripe (international) / Razorpay (India).
- **Monitoring:** Sentry + platform logs (Axiom/Grafana when budget allows).
- **AI:** backend-routed provider abstraction with model routing, prompt logging, rate limits, retrieval where needed. Convex vector search for small/medium retrieval workloads.

## Architecture templates

**Template A — Low-Cost Website.** Company sites, portfolios, landing pages, blogs. Next.js (SSG) + Tailwind + Cloudflare Pages. Forms via Resend/Formspree or a small Cloudflare Pages Function. Optional CMS: Contentlayer/MDX, Decap, Sanity, or headless WordPress. Next.js SSG is the default static stack; grows into Template B without a framework change.

**Template B — Standard Web App / SaaS.** Dashboards, portals, booking systems, SaaS MVPs, CRMs. Next.js + TypeScript + Tailwind + shadcn/ui + Convex (DB + backend logic + real-time) + Clerk (auth) + Convex file storage + Resend + Stripe/Razorpay + Sentry. Deploy on Vercel when DX matters, Cloudflare when cost/edge matters.

**Template C — Cloudflare-First Full-Stack.** Cheapest production hosting for edge/serverless-fit workloads. React Router/TanStack Start/Next-on-CF + Hono + Cloudflare Workers + D1 or Convex + R2 + KV + Queues + Durable Objects. Best for low-cost apps, APIs, chatbots, edge-heavy, read-heavy multi-region. Avoid for long CPU-heavy jobs or many Node-only packages.

**Template D — Mobile App.** Expo React Native + TypeScript + NativeWind/Tamagui + shared backend from Template B or C (Convex works natively with React Native via the Convex React SDK) + Expo Notifications/FCM for push.

**Template E — AI-Enabled App.** Layered on B or C. AI provider abstraction layer, prompt templates in code, conversation/request logs in Convex, Convex vector search for small/medium retrieval (pgvector on Neon for larger vector workloads), background jobs via Convex scheduled functions, human review for sensitive actions. Providers: OpenAI primary; Anthropic/Gemini secondary. Rules: never call AI from the frontend; always route through Convex actions; log model/prompt-version/cost/latency/user; per-client and per-user rate limits.

**Template F — On-Premises / Private Server.** Enterprises, government, sensitive/offline data. Convex is cloud-only — do NOT use for on-prem. Use: Docker Compose + Next.js + NestJS or FastAPI + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak for SSO + Prometheus/Grafana. Default to Docker Compose; Kubernetes only with real scale.

## Backend modes

1. **Convex backend (new default)** — SaaS MVPs, dashboards, portals, real-time apps, small/medium apps. Fastest path. Convex queries/mutations/actions co-located with Next.js frontend; Clerk for auth; Convex file storage; Convex scheduled functions for background jobs.
2. **Hono on Cloudflare Workers** — cheap APIs, chatbots, edge workloads, high-read traffic. Pairs with R2/KV/Queues/D1. Use when Cloudflare-first stack is required and Convex is not needed.
3. **NestJS** — enterprise, on-prem, complex integrations, long-running jobs, multiple frontends; TypeScript end-to-end. PostgreSQL + Drizzle/Prisma.
4. **FastAPI (Python)** — AI-heavy systems, data workflows, document processing, Python-library integrations. PostgreSQL + pgvector for large vector workloads.
5. **Spring Boot / .NET** — enterprise/government, large internal systems, existing Java/.NET teams.
6. **Laravel** — budget CRUD systems, admin panels, content-driven apps, PHP teams, traditional hosting.

## Hosting cost tiers

| Tier | Use case | Stack |
| --- | --- | --- |
| Free / ultra-low | Static sites, demos | Cloudflare Pages, Vercel Hobby where allowed |
| Low-cost production | Small apps, MVPs | Convex free tier + Vercel/Cloudflare |
| Standard SaaS | Serious production | Convex Pro + Vercel/Cloudflare + Clerk |
| Enterprise cloud | Larger clients | AWS/GCP/Azure or managed containers |
| On-premises | Private/government/regulated | Docker Compose / Kubernetes (no Convex) |

Cost notes (verify before quoting): Convex free tier generous for MVPs; Pro from ~$25/mo. Vercel Pro ~$20/mo + usage. Clerk free up to 10k MAU; Pro ~$25/mo. Cloudflare Workers Paid from ~$5/mo. Neon free tier available; paid from ~$19/mo.
