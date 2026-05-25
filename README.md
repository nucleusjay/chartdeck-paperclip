# chartdeck-dev — Paperclip Company Package

Importable Paperclip company package for [chartdeck](https://github.com/nucleusjay/chartdeck), conforming to the [Agent Companies specification](https://agentcompanies.io/specification) (`agentcompanies/v1`) with the Paperclip vendor extension (`paperclip/v1`) for adapter, runtime, and permissions configuration.

## Layout (Agent Companies v1 + Paperclip vendor extension)

```
chartdeck-dev-paperclip/
├── COMPANY.md                        # canonical company root (frontmatter + body)
├── README.md                         # this file (import instructions)
├── .paperclip.yaml                   # Paperclip vendor extension: adapters, runtime, permissions, envInputs
├── environments/
│   └── default.env                   # shared environment template (secrets blank)
└── agents/
    ├── ceo/AGENTS.md                 # Hermes — CEO, strategic orchestration, Telegram bridge
    ├── coding/AGENTS.md              # Codex — Senior Implementation Engineer (TDD)
    ├── design/AGENTS.md              # Google — Design Lead (UI/UX, CSS authoring)
    └── code-review/AGENTS.md         # Claude — Principal Code Reviewer (two-stage gate)
```

Each `AGENTS.md` carries the agent's identity, workflow, hard rules, and capabilities directly in the file body (per spec section 8). The companion `.paperclip.yaml` only declares runtime, adapter selection, filesystem/shell permissions, and env inputs — never instructions or identity.

## How to import

1. Open Paperclip at `https://paperclip.inlakesh.tech/`.
2. Authenticate through the Traefik basic-auth pop-up and log in.
3. Select **Org Chart → Import → GitHub repo**.
4. URL: `https://github.com/nucleusjay/chartdeck-paperclip`
5. Target: **Create new company** (collision strategy: rename).
6. **Preview import** → confirm.

## After import — set secrets

In Paperclip's secret store (do **not** commit these to the repository):

- `ANTHROPIC_API_KEY` — Hermes (CEO) + Claude (reviewer). Required as a fallback when the Hermes Anthropic proxy is unreachable or rate-limited.
- `OPENAI_API_KEY` — Codex (coding). Direct API; no Hermes proxy exists for OpenAI.
- `GEMINI_API_KEY` — Google (design). Direct Google AI Studio; free tier is already cheapest.
- `GITHUB_TOKEN` — Hermes (pushes + PRs).
- `TELEGRAM_BOT_TOKEN` — Hermes Telegram bridge.
- `TELEGRAM_ALLOWED_USERS` — Comma-separated Telegram user IDs allowed to message the bridge.
- `SSH_PRIVATE_KEY_PATH` — Path to deploy key inside the agent container, e.g. `/home/paperclip/.ssh/chartdeck_deploy`.

### Auth strategy

| Agent | Model | Auth | Cost |
|---|---|---|---|
| Hermes (CEO) | Claude Opus | Hermes Anthropic proxy → Claude Pro subscription (with `ANTHROPIC_API_KEY` fallback) | $0 marginal when proxy is reachable |
| Claude (reviewer) | Claude Sonnet | Hermes Anthropic proxy → Claude Pro subscription (with `ANTHROPIC_API_KEY` fallback) | $0 marginal when proxy is reachable |
| Codex (coding) | Codex CLI default | Direct OpenAI API via `OPENAI_API_KEY` | Pay-as-you-go |
| Google (design) | Gemini 2.5 Pro | Direct Google AI Studio via `GEMINI_API_KEY` | Free tier |

## Hermes proxy wiring

The two Claude agents (`ceo`, `code-review`) are configured with `ANTHROPIC_BASE_URL=http://host.docker.internal:8645/v1` in their adapter `config.env` block (see `.paperclip.yaml`). This routes the local `claude` CLI through the Hermes Anthropic proxy on the VPS, which translates the OpenAI-compatible request into a Claude Pro OAuth call — using your subscription instead of billing the API key.

**Two prerequisites for this to work on the VPS:**

1. **The Hermes proxy must be reachable from inside the Paperclip server container.** The systemd unit ships bound to `127.0.0.1:8645`, which is unreachable from a sibling container. Either:
   - Rebind the proxy to the Docker bridge interface (e.g. `--host 172.17.0.1`) by editing `/root/.config/systemd/user/hermes-proxy-anthropic.service` and restarting, **or**
   - Run the Paperclip server with `network_mode: host` in `docker/docker-compose.yml`, **or**
   - Add `extra_hosts: ["host.docker.internal:host-gateway"]` to the Paperclip server service and rebind the proxy to `--host 0.0.0.0` (firewall the port).

2. **Verify post-import:** `docker exec -it <paperclip-server> curl -sf http://host.docker.internal:8645/v1/models` should return a model list. If it 404s/refuses, the agents will fall back to `ANTHROPIC_API_KEY` (direct API, billed).

If you don't want to wire the proxy yet, remove the `env: ANTHROPIC_BASE_URL: …` line from the `ceo` and `code-review` adapter configs in `.paperclip.yaml` and the agents will use direct API from day one.

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
