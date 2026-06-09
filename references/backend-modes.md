# Nexara Backend Modes (reference)

Build to the mode the architect approved. Each mode below lists when to use it and its shape.

## Mode 1 — Next.js backend
- **Use for:** SaaS MVPs, dashboards, portals, small/medium business apps. Fastest path.
- **Shape:** server actions + route handlers in the Next.js app; PostgreSQL via Drizzle/Prisma; auth via Supabase Auth/Auth.js/NextAuth. Co-located with the frontend (Template B).

## Mode 2 — Hono on Cloudflare Workers
- **Use for:** cheap APIs, chatbots, edge workloads, high-read traffic (Template C).
- **Shape:** Hono routers on Workers; D1 or Neon/Supabase Postgres; R2 for files; KV for cache; Queues for async; Durable Objects for realtime/stateful coordination. Avoid for long CPU-heavy jobs or many Node-only packages.

## Mode 3 — NestJS (dedicated TypeScript backend)
- **Use for:** enterprise, on-prem, complex integrations, long-running jobs, multiple frontends.
- **Shape:** modular Nest architecture (modules, providers, DI), DTO validation, guards for RBAC, separate API service. TypeScript end-to-end.

## Mode 4 — FastAPI (Python)
- **Use for:** AI-heavy systems, data workflows, document processing, Python-library integrations (Template E AI layer).
- **Shape:** FastAPI app with pydantic validation; pytest + pytest-asyncio for tests; PostgreSQL + pgvector for retrieval; background workers for long AI work; provider-abstraction layer for AI calls.

## Mode 5 — Spring Boot / .NET
- **Use for:** enterprise/government, large internal systems, existing Java/.NET teams.
- **Shape:** layered enterprise architecture; Spring Security (Keycloak roles mapped in) or ASP.NET Identity; PostgreSQL/RDS; containerized; strong observability.

## Mode 6 — Laravel
- **Use for:** budget CRUD systems, admin panels, content-driven apps, PHP teams, traditional hosting.
- **Shape:** Eloquent models, policies for permissions, queues for jobs; PostgreSQL/MySQL; VPS/shared/cloud hosting.

## Cross-mode reminders
- The standard rules in SKILL.md (tenancy, security baseline, AI routing, file storage) apply to every mode.
- Match the deployment model the architect approved (e.g. Mode 3/4 under Docker Compose for on-prem Template F; Mode 2 on Workers for Template C).
- Reuse the boilerplate modules where they exist: auth, RBAC, audit logs, file upload, payment webhooks, AI gateway, notifications.
