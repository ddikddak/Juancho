# Project Research Summary

**Project:** Juancho - Autonomous AI Developer Agent
**Domain:** Autonomous AI coding agent built on OpenClaw fork
**Researched:** 2026-02-14
**Confidence:** HIGH

## Executive Summary

Juancho is built by extending OpenClaw (a TypeScript-based AI assistant platform) with GSD methodology to create a disciplined, autonomous coding agent. The key insight from research is that OpenClaw already provides 90% of the required infrastructure—multi-provider AI, Telegram integration, browser automation, testing framework, and session management. What's needed is not a rewrite, but a discipline layer: rewritten system prompts, structured workspace files (`.planning/` directory), git automation, and GSD-specific tools for phase-based execution.

The recommended approach is workspace-driven behavior through prompt engineering and file-based context rather than forking core Gateway logic. Juancho will be configured as a specialized agent with developer-focused prompts (SOUL.md, AGENTS.md), extended with simple-git for version control automation, and deployed on VPS with Xvfb for headless browser reliability. The primary interface is Telegram, leveraging OpenClaw's existing Grammy integration with message chunking strategies for long outputs.

Critical risks center on three areas: (1) fork drift from upstream OpenClaw requiring strict upstream sync discipline, (2) security vulnerabilities from autonomous code execution demanding microVM sandboxing, and (3) cascading AI errors requiring checkpoint validation after each autonomous phase. The GSD methodology itself mitigates the fourth major risk—silent AI failures—by enforcing test-driven development and phase-by-phase verification gates.

## Key Findings

### Recommended Stack

OpenClaw provides a battle-tested foundation with Node.js 22 LTS (supported until 2027), TypeScript 5.9, Vitest for testing, and the pi-coding-agent toolkit for LLM orchestration. The browser automation layer uses Playwright 1.58 (cross-browser, auto-wait patterns) and Telegram integration via Grammy 1.40 with full bot handlers.

**Core technologies (inherited from OpenClaw):**
- **Node.js 22.12+**: Active LTS runtime, TypeScript-native, ESM by default
- **pi-coding-agent 0.52**: Coding-specific agent capabilities from badlogic/pi-mono (~8.9k stars)
- **Playwright 1.58**: Headless browser automation with multi-browser support and robust network interception
- **Grammy 1.40**: TypeScript-first Telegram framework with async/await, webhook support
- **Vitest 4.0**: 10-20x faster than Jest, built-in TypeScript/ESM support
- **SQLite + sqlite-vec 0.1**: Vector storage for memory search and context recall

**New additions for Juancho:**
- **simple-git 3.30**: Git operations (7.9M weekly downloads, shell-based, widely battle-tested)
- **Xvfb (system)**: Virtual X11 display for headless browser on VPS (industry standard)
- **GSD tools**: Custom TypeScript tools for workflow orchestration (gsd_orchestrator, gsd_researcher, gsd_verifier)

### Expected Features

**Must have (table stakes):**
- Multi-file code generation and editing (OpenClaw ✅ via pi-coding-agent)
- Terminal/CLI execution with PTY support (OpenClaw ✅ has bash tool)
- Git integration for commits, branches, PRs (OpenClaw ✅ partial, needs convention enforcement)
- Testing execution and validation (OpenClaw ✅ via Vitest)
- Repository awareness and context management (OpenClaw ⚠️ partial, needs scoping)
- Multi-provider AI support (OpenClaw ✅ Anthropic/OpenAI/Gemini/Ollama)
- Error handling and debugging (OpenClaw ⚠️ needs verification)
- Background/async task execution (OpenClaw ✅ full support)

**Should have (differentiators):**
- **GSD methodology integration**: Phase-based execution (research → plan → build → test) prevents monolithic failures
- **Intensive project definition**: Front-loads requirements to avoid mid-project confusion
- **Built-in testing discipline**: Test-first enforcement catches AI's 1.7x higher bug rate early
- **Browser research capability**: Agent researches libraries/docs before coding (OpenClaw ✅ has browser)
- **Documentation generation**: Auto-generates planning docs, work logs, code docs
- **Configurable autonomy levels**: User controls how much agent asks vs. decides
- **Telegram as primary interface**: Always-on, mobile-accessible remote developer (OpenClaw ✅)
- **Auto-commit with conventions**: Professional git hygiene (OpenClaw ⚠️ partial)
- **Work logging and reasoning**: Documents decisions, rationale, tradeoffs

