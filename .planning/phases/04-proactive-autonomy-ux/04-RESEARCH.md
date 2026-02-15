# Phase 4: Proactive Autonomy & UX - Research

**Researched:** 2026-02-15
**Domain:** Cron/heartbeat automation, Telegram UX, onboarding workflows, documentation generation
**Confidence:** HIGH

## Summary

Phase 4 adds proactive autonomous execution, polished Telegram UX, and developer-focused onboarding to Juancho. The critical insight: **OpenClaw already has 95% of the required infrastructure**. The heartbeat system (`src/infra/heartbeat-runner.ts`) provides configurable proactive behavior with active hours, schedule control, and delivery targets. The cron system (`src/cron/`) provides job scheduling with systemEvent and agentTurn payloads. The Telegram infrastructure handles message chunking, file uploads, and inline keyboards out-of-the-box. The wizard system (`src/wizard/onboarding.ts`) provides an interactive CLI onboarding flow.

What's needed: (1) Thin wrapper skills for Telegram commands that call existing GSD functionality, (2) Heartbeat configuration in `openclaw.json` to enable proactive execution, (3) Juancho-specific onboarding flow that wraps the GSD pipeline, (4) Work log and documentation templates integrated into execution workflow. This is NOT new infrastructure—it's CONFIGURING and WRAPPING existing OpenClaw capabilities.

**Primary recommendation:** Configure heartbeat for proactive phase checking, create lightweight command wrapper skills for Telegram, design Juancho onboarding flow as interactive wrapper around `/new-project`, integrate documentation generation into verify-phase skill, and add work logging to execute-phase skill. All implementation via workspace configuration and skill enhancement within existing 20k char limits.

## Standard Stack

### Core (Already In OpenClaw)

| Component | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| heartbeat-runner.ts | OpenClaw 2026.2.13 | Proactive agent polling system | Runs on interval, checks system events, delivers to configured channel |
| cron service | Built-in | Scheduled job execution | Supports systemEvent and agentTurn payloads, every/at/cron schedules |
| croner | Latest | Cron expression parser | Industry standard cron library for Node.js, used by OpenClaw |
| grammy | Latest (via OpenClaw) | Telegram Bot API framework | Modern, type-safe Telegram bot library |
| Telegram chunking | Built-in | Message splitting (4096 char limit) | OpenClaw handles automatically via draft-chunking.ts |
| Telegram file upload | Built-in | sendDocument for large content | Bot API supports 2GB files, 512KB chunks |
| Inline keyboards | Built-in | Button-based interactions | buildInlineKeyboard helper in send.ts |
| Wizard system | Built-in | CLI onboarding flows | Interactive prompts via clack, session management |

### Supporting

| Component | Purpose | When to Use |
|---------|---------|-------------|
| Active hours config | Time-based heartbeat gating | Configure user working hours for proactive notifications |
| Heartbeat visibility | Message delivery control | "ack" sends confirmations, "silent" suppresses unless actionable |
| System events queue | Event-driven heartbeat triggers | Queue events for heartbeat to process (exec completion, phase transition) |
| Native command menu | Telegram /command registration | Auto-synced to BotFather via bot-native-command-menu.ts |
| TypeDoc | Documentation generation | Generate API docs from TypeScript (already in OpenClaw ecosystem) |

### Not Needed

| Alternative | Tradeoff |
|------------|----------|
| External cron daemon | OpenClaw has built-in cron service with better integration |
| Custom Telegram library | grammy provides all needed features (chunking, files, keyboards) |
| Separate onboarding app | Wizard system provides interactive CLI flows |
| Documentation build system | TypeDoc + simple templates sufficient for Juancho use cases |

**Installation:**
All components built into OpenClaw. No additional packages required.

## Architecture Patterns

### Recommended Configuration Structure

