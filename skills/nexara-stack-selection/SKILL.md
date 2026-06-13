---
name: nexara-stack-selection
description: Selects the recommended Nexara technology stack, architecture template (A-F), and backend mode (1-6) for a software project, with a hosting cost estimate, using the Nexara Standard Tech Stack Playbook. Use this skill whenever a new Nexara project is being scoped, planned, or quoted; whenever someone asks "what stack should we use", "which template", or "how should we build this"; or whenever someone describes something to be built - a website, landing page, web app, SaaS, customer portal, mobile app, API, microservice, chatbot, AI feature, ecommerce store, internal tool, CRM/ERP, or on-premises/private system - even if they never use the words "stack" or "template". Also use it when a hosting or monthly-cost estimate for a project is requested.
---

# Nexara Stack Selection

Nexara does not sell one tech stack. It sells one engineering system with several architecture templates, then tweaks based on client budget, scale, compliance, and team skill. This skill turns a project description into a concrete recommendation: a template, a backend mode, a default stack, and a starting cost target.

## How to use this skill

1. Read the project description the user gives you. If key facts are missing (login required? on-prem? AI core? expected scale? budget?), ask only for the ones that actually change the branch you land on — do not run a full questionnaire.
2. Walk the decision procedure below **in order, stopping at the first match**. The order matters: mobile and content sites are decided before app-architecture questions, because they short-circuit the rest.
3. Produce the recommendation using the output format at the end.
4. Pull stack details, full decision tables, and exact cost notes from the reference files only when you need them:
   - `references/decision-matrix.md` — full application-type and use-case tables with the "why it is better" rationale and starting-cost targets.
   - `references/templates-and-costs.md` — the stack contents of Templates A-F, the six backend modes, the common foundation defaults, and the hosting cost tiers.

## Decision procedure

Answer each question in order. Stop at the first match.

**Q1 — Is it a mobile app?**
- Mobile, and no deep native device features needed -> **Template D** (Expo React Native) with a shared backend from Template B or C.
- Mobile, and deep native features or heavy performance required -> **Native Kotlin (Android) + Swift (iOS)**. Custom scoping; not a default template.
- Not mobile -> go to Q2.

**Q2 — Is it primarily a content or marketing site with no user login?**
- Yes, and developers manage all content -> **Template A / Next.js (SSG) + Cloudflare Pages** (per Architecture Overview section 01; optional Contentlayer/MDX for content). Cost target: $0-$5/month.
- Yes, and the client needs to self-edit content -> **Template A / WordPress or Webflow**. Cost target: $5-$50/month.
- Yes, but it may need login or app features later -> **Template A / Next.js**, so it can grow into Template B without a rewrite.
- No -> go to Q3.

**Q3 — Is it an ecommerce store?**
- Standard store -> **Shopify or WooCommerce**. Do not custom-build unless the client has genuinely outgrown these platforms.
- Custom marketplace (multi-vendor, complex pricing, commissions) -> **Template B (Next.js + Convex + Clerk)**. Cost target: $50-$200/month.
- No -> go to Q4.

**Q4 — Does the client require on-premises or private-server deployment?**
- Yes -> **Template F** (Docker Compose + PostgreSQL + Redis + MinIO + Caddy/Nginx, Keycloak if SSO needed) with **Backend Mode 3 (NestJS)**, or **Mode 4 (FastAPI)** if AI-heavy. Cost: client-provided server. IMPORTANT: Convex, Supabase, Vercel, and Cloudflare Pages cannot be used for Template F — all are hosted-only.
- No -> go to Q5.

**Q5 — Is AI a core feature, not just a minor add-on?**
- Yes (chatbot, document Q&A, automation, recommendations, agents) -> **Template E** layered on Template B or C, with Convex actions for AI routing and **Backend Mode 4 (FastAPI)** for the Python AI processing layer. Use Convex vector search for retrieval (Mode 1) or pgvector on Neon (Mode 4). Estimate AI usage costs separately.
- No -> go to Q6.

