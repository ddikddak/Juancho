# Technology Stack

**Project:** Juancho - Autonomous AI Developer Agent
**Researched:** 2026-02-14

## Executive Summary

Juancho is built by reconfiguring OpenClaw (an established TypeScript-based AI assistant platform) into a coding-focused autonomous developer. The stack leverages OpenClaw's existing infrastructure (browser automation, Telegram integration, multi-provider AI) while adding GSD workflow integration, enhanced git automation, and headless browser capabilities for VPS deployment.

**Stack Philosophy:** Extend and reconfigure, don't rebuild. OpenClaw provides 90% of the infrastructure; Juancho adds the discipline layer.

---

## Core Foundation (Already Provided by OpenClaw)

### Runtime & Build System

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| Node.js | >=22.12.0 (LTS) | Runtime environment | **INHERITED** - OpenClaw requires this |
| TypeScript | ^5.9.3 | Type-safe development | **INHERITED** - OpenClaw codebase |
| pnpm | 10.23.0 | Package manager | **INHERITED** - OpenClaw standard |
| Vitest | ^4.0.18 | Testing framework | **INHERITED** - Already configured |
| tsdown | ^0.20.3 | TypeScript bundler | **INHERITED** - Build tooling |

**Rationale:** Node.js 22.x is in Active LTS until 2027-04-30, providing long-term stability. OpenClaw already requires >=22.12.0 and has comprehensive TypeScript/Vitest setup. No changes needed.

**Confidence:** HIGH (verified from package.json and official Node.js releases)

### AI & Agent Framework

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| @mariozechner/pi-coding-agent | 0.52.12 | Coding agent core | **INHERITED** - OpenClaw dependency |
| @mariozechner/pi-agent-core | 0.52.12 | Agent runtime | **INHERITED** - OpenClaw dependency |
| @mariozechner/pi-ai | 0.52.12 | Multi-provider AI interface | **INHERITED** - OpenClaw dependency |
| @mariozechner/pi-tui | 0.52.12 | Terminal UI | **INHERITED** - OpenClaw dependency |

**Rationale:** OpenClaw already integrates the pi-coding-agent toolkit from badlogic/pi-mono (~8.9k stars). This provides coding-specific agent capabilities, session management, and unified LLM API support (OpenAI, Anthropic, Google, etc.). Perfect foundation for Juancho's autonomous coding workflows.

**Confidence:** HIGH (verified in package.json and pi-mono GitHub)

