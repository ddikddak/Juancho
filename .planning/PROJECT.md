# Juancho

## What This Is

An autonomous AI developer built on OpenClaw. Juancho lives on a remote server, communicates via Telegram, and builds software end-to-end using structured GSD methodology — research, plan, build, test, iterate. It takes a feature request and delivers working, tested, documented code with minimal human intervention, executing proactively via heartbeat cron with configurable autonomy.

## Core Value

An AI developer that follows real engineering discipline — it never just dumps code. It researches before building, builds in phases, tests each step, documents everything, and delivers working software through a structured, verifiable process.

## Requirements

### Validated

- ✓ Rewrite OpenClaw system prompts and agent instructions to be coding-focused — v1.0
- ✓ Rewrite onboarding flow for developer context (not general assistant) — v1.0
- ✓ Integrate full GSD pipeline into agent workflow (.planning/, phases, roadmaps) — v1.0
- ✓ Intensive project definition stage at task start (GSD-style questioning) — v1.0
- ✓ Phase-based execution: research → plan → build → test per phase — v1.0
- ✓ Configurable autonomy levels (user decides how much Juancho asks vs. decides) — v1.0
- ✓ Git workflow integration (commits per phase, branches, pull requests) — v1.0
- ✓ QA testing capabilities (run tests, verify builds, validate output) — v1.0
- ✓ Browser use on VPS (virtual display/headless Chrome for research, testing, scraping) — v1.0 (inherited from OpenClaw, verified)
- ✓ Full documentation generation: GSD planning docs per project — v1.0
- ✓ Full documentation generation: code docs (READMEs, API docs, inline comments) — v1.0
- ✓ Full documentation generation: work logs (decisions, reasoning, what was done) — v1.0
- ✓ Telegram as primary interaction interface — v1.0 (inherited from OpenClaw, verified)
- ✓ Remote server deployment on VPS with headless browser infrastructure — v1.0 (inherited from OpenClaw, verified)
- ✓ Multi-provider AI support (keep OpenClaw's existing provider flexibility) — v1.0 (inherited from OpenClaw, verified)

### Active

(None yet — define in next milestone)

### Out of Scope

- General assistant/chatbot behavior — Juancho is a developer, period
- Multi-user support — personal use only, single operator
- Custom AI model training — uses existing providers as-is
- Mobile app — Telegram is the interface
- Rebuilding OpenClaw features from scratch — leverage what exists, reconfigure it

## Context

- OpenClaw is an active open-source AI assistant platform (TypeScript, 191K+ stars, very active development with 400+ branches)
- Fork exists as ddikddak/Juancho on GitHub, cloned to /Users/didac/Juancho
- Upstream tracking configured: origin → ddikddak/Juancho, upstream → openclaw/openclaw
- OpenClaw already has: Telegram integration, browser use (Stagehand/Puppeteer), multi-provider AI (Claude, OpenAI, Gemini, Ollama), git capabilities
- GSD methodology reference available at ~/.claude/get-shit-done/ with full pipeline templates
- v1.0 shipped: 4 phases, 10 plans, 22 commits, 29 files, 5,866 lines of workspace configuration
- All customization via workspace files only (~/.openclaw/workspace-juancho/) — zero src/ modifications
- Workspace contains: 4 bootstrap files (SOUL.md, AGENTS.md, TOOLS.md, IDENTITY.md), 11 skills, 2 templates, 1 heartbeat config

## Constraints

- **Foundation**: Built on OpenClaw fork — extend and reconfigure, don't rewrite from scratch
- **Deployment**: Self-hosted remote VPS with headless browser support (Xvfb or equivalent)
- **Interface**: Telegram via OpenClaw's existing integration
- **AI Models**: Multi-provider (whatever OpenClaw supports), default to Claude for reasoning-heavy tasks
- **Sync**: Must stay syncable with upstream OpenClaw to pull improvements
- **Workspace limits**: Skills under 350 lines / 15k chars, bootstrap files under 20k chars total

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Fork OpenClaw rather than build from scratch | Massive existing capability (browser, Telegram, multi-AI) — reconfigure, don't rebuild | ✓ Good — zero src/ changes, full capability inherited |
| Telegram as primary interface | Already supported in OpenClaw, always accessible, no custom UI needed | ✓ Good — 11 commands registered via nativeCommand |
| Full GSD pipeline integration | User's core pain point is AI tools lacking discipline — GSD provides the structured workflow | ✓ Good — 8 skills implementing full pipeline |
| Configurable autonomy over fixed behavior | User wants intensive questioning at project start but minimal interruption during execution | ✓ Good — 3 levels (minimal/moderate/maximum) in config.json |
| Headless browser on VPS | Browser use is critical for research, testing, and context gathering — needs virtual display on server | ✓ Good — Playwright + Xvfb verified in OpenClaw |
| Workspace-only customization | Maintain upstream sync by never modifying OpenClaw source code | ✓ Good — all config in ~/.openclaw/workspace-juancho/ |
| Skills reference GSD templates by path | Avoid content duplication, stay under size limits | ✓ Good — all skills under limits |
| Heartbeat defaults: 30m interval, UTC timezone | Reasonable defaults, user-configurable in openclaw.json | ✓ Good — configurable via PROA-03 |

---
*Last updated: 2026-02-15 after v1.0 milestone*
