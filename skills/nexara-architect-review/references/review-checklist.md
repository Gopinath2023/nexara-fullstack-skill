# Architect Review Checklist (reference)

Work each area. Mark pass / concern / blocker. Sources: Nexara Standard Tech Stack Playbook, Nexara Technical Architecture Overview, and references/guardrails.md (authoritative approved/forbidden list).

## 0. Technology compliance (check against guardrails.md first)

Before all other sections, scan the proposal for forbidden technologies:
- Any Supabase DB, Supabase Auth, or Supabase Storage in a Mode 1 (Convex) proposal -> blocker.
- Convex in a Template F (on-prem) proposal -> blocker.
- Supabase or Vercel or Cloudflare Pages in an on-prem proposal -> blocker.
- AI provider called from the frontend (React component, Next.js API route, server action) -> blocker.
- No auth check in a Convex function or API route -> blocker.
- ORM (Drizzle, Prisma, TypeORM) in a Mode 1 (Convex) project -> blocker.
- React Query, TanStack Query, SWR, or useEffect+fetch to read Convex data -> blocker (use useQuery from convex/react).
- NextAuth.js or Auth.js in a Mode 1 (Convex) project -> blocker (use Clerk).
- Socket.io, Pusher, or Ably in a Mode 1 project -> blocker (Convex reactive queries are real-time).
- BullMQ, Inngest, Trigger.dev in a Mode 1 project -> blocker (use Convex crons.ts + ctx.scheduler).
- Jest in a new project -> concern (standard is Vitest).
- MUI, Chakra UI, or Ant Design in a project using shadcn/ui -> blocker.

## 1. Stack and template fit

- Does the chosen template (A-F) match what the decision flowchart would pick for this project? If it diverges, is there a stated reason (budget, team skill, client ecosystem)?
- Is the backend mode (1-6) appropriate for the workload (CRUD vs AI vs edge vs enterprise)?
- Language fit: TypeScript for web/mobile/most APIs; Python where AI/data is central.
- Is Mode 1 (Convex) being used for an on-prem deployment? Automatic blocker.

## 2. Deployment and update strategy

- Is the deployment model explicit (desktop/Electron, self-hosted server, static, dynamic serverless, enterprise cloud, on-prem)?
- Does the update mechanism match the model? Desktop -> electron-updater + force-update flag for security patches; self-hosted -> Docker image pull / git + PM2 restart; static -> CI/CD auto-build + instant rollback; dynamic -> canary/blue-green; enterprise -> blue/green EC2 swap + DB snapshot before migrations.
- Is rollback defined?

## 3. CI/CD

- Pipeline present: code push -> lint and test -> build -> staging/preview -> QA gate -> production.
- Is there a QA gate before production? (Required for production+ tiers.)

## 4. Authentication per variant

- **Mode 1 (Convex cloud SaaS):** Clerk for identity + ctx.auth.getUserIdentity() in every Convex function. Not Supabase Auth. Not NextAuth.js. Not Auth0. Not custom JWT.
- **Self-hosted / Template C:** Auth.js (NextAuth.js) credentials, HTTP-only cookie/JWT, API middleware. Not Supabase Auth.
- **Enterprise / Template F:** Keycloak or equivalent, OAuth2/OIDC, SSO/SAML/LDAP/MFA. Not Clerk (hosted). Not Supabase Auth.
- **Desktop:** local credentials + bcrypt, JWT in Electron safeStorage.
- Is RBAC enforced at the backend layer (not just the UI)? For Mode 1: role/permission tables in Convex DB, checked in every query/mutation. For SQL modes: middleware or service-layer checks.

## 5. Security baseline (blocker if missing on production+)

- Secrets in env/secrets manager, never in code; rotation plan for JWT keys, API keys, DB passwords.
- Input validation using Convex validators (Mode 1) or Zod/class-validator (other modes); auth middleware; role/permission checks; tenant scoping.
- HTTPS everywhere; HTTP-only cookies for web sessions where applicable; rate limits on auth, forms, and AI endpoints.
- MFA enforced for admin roles.
- Audit logs for sensitive actions; production DB backups; error monitoring (Sentry); dependency scanning.
- Secure file upload: for Mode 1, use Convex generateUploadUrl + store storageId (never store raw file bytes in DB); for other modes, signed URLs for private files; virus scanning for enterprise/sensitive.

