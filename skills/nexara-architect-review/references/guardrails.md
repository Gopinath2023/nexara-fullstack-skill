# Nexara Technology Guardrails

This is the authoritative approved/forbidden list for all Nexara projects. Any tool, library, or pattern not on an APPROVED list must not be recommended or generated without explicit written architect approval and documented client justification. When in doubt: do not add it.

---

## Mode 1 — Convex (default for Template B cloud SaaS)

| Layer | APPROVED — use these | FORBIDDEN — never substitute |
|-------|----------------------|------------------------------|
| Database | Convex DB (defineTable in convex/schema.ts) | Supabase DB, Firebase Firestore, MongoDB, PlanetScale, Turso, CockroachDB, SQLite (production), DynamoDB, Redis as primary DB, any SQL DB |
| ORM / query builder | None — Convex TypeScript API is the data layer | Drizzle, Prisma, TypeORM, Mongoose, Sequelize, MikroORM, Knex — no ORM in a Mode 1 project |
| Auth | Clerk (identity) + Convex ctx.auth.getUserIdentity() (enforcement) | Supabase Auth, Firebase Auth, Auth0, NextAuth.js / Auth.js, Lucia, Passport.js, custom JWT sessions, custom bcrypt password flows |
| Backend function style | Convex queries (reads), mutations (writes), actions (side effects / AI / external APIs) | REST endpoints in Next.js API routes, server actions calling DB directly, tRPC, GraphQL, Apollo, Hono routes on top of Convex |
| Client data reads | useQuery hook from convex/react | useEffect + fetch, React Query, TanStack Query, SWR, Apollo Client, RTK Query, axios to Convex endpoints |
| Client data writes | useMutation hook from convex/react | fetch/axios to Convex HTTP API, REST mutations, server actions calling DB |
| Server state management | Convex reactive state replaces client state libraries for server data | Zustand, Redux / Redux Toolkit, MobX, Jotai, Recoil, Valtio — none for server state |
| Local UI state | React useState, useReducer, useContext | Nothing forbidden — these are correct for local UI |
| Real-time / live updates | Convex reactive queries (useQuery auto-updates on data change) | Socket.io, Pusher, Ably, Liveblocks, Partykit, Server-Sent Events libraries, custom WebSocket servers |
| File storage | Convex file storage (generateUploadUrl + storageId pattern) | AWS S3 directly, Supabase Storage, Cloudflare R2 (Mode 1), Cloudinary, uploadthing, imgix |
| Background / scheduled jobs | convex/crons.ts for recurring; ctx.scheduler.runAfter for delayed | BullMQ, Inngest, Trigger.dev, Temporal, QStash, Cloudflare Queues, Redis queues, cron endpoints in Next.js |
| Vector / semantic search | Convex vector search (vectorIndexes in schema) | Pinecone, Weaviate, Qdrant, Chroma, Milvus, pgvector — redundant for Mode 1 |
| AI call origin | Convex actions ONLY | React components, Next.js API routes, server actions, edge middleware, any client-side code |
| AI logging | Log model, prompt version, cost estimate, latency, userId, orgId inside every action | Unlogged AI calls — automatic blocker |
| Email | Resend | SendGrid, Mailgun, Nodemailer, AWS SES directly, Postmark, Brevo (unless client explicitly requests) |
| CSS framework | Tailwind CSS | styled-components, Emotion, CSS Modules alongside Tailwind, Vanilla Extract, Stitches |
| UI component library | shadcn/ui (wraps Radix UI) | Material UI (MUI), Chakra UI, Ant Design, Mantine, Radix UI used directly without shadcn |
| Payments | Stripe (international), Razorpay (India) | PayPal, Braintree, LemonSqueezy, Square, Paddle (unless client explicitly requests) |
| Error monitoring | Sentry | Datadog, New Relic, Rollbar, Bugsnag (unless client requests) |
| Log aggregation | Axiom or Grafana (when budget allows) | Splunk, Datadog Logs, CloudWatch (unless on AWS by explicit choice) |
| Unit / integration tests | Vitest | Jest — always Vitest |
| E2E tests | Playwright | Cypress, WebdriverIO |
| Component tests | React Testing Library | Enzyme |
| Deployment target | Vercel (Next.js primary), Cloudflare Pages (edge) | Railway, Render, Fly.io, Heroku, DigitalOcean App Platform, Netlify (unless client requests) |
| Schema changes | Edit convex/schema.ts only | SQL migration files, Flyway, Liquibase — Convex has no SQL migrations |
| Package manager | npm (do not change unless client specifies) | Do not swap to pnpm or yarn without being asked |

---

## Mode 2 — Hono on Cloudflare Workers (Template C)

Differences from Mode 1 defaults:

| Layer | APPROVED | FORBIDDEN |
|-------|----------|-----------|
| Database | Cloudflare D1, Neon PostgreSQL + Drizzle, or KV for ephemeral data | Convex (not edge-compatible), Supabase DB, MongoDB |
| ORM | Drizzle (D1/Neon) | Prisma (too heavy for edge), TypeORM |
| Auth | Clerk or Auth.js | Supabase Auth |
| File storage | Cloudflare R2 | Supabase Storage, S3 directly |
| Background jobs | Cloudflare Queues + Durable Objects | BullMQ, Redis queues (no persistent Node process on Workers) |
| Caching | Cloudflare KV, Cache API | Redis standalone |

---

## Modes 3-6 — NestJS / FastAPI / Spring Boot / Laravel

Differences from Mode 1 defaults:

| Layer | APPROVED | FORBIDDEN |
|-------|----------|-----------|
| Database | Neon PostgreSQL or managed PostgreSQL | Convex (not for SQL modes), Firebase, MongoDB (unless document-model justified by architect) |
| ORM | Drizzle (lean) or Prisma (rich tooling) — pick one, be consistent | Mixing Drizzle and Prisma in one project; TypeORM with NestJS (use Prisma/Drizzle instead); Mongoose unless MongoDB |
| Auth | NextAuth.js (self-hosted Next.js), Keycloak (enterprise SSO), Clerk (if frontend is Next.js) | Supabase Auth, Firebase Auth |
| File storage | Cloudflare R2 (cloud), MinIO (on-prem) | Supabase Storage, S3 directly (prefer R2 unless already on AWS) |
| Background jobs | BullMQ + Redis (NestJS), Celery + Redis (FastAPI) | Inngest, Trigger.dev, Temporal (unless complex workflow approved by architect) |
| Vector search | pgvector on Neon (Mode 4 AI systems) | Pinecone, Weaviate, Qdrant, Chroma (only if scale justifies external vector DB — requires architect approval) |

---

## Template F — On-premises (hard rules, zero exceptions)

| Rule | Detail |
|------|--------|
| NO CONVEX | Convex is cloud-only. Any on-prem proposal using Convex is an automatic architect blocker. |
| NO SUPABASE | Supabase is a hosted service — cannot be used in private/on-prem deployments. |
| NO VERCEL or CLOUDFLARE PAGES | All compute must run on client-provided servers. |
| NO CLERK (hosted) | Use Keycloak (self-hosted) for SSO/SAML/LDAP. |
| Required stack | Docker Compose + Next.js frontend + NestJS or FastAPI backend + PostgreSQL + Redis + MinIO + Caddy or Nginx + Keycloak (if SSO needed) + Prometheus/Grafana |

---

## Universal rules — apply to every template and every mode

Violations are automatic architect blockers. They must not appear in generated code.

1. **AI never from the frontend.** Every AI call must go through the backend. A React component that imports an AI SDK or calls a provider endpoint directly is a blocker.

2. **Auth check in every backend function.** Every Convex query/mutation/action, every API route, every server action must validate the authenticated user before reading or writing data.

3. **Tenant scope on every query in multi-tenant systems.** Every read and write against a tenant-owned table must filter by orgId (Convex) or tenant_id (SQL modes). Unscoped data access is a blocker.

4. **No plaintext secrets in source code.** All API keys, DB connection strings, signing secrets, and tokens belong in environment variables or a secrets manager. Never commit them.

5. **No custom auth from scratch.** Do not implement JWT signing, session token generation, password hashing, or OAuth flows from scratch. Use Clerk (Mode 1), Auth.js (self-hosted web), or Keycloak (enterprise).

6. **Convex schema is TypeScript — no SQL migrations.** convex/schema.ts is the schema. Do not create migration files or schema-version tables for Mode 1.

7. **No ctx.db in Convex actions.** Actions call internal queries or mutations to read/write data. Direct ctx.db access inside an action is wrong.

8. **No REST layer on top of Convex.** Convex functions are the API. Do not create Next.js API routes or any REST/GraphQL layer between the frontend and Convex.

9. **One ORM, consistently applied.** For SQL modes: choose Drizzle or Prisma at project start and never mix both in the same project.

10. **Resend is the email provider for all new projects.** Do not default to SendGrid, Mailgun, or Nodemailer without an explicit client or architect override.

11. **shadcn/ui is the component system for all web frontends.** Do not add MUI, Chakra, Ant Design, or any other full component library alongside it.

12. **Vitest not Jest.** Do not generate Jest config, Jest imports, or jest.config files. Use Vitest throughout.

13. **Do not add state management libraries for server data.** useQuery is the data layer. Zustand, Redux, MobX, Jotai, Recoil — none for server data. These are only acceptable for complex local UI state that has no server component.

14. **Payments via tested provider only.** Never implement custom payment flows or direct card tokenization. Use Stripe or Razorpay.
