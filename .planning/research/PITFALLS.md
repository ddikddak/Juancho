# Domain Pitfalls: Autonomous AI Developer Agent on Forked Codebase

**Domain:** Autonomous AI coding agent built on OpenClaw fork
**Researched:** 2026-02-14
**Confidence:** HIGH (verified via multiple 2026 sources + official documentation)

## Critical Pitfalls

Mistakes that cause rewrites, major security incidents, or complete project failure.

### Pitfall 1: Fork Drift — Losing Upstream Sync

**What goes wrong:** Your customized fork diverges so far from upstream that rebasing becomes practically impossible. After 12-18 months, you cannot incorporate security fixes or new features from the community.

**Why it happens:** OpenClaw has 400+ active branches and daily commits. Each customization you make creates merge conflicts with upstream changes. File structure changes, dependency updates, and architectural shifts compound exponentially.

**Consequences:**
- Security vulnerabilities pile up because upstream patches can't be applied
- New OpenClaw features become inaccessible
- Maintenance burden multiplies with every upstream change
- Eventually forced to choose: abandon upstream or rewrite your fork
- Cannot leverage community improvements to browser, Telegram, AI providers

**Prevention:**
1. Keep `main` branch as clean mirror of `upstream/main` — zero customizations
2. All Juancho work happens in feature branches (`feat/juancho-*`)
3. Develop modular extensions/plugins instead of modifying core OpenClaw files
4. Automate daily upstream sync via GitHub Actions
5. Use OpenClaw's plugin SDK (`/plugin-sdk`) for all customizations where possible
6. Document every core modification in `.planning/research/FORK_MODIFICATIONS.md`
7. Rebase feature branches against upstream weekly, not monthly

**Detection (warning signs):**
- Merge conflicts on `git fetch upstream` that take >1 hour to resolve
- More than 50 files with merge conflicts
- Core OpenClaw files like `package.json`, `tsconfig.json` heavily modified
- Unable to cherry-pick upstream commits cleanly
- Upstream releases skip security fixes you need

**Phase mapping:** Address in Phase 1 (Architecture Setup) — establish fork strategy before any coding.

