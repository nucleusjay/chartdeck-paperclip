---
schema: agentcompanies/v1
kind: company
name: chartdeck-dev
slug: chartdeck-dev
description: Multi-agent dev shop for chartdeck — a free, self-hosted multi-pane live charting + alerts platform. Hermes orchestrates; Codex writes code; Google designs UI; Claude reviews. Single human owner. Production target VPS 2.24.192.16:8091 and (long-term) M5 dashboard fold-in at dashboard.inlakesh.tech.
version: 1.0.0
license: MIT
authors:
  - name: nucleusjay
goals:
  - Ship chartdeck features safely on a tight single-owner loop
  - Keep the spec → plan → code → review → deploy pipeline atomic and observable
  - Reuse the Claude Pro subscription + Gemini free tier to keep marginal model cost near zero
requirements:
  secrets:
    - ANTHROPIC_API_KEY
    - OPENAI_API_KEY
    - GEMINI_API_KEY
    - GITHUB_TOKEN
    - TELEGRAM_BOT_TOKEN
    - TELEGRAM_ALLOWED_USERS
    - SSH_PRIVATE_KEY_PATH
---

# chartdeck-dev

Multi-agent dev shop for [chartdeck](https://github.com/nucleusjay/chartdeck). Hermes (CEO) orchestrates the brainstorm → spec → plan → dispatch → review → deploy loop. Codex implements with TDD discipline, Google handles UI design, Claude runs the two-stage review gate. The single human owner approves at each transition and ships via Telegram.

Production target: VPS `2.24.192.16:8091`. Long-term: fold into `dashboard.inlakesh.tech`.
