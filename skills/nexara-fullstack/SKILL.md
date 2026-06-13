---
name: nexara-fullstack
description: Designs and implements both the UI/UX and backend for a Nexara project in one pass. Enforces the full pipeline - runs nexara-stack-selection to pick the stack, then nexara-architect-review to approve it, then implements UI and backend together. Default backend is Convex with Clerk for auth. Use this skill for full-stack features, end-to-end flows, or complete project build-outs.
---

# Nexara Fullstack (UI/UX + Backend)

This skill runs the full Nexara delivery pipeline: stack selection -> architect review -> UI/UX + backend. The default backend is Convex (reactive DB + serverless queries/mutations/actions) with Clerk for auth. Do not skip the gatekeeping steps.

---

## MANDATORY PIPELINE - do not skip steps

If the user already ran nexara-stack-selection and nexara-architect-review in this session, read their output and proceed to Step 3. Otherwise run Steps 1 and 2 now.

### Step 1 - Stack Selection

Apply the nexara-stack-selection decision procedure:

1. Ask only for missing facts that change the branch (on-prem? AI core? scale? budget? login?).
2. Walk the decision tree in order, stop at the first match:
   - Mobile? -> Template D + Convex + Clerk.
   - Content site, no login? -> Template A (Next.js SSG or WordPress/Webflow).
   - Ecommerce? -> Shopify/WooCommerce or custom marketplace (B + Convex).
   - On-premises? -> Template F + Mode 3 or 4 + PostgreSQL. No Convex.
   - AI is core? -> Template E on B/C + Convex actions + Convex vector search.
   - Lowest cost? -> Template C + Mode 2. Standard SaaS? -> Template B + Mode 1 (Convex). Enterprise? -> B + Mode 3/5/6.
3. Output:

```
## Stack recommendation: [project]
Template: [A-F] - [name]
Backend mode: [1-6]
Stack: Frontend / Backend-DB / Auth / Hosting / Other
Cost target: $X-$Y/month
Why: [2-3 sentences]
Watch-outs: [when to revisit]
```

### Step 2 - Architect Review

Review the Step 1 recommendation. Issue a verdict:

- Stack fit: does the template match the decision tree?
- On-prem + Convex? Automatic blocker - Convex is cloud-only.
- Deployment: model explicit and consistent with hosting?
- Auth: Clerk + Convex for Mode 1; NextAuth/Keycloak for self-hosted/enterprise. RBAC at backend layer?
- Security: secrets in env; every Convex function checks ctx.auth.getUserIdentity(); input validation; HTTPS; rate limits; audit logs; Sentry.
- Data/tenancy: Convex default for cloud; PostgreSQL for on-prem/SQL-heavy. If multi-tenant: orgId on every Convex table, all queries filtered by orgId.
- AI: all AI calls through Convex actions only - frontend-to-provider is automatic blocker.
- Cost: matches budget target?
- Contradictions: deployment vs hosting, auth vs deployment, Convex in on-prem proposal.

Output:

```
## Architect review: [project]
Verdict: APPROVE | APPROVE WITH CONDITIONS | REJECT
Summary: [2-3 sentences]
Blockers: [issue -> fix]
Conditions: [issue -> fix during build]
Strengths: [aligned items]
```

Do not proceed to Step 3 if verdict is REJECT.

---

### Step 3 - UI/UX Implementation

Only after Step 2 returns APPROVE or APPROVE WITH CONDITIONS.

1. **Generate design direction with ui-ux-pro-max** (if installed). Let it propose design system, palette, typography.
2. **Apply Nexara guardrails** from `references/nexara-design-defaults.md`:
   - Component foundation: Tailwind CSS + shadcn/ui for web; NativeWind/Tamagui for Expo mobile.
   - Accessibility: contrast >= 4.5:1, visible focus states, aria-label on icon buttons, full keyboard support.
   - Tokens only - no hardcoded hex or pixel values.
   - Dark mode where brand allows.
   - No AI calls from the frontend.
   - Responsive by default for web; Apple HIG / Material for mobile.
3. **Build to the approved stack.** For Convex: use `useQuery` for reactive reads, `useMutation` for writes, no direct DB access from components.
4. **Produce the handoff contract:** for each component, the data shape it reads (Convex query) and the mutations/actions it triggers.

### Step 4 - Backend Implementation (Convex default - Mode 1)

Build to the approved mode from `references/backend-modes.md`.

**For Mode 1 (Convex) - apply these rules:**
- **Schema:** define all tables in `convex/schema.ts` with `defineTable`. Add indexes for every query pattern. Add `orgId`/`tenantId` to every tenant-owned table.
- **Queries:** read-only, reactive. Always filter by `ctx.auth.getUserIdentity()` and `orgId`. Return only what the UI needs.
- **Mutations:** all writes. Validate inputs. Check auth and org membership before mutating.
- **Actions:** side-effects only (AI calls, email, external APIs). Never access DB directly in actions - call internal mutations/queries instead.
- **Auth:** Clerk provides identity. Every Convex function validates `ctx.auth.getUserIdentity()`. Implement roles in the Convex DB (user -> org -> role -> permissions).
- **File storage:** use Convex file storage. Generate upload URLs server-side. Store file IDs in DB, generate download URLs on demand.
- **Scheduled functions:** use `crons.ts` for recurring jobs; `ctx.scheduler.runAfter` for delayed jobs.
- **AI routing (hard rule):** all AI calls inside Convex actions only. Log model, prompt version, cost estimate, latency, userId, orgId. Apply per-org and per-user rate limits. Set monthly spend cap.

**For Modes 2-6:** apply standard rules from `references/backend-modes.md`. PostgreSQL with Drizzle/Prisma for Modes 3-6. tenant_id on every table, all queries tenant-scoped.

### Step 5 - Hand off to testing

List for nexara-testing:
- All screens and flows, naming critical paths (login, signup, core journey, checkout) for Playwright.
- All Convex queries/mutations/actions (or REST endpoints for other modes) with input/output shapes and auth requirements.
- Components with complex states for component-level tests.

---

## Output

Step 1 recommendation + Step 2 verdict + then:

**UI layer:** component/page code + design-decision note + list of flows.

**Backend layer (Convex):** schema definition + queries/mutations/actions + cron jobs + auth rules.

**Backend layer (other modes):** code in approved framework + endpoint list + migrations.

---

See `references/nexara-design-defaults.md` for UI guardrails.
See `references/backend-modes.md` for all backend mode details.
