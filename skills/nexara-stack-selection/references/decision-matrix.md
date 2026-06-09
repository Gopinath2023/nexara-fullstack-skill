# Decision Matrix (reference)

Load this when you need the full per-type rationale or the starting-cost targets behind a recommendation. The main flowchart in SKILL.md is the fast path; this is the lookup detail.

## By application type

| Application type | Recommended stack | Why | Starting cost |
| --- | --- | --- | --- |
| Static website / landing page | Astro + Cloudflare Pages/Workers | Faster, simpler, cheaper, strong SEO, less complexity than a full app framework | $0–$5/mo |
| Corporate site with CMS | WordPress, Webflow, or Astro + headless CMS | Non-technical clients edit content; cheaper than a custom admin panel | $5–$50/mo |
| Dynamic business website | Next.js or Laravel + PostgreSQL/Supabase | Handles forms, login, admin, SEO, and future app growth | $0–$30/mo |
| SaaS dashboard / customer portal | Next.js + PostgreSQL + Drizzle/Prisma + Supabase/Auth.js/Clerk | Best speed-to-quality balance for modern SaaS | $25–$75/mo |
| Multi-tenant SaaS product | Next.js + dedicated API if needed + PostgreSQL tenant model | Strong reuse, scalable tenancy, easier billing/org management | $25–$150/mo |
| CRM / ERP / internal business app | Laravel, .NET, Spring Boot, or Next.js | CRUD-heavy systems benefit from mature backend patterns, reporting, permissions | $25–$150/mo |
| Admin-only internal tool | Retool, Appsmith, ToolJet, or Next.js | Low-code is faster and cheaper when the UI is mostly internal ops | $0–$100/mo |
| Cheap API / microservice | Hono + Cloudflare Workers | Very low hosting cost, fast global edge, simple TypeScript | $5–$30/mo |
| Complex enterprise API | NestJS, Spring Boot, .NET, or FastAPI | Better structure for large domains, integrations, jobs, team scaling | $50+/mo |
| AI / data-heavy backend | FastAPI + Python workers + PostgreSQL/pgvector | Strongest AI/data ecosystem and ML integration | $25–$150/mo + AI usage |
| Chatbot | Hono or FastAPI + AI provider layer + PostgreSQL logs + optional vector search | Lightweight for simple bots; Python better for document pipelines | $5–$100/mo + AI usage |
| AI automation tool | FastAPI or Hono + queues + database + AI gateway | Keeps cost, retries, rate limits, and long-running work controlled | $10–$150/mo + AI usage |
| Mobile app | Expo React Native + shared backend | One codebase for iOS+Android; cheaper than two native apps | $25–$100/mo |
| High-performance native mobile | Native Kotlin + Swift | Better for heavy device APIs, advanced animation, offline-first | Custom |
| Ecommerce store | Shopify or WooCommerce | Payments, inventory, checkout, taxes, plugins already solved | Platform cost |
| Custom ecommerce / marketplace | Next.js or Laravel + PostgreSQL + payments + jobs | Needed when vendors/commissions/custom pricing exceed Shopify | $50–$200/mo |
| On-premises server app | Docker Compose + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak if needed | Portable, private, works for enterprise/government/private networks | Server cost |
| Enterprise / government app | Spring Boot, .NET, NestJS, or FastAPI + PostgreSQL + Keycloak + containers | Strong compliance, hiring availability, observability, deployment control | Custom |

## By use case (stack selection guide)

| Use case | Recommended stack | Why |
| --- | --- | --- |
| Marketing / portfolio / brochure site | Astro | Minimal runtime cost, very fast, simple content, strong SEO |
| Client edits pages frequently | WordPress or Webflow | Cheaper content ops; no developer needed per edit |
| Blog / docs-heavy site | Astro, Docusaurus, or WordPress | Content-first tooling, SEO, markdown/CMS, low maintenance |
| Modern SaaS MVP | Next.js + PostgreSQL + Supabase/Neon | Fastest path to production with login, dashboard, API, DB |
| Enterprise CRUD system | Laravel, .NET, Spring Boot, or NestJS | Mature patterns for forms, permissions, audit, transactions |
| Low-cost serverless API | Hono + Cloudflare Workers | Small, fast, cheap, TypeScript-friendly, deploys near users |
| Large backend platform | NestJS, Spring Boot, or .NET | Modular architecture, DI, testing patterns, enterprise familiarity |
| AI service / document processing | FastAPI + Python | Best ecosystem for AI libraries, embeddings, parsing, data workflows |
| Mobile MVP | Expo React Native | One team ships Android+iOS fast with shared TypeScript |
| Enterprise mobile, deep native | Kotlin + Swift | Native APIs, performance, offline behavior, platform UX |
| Standard ecommerce | Shopify | Checkout, payments, inventory, discounts, security handled |
| WordPress-based commerce | WooCommerce | Good when client already lives in WordPress |
| Internal admin dashboard | Retool / Appsmith / ToolJet | Fastest/cheapest for ops teams; custom UI not always worth it |
| On-prem / private deployment | Docker Compose + PostgreSQL + Redis + MinIO | Easy to install, back up, migrate, run without cloud |
| Regulated enterprise identity | Keycloak + OIDC/SAML | Open-source, self-hostable, enterprise SSO |
