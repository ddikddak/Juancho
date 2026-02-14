# Feature Landscape

**Domain:** Autonomous AI Developer Agents
**Researched:** 2026-02-14

## Table Stakes

Features users expect from any autonomous AI dev agent in 2026. Missing = product feels incomplete or broken.

| Feature | Why Expected | Complexity | Notes | OpenClaw Status |
|---------|--------------|------------|-------|-----------------|
| Code generation and editing | Core capability - agents must write code from requirements | Medium | Multi-file operations are standard; single-file edits are table stakes | ✅ Has via coding-agent skill (Codex/Claude Code/Pi integration) |
| Terminal/CLI execution | Agents must run commands, tests, builds to verify their work | Low | PTY support required for interactive tools | ✅ Has bash tool with PTY support |
| Multi-file operations | Editing across codebase (not just single files) is baseline expectation | Medium | Agents that use fragile cat/rewrite patterns cause corruption; need proper file editing | ✅ Via coding agents (Codex --full-auto, Claude Code) |
| Repository awareness | Understanding codebase structure, existing code, dependencies | High | Failure pattern: agents lose context as file count grows | ⚠️ Partial - coding agents provide this, needs scoping |
| Git integration | Commit, branch, push, PR creation are expected workflows | Low | Must follow repository conventions (branch naming, commit style) | ✅ Has git capabilities built-in |
| Testing execution | Run test suites, interpret results, verify builds | Medium | 70%+ of devs rewrite AI code before production; testing is mandatory | ✅ Via bash tool + coding agents |
| Error handling and debugging | Detect failures, read stack traces, propose fixes | High | Critical failure pattern: agents suppress errors instead of reporting them | ⚠️ Needs verification - coding agents handle this but may suppress |
| Model context management | Handle large codebases without hitting context limits | High | Simply throwing massive specs at agents doesn't work; need smart chunking | ⚠️ Needs investigation - Context7 integration exists |
| Multi-provider AI support | Support multiple LLM providers (Claude, OpenAI, Gemini, etc.) | Medium | No vendor lock-in; cost optimization | ✅ Full multi-provider support (Anthropic, OpenAI, Gemini, Ollama, Bedrock) |
| Async/background execution | Long-running tasks must run without blocking | Medium | Critical for builds, large refactors, batch operations | ✅ Has background process support with monitoring |

## Differentiators

Features that set Juancho apart from AI copilots and generic coding assistants. Not expected, but highly valued.

| Feature | Value Proposition | Complexity | Notes | OpenClaw Status |
|---------|-------------------|------------|-------|-----------------|
| **GSD Methodology Integration** | Solves user's #1 pain point: AI tools lack discipline | High | Research → Plan → Build → Test per phase; prevents "build everything at once" failures | ❌ Need to build - templates exist at ~/.claude/get-shit-done/ |
| **Intensive project definition** | Front-loads questions to avoid mid-project confusion | Medium | AI agents fail on business logic; intensive upfront requirements gathering prevents this | ❌ Need to build - part of GSD pipeline |
| **Phase-based execution** | Incremental delivery prevents monolithic failures | High | Each phase: research, build, test, verify before moving forward | ❌ Need to build - core GSD principle |
| **Configurable autonomy levels** | User controls how much agent asks vs. decides | Medium | Intensive at project start, hands-off during execution | ❌ Need to build |
| **Built-in testing discipline** | Agent writes tests, runs them, verifies output per phase | High | Addresses core failure: AI creates 1.7x more bugs than humans; testing catches them early | ⚠️ Partial - can run tests, need to enforce test-first discipline |
| **Documentation generation** | Auto-generates planning docs, code docs, work logs | Medium | Addresses pain point: existing AI tools don't document | ⚠️ Partial - coding agents can write docs, need structured approach |
| **Browser use for research** | Agent researches libraries, docs, patterns before coding | High | Addresses failure pattern: agents make outdated tech choices | ✅ Has browser via Stagehand/Puppeteer |
| **Parallel task execution** | Multiple worktrees, parallel issue fixing | Medium | Git worktrees + background tasks enable simultaneous work on multiple issues | ✅ Supports worktrees, background tasks, parallel Codex instances |
| **Telegram as primary interface** | Always-on, mobile-accessible, no custom UI needed | Low | Remote developer employee accessible from anywhere | ✅ Full Telegram integration |
| **Remote VPS deployment** | Runs on server, not local machine; always available | Medium | True "employee" model - agent works independently on remote infrastructure | ✅ OpenClaw supports remote deployment |
| **Auto-commit with conventions** | Follows repository commit conventions, branch naming | Low | Professional git hygiene without hand-holding | ⚠️ Partial - git support exists, need convention enforcement |
| **Work logging and reasoning** | Documents decisions, rationale, tradeoffs made | Medium | Transparency into agent's thinking; critical for trust | ❌ Need to build |

## Anti-Features

