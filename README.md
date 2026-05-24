# chartdeck-dev — Paperclip Company Package

Importable Paperclip company definition for [chartdeck](https://github.com/nucleusjay/chartdeck).

## Layout (Paperclip portable-package schema v5)

```
chartdeck-dev-paperclip/
├── COMPANY.md                   # required — frontmatter (name, slug, description, includes)
├── .paperclip.yaml              # extension — adapter/runtime/permissions/envInputs per agent
├── agents/
│   ├── ceo/AGENTS.md            # Hermes — claude_local (opus), orchestrator
│   ├── coding/AGENTS.md         # Codex — codex_local (gpt-5-codex), TDD writer
│   ├── design/AGENTS.md         # Google — gemini_local (gemini-2.5-pro), multimodal
│   └── code-review/AGENTS.md    # Claude — claude_local (sonnet), reviewer
└── README.md                    # this file
```

## How to import

1. Open Paperclip at `https://paperclip.inlakesh.tech/`.
2. Get past the Traefik basic-auth popup, then log in to Paperclip.
3. **Org Chart → Import → GitHub repo**.
4. URL: `https://github.com/nucleusjay/chartdeck-paperclip`
5. Target: **Create new company** (or merge into the existing one with the orphan CEO entry; collision strategy defaults to "Rename on conflict").
6. **Preview import** → confirm.

## After import — set secrets

In Paperclip's secret store (do **not** commit these to the repo):

- `ANTHROPIC_API_KEY` — Hermes (CEO) + Claude (reviewer)
- `OPENAI_API_KEY` — Codex (coding)
- `GEMINI_API_KEY` — Google (design)
- `GITHUB_TOKEN` — Hermes (pushes + PRs)
- `TELEGRAM_BOT_TOKEN` — Hermes (owner bridge) — provision via `@BotFather` first
- `TELEGRAM_ALLOWED_USERS` — your Telegram user ID (comma-separated for multiple)
- `SSH_PRIVATE_KEY_PATH` — path inside the agent's container to the chartdeck deploy key

## Dispatch flow

```
Owner → Telegram → Hermes
Hermes → brainstorm → spec → Owner approves
Hermes → plan → Owner approves
Hermes → loop:
  Hermes dispatches Codex (one task) → DONE + commit SHA
  Hermes dispatches Claude (spec review) → ✅ or ❌
  Hermes dispatches Codex (fix) if ❌
  Hermes dispatches Claude (code-quality review) → ✅ or ❌
  Hermes dispatches Codex (fix) if ❌
Hermes merges feature branch → master (with owner confirmation)
Hermes runs deploy/update.sh on VPS (with owner confirmation)
Hermes reports "shipped" on Telegram
```

Google fires in parallel only on design-flagged tasks; output routes through Hermes to Codex.

## Schema notes

This package follows Paperclip portable schema v5 (the `company-portability.js` service):

- `COMPANY.md` frontmatter: `name` (required), `slug`, `description`, `includes` (list of agent/project/skill paths).
- `agents/<slug>/AGENTS.md` frontmatter: `kind: agent` (required to avoid warning), `name`, `slug`, `title`, `role`, `reportsTo`.
- `.paperclip.yaml`: extension config — `company.brandColor`, `company.requireBoardApprovalForNewAgents`, `agents.<slug>.{role,icon,capabilities,adapter,runtime,permissions,budgetMonthlyCents,envInputs}`, `sidebar`.

Adapter types installed on the target Paperclip server (`/home/paperclip/.npm/_npx/.../node_modules/@paperclipai/`):

- `adapter-claude-local` → Hermes, Claude
- `adapter-codex-local` → Codex
- `adapter-gemini-local` → Google

If Paperclip rejects any field in `.paperclip.yaml`, it's silently ignored on import (the markdown is the source of truth for the agent's identity/instructions). The YAML is best-effort sugar.