```
~/.openclaw/
├── openclaw.json                # Heartbeat config for proactive behavior
│   agents:
│     defaults:
│       heartbeat:
│         enabled: true
│         every: "5m"            # Check every 5 minutes
│         prompt: "heartbeat.md" # Custom heartbeat instructions
│         target: "telegram"     # Deliver to Telegram
│         ackMaxChars: 200       # Brief status updates
│         activeHours:
│           start: "08:00"       # User working hours
│           end: "18:00"
│           timezone: "user"
├── workspace-juancho/
│   ├── heartbeat.md             # Heartbeat behavior (NEW)
│   ├── skills/
│   │   ├── execute-phase/       # ENHANCE with work logging
│   │   ├── verify-phase/        # ENHANCE with docs generation
│   │   ├── onboard/             # NEW: Juancho onboarding wrapper
│   │   ├── telegram-status/     # NEW: Telegram /status command
│   │   ├── telegram-commit/     # NEW: Telegram /commit command
│   │   ├── telegram-test/       # NEW: Telegram /test command
│   │   ├── telegram-phase/      # NEW: Telegram /phase command
│   │   └── telegram-research/   # NEW: Telegram /research command
│   └── templates/
│       ├── WORK_LOG.md          # Work log template (NEW)
│       └── PROJECT_DOCS.md      # Documentation template (NEW)
```

### Pattern 1: Heartbeat-Driven Proactive Execution

**What:** OpenClaw heartbeat runs on schedule, checks `.planning/STATE.md`, determines next action autonomously.

**When to use:** After phase planning complete, enable proactive mode for autonomous execution.

**How it works:**

1. **Heartbeat configuration** (in `openclaw.json`):
```json5
{
  agents: {
    defaults: {
      heartbeat: {
        enabled: true,
        every: "5m",              // Check every 5 minutes
        prompt: "heartbeat.md",   // Reference workspace file
        target: "telegram",       // Deliver to Telegram
        ackMaxChars: 200,         // Keep updates brief
        activeHours: {
          start: "08:00",         // User working hours
          end: "18:00",
          timezone: "user"        // Use user's configured timezone
        }
      }
    }
  }
}
```

2. **Heartbeat prompt** (in `~/.openclaw/workspace-juancho/heartbeat.md`):
```markdown
# Juancho Heartbeat Protocol

Check project state and determine next action:

1. Read .planning/STATE.md
2. Check current phase status:
   - If plans exist but not executed → Run /execute-phase
   - If phase executed but not verified → Run /verify-phase
   - If phase complete → Check next phase, suggest /plan-phase
   - If blocked → Report blocker via Telegram, await human input

3. Response format:
   - If action taken: "▶️ {action} started"
   - If waiting: "✓ Idle - {reason}"
   - If blocked: "🚫 Blocked: {issue}"
   - If tests fail: "⚠️ Test failure - debugging"

4. Deliver updates via configured target (Telegram)

**HEARTBEAT_OK when idle.** Only act when work is ready.
```

3. **System event triggers** (from execute-phase, verify-phase skills):
```typescript
// When phase transitions or blockers occur, emit system event
// Heartbeat runner picks up events via peekSystemEvents()
// Example: Phase completion triggers notification
```

**Implementation Location:** `openclaw.json` heartbeat config + `heartbeat.md` workspace file.

**Key Insight:** Heartbeat runner already handles scheduling, active hours, delivery. We just configure it and provide instructions.

### Pattern 2: Telegram Command Wrappers

**What:** Thin skills that expose GSD functionality via Telegram native commands.

**When to use:** User needs to check status, trigger actions, or query state from Telegram.

**Command mapping:**

| Telegram Command | GSD Skill | Purpose |
|-----------------|-----------|---------|
| /status | telegram-status → /status | Show current project state |
| /commit | telegram-commit → execute git commit | Quick commit without full phase execution |
| /test | telegram-test → run test suite | Run tests independently |
| /phase N | telegram-phase → /execute-phase N | Execute specific phase |
| /research TOPIC | telegram-research → /research-phase N | Research specific domain |