**Q6 — What matters most: lowest cost, or developer experience and Next.js features?**
- Lowest cost and edge performance -> **Template C** (Cloudflare Workers + Hono + R2 + D1/Neon) with **Backend Mode 2**. Cost target: $5-$30/month.
- Standard SaaS or portal where DX and ecosystem matter -> **Template B** (Next.js + Convex + Clerk) with **Backend Mode 1 (Convex)**. Cost target: $25-$75/month.
- Enterprise client, large team, complex integrations, or long-running background jobs -> **Template B frontend** with **Backend Mode 3 (NestJS)**, **Mode 5 (Spring Boot/.NET)**, or **Mode 6 (Laravel)**, chosen by the client's existing ecosystem and hiring pool. Cost target: $50+/month.

## Cross-cutting rules that always apply

Apply these regardless of which template you land on:

- **Default language:** TypeScript for web, mobile, and most APIs; Python where AI or data processing is central.
- **Default database for cloud SaaS (Mode 1):** Convex. Use PostgreSQL for on-prem (Template F), SQL-heavy modes (3-6), or when the client explicitly requires relational SQL. Never default to Supabase DB or Firebase on new projects.
- **Default auth for cloud SaaS (Mode 1):** Clerk. Use Auth.js for self-hosted Next.js. Use Keycloak for enterprise/on-prem SSO/SAML.
- **AI is always backend-routed.** Never call an AI provider from the frontend. Route every AI call through Nexara's backend (Convex action for Mode 1), log model/prompt-version/cost/latency/user, and apply per-client and per-user rate limits.
- **Multi-tenant** starts with an orgId field on every Convex tenant-owned table (Mode 1) or tenant_id on every SQL table (Modes 3-6). All queries must be scoped by org/tenant.
- **Cheapest-good default for small cloud clients:** Cloudflare hosting + Convex (Mode 1) + Resend + Sentry free tier.
- When the answer is close between two templates, prefer the one that scales toward the client's likely next stage without a rewrite.

## Output format

Always respond with this structure:

```
## Recommendation: [Project name or description]

**Template:** [A-F or "Native mobile" / "Shopify" etc.] - [one-line name]
**Backend mode:** [1-6, or "n/a"]
**Recommended stack:**
- Frontend: ...
- Backend/API: ...
- Database: ...
- Auth: ...
- Hosting: ...
- Storage / Email / Payments / AI: [only the ones relevant]

**Starting cost target:** $X-$Y/month [+ AI usage if applicable]

**Why this fits:** [2-4 sentences tying the choice to the decision branch and the client's constraints]

**Watch-outs / when to revisit:** [e.g. "move to Mode 3 if background jobs appear", "switch off Convex if client requires on-prem"]
```

Fill the stack lines from `references/templates-and-costs.md` for the chosen template. Keep the rationale specific to what the user told you — reference their actual constraints (budget, scale, compliance, team), not generic boilerplate.

## Examples

**Example 1**
Input: "A booking dashboard for a gym chain, staff log in to manage classes, maybe 2,000 users in year one, modest budget."
-> No mobile, has login, not ecommerce, not on-prem, AI not core, DX matters over rock-bottom cost -> **Template B + Mode 1**. Next.js + Convex + Clerk, Vercel hosting, ~$25-$75/month.

**Example 2**
Input: "A public document Q&A assistant over a client's policy PDFs."
-> AI is the core feature -> **Template E on Template B/C + Mode 4 (FastAPI)** with pgvector for document retrieval. Base hosting + AI usage; estimate AI cost separately and set a spend cap.

**Example 3**
Input: "A marketing site for a local bakery; the owner wants to update photos and text herself."
-> Content site, no login, client self-edits -> **Template A / WordPress or Webflow**, $5-$50/month. Can be rebuilt on Next.js later if they add online ordering.
