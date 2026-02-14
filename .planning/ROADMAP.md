# Roadmap: Juancho

## Overview

Transform OpenClaw into Juancho, an autonomous AI developer that follows structured GSD methodology. This is a reconfiguration/extension of OpenClaw's existing capabilities (Telegram, VPS deployment, multi-provider AI, browser automation) rather than building from scratch. The journey: rewrite the developer identity/persona, integrate GSD pipeline, add git/testing workflows, enable proactive autonomy, and wrap with polished Telegram UX.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Developer Identity & Persona** - Rewrite system prompts and agent identity for coding-focused developer
- [ ] **Phase 2: GSD Pipeline Integration** - Build .planning/ workspace structure and orchestration tools
- [ ] **Phase 3: Git & Testing Workflow** - Automated phase commits, test enforcement, and browser research
- [ ] **Phase 4: Proactive Autonomy & UX** - Cron/heartbeat execution, Telegram commands, and onboarding flow

## Phase Details

### Phase 1: Developer Identity & Persona
**Goal**: Juancho has a distinct developer personality and coding-focused identity, separate from OpenClaw's general assistant persona
**Depends on**: Nothing (first phase)
**Requirements**: IDEN-01, IDEN-03
**Success Criteria** (what must be TRUE):
  1. System prompts (SOUL.md, AGENTS.md) rewritten to define Juancho as coding-focused developer, not general assistant
  2. Agent identity reflects engineering discipline in all interactions (research before code, test after changes, document decisions)
  3. Juancho responds with developer context (talks about phases, tests, PRs, not generic tasks)
**Plans**: TBD

Plans:
- [ ] 01-01: [Plan description TBD during planning]

### Phase 2: GSD Pipeline Integration
**Goal**: Juancho creates and manages .planning/ directory structure, orchestrates phase-based execution, and maintains project state
**Depends on**: Phase 1 (developer persona established)
**Requirements**: GSD-01, GSD-02, GSD-03, GSD-04, GSD-05, DOCS-01, BROW-01
**Success Criteria** (what must be TRUE):
  1. New project creates .planning/ directory with research/, requirements/, roadmap/, phases/ structure
  2. Intensive questioning stage captures requirements before any coding starts
  3. Each phase follows research → plan → build → test workflow
  4. Autonomy level configurable (user decides how much agent asks vs. decides)
  5. Task stack tracks what's done, in progress, blocked, pending across all phases
  6. GSD planning docs persist in .planning/ and drive execution
  7. Browser research runs before coding (using OpenClaw's existing Stagehand/Puppeteer capabilities)
**Plans**: TBD

Plans:
- [ ] 02-01: [Plan description TBD during planning]

### Phase 3: Git & Testing Workflow
**Goal**: Every phase produces commits with conventional format, tests run automatically, phase transitions require passing test suite, and browser testing verifies built features
**Depends on**: Phase 2 (GSD structure exists)
**Requirements**: GIT-01, GIT-02, GIT-03, GIT-04, TEST-01, TEST-02, TEST-03, TEST-04, BROW-02, BROW-03
**Success Criteria** (what must be TRUE):
  1. Git commits created automatically per phase with conventional commit format (using OpenClaw's existing git capabilities)
  2. Feature branches created per project/milestone with naming conventions enforced
  3. Pull requests generated with summary of changes when phase completes
  4. Tests run after each code change (test-first discipline)
  5. Phase transitions blocked until test suite passes (checkpoint validation)
  6. Build verification runs before moving to next phase
  7. Errors surfaced and debugged, not suppressed
  8. Headless browser works reliably on VPS with Xvfb (verifying OpenClaw's browser automation works headless)
  9. Built web apps tested via browser to verify they actually work
**Plans**: TBD

Plans:
- [ ] 03-01: [Plan description TBD during planning]

### Phase 4: Proactive Autonomy & UX
**Goal**: Juancho executes work autonomously via cron/heartbeat, exposes functionality through Telegram commands, and provides polished onboarding experience
**Depends on**: Phase 3 (git/testing workflows operational)
**Requirements**: PROA-01, PROA-02, PROA-03, PROA-04, PROA-05, PROA-06, IDEN-02, IDEN-04, DOCS-02, DOCS-03, INFR-01, INFR-02, INFR-03, INFR-04
**Success Criteria** (what must be TRUE):
  1. Heartbeat cron job (reusing OpenClaw's existing cron system) checks work status periodically and determines next action
  2. Agent transitions to next phase automatically when tests pass
  3. Proactivity schedule configurable (active hours, frequency, intensity)
  4. When blocked, agent researches alternatives or works on other tasks autonomously
  5. Full plan persists in .planning/, cron reads and executes from files
  6. Status updates sent via Telegram at phase transitions and when blocked
  7. Coding-specific Telegram commands work (/status, /commit, /test, /phase, /research) - thin wrappers calling functionality built in earlier phases
  8. Onboarding flow redesigned for developer context (repo setup, git config, AI provider setup, first project definition) - wraps GSD pipeline from Phase 2
  9. Code documentation auto-generated (README, API docs, inline comments)
  10. Work logs capture decisions, reasoning, and tradeoffs
  11. Telegram message chunking handles long code/diffs (4096 char limit)
  12. Large content delivered via file upload when appropriate
  13. Inline keyboards enable approvals and interactions
  14. Multi-provider AI support verified (inherited from OpenClaw - Claude, OpenAI, Gemini, Ollama)
  15. VPS deployment verified with headless browser infrastructure (inherited from OpenClaw)
  16. Upstream sync capability verified (can pull OpenClaw improvements without breaking customizations)
**Plans**: TBD

Plans:
- [ ] 04-01: [Plan description TBD during planning]

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Developer Identity & Persona | 0/? | Not started | - |
| 2. GSD Pipeline Integration | 0/? | Not started | - |
| 3. Git & Testing Workflow | 0/? | Not started | - |
| 4. Proactive Autonomy & UX | 0/? | Not started | - |

---
*Roadmap created: 2026-02-14*
*Last updated: 2026-02-14 (revised after feedback - consolidated to 4 phases, moved infrastructure verification and UX wrappers to Phase 4)*
