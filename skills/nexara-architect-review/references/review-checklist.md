# Architect Review Checklist (reference)

Work each area. Mark pass / concern / blocker. Sources: Nexara Standard Tech Stack Playbook and Nexara Technical Architecture Overview.

## 1. Stack & template fit
- Does the chosen template (A-F) match what the decision flowchart would pick for this project? If it diverges, is there a stated reason (budget, team skill, client ecosystem)?
- Is the backend mode (1-6) appropriate for the workload (CRUD vs AI vs edge vs enterprise)?
- Language fit: TypeScript for web/mobile/most APIs; Python where AI/data is central.

## 2. Deployment & update strategy
- Is the deployment model explicit (desktop/Electron, self-hosted server, static, dynamic serverless, enterprise EC2, on-prem)?
- Does the update mechanism match the model? Desktop → electron-updater + force-update flag for security patches; self-hosted → Docker image pull / git + PM2 restart; static → CI/CD auto-build + instant rollback; dynamic → canary/blue-green; enterprise → blue/green EC2 swap + RDS snapshot before migrations.
- Is rollback defined?

## 3. CI/CD
- Pipeline present: code push → lint & test → build → staging/preview → QA gate → production.
- Is there a QA gate before production? (Required for production+ tiers.)

## 4. Authentication per variant
- Desktop → local credentials + bcrypt, JWT in Electron safeStorage.
- Self-hosted → NextAuth.js credentials, HTTP-only cookie/JWT, API middleware.
- Dynamic web → OAuth/magic link, NextAuth.js or Supabase Auth, JWT in HTTP-only cookie.
- Enterprise → Keycloak or AWS Cognito, OAuth2/OIDC, SSO/SAML/LDAP/MFA.
- Is RBAC enforced at the API layer (not just the UI)?

## 5. Security baseline (blocker if missing on production+)
- Secrets in env/secrets manager, never in code; rotation plan for JWT keys, API keys, DB passwords.
- Input validation, auth middleware, role/permission checks, tenant scoping.
- HTTPS everywhere; HTTP-only cookies for web sessions; rate limits on auth, forms, and AI endpoints.
- MFA enforced for admin roles.
- Audit logs for sensitive actions; production DB backups; error monitoring (Sentry); dependency scanning.
- Secure file upload rules; signed URLs for private files; virus scanning for enterprise/sensitive.

## 6. Data & multi-tenancy
- PostgreSQL default unless document-model justifies otherwise.
- If multi-tenant: `tenant_id` on every tenant-owned table, all queries tenant-scoped, indexes on `tenant_id`, per-tenant storage prefixes/buckets.
- Tenancy upgrade path appropriate to stage (shared schema → RLS → schema/db per large client → dedicated for regulated).
- Connection pooling and backup strategy defined for the DB host.

## 7. Testing plan (must match project tier)
- MVP: unit tests for business logic + one Playwright smoke test (login + core journey).
- Production: ~60% backend line coverage, Playwright for all critical paths, component tests for shared UI.
- Enterprise/government: ~80% coverage, full Playwright suite, axe-core accessibility audit in CI, dependency vuln scan per PR.
- CI rules: Vitest on every PR (block on failure); Playwright on merge to main.

## 8. AI handling (hard rules — frontend AI call is an automatic blocker)
- All AI calls routed through the backend; never frontend-to-provider.
- Logging of model, prompt version, cost estimate, latency, user/client ID.
- Per-client and per-user rate limits; hard monthly spend cap with 80% alert.
- For government/healthcare/finance/legal/enterprise: confirm data may leave the client environment before using cloud AI.
- AI cost estimated and shared with client before build (separate from hosting).

## 9. Cost sanity
- Does the proposed infrastructure match the stated monthly budget target for the template?
- Are "+ AI usage" costs estimated separately, not folded into hosting?

## 10. Cross-document conflicts (require an explicit, consistent decision)
- ORM: Drizzle (playbook) vs Prisma (overview).
- Auth: Supabase Auth/Auth.js (playbook) vs NextAuth.js (overview).
- Test runner: Vitest (playbook) vs Jest/Vitest (overview).
- Hosting bias: Cloudflare-first (playbook) vs deployment-model-specific incl. AWS EC2 (overview).
Flag any place the proposal mixes these inconsistently within one app.
