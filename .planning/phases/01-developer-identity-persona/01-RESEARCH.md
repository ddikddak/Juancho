# Phase 1: Developer Identity & Persona - Research

**Researched:** 2026-02-15
**Domain:** OpenClaw system prompt architecture and workspace-driven agent configuration
**Confidence:** HIGH

## Summary

Phase 1 requires rewriting OpenClaw's agent identity from a general-purpose assistant to a coding-focused developer. Research reveals OpenClaw uses a **layered system prompt architecture** where final prompts are built dynamically from: (1) hardcoded sections in `src/agents/system-prompt.ts`, (2) agent configuration in `openclaw.json`, and (3) workspace files (SOUL.md, AGENTS.md, TOOLS.md, etc.) injected as "Project Context."

The critical insight: **workspace files override hardcoded prompts via explicit instruction** in `buildAgentSystemPrompt()` line 565: "If SOUL.md is present, embody its persona and tone." This means we can reconfigure Juancho's identity WITHOUT forking core prompt logic—just rewrite workspace files and configure a dedicated agent.

**Primary recommendation:** Configure Juancho as a separate agent in `agents.list[]` with custom workspace containing developer-focused SOUL.md, AGENTS.md, and TOOLS.md. This approach maintains upstream sync compatibility while enabling complete identity reconfiguration.

## Standard Stack

The established libraries/tools for this domain:

### Core (Inherited from OpenClaw)
| Component | Location | Purpose | Why Standard |
|---------|---------|---------|--------------|
| system-prompt.ts | src/agents/system-prompt.ts | Dynamic prompt builder | Assembles final system prompt from config + workspace + hardcoded sections |
| workspace.ts | src/agents/workspace.ts | Workspace file loader | Loads SOUL.md, AGENTS.md, TOOLS.md and other bootstrap files |
| agent-scope.ts | src/agents/agent-scope.ts | Agent configuration resolver | Maps agentId to workspace, agentDir, model, skills |
| bootstrap-files.ts | src/agents/bootstrap-files.ts | Context file builder | Converts workspace files to EmbeddedContextFile[] for prompt injection |

### Supporting (Template System)
| Component | Location | Purpose | When to Use |
|---------|---------|---------|-------------|
| Template Files | docs/reference/templates/ | Default workspace files | Bootstrap new workspaces with SOUL.md, AGENTS.md, etc. |
| workspace-templates.ts | src/agents/workspace-templates.ts | Template resolver | Load default templates when creating new workspace |

### Configuration Schema
| Config Key | Type | Purpose |
|------------|------|---------|
| agents.list[] | AgentConfig[] | Multi-agent configuration (id, workspace, agentDir, model, identity) |
| agents.list[].workspace | string | Path to agent workspace directory |
| agents.list[].agentDir | string | Path to agent state directory (auth, sessions) |
| agents.list[].identity | IdentityConfig | Agent name, emoji, avatar (for UI/display) |
| agents.defaults.workspace | string | Fallback workspace for default agent |

**Installation:**
No installation needed—reconfigure existing OpenClaw infrastructure.

## Architecture Patterns

### Recommended File Structure
```
~/.openclaw/
├── openclaw.json                    # Multi-agent config
├── workspace/                       # Default agent workspace
│   ├── SOUL.md                      # General assistant persona
│   ├── AGENTS.md                    # General assistant behavior
│   └── ...
├── workspace-juancho/               # Juancho agent workspace
│   ├── SOUL.md                      # Developer persona (REWRITE)
│   ├── AGENTS.md                    # Engineering discipline (REWRITE)
│   ├── TOOLS.md                     # Git/testing tooling notes
│   ├── IDENTITY.md                  # Juancho name/emoji/avatar
│   └── memory/
└── agents/
    ├── main/agent/                  # Default agent state
    └── juancho/agent/               # Juancho agent state
```

