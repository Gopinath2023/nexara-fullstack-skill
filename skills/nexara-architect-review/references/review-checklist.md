# Architect Review Checklist (reference)

Work each area. Mark pass / concern / blocker.

## 1. Stack & template fit
- Does the chosen template (A-F) match the decision flowchart for this project?
- Is the backend mode (1-6) appropriate? Mode 1 (Convex) is the default for new SaaS/portal/real-time apps.
- Language fit: TypeScript for web/mobile/most APIs; Python where AI/data is central.
- Convex chosen for on-prem? That is a blocker — Convex is cloud-only. Use Mode 3/4 + PostgreSQL for Template F.

## 2. Deployment & update strategy
- Is the deployment model explicit (static, dynamic serverless, enterprise cloud, on-prem)?
- Does the update mechanism match the model?
- Is rollback defined?

## 3. CI/CD
- Pipeline present: code push -> lint & test -> build -> staging/preview -> QA gate -> production.
- QA gate before production required for production+ tiers.

## 4. Authentication per variant
- Convex stack (Mode 1): Clerk as identity provider + Convex auth integration (`ctx.auth.getUserIdentity()`).
- Self-hosted / on-prem: NextAuth.js credentials or Keycloak, HTTP-only cookie/JWT, API middleware.
- Dynamic web (non-Convex): Clerk, NextAuth.js, or Auth.js. JWT in HTTP-only cookie.
- Enterprise: Keycloak or AWS Cognito, OAuth2/OIDC, SSO/SAML/LDAP/MFA.
- Is RBAC enforced at the backend layer (Convex functions or API routes), not just the UI?

## 5. Security baseline (blocker if missing on production+)
- Secrets in env/secrets manager, never in code.
- Convex functions: every function checks `ctx.auth.getUserIdentity()` before accessing data.
- Input validation in all mutations/actions/routes.
- HTTPS everywhere; HTTP-only cookies for non-Convex web sessions; rate limits on auth, forms, and AI endpoints.
- MFA enforced for admin roles.
- Audit logs for sensitive actions; error monitoring (Sentry); dependency scanning.
- Secure file upload; signed URLs for private Convex files; virus scanning for enterprise/sensitive.

## 6. Data & multi-tenancy
- Convex is the default DB for Modes 1. PostgreSQL (Neon) for SQL-heavy workloads or Mode 3/4/5/6.
- On-prem (Template F): PostgreSQL only — no Convex.
- If multi-tenant with Convex: `orgId` / `tenantId` field on every tenant-owned table; every query filtered by `orgId`; indexes on `orgId`; per-tenant file storage prefixes.
- If multi-tenant with PostgreSQL: `tenant_id` on every tenant-owned table; all queries tenant-scoped; indexes on `tenant_id`.
- Backup strategy: Convex handles backups automatically (verify on paid plan); PostgreSQL needs explicit backup strategy.

## 7. Testing plan (must match project tier)
- MVP: unit tests for Convex functions + one Playwright smoke test (login + core journey).
- Production: ~60% backend coverage, Playwright for all critical paths, component tests for shared UI.
- Enterprise/government: ~80% coverage, full Playwright suite, axe-core accessibility audit in CI, dependency vuln scan per PR.
- CI rules: Vitest on every PR (block on failure); Playwright on merge to main.

## 8. AI handling (hard rules — frontend AI call is an automatic blocker)
- All AI calls routed through Convex actions (Mode 1) or backend routes (Modes 2-6). Never frontend-to-provider.
- Logging of model, prompt version, cost estimate, latency, user/client ID.
- Per-client and per-user rate limits; hard monthly spend cap with 80% alert.
- Vector search: Convex built-in vector search for small/medium workloads; pgvector on Neon for large-scale retrieval.
- For regulated industries: confirm data may leave the client environment before using cloud AI or Convex.

## 9. Cost sanity
- Does the infrastructure match the stated budget target?
- Convex pricing: free tier for MVPs; Pro ~$25/mo. Clerk free to 10k MAU. Vercel Pro ~$20/mo.
- AI usage costs estimated separately, not folded into hosting.

## 10. Consistency checks (flag any mixing within one app)
- DB choice: Convex vs PostgreSQL — pick one per service; do not mix in the same data layer.
- Auth: Clerk + Convex vs NextAuth.js/Auth.js — must be consistent across the app.
- ORM (when using PostgreSQL): Drizzle vs Prisma — one per app.
- Hosting: Cloudflare-first vs Vercel — consistent with the template chosen.
- Convex in on-prem proposal: automatic blocker — Convex cannot be self-hosted.