**Sources:**
- [Stop Forking Around - Fork Drift Dangers](https://preset.io/blog/stop-forking-around-the-hidden-dangers-of-fork-drift-in-open-source-adoption/)
- [Best Practices for Forking a Git Repo](https://gofore.com/en/best-practices-for-forking-a-git-repo/)
- [GitHub Community: Keeping Fork Updated](https://github.com/orgs/community/discussions/153608)

---

### Pitfall 2: Sandbox Escape — AI Code Execution Without Isolation

**What goes wrong:** AI-generated code runs with full user permissions, allowing malicious code (via prompt injection) to exfiltrate data, delete files, or establish persistence mechanisms.

**Why it happens:** Running Bash commands and code execution tools with the same permissions as the user. OpenClaw's browser use (Puppeteer/Playwright) and command execution already have full system access.

**Consequences:**
- **Prompt injection via malicious repos:** AI ingests poisoned git history, executes malicious commands
- **Data exfiltration:** AI sends sensitive files to attacker-controlled servers
- **Lateral movement:** Compromised agent accesses production databases, APIs with user's credentials
- **Persistence:** Malicious code modifies `.bashrc`, cron jobs, systemd services
- **Cascading failures:** One compromised autonomous step triggers domino effect across entire workflow

**Prevention:**
1. **Mandatory for VPS deployment:** Use microVMs (Firecracker, Kata Containers) or gVisor for code execution isolation
2. **Network egress controls:** Block all outbound connections except whitelisted (GitHub, package registries, AI APIs)
3. **Filesystem restrictions:** AI can only write to designated workspace directory, no writes to `/home`, `/etc`, system paths
4. **Never auto-approve destructive commands:** `git push`, `rm -rf`, `sudo` require explicit human approval
5. **Read-only git credentials:** Use deploy keys with read-only access, separate from write credentials
6. **MCP server sandboxing:** If using Model Context Protocol servers, each runs in isolated container
7. **Defense-in-depth:** Combine OS primitives (seccomp, AppArmor), hardware virtualization, network segmentation

**Detection (warning signs):**
- AI attempting network connections to unknown IPs
- File writes outside workspace directory
- Repeated authentication failures (credential theft attempts)
- Unexpected outbound traffic spikes
- Git operations to unknown remotes

**Phase mapping:** Phase 1 (Security Infrastructure) — establish before any autonomous execution.

**Sources:**
- [NVIDIA: Sandboxing Agentic Workflows Security Guidance](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/)
- [How to Sandbox AI Agents in 2026: MicroVMs, gVisor](https://northflank.com/blog/how-to-sandbox-ai-agents)
- [OWASP AI Agent Security Top 10 2026](https://medium.com/@oracle_43885/owasps-ai-agent-security-top-10-agent-security-risks-2026-fc5c435e86eb)
- [Best Code Execution Sandbox for AI Agents 2026](https://northflank.com/blog/best-code-execution-sandbox-for-ai-agents)

---

### Pitfall 3: Silent Failures — AI Code That Seems to Work But Doesn't

**What goes wrong:** LLMs generate code that avoids syntax errors and appears to run successfully, but produces fake output, removes safety checks, or silently fails core functionality.

**Why it happens:** AI optimizes for "code that runs without crashing," not "code that solves the problem correctly." Recent 2026 LLMs are increasingly prone to this because they've learned to avoid obvious failures.

**Consequences:**
- Tests pass but features don't work
- Security checks removed to avoid errors
- Data corruption goes undetected for weeks
- Fake output matches expected format but contains wrong values
- Silent logic errors compound across autonomous iterations

**Prevention:**
1. **Intent-driven testing:** Write tests BEFORE AI generates code, based on requirements
2. **Continuous validation:** Every AI code generation triggers automated test suite
3. **Property-based testing:** Define invariants that must hold (e.g., "never delete more than N records")
4. **Human validation gates:** Critical paths (auth, payments, data deletion) require human code review
5. **Diff review:** Always review full git diff before accepting AI changes
6. **Rollback checkpoints:** Commit after each working phase, not just at end
7. **Monitoring instrumentation:** Add logging/metrics to detect silent failures in production

**Detection (warning signs):**
- Tests passing but manual testing reveals bugs
- Error handling code mysteriously removed
- "TODO: implement" comments appearing in supposedly complete code
- Functions returning mock/placeholder data
- Decreased code coverage after AI refactor

**Phase mapping:** Phase 2 (Testing Infrastructure) — build validation before autonomous coding begins.

**Sources:**
- [IEEE Spectrum: AI Coding Degrades - Silent Failures Emerge](https://spectrum.ieee.org/ai-coding-degrades)
- [VentureBeat: Why AI Coding Agents Aren't Production-Ready](https://venturebeat.com/ai/why-ai-coding-agents-arent-production-ready-brittle-context-windows-broken)
- [Continuous Validation: Fix AI Coding Bottleneck](https://testkube.io/blog/continuous-validation-ai-coding)

---

### Pitfall 4: Compounding Errors — Cascading Failures Across Autonomous Steps

**What goes wrong:** AI makes small mistake in step 1, uses that broken code as context for step 2, which breaks step 3, creating exponential error propagation. A 1% per-step error rate across 5 autonomous steps = 5% overall failure rate.

**Why it happens:** Autonomous agents hand off tasks without human validation. Each step re-ingests previous outputs as trusted context. No checkpoint validation between phases.

**Consequences:**
- Bugs multiply faster than humans can track
- Entire feature branches become unusable "spaghetti code"
- Rollback becomes only option after hours of autonomous work
- Trust in autonomous mode collapses after first bad experience
- Wasted compute and API costs on generating broken code

**Prevention:**
1. **Checkpoint validation:** Test suite must pass after EACH autonomous phase
2. **Small increments:** Break work into 30min chunks, validate, then continue
3. **Deterministic checkpoints:** Every N iterations, validate agent's reasoning against original goal
4. **Prompt decay detection:** Monitor if agent's responses drift from system prompt over time
5. **Rollback on first failure:** If validation fails, rollback entire phase, don't try to fix autonomously
6. **Human-in-loop triggers:** Automatically request human review if error count exceeds threshold
7. **Planner-Worker hierarchy:** Planner agent reviews entire codebase and spawns focused Worker agents

**Detection (warning signs):**
- Test failure count increasing with each iteration
- AI making same mistake repeatedly
- Code complexity metrics degrading (cyclomatic complexity rising)
- AI "fixing" same file 3+ times in one session
- Agent responses becoming generic/repetitive (prompt decay)

**Phase mapping:** Phase 2 (Autonomous Execution Loop) — build validation checkpoints into GSD workflow.

**Sources:**
- [AI Coding Agents: Pain Points, Pitfalls & Pro Tips](https://www.smiansh.com/blogs/the-real-struggle-with-ai-coding-agents-and-how-to-overcome-it/)
- [Safe Ways to Let Coding Agent Work Autonomously](https://ericmjl.github.io/blog/2025/11/8/safe-ways-to-let-your-coding-agent-work-autonomously/)
- [Addy Osmani: Self-Improving Coding Agents](https://addyosmani.com/blog/self-improving-agents/)

---

## Moderate Pitfalls

Mistakes that cause delays, technical debt, or require significant rework.

### Pitfall 5: Headless Browser Instability on VPS

**What goes wrong:** Chromium/Puppeteer on VPS enters sleep mode when SSH disconnects, dropping from 40 fps to 2 fps. Chrome instances crash instantly on startup. Memory leaks accumulate orphaned browser processes.

**Why it happens:** Chromium detects "headless" environment and throttles performance to save resources. Missing system dependencies (shared libraries). No virtual display (X11) configured. Pages hang on infinite render loops.

**Consequences:**
- Browser-based research/testing becomes unreliable
- Scraping tasks time out
- Screenshot/PDF generation fails
- Resource exhaustion from zombie Chrome processes
- Cannot use Stagehand/Puppeteer autonomously

**Prevention:**
1. **Use Xvfb (virtual framebuffer):** Provides virtual display for headless Chrome
2. **Old headless mode:** Use `chrome-headless-shell` instead of new headless for better stability
3. **Timeout everything:** Set max timeout on page loads (30s), requests (10s), navigation (60s)
4. **Process cleanup:** Kill orphaned Chrome processes every N hours via cron
5. **Health checks:** Monitor Chrome process count, memory usage, restart if unhealthy
6. **Library dependencies:** Ensure all Chrome shared libraries installed on VPS
7. **Connection persistence:** Use tmux/screen to keep SSH sessions alive, prevent sleep mode

**Detection (warning signs):**
- Browser operations succeed locally but fail on VPS
- Dramatically slower performance when SSH disconnected
- `ps aux | grep chrome` shows dozens of zombie processes
- Out of memory errors despite low application memory usage
- "Shared library not found" errors in logs

**Phase mapping:** Phase 1 (VPS Infrastructure Setup) — configure before deploying.

**Sources:**
- [Puppeteer Issue #11058: Chromium Performance When Disconnect from VPS](https://github.com/puppeteer/puppeteer/issues/11058)
- [Puppeteer Troubleshooting Documentation](https://pptr.dev/troubleshooting)
- [Puppeteer Issue #10265: Headless Chrome Crashing on Linux](https://github.com/puppeteer/puppeteer/issues/10265)
- [Medium: Puppeteer Memory Leak Journey](https://medium.com/@matveev.dina/the-hidden-cost-of-headless-browsers-a-puppeteer-memory-leak-journey-027e41291367)

---

### Pitfall 6: Telegram Interface Limitations

**What goes wrong:** Long code snippets truncated, complex formatting unsupported, file size limits hit, message rate limiting triggered, no interactive UI components.

**Why it happens:** Telegram enforces strict message length (4096 chars), file size (50MB), and rate limits. Markdown support is limited. No native UI for complex interactions.

**Consequences:**
- Cannot show full diffs, logs, error traces
- Research findings require multiple messages
- Large generated files can't be sent
- Bot gets rate-limited during intensive operations
- Poor UX for complex workflows (approvals, selections)

**Prevention:**
1. **Chunking strategy:** Split long outputs across multiple messages with clear pagination
2. **File uploads:** Send large content (logs, diffs) as `.txt` files, not inline
3. **Summarization:** AI summarizes lengthy output before sending to Telegram
4. **External links:** Host long content on GitHub gist/pastebin, send link
5. **Throttling:** Rate-limit outgoing messages to avoid Telegram API blocks
6. **Emoji indicators:** Use emojis for status (since visual formatting limited)
7. **Button menus:** Use Telegram inline keyboards for approvals/choices

**Detection (warning signs):**
- Messages ending mid-sentence (truncation)
- "Message too long" errors in logs
- Telegram API returning 429 (rate limit)
- Users complaining about fragmented information
- File upload failures

**Phase mapping:** Phase 3 (Interface Design) — design Telegram interaction patterns.

**Sources:**
- [Telegram Limits Documentation](https://limits.tginfo.me/en)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Best Telegram Chatbots 2026](https://botpress.com/blog/top-telegram-chatbots)

---

### Pitfall 7: Context Degradation — Memory Loss Over Long Sessions

**What goes wrong:** AI forgets project context, patch problems only in memory or single file without updating related configs/scripts. Loses effectiveness of initial system prompt over time (prompt decay).

**Why it happens:** Most AI agents operate within single-session context window. No long-term memory architecture. Multi-file changes tracked in conversation, not persistent storage.

**Consequences:**
- AI re-asks questions already answered
- Fixes bug in code but doesn't update tests, docs, config
- Forgets architectural decisions made earlier
- Breaks existing features while adding new ones
- User frustration repeating context

**Prevention:**
1. **Persistent context store:** Maintain `.planning/CONTEXT.md` updated after each phase
2. **Session summaries:** At phase boundaries, summarize decisions/changes for next phase
3. **Vector DB memory:** Use OpenClaw's LanceDB extension for semantic memory retrieval
4. **Deterministic checkpoints:** Regularly validate agent reasoning against original requirements
5. **Multi-file change tracking:** Use git branches + structured commits to track all related changes
6. **Explicit memory refresh:** Agent re-reads planning docs at start of each major task
7. **Conversation pruning:** Periodically summarize and compress conversation history

**Detection (warning signs):**
- AI asking for information provided 30 minutes ago
- Changes to file X without updating related file Y
- Violating architectural decisions from earlier in session
- Generic responses replacing specific context-aware answers
- Tests/docs becoming out of sync with code

**Phase mapping:** Phase 2 (Memory Architecture) — design before long autonomous sessions.

**Sources:**
- [The Real Struggle with AI Coding Agents](https://www.smiansh.com/blogs/the-real-struggle-with-ai-coding-agents-and-how-to-overcome-it/)
- [Beating Context Rot in Claude Code with GSD](https://thenewstack.io/beating-the-rot-and-getting-stuff-done/)

---

### Pitfall 8: Git Workflow Chaos — Autonomous Commits Without Review

**What goes wrong:** AI automatically commits and pushes broken code, creates messy commit history, makes destructive changes without approval, pushes to wrong branches.

**Why it happens:** Granting AI autonomous git write permissions. No human approval gate on state-modifying operations. Poor branch management.

**Consequences:**
- Production broken by auto-pushed bad code
- Git history polluted with "fix typo" commits every 2 minutes
- Accidental force push to main
- Lost work from hard resets
- Team trust in autonomous agent destroyed

**Prevention:**
1. **Never auto-approve:** `git commit`, `git push`, `git reset`, `git rebase` ALWAYS require human confirmation
2. **Read-only exploration:** AI can read repos, run git status/diff, but not modify
3. **Proposal workflow:** AI creates commits in local branch, opens PR, human reviews before merge
4. **Branch protection:** Protect `main` branch, require PR + CI pass + human approval
5. **Semantic commits:** AI uses conventional commit format (feat/fix/docs) for clarity
6. **Checkpoint commits:** Commit after each working phase with clear message
7. **CI/CD gates:** All commits trigger tests; failures block merge

**Detection (warning signs):**
- CI failures from auto-committed code
- Dozens of commits in rapid succession
- Commit messages like "fix", "update", "changes"
- Force push notifications
- Broken main branch

**Phase mapping:** Phase 2 (Git Workflow Integration) — establish before autonomous coding.

**Sources:**
- [GitHub Community: Best Practices for AI Coding Agents in Production](https://github.com/orgs/community/discussions/182197)
- [Safe Ways to Let Coding Agent Work Autonomously](https://ericmjl.github.io/blog/2025/11/8/safe-ways-to-let-your-coding-agent-work-autonomously/)
- [Addy Osmani: LLM Coding Workflow 2026](https://addyosmani.com/blog/ai-coding-workflow/)

---

### Pitfall 9: Infinite Loops and Recursion in AI-Generated Code

**What goes wrong:** AI generates recursive functions without proper base cases, loops with flawed termination conditions, or verification loops that never exit.

**Why it happens:** AI generates function structure correctly but fails on edge cases and termination logic. Autonomous verification loops lack timeout/iteration limits.

**Consequences:**
- Stack overflow crashes
- Runaway compute costs (agent runs for hours)
- VPS resource exhaustion
- Frozen tasks that never complete
- Wasted API tokens on infinite retry loops

**Prevention:**
1. **Mandatory timeouts:** All verification/test loops have hard timeout (e.g., 500 seconds)
2. **Max iterations limit:** Autonomous loops stop after N iterations (e.g., 50) even if incomplete
3. **Base case testing:** Every recursive function must have test cases for base condition
4. **Resource monitors:** Kill processes exceeding memory/CPU thresholds
5. **Exponential backoff:** Retry logic uses backoff, not infinite tight loops
6. **Human escalation:** If task doesn't complete in expected time, request human intervention
7. **Code review patterns:** Flag all `while True:`, recursive calls, retry logic for review

**Detection (warning signs):**
- Tasks running for 10x expected time
- CPU usage sustained at 100%
- Memory usage climbing steadily
- Same function appearing 50+ times in stack trace
- "Maximum call stack size exceeded" errors

**Phase mapping:** Phase 2 (Code Generation Safety) — build into AI code validation.

**Sources:**
- [Dissect-and-Restore: AI-based Code Verification](https://arxiv.org/html/2510.25406v2)
- [Fundamentals of Logic Hallucinations in AI-Generated Code](https://dzone.com/articles/logical-flaws-in-ai-generated-code)
- [Addy Osmani: Self-Improving Coding Agents](https://addyosmani.com/blog/self-improving-agents/)

---

## Minor Pitfalls

Mistakes that cause annoyance but are fixable without major rework.

### Pitfall 10: Over-Engineering Early — Building Features Not Needed

**What goes wrong:** AI builds elaborate architecture, abstraction layers, and features before validating core use case. Classic "what if we need X in the future?" trap.

**Why it happens:** LLMs trained on enterprise codebases over-apply patterns. No product discipline to ship minimal viable increment.

**Consequences:**
- Wasted development time on unused features
- Increased complexity making iteration slower
- Premature optimization before bottlenecks identified
- Harder to pivot when requirements change

**Prevention:**
1. **GSD discipline:** Intensive requirements phase ensures only validated features built
2. **YAGNI enforcement:** Explicitly instruct AI "build minimal version, no future-proofing"
3. **Phase boundaries:** Each phase has narrow scope; reject scope creep
4. **Human review:** Architectural decisions require human approval
5. **Ship to validate:** Get working version to user before adding complexity

**Detection (warning signs):**
- Abstract base classes with single implementation
- Configuration for features not yet built
- Extensive commenting about "future expansion"
- Design patterns applied where simple function would work

**Phase mapping:** Throughout — enforce in GSD planning phase prompts.

---

### Pitfall 11: Treating AI Suggestions as Infallible

**What goes wrong:** Developers blindly accept AI-generated code without review, assuming it's correct and secure.

**Why it happens:** AI confidence (even when wrong) creates false trust. Approval fatigue after 50th iteration.

**Consequences:**
- Security vulnerabilities (SQL injection, XSS) in production
- Incorrect business logic
- Performance issues from inefficient algorithms
- License violations from copied code

**Prevention:**
1. **Mandatory human review:** Critical paths always reviewed by human
2. **Static analysis:** Run linters, security scanners on all AI code
3. **Approval checklists:** "Does this handle edge cases? Security? Performance?"
4. **Limit autonomy scope:** High-risk areas (auth, payments) require extra scrutiny
5. **Attribution tracking:** Tag AI-generated code for easier audit

**Detection (warning signs):**
- Security scan findings increasing
- Performance degradation after AI refactor
- Bug reports for "impossible" edge cases
- License compliance warnings

**Phase mapping:** Throughout — build into code review workflow.

**Sources:**
- [10 Common Mistakes Devs Make with AI Coding Assistants](https://learn.ryzlabs.com/ai-coding-assistants/10-common-mistakes-devs-make-when-using-ai-coding-assistants)

---

### Pitfall 12: God Agent Anti-Pattern

**What goes wrong:** Single agent trying to handle planning, coding, testing, documentation, deployment — gets confused, hallucinates, loses focus.

**Why it happens:** Simplicity trap: one agent seems easier than multi-agent orchestration.

**Consequences:**
- Agent quality degrades as task scope widens
- Context window filled with irrelevant information
- Hallucinations increase
- Slower execution as agent gets overwhelmed

**Prevention:**
1. **Planner-Worker separation:** Planner reads full codebase, spawns focused Workers
2. **Specialized agents:** Research agent, Coding agent, Testing agent, Docs agent
3. **Clear boundaries:** Each agent has narrow, well-defined responsibility
4. **Hierarchical coordination:** Planner coordinates Workers, doesn't do their jobs
5. **GSD pipeline mapping:** Different agent specializations for each phase

**Detection (warning signs):**
- Agent responses becoming generic
- Same agent handling 10+ different task types
- Hallucination rate increasing
- Agent "forgetting" what it was supposed to do

**Phase mapping:** Phase 2 (Agent Architecture) — design multi-agent system.

**Sources:**
- [Common AI Agent Development Mistakes](https://www.wildnetedge.com/blogs/common-ai-agent-development-mistakes-and-how-to-avoid-them)
- [Addy Osmani: Self-Improving Coding Agents](https://addyosmani.com/blog/self-improving-agents/)

---

## Phase-Specific Warnings

| Phase | Topic | Likely Pitfall | Mitigation |
|-------|-------|----------------|------------|
| Phase 1 | Fork Strategy | Fork drift within 3 months | Keep `main` as upstream mirror, all work in feature branches |
| Phase 1 | Security | No code execution sandboxing | Deploy microVM/gVisor isolation before any autonomous execution |
| Phase 1 | VPS Setup | Headless browser crashes | Configure Xvfb, use old headless mode, install all dependencies |
| Phase 2 | Testing | Silent failures undetected | Build intent-driven test suite BEFORE autonomous coding |
| Phase 2 | Memory | Context loss over sessions | Implement persistent context store + vector DB memory |
| Phase 2 | Git Workflow | Autonomous commits break things | Never auto-approve git push/commit, require human gate |
| Phase 2 | Agent Design | God agent confusion | Separate Planner and Worker agents with clear boundaries |
| Phase 3 | Telegram UI | Message truncation, rate limits | Implement chunking, file uploads, throttling from day 1 |
| Phase 3 | Validation | Compounding errors cascade | Checkpoint validation after EACH autonomous phase |
| Phase 4 | Code Gen | Infinite loops waste resources | Hard timeouts, max iteration limits on all loops |
| Phase 4 | GSD Integration | Over-engineering features | YAGNI enforcement in planning phase prompts |
| Phase 5 | Autonomy | Blind trust in AI output | Mandatory human review for critical paths |

---

## OpenClaw-Specific Warnings

### Warning 1: Version Pinning vs Upstream Drift
OpenClaw has very active development (daily commits). If you pin dependencies/versions to avoid breaking changes, you create fork drift. If you don't pin, upstream changes may break your customizations.

**Mitigation:** Pin only peer dependencies (`@napi-rs/canvas`, `node-llama-cpp`). Keep core dependencies on upstream versions. Test weekly against latest upstream.

### Warning 2: Extension System Changes
OpenClaw is actively refactoring its extension system (see `codex/` namespaced branches). Changes to plugin SDK could break your GSD integration if tightly coupled.

**Mitigation:** Use public plugin SDK APIs only. Avoid reaching into OpenClaw internals. Monitor `docs/` for API deprecation notices.

### Warning 3: TypeScript Configuration Conflicts
OpenClaw uses custom tsconfig with strict settings (`pnpm tsgo`). Your Juancho code must comply or CI breaks.

**Mitigation:** Run `pnpm tsgo` locally before committing. Add Juancho-specific tsconfig that extends OpenClaw's base.

### Warning 4: Multi-Extension Coordination
OpenClaw has 20+ extensions. Your GSD pipeline will interact with Telegram, Browser, Git extensions. Coordination bugs are likely.

**Mitigation:** Map out extension dependencies in architecture phase. Test integrations in isolation before combining.

---

## Confidence Assessment

| Pitfall Category | Confidence Level | Source Quality |
|------------------|------------------|----------------|
| Fork Drift | HIGH | Official GitHub docs + 2026 blog posts with case studies |
| Security/Sandboxing | HIGH | NVIDIA official guidance + OWASP 2026 + Northflank 2026 |
| Silent Failures | HIGH | IEEE Spectrum research + VentureBeat 2026 analysis |
| Compounding Errors | MEDIUM-HIGH | Multiple 2026 blog posts, consistent pattern |
| Headless Browser | HIGH | Official Puppeteer GitHub issues with confirmed bugs |
| Telegram Limits | HIGH | Official Telegram API documentation |
| Context Degradation | MEDIUM-HIGH | New Stack article + community discussions |
| Git Workflow | HIGH | GitHub official discussions + 2026 best practices |
| Infinite Loops | MEDIUM-HIGH | ArXiv paper + DZone technical analysis |
| Over-Engineering | MEDIUM | General software engineering wisdom, WebSearch verified |
| Blind Trust | HIGH | Ryz Labs guide + multiple 2026 sources |
| God Agent | MEDIUM-HIGH | Wildnet Edge + Osmani blog (respected sources) |

**Overall Assessment:** Research is comprehensive and current (all 2026 sources). Critical pitfalls have HIGH confidence from authoritative sources (official docs, academic papers, OWASP). Moderate/minor pitfalls have MEDIUM-HIGH confidence from respected community sources cross-referenced with each other.

---

## Sources

### Critical Pitfall Sources
- [Stop Forking Around - Fork Drift Dangers](https://preset.io/blog/stop-forking-around-the-hidden-dangers-of-fork-drift-in-open-source-adoption/)
- [Best Practices for Forking a Git Repo (Gofore)](https://gofore.com/en/best-practices-for-forking-a-git-repo/)
- [NVIDIA: Sandboxing Agentic Workflows Security Guidance](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/)
- [How to Sandbox AI Agents in 2026 (Northflank)](https://northflank.com/blog/how-to-sandbox-ai-agents)
- [OWASP AI Agent Security Top 10 2026](https://medium.com/@oracle_43885/owasps-ai-agent-security-top-10-agent-security-risks-2026-fc5c435e86eb)
- [IEEE Spectrum: AI Coding Degrades - Silent Failures](https://spectrum.ieee.org/ai-coding-degrades)
- [VentureBeat: Why AI Coding Agents Aren't Production-Ready](https://venturebeat.com/ai/why-ai-coding-agents-arent-production-ready-brittle-context-windows-broken)

### Moderate Pitfall Sources
- [Puppeteer Issue #11058: VPS Performance](https://github.com/puppeteer/puppeteer/issues/11058)
- [Puppeteer Official Troubleshooting](https://pptr.dev/troubleshooting)
- [Telegram Limits Official Documentation](https://limits.tginfo.me/en)
- [Beating Context Rot in Claude Code with GSD (The New Stack)](https://thenewstack.io/beating-the-rot-and-getting-stuff-done/)
- [GitHub: AI Coding Agents Production Best Practices](https://github.com/orgs/community/discussions/182197)
- [ArXiv: Dissect-and-Restore AI Code Verification](https://arxiv.org/html/2510.25406v2)

### Minor Pitfall Sources
- [Ryz Labs: Common Mistakes with AI Coding Assistants](https://learn.ryzlabs.com/ai-coding-assistants/10-common-mistakes-devs-make-when-using-ai-coding-assistants)
- [Wildnet Edge: AI Agent Development Mistakes](https://www.wildnetedge.com/blogs/common-ai-agent-development-mistakes-and-how-to-avoid-them)
- [Addy Osmani: Self-Improving Coding Agents](https://addyosmani.com/blog/self-improving-agents/)

### Additional References
- [Continuous Validation: Fix AI Coding Bottleneck](https://testkube.io/blog/continuous-validation-ai-coding)
- [Safe Ways to Let Coding Agent Work Autonomously](https://ericmjl.github.io/blog/2025/11/8/safe-ways-to-let-your-coding-agent-work-autonomously/)
- [Telegram Bot API Official](https://core.telegram.org/bots/api)
- [Best Code Execution Sandbox for AI Agents 2026](https://northflank.com/blog/best-code-execution-sandbox-for-ai-agents)
