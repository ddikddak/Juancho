# Phase 3: Git & Testing Workflow - Research

**Researched:** 2026-02-15
**Domain:** Git automation, testing discipline, browser automation for VPS
**Confidence:** HIGH

## Summary

Phase 3 adds git workflow automation, test-first discipline, and headless browser verification to the GSD pipeline. The key insight: **most requirements are already satisfied** by existing OpenClaw capabilities and Phase 2 workspace skills. OpenClaw already has git command execution via Bash tool, browser automation via Playwright, and Xvfb-based headless browser infrastructure. The execute-phase skill already implements per-task conventional commits. The verify-phase skill already has 3-level artifact verification.

**What's missing:** Feature branch management, pull request generation, test enforcement gates, and build verification commands in skill behavioral instructions. This is NOT new tooling—it's ENHANCING existing skills with additional git/testing instructions.

**Primary recommendation:** Enhance existing execute-phase and verify-phase skills with git branching, PR creation, test gates, and build verification behavioral instructions. Add test-first discipline to plan-phase templates. No new skills needed—this is pure instruction augmentation within the 20k char limits.

## Standard Stack

### Core (Already In Place)

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Git | System | Version control, branching, commits | Industry standard VCS, OpenClaw runs git via Bash tool |
| Vitest | 4.0.18 | Test framework (already in OpenClaw) | Fast, modern, ESM-native test runner for TypeScript/JavaScript |
| Playwright | 1.58.2 | Browser automation (already in OpenClaw) | Headless browser testing, OpenClaw uses playwright-core |
| GitHub CLI (`gh`) | Latest | PR creation, issue management | Official GitHub tool, already used in OpenClaw's `.agents/skills/` for PR workflows |
| Xvfb | System (apt) | Virtual X11 display | Industry standard for headless GUI apps on Linux VPS |

**Rationale:** OpenClaw already has ALL required dependencies. Git runs via Bash tool. Vitest is the test framework (see `package.json`, `vitest.config.ts`). Playwright provides browser automation. GitHub CLI already used in `.agents/skills/review-pr/`, `.agents/skills/prepare-pr/`, `.agents/skills/merge-pr/`. Xvfb configured in `Dockerfile.sandbox-browser` and `scripts/sandbox-browser-entrypoint.sh`.

**Installation:** Already installed in OpenClaw. For new projects created by Juancho, test framework depends on project type (Vitest for Node/TS, pytest for Python, Jest legacy codebases, etc.).

### Supporting (Behavioral Instructions, Not Libraries)

| Pattern | Purpose | Implementation |
|---------|---------|----------------|
| Conventional Commits | Standardized commit format | Instructions in execute-phase skill |
| Test-First Discipline | Run tests after each change | Instructions in execute-phase, verify-phase skills |
| Branch Naming Conventions | feat/, fix/, chore/ prefixes | Instructions in execute-phase skill |
| PR Templates | Consistent PR descriptions | Instructions in execute-phase skill (gh pr create) |

**Key Insight:** These are NOT libraries to install. These are BEHAVIORAL INSTRUCTIONS in workspace skills that tell the AI agent how to format commits, when to run tests, how to name branches, etc.

## Architecture Patterns

### Git Workflow Pattern (Per-Phase Branching)

**What:** Each phase creates a feature branch, commits tasks atomically, creates PR when phase completes.

**When to use:** Every phase execution (execute-phase skill).

**Structure:**
```
main
├── feat/phase-01-developer-identity  (Phase 1 branch)
│   └── commits: task-by-task with conventional format
├── feat/phase-02-gsd-pipeline        (Phase 2 branch)
│   └── commits: task-by-task with conventional format
└── feat/phase-03-git-testing         (Phase 3 branch, current)
    └── commits: task-by-task with conventional format
```

**Implementation Location:** execute-phase skill instructions (behavioral, not code).