**Implementation pattern:**

```markdown
---
name: telegram-status
description: Telegram wrapper for /status. Registered as native command, calls existing status skill.
nativeCommand: status
nativeDescription: "Show current project status"
---

# Telegram Status Command

Thin wrapper - delegates to existing /status skill.

## Workflow

1. Call existing status skill:
   ```
   /status
   ```

2. Format output for Telegram:
   - If > 4096 chars: chunk via OpenClaw's automatic chunking
   - If very long: offer file upload instead
   - Use monospace formatting for structure

3. Add interactive buttons:
   - "Execute Current Phase" → /phase {current}
   - "Verify Phase" → /verify-phase {current}
   - "View Plans" → show plan list

Implementation: ~30 lines, delegates to existing functionality.
```

**Registration:** Skills with `nativeCommand` frontmatter auto-register with Telegram bot via `bot-native-commands.ts`.

**Key Insight:** These are NOT duplicating functionality—they're EXPOSING existing skills via Telegram interface.

### Pattern 3: Message Chunking and File Uploads

**What:** OpenClaw automatically handles Telegram's 4096 character limit, with manual file upload for very large content.

**When to use:** Status updates, diffs, logs, documentation from Telegram commands.

**Automatic chunking** (already implemented in `draft-chunking.ts`):

```typescript
// OpenClaw configuration (already in place)
channels: {
  telegram: {
    draftChunk: {
      minChars: 200,
      maxChars: 800,
      breakPreference: "paragraph"  // Smart splitting
    }
  }
}
```

**When chunking isn't enough** (manual file upload pattern):

```markdown
## Handling Large Content in Telegram Commands

1. Estimate response length
2. If < 4096 chars: send normally (auto-chunks)
3. If 4096-20000 chars: send as multiple messages (auto-chunks)
4. If > 20000 chars: write to file, send via sendDocument

Example:
```bash
# In telegram-status skill
OUTPUT_LENGTH=$(wc -c < status.txt)
if [ $OUTPUT_LENGTH -gt 20000 ]; then
  # Send as file
  echo "Status report too large, sending as file..."
  # OpenClaw Telegram tool handles file upload
else
  # Send as text (auto-chunks)
  cat status.txt
fi
```

**Implementation Location:** Telegram command wrapper skills (conditional logic for file vs text).

**Source:** [Telegram Limits](https://limits.tginfo.me/en) - Messages: 4096 chars, Files: 2GB max

### Pattern 4: Inline Keyboards for Interactions

**What:** Clickable buttons in Telegram messages for approvals, choices, and actions.

**When to use:** Phase transitions, verification checkpoints, approval workflows.

**Pattern:**

```markdown
## Inline Keyboard for Phase Approval

When verify-phase completes:

1. Send verification summary
2. Add inline keyboard:
   ```
   [[Approve Phase] [Request Changes]]
   [[View Details] [Run Tests Again]]
   ```

3. Handle callback:
   - Approve → Mark phase complete, trigger heartbeat
   - Request Changes → Prompt for feedback, create gap closure plan
   - View Details → Send detailed verification report
   - Run Tests Again → Re-run verify-phase

Implementation via OpenClaw's buildInlineKeyboard:
```typescript
// Example from send.ts
const buttons = [
  [
    { text: "Approve Phase", callback_data: "approve_phase_04" },
    { text: "Request Changes", callback_data: "request_changes" }
  ],
  [
    { text: "View Details", callback_data: "view_details" },
    { text: "Run Tests Again", callback_data: "rerun_tests" }
  ]
];
```

**Implementation Location:** verify-phase skill enhancement (add keyboard to completion message).

**Key Insight:** OpenClaw already has keyboard infrastructure—just add button definitions to skill outputs.

### Pattern 5: Juancho-Specific Onboarding

**What:** Interactive CLI flow tailored to developer setup (repo, git, AI provider, first project).

