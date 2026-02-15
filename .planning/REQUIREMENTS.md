# Requirements: Juancho

**Defined:** 2026-02-14
**Core Value:** An AI developer that follows real engineering discipline and works proactively — writes the plan, then autonomously executes it phase by phase via cron/heartbeat, only interrupting humans for blockers and decisions.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Developer Identity

- [x] **IDEN-01**: System prompts (SOUL.md, AGENTS.md) rewritten for coding-focused developer persona, reusing OpenClaw's existing workspace file system
- [ ] **IDEN-02**: Onboarding flow redesigned as Juancho-branded developer experience (not generic assistant setup — walks user through repo connection, git config, AI provider setup, first project definition)
- [x] **IDEN-03**: Juancho developer persona with engineering discipline baked into all interactions, leveraging OpenClaw's existing agent identity system
- [ ] **IDEN-04**: Coding-specific Telegram commands reusing OpenClaw's existing native command system (status, commit, test, phase, research)

### GSD Workflow Pipeline

- [x] **GSD-01**: Full GSD pipeline integrated (.planning/ directory with research, requirements, roadmap, phases), reusing OpenClaw's workspace directory and file I/O tools
- [x] **GSD-02**: Intensive project definition stage at task start (structured questioning before coding), leveraging OpenClaw's existing conversation/session system
- [x] **GSD-03**: Phase-based execution: research → plan → build → test per phase, orchestrated via OpenClaw's tool system
- [x] **GSD-04**: Configurable autonomy levels (user decides how much Juancho asks vs. decides), stored in workspace config
- [x] **GSD-05**: Task stack tracking across all phases (what's done, in progress, blocked, pending), persisted in .planning/ state files

### Proactive Autonomy (Cron/Heartbeat)

- [ ] **PROA-01**: Heartbeat cron job (reusing OpenClaw's existing cron/heartbeat system) that periodically checks work status and determines next action
- [ ] **PROA-02**: Self-directed phase transitions (agent decides to move to next phase when tests pass)
- [ ] **PROA-03**: Configurable proactivity schedule (active hours, frequency, intensity)
- [ ] **PROA-04**: When blocked, agent autonomously researches alternatives or works on other tasks
- [ ] **PROA-05**: Plan persistence — full plan written to .planning/ files, cron reads and executes
- [ ] **PROA-06**: Status reporting via Telegram at phase transitions and when blocked

### Git Workflow

- [x] **GIT-01**: Git commits per phase with conventional commit format
- [x] **GIT-02**: Feature branches per project/milestone
- [x] **GIT-03**: Pull request creation with summary of changes
- [x] **GIT-04**: Branch naming conventions enforced

### Testing & QA

- [x] **TEST-01**: Test-first discipline — run tests after each code change
- [x] **TEST-02**: Checkpoint validation — test suite must pass before moving to next phase
- [x] **TEST-03**: Build verification — ensure project builds successfully per phase
- [x] **TEST-04**: Error detection and debugging — surface failures, don't suppress them

### Browser & Research

- [x] **BROW-01**: Browser research phase before coding (docs, APIs, patterns, examples)
- [x] **BROW-02**: Headless browser on VPS with Xvfb for reliable browser automation
- [x] **BROW-03**: Web app testing via browser (verify built features actually work)

### Documentation

- [x] **DOCS-01**: GSD planning docs per project (.planning/ with research, requirements, roadmap)
- [ ] **DOCS-02**: Code documentation generation (README, API docs, inline comments)
- [ ] **DOCS-03**: Work logs with decisions, reasoning, and tradeoffs documented

### Infrastructure

**Note:** INFR-01 through INFR-04 are inherited capabilities from OpenClaw. Phase 4 verifies they work in Juancho context, no major implementation needed.

- [ ] **INFR-01**: Telegram as primary interface with rich formatting (code blocks, files, buttons) — INHERITED from OpenClaw
- [ ] **INFR-02**: Remote VPS deployment with headless browser infrastructure — INHERITED from OpenClaw
- [ ] **INFR-03**: Multi-provider AI support (keep OpenClaw's existing flexibility) — INHERITED from OpenClaw
- [ ] **INFR-04**: Upstream sync capability (pull OpenClaw improvements without breaking customizations) — INHERITED from OpenClaw

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Safety & Isolation

- **SAFE-01**: MicroVM/gVisor sandbox for autonomous code execution
- **SAFE-02**: Verification gates per phase (formal validation before proceeding)
- **SAFE-03**: Automated security scanning on generated code

### Advanced Workflows

- **ADVW-01**: Multi-project management (work on multiple repos simultaneously)
- **ADVW-02**: Advanced git operations (rebase, cherry-pick, conflict resolution)
- **ADVW-03**: Deployment automation (CI/CD integration)
- **ADVW-04**: Code review mode (separate from code generation)

### Intelligence

- **INTL-01**: Long-term memory across projects (learn from past work)
- **INTL-02**: Custom skill creation (user-defined workflows)
- **INTL-03**: Voice/audio interaction via Telegram

## Out of Scope

| Feature | Reason |
|---------|--------|
| General chatbot behavior | Juancho is a developer, not a personal assistant |
| Multi-user support | Personal use only, single operator |
| Custom web dashboard | Telegram is sufficient, custom UI is scope creep |
| Custom AI model training | Uses existing providers as-is |
| Mobile app | Telegram handles mobile access |
| Rebuilding OpenClaw features | Leverage what exists, reconfigure it |

## Traceability

| Requirement | Phase | Status | Notes |
|-------------|-------|--------|-------|
| IDEN-01 | Phase 1 | Complete | Core system prompts rewrite |
| IDEN-03 | Phase 1 | Complete | Developer persona in interactions |
| GSD-01 | Phase 2 | Complete | .planning/ structure |
| GSD-02 | Phase 2 | Complete | Intensive questioning stage |
| GSD-03 | Phase 2 | Complete | Research → plan → build → test |
| GSD-04 | Phase 2 | Complete | Configurable autonomy |
| GSD-05 | Phase 2 | Complete | Task stack tracking |
| DOCS-01 | Phase 2 | Complete | GSD planning docs |
| BROW-01 | Phase 2 | Complete | Browser research before coding |
| GIT-01 | Phase 3 | Complete | Conventional commits in execute-phase |
| GIT-02 | Phase 3 | Complete | Feature branches in execute-phase |
| GIT-03 | Phase 3 | Complete | PR creation via gh CLI |
| GIT-04 | Phase 3 | Complete | Branch naming feat/phase-{XX}-{slug} |
| TEST-01 | Phase 3 | Complete | Test-first discipline in AGENTS.md |
| TEST-02 | Phase 3 | Complete | Test gate in verify-phase |
| TEST-03 | Phase 3 | Complete | Build verification in verify-phase |
| TEST-04 | Phase 3 | Complete | Error surfacing in execute-phase |
| BROW-02 | Phase 3 | Complete | Xvfb/Playwright in verify-phase |
| BROW-03 | Phase 3 | Complete | Browser testing checklist |
| PROA-01 | Phase 4 | Pending | Heartbeat cron job |
| PROA-02 | Phase 4 | Pending | Self-directed phase transitions |
| PROA-03 | Phase 4 | Pending | Configurable proactivity schedule |
| PROA-04 | Phase 4 | Pending | Autonomous research when blocked |
| PROA-05 | Phase 4 | Pending | Plan persistence |
| PROA-06 | Phase 4 | Pending | Status reporting via Telegram |
| IDEN-02 | Phase 4 | Pending | Onboarding flow (wraps Phase 2 GSD pipeline) |
| IDEN-04 | Phase 4 | Pending | Telegram commands (thin wrappers for built functionality) |
| DOCS-02 | Phase 4 | Pending | Code documentation generation |
| DOCS-03 | Phase 4 | Pending | Work logs |
| INFR-01 | Phase 4 | Pending | Telegram interface verification (INHERITED) |
| INFR-02 | Phase 4 | Pending | VPS deployment verification (INHERITED) |
| INFR-03 | Phase 4 | Pending | Multi-provider AI verification (INHERITED) |
| INFR-04 | Phase 4 | Pending | Upstream sync verification (INHERITED) |

**Coverage:**
- v1 requirements: 33 total
- Mapped to phases: 33 (100% coverage)
- Unmapped: 0
- Infrastructure requirements: 4 inherited from OpenClaw (verification only)

---
*Requirements defined: 2026-02-14*
*Last updated: 2026-02-15 — Phase 3 requirements (GIT-01–04, TEST-01–04, BROW-02–03) marked complete*
