# chartdeck-dev — Paperclip Company

Multi-agent dev shop for the [chartdeck](https://github.com/nucleusjay/chartdeck) project. Designed for import into [Paperclip](https://paperclip.run/).

## Org chart

```
                  ┌─────────────┐
                  │   Hermes    │ (CEO — orchestration)
                  │  claude opus│
                  └──────┬──────┘
                         │ dispatches
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   ┌─────────┐     ┌──────────┐    ┌─────────────┐
   │  Codex  │     │  Google  │    │   Claude    │
   │  gpt-5  │     │ gemini   │    │ claude-sonnet│
   │ coding  │     │ design   │    │ code-review │
   └─────────┘     └──────────┘    └─────────────┘
```

## How to import

Paperclip's import dialog warns that **hand-rezipped archives may not import correctly**. Two safe paths:

### Path A — GitHub repo (recommended)

1. Create a private GitHub repo, e.g. `nucleusjay/chartdeck-dev-paperclip`.
2. Push this folder's contents to it:
   ```bash
   cd D:/Claulde/Projects/chartdeck-dev-paperclip
   git init && git add -A && git commit -m "Initial chartdeck-dev company"
   gh repo create chartdeck-dev-paperclip --private --source=. --remote=origin --push
   ```
3. In Paperclip → Org Chart → Import → **GitHub repo** → paste the URL.
4. Target: **Create new company** (or merge into the existing one that already has a CEO entry).

### Path B — Paperclip-native zip

1. In Paperclip, export the current (empty / CEO-only) company once.
2. Inspect the exported zip's exact file/folder layout.
3. Adjust the file names/extensions in this folder to match (Paperclip's schema is likely close to but may not exactly match `company.json` + `agents/<id>/agent.json` + `instructions.md` as used here).
4. Re-zip using Paperclip's own format (or write a script that mirrors the exporter).

## Layout

```
chartdeck-dev-paperclip/
├── company.json              # top-level manifest (packages, env files, agent refs)
├── environments/
│   └── default.env           # shared env vars (placeholders for secrets)
├── agents/
│   ├── ceo/                  # Hermes — Claude Opus, orchestrator
│   │   ├── agent.json
│   │   └── instructions.md
│   ├── coding/               # Codex — GPT-5, TDD code writer
│   │   ├── agent.json
│   │   └── instructions.md
│   ├── design/               # Google — Gemini 2.5 Pro, multimodal designer
│   │   ├── agent.json
│   │   └── instructions.md
│   └── code-review/          # Claude — Sonnet, principal reviewer
│       ├── agent.json
│       └── instructions.md
└── README.md
```

## After import — finishing steps

1. **Set secrets** via Paperclip's secret store (not committed to this repo):
   - `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `GITHUB_TOKEN`
   - `TELEGRAM_BOT_TOKEN`, `TELEGRAM_ALLOWED_USERS` (provision via @BotFather first)
   - SSH private key file at `/secrets/chartdeck_paperclip_id_ed25519` (matching the deploy key already on the VPS)
2. **Verify dispatch wiring** — Hermes' `dispatch_to: ["coding", "design", "code-review"]` permission must match the actual agent IDs after import. If Paperclip renamed on collision (existing "CEO" entry), edit Hermes' permissions accordingly.
3. **Owner Telegram handshake** — message the chartdeck bot once with `/start`; Hermes' heartbeat will pick you up as the allowed user.
4. **Smoke** — `/status` via Telegram should return queue depth + branch state.

## Field reference (what each agent.json field controls)

| Field | Meaning |
|---|---|
| `command` | The CLI to invoke per dispatch |
| `model` | Specific model snapshot |
| `thinking_effort` | `low` / `medium` / `high` — passed to the underlying CLI |
| `extra_args` | Appended after `command` on each invocation |
| `env` | Names of env vars from the shared `default.env` (resolved at run) |
| `timeout_sec` | Hard wall-clock cap per dispatch |
| `interrupt_grace_sec` | Time given after SIGTERM before SIGKILL |
| `run_policy.heartbeat_interval_sec` | 0 = no heartbeat; >0 = wake every N seconds |
| `run_policy.resume_on` | What re-wakes the agent (dispatch / message / etc.) |
| `permissions.filesystem.read/write/deny_write` | Glob patterns — Paperclip enforces at FS level |
| `permissions.shell.allow/deny/require_owner_confirmation_for` | Command allowlist + denylist |
| `permissions.network.allow` | URL prefix allowlist |
| `permissions.dispatch_to` | Which other agents this one can spawn |

## Dispatch flow (the production pattern)

```
Owner → Telegram → Hermes
Hermes → brainstorm → spec → Owner approves
Hermes → plan → Owner approves
Hermes → loop:
  Hermes dispatches Codex on one task → Codex reports DONE + commit SHA
  Hermes dispatches Claude for spec review → ✅ or ❌
  Hermes dispatches Codex (fix) if ❌
  Hermes dispatches Claude for code-quality review → ✅ or ❌
  Hermes dispatches Codex (fix) if ❌
Hermes merges feature branch → master
Hermes runs deploy/update.sh on VPS (after owner confirmation)
Hermes reports "shipped" on Telegram
```

Google fires in parallel only on design-flagged tasks; its output flows through Hermes to Codex.
