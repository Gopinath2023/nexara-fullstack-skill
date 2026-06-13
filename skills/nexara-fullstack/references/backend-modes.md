# Nexara Backend Modes (reference)

Build to the mode the architect approved. Each mode lists when to use it and its shape.

## Mode 1 — Convex backend (default)
- **Use for:** SaaS MVPs, dashboards, portals, real-time apps, small/medium apps. Fastest path.
- **Shape:** Convex queries (read), mutations (write), and actions (side-effects/AI calls) co-located with a Next.js frontend. Clerk for auth. Convex file storage for uploads. Convex scheduled functions for background jobs. No ORM or separate DB server needed.
- **Key Convex patterns:**
  - Define schema in `convex/schema.ts` with `defineTable` + indexes for all query patterns.
  - Queries are reactive — frontend subscribes via `useQuery`, updates automatically.
  - Mutations are transactional — use for all writes.
  - Actions are for side-effects (calling AI, sending email, calling external APIs) — not transactional.
  - Multi-tenancy: add `orgId` / `tenantId` field on every tenant-owned table; index on it; scope all queries by `orgId`.
  - Auth: Convex integrates with Clerk via `ctx.auth.getUserIdentity()` in every function.

## Mode 2 — Hono on Cloudflare Workers
- **Use for:** cheap APIs, chatbots, edge workloads, high-read traffic (Template C).
- **Shape:** Hono routers on Workers; D1 or Neon Postgres; R2 for files; KV for cache; Queues for async; Durable Objects for realtime/stateful coordination. Use when Cloudflare-first stack is required and Convex is not needed.

## Mode 3 — NestJS (dedicated TypeScript backend)
- **Use for:** enterprise, on-prem, complex integrations, long-running jobs, multiple frontends.
- **Shape:** modular Nest architecture (modules, providers, DI), DTO validation, guards for RBAC, separate API service. PostgreSQL + Drizzle/Prisma. TypeScript end-to-end.

## Mode 4 — FastAPI (Python)
- **Use for:** AI-heavy systems, data workflows, document processing, Python-library integrations (Template E AI layer).
- **Shape:** FastAPI app with pydantic validation; pytest + pytest-asyncio for tests; PostgreSQL + pgvector for large vector workloads; background workers for long AI work; provider-abstraction layer for AI calls.

## Mode 5 — Spring Boot / .NET
- **Use for:** enterprise/government, large internal systems, existing Java/.NET teams.
- **Shape:** layered enterprise architecture; Spring Security (Keycloak roles mapped in) or ASP.NET Identity; PostgreSQL/RDS; containerized; strong observability.

## Mode 6 — Laravel
- **Use for:** budget CRUD systems, admin panels, content-driven apps, PHP teams, traditional hosting.
- **Shape:** Eloquent models, policies for permissions, queues for jobs; PostgreSQL/MySQL; VPS/shared/cloud hosting.

## Cross-mode reminders
- Convex is cloud-only. For on-prem (Template F) always use Mode 3 or 4 with PostgreSQL.
- All AI calls go through Convex actions (Mode 1) or backend routes (Modes 2-6) — never from the frontend.
- Reuse boilerplate modules where they exist: auth, RBAC, audit logs, file upload, payment webhooks, AI gateway, notifications.
- For Convex: Clerk handles identity; Convex handles data access control. Never expose a Convex function without auth checks via `ctx.auth.getUserIdentity()`.
