# Decision Matrix (reference)

Load this when you need the full per-type rationale or the starting-cost targets behind a recommendation. The main flowchart in SKILL.md is the fast path; this is the lookup detail.

## By application type

| Application type | Recommended stack | Why | Starting cost |
| --- | --- | --- | --- |
| Static website / landing page | Next.js (SSG) + Cloudflare Pages | Architecture Overview section 01: Next.js static export on global Cloudflare CDN, near-zero hosting cost, strong SEO | $0-$5/mo |
| Corporate site with CMS | WordPress, Webflow, or Next.js + headless CMS | Non-technical clients edit content; cheaper than a custom admin panel | $5-$50/mo |
| Dynamic business website | Next.js + Convex + Clerk | Handles forms, login, admin, SEO, and future app growth | $0-$30/mo |
| SaaS dashboard / customer portal | Next.js + Convex + Clerk (Template B + Mode 1) | Best speed-to-quality balance for modern SaaS; Convex replaces DB + auth + storage + real-time | $25-$75/mo |
| Multi-tenant SaaS product | Next.js + Convex + Clerk, orgId on every tenant-owned table | Strong reuse, reactive real-time, scalable tenancy, easier billing/org management | $25-$150/mo |
| CRM / ERP / internal business app | Laravel, .NET, Spring Boot, or NestJS + PostgreSQL | CRUD-heavy systems benefit from mature backend patterns, reporting, permissions | $25-$150/mo |
| Admin-only internal tool | Retool, Appsmith, ToolJet, or Next.js | Low-code is faster and cheaper when the UI is mostly internal ops | $0-$100/mo |
| Cheap API / microservice | Hono + Cloudflare Workers | Very low hosting cost, fast global edge, simple TypeScript | $5-$30/mo |
| Complex enterprise API | NestJS, Spring Boot, .NET, or FastAPI | Better structure for large domains, integrations, jobs, team scaling | $50+/mo |
| AI / data-heavy backend | FastAPI + Python workers + PostgreSQL/pgvector | Strongest AI/data ecosystem and ML integration | $25-$150/mo + AI usage |
| Chatbot | Hono or FastAPI + Convex actions or AI gateway + logs + optional vector search | Lightweight for simple bots; Python better for document pipelines | $5-$100/mo + AI usage |
| AI automation tool | FastAPI or Convex actions + scheduled functions + database + AI gateway | Keeps cost, retries, rate limits, and long-running work controlled | $10-$150/mo + AI usage |
| Mobile app | Expo React Native + Convex backend | One codebase for iOS+Android; cheaper than two native apps; Convex provides real-time sync | $25-$100/mo |
| High-performance native mobile | Native Kotlin + Swift | Better for heavy device APIs, advanced animation, offline-first | Custom |
| Ecommerce store | Shopify or WooCommerce | Payments, inventory, checkout, taxes, plugins already solved | Platform cost |
| Custom ecommerce / marketplace | Next.js + Convex + Clerk + Stripe | Needed when vendors/commissions/custom pricing exceed Shopify | $50-$200/mo |
| On-premises server app | Docker Compose + PostgreSQL + Redis + MinIO + Caddy/Nginx + Keycloak if needed | Portable, private, works for enterprise/government/private networks. No Convex. No Supabase. | Server cost |
| Enterprise / government app | Spring Boot, .NET, NestJS, or FastAPI + PostgreSQL + Keycloak + containers | Strong compliance, hiring availability, observability, deployment control | Custom |

## By use case (stack selection guide)

| Use case | Recommended stack | Why |
| --- | --- | --- |
| Marketing / portfolio / brochure site | Next.js (SSG) | Static export on Cloudflare Pages — minimal runtime cost, very fast, strong SEO |
| Client edits pages frequently | WordPress or Webflow | Cheaper content ops; no developer needed per edit |
| Blog / docs-heavy site | Next.js (SSG) + MDX/Contentlayer, Docusaurus, or WordPress | Content-first tooling, SEO, markdown/CMS, low maintenance |
| Modern SaaS MVP | Next.js + Convex + Clerk | Fastest path to production with login, dashboard, reactive data, real-time, file storage in one platform |
| Enterprise CRUD system | Laravel, .NET, Spring Boot, or NestJS | Mature patterns for forms, permissions, audit, transactions |
| Low-cost serverless API | Hono + Cloudflare Workers | Small, fast, cheap, TypeScript-friendly, deploys near users |
| Large backend platform | NestJS, Spring Boot, or .NET | Modular architecture, DI, testing patterns, enterprise familiarity |
| AI service / document processing | FastAPI + Python | Best ecosystem for AI libraries, embeddings, parsing, data workflows |
| Mobile MVP | Expo React Native + Convex | One team ships Android+iOS fast with shared TypeScript + real-time sync |
| Enterprise mobile, deep native | Kotlin + Swift | Native APIs, performance, offline behavior, platform UX |
| Standard ecommerce | Shopify | Checkout, payments, inventory, discounts, security handled |
| WordPress-based commerce | WooCommerce | Good when client already lives in WordPress |
| Internal admin dashboard | Retool / Appsmith / ToolJet | Fastest/cheapest for ops teams; custom UI not always worth it |
| On-prem / private deployment | Docker Compose + PostgreSQL + Redis + MinIO | Easy to install, back up, migrate, run without cloud. No Convex. No Supabase. |
| Regulated enterprise identity | Keycloak + OIDC/SAML | Open-source, self-hostable, enterprise SSO |