**Example Instructions (what goes in skill):**
```markdown
### Git Workflow Per Phase

Before executing first plan in phase:
1. Create feature branch: `git checkout -b feat/phase-{XX}-{slug}`
2. Branch naming: feat/ for new features, fix/ for bug fixes, chore/ for tooling
3. Commit each task atomically: `{type}({phase}-{plan}): {description}`

After all plans complete (in step 11):
1. Push branch: `git push -u origin feat/phase-{XX}-{slug}`
2. Create PR: `gh pr create --title "Phase {X}: {Name}" --body "{summary}"`
3. Include PR URL in phase completion report
```

### Test-First Discipline Pattern

**What:** Run tests immediately after each code change, block progress if tests fail.

**When to use:** Every task execution (execute-phase skill), every phase transition (verify-phase skill).

**Workflow:**
```
1. Write/modify code (task action)
2. Run verification commands (includes tests)
3. If tests fail → surface error, debug, fix
4. If tests pass → commit task
5. Repeat for next task
```

**Implementation Location:** execute-phase skill task execution protocol, verify-phase skill verification steps.

**Example Instructions (what goes in skill):**
```markdown
### Test Enforcement

After each task action:
1. Run `<verify>` commands (must include test command)
2. If ANY command fails (including tests):
   - Surface full error output
   - Debug the failure (read logs, check code)
   - Fix the issue
   - Re-run verification
3. NEVER suppress test failures
4. NEVER commit without passing tests

Phase transition blocked if test suite fails.
```

### Build Verification Pattern

**What:** Ensure project builds successfully before phase completion.

**When to use:** verify-phase skill, before marking phase complete.

**Check Sequence:**
```
1. Run project build command (npm run build, cargo build, go build, etc.)
2. Check build artifacts exist
3. Verify no build warnings (or document acceptable ones)
4. Only if build succeeds → allow phase transition
```

**Implementation Location:** verify-phase skill verification steps.

### Browser Testing Pattern (Headless on VPS)

**What:** Test built web apps via browser to verify they actually work, using Xvfb for headless mode on VPS.

**When to use:** Web app projects, verify-phase human verification checklist.

**Pattern:**
```
1. Xvfb already running (configured in OpenClaw sandbox-browser)
2. Use Playwright via OpenClaw's browser tool
3. Navigate to local dev server (e.g., http://localhost:3000)
4. Verify UI elements exist, interactions work
5. Take screenshots for human verification if needed
```

**Implementation Location:** verify-phase skill human verification tasks.

**OpenClaw Integration:** Browser tool already available, Xvfb configured in `Dockerfile.sandbox-browser`:
```dockerfile
# From Dockerfile.sandbox-browser (lines 20-21)
xvfb \
# From scripts/sandbox-browser-entrypoint.sh (line 17)
Xvfb :1 -screen 0 1280x800x24 -ac -nolisten tcp &
```

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Commit message format | Custom commit parser | Conventional Commits spec | Standardized, tooling support, semantic versioning integration |
| PR description generation | Custom template engine | `gh pr create --body "$(cat <<'EOF' ... EOF)"` | GitHub CLI built-in, heredoc for multi-line |
| Test runner per language | Custom test harness | Vitest (Node/TS), pytest (Python), cargo test (Rust), go test (Go) | Language ecosystems have mature test frameworks |
| Headless browser on VPS | Custom display emulation | Xvfb + Playwright | Industry standard, OpenClaw already configured |
| Branch naming enforcement | Custom git hooks | Behavioral instructions in skills | Skills are prompts—AI follows instructions without code |

**Key insight:** Phase 3 is about INSTRUCTING the AI agent's behavior, not building tools. OpenClaw already has the tools (git, gh, vitest, playwright, xvfb). We just need to tell the agent WHEN and HOW to use them via skill instructions.

## Common Pitfalls

### Pitfall 1: Thinking New Tools Are Needed

