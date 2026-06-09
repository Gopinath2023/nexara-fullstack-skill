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
   git clone https://github.com/ganumalasetty/nexara-fullstack-skill.git
   ```
2. Copy the `nexara-fullstack/` folder into your Claude skills directory:
   - **Windows:** `%APPDATA%\Claude\skills\`
   - **macOS:** `~/Library/Application Support/Claude/skills/`
3. Restart Cowork — the skill will appear automatically.

## Dependencies

This skill requires the `ui-ux-pro-max` skill for full design intelligence. Install it in Cowork:

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

If not installed, the skill still works but with reduced design output.

## Usage

Use this skill after `nexara-architect-review` has approved the approach. Trigger it by describing a full-stack build task, e.g.:

- *"Build the dashboard UI and API for this project"*
- *"Implement the login flow end to end"*
- *"Set up the backend and frontend for the onboarding screen"*

## Pipeline position

```
nexara-client-discovery
       ↓
nexara-stack-selection
       ↓
nexara-architect-review   ← must run first
       ↓
nexara-fullstack          ← this skill (UI + Backend together)
       ↓
nexara-testing
```

## License

MIT