## 6. Data and multi-tenancy

- **Mode 1:** Convex DB is the default. orgId field on every tenant-owned table. All queries filtered by orgId. Indexes on orgId. No SQL. No ORM.
- **Modes 3-6 and on-prem:** PostgreSQL. tenant_id on every tenant-owned table. All queries scoped by tenant_id. Indexes on tenant_id. One ORM (Drizzle or Prisma) — never mix both.
- Tenancy upgrade path appropriate to stage (shared DB -> RLS -> schema/DB per large client -> dedicated for regulated).
- Connection pooling and backup strategy defined for the DB host.
- If Convex: no ctx.db access inside actions — actions call internal queries/mutations.

## 7. Testing plan (must match project tier)

- MVP: unit tests for business logic + one Playwright smoke test (login + core journey).
- Production: ~60% backend line coverage, Playwright for all critical paths, component tests for shared UI.
- Enterprise/government: ~80% coverage, full Playwright suite, axe-core accessibility audit in CI, dependency vuln scan per PR.
- CI rules: Vitest on every PR (block on failure); Playwright on merge to main. Not Jest. Not Cypress.

## 8. AI handling (hard rules — frontend AI call is an automatic blocker)

- All AI calls routed through the backend only. For Mode 1: Convex actions. For other modes: backend API routes. Never from the frontend.
- Logging of model, prompt version, cost estimate, latency, user/client ID inside every AI call.
- Per-client and per-user rate limits; hard monthly spend cap with 80% alert.
- For government/healthcare/finance/legal/enterprise: confirm data may leave the client environment before using cloud AI.
- AI cost estimated and shared with client before build (separate from hosting).

## 9. Cost sanity

- Does the proposed infrastructure match the stated monthly budget target for the template?
- Are "+ AI usage" costs estimated separately, not folded into hosting?

## 10. Nexara standard choices (flag any deviation without documented reason)

These are the resolved Nexara defaults. Any proposal that deviates without explicit justification requires a condition or blocker:

| Area | Nexara default | Common wrong alternative |
|------|----------------|--------------------------|
| Database (cloud, Mode 1) | Convex | Supabase DB, PostgreSQL, Firebase |
| Database (on-prem / SQL modes) | PostgreSQL on Neon or managed | MongoDB (unless doc-model), Firebase |
| ORM (Mode 1) | None | Drizzle, Prisma (wrong for Convex) |
| ORM (SQL modes) | Drizzle or Prisma — one per project | TypeORM, Knex raw, mixing Drizzle+Prisma |
| Auth (Mode 1) | Clerk | Supabase Auth, NextAuth.js, Auth0 |
| Auth (self-hosted) | Auth.js (NextAuth) | Supabase Auth |
| Auth (enterprise) | Keycloak | AWS Cognito (flag for consideration) |
| Client data reads | useQuery from convex/react | React Query, SWR, TanStack Query, fetch |
| State (server data) | Convex reactive state | Zustand, Redux, MobX, Jotai |
| Real-time | Convex reactive queries | Socket.io, Pusher, Ably |
| File storage (Mode 1) | Convex file storage | S3, Supabase Storage, R2 |
| Background jobs (Mode 1) | Convex crons.ts + ctx.scheduler | BullMQ, Inngest, Trigger.dev |
| Vector search (Mode 1) | Convex vector search | Pinecone, pgvector |
| Email | Resend | SendGrid, Mailgun |
| UI components | shadcn/ui + Tailwind | MUI, Chakra UI, Ant Design |
| Test runner | Vitest | Jest |
| E2E tests | Playwright | Cypress |
| Static site | Next.js (SSG) + Cloudflare Pages | Astro (not default) |
| Deployment | Vercel (Next.js) or Cloudflare Pages (edge) | Railway, Render, Fly.io |
