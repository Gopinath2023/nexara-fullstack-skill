# Nexara Design Defaults & Handoff Contract (reference)

Load this when generating or reviewing UI for a Nexara project. These rules sit on top of ui-ux-pro-max's recommendations and win on conflict.

## Component foundation by platform

| Platform | Foundation |
| --- | --- |
| Custom web app (Next.js / React) | Tailwind CSS + shadcn/ui (Radix primitives) |
| Content site (Next.js SSG) | Tailwind CSS; lightweight components, minimal JS, static export |
| Client with a mandated design system | That system, wrapped to keep Nexara token discipline |
| Mobile (Expo React Native) | NativeWind or Tamagui |

Default Nexara web stack when custom-building: TypeScript + Tailwind + shadcn/ui + React/Next.js.

## Accessibility baseline (apply to every project)

- Color contrast: ≥ 4.5:1 normal text, ≥ 3:1 large text.
- Focus states: visible focus ring (2-4px) on every interactive element.
- Images: descriptive alt text for meaningful images; empty alt for decorative.
- Icon-only buttons: `aria-label` (web) / `accessibilityLabel` (native).
- Keyboard: tab order matches visual order; full keyboard operability.
- Forms: persistent helper text below complex inputs (not placeholder-only); inline validation; disabled states use reduced opacity + cursor change + semantic attribute.
- Progressive disclosure: reveal complex options gradually; don't overwhelm.
- Enterprise/government tier: axe-core audit runs in CI (matches the testing standard).

## Token discipline

- Define color, spacing, radius, shadow, and typography as design tokens (Tailwind theme / CSS variables). Components reference tokens, never raw values.
- Two-3 font weights max; consistent type scale.
- Keep a single source of truth for the palette so light/dark mode and theming stay coherent.

## System-wide rules inherited from the playbook

- No AI calls from the frontend — UI invokes AI only through the Nexara backend.
- Respect the chosen template's hosting model (e.g. Next.js SSG static export for content sites; SSR/Edge for app UI on the approved host).

## Handoff contract

**From architect-review (in):** approved template, platform, design-system constraints, compliance/accessibility tier.

**To backend (out):** for each interactive component, the data shape it renders and the actions it triggers (so the backend exposes matching endpoints/server actions). Flag anything that needs auth or tenant scoping.

**To testing (out):** the list of delivered screens and user flows, explicitly naming the critical paths (login, signup, core business flow, checkout) so the tester can write Playwright coverage. Note any components with complex states for component-level tests.