Features to explicitly NOT build. Common mistakes in AI dev agent domain.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **General chatbot behavior** | Juancho is a developer, not a personal assistant | Coding-focused system prompts; reject non-dev requests |
| **Multi-user support** | Adds complexity without value for personal dev tool | Single operator only; simplify authentication |
| **Custom UI/web dashboard** | Telegram is sufficient; custom UI is scope creep | Leverage Telegram's rich messaging (code blocks, files, buttons) |
| **Full autonomy without guardrails** | Leads to security issues, bad commits, broken code | Configurable autonomy with phase-based verification gates |
| **Build everything at once** | #1 user pain point with existing AI tools | Phase-based execution with verification per phase |
| **Skip research phase** | Causes outdated tech choices, business logic failures | Mandatory research phase before any coding |
| **Suppress errors silently** | Critical failure pattern in AI agents | Surface errors, explain them, propose fixes with context |
| **Rewrite entire files for minor edits** | Fragile pattern causing corruption in large files | Use proper file editing (not cat > file rewrites) |
| **Accept any task without definition** | Leads to business logic mismatches, wrong implementations | Intensive project definition stage (GSD-style questioning) |
| **No testing until end** | AI code has 1.7x more bugs; late testing = rewrite hell | Test-driven: write tests, implement, verify per phase |
| **Generate code without understanding** | 70%+ of devs rewrite AI code; blind generation fails | Repository-aware generation; verify against existing patterns |

## Feature Dependencies

Critical dependencies between features that affect implementation order.

```
Core Infrastructure (OpenClaw base)
├── Telegram integration ✅
├── Multi-provider AI ✅
├── Git capabilities ✅
├── Browser use ✅
└── Background execution ✅

GSD Pipeline (to build)
├── Project definition workflow
│   ├── Intensive questioning system
│   └── Requirements documentation generation
├── Phase structure
│   ├── Research phase (uses browser)
│   ├── Planning phase (uses documentation generation)
│   ├── Build phase (uses coding agents)
│   └── Test phase (uses testing execution)
└── Verification gates
    ├── Test execution (exists)
    └── Quality checks (to build)

Developer Discipline (to build)
├── Testing discipline
│   ├── Test execution ✅
│   ├── Test-first enforcement (to build)
│   └── Coverage tracking (to build)
├── Documentation discipline
│   ├── Planning docs (to build)
│   ├── Code documentation (partial)
│   └── Work logs (to build)
└── Git discipline
    ├── Git operations ✅
    ├── Convention enforcement (to build)
    └── Branch/commit structure (to build)

Autonomy Control (to build)
├── Configurable questioning
├── Approval gates
└── Auto-proceed rules
```

## MVP Recommendation

For MVP (Milestone 1), prioritize table stakes + one key differentiator to validate core value prop.

**Phase 1: Core Foundation (Table Stakes)**
1. Coding-focused system prompts (rewrite OpenClaw agent instructions)
2. Developer onboarding flow (not general assistant)
3. Repository-aware code generation (leverage existing coding agents)
4. Testing execution discipline (enforce test runs per change)
5. Git workflow integration (commits, branches, PRs with conventions)

**Phase 2: GSD Differentiator (Core Value)**
6. Intensive project definition workflow
7. Phase-based execution structure (research, plan, build, test)
8. Documentation generation (planning docs, work logs)
9. Verification gates per phase

**Phase 3: Polish (Enhanced Discipline)**
10. Configurable autonomy levels
11. Browser research integration
12. Parallel task execution
13. Full work logging and reasoning transparency

## Defer to Post-MVP

These features are valuable but not critical for validating core value proposition:

- **Voice/audio interaction**: Telegram text is sufficient; voice adds complexity without clear dev workflow benefit
- **Custom skill creation**: OpenClaw has this; focus on coding workflow first
- **Multi-project management**: Start with single-project focus; add later if needed
- **Advanced git operations**: Rebase, cherry-pick, etc. — basic commit/branch/PR is MVP
- **Code review as separate mode**: Can be added as enhancement once core coding workflow proven
- **Deployment automation**: CI/CD integration valuable but not MVP-critical
- **Performance optimization**: Agent speed less critical than agent correctness in MVP

## OpenClaw Capability Mapping

What OpenClaw already provides that Juancho can leverage:

| Capability | OpenClaw Feature | How Juancho Uses It |
|------------|------------------|---------------------|
| **Messaging** | Telegram extension | Primary interface for developer interaction |
| **Browser automation** | Stagehand/Puppeteer integration | Research phase: library docs, patterns, examples |
| **Code execution** | Bash tool with PTY | Run builds, tests, commands |
| **Coding agents** | coding-agent skill (Codex/Claude/Pi) | Core code generation engine |
| **Git operations** | Built-in git capabilities | Version control workflow |
| **Multi-provider AI** | Anthropic/OpenAI/Gemini/Ollama support | Model flexibility, cost optimization |
| **Background processes** | Process management system | Long-running builds, parallel tasks |
| **File operations** | Read/Write tools | Documentation generation, config editing |
| **Skills system** | Extensible skill architecture | Add GSD workflow as skills/agents |
| **Remote deployment** | Docker/VPS support | Deploy on remote server for 24/7 availability |