**When to use:** New user installing Juancho for first time.

**Flow design:**

```
openclaw onboard --workspace juancho

1. Welcome to Juancho 🛠️
   "I'm your AI developer. I follow engineering discipline and work autonomously."

2. Git Configuration Check
   - Verify git user.name and user.email
   - If missing, prompt to configure
   - Test git authentication (GitHub/GitLab)

3. Repository Setup
   - "Where's your project?" [path input]
   - Initialize .planning/ directory
   - Create initial PROJECT.md (from template)

4. AI Provider Setup
   - "Which AI provider?" [Claude/OpenAI/Gemini/Ollama]
   - API key or OAuth flow
   - Test connection

5. Telegram Integration
   - "Link Telegram for updates?" [yes/no]
   - If yes: Bot token setup, pair account
   - Test message delivery

6. First Project Definition
   - "Describe your project" [multiline input]
   - Run /new-project with description
   - Generate initial roadmap

7. Completion
   - "Setup complete! Try /status or /plan-phase 1"
   - Start heartbeat if configured
```

**Implementation Location:** New skill `~/.openclaw/workspace-juancho/skills/onboard/SKILL.md` (~150 lines).

**Key Insight:** Wraps existing `/new-project` skill, adds developer-specific setup checks.

### Pattern 6: Documentation Generation

**What:** Auto-generate README, API docs, inline comments during phase verification.

**When to use:** verify-phase skill, before marking phase complete.

**Documentation types:**

| Type | Tool | When Generated | Location |
|------|------|----------------|----------|
| README.md | Template | After phase 1 (foundation) | Project root |
| API docs | TypeDoc | After implementing public APIs | docs/api/ |
| Inline comments | Manual | During task execution | Source files |
| Work logs | Template | After each plan | .planning/phases/{XX}-{name}/ |

**Work log pattern:**

```markdown
# .planning/phases/04-proactive-autonomy-ux/WORK_LOG.md

## Plan 04-01: Heartbeat Configuration

**Started:** 2026-02-15 10:30
**Completed:** 2026-02-15 10:45

### Decisions Made

- Heartbeat interval: 5 minutes (balances responsiveness with API costs)
- Active hours: 08:00-18:00 user timezone (avoid off-hours notifications)
- Ack mode: brief (200 char max) for non-actionable updates

### Tradeoffs Considered

- Shorter interval (1min) → more responsive but higher costs
- Longer interval (15min) → lower costs but slower reaction
- Chose 5min as sweet spot for developer workflow

### Issues Encountered

- Initial config had invalid cron expression
- Fixed by using "every" schedule instead of cron syntax

### Code Changes

- Modified openclaw.json heartbeat config
- Created heartbeat.md with status check protocol
```

**Implementation Location:** execute-phase skill enhancement (generate work log after plan completion).

**TypeDoc integration pattern:**

```bash
# In verify-phase skill, for TypeScript projects
if [ -f "tsconfig.json" ]; then
  echo "Generating API documentation..."
  npx typedoc --out docs/api src/
  git add docs/api/
  git commit -m "docs: generate API documentation"
fi
```

