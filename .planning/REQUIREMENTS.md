# Requirements: Juancho

**Defined:** 2026-02-14
**Core Value:** An AI developer that follows real engineering discipline and works proactively — writes the plan, then autonomously executes it phase by phase via cron/heartbeat, only interrupting humans for blockers and decisions.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Developer Identity

- [ ] **IDEN-01**: System prompts (SOUL.md, AGENTS.md) rewritten for coding-focused developer persona, reusing OpenClaw's existing workspace file system
- [ ] **IDEN-02**: Onboarding flow redesigned as Juancho-branded developer experience (not generic assistant setup — walks user through repo connection, git config, AI provider setup, first project definition)
- [ ] **IDEN-03**: Juancho developer persona with engineering discipline baked into all interactions, leveraging OpenClaw's existing agent identity system
- [ ] **IDEN-04**: Coding-specific Telegram commands reusing OpenClaw's existing native command system (status, commit, test, phase, research)

### GSD Workflow Pipeline

- [ ] **GSD-01**: Full GSD pipeline integrated (.planning/ directory with research, requirements, roadmap, phases), reusing OpenClaw's workspace directory and file I/O tools
- [ ] **GSD-02**: Intensive project definition stage at task start (structured questioning before coding), leveraging OpenClaw's existing conversation/session system
- [ ] **GSD-03**: Phase-based execution: research → plan → build → test per phase, orchestrated via OpenClaw's tool system
- [ ] **GSD-04**: Configurable autonomy levels (user decides how much Juancho asks vs. decides), stored in workspace config
- [ ] **GSD-05**: Task stack tracking across all phases (what's done, in progress, blocked, pending), persisted in .planning/ state files

### Proactive Autonomy (Cron/Heartbeat)

- [ ] **PROA-01**: Heartbeat cron job (reusing OpenClaw's existing cron/heartbeat system) that periodically checks work status and determines next action
- [ ] **PROA-02**: Self-directed phase transitions (agent decides to move to next phase when tests pass)
- [ ] **PROA-03**: Configurable proactivity schedule (active hours, frequency, intensity)
- [ ] **PROA-04**: When blocked, agent autonomously researches alternatives or works on other tasks
- [ ] **PROA-05**: Plan persistence — full plan written to .planning/ files, cron reads and executes
- [ ] **PROA-06**: Status reporting via Telegram at phase transitions and when blocked

### Git Workflow

- [ ] **GIT-01**: Git commits per phase with conventional commit format
- [ ] **GIT-02**: Feature branches per project/milestone
- [ ] **GIT-03**: Pull request creation with summary of changes
- [ ] **GIT-04**: Branch naming conventions enforced

### Testing & QA

- [ ] **TEST-01**: Test-first discipline — run tests after each code change
- [ ] **TEST-02**: Checkpoint validation — test suite must pass before moving to next phase
- [ ] **TEST-03**: Build verification — ensure project builds successfully per phase
- [ ] **TEST-04**: Error detection and debugging — surface failures, don't suppress them

### Browser & Research

- [ ] **BROW-01**: Browser research phase before coding (docs, APIs, patterns, examples)
- [ ] **BROW-02**: Headless browser on VPS with Xvfb for reliable browser automation
- [ ] **BROW-03**: Web app testing via browser (verify built features actually work)

### Documentation

- [ ] **DOCS-01**: GSD planning docs per project (.planning/ with research, requirements, roadmap)
- [ ] **DOCS-02**: Code documentation generation (README, API docs, inline comments)
- [ ] **DOCS-03**: Work logs with decisions, reasoning, and tradeoffs documented

### Infrastructure

- [ ] **INFR-01**: Telegram as primary interface with rich formatting (code blocks, files, buttons)
- [ ] **INFR-02**: Remote VPS deployment with headless browser infrastructure
- [ ] **INFR-03**: Multi-provider AI support (keep OpenClaw's existing flexibility)
- [ ] **INFR-04**: Upstream sync capability (pull OpenClaw improvements without breaking customizations)

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

| Requirement | Phase | Status |
|-------------|-------|--------|
| IDEN-01 | TBD | Pending |
| IDEN-02 | TBD | Pending |
| IDEN-03 | TBD | Pending |
| IDEN-04 | TBD | Pending |
| GSD-01 | TBD | Pending |
| GSD-02 | TBD | Pending |
| GSD-03 | TBD | Pending |
| GSD-04 | TBD | Pending |
| GSD-05 | TBD | Pending |
| PROA-01 | TBD | Pending |
| PROA-02 | TBD | Pending |
| PROA-03 | TBD | Pending |
| PROA-04 | TBD | Pending |
| PROA-05 | TBD | Pending |
| PROA-06 | TBD | Pending |
| GIT-01 | TBD | Pending |
| GIT-02 | TBD | Pending |
| GIT-03 | TBD | Pending |
| GIT-04 | TBD | Pending |
| TEST-01 | TBD | Pending |
| TEST-02 | TBD | Pending |
| TEST-03 | TBD | Pending |
| TEST-04 | TBD | Pending |
| BROW-01 | TBD | Pending |
| BROW-02 | TBD | Pending |
| BROW-03 | TBD | Pending |
| DOCS-01 | TBD | Pending |
| DOCS-02 | TBD | Pending |
| DOCS-03 | TBD | Pending |
| INFR-01 | TBD | Pending |
| INFR-02 | TBD | Pending |
| INFR-03 | TBD | Pending |
| INFR-04 | TBD | Pending |

**Coverage:**
- v1 requirements: 33 total
- Mapped to phases: 0 (pending roadmap)
- Unmapped: 33

---
*Requirements defined: 2026-02-14*
*Last updated: 2026-02-14 after initial definition*
