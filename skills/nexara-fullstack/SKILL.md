---
name: nexara-fullstack
description: Designs and implements both the UI/UX and backend for a Nexara project in one pass. Enforces the full pipeline - runs nexara-stack-selection to pick the stack, then nexara-architect-review to approve it, then implements UI and backend together. Default backend is Convex with Clerk for auth. Use this skill for full-stack features, end-to-end flows, or complete project build-outs.
---

# Nexara Fullstack (UI/UX + Backend)

This skill runs the full Nexara delivery pipeline: stack selection -> architect review -> UI/UX + backend. The default backend is Convex (reactive DB + serverless queries/mutations/actions) with Clerk for auth. Do not skip the gatekeeping steps.

---

## TECHNOLOGY LOCK — read before generating any code

The tables below are non-negotiable. Deviating requires explicit written architect approval with documented client justification. **When in doubt, do not add the tool.**

The full approved/forbidden reference is in `references/guardrails.md`. The critical rules for Mode 1 (Convex) are:

### NEVER use these in a Mode 1 (Convex) project

| What Claude might hallucinate | What to use instead |
|-------------------------------|---------------------|
| Supabase (DB, Auth, or Storage) | Convex DB + Convex file storage + Clerk |
| Firebase / Firestore | Convex DB |
| PostgreSQL / any SQL DB | Convex DB |
| Prisma or Drizzle ORM | None — Convex TypeScript API is the data layer |
| NextAuth.js / Auth.js | Clerk + ctx.auth.getUserIdentity() |
| Auth0, Lucia, Passport.js, custom JWT | Clerk |
| useEffect + fetch to read data | useQuery from convex/react |
| React Query / TanStack Query / SWR | useQuery from convex/react |
| Zustand / Redux / MobX / Jotai / Recoil (for server data) | Convex reactive state — do not add |
| Socket.io / Pusher / Ably / Liveblocks | Convex reactive queries (built-in real-time) |
| AWS S3 / Cloudinary / uploadthing | Convex file storage |
| BullMQ / Inngest / Trigger.dev / Temporal | Convex crons.ts + ctx.scheduler.runAfter |
| Pinecone / Weaviate / Qdrant / pgvector | Convex vector search |
| AI called from React component or API route | Must go through Convex action only |
| tRPC / GraphQL / Apollo / REST on top of Convex | Convex functions are the API — no extra layer |
| SQL migration files | convex/schema.ts — Convex has no SQL migrations |
| ctx.db inside a Convex action | Call an internal query/mutation instead |
| Material UI / Chakra UI / Ant Design / Mantine | shadcn/ui + Tailwind CSS |
| styled-components / Emotion alongside Tailwind | Tailwind CSS only |
| Jest | Vitest |
| Cypress | Playwright |
| SendGrid / Mailgun / Nodemailer | Resend |

### Automatic blockers — stop and flag immediately if any of these appear in the proposal

- AI provider called from the frontend (React component, Next.js API route, server action)
- Convex proposed for Template F (on-premises) deployment
- Backend function with no auth check before data access
- Multi-tenant query with no orgId/tenant_id filter
- API keys or secrets in source code

---

## MANDATORY PIPELINE — do not skip steps

If the user already ran nexara-stack-selection and nexara-architect-review in this session, read their output and proceed to Step 3. Otherwise run Steps 1 and 2 now.

### Step 1 — Stack Selection

Apply the nexara-stack-selection decision procedure:

1. Ask only for missing facts that change the branch (on-prem? AI core? scale? budget? login?).
2. Walk the decision tree in order, stop at the first match:
   - Mobile? -> Template D + Convex + Clerk.
   - Content site, no login? -> Template A (Next.js SSG or WordPress/Webflow).
   - Ecommerce? -> Shopify/WooCommerce or custom marketplace (Template B + Convex).
   - On-premises? -> Template F + Mode 3 or 4 + PostgreSQL. No Convex. No Supabase. No Clerk (use Keycloak).
   - AI is core? -> Template E on B/C + Convex actions + Convex vector search.
   - Lowest cost / edge? -> Template C + Mode 2 (Hono + Cloudflare).
   - Standard SaaS or portal? -> Template B + Mode 1 (Convex + Clerk).
   - Enterprise client, large team? -> Template B frontend + Mode 3/5/6.
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

### Step 2 — Architect Review

Review the Step 1 recommendation against `references/guardrails.md`. Issue a verdict:

- Stack fit: does the template match the decision tree?
- On-prem + Convex? Automatic blocker — Convex is cloud-only.
- On-prem + Supabase or Vercel? Automatic blocker — both are hosted services.
- Auth: Clerk + Convex for Mode 1; Auth.js or Keycloak for self-hosted/enterprise. RBAC at backend layer.
- Security: secrets in env; every Convex function checks ctx.auth.getUserIdentity(); input validation; HTTPS; rate limits; audit logs; Sentry.
- Data/tenancy: Convex default for cloud (Mode 1); PostgreSQL for on-prem/SQL-heavy modes. If multi-tenant: orgId on every Convex table, all queries filtered by orgId.
- AI: all AI calls through Convex actions only — frontend-to-provider is automatic blocker. Log model/cost/latency/userId.
- Technology compliance: no forbidden substitutions from the TECHNOLOGY LOCK above.
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

### Step 3 — UI/UX Implementation

Only after Step 2 returns APPROVE or APPROVE WITH CONDITIONS.

1. **Apply Nexara design defaults** from `references/nexara-design-defaults.md`:
   - Component foundation: Tailwind CSS + shadcn/ui for web; NativeWind/Tamagui for Expo mobile.
   - No other component library. No styled-components. No Emotion.
   - Accessibility: contrast >= 4.5:1, visible focus states, aria-label on icon buttons, full keyboard support.
   - Design tokens only — no hardcoded hex or pixel values.
   - Dark mode where brand allows.
   - No AI calls from the frontend. No direct API calls from components.
   - Responsive by default for web; Apple HIG / Material for mobile.
2. **Build to the approved stack.** For Convex: use `useQuery` for reactive reads, `useMutation` for writes, no direct DB access from components. No useEffect + fetch. No React Query.
3. **Produce the handoff contract:** for each component, the data shape it reads (Convex query name + return type) and the mutations/actions it triggers (names + args).

### Step 4 — Backend Implementation

Build to the approved mode from `references/backend-modes.md`.

**For Mode 1 (Convex) — apply these rules exactly:**

- **Schema:** define all tables in `convex/schema.ts` with `defineTable`. Add indexes for every query pattern. Add `orgId` field to every tenant-owned table. Never create SQL migration files.
- **Queries:** read-only, reactive. Always validate `ctx.auth.getUserIdentity()` at the start. Always filter by `orgId`. Return only what the UI needs — no over-fetching.
- **Mutations:** all writes. Validate inputs with a schema (use convex/values validators). Check auth and org membership before mutating.
- **Actions:** side-effects only (AI calls, email via Resend, external APIs). Never access `ctx.db` directly in an action — call internal mutations or queries instead. Log model, prompt version, cost, latency, userId, orgId for every AI call.
- **Auth:** Clerk provides identity. Every function validates `ctx.auth.getUserIdentity()`. Implement roles in the Convex DB (user -> org -> role -> permissions table). Never use Supabase Auth, NextAuth, or custom JWT.
- **File storage:** use Convex file storage. Generate upload URLs server-side via a mutation. Store `storageId` in DB. Generate download URLs on demand in a query. Never use S3, R2, or Cloudinary for Mode 1.
- **Scheduled functions:** `convex/crons.ts` for recurring jobs; `ctx.scheduler.runAfter` for delayed jobs. Never use BullMQ, Inngest, or cron endpoints in Next.js for Mode 1.
- **Vector search:** `vectorIndexes` in convex/schema.ts. Never add Pinecone, Weaviate, or pgvector for Mode 1.
- **AI routing (hard rule):** all AI calls inside Convex actions only. Log model, prompt version, cost estimate, latency, userId, orgId on every call. Apply per-org and per-user rate limits. Set a monthly spend cap.
- **Real-time (hard rule):** Convex reactive queries are real-time by default. Never add Socket.io, Pusher, Ably, or any WebSocket library for Mode 1.
- **State management (hard rule):** do not install Zustand, Redux, MobX, Jotai, or Recoil for server data. Convex handles reactive server state. React useState/useReducer/useContext are fine for local UI state.

**For Modes 2-6:** apply standard rules from `references/backend-modes.md`. PostgreSQL + Drizzle or Prisma (pick one, never mix) for Modes 3-6. tenant_id on every tenant-owned table, all queries tenant-scoped. See `references/guardrails.md` for mode-specific forbidden substitutions.

### Step 5 — Hand off to testing

List for nexara-testing:
- All screens and flows, naming critical paths (login, signup, core journey, checkout) for Playwright.
- All Convex queries/mutations/actions (or REST endpoints for other modes) with input/output shapes and auth requirements.
- Components with complex states for component-level tests.
- Vitest for all unit and integration tests — no Jest.

---

## Output

Step 1 recommendation + Step 2 verdict + then:

**UI layer:** component/page code + design-decision note + list of flows.

**Backend layer (Convex):** schema definition + queries/mutations/actions + cron jobs + auth rules.

**Backend layer (other modes):** code in approved framework + endpoint list + migrations.

---

See `references/nexara-design-defaults.md` for UI guardrails.
See `references/backend-modes.md` for all backend mode details.
See `references/guardrails.md` for the full approved/forbidden technology table.