**Defer (v2+):**
- Voice/audio interaction (Telegram text sufficient for MVP)
- Multi-project management (start single-project)
- Advanced git operations (rebase, cherry-pick beyond basic commit/branch/PR)
- Deployment automation (CI/CD integration valuable but not MVP-critical)
- Code review as separate mode (can enhance after core workflow proven)

### Architecture Approach

OpenClaw uses a Gateway-centric architecture with WebSocket control plane orchestrating agents, channels (Telegram/Slack), tools, and sessions. The Pi agent runtime executes embedded in Gateway, with system prompts built dynamically from workspace files (SOUL.md, TOOLS.md, AGENTS.md) plus agent configuration. This architecture is already designed for workflow orchestration—it just needs new prompts, workspace structure, and GSD tooling.

**Major components:**
1. **Gateway** (unchanged) — Control plane, WebSocket server, session orchestration, channel routing
2. **Agent Runtime** (extended) — Rewritten system prompts for developer persona, GSD-aware instructions
3. **Workspace** (extended) — Add `.planning/` directory (research/, requirements/, roadmap/, phases/) with GSD state files
4. **Tools** (new GSD tools) — gsd_orchestrator, gsd_researcher, gsd_requirements, gsd_phase_executor, gsd_verifier
5. **Telegram Channel** (configured) — Route to Juancho agent, implement chunking/throttling for long outputs
6. **Git Integration** (new) — simple-git wrapper tools for commit/branch/PR automation with convention enforcement

**Key architectural patterns to follow:**
- **Workspace-driven behavior**: Agent behavior controlled by workspace files, not hardcoded logic
- **Tool-based workflows**: Implement GSD pipeline as tools agents can call
- **Session-based state**: Store state in `.planning/state.json`, not in-memory
- **Layered prompts**: System prompt built from hardcoded sections + config + workspace + runtime context

### Critical Pitfalls

1. **Fork drift**: Customizations diverge from upstream OpenClaw, making security patches impossible to apply after 12-18 months. Prevent by keeping `main` as clean mirror, all work in feature branches, weekly upstream rebase, modular plugins over core modifications.

2. **Sandbox escape**: AI code executes with full user permissions, enabling data exfiltration or system compromise via prompt injection. Prevent with microVM isolation (Firecracker/gVisor), network egress controls, filesystem restrictions, never auto-approve destructive commands.

3. **Silent failures**: AI generates code that runs without crashing but produces fake output, removes safety checks, or silently fails core functionality. Prevent with intent-driven testing, continuous validation, property-based testing, human validation gates for critical paths.

4. **Compounding errors**: Small mistake in phase 1 propagates exponentially through autonomous steps. Prevent with checkpoint validation after EACH phase, small increments (30min chunks), rollback on first failure, max iteration limits.

5. **Headless browser instability on VPS**: Chromium throttles to 2 fps when SSH disconnects, memory leaks from zombie processes. Prevent with Xvfb virtual display, old headless mode, process cleanup cron, timeout everything, tmux/screen for session persistence.

## Implications for Roadmap

Based on research, the suggested phase structure balances foundation setup (security, infrastructure) with incremental GSD feature delivery. The fork drift risk demands immediate upstream sync discipline, while security sandboxing is non-negotiable before any autonomous execution.

### Phase 1: Foundation & Fork Strategy
**Rationale:** Must establish upstream sync discipline and security infrastructure before any coding. Fork drift and sandbox escape are both critical pitfalls that require architectural decisions upfront.

**Delivers:**
- Clean `main` branch mirroring upstream, all work in `feat/juancho-*` branches
- Automated daily upstream sync via GitHub Actions
- Developer-focused system prompts (SOUL.md, AGENTS.md)
- Juancho agent configuration in OpenClaw config
- VPS setup with Xvfb, Node 22 LTS, system dependencies