## What Needs Building

| Feature | Why Not Leveraging OpenClaw | Approach |
|---------|----------------------------|----------|
| **GSD pipeline** | OpenClaw is general assistant, not dev-focused | Implement as agent mode or skill system |
| **Intensive questioning** | OpenClaw conversation is freeform | Build structured requirements gathering workflow |
| **Phase execution** | No phase concept in OpenClaw | Add phase state management to agent |
| **Testing discipline** | OpenClaw can run tests but doesn't enforce them | Add verification gates, test-first checks |
| **Documentation generation** | No structured doc generation | Build templates for planning docs, work logs |
| **Configurable autonomy** | Fixed assistant behavior | Add autonomy level configuration |
| **Work logging** | No decision/reasoning logs | Capture and persist agent reasoning per phase |
| **Developer onboarding** | General assistant onboarding | Rewrite for developer context (repos, git, testing) |
| **Coding-focused prompts** | General assistant system prompts | Rewrite to focus on engineering discipline |

## Complexity Assessment

| Feature Category | Overall Complexity | Risk Level | Dependencies |
|------------------|-------------------|------------|--------------|
| Table Stakes (OpenClaw leverage) | Low | Low | Mostly exists, needs configuration |
| GSD Pipeline Integration | High | Medium | New state management, workflow orchestration |
| Testing Discipline | Medium | Medium | Enforcement logic, verification gates |
| Documentation Generation | Medium | Low | Template-based, well-understood problem |
| Autonomy Configuration | Medium | Medium | User preference management, decision points |
| Browser Research | Low | Low | Exists in OpenClaw, needs workflow integration |
| Developer Onboarding | Low | Low | Rewrite existing onboarding flow |
| Work Logging | Medium | Low | Structured logging, persistence |

## Sources

**AI Developer Agent Capabilities (2026):**
- [Microsoft Copilot 2026: AI Agents & Autonomous Features](https://www.sentisight.ai/what-expect-from-microsoft-copilot-2026/)
- [Best AI Coding Agents for 2026: Real-World Developer Reviews | Faros AI](https://www.faros.ai/blog/best-ai-coding-agents-2026)
- [Windsurf vs Cursor | AI IDE Comparison](https://windsurf.com/compare/windsurf-vs-cursor)

**Table Stakes and Differentiation:**
- [The AI Agent Tools Landscape: 120+ Tools Mapped [2026]](https://www.stackone.com/blog/ai-agent-tools-landscape-2026)
- [AI Copilots vs AI Agents: Difference & Guide (2026)](https://skywork.ai/blog/ai-copilots-vs-ai-agents-2026-comparison/)
- [Cursor vs Windsurf: which is the better AI code editor?](https://www.builder.io/blog/windsurf-vs-cursor)

**GSD Methodology:**
- [GSD Framework: The System Revolutionizing Development with Claude Code](https://pasqualepillitteri.it/en/news/169/gsd-framework-claude-code-ai-development)
- [I Tested GSD Claude Code: Meta-Prompting System That Ships Faster](https://medium.com/@joe.njenga/i-tested-gsd-claude-code-meta-prompting-that-ships-faster-no-agile-bs-ca62aff18c04)

**AI Agent Failures and Anti-Patterns:**
- [9 Critical Failure Patterns of Coding Agents | DAPLab](https://daplab.cs.columbia.edu/general/2026/01/08/9-critical-failure-patterns-of-coding-agents.html)
- [AI vs human code gen report: AI code creates 1.7x more issues](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report)
- [Are bugs and incidents inevitable with AI coding agents? - Stack Overflow](https://stackoverflow.blog/2026/01/28/are-bugs-and-incidents-inevitable-with-ai-coding-agents)

**Testing and Documentation Issues:**
- [As Coders Adopt AI Agents, Security Pitfalls Lurk in 2026](https://www.darkreading.com/application-security/coders-adopt-ai-agents-security-pitfalls-lurk-2026)
- [The Current State of Agentic Software Development | Awesome Testing](https://www.awesome-testing.com/2026/02/ai-coding-2026-hype-vs-reality)
- [AddyOsmani.com - How to write a good spec for AI agents](https://addyosmani.com/blog/good-spec/)

**Engineering Discipline:**
- [Anthropic: 8 agentic coding trends shaping software engineering in 2026](https://tessl.io/blog/8-trends-shaping-software-engineering-in-2026-according-to-anthropics-agentic-coding-report/)
- [Compound Engineering: AI-Assisted Software Development Methodology](https://reading.torqsoftware.com/notes/software/ai-ml/agentic-coding/2026-01-19-compound-engineering-claude-code/)

**Test-Driven Development:**
- [What is test-driven development (TDD)? The complete guide for 2026](https://monday.com/blog/rnd/test-driven-development-tdd/)
- [Getting Feedback from Test-Driven Development and Testing in Production - InfoQ](https://www.infoq.com/news/2026/02/feedback-TDD-production/)
