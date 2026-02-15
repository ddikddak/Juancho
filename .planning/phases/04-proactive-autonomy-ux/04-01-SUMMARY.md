---
phase: 04
plan: 01
subsystem: proactive-execution
tags: [heartbeat, automation, cron, telegram, autonomous-execution]
requires: [03-03]
provides:
  - Heartbeat configuration for proactive phase execution
  - Autonomous work detection and execution without human prompting
  - Idle state handling with HEARTBEAT_OK token
  - Infinite loop prevention via execution state checking
affects: [04-02, 04-03, 04-04]
tech-stack:
  added: []
  patterns:
    - Heartbeat-driven proactive execution
    - Event-driven autonomous agent behavior
    - State-based decision trees for work determination
decisions:
  - id: heartbeat-interval
    choice: 5-minute check interval
    rationale: Balances responsiveness with API costs
  - id: active-hours-timezone
    choice: Use "user" timezone (not "local")
    rationale: Avoids VPS timezone mismatch, respects user working hours
  - id: idle-token
    choice: HEARTBEAT_OK for idle state
    rationale: Prevents unnecessary Telegram notifications when no work available
key-files:
  created:
    - ~/.openclaw/workspace-juancho/heartbeat.md
  modified:
    - ~/.openclaw/openclaw.json
metrics:
  duration: 1min
  completed: 2026-02-15
---

# Phase 4 Plan 01: Heartbeat Configuration Summary

**One-liner:** Configured OpenClaw heartbeat for autonomous phase execution every 5 minutes during 08:00-18:00 Europe/Madrid, with HEARTBEAT_OK idle handling and infinite loop prevention.

## What Was Built

Configured the heartbeat system that enables Juancho's core proactive autonomy. The heartbeat checks project state every 5 minutes, reads `.planning/STATE.md`, determines the next action (execute phase, verify phase, transition to next phase), and either acts autonomously or reports blockers via Telegram.

**Key capability:** Juancho now works WITHOUT being prompted. After planning a phase, the heartbeat detects unexecuted plans and executes them automatically during active hours.

## Tasks Completed

### Task 1: Configure heartbeat in openclaw.json

**Commit:** 34b7273

**What was done:**
- Added `agents.defaults.heartbeat` configuration to openclaw.json
- Set check interval to "5m" for balance between responsiveness and API costs
- Configured active hours 08:00-18:00 with "user" timezone (Europe/Madrid)
- Set ackMaxChars to 200 for brief Telegram status updates
- Referenced "heartbeat.md" workspace file for behavior protocol
- Preserved all existing agents.list and bindings configuration

**Files modified:**
- `~/.openclaw/openclaw.json` (+15 lines)

**Verification:**
- Valid JSON (passed node JSON.parse)
- heartbeat.enabled = true
- heartbeat.every = "5m"
- heartbeat.prompt = "heartbeat.md"
- heartbeat.activeHours.timezone = "user"
- agents.list and bindings intact

### Task 2: Create heartbeat behavior protocol

**Commit:** 159b1b7

**What was done:**
- Created `~/.openclaw/workspace-juancho/heartbeat.md` (214 lines, 6182 chars)
- Implemented complete decision tree for autonomous execution:
  1. Gate checks: Verify project initialized before acting
  2. Read STATE.md: Get current phase/plan/status
  3. Execution state check: Prevent re-triggering running work (infinite loop prevention)
  4. Determine next action:
     - Unexecuted plans → Execute via /gsd:execute-phase
     - Executed but not verified → Verify via /gsd:verify-phase
     - Phase complete → Transition to next phase automatically
     - Blocked → Report blocker with suggestions via Telegram
  5. HEARTBEAT_OK for idle state (suppresses delivery, no noise)

**Files created:**
- `~/.openclaw/workspace-juancho/heartbeat.md` (214 lines, 6182 chars)

**Verification:**
- File exists at correct path
- Contains HEARTBEAT_OK token usage (5 instances)
- Contains STATE.md reading logic
- Contains execution state check (prevents infinite loops)
- Contains decision tree for execute/verify/plan transitions
- Under 350 lines: 214 (PASS)
- Under 15000 chars: 6182 (PASS)

## Decisions Made

### Decision 1: Heartbeat Interval - 5 Minutes

**Options considered:**
- 1 minute: More responsive, higher API costs, more frequent interruptions
- 5 minutes: Balance between responsiveness and cost
- 15 minutes: Lower costs, slower reaction to completed work

**Choice:** 5 minutes