**Addresses:**
- Table stakes: Repository awareness foundation, multi-provider AI (inherited)
- Differentiator: Remote VPS deployment
- Pitfall avoidance: Fork drift (#1), headless browser instability (#5)

**Avoids:**
- Fork drift by establishing sync workflow from day 1
- VPS browser issues by configuring Xvfb before deployment

### Phase 2: Security & Git Integration
**Rationale:** Autonomous code execution requires sandboxing before any GSD automation. Git automation is foundation for phase-based commits.

**Delivers:**
- MicroVM/gVisor sandbox for code execution
- Network egress controls, filesystem restrictions
- simple-git integration with wrapper tools
- Git convention enforcement (semantic commits, branch protection)
- Human approval gates for destructive operations

**Addresses:**
- Table stakes: Git integration with conventions
- Differentiator: Auto-commit with conventions
- Pitfall avoidance: Sandbox escape (#2), git workflow chaos (#8 from PITFALLS.md)

**Avoids:**
- Security vulnerabilities from uncontrolled code execution
- Broken production from auto-pushed bad code

### Phase 3: GSD Workspace & Tools
**Rationale:** `.planning/` directory structure and GSD orchestration tools enable phase-based execution. This is the core differentiator.

**Delivers:**
- `.planning/` directory structure (research/, requirements/, roadmap/, phases/)
- GSD orchestrator tool (manages phase state transitions)
- Planning state persistence (`.planning/state.json`)
- Workspace templates for new projects
- Phase-specific prompts (research, plan, build, test)

**Addresses:**
- Differentiator: GSD methodology integration, phase-based execution
- Pitfall avoidance: Context degradation (#7) via persistent context store

**Uses:**
- Workspace-driven behavior pattern (from ARCHITECTURE.md)
- Tool-based workflows pattern
- Session-based state pattern

**Implements:**
- Workspace extension architecture component
- GSD tools component

### Phase 4: Testing Discipline & Validation
**Rationale:** AI code has 1.7x more bugs; testing must be enforced before autonomous coding. Addresses silent failures and compounding errors.

**Delivers:**
- Test-first enforcement in build phase prompts
- Checkpoint validation after each autonomous phase
- Intent-driven test generation
- Property-based testing for invariants
- Coverage tracking and reporting
- Human validation gates for critical paths

**Addresses:**
- Table stakes: Testing execution
- Differentiator: Built-in testing discipline
- Pitfall avoidance: Silent failures (#3), compounding errors (#4)

**Avoids:**
- Cascading failures from unvalidated phases
- Fake output passing tests but failing in production

### Phase 5: Telegram Interface & Documentation
**Rationale:** Telegram is primary interface; must handle long outputs gracefully. Documentation generation completes GSD pipeline.

**Delivers:**
- Message chunking for long code/diffs (4096 char limit)
- File upload strategy for large content
- Rate limiting/throttling to avoid Telegram API blocks
- Inline keyboards for approvals
- Documentation generation (planning docs, code docs, work logs)
- Summarization prompts for lengthy output

**Addresses:**
- Table stakes: Multi-file operations display
- Differentiator: Documentation generation, work logging, Telegram interface
- Pitfall avoidance: Telegram interface limitations (#6)

**Uses:**
- Grammy integration (inherited)
- Documentation generation architecture component

### Phase 6: Autonomy & Iteration
**Rationale:** Core pipeline working, now add configurable autonomy and refinement.

**Delivers:**
- Configurable autonomy levels (intensive → hands-off)
- Browser research integration into research phase
- Parallel task execution (multiple worktrees)
- Work reasoning transparency
- Loop safety (max iterations, timeouts)
- Monitoring and health checks

**Addresses:**
- Differentiator: Configurable autonomy, browser research, parallel tasks, work logging
- Pitfall avoidance: Infinite loops (#9), over-engineering (#10), blind trust (#11), god agent (#12)

### Phase Ordering Rationale

- **Foundation first (Phase 1)** because fork drift compounds daily; upstream sync must be established before customizations accumulate
- **Security second (Phase 2)** because autonomous execution without sandboxing is existential risk; non-negotiable before GSD automation
- **GSD workspace third (Phase 3)** because phase structure is foundation for remaining features; tools depend on workspace state
- **Testing fourth (Phase 4)** because validation gates prevent cascading errors; must be in place before heavy autonomous use
- **Interface fifth (Phase 5)** because Telegram limitations affect UX but not core functionality; can iterate after pipeline works
- **Autonomy last (Phase 6)** because refinement and optimization come after core workflow proven; avoids premature over-engineering

**Architecture dependency order:**
1. Gateway + Agent Runtime (inherited, configure)
2. Workspace extension (enable `.planning/` state)
3. GSD Tools (orchestrate workflow)
4. Git Integration (enable phase commits)
5. Telegram optimization (improve UX)
6. Monitoring/safety (production hardening)

**Pitfall avoidance order:**
1. Fork drift (Phase 1) — daily upstream sync
2. Sandbox escape (Phase 2) — microVM isolation
3. Silent failures (Phase 4) — test enforcement
4. Compounding errors (Phase 4) — checkpoint validation
5. Headless browser (Phase 1) — Xvfb setup
6. Context degradation (Phase 3) — persistent state
7. Git chaos (Phase 2) — approval gates
8. Telegram limits (Phase 5) — chunking strategy
9. Infinite loops (Phase 6) — timeouts/limits

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (Security):** MicroVM implementation details, gVisor integration with Node.js, seccomp profiles for Playwright
- **Phase 4 (Testing):** Property-based testing frameworks for TypeScript, intent-driven test generation patterns, coverage thresholds

Phases with standard patterns (skip research-phase):
- **Phase 1 (Foundation):** Git fork management well-documented, Xvfb setup standard, OpenClaw config straightforward
- **Phase 3 (GSD Workspace):** File-based state management standard, tool development documented in OpenClaw plugin SDK
- **Phase 5 (Telegram):** Grammy API well-documented, message chunking patterns established
- **Phase 6 (Autonomy):** Timeout patterns standard, configuration management well-understood

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Verified from package.json, official Node.js releases, npm stats, OpenClaw codebase inspection |
| Features | HIGH | Verified from OpenClaw source code, pi-coding-agent README, established 2026 AI agent landscape |
| Architecture | HIGH | Direct code inspection of OpenClaw codebase, architectural patterns evident in Gateway/agent/workspace implementations |
| Pitfalls | HIGH | 2026 sources from authoritative channels (NVIDIA, OWASP, IEEE Spectrum, GitHub official discussions, Puppeteer maintainers) |

**Overall confidence:** HIGH

Research is comprehensive and current (all 2026 sources for pitfalls, official documentation for stack/architecture). Stack findings verified via package.json and codebase inspection. Features validated against OpenClaw capabilities and domain research. Architecture verified through source code analysis. Pitfalls have authoritative sources (official docs, academic papers, OWASP standards).

### Gaps to Address

**During Phase 2 (Security) planning:**
- Specific microVM implementation choice (Firecracker vs gVisor vs Kata) depends on VPS provider constraints
- Network egress whitelist needs refinement based on actual package registries used
- Filesystem permission boundaries need OpenClaw workspace directory structure mapping

**During Phase 4 (Testing) planning:**
- Test coverage thresholds need calibration based on codebase characteristics
- Property-based testing framework selection (fast-check vs testcheck-js) needs evaluation
- Integration test strategy for GSD orchestration flow needs design

**During Phase 6 (Autonomy) planning:**
- Timeout values need empirical calibration based on actual task performance
- Max iteration limits need tuning based on observed AI behavior patterns
- Monitoring alert thresholds need baseline establishment

**Cross-phase validation:**
- Fork strategy effectiveness needs weekly review against upstream changes
- Sandbox overhead impact on autonomous execution speed needs measurement
- Telegram UX patterns need user feedback iteration

## Sources

### Primary (HIGH confidence)

**Stack:**
- OpenClaw codebase at /Users/didac/Juancho/ (package.json, src/ inspection)
- Node.js 22 LTS Release: https://nodejs.org/en/blog/release/v22.22.0
- pi-mono GitHub: https://github.com/badlogic/pi-mono
- Playwright Documentation: https://playwright.dev/
- Grammy Telegram Framework: https://grammy.dev/
- simple-git npm stats: https://npmtrends.com/isomorphic-git-vs-nodegit-vs-simple-git

**Architecture:**
- OpenClaw system prompt builder: src/agents/system-prompt.ts
- OpenClaw workspace loading: src/agents/workspace.ts
- OpenClaw agent scope resolution: src/agents/agent-scope.ts
- Pi agent runtime: src/agents/pi-embedded-runner/run.ts
- Configuration schema: src/config/types.ts

**Pitfalls (Critical):**
- NVIDIA Sandboxing Guidance: https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/
- OWASP AI Agent Security Top 10 2026: https://medium.com/@oracle_43885/owasps-ai-agent-security-top-10-agent-security-risks-2026-fc5c435e86eb
- Northflank Sandbox Guide 2026: https://northflank.com/blog/how-to-sandbox-ai-agents
- IEEE Spectrum AI Coding Degrades: https://spectrum.ieee.org/ai-coding-degrades
- VentureBeat AI Agents Production-Readiness: https://venturebeat.com/ai/why-ai-coding-agents-arent-production-ready-brittle-context-windows-broken
- Puppeteer Issue #11058 (VPS Performance): https://github.com/puppeteer/puppeteer/issues/11058
- Telegram Limits Official: https://limits.tginfo.me/en

### Secondary (MEDIUM confidence)

**Features:**
- Faros AI Best Coding Agents 2026: https://www.faros.ai/blog/best-ai-coding-agents-2026
- GSD Framework Article: https://pasqualepillitteri.it/en/news/169/gsd-framework-claude-code-ai-development
- I Tested GSD Claude Code: https://medium.com/@joe.njenga/i-tested-gsd-claude-code-meta-prompting-that-ships-faster-no-agile-bs-ca62aff18c04
- 9 Critical Failure Patterns (DAPLab): https://daplab.cs.columbia.edu/general/2026/01/08/9-critical-failure-patterns-of-coding-agents.html
- CodeRabbit AI vs Human Report: https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report

**Pitfalls (Moderate/Minor):**
- The New Stack: Beating Context Rot: https://thenewstack.io/beating-the-rot-and-getting-stuff-done/
- GitHub Community Best Practices: https://github.com/orgs/community/discussions/182197
- Addy Osmani Self-Improving Agents: https://addyosmani.com/blog/self-improving-agents/
- Continuous Validation: https://testkube.io/blog/continuous-validation-ai-coding
- Preset.io Fork Drift: https://preset.io/blog/stop-forking-around-the-hidden-dangers-of-fork-drift-in-open-source-adoption/
- Gofore Git Forking Best Practices: https://gofore.com/en/best-practices-for-forking-a-git-repo/

### Tertiary (context validation)

**Stack comparisons:**
- Playwright vs Puppeteer (BrowserStack): https://www.browserstack.com/guide/playwright-vs-puppeteer
- Vitest vs Jest comparison: https://howtotestfrontend.com/resources/vitest-vs-jest-which-to-pick
- Xvfb Chrome Setup: https://www.mattzeunert.com/2018/07/21/running-headful-chrome-on-ubuntu-server.html

**Feature landscape:**
- StackOne AI Agent Tools 2026: https://www.stackone.com/blog/ai-agent-tools-landscape-2026
- AI Copilots vs Agents 2026: https://skywork.ai/blog/ai-copilots-vs-ai-agents-2026-comparison/
- Anthropic 8 Agentic Coding Trends: https://tessl.io/blog/8-trends-shaping-software-engineering-in-2026-according-to-anthropics-agentic-coding-report/

---
*Research completed: 2026-02-14*
*Ready for roadmap: yes*
