# Alterra — Agentic SDLC Lab 1

This repo contains the **pets-workshop** application plus the BMAD-driven planning
and design work products for the Agentic SDLC lab.

## What's tracked vs. generated

The BMAD framework and its per-IDE skill/agent files are **not committed** — they are
vendored/generated and reproduced locally by the BMAD installer (same idea as
`node_modules`). What *is* committed:

- `pets-workshop/` — the application
- `_bmad-output/`, `design-artifacts/`, `docs/` — BMAD work products (briefs, PRDs,
  architecture, stories, design system)
- `_bmad/config.toml`, `_bmad/custom/`, `_bmad/_config/manifest.yaml` — the inputs that
  pin and reproduce the toolset

Ignored (regenerated per developer): `_bmad/` modules, `.claude/skills/`, `.agent/`,
`.agents/`, `.github/agents/`, and personal config (`config.user.toml`,
`.claude/settings.local.json`).

## Setup (one-time, per developer)

Install BMAD to regenerate the agents/skills for your IDE. The pinned module versions
are recorded in `_bmad/_config/manifest.yaml`.

```bash
npx bmad-method install
# When prompted, select your IDE target(s): claude-code (and/or codex, cursor, github-copilot)
```

After install, the BMAD skills/agents are available in your tool of choice.

## Running the app

See [`pets-workshop/README.md`](pets-workshop/README.md).
