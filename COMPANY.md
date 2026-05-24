---
name: chartdeck-dev
slug: chartdeck-dev
description: Multi-agent dev shop for chartdeck — a free, self-hosted multi-pane live charting + alerts platform. Hermes orchestrates; Codex writes code; Google designs UI; Claude reviews. Single human owner. Production target VPS 2.24.192.16:8091 and (long-term) M5 dashboard fold-in at dashboard.inlakesh.tech.
includes:
  - agents/ceo/AGENTS.md
  - agents/coding/AGENTS.md
  - agents/design/AGENTS.md
  - agents/code-review/AGENTS.md
---

# chartdeck-dev

## Mission

Ship and operate **chartdeck**, a free, self-hosted multi-pane live charting web app — a TradingView alternative for self-directed traders. Server-side indicators, always-on alerts (Telegram + webhook), no SaaS, single-user, accessed via SSH tunnel today, eventually folded into the M5 dashboard at `dashboard.inlakesh.tech`.

## Current state

- **Phase 1** (multi-pane live charting) — shipped, deployed.
- **Phase 2a** (server-side indicators + multi-timeframe) — shipped, deployed.
- **Phase 2b** (alerts + Telegram + webhook delivery) — shipped, deployed (Telegram bot provisioning pending owner).
- **120 tests passing** on `master`. Live at `https://github.com/nucleusjay/chartdeck` and `2.24.192.16:8091`.

## Org structure

```
Hermes (CEO, Chief Executive Agent) — orchestration, delegation
├── Codex (Coding, Senior Implementation Engineer) — TDD code writer
├── Google (Design, Design Lead) — UI/UX, screenshots, mockups
└── Claude (Code Review, Principal Code Reviewer) — spec compliance + code quality
```

Hermes alone speaks to the owner (via Telegram). Specialists are dispatched per task and exit when done.

## Operating constraints

- All shell commands prefixed with `rtk` (Rust Token Killer — token-optimized CLI proxy per project convention).
- `must_be_localhost` validator in `chartdeck/config.py` is load-bearing — production binds to `127.0.0.1:8091` only.
- Single user. Multi-tenancy, public exposure, and auth are explicit non-goals until the M5 fold-in.
- One Codex task per file at a time. Never push to `master` without owner confirmation.
- Every coding task goes through Claude's two-stage review (spec compliance → code quality) before being declared done.

## Roadmap (next, in priority order)

1. Pre-prod hardening punch list (webhook URL log scrubbing, `eng._telegram_*` private access, `telegram_chat_ids` parse warnings, `update_rule` non-rekey test, BingX symbol fix).
2. Telegram bot provisioning via `@BotFather` + `/test` smoke.
3. Plug in the 17 Gold-Star indicator ports from `/root/workspace/quant_backtest/indicators.py`.
4. Phase 2c — event-pattern alerts (new FVG, RSI divergence).
5. BingX provider made real (key + symbol format).
6. Alert history + audit log.
7. M5 dashboard fold-in (Traefik, basic-auth, multi-user safety).
8. Phase 3 — backtest overlay integration.
9. Phase 4 — AI agent control surface.

## References

- Specs and plans live in `docs/superpowers/specs/` and `docs/superpowers/plans/` inside the chartdeck repo.
- Latest design doc: `2026-05-23-chartdeck-phase2b-design.md`.
- Latest plan: `2026-05-23-chartdeck-phase2b.md`.
