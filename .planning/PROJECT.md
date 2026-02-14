# Juancho

## What This Is

An autonomous AI developer employee built on OpenClaw. Juancho lives on a remote server, communicates via Telegram, and builds software end-to-end using structured GSD methodology — research, plan, build, test, iterate. It's a personal dev tool that takes a feature request and delivers working, tested, documented code with minimal human intervention.

## Core Value

An AI developer that follows real engineering discipline — it never just dumps code. It researches before building, builds in phases, tests each step, documents everything, and delivers working software through a structured, verifiable process.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Rewrite OpenClaw system prompts and agent instructions to be coding-focused
- [ ] Rewrite onboarding flow for developer context (not general assistant)
- [ ] Integrate full GSD pipeline into agent workflow (.planning/, phases, roadmaps)
- [ ] Intensive project definition stage at task start (GSD-style questioning)
- [ ] Phase-based execution: research → plan → build → test per phase
- [ ] Configurable autonomy levels (user decides how much Juancho asks vs. decides)
- [ ] Git workflow integration (commits per phase, branches, pull requests)
- [ ] QA testing capabilities (run tests, verify builds, validate output)
- [ ] Browser use on VPS (virtual display/headless Chrome for research, testing, scraping)
- [ ] Full documentation generation: GSD planning docs per project
- [ ] Full documentation generation: code docs (READMEs, API docs, inline comments)
- [ ] Full documentation generation: work logs (decisions, reasoning, what was done)
- [ ] Telegram as primary interaction interface
- [ ] Remote server deployment on VPS with headless browser infrastructure
- [ ] Multi-provider AI support (keep OpenClaw's existing provider flexibility)

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
- OpenClaw uses namespaced branch conventions (feat/, fix/, docs/, security/, codex/) and active refactoring

## Constraints

- **Foundation**: Built on OpenClaw fork — extend and reconfigure, don't rewrite from scratch
- **Deployment**: Self-hosted remote VPS with headless browser support (Xvfb or equivalent)
- **Interface**: Telegram via OpenClaw's existing integration
- **AI Models**: Multi-provider (whatever OpenClaw supports), default to Claude for reasoning-heavy tasks
- **Sync**: Must stay syncable with upstream OpenClaw to pull improvements

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Fork OpenClaw rather than build from scratch | Massive existing capability (browser, Telegram, multi-AI) — reconfigure, don't rebuild | — Pending |
| Telegram as primary interface | Already supported in OpenClaw, always accessible, no custom UI needed | — Pending |
| Full GSD pipeline integration | User's core pain point is AI tools lacking discipline — GSD provides the structured workflow | — Pending |
| Configurable autonomy over fixed behavior | User wants intensive questioning at project start but minimal interruption during execution | — Pending |
| Headless browser on VPS | Browser use is critical for research, testing, and context gathering — needs virtual display on server | — Pending |

---
*Last updated: 2026-02-14 after initialization*