**Sources:**
- [pi-mono GitHub repository](https://github.com/badlogic/pi-mono)
- [pi-coding-agent README](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md)

### Browser Automation

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| playwright-core | 1.58.2 | Headless browser automation | **INHERITED** - OpenClaw uses this |

**Rationale:** OpenClaw already uses Playwright for browser automation (via `src/browser/` with extensive CDP integration). Playwright supports cross-browser testing (Chromium, Firefox, WebKit), built-in automatic waits, and robust network interception. While Puppeteer is slightly faster for Chrome-only tasks, Playwright's multi-browser support and modern API make it the better choice for research and testing workflows.

**Why Playwright over Puppeteer:**
- Cross-browser support (useful for testing different environments)
- Automatically waits for elements and network events (reduces flaky tests)
- Better TypeScript support out of the box
- OpenClaw already has extensive Playwright integration

**Confidence:** HIGH (verified in package.json and src/browser/ directory)

**Sources:**
- [Playwright vs Puppeteer comparison](https://www.browserstack.com/guide/playwright-vs-puppeteer)
- [Playwright advantages](https://www.zenrows.com/blog/playwright-vs-puppeteer)

### Telegram Integration

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| grammy | ^1.40.0 | Telegram bot framework | **INHERITED** - OpenClaw uses this |
| @grammyjs/runner | ^2.0.3 | Long polling runner | **INHERITED** - OpenClaw dependency |
| @grammyjs/transformer-throttler | ^1.2.1 | Rate limiting | **INHERITED** - OpenClaw dependency |

**Rationale:** OpenClaw has full Telegram integration in `src/telegram/` with bot handlers, message context, native commands, and webhook support. Grammy is a modern TypeScript-first Telegram bot framework with excellent async/await support. Perfect for developer workflows - supports webhooks for GitHub events, scheduled tasks, and real-time interactions.

**Confidence:** HIGH (verified in package.json and src/telegram/ directory)

**Sources:**
- [Developer's Guide to Building Telegram Bots in 2025](https://stellaray777.medium.com/a-developers-guide-to-building-telegram-bots-in-2025-dbc34cd22337)

---

## New Additions for Juancho

### GSD Workflow Integration

| Component | Approach | Implementation |
|-----------|----------|----------------|
| System prompts | **Reconfigure existing** | Modify `src/agents/system-prompt.ts` to inject GSD methodology instructions |
| Agent instructions | **Extend existing** | Add GSD phases to pi-coding-agent workflow via custom instructions |
| Planning files | **Add file operations** | Create `.planning/` directory structure (research/, phases/, roadmaps/) |
| Phase execution | **Orchestrate existing tools** | Use OpenClaw's existing tool system to coordinate GSD phases |

**Rationale:** OpenClaw already has system prompt architecture (`src/agents/system-prompt.ts`) and tool-based workflows. GSD integration is primarily prompt engineering + file structure conventions, not new infrastructure. The pi-coding-agent toolkit already supports skills and session management, which align with GSD's phase-based approach.

**Implementation Strategy:**
1. Create GSD system prompt templates in `src/agents/gsd-prompts/` (research, plan, build, test phases)
2. Add `.planning/` directory initialization on project start
3. Inject phase-specific instructions via OpenClaw's existing `PromptMode` system
4. Use existing file read/write tools for managing planning documents

**Confidence:** MEDIUM (approach is sound, but requires verification that OpenClaw's prompt system supports this level of customization)

**Sources:**
- [Agentic coding best practices](https://www.teamday.ai/blog/complete-guide-agentic-coding-2026)
- [System prompt engineering patterns](https://www.ibm.com/think/prompt-engineering)

### Git Workflow Automation

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| simple-git | ^3.30.0 | Git operations | Most popular, 7.9M weekly downloads, simple shell-based API |

**Rationale:** OpenClaw does NOT currently have git integration (no simple-git, nodegit, or isomorphic-git in dependencies). We need to add this for commit-per-phase, branch management, and PR creation.

**Why simple-git over alternatives:**
- **simple-git:** 7.9M weekly downloads, shell-based (works everywhere), simple API
- **isomorphic-git:** Pure JS (browser + Node), but slower for large repos, only 628K downloads
- **nodegit:** Native bindings (fast), but only 29K downloads, build complexity, no browser support

For an autonomous coding agent on VPS, simple-git is the clear winner: reliable, widely used, no native compilation, straightforward API.

**Key Operations Needed:**
- Initialize/clone repositories
- Create feature branches (per milestone/phase)
- Stage and commit changes (per phase with GSD-structured messages)
- Create pull requests (via GitHub CLI integration)
- Check git status, diff, log

**Installation:**
```bash
pnpm add simple-git@^3.30.0
```

**Confidence:** HIGH (clear use case, well-established library)

**Sources:**
- [simple-git vs alternatives comparison](https://npm-compare.com/isomorphic-git,nodegit,simple-git)
- [npm trends showing popularity](https://npmtrends.com/isomorphic-git-vs-nodegit-vs-simple-git)

### Headless Browser on VPS

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Xvfb | Latest (apt) | Virtual X11 display server | Industry standard for headless GUI apps on Linux |
| xvfb-run | Latest (apt) | Xvfb wrapper script | Simplifies running commands with virtual display |

**Rationale:** Playwright can run in true headless mode (`--headless=new`), but some sites detect headless browsers or require full browser features. Xvfb provides a virtual display for running "headed" browsers on headless servers.

**VPS Setup (Ubuntu/Debian):**
```bash
# Install Xvfb and dependencies
sudo apt-get update
sudo apt-get install -y xvfb libgbm-dev libasound2

# Install Playwright browsers
npx playwright install --with-deps chromium

# Run with Xvfb wrapper
xvfb-run -a --server-args="-screen 0 1280x800x24 -ac -nolisten tcp -dpi 96 +extension RANDR" \
  node /path/to/juancho/openclaw.mjs gateway
```

**Docker Alternative:**
For containerized deployment, use existing Xvfb+Chrome images:
- `markadams/chromium-xvfb` (popular, well-maintained)
- `socrateslee/xvfb-chrome` (supports non-root users)

**Environment Variables:**
```bash
# For Playwright to use Xvfb display
export DISPLAY=:99
```

**Rationale for Xvfb:**
- Playwright already handles modern headless Chrome well
- Xvfb needed ONLY for sites with anti-bot detection or WebGL/canvas requirements
- Minimal overhead, battle-tested solution
- Works seamlessly with existing Playwright code (no code changes)

**Confidence:** HIGH (standard approach, well-documented)

**Sources:**
- [Xvfb with Chrome setup guide](https://www.mattzeunert.com/2018/07/21/running-headful-chrome-on-ubuntu-server.html)
- [Docker Chromium Xvfb images](https://github.com/atlassian/docker-chromium-xvfb)
- [WebdriverIO Xvfb guide](https://webdriver.io/docs/headless-and-xvfb/)

### Testing & QA Capabilities

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| Vitest | ^4.0.18 | Test runner | **INHERITED** - Already configured |
| @vitest/coverage-v8 | ^4.0.18 | Code coverage | **INHERITED** - Already configured |

**Rationale:** OpenClaw already has Vitest fully configured (`vitest.config.ts`, `vitest.e2e.config.ts`, `vitest.live.config.ts`). Vitest is 10-20x faster than Jest on large codebases, has built-in TypeScript support, and uses ESM by default (matches OpenClaw's `"type": "module"`). No changes needed - Juancho will use existing test infrastructure.

**Test Types Already Configured:**
- Unit tests: `src/**/*.test.ts`
- E2E tests: `**/*.e2e.test.ts`
- Live tests: `**/*.live.test.ts`

**For Juancho's QA Phase:**
- Run tests: `pnpm test`
- Run specific suites: `vitest run --config vitest.e2e.config.ts`
- Coverage reports: `vitest run --coverage`

**Confidence:** HIGH (verified in vitest.config.ts and package.json)

**Sources:**
- [Vitest vs Jest comparison](https://howtotestfrontend.com/resources/vitest-vs-jest-which-to-pick)
- [Vitest performance benefits](https://medium.com/@onix_react/vitest-a-faster-modern-alternative-to-jest-9d5eaa15092f)

### Documentation Generation

| Component | Approach | Implementation |
|-----------|----------|----------------|
| Planning docs | **File operations** | Write `.planning/` files via existing file tools |
| Code docs | **Template system** | Generate README.md, API docs using templates |
| Inline comments | **Code generation** | Include documentation in code generation prompts |
| Work logs | **Session transcripts** | Use OpenClaw's session management (already tracks interactions) |

**Rationale:** Documentation generation doesn't require new libraries - it's content generation via LLM + file writes. OpenClaw already has file I/O tools and session transcripts. The GSD methodology provides the structure; the AI generates the content.

**Documentation Types:**
1. **Planning Docs:** Research findings, roadmaps, phase plans (stored in `.planning/`)
2. **Code Docs:** README.md, API documentation, usage examples
3. **Inline Comments:** JSDoc, function documentation, complex logic explanations
4. **Work Logs:** Decision rationale, alternative approaches considered, blockers encountered

**Confidence:** HIGH (no new dependencies needed, uses existing capabilities)

---

## Supporting Infrastructure

### Database & Storage

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| sqlite-vec | 0.1.7-alpha.2 | Vector storage for memory | **INHERITED** - OpenClaw uses this |

**Rationale:** OpenClaw uses SQLite with vector extensions for memory search. Perfect for storing project context, code patterns, and decisions. No changes needed - Juancho inherits this for context recall.

**Confidence:** HIGH (verified in package.json)

### Process & Terminal

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| @lydell/node-pty | 1.2.0-beta.3 | Terminal/shell integration | **INHERITED** - OpenClaw uses this |
| commander | ^14.0.3 | CLI framework | **INHERITED** - OpenClaw CLI |

**Rationale:** OpenClaw already has terminal integration (for running commands, tests, builds) and CLI framework. Juancho will use these for executing development commands (npm install, git operations, test runs).

**Confidence:** HIGH (verified in package.json)

---

## Configuration & Environment

### Environment Variables for VPS Deployment

```bash
# Node.js
NODE_ENV=production
NODE_OPTIONS="--max-old-space-size=4096"

# OpenClaw/Juancho
OPENCLAW_PROFILE=production
OPENCLAW_WORKSPACE=/opt/juancho/workspace

# Display (for Xvfb)
DISPLAY=:99

# AI Providers (configure as needed)
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Telegram
TELEGRAM_BOT_TOKEN=...
```

### System Requirements (VPS)

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Disk | 20 GB | 50+ GB SSD |
| OS | Ubuntu 22.04+ | Ubuntu 24.04 LTS |

**Rationale:** Node.js 22 + Playwright + Xvfb + AI models require decent resources. 4GB minimum for basic operation; 8GB recommended for smooth multi-tasking (browser automation + LLM inference + git operations).

---

## Architecture Decisions

### What OpenClaw Provides vs. What Juancho Adds

| Category | OpenClaw (Inherited) | Juancho (Added) |
|----------|---------------------|-----------------|
| **AI Framework** | pi-coding-agent toolkit, multi-provider support | GSD-specific system prompts and workflow orchestration |
| **Browser** | Playwright integration, CDP tools | Xvfb setup for VPS headless mode |
| **Messaging** | Telegram bot with Grammy | Developer-focused command patterns (commit, test, deploy) |
| **File I/O** | Read/write/search tools | `.planning/` directory structure and GSD file conventions |
| **Testing** | Vitest with unit/e2e/live configs | QA phase automation (run tests, validate builds) |
| **Git** | ❌ None | simple-git integration for commit/branch/PR automation |
| **Sessions** | Session management, transcripts | Phase-based session structuring (research → plan → build → test) |
| **System Prompts** | General assistant prompts | Coding-focused, GSD methodology, autonomous decision-making |

### Integration Points

1. **System Prompt Injection:** Modify `src/agents/system-prompt.ts` to load GSD phase instructions
2. **Git Tool Addition:** Create new tools in `src/agents/tools/` for git operations (using simple-git)
3. **Telegram Command Extension:** Add developer commands in `src/telegram/bot-native-commands.ts`
4. **VPS Startup:** Wrap OpenClaw gateway start with Xvfb (`xvfb-run openclaw gateway`)
5. **Phase Orchestration:** Create GSD orchestrator that uses existing tools in sequence

---

## Migration & Deployment Strategy

### Phase 1: Local Development (Current)
- Use existing OpenClaw setup
- Develop GSD integration locally
- Test with local browser (no Xvfb needed)

### Phase 2: VPS Setup
```bash
# Install system dependencies
sudo apt-get update
sudo apt-get install -y xvfb libgbm-dev libasound2 git

# Install Node.js 22.x (LTS)
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install pnpm
curl -fsSL https://get.pnpm.io/install.sh | sh -

# Clone and build Juancho
git clone https://github.com/ddikddak/Juancho.git
cd Juancho
pnpm install
pnpm build

# Install Playwright browsers
npx playwright install --with-deps chromium

# Start with Xvfb
xvfb-run -a --server-args="-screen 0 1280x800x24" \
  pnpm openclaw gateway --port 18789
```

### Phase 3: Systemd Service (Production)
```ini
# /etc/systemd/system/juancho.service
[Unit]
Description=Juancho - Autonomous AI Developer
After=network.target

[Service]
Type=simple
User=juancho
WorkingDirectory=/opt/juancho
Environment="DISPLAY=:99"
Environment="NODE_ENV=production"
ExecStartPre=/usr/bin/Xvfb :99 -screen 0 1280x800x24 -ac &
ExecStart=/usr/local/bin/pnpm openclaw gateway
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

## Confidence Assessment

| Component | Confidence | Source |
|-----------|-----------|--------|
| Node.js 22 LTS | HIGH | Official Node.js releases, package.json |
| TypeScript/Vitest | HIGH | package.json, vitest.config.ts verified |
| pi-coding-agent | HIGH | npm package, pi-mono GitHub |
| Playwright | HIGH | package.json, src/browser/ verified |
| Grammy (Telegram) | HIGH | package.json, src/telegram/ verified |
| simple-git | HIGH | npm stats, clear use case |
| Xvfb setup | HIGH | Industry standard, well-documented |
| GSD integration | MEDIUM | Approach sound, needs verification of prompt system flexibility |
| Vitest for QA | HIGH | Already configured, verified in codebase |

---

## Installation Commands

### Add New Dependencies
```bash
# Git automation
pnpm add simple-git@^3.30.0
```

### System Dependencies (VPS)
```bash
# Xvfb and browser dependencies
sudo apt-get install -y xvfb libgbm-dev libasound2 git

# Playwright browsers (run after pnpm install)
npx playwright install --with-deps chromium
```

---

## Sources

### Official Documentation
- [Node.js 22 LTS Release](https://nodejs.org/en/blog/release/v22.22.0)
- [Node.js Release Schedule](https://github.com/nodejs/Release)
- [Playwright Documentation](https://playwright.dev/)
- [Grammy Telegram Bot Framework](https://grammy.dev/)

### Package Repositories
- [pi-mono GitHub](https://github.com/badlogic/pi-mono)
- [pi-coding-agent README](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md)
- [simple-git npm](https://www.npmjs.com/package/simple-git)

### Comparison & Best Practices
- [Playwright vs Puppeteer (BrowserStack)](https://www.browserstack.com/guide/playwright-vs-puppeteer)
- [Playwright vs Puppeteer (ZenRows)](https://www.zenrows.com/blog/playwright-vs-puppeteer)
- [Vitest vs Jest (How To Test Frontend)](https://howtotestfrontend.com/resources/vitest-vs-jest-which-to-pick)
- [simple-git comparison](https://npm-compare.com/isomorphic-git,nodegit,simple-git)
- [npm trends](https://npmtrends.com/isomorphic-git-vs-nodegit-vs-simple-git)

### VPS & Browser Setup
- [Xvfb Chrome Setup](https://www.mattzeunert.com/2018/07/21/running-headful-chrome-on-ubuntu-server.html)
- [Docker Chromium Xvfb](https://github.com/atlassian/docker-chromium-xvfb)
- [WebdriverIO Xvfb Guide](https://webdriver.io/docs/headless-and-xvfb/)

### Agentic Coding & Workflows
- [Complete Guide to Agentic Coding 2026](https://www.teamday.ai/blog/complete-guide-agentic-coding-2026)
- [Prompt Engineering 2026](https://www.ibm.com/think/prompt-engineering)
- [Telegram Bots for Developers 2025](https://stellaray777.medium.com/a-developers-guide-to-building-telegram-bots-in-2025-dbc34cd22337)
- [Vitest Performance](https://medium.com/@onix_react/vitest-a-faster-modern-alternative-to-jest-9d5eaa15092f)
