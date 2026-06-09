---
name: nexara-fullstack
description: Designs and implements both the UI/UX and backend for a Nexara project in one pass. Enforces the full Nexara pipeline - runs nexara-stack-selection to pick the stack, then nexara-architect-review to approve it, then implements UI and backend together. Use this skill for full-stack features, end-to-end flows, or complete project build-outs. Can also be used for either layer alone after the pipeline has already run.
---

# Nexara Fullstack (UI/UX + Backend)

This skill runs the full Nexara delivery pipeline in one session: stack selection -> architect review -> UI/UX + backend implementation. It does not skip the gatekeeping steps.

---

## MANDATORY PIPELINE - do not skip steps

Before writing any code, you must complete Steps 1 and 2. If the user has already run nexara-stack-selection and nexara-architect-review in this session, read their output and proceed. If not, run them now inline.

### Step 1 - Stack Selection (nexara-stack-selection)

Apply the full nexara-stack-selection decision procedure:

1. Read the project description. Ask only for missing facts that change the template branch (login required? on-prem? AI core? scale? budget?).
2. Walk the decision tree in order, stop at the first match:
   - Mobile? -> Template D + shared B/C backend.
   - Content site, no login? -> Template A (Next.js SSG or WordPress/Webflow by edit needs).
   - Ecommerce? -> Shopify/WooCommerce or custom marketplace (B/Laravel).
   - On-premises? -> Template F + Mode 3 or 4.
   - AI is core? -> Template E on B/C + Mode 4 (FastAPI) + pgvector.
   - Lowest cost? -> Template C + Mode 2. Standard SaaS? -> Template B + Mode 1. Enterprise? -> B + Mode 3/5/6.
3. Output the recommendation in this format:

```
## Stack recommendation: [project]
Template: [A-F] - [name]
Backend mode: [1-6]
Stack: Frontend / Backend / DB / Auth / Hosting / Other
Cost target: $X-$Y/month
Why: [2-3 sentences]
Watch-outs: [when to revisit]
```

### Step 2 - Architect Review (nexara-architect-review)

Review the Step 1 recommendation before proceeding. Check each area and issue a verdict:

- Stack fit: does the template match the decision tree for this project?
- Deployment: is the model explicit and consistent with hosting choice?
- Auth: correct variant for the deployment model? RBAC at API layer?
- Security: secrets in env, input validation, HTTPS, rate limits, audit logs, DB backups, Sentry.
- Data/tenancy: PostgreSQL default; tenant_id on every tenant-owned table if multi-tenant.
- AI handling: all AI calls through backend only - any frontend-to-provider call is an automatic blocker.
- Cost: infrastructure matches the budget target?
- Contradictions: deployment vs hosting, auth vs deployment, tenancy vs data model, cost vs infra.

Output the verdict:

```
## Architect review: [project]
Verdict: APPROVE | APPROVE WITH CONDITIONS | REJECT
Summary: [2-3 sentences]
Blockers: [issue -> fix] (must resolve before building)
Conditions: [issue -> fix] (resolve during build)
Strengths: [what is correctly aligned]
```

Do not proceed to Step 3 if the verdict is REJECT. Address blockers first, then re-run Step 2.

---

### Step 3 - UI/UX Implementation

Only begin after Step 2 returns APPROVE or APPROVE WITH CONDITIONS.

1. **Generate design direction with ui-ux-pro-max** (if installed). Let it propose design system, palette, typography for the project's industry.
2. **Apply Nexara guardrails** from `references/nexara-design-defaults.md`:
   - Component foundation: Tailwind CSS + shadcn/ui for web; NativeWind/Tamagui for Expo mobile.
   - Accessibility: contrast >= 4.5:1, visible focus states, aria-label on icon buttons, full keyboard support.
   - Tokens only - no hardcoded hex or pixel values.
   - Dark mode where brand allows.
   - No AI calls from the frontend.
   - Responsive by default for web; Apple HIG / Material for mobile.
3. **Build to the approved stack** from Step 1.
4. **Produce the handoff contract**: for each component, document the data shape it renders and actions it triggers.

### Step 4 - Backend Implementation

Build to the approved mode from `references/backend-modes.md`. Apply to every mode:

- **Database:** PostgreSQL. Files in R2/Supabase Storage/MinIO; metadata in DB; signed URLs for private files.
- **ORM:** architect-approved (Drizzle default; Prisma when richer tooling chosen). One ORM per app.
- **Auth:** enforce in API/middleware, never UI-only. user -> org/tenant -> role -> permission -> membership -> audit log. RBAC at API layer.
- **Multi-tenant:** tenant_id on every tenant-owned table, all queries tenant-scoped, indexes on tenant_id.
- **Security:** secrets in env; input validation; rate limits on auth/forms/AI; HTTPS; HTTP-only cookies; audit logs; DB backups; Sentry; dependency scanning.
- **AI routing (hard rule):** never from the frontend. All AI through backend provider-abstraction layer with model/prompt/cost/latency/user logging, rate limits, and spend cap.
- **Background jobs:** Cloudflare Queues/Inngest by default; BullMQ+Redis or Temporal for heavier workflows.

### Step 5 - Hand off to testing

List for nexara-testing:
- All screens and flows, naming critical paths (login, signup, core journey, checkout) for Playwright.
- All endpoints/server actions with request/response shapes and auth/tenant requirements.
- Components with complex states for component-level tests.

---

## Output

**Stack recommendation** (Step 1) + **Architect verdict** (Step 2) + then:

**UI layer:** component/page code + design-decision note + list of flows delivered.

**Backend layer:** code in approved mode + endpoint list with shapes + migrations with tenant_id where needed.

---

See `references/nexara-design-defaults.md` for UI guardrails.
See `references/backend-modes.md` for backend mode detail.