**Rationale:** Developer workflow needs reasonable responsiveness (don't want to wait 15 minutes for next phase to start), but 1-minute checks are excessive for task-based work that typically takes 5-15 minutes per task. 5 minutes is the sweet spot for proactive execution without constant polling.

### Decision 2: Active Hours Timezone - "user" (not "local")

**Options considered:**
- "local": Use VPS system timezone
- "user": Use user's configured timezone from agents.defaults.userTimezone

**Choice:** "user" with Europe/Madrid

**Rationale:** Critical for VPS deployment. If Juancho runs on a VPS in a different timezone, using "local" would fire heartbeat during user's off-hours. Research highlighted this as Pitfall 6. Using "user" timezone ensures heartbeat respects user's actual working hours (08:00-18:00 Madrid time), not VPS server time.

### Decision 3: HEARTBEAT_OK Token for Idle State

**Options considered:**
- Always send status updates: Every heartbeat sends "Idle, no work" message
- HEARTBEAT_OK token: Suppress delivery when idle, only notify when action taken or blocked

**Choice:** HEARTBEAT_OK token

**Rationale:** Without idle suppression, user receives Telegram messages every 5 minutes saying "nothing to do" during idle periods. HEARTBEAT_OK token (from OpenClaw heartbeat-runner.ts) suppresses delivery, keeping Telegram clean. Only real updates (execution started, blocker encountered) get delivered.

### Decision 4: Execution State Check for Infinite Loop Prevention

**Options considered:**
- Trust heartbeat won't re-trigger: Assume STATE.md updates fast enough
- Check execution state: Explicitly verify work isn't already running before acting

**Choice:** Check execution state before every action

**Rationale:** Research identified Pitfall 1 (heartbeat infinite loops) as critical. If heartbeat fires while execute-phase is still running, it could trigger another execute-phase, creating infinite loop. Checking STATE.md for "executing" or "In progress" status before acting prevents this entirely.

## Deviations from Plan

None - plan executed exactly as written.

## Technical Implementation

### Heartbeat Configuration Structure

```json5
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "enabled": true,
        "every": "5m",              // Check every 5 minutes
        "prompt": "heartbeat.md",   // Reference workspace file
        "target": "telegram",       // Deliver to Telegram
        "ackMaxChars": 200,         // Brief status updates
        "activeHours": {
          "start": "08:00",         // User working hours
          "end": "18:00",
          "timezone": "user"        // Use userTimezone, not VPS time
        }
      },
      "userTimezone": "Europe/Madrid"  // User's actual timezone
    }
  }
}
```

### Heartbeat Decision Tree

```
Gate Checks
  ↓
  .planning/ exists? → No → HEARTBEAT_OK
  ↓ Yes
  ROADMAP.md exists? → No → HEARTBEAT_OK
  ↓ Yes
Read STATE.md
  ↓
  Status = "executing"? → Yes → HEARTBEAT_OK (prevent infinite loop)
  ↓ No
Determine Next Action
  ↓
  Plans exist but not executed? → Yes → /gsd:execute-phase
  ↓ No
  Phase executed but not verified? → Yes → /gsd:verify-phase
  ↓ No
  Phase complete? → Yes → Check next phase
    ↓
    Next phase has plans? → Yes → /gsd:execute-phase (next phase)
    ↓ No
    Next phase exists but no plans? → Yes → Report "Needs planning" + HEARTBEAT_OK
    ↓ No
    No more phases? → Yes → Report "All complete" + HEARTBEAT_OK
  ↓
  Blockers in STATE.md? → Yes → Report blocker + suggestions
  ↓ No
  HEARTBEAT_OK (idle)
```

### Key Code Patterns

**Infinite Loop Prevention:**
```bash
if [[ "$STATUS" == *"executing"* ]] || [[ "$STATUS" == *"In progress"* ]]; then
  echo "HEARTBEAT_OK"
  exit 0
fi
```

**HEARTBEAT_OK for Idle:**
```bash
# No active project
if [ ! -d ".planning/" ]; then
  echo "HEARTBEAT_OK"
  exit 0
fi
```

**Autonomous Execution:**
```bash
if [ $PLAN_COUNT -gt $SUMMARY_COUNT ]; then
  echo "Starting execution of plan $NEXT_PLAN..."
  /gsd:execute-phase $CURRENT_PHASE
  exit 0
fi
```

## Verification Results

All verification criteria passed:

1. **openclaw.json validity:**
   - Valid JSON (node parse successful)
   - heartbeat.enabled = true ✓
   - heartbeat.every = "5m" ✓
   - heartbeat.prompt = "heartbeat.md" ✓
   - heartbeat.activeHours.timezone = "user" ✓
   - agents.list and bindings preserved ✓

2. **heartbeat.md completeness:**
   - File exists at ~/.openclaw/workspace-juancho/heartbeat.md ✓
   - Contains HEARTBEAT_OK token usage ✓
   - Contains STATE.md reading logic ✓
   - Contains execution state check (prevent re-trigger) ✓
   - Contains decision tree for execute/verify/plan transitions ✓
   - Under 350 lines (214 lines) ✓
   - Under 15000 chars (6182 chars) ✓

## Testing Notes

**Not yet tested in live OpenClaw instance** - workspace override effectiveness pending verification when OpenClaw is running. If heartbeat doesn't work, may need to verify OpenClaw heartbeat-runner.ts is enabled and configured correctly.

**Manual test plan for next verification:**
1. Start OpenClaw with Juancho workspace
2. Initialize a test project with .planning/ and ROADMAP.md
3. Create a plan in Phase 1
4. Wait 5 minutes (or trigger heartbeat manually)
5. Verify heartbeat detects unexecuted plan
6. Verify /gsd:execute-phase is invoked automatically
7. During execution, verify heartbeat returns HEARTBEAT_OK (doesn't re-trigger)
8. After execution, verify heartbeat transitions to verification
9. Test idle state: Verify HEARTBEAT_OK suppresses Telegram delivery

## Next Phase Readiness

**Ready for Phase 4 Plan 02** (Telegram Command Wrappers):

The heartbeat configuration is complete. Next steps:
- Create Telegram command wrapper skills (/status, /commit, /test, /phase, /research)
- These skills will be thin wrappers (5-30 lines) delegating to existing GSD skills
- Users can trigger actions manually via Telegram while heartbeat handles automation
- Heartbeat will continue working in background regardless of manual commands

**No blockers for next plan.**

## Human Decisions Required

None - plan was fully autonomous.

## Work Log

**Started:** 2026-02-15 10:37 UTC
**Completed:** 2026-02-15 10:38 UTC
**Duration:** 1 minute

### Timeline

- 10:37: Read plan and context files
- 10:37: Task 1 - Updated openclaw.json with heartbeat config
- 10:37: Verified JSON validity and configuration values
- 10:37: Committed Task 1 (34b7273)
- 10:37: Task 2 - Created heartbeat.md with decision tree
- 10:38: Verified heartbeat.md requirements (line count, char count, content)
- 10:38: Committed Task 2 (159b1b7)
- 10:38: Verified all success criteria
- 10:38: Created SUMMARY.md

### Issues Encountered

None - plan execution was straightforward.

### Tradeoffs Evaluated

| Aspect | Option A | Option B | Chosen | Why |
|--------|----------|----------|--------|-----|
| Check interval | 1min (responsive) | 5min (balanced) | 5min | Balance between responsiveness and API costs |
| Timezone | "local" (VPS) | "user" (config) | "user" | Respects user working hours, not VPS location |
| Idle handling | Always notify | HEARTBEAT_OK | HEARTBEAT_OK | Prevents Telegram noise during idle periods |
| Loop prevention | Trust updates | Explicit check | Explicit check | Guarantees no infinite loops from race conditions |

## Links to Resources

**Configuration files:**
- `~/.openclaw/openclaw.json` - Heartbeat configuration
- `~/.openclaw/workspace-juancho/heartbeat.md` - Behavior protocol

**Related phase docs:**
- `.planning/phases/04-proactive-autonomy-ux/04-RESEARCH.md` - Research on heartbeat patterns and pitfalls
- `.planning/ROADMAP.md` - Phase 4 overview

**OpenClaw references:**
- OpenClaw heartbeat-runner.ts - Implementation of heartbeat system
- OpenClaw heartbeat-active-hours.ts - Timezone handling
- OpenClaw config schema (agents.defaults.heartbeat)

## Metrics

**File changes:**
- Files created: 1 (heartbeat.md)
- Files modified: 1 (openclaw.json)
- Lines added: 229 (15 in openclaw.json, 214 in heartbeat.md)
- Commits: 2

**Execution:**
- Duration: 1 minute
- Tasks completed: 2/2
- Deviations: 0
- Human decisions: 0
- Blockers encountered: 0

**Quality:**
- All verification criteria passed
- File size limits met (214 lines, 6182 chars < 15k limit)
- Valid JSON configuration
- Complete decision tree implementation