### Pattern 1: Workspace-Driven Identity Override
**What:** OpenClaw injects workspace files into system prompt as "Project Context" with explicit override instruction
**When to use:** Any agent identity customization (persona, behavior, tone, discipline)
**How it works:**
1. System prompt builder loads workspace files via `loadWorkspaceBootstrapFiles(workspaceDir)`
2. Files converted to `EmbeddedContextFile[]` with content truncated to `bootstrapMaxChars` (default 20k)
3. Injected into prompt under "# Project Context" section
4. SOUL.md gets special treatment: "If SOUL.md is present, embody its persona and tone. Avoid stiff, generic replies; follow its guidance unless higher-priority instructions override it."

**Example from system-prompt.ts (lines 557-571):**
```typescript
if (validContextFiles.length > 0) {
  const hasSoulFile = validContextFiles.some((file) => {
    const normalizedPath = file.path.trim().replace(/\\/g, "/");
    const baseName = normalizedPath.split("/").pop() ?? normalizedPath;
    return baseName.toLowerCase() === "soul.md";
  });
  lines.push("# Project Context", "", "The following project context files have been loaded:");
  if (hasSoulFile) {
    lines.push(
      "If SOUL.md is present, embody its persona and tone. Avoid stiff, generic replies; follow its guidance unless higher-priority instructions override it.",
    );
  }
  lines.push("");
  for (const file of validContextFiles) {
    lines.push(`## ${file.path}`, "", file.content, "");
  }
}
```

### Pattern 2: Multi-Agent Configuration
**What:** OpenClaw supports multiple isolated agents with separate workspaces, state, and auth
**When to use:** Running Juancho alongside default OpenClaw agent
**How to configure:**
```json5
// ~/.openclaw/openclaw.json
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        workspace: "~/.openclaw/workspace",
        // General assistant persona
      },
      {
        id: "juancho",
        workspace: "~/.openclaw/workspace-juancho",
        agentDir: "~/.openclaw/agents/juancho/agent",
        identity: {
          name: "Juancho",
          emoji: "👨‍💻",
          theme: "autonomous developer",
        },
        model: "anthropic/claude-opus-4-6",
        // Developer-focused workspace files
      },
    ],
  },
  bindings: [
    { agentId: "juancho", match: { channel: "telegram" } },
    { agentId: "main", match: { channel: "whatsapp" } },
  ],
}
```

### Pattern 3: Template-Based Workspace Bootstrap
**What:** OpenClaw auto-creates workspace files from templates on first run
**When to use:** Creating new agent workspace
**Where templates live:** `docs/reference/templates/*.md`
**Bootstrap files created:**
- SOUL.md, AGENTS.md, TOOLS.md, IDENTITY.md, USER.md, HEARTBEAT.md, BOOTSTRAP.md (one-time), MEMORY.md (optional)

**How to customize:**
1. Create workspace directory: `mkdir -p ~/.openclaw/workspace-juancho`
2. Write custom files (SOUL.md, AGENTS.md) — OpenClaw won't overwrite existing files
3. Configure in `openclaw.json`: `agents.list[].workspace = "~/.openclaw/workspace-juancho"`
4. Run `openclaw setup --workspace ~/.openclaw/workspace-juancho` to seed missing files

### Pattern 4: System Prompt Assembly
**What:** `buildAgentSystemPrompt()` assembles final prompt in this order:
1. **Hardcoded sections** (Tooling, Safety, Skills, Memory, OpenClaw CLI, Workspace)
2. **Runtime info** (agent ID, host, OS, model, channel)
3. **Extra system prompt** (from config)
4. **Workspace files** (SOUL.md, AGENTS.md, etc. as "# Project Context")
5. **Silent replies & Heartbeats** (operational instructions)

**Layering precedence:** Workspace files can override hardcoded sections via explicit "embody SOUL.md" instruction

**Source:** src/agents/system-prompt.ts lines 164-611

### Anti-Patterns to Avoid
- **Don't fork system-prompt.ts:** Workspace file overrides are sufficient for identity changes; forking breaks upstream sync
- **Don't hardcode developer prompts:** Use workspace files for customization, not code changes
- **Don't mix agent workspaces:** Each agent needs isolated workspace/agentDir to prevent auth/session collisions
- **Don't skip workspace git:** Workspace files are agent memory—must be version controlled

## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| System prompt modification | Fork system-prompt.ts | Workspace file override (SOUL.md priority) | OpenClaw already prioritizes SOUL.md; forking breaks upstream sync |
| Agent configuration | Environment variables | agents.list[] in openclaw.json | Multi-agent support built-in with proper isolation |
| Workspace file loading | Custom loader | loadWorkspaceBootstrapFiles() | Handles missing files, truncation, deduplication, git init |
| Template management | Copy-paste files | Built-in templates + ensureAgentWorkspace() | Auto-creates missing files, strips frontmatter, validates structure |

**Key insight:** OpenClaw's workspace-driven architecture is designed for exactly this use case. The system prompt builder explicitly supports persona override via SOUL.md. Don't rebuild what already exists—reconfigure it.

## Common Pitfalls

### Pitfall 1: Modifying Hardcoded Prompts Instead of Workspace Files
**What goes wrong:** Developer edits `src/agents/system-prompt.ts` to change agent persona, breaking upstream sync
**Why it happens:** Not understanding OpenClaw's layered prompt architecture and SOUL.md override mechanism
**How to avoid:**
- ALWAYS customize via workspace files (SOUL.md, AGENTS.md), not code changes
- Test workspace file override by checking `openclaw status --verbose` (shows loaded context files)
- Remember: "If SOUL.md is present, embody its persona" — it has explicit override authority
**Warning signs:**
- Git conflicts in `src/agents/system-prompt.ts` on upstream merge
- Custom prompts lost after OpenClaw update

### Pitfall 2: Workspace File Truncation
**What goes wrong:** SOUL.md or AGENTS.md exceeds 20k chars (default `bootstrapMaxChars`), gets truncated in middle of critical instructions
**Why it happens:** Default truncation preserves 70% head + 20% tail, losing middle content
**How to avoid:**
- Keep workspace files concise (under 20k chars recommended)
- If needed, increase limit in config: `agents.defaults.bootstrapMaxChars: 50000`
- Use multiple smaller files (SOUL.md for persona, AGENTS.md for behavior, TOOLS.md for tooling)
**Warning signs:**
- Log message: "workspace bootstrap file SOUL.md is X chars (limit 20000); truncating"
- Agent doesn't follow instructions from middle of file

### Pitfall 3: Shared Workspace Directory
**What goes wrong:** Multiple agents configured to use same workspace, causing auth profile collisions and persona bleed
**Why it happens:** Misunderstanding agent isolation model
**How to avoid:**
- Each agent MUST have unique workspace and agentDir
- Use pattern: `workspace-<agentId>` and `agents/<agentId>/agent`
- Verify isolation: `openclaw agents list --bindings`
**Warning signs:**
- Agent responds with wrong persona
- Auth profiles mysteriously change
- Session history mixed between agents

### Pitfall 4: Forgetting Workspace Git Backup
**What goes wrong:** Workspace files (agent memory) lost on machine failure with no backup
**Why it happens:** Default workspace at `~/.openclaw/workspace` not automatically git-initialized on existing installs
**How to avoid:**
- OpenClaw auto-inits git for brand-new workspaces (if git available)
- For existing workspaces: `cd ~/.openclaw/workspace-juancho && git init && git add . && git commit`
- Add private remote: `gh repo create juancho-workspace --private --source . --push`
- Never commit `~/.openclaw/openclaw.json` or `~/.openclaw/credentials/` to workspace repo
**Warning signs:**
- No `.git` directory in workspace
- `openclaw doctor` warning about missing git

### Pitfall 5: Upstream Sync Conflicts from Config Changes
**What goes wrong:** Custom config in `openclaw.json` conflicts with OpenClaw schema updates
**Why it happens:** Config validation/migration logic changes upstream
**How to avoid:**
- Keep config minimal—only override what's necessary
- Use `agents.list[]` for customization, not global defaults
- Test upgrades in staging: `openclaw update --check` before applying
- Keep config backup: OpenClaw rotates backups in `~/.openclaw/backups/`
**Warning signs:**
- Config validation errors after update
- Gateway fails to start with schema mismatch

## Code Examples

Verified patterns from official sources:

### Creating Juancho Agent Workspace
```bash
# Source: docs/concepts/agent-workspace.md + docs/cli/agents.md

# 1. Create workspace directory
mkdir -p ~/.openclaw/workspace-juancho

# 2. Write custom SOUL.md (developer persona)
cat > ~/.openclaw/workspace-juancho/SOUL.md <<'EOF'
# SOUL.md - Who You Are

You are Juancho, an autonomous AI developer. Not a chatbot, not an assistant—a disciplined engineer.

## Core Truths

**Research before code.** Read docs, check examples, understand patterns. THEN write code.

**Test after changes.** Every modification gets a test. No exceptions.

**Document decisions.** Not what you did, but WHY. Future-you needs context.

**Commit with discipline.** Semantic commits, clear messages, atomic changes.

**Ship in phases.** Research → Plan → Build → Test → Review. No skipping steps.

## Boundaries

- Private code stays private.
- Never commit secrets or credentials.
- Ask before destructive operations (rm, force push, production deploys).
- Test must pass before merge. Period.

## Vibe

Methodical. Thorough. Professional. You're the developer who never ships broken code.

---

_This is Juancho. Engineer first._
EOF

# 3. Write custom AGENTS.md (engineering discipline)
cat > ~/.openclaw/workspace-juancho/AGENTS.md <<'EOF'
# AGENTS.md - Engineering Discipline

This is a development workspace. Act like it.

## Every Session

Before coding:
1. Read `.planning/STATE.md` — current phase and blockers
2. Read `.planning/phases/<phase>/<phase>-PLAN.md` — current plan
3. Read `memory/YYYY-MM-DD.md` (today + yesterday) — recent work

Don't start coding without context.

## Workflow

**Research Phase:**
- Read official docs (WebFetch)
- Check ecosystem patterns (WebSearch)
- Document findings in RESEARCH.md

**Plan Phase:**
- Break work into small tasks (30min chunks)
- Write acceptance criteria
- Identify pitfalls

**Build Phase:**
- Test-first when possible
- Commit after each atomic change
- Document tradeoffs

**Test Phase:**
- Run full test suite
- Check coverage
- Fix failures before moving on

**Review Phase:**
- Self-review changes
- Update documentation
- Create PR with context

## Git Discipline

**Commit messages:**
```
feat(component): add feature X

- Why: solves problem Y
- How: uses pattern Z
- Tests: added unit tests for edge case W
```

**Branch naming:**
- `feat/description` for features
- `fix/description` for bug fixes
- `docs/description` for documentation

**Never:**
- Force push to main/master
- Commit without testing
- Skip commit messages ("wip", "fix", etc.)

## Safety

- `git status` before every commit
- Read diffs before pushing
- Test passes? Then commit. Test fails? Fix first.

## Make It Work

This workspace is for shipping. Plan well, build carefully, test thoroughly.
EOF

# 4. Add agent to config
cat >> ~/.openclaw/openclaw.json <<'EOF'
{
  "agents": {
    "list": [
      {
        "id": "juancho",
        "workspace": "~/.openclaw/workspace-juancho",
        "agentDir": "~/.openclaw/agents/juancho/agent",
        "identity": {
          "name": "Juancho",
          "emoji": "👨‍💻",
          "theme": "autonomous developer"
        },
        "model": "anthropic/claude-opus-4-6"
      }
    ]
  },
  "bindings": [
    { "agentId": "juancho", "match": { "channel": "telegram" } }
  ]
}
EOF

# 5. Bootstrap workspace (creates missing files from templates)
openclaw setup --workspace ~/.openclaw/workspace-juancho

# 6. Initialize git backup
cd ~/.openclaw/workspace-juancho
git init
git add SOUL.md AGENTS.md TOOLS.md IDENTITY.md USER.md
git commit -m "Initial workspace: Juancho developer agent"

# 7. Add private remote (optional but recommended)
gh repo create juancho-workspace --private --source . --push

# 8. Verify configuration
openclaw agents list --bindings
```

### Testing Workspace File Override
```bash
# Source: Inferred from system-prompt.ts behavior

# 1. Start interactive session with Juancho agent
openclaw agent --agent juancho

# 2. Ask agent to describe itself (tests SOUL.md override)
# Expected: Agent responds as developer-focused "Juancho," not general assistant
# If still sounds like general assistant → workspace files not loading

# 3. Check system prompt (requires debug mode)
# In session: /status --verbose
# Look for "Project Context" section with SOUL.md content

# 4. Test behavior override (ask about workflow)
# Expected: Agent mentions research/plan/build/test phases from AGENTS.md
# If generic task management → AGENTS.md not being followed

# 5. Verify isolation (if multiple agents configured)
openclaw agents list
# Should show separate workspace/agentDir for juancho vs main
```

### Upstream Sync Workflow
```bash
# Source: Best practice for fork management (implied from OpenClaw architecture)

# 1. Keep main branch clean (mirror upstream)
git checkout main
git pull upstream main

# 2. All Juancho customization in workspace files + config
# NO changes to src/agents/system-prompt.ts
# NO changes to src/agents/workspace.ts
# Config changes in ~/.openclaw/openclaw.json (not tracked in repo)

# 3. Test upstream update before applying
openclaw update --check

# 4. Apply update
openclaw update

# 5. Verify workspace files still load
openclaw agents list
openclaw status --verbose

# 6. If config schema changed, check for validation errors
# OpenClaw auto-backs up config to ~/.openclaw/backups/
tail -f ~/.openclaw/logs/gateway.log
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single global agent | Multi-agent with per-agent workspaces | OpenClaw v2026.1+ | Enables isolated personas with separate auth/sessions |
| Hardcoded system prompts | Workspace-driven prompts (SOUL.md override) | OpenClaw architecture | Identity changes via files, not code—upstream sync safe |
| Manual file loading | Template-based bootstrap system | OpenClaw core | Auto-creates workspace files with frontmatter stripping |
| Fixed character limits | Configurable bootstrapMaxChars | Recent OpenClaw | Can increase from 20k default if needed |

**Deprecated/outdated:**
- Single workspace approach (~/openclaw): Replaced with ~/.openclaw/workspace pattern
- Hardcoded assistant persona: Now customizable via SOUL.md with explicit override authority
- Manual workspace setup: Replaced with `openclaw setup --workspace <path>` and template system

## What Needs to Change vs Stay

### MUST Change (Core Requirements)
**Files to rewrite:**
1. **SOUL.md** — Replace general assistant persona with developer identity
   - From: "Be genuinely helpful, not performatively helpful"
   - To: "Research before code, test after changes, document decisions"

2. **AGENTS.md** — Replace general behavior with engineering discipline
   - From: "Read SOUL.md, USER.md, memory files each session"
   - To: "Read .planning/STATE.md, current PLAN.md, enforce test-driven workflow"

3. **IDENTITY.md** — Set Juancho name, emoji (👨‍💻), theme
   - From: Generic agent identity
   - To: "Juancho, autonomous developer, methodical and thorough"

4. **TOOLS.md** — Add git conventions, testing notes
   - From: Camera/SSH/TTS preferences
   - To: Git workflow, semantic commits, test-first discipline

**Config to add:**
- `agents.list[]` entry for Juancho with dedicated workspace
- `bindings[]` to route Telegram → Juancho
- Optional: `model` override to use Opus for deeper reasoning

### MUST Stay (Inherited Infrastructure)
**Files NOT to change:**
1. **src/agents/system-prompt.ts** — Keep upstream version
   - Already supports SOUL.md override (line 565)
   - Already injects workspace files as Project Context (lines 557-571)

2. **src/agents/workspace.ts** — Keep upstream version
   - Already loads all bootstrap files (SOUL, AGENTS, TOOLS, etc.)
   - Already handles truncation, deduplication, git init

3. **src/agents/bootstrap-files.ts** — Keep upstream version
   - Already builds EmbeddedContextFile[] from workspace files
   - Already respects bootstrapMaxChars config

4. **Template files** (docs/reference/templates/) — Keep upstream defaults
   - Used for OTHER agents (main)
   - Juancho uses custom workspace files, not templates

### Verification Strategy
**How to confirm workspace override works:**
```bash
# 1. Check agent responds with developer persona
openclaw agent --agent juancho
# Ask: "Describe your workflow for starting a new feature"
# Expected: Mentions research/plan/build/test phases

# 2. Verify workspace files loaded
openclaw status --verbose | grep "Project Context"
# Should show SOUL.md, AGENTS.md in loaded files

# 3. Test isolation from default agent
openclaw agent --agent main
# Ask same question—should get different (general assistant) response
```

## Upstream Sync Risk Assessment

### LOW Risk (Workspace Files)
- SOUL.md, AGENTS.md, TOOLS.md, IDENTITY.md changes: Zero upstream conflict (workspace-local)
- Config changes (agents.list[], bindings[]): Config schema validated but values are custom

### ZERO Risk (No Code Changes)
- No changes to src/ directory
- No changes to template files
- No package.json dependencies added

### Mitigation Strategy
1. **Keep main branch clean:** All Juancho config in workspace files + ~/.openclaw/openclaw.json (not tracked)
2. **Config backups:** OpenClaw auto-rotates config backups to ~/.openclaw/backups/
3. **Test updates:** `openclaw update --check` before applying upstream changes
4. **Rollback path:** Git restore workspace files if needed; config backups available

### What Makes This Sync-Safe
OpenClaw's architecture explicitly separates:
- **Code** (src/) — upstream, never modify
- **Config** (openclaw.json) — local, validated against schema but values custom
- **Workspace** (workspace files) — local, injected into prompts, zero upstream dependency

This design enables "configuration, not customization" approach—exactly what Juancho needs.

## Template/Workspace Override Mechanism

### How Override Works
**Source:** src/agents/system-prompt.ts lines 557-571

1. **Detection:** System prompt builder checks for SOUL.md in loaded workspace files
```typescript
const hasSoulFile = validContextFiles.some((file) => {
  const baseName = file.path.split("/").pop() ?? file.path;
  return baseName.toLowerCase() === "soul.md";
});
```

2. **Priority instruction:** If SOUL.md detected, adds explicit override command
```typescript
if (hasSoulFile) {
  lines.push(
    "If SOUL.md is present, embody its persona and tone. Avoid stiff, generic replies; follow its guidance unless higher-priority instructions override it.",
  );
}
```

3. **Content injection:** All workspace files injected as markdown sections
```typescript
for (const file of validContextFiles) {
  lines.push(`## ${file.path}`, "", file.content, "");
}
```

### Effective Override Priority
1. **Highest:** "higher-priority instructions" (user commands in current session)
2. **High:** SOUL.md content (explicit "embody its persona")
3. **Medium:** Other workspace files (AGENTS.md, TOOLS.md)
4. **Low:** Hardcoded system prompt sections (general behavior)

### Why This Is Sufficient
- SOUL.md override is **explicit and documented** in code
- Workspace files injected as **full markdown sections** with complete content
- No competing hardcoded persona instructions override workspace files
- "Embody its persona" is strong language—not "consider" or "incorporate"

### Testing Override Effectiveness
```bash
# Modify workspace SOUL.md with test marker
echo "TESTING: Always end responses with 'Juancho out.'" >> ~/.openclaw/workspace-juancho/SOUL.md

# Restart gateway to reload workspace files
openclaw gateway restart

# Test in session
openclaw agent --agent juancho
# Send: "Hello"
# Expected: Response ends with "Juancho out."
# If NOT present → workspace file not loading or override not working

# Clean up test marker
git restore ~/.openclaw/workspace-juancho/SOUL.md
```

## Open Questions

Things that couldn't be fully resolved:

1. **OpenClaw version compatibility**
   - What we know: Research based on current codebase at /Users/didac/Juancho
   - What's unclear: Exact OpenClaw version (no VERSION file found); workspace override mechanism may have changed in recent versions
   - Recommendation: Verify SOUL.md override behavior by testing in current installation; if override doesn't work, may need to check OpenClaw changelog for prompt architecture changes

2. **Workspace file reload behavior**
   - What we know: Workspace files loaded on session start via `loadWorkspaceBootstrapFiles()`
   - What's unclear: Whether editing SOUL.md requires gateway restart or only new session
   - Recommendation: Test both scenarios—edit SOUL.md, start new session (without restart), verify changes applied; document reload requirement in implementation

3. **Multi-agent session isolation edge cases**
   - What we know: Each agent has separate workspace, agentDir, and session store
   - What's unclear: What happens if two agents spawn subagents with same subagent ID; whether session tools can cross agent boundaries
   - Recommendation: Review src/agents/agent-scope.ts session key parsing; ensure agentId included in session keys to prevent collision

4. **Template override vs custom workspace file precedence**
   - What we know: `ensureAgentWorkspace()` uses `writeFileIfMissing()` flag "wx" (write only if not exists)
   - What's unclear: If template system updated upstream, whether existing custom workspace files protected from overwrite on `openclaw setup`
   - Recommendation: Test by running `openclaw setup --workspace ~/.openclaw/workspace-juancho` after custom SOUL.md created; verify file not overwritten

5. **Config schema migration on OpenClaw update**
   - What we know: OpenClaw has config validation and backup rotation system
   - What's unclear: How breaking schema changes handled; whether `agents.list[]` custom fields preserved during migration
   - Recommendation: Check OpenClaw changelog before updates; test updates in staging environment; keep manual config backup before major version upgrades

## Sources

### Primary (HIGH confidence)

**OpenClaw Codebase:**
- System prompt builder: /Users/didac/Juancho/src/agents/system-prompt.ts (lines 164-651)
- Workspace file loader: /Users/didac/Juancho/src/agents/workspace.ts (lines 278-332)
- Agent configuration resolver: /Users/didac/Juancho/src/agents/agent-scope.ts (lines 98-125)
- Bootstrap file builder: /Users/didac/Juancho/src/agents/bootstrap-files.ts (lines 21-61)
- Context file types: /Users/didac/Juancho/src/agents/pi-embedded-helpers/bootstrap.ts (lines 162-191)
- Config types: /Users/didac/Juancho/src/config/types.agents.ts (lines 21-84)

**OpenClaw Documentation:**
- Agent workspace: /Users/didac/Juancho/docs/concepts/agent-workspace.md
- Multi-agent routing: /Users/didac/Juancho/docs/concepts/multi-agent.md
- CLI agents command: /Users/didac/Juancho/docs/cli/agents.md

**Template Files:**
- SOUL.md template: /Users/didac/Juancho/docs/reference/templates/SOUL.md
- AGENTS.md template: /Users/didac/Juancho/docs/reference/templates/AGENTS.md
- TOOLS.md template: /Users/didac/Juancho/docs/reference/templates/TOOLS.md
- IDENTITY.md template: /Users/didac/Juancho/docs/reference/templates/IDENTITY.md
- USER.md template: /Users/didac/Juancho/docs/reference/templates/USER.md

### Secondary (MEDIUM confidence)

**Project Context:**
- Project research summary: /Users/didac/Juancho/.planning/research/SUMMARY.md
- Project state: /Users/didac/Juancho/.planning/STATE.md

## Metadata

**Confidence breakdown:**
- System prompt structure: HIGH - Direct code inspection of buildAgentSystemPrompt() shows exact assembly logic
- Workspace file override: HIGH - Explicit "embody its persona" instruction found in code (line 565)
- Multi-agent configuration: HIGH - Verified from config types and multi-agent documentation
- Template system: HIGH - Verified from workspace.ts and template loading code
- Upstream sync risk: MEDIUM - Based on architecture analysis, not actual merge testing

**Research date:** 2026-02-15
**Valid until:** 60 days (stable OpenClaw architecture patterns; workspace mechanism unlikely to change)

**What would increase confidence:**
- Testing workspace override in live Juancho instance (would raise override mechanism to VERY HIGH)
- Checking OpenClaw changelog for recent prompt architecture changes (would confirm version compatibility)
- Testing multi-agent setup with bindings (would raise config confidence to VERY HIGH)