**Source:** [TypeDoc](https://typedoc.org/) - Documentation generator for TypeScript

### Pattern 7: Configurable Proactivity Schedule

**What:** User controls when and how aggressively Juancho works autonomously.

**When to use:** Project setup, user preference changes.

**Configuration dimensions:**

| Setting | Values | Effect |
|---------|--------|--------|
| heartbeat.every | "1m" to "1h" | How often to check for work |
| heartbeat.activeHours | time range | When proactive behavior is allowed |
| heartbeat.ackMaxChars | 50-500 | How verbose status updates are |
| autonomy level | low/moderate/high | How much to do without asking |

**Example configurations:**

```json5
// Low autonomy - ask frequently
{
  heartbeat: {
    every: "15m",           // Check less often
    ackMaxChars: 500,       // More detailed updates
  },
  autonomy: "low"           // More human_needed checkpoints
}

// High autonomy - minimize interruptions
{
  heartbeat: {
    every: "5m",            // Check frequently
    ackMaxChars: 100,       // Brief updates only
    activeHours: {
      start: "00:00",       // Work 24/7
      end: "24:00"
    }
  },
  autonomy: "high"          // Fewer human_needed checkpoints
}
```

**Implementation Location:** `.planning/config.json` (user-editable), referenced by heartbeat and skills.

**Key Insight:** OpenClaw's activeHours + heartbeat interval provide scheduling control. Autonomy level (from Phase 2) controls checkpoint frequency.

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Telegram message chunking | Manual string splitting | OpenClaw draft-chunking.ts | Handles emoji/unicode, smart breaks, auto-sends |
| Cron scheduling | Custom timer loops | OpenClaw cron service + croner | Handles edge cases, DST, error recovery |
| Telegram command registration | Manual /setMyCommands API | OpenClaw bot-native-command-menu.ts | Auto-syncs to BotFather, handles conflicts |
| File upload logic | Manual multipart handling | grammy InputFile + sendDocument | Handles chunking, retries, errors |
| Inline keyboard state | Manual callback_data parsing | OpenClaw callback query handlers | Deduplication, context preservation |
| Onboarding flows | Custom CLI prompts | OpenClaw wizard system (clack) | Interactive, resumable, validated |
| Documentation generation | Manual template filling | TypeDoc for APIs, templates for others | Extracts from source, stays in sync |
| Active hours checking | Manual timezone math | OpenClaw heartbeat-active-hours.ts | Handles DST, timezone edge cases |

**Key insight:** OpenClaw has mature implementations for ALL these patterns. Reuse, don't rebuild.

## Common Pitfalls

### Pitfall 1: Heartbeat Infinite Loops

**What goes wrong:** Heartbeat triggers action that emits event that triggers heartbeat again, creating infinite loop.

**Why it happens:** System events queue persists, heartbeat processes same event repeatedly.

**How to avoid:**
- Use HEARTBEAT_OK token for idle state (prevents delivery)
- Clear processed system events after handling
- Gate actions with state checks (don't execute if already running)

**Warning signs:** Rapid Telegram messages, same action repeating, high API usage.

**Implementation check:**
```markdown
# In heartbeat.md
Before taking action:
1. Check if action already running (read STATE.md)
2. If running → HEARTBEAT_OK
3. If idle and work ready → take action and CLEAR event
```

### Pitfall 2: Telegram Command Duplication

**What goes wrong:** Creating full duplicate implementations of GSD skills as Telegram commands.

**Why it happens:** Misunderstanding that Telegram commands need separate logic.

**How to avoid:**
- Telegram skills are THIN WRAPPERS (5-30 lines)
- Delegate to existing skills immediately
- Only add Telegram-specific formatting/keyboards

**Warning signs:** Telegram skills > 100 lines, duplicated business logic, maintenance drift.

**Implementation pattern:**
```markdown
# telegram-status skill (CORRECT - thin wrapper)
1. Call /status skill
2. Format for Telegram (if needed)
3. Add buttons (if interactive)
Total: ~20 lines

# telegram-status skill (WRONG - duplicate logic)
1. Read STATE.md
2. Calculate metrics
3. Generate report
4. Format output
Total: ~200 lines (duplicates status skill)
```

### Pitfall 3: Message Chunking Edge Cases

**What goes wrong:** Manual chunking breaks on emoji, unicode, or code blocks.

**Why it happens:** Naive character counting doesn't account for multi-byte characters or Markdown structure.

**How to avoid:**
- Let OpenClaw auto-chunk (draft-chunking.ts handles this)
- For manual chunking: use Buffer.byteLength, not string.length
- For very large content: use file upload instead

**Warning signs:** Telegram errors "can't parse entities", broken emoji, mid-word splits.

**Implementation check:**
```bash
# WRONG
if [ ${#OUTPUT} -gt 4096 ]; then
  echo "${OUTPUT:0:4096}"  # Naive substring
fi

# CORRECT
# Let OpenClaw handle it automatically
cat output.txt  # Auto-chunks via draft-chunking.ts

# OR for manual file upload
if [ $(wc -c < output.txt) -gt 20000 ]; then
  # Send as file
fi
```

**Source:** [Telegram Bot API](https://core.telegram.org/bots/faq) - Message length calculation complexity

### Pitfall 4: Hardcoded Paths in Onboarding

**What goes wrong:** Onboarding flow assumes specific directory structure or file locations.

**Why it happens:** Testing in single environment, not accounting for user customization.

**How to avoid:**
- Read paths from openclaw.json (workspace, agentDir)
- Resolve relative to workspace, not hardcoded absolute paths
- Validate paths exist before using

**Warning signs:** Onboarding fails on different machines, "file not found" errors.

**Implementation check:**
```bash
# WRONG
WORKSPACE="~/.openclaw/workspace-juancho"

# CORRECT
WORKSPACE=$(cat ~/.openclaw/openclaw.json | grep -o '"workspace"[[:space:]]*:[[:space:]]*"[^"]*"' | cut -d'"' -f4)
```

### Pitfall 5: Documentation Generation Blocking Execution

**What goes wrong:** TypeDoc or doc generation takes too long, blocks phase completion.

**Why it happens:** Large codebases have thousands of files to process.

**How to avoid:**
- Run doc generation asynchronously (background task)
- Set timeout for doc generation (fail gracefully if slow)
- Make doc generation optional for large projects

**Warning signs:** verify-phase hangs, timeouts, phase completion delayed.

**Implementation check:**
```bash
# In verify-phase skill
if [ -f "tsconfig.json" ]; then
  # Run with timeout, don't block on failure
  timeout 60s npx typedoc --out docs/api src/ || {
    echo "⚠️ Doc generation timed out (large project), skipping"
  }
fi
```

### Pitfall 6: Active Hours Timezone Confusion

**What goes wrong:** Heartbeat fires at wrong times, wakes user overnight.

**Why it happens:** Timezone config defaults to "local" (VPS timezone) instead of "user" (user's timezone).

**How to avoid:**
- Always set activeHours.timezone: "user" in config
- Validate timezone against Intl.DateTimeFormat.supportedValuesOf()
- Test with user in different timezone than VPS

**Warning signs:** Notifications at unexpected hours, user complaints about timing.

**Implementation check:**
```json5
// WRONG
activeHours: {
  start: "08:00",
  end: "18:00"
  // Missing timezone - defaults to VPS local time
}

// CORRECT
activeHours: {
  start: "08:00",
  end: "18:00",
  timezone: "user"  // Uses agents.defaults.userTimezone
}
```

**Source:** OpenClaw `heartbeat-active-hours.ts` implementation

## Code Examples

Verified patterns from OpenClaw source:

### Heartbeat Configuration

```json5
// ~/.openclaw/openclaw.json
{
  agents: {
    defaults: {
      heartbeat: {
        enabled: true,
        every: "5m",              // Check every 5 minutes
        prompt: "heartbeat.md",   // Custom heartbeat instructions
        target: "telegram",       // Deliver to Telegram
        ackMaxChars: 200,         // Brief status updates
        activeHours: {
          start: "08:00",         // User working hours
          end: "18:00",
          timezone: "user"        // Use user's configured timezone
        }
      },
      userTimezone: "America/Los_Angeles"  // User's timezone
    }
  }
}
```

**Source:** OpenClaw config schema (`src/config/types.agent-defaults.ts`)

### Inline Keyboard Creation

```typescript
// From src/telegram/send.ts lines 209-229
export function buildInlineKeyboard(
  rows?: Array<Array<{ text: string; callback_data: string }>>,
): InlineKeyboardMarkup | undefined {
  if (!rows || rows.length === 0) {
    return undefined;
  }
  return {
    inline_keyboard: rows
      .filter((row) => Array.isArray(row) && row.length > 0)
      .map((row) =>
        row
          .filter((button) => button && typeof button.text === "string" && button.text.trim())
          .map(
            (button): InlineKeyboardButton => ({
              text: button.text,
              callback_data: button.callback_data || "noop",
            }),
          ),
      )
      .filter((row) => row.length > 0),
  };
}
```

**Usage in skill:**
```markdown
Send verification summary with buttons:
```
Message: "Phase 4 verification complete. All tests passed."
Buttons: [[Approve, Reject], [View Details]]
```

### Cron Job Creation

```typescript
// From src/cron/types.ts
// Example: Phase transition check every 10 minutes
{
  id: "phase-transition-check",
  name: "Check for phase completion",
  enabled: true,
  schedule: { kind: "every", everyMs: 600000 },  // 10 minutes
  sessionTarget: "main",
  wakeMode: "next-heartbeat",  // Trigger heartbeat instead of immediate
  payload: {
    kind: "systemEvent",
    text: "Check if current phase is complete and ready to transition"
  },
  delivery: {
    mode: "none"  // Don't send message, just trigger heartbeat
  }
}
```

**Source:** OpenClaw cron types (`src/cron/types.ts`)

### File Upload for Large Content

```bash
# In Telegram command skill
OUTPUT_FILE=".planning/status-full.txt"
/status > "$OUTPUT_FILE"

if [ $(wc -c < "$OUTPUT_FILE") -gt 20000 ]; then
  echo "📄 Status report is large, sending as file..."
  # OpenClaw Telegram tool handles sendDocument automatically
  # when file path provided instead of text
else
  cat "$OUTPUT_FILE"
  rm "$OUTPUT_FILE"
fi
```

**Pattern:** OpenClaw detects file paths in output and auto-uploads via `sendDocument`.

### Work Log Template

```markdown
# .planning/phases/{XX}-{name}/WORK_LOG_{PLAN}.md

## Plan {XX}-{YY}: {Plan Name}

**Started:** {timestamp}
**Completed:** {timestamp}
**Duration:** {duration}

### Objective
{Plan objective from PLAN.md}

### Decisions Made
- {Decision 1 with rationale}
- {Decision 2 with rationale}

### Tradeoffs Considered
| Option | Pros | Cons | Chosen |
|--------|------|------|--------|
| {A} | {pros} | {cons} | {✓/✗} |

### Issues Encountered
1. **{Issue description}**
   - Symptom: {what happened}
   - Root cause: {analysis}
   - Solution: {fix applied}

### Code Changes
- {File modified}: {what changed and why}
- {File created}: {purpose}

### Verification Results
{Pass/fail for each verification command}

### Human Decisions Required
{List any decisions that required human input}
```

**Implementation:** execute-phase skill generates this after plan completion.

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Manual cron daemon | Built-in cron service | OpenClaw architecture | Integrated with heartbeat, better error handling |
| node-cron library | croner library | OpenClaw current | Better timezone support, more reliable |
| Manual Telegram chunking | Auto-chunking via draft-chunking | OpenClaw current | Handles emoji/unicode correctly |
| JSDoc for TypeScript | TSDoc + TypeDoc | TypeScript ecosystem 2024+ | Better type extraction, standards compliance |
| Polling for updates | Event-driven heartbeat | OpenClaw architecture | More efficient, lower latency |

**Deprecated/outdated:**
- node-cron: OpenClaw uses croner for better timezone/DST handling
- Manual /setMyCommands: OpenClaw auto-syncs via bot-native-command-menu.ts
- Separate documentation systems: TypeDoc unified for TypeScript projects

**Sources:**
- [Node.js Job Scheduling 2026](https://forwardemail.net/en/blog/docs/node-js-job-scheduler-cron)
- [TypeDoc Documentation](https://typedoc.org/)

## Open Questions

1. **Multi-provider AI failover during heartbeat**
   - What we know: OpenClaw supports multiple AI providers with failover
   - What's unclear: How heartbeat handles provider failures, retry logic
   - Recommendation: Test heartbeat behavior when primary provider unavailable, verify failover works

2. **Telegram rate limits with proactive behavior**
   - What we know: Telegram has rate limits (30 msg/sec to same user)
   - What's unclear: How OpenClaw's throttling (grammy-throttler) interacts with heartbeat
   - Recommendation: Monitor rate limit errors in logs, adjust heartbeat frequency if needed

3. **Work log storage limits**
   - What we know: Work logs accumulate in .planning/ directory
   - What's unclear: Size limits, archival strategy for long-running projects
   - Recommendation: Add work log rotation/archival to verify-phase after N phases

4. **Onboarding flow customization**
   - What we know: OpenClaw wizard system supports interactive flows
   - What's unclear: How to integrate custom Juancho onboarding alongside default OpenClaw onboarding
   - Recommendation: Create separate `openclaw onboard --juancho` flow, or workspace-specific wizard

5. **Documentation generation for non-TypeScript projects**
   - What we know: TypeDoc works for TypeScript, Sphinx for Python
   - What's unclear: Best approach for multi-language projects or mixed codebases
   - Recommendation: Project-type detection in verify-phase, use appropriate tool per language

## Sources

### Primary (HIGH confidence)

- OpenClaw source code (`src/infra/heartbeat-runner.ts`, `src/cron/`, `src/telegram/`)
- OpenClaw config schema (`src/config/types.agent-defaults.ts`, `src/config/zod-schema.ts`)
- croner library (used by OpenClaw for cron expressions)
- grammy library (Telegram Bot API framework used by OpenClaw)
- TypeDoc official documentation - [https://typedoc.org/](https://typedoc.org/)

### Secondary (MEDIUM confidence)

- [Node.js Job Scheduling 2026](https://forwardemail.net/en/blog/docs/node-js-job-scheduler-cron) - Cron patterns verified against OpenClaw implementation
- [Telegram Limits](https://limits.tginfo.me/en) - Message limits (4096 chars) verified in OpenClaw draft-chunking.ts
- [Telegram Bot API FAQ](https://core.telegram.org/bots/faq) - File upload limits verified in OpenClaw send.ts
- [Agentic AI Workflows 2026](https://www.wrike.com/blog/what-are-agentic-workflows/) - Proactive behavior patterns

### Tertiary (LOW confidence)

- WebSearch results on autonomous agent patterns (general concepts, not OpenClaw-specific)
- Community discussions on Telegram bot best practices (not verified against OpenClaw implementation)

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All components verified in OpenClaw source code
- Architecture patterns: HIGH - Patterns extracted from existing OpenClaw infrastructure
- Pitfalls: HIGH - Based on OpenClaw implementation details and common integration issues
- Documentation generation: MEDIUM - TypeDoc verified, other tools based on ecosystem standards
- Multi-language onboarding: MEDIUM - OpenClaw wizard verified, Juancho customization not yet implemented

**Research date:** 2026-02-15
**Valid until:** 30 days (OpenClaw stable, patterns unlikely to change rapidly)

**Key constraints from prior phases:**
- Zero changes to OpenClaw src/ code (workspace files only)
- Skills under 350 lines and 15k chars (under 20k limit)
- All customization via ~/.openclaw/workspace-juancho/

**What this enables for planning:**
- Heartbeat configuration for proactive execution
- Telegram command wrapper skills (thin, delegate to existing)
- Juancho-specific onboarding flow
- Work log generation in execute-phase
- Documentation generation in verify-phase
- Configurable autonomy schedule
