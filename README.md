# nexara-fullstack skill

A combined Cowork skill that merges `nexara-uiux` and `nexara-backend` into a single end-to-end build pass for Nexara projects.

## What it does

Handles both layers of a Nexara project in one skill:
- **UI/UX** — Applies Nexara's design guardrails (Tailwind CSS + shadcn/ui, accessibility baseline, token discipline) on top of the `ui-ux-pro-max` design intelligence skill.
- **Backend** — Implements the approved backend mode (1–6) with all standard Nexara rules: PostgreSQL, auth/RBAC, multi-tenancy, security baseline, AI routing, background jobs.

Both layers produce a clean handoff contract to `nexara-testing`.

## Files

```
nexara-fullstack/
├── SKILL.md                          # Main skill definition
└── references/
    ├── nexara-design-defaults.md     # UI guardrails & handoff contract
    └── backend-modes.md              # Backend mode 1–6 detail
```

## Installation

### Option 1 — Install via .skill file (recommended)

1. Download `nexara-fullstack.skill` from this repo
2. Open **Claude Cowork**
3. Go to **Settings → Capabilities → Skills**
4. Click **Install from file** and select `nexara-fullstack.skill`

### Option 2 — Manual install

1. Clone this repo:
   ```bash
   git clone https://github.com/Gopinath2023/nexara-fullstack-skill.git
   ```
2. Copy the `nexara-fullstack/` folder into your Claude skills directory:
   - **Windows:** `%APPDATA%\Claude\skills\`
   - **macOS:** `~/Library/Application Support/Claude/skills/`
3. Restart Cowork — the skill will appear automatically.

## Dependencies

This skill is partly a **wrapper**:

| Layer | Type | Depends on |
|---|---|---|
| Backend | Self-contained | Nothing — all rules are built in |
| UI/UX | Wrapper | `ui-ux-pro-max` by nextlevelbuilder |

The backend half (database, auth, multi-tenancy, security, AI routing, backend modes 1–6) works fully standalone.

The UI/UX half wraps `ui-ux-pro-max`, which provides the design intelligence (palettes, typography, UX patterns, per-industry reasoning). Without it, Nexara's guardrails (Tailwind + shadcn/ui, accessibility, token discipline) still apply, but you lose the design intelligence layer.

### Install the required third-party skill

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

> **Recommended:** Install `ui-ux-pro-max` before using this skill for any UI work. The