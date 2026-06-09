---
name: nexara-fullstack
description: Designs and implements both the UI/UX and backend for a Nexara project in one pass. Applies Nexara mandated design defaults (Tailwind CSS + shadcn/ui, accessibility baseline) for the frontend and the approved backend mode (1-6) for the server side, following all standard Nexara rules for database, auth, multi-tenancy, security, and AI routing. Use this skill whenever you need to build or review both the UI and backend together - full-stack features, end-to-end flows, or complete project build-outs after architect approval. Can also be used for either layer alone when the other already exists.
---

# Nexara Fullstack (UI/UX + Backend)

This skill merges `nexara-uiux` and `nexara-backend` into a single end-to-end build pass. Both layers are designed to hand off cleanly to each other - the UI exposes the data shapes it needs, the backend exposes matching endpoints/server actions, and both feed into `nexara-testing`.

---

## Dependency (UI layer)

The UI layer requires the `ui-ux-pro-max` skill (MIT-licensed, by nextlevelbuilder). Install it as-is:

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

If unavailable, Nexara guardrails below still apply - but say so and recommend installing for full design intelligence.

---

## Workflow

### Step 1 - Read the approved approach
Read the output of `nexara-architect-review`: approved template (A-F), backend mode (1-6), target platform (web / mobile / desktop), database, auth, tenancy model, and any conditions. Do not start either layer before architect approval.

### Step 2 - UI/UX pass

1. **Generate design direction with ui-ux-pro-max.** Let it propose the design system (pattern, palette, typography, components) for the project's industry and product type.
2. **Apply Nexara guardrails** from `references/nexara-design-defaults.md` - these override anything that conflicts:
   - Component foundation: Tailwind CSS + shadcn/ui for custom web UI; NativeWind/Tamagui for Expo mobile; client-mandated design system when required.
   - Accessibility baseline: contrast >= 4.5:1 (3:1 large), visible focus states, descriptive alt text, aria-label for icon-only buttons, full keyboard support.
   - Tokens only - no hardcoded hex or pixel values.
   - Dark mode where the brand allows.
   - No AI calls from the frontend (enforced system-wide).
   - Responsive by default for web; platform conventions (Apple HIG / Material) for mobile.
3. **Build to the approved stack** - Next.js + React for app-like web; Next.js SSG for static content sites; Expo React Native for mobile.
4. **Produce the handoff contract:** for each interactive component, document the data shape it renders and the actions it triggers.

### Step 3 - Backend pass

1. **Pick the mode** from `references/backend-modes.md` and build to its patterns. Do not switch modes, ORMs, or auth providers mid-build.
2. **Apply standard backend rules** (apply to every mode):
   - **Database:** PostgreSQL by default. Files in R2/Supabase Storage/MinIO; metadata in PostgreSQL; signed URLs for private files.
   - **ORM:** use what the architect approved (Drizzle default; Prisma when richer tooling was chosen). One ORM per app.
   - **Auth and permissions:** enforce in API/middleware, never UI-only. Implement user to organization to role to permission to membership to audit log. RBAC at the API layer.
   - **Multi-tenant:** tenant_id on every tenant-owned table, every query tenant-scoped, indexes on tenant_id, per-tenant storage prefixes/buckets.
   - **Security baseline:** secrets in env/secret manager; input validation on every endpoint; rate limits on auth, forms, and AI endpoints; HTTPS only; HTTP-only cookies for web sessions; audit logs for sensitive actions; production DB backups; error monitoring (Sentry); dependency scanning.
   - **AI routing (hard rule):** never call an AI provider from the frontend. All AI goes through the backend via the provider-abstraction layer, logging model, prompt version, cost estimate, latency, and user/client ID, with per-client and per-user rate limits and a monthly spend cap.
   - **Background jobs:** Cloudflare Queues/Inngest by default; BullMQ+Redis or Temporal for heavier workflows.
3. **Expose endpoints/server actions** that match the data shapes the UI needs (from Step 2).

### Step 4 - Hand off to testing

List for `nexara-testing`:
- All delivered screens and user flows, naming critical paths (login, signup, core business flow, checkout) for Playwright coverage.
- All endpoints/server actions with request/response shapes and which require auth or tenant scoping.
- Components with complex states that need component-level tests.

---

## Output

**UI layer:**
- Component or page code in the approved stack (Tailwind + shadcn/ui or platform equivalent).
- Design-decision note: pattern, palette, typography, guardrails applied.
- List of delivered flows/screens.

**Backend layer:**
- Backend code in the approved mode's framework, structured per `references/backend-modes.md`.
- Endpoint/server action list with request/response shapes and auth/tenant requirements.
- Migrations for any schema change, with tenant_id and indexes where multi-tenant.

---

See `references/nexara-design-defaults.md` for full UI guardrail detail and handoff contract.
See `references/backend-modes.md` for per-mode framework, structure, and when-to-use detail.
