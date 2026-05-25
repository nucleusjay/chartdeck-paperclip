# chartdeck-dev — Paperclip Company Package

Importable JSON-based Paperclip company definition for [chartdeck](https://github.com/nucleusjay/chartdeck).

## Layout (Paperclip JSON-Package Schema)

```
chartdeck-dev-paperclip/
├── company.json                      # company manifest (name, slug, description, brand color)
├── README.md                         # this file (import instructions)
├── environments/
│   └── default.env                   # shared environment template (secrets blank)
└── agents/
    ├── ceo/                          # Hermes (CEO, strategic orchestration)
    │   ├── agent.json                # role, runtime, hermes_local adapter config, permissions
    │   └── instructions.md           # identity, delegation rules, Telegram bridge commands
    ├── coding/                       # Codex (implementation engineer, TDD writer)
    │   ├── agent.json
    │   └── instructions.md
    ├── design/                       # Google (UI/UX lead, custom layouts and styling)
    │   ├── agent.json
    │   └── instructions.md
    └── code-review/                  # Claude (reviewer, spec compliance and quality gate)
        ├── agent.json
        └── instructions.md
```

## How to import

1. Open Paperclip at `https://paperclip.inlakesh.tech/`.
2. Authenticate through the Traefik basic-auth pop-up and log in.
3. Select **Org Chart → Import → GitHub repo**.
4. URL: `https://github.com/nucleusjay/chartdeck-paperclip`
5. Target: **Create new company** (collision strategy: rename).
6. **Preview import** → confirm.

## After import — set secrets

In Paperclip's secret store (do **not** commit these to the repository):

- `ANTHROPIC_API_KEY` — Sourced from your active OpenRouter key `sk-or-v1-...` (Hermes + Claude reviewer).
- `OPENAI_API_KEY` — Sourced from your active OpenRouter key `sk-or-v1-...` (Codex coding).
- `GEMINI_API_KEY` — Sourced from your active Google API Key `AIzaSyBM...` (Google designer).
- `GITHUB_TOKEN` — Hermes (pushes + PRs) — populated with a placeholder or GitHub PAT.
- `TELEGRAM_BOT_TOKEN` — Hermes bridge token `8424082896:AAHAIK2fyf...`.
- `TELEGRAM_ALLOWED_USERS` — Your Telegram ID `7629196096`.
- `SSH_PRIVATE_KEY_PATH` — Path to deploy key `/home/paperclip/.ssh/chartdeck_deploy`.

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