**What goes wrong:** Assuming Phase 3 requires new code/libraries/skills.

**Why it happens:** Requirements list looks like new features ("git commits per phase", "test enforcement").

**How to avoid:** Recognize these are BEHAVIORAL requirements satisfied by INSTRUCTIONS in existing skills. execute-phase already commits. We just need to enhance commit instructions with branching/PR creation. verify-phase already verifies. We just need to add test enforcement gates.

**Warning signs:** Planning to create new skills, writing new git wrapper code, building custom test runners.

**Correct approach:** Enhance existing skills' behavioral instructions within their 20k char limits.

### Pitfall 2: Suppressing Test Failures

**What goes wrong:** Agent sees test failure, continues anyway, commits broken code.

**Why it happens:** Default behavior might be to report failure and move on.

**How to avoid:** Explicit instructions in execute-phase: "NEVER commit without passing tests. If tests fail, debug and fix."

**Warning signs:** Tasks marked complete despite test failures, commits with failing tests.

**Verification:** verify-phase must check test suite status before allowing phase transition.

### Pitfall 3: Committing Without Running Tests

**What goes wrong:** Code committed, tests not run, failures discovered later.

**Why it happens:** Test command not included in task `<verify>` section.

**How to avoid:** plan-phase templates must include test commands in every task's `<verify>` section. execute-phase must enforce running all verify commands before commit.

**Warning signs:** `<verify>` sections with only `ls` or `grep`, no test commands.

**Template requirement:** plan-phase skill should include test command template in task examples.

### Pitfall 4: Overcomplicating Branch Strategy

**What goes wrong:** Complex branching (develop, staging, release branches), merge conflicts, confusion.

**Why it happens:** Copying enterprise git workflows into small project.

**How to avoid:** Simple strategy: one feature branch per phase, merge to main via PR when phase completes.

**Warning signs:** Plans referencing develop branch, release branches, hotfix branches.

**Correct approach:** feat/phase-{XX}-{slug} branch per phase, squash merge to main.

### Pitfall 5: Treating Xvfb as Optional

**What goes wrong:** Browser tests run headless without Xvfb, get throttled/detected as bots.

**Why it happens:** Not understanding Xvfb provides "headed" mode on headless server.

**How to avoid:** Document that Xvfb is ALREADY configured in OpenClaw, verify DISPLAY environment variable set.

**Warning signs:** Browser tests flaky on VPS, screenshots show throttling artifacts.

**Verification:** Check `DISPLAY=:1` in sandbox-browser environment, Xvfb process running.

## Code Examples

### Conventional Commit Format (In Skills)

```bash
# From execute-phase skill (task execution protocol)
# Commit format: {type}({phase}-{plan}): {description}

# Examples:
git commit -m "feat(03-01): add feature branch creation to execute-phase"
git commit -m "test(03-01): add test enforcement to verify-phase"
git commit -m "docs(03-02): update TOOLS.md with git conventions"
git commit -m "fix(03-01): handle test failures by debugging before retry"
git commit -m "refactor(03-02): simplify PR description generation logic"
```

**Commit Types:**
- `feat`: New feature (correlates with MINOR in semver)
- `fix`: Bug fix (correlates with PATCH in semver)
- `docs`: Documentation only
- `test`: Adding/updating tests
- `refactor`: Code change that neither fixes bug nor adds feature
- `perf`: Performance improvement
- `chore`: Build process, tooling changes

**Scope Format:** `{phase}-{plan}` or component name. Phase-plan scope enables tracking which phase/plan the commit belongs to.

**Source:** [Conventional Commits Specification](https://www.conventionalcommits.org/en/v1.0.0/)

### Feature Branch Creation (In execute-phase Skill)

```bash
# In execute-phase skill workflow, before first plan execution:

PHASE_NUMBER="03"
PHASE_NAME="git-testing-workflow"
BRANCH_NAME="feat/phase-${PHASE_NUMBER}-${PHASE_NAME}"

# Check if already on feature branch
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

if [[ "$CURRENT_BRANCH" == "main" ]] || [[ "$CURRENT_BRANCH" == "master" ]]; then
  # Create and checkout feature branch
  git checkout -b "$BRANCH_NAME"
  echo "Created feature branch: $BRANCH_NAME"
else
  echo "Already on branch: $CURRENT_BRANCH (skipping branch creation)"
fi
```

**Branch Naming Convention:**
- `feat/phase-{XX}-{slug}` - Feature phases
- `fix/phase-{XX}-{slug}` - Bug fix phases
- `chore/phase-{XX}-{slug}` - Tooling/infrastructure phases

**Integration Point:** execute-phase skill, step 5 (before wave execution), check current branch, create if on main.

### PR Creation (In execute-phase Skill)

```bash
# In execute-phase skill workflow, step 11 (after phase completion):

PHASE_NUMBER="03"
PHASE_NAME="Git & Testing Workflow"
BRANCH_NAME="feat/phase-${PHASE_NUMBER}-git-testing-workflow"

# Push branch to remote
git push -u origin "$BRANCH_NAME"

# Create PR with summary from VERIFICATION.md and SUMMARY.md files
gh pr create --title "Phase ${PHASE_NUMBER}: ${PHASE_NAME}" --body "$(cat <<'EOF'
## Summary

Phase ${PHASE_NUMBER} adds git workflow automation, test enforcement, and browser verification to the GSD pipeline.

### Deliverables

- Feature branches created per phase
- Conventional commits with phase-plan scope
- Test-first discipline enforced
- Build verification before phase transition
- Browser testing verified with Xvfb headless mode

### Verification

- All tests pass (verified by verify-phase)
- Build succeeds
- Browser automation tested on VPS

### Changes

$(cat .planning/phases/${PHASE_NUMBER}-*/0${PHASE_NUMBER}-VERIFICATION.md | grep -A 10 "## Goal Achievement")

## Test Plan

- [ ] All unit tests pass: \`npm test\`
- [ ] Build succeeds: \`npm run build\`
- [ ] Phase verification passed: see VERIFICATION.md
- [ ] Browser tests work on VPS (manual verification)

---

Generated by Juancho GSD Pipeline
EOF
)"
```

**Integration Point:** execute-phase skill, step 11 (git commit phase completion), add PR creation after ROADMAP.md update.

### Test Enforcement (In execute-phase Skill)

```bash
# In execute-phase skill, task execution protocol (step 6.2):

# After executing task action, run verification commands
for cmd in "${VERIFY_COMMANDS[@]}"; do
  echo "Running verification: $cmd"

  if ! eval "$cmd"; then
    echo "ERROR: Verification failed: $cmd"
    echo "Output above shows the failure. Debugging..."

    # Surface error, DO NOT suppress
    # Agent should read error output, understand failure, fix code
    # DO NOT PROCEED to commit

    # Return checkpoint with failure details
    cat <<EOF
## CHECKPOINT: Verification Failed

**Task:** ${TASK_NUMBER} - ${TASK_NAME}
**Failed Command:** ${cmd}

**Error Output:** (see above)

**What to do:**
1. Read the error output carefully
2. Identify the root cause
3. Fix the code/tests
4. Re-run verification commands
5. Only commit when ALL verification passes

**DO NOT:**
- Skip this verification
- Commit broken code
- Suppress test failures

Type "retry" when code is fixed, or describe the issue if stuck.
EOF
    exit 1
  fi
done

# All verifications passed, safe to commit
git add ${MODIFIED_FILES}
git commit -m "${COMMIT_MESSAGE}"
```

**Integration Point:** execute-phase skill, task execution protocol, replace "Run verification commands" section with failure handling.

### Build Verification (In verify-phase Skill)

```bash
# In verify-phase skill, verification steps (before status determination):

# Detect project build command
if [[ -f "package.json" ]]; then
  BUILD_CMD="npm run build"
elif [[ -f "Cargo.toml" ]]; then
  BUILD_CMD="cargo build"
elif [[ -f "go.mod" ]]; then
  BUILD_CMD="go build"
elif [[ -f "setup.py" ]] || [[ -f "pyproject.toml" ]]; then
  BUILD_CMD="python -m build"
else
  echo "No recognized build system, skipping build verification"
  BUILD_CMD=""
fi

# Run build if applicable
if [[ -n "$BUILD_CMD" ]]; then
  echo "Running build verification: $BUILD_CMD"

  if ! $BUILD_CMD; then
    echo "ERROR: Build failed"
    echo "Phase verification BLOCKED - build must succeed before phase transition"
    STATUS="gaps_found"
    # Add to gaps list
  else
    echo "Build verification passed"
  fi
fi
```

**Integration Point:** verify-phase skill, after artifact/link verification, before status determination.

### Browser Testing with Xvfb (In verify-phase Skill)

```bash
# In verify-phase skill, human verification checklist (for web app projects):

# Xvfb already running in OpenClaw sandbox-browser
# DISPLAY=:1 already set in environment

# Example browser test (using OpenClaw's browser tool via Task)
# This is BEHAVIORAL - agent uses browser tool, not custom code

# In VERIFICATION.md human verification checklist:
cat <<EOF
## Human Verification Required

### Web App Browser Test

1. Start dev server: \`npm run dev\` (or project-specific command)
2. Use browser tool to navigate to http://localhost:3000
3. Verify:
   - [ ] Homepage loads without errors
   - [ ] Navigation works (click links)
   - [ ] Forms submit successfully
   - [ ] No console errors in browser DevTools

**Browser Environment:**
- Headless: Xvfb virtual display (DISPLAY=:1)
- Browser: Chromium via Playwright
- Screenshots available for verification

Type "approved" when browser tests pass, or describe issues.
EOF
```

**Integration Point:** verify-phase skill, human verification section, add browser testing checklist for web projects.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Manual commits per plan | Automated commits per task | Phase 3 (this phase) | Granular git history, easier rollback |
| Tests run manually | Tests enforced in task verification | Phase 3 (this phase) | Prevents broken code commits |
| No branch per phase | Feature branch per phase | Phase 3 (this phase) | Clean PR workflow, isolated changes |
| Manual PR creation | Automated PR via gh CLI | Phase 3 (this phase) | Consistent PR format, faster workflow |
| Playwright headless mode | Xvfb + Playwright headed mode | Already in OpenClaw | Reduces bot detection, better compatibility |

**Deprecated/outdated:**
- **Headless mode without Xvfb:** Modern approach uses Xvfb even on VPS for "headed" mode behavior (reduces throttling, better WebGL/canvas support).
- **Git hooks for commit format:** Skills are system prompts—behavioral instructions are more flexible than hooks.
- **Separate test phase:** Test-first discipline integrates testing into every task, not separate phase.

## Open Questions

1. **Branch merge strategy: squash vs rebase?**
   - What we know: GitHub CLI supports both (`gh pr merge --squash` or `gh pr merge --rebase`)
   - What's unclear: Which strategy for phase PRs? Squash preserves task commits in PR, loses them in main. Rebase keeps all task commits.
   - Recommendation: **Squash merge** per phase. Rationale: main branch stays clean (one commit per phase), full task history preserved in PR and feature branch. If task-level history needed, check out feature branch.

2. **Test command detection: how to find project's test command?**
   - What we know: package.json has `"scripts": {"test": "..."}`, Rust uses `cargo test`, Go uses `go test`
   - What's unclear: How to auto-detect for every project type?
   - Recommendation: **Plan-phase templates include test command per project type**. plan-phase skill can detect project type (package.json = Node, Cargo.toml = Rust, go.mod = Go, etc.) and include appropriate test command in task `<verify>` sections.

3. **Build command detection: similar to test command?**
   - What we know: package.json has `"scripts": {"build": "..."}`, Rust uses `cargo build`, etc.
   - What's unclear: Same detection challenge as test commands.
   - Recommendation: Same as test commands—plan-phase templates per project type, verify-phase detects and runs project-specific build command.

4. **Xvfb configuration: is DISPLAY=:1 always correct?**
   - What we know: OpenClaw's sandbox-browser-entrypoint.sh sets DISPLAY=:1 and starts Xvfb :1
   - What's unclear: Different DISPLAY numbers for multiple sandboxes?
   - Recommendation: **Use DISPLAY=:1 as default**, OpenClaw's configuration is battle-tested. If multiple browser sandboxes needed, OpenClaw already handles this (separate containers with different DISPLAY numbers).

## Sources

### Primary (HIGH confidence)

- **OpenClaw Codebase Analysis:**
  - `/Users/didac/Juancho/src/infra/git-commit.ts` - Git commit hash resolution (confirms git integration)
  - `/Users/didac/Juancho/Dockerfile.sandbox-browser` - Xvfb configuration (lines 20-21)
  - `/Users/didac/Juancho/scripts/sandbox-browser-entrypoint.sh` - Xvfb startup (line 17)
  - `/Users/didac/Juancho/package.json` - Vitest 4.0.18, Playwright 1.58.2 (lines 180, 156)
  - `/Users/didac/Juancho/vitest.config.ts` - Test configuration with 70% coverage thresholds
  - `/Users/didac/.openclaw/workspace-juancho/skills/execute-phase/SKILL.md` - Existing per-task commit format (line 99)
  - `/Users/didac/.openclaw/workspace-juancho/skills/verify-phase/SKILL.md` - Existing 3-level verification (lines 101-115)
  - `/Users/didac/Juancho/.agents/skills/review-pr/SKILL.md` - GitHub CLI usage patterns (gh pr view, gh pr diff)
  - `/Users/didac/Juancho/.agents/skills/prepare-pr/SKILL.md` - GitHub CLI PR creation patterns

- **OpenClaw Documentation:**
  - `/Users/didac/Juancho/docs/install/docker.md` - Xvfb headless browser setup documentation

- **Conventional Commits Specification:**
  - [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) - Official specification
  - [Conventional Commits Cheatsheet](https://gist.github.com/qoomon/5dfcdf8eec66a051ecd85625518cfd13) - Quick reference

### Secondary (MEDIUM confidence)

- **Best Practices Articles:**
  - [Mastering commit messages: A guide to Conventional Commits and best practices](https://www.enlume.com/blogs/mastering-commit-messages-a-guide-to-conventional-commits-and-best-practices/) - Commit message patterns
  - [Conventional Commit Specification by Pranay Bathini](https://pranaybathini.medium.com/conventional-commit-specification-ecd701b0bbb2) - Specification overview

### Tertiary (LOW confidence)

None - all findings verified with OpenClaw codebase and official documentation.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All tools already in OpenClaw, versions confirmed in package.json and Dockerfiles
- Architecture: HIGH - Patterns verified in existing OpenClaw skills (.agents/skills/) and workspace skills
- Pitfalls: HIGH - Based on actual OpenClaw implementation (test enforcement, Xvfb configuration)

**Research date:** 2026-02-15
**Valid until:** 60 days (stable domain - git, testing, browser automation don't change rapidly)

**Key Finding:** Phase 3 is NOT about new tools or skills. It's about enhancing existing execute-phase and verify-phase skills with 200-400 lines of behavioral instructions each. OpenClaw already has git (via Bash), tests (Vitest), browser (Playwright), Xvfb (sandbox-browser), and GitHub CLI (verified in .agents/skills/). The work is INSTRUCTION AUTHORING, not coding.
