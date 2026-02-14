# Architecture Patterns

**Domain:** Autonomous AI developer agent built on OpenClaw
**Researched:** 2026-02-14
**Confidence:** HIGH

## Executive Summary

OpenClaw is a TypeScript-based personal AI assistant platform with a Gateway-centric architecture. The system uses a WebSocket-based control plane (Gateway) that orchestrates agents, channels (Telegram, Slack, etc.), tools, and sessions. The Pi agent runtime executes inside the Gateway process, with system prompts built dynamically from workspace files and agent configuration.

For Juancho (GSD pipeline integration), the key insight is that OpenClaw's architecture is **already designed for workflow orchestration** — it just needs new prompts, new workspace structures, and workflow tooling layered on top.

## Recommended Architecture for Juancho

Juancho should be built as:

1. **Rewritten system prompts** (developer-focused, GSD-aware)
2. **Extended workspace structure** (`.planning/` directory for GSD artifacts)
3. **New GSD tools** (orchestrators, researchers, testers)
4. **Agent configuration** (Juancho as a specialized agent in `agents.list`)
5. **Telegram as primary channel** (already supported)

**Do NOT fork Gateway or create parallel systems.** Extend what exists.

---

## OpenClaw Component Structure

### Top-Level Directory Structure

```
/Users/didac/Juancho/
├── src/                      # Core TypeScript source
│   ├── agents/              # Agent runtime, system prompt building, Pi integration
│   ├── gateway/             # Gateway server (WebSocket control plane)
│   ├── commands/            # CLI command implementations
│   ├── cli/                 # CLI entry point and program registration
│   ├── channels/            # Channel plugins (routing layer)
│   ├── config/              # Configuration loading, validation, schemas
│   ├── auto-reply/          # Reply dispatching, triggers, agent orchestration
│   ├── browser/             # Browser control (Stagehand, Puppeteer)
│   ├── telegram/            # Telegram channel integration
│   ├── web/                 # Control UI (served by Gateway)
│   ├── infra/               # Infrastructure (sessions, workspace, tools)
│   ├── memory/              # Memory search, vector storage
│   ├── cron/                # Cron jobs, wake events
│   ├── hooks/               # Bundled hooks (session-memory, bootstrap files)
│   └── plugins/             # Plugin system, tool registration
│
├── extensions/              # Channel plugins (Discord, Matrix, BlueBubbles, etc.)
├── docs/                    # Documentation (Mintlify)
├── .agents/                 # Agent instructions, skills
│   └── skills/             # PR workflow, Mintlify, etc.
├── package.json             # Root package
└── pnpm-workspace.yaml     # Workspace configuration
```

### Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| **Gateway** | Control plane, WebSocket server, session orchestration, channel routing | Agents (Pi runtime), Channels (Telegram, etc.), Tools, Browser, Nodes |
| **Agents** | Pi agent runtime, system prompt building, tool execution, session management | Gateway (as embedded runtime), Workspace (file I/O), Channels (reply delivery) |
| **Channels** | Message ingress/egress, platform-specific routing (Telegram, Slack, Discord) | Gateway (via channel plugins), Auto-Reply (triggers), Sessions |
| **Auto-Reply** | Trigger detection, reply dispatching, agent invocation | Agents (Pi runtime), Sessions, Channels (delivery) |
| **Config** | Configuration loading, validation, agent routing | Gateway (runtime config), Agents (agent-specific config), Channels |
| **Workspace** | File-based agent context (SOUL.md, TOOLS.md, AGENTS.md, MEMORY.md) | Agents (system prompt), Pi runtime (context files), Skills |
| **CLI** | Command-line interface, wizard, onboarding | Gateway (via RPC), Config, Agents (direct invocation) |
| **Browser** | Browser control, Stagehand/Puppeteer wrapper | Tools (exposed as `browser` tool), Gateway (lifecycle) |
| **Memory** | Memory search, vector storage, embedding | Agents (memory tools), Sessions (context retrieval) |
| **Cron** | Scheduled tasks, wake events, isolated agent runs | Gateway (cron service), Agents (isolated execution) |

---

## System Prompt and Agent Instructions: Where They Live

### System Prompt Generation (Runtime)

**Location:** `src/agents/system-prompt.ts` → `buildAgentSystemPrompt()`

This function **dynamically constructs the system prompt** from:

1. **Hardcoded sections** (Tooling, Safety, OpenClaw CLI Quick Reference)
2. **Agent configuration** (`agents.list[].identity`, `agents.list[].groupChat`)
3. **Workspace files** (loaded by `src/agents/workspace.ts`):
   - `SOUL.md` → persona/tone
   - `TOOLS.md` → user-specific tool notes
   - `AGENTS.md` → agent instructions (injected as context)
   - `IDENTITY.md` → user identity
   - `USER.md` → user preferences
   - `HEARTBEAT.md` → heartbeat prompt
   - `MEMORY.md` → memory notes
4. **Skills prompt** (from skills registry, workspace skills)
5. **Runtime info** (agent ID, host, OS, arch, node version, model, channel, capabilities)
6. **Sandbox info** (if enabled)
7. **Context files** (explicitly injected via `contextFiles` param)

**Prompt Modes:**
- `"full"` (default, main agent): All sections
- `"minimal"` (subagents): Reduced sections (Tooling, Workspace, Runtime)
- `"none"` (special): Basic identity line only

**Key insight for Juancho:** To change agent behavior, you **rewrite workspace files** (SOUL.md, AGENTS.md) and **add sections to the system prompt builder** (e.g., GSD workflow instructions).

### Agent Instructions Storage

**Location:** `.agents/skills/` (workspace-level skills)

Examples:
- `.agents/skills/PR_WORKFLOW.md` → Maintainer PR workflow
- `.agents/skills/prepare-pr/SKILL.md` → Prepare PR skill
- `.agents/skills/review-pr/SKILL.md` → Review PR skill
- `.agents/skills/merge-pr/SKILL.md` → Merge PR skill

Skills are **read dynamically** when needed (not injected into every prompt). The system prompt includes:

```markdown
## Skills (mandatory)
Before replying: scan <available_skills> <description> entries.
- If exactly one skill clearly applies: read its SKILL.md at <location> with `read`, then follow it.
- If multiple could apply: choose the most specific one, then read/follow it.
- If none clearly apply: do not read any SKILL.md.
```

**Key insight for Juancho:** GSD workflow can be implemented as **workspace skills** or **injected into the system prompt** via AGENTS.md.

---

## Agent Execution Loop

### High-Level Flow

```
1. Message arrives → Channel (Telegram, Slack, etc.)
2. Channel plugin → Gateway (via channel.send event)
3. Gateway → Auto-Reply (trigger detection)
4. Auto-Reply → Session resolution (sessionKey, agentId)
5. Agent runtime → Pi agent (embedded in Gateway)
6. Pi agent → Tool execution (exec, browser, memory, etc.)
7. Tool results → Pi agent (streaming or blocking)
8. Pi agent → Reply text
9. Reply → Channel (via Gateway routing)
10. Channel → Telegram/Slack/etc. (message delivered)
```

### Detailed Flow with Files

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Telegram message arrives                                │
│    /extensions/telegram/src/monitor.ts                     │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Channel plugin dispatches to Gateway                    │
│    /src/channels/plugins/index.ts                          │
│    → Gateway.channels.send()                               │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Gateway triggers auto-reply                             │
│    /src/gateway/server-chat.ts → createAgentEventHandler() │
│    → /src/auto-reply/reply/agent-runner.ts                 │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Resolve agent and workspace                             │
│    /src/agents/agent-scope.ts                              │
│    → resolveAgentWorkspaceDir()                            │
│    → resolveAgentDir()                                     │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. Load workspace files                                     │
│    /src/agents/workspace.ts                                │
│    → loadWorkspaceBootstrapFiles()                         │
│    Reads: SOUL.md, TOOLS.md, AGENTS.md, MEMORY.md, etc.   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. Build system prompt                                      │
│    /src/agents/system-prompt.ts                            │
│    → buildAgentSystemPrompt()                              │
│    Injects workspace files as "Project Context"            │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. Run Pi agent                                             │
│    /src/agents/pi-embedded-runner/run.ts                   │
│    → runEmbeddedPiAgent()                                  │
│    Executes model API call with tools                      │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 8. Tool execution (exec, browser, memory, etc.)            │
│    /src/agents/tools/                                      │
│    Tools stream results back to Pi agent                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 9. Reply text generated                                     │
│    /src/agents/pi-embedded-subscribe.ts                    │
│    → subscribeEmbeddedPiSession()                          │
│    Streams or blocks until complete                        │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ 10. Deliver reply to channel                                │
│    /src/auto-reply/reply/normalize-reply.ts                │
│    → /extensions/telegram/src/send.ts                      │
│    → Telegram API                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Configuration: Where Agent Settings Live

### Configuration File

**Location:** `~/.openclaw/config.json` (or `OPENCLAW_CONFIG_PATH`)

**Schema:** `src/config/types.ts` → `OpenClawConfig`

Example agent configuration:

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace"
    },
    "list": [
      {
        "id": "juancho",
        "name": "Juancho",
        "default": false,
        "workspace": "~/projects/juancho-workspace",
        "agentDir": "~/.openclaw/agents/juancho/agent",
        "model": {
          "primary": "anthropic/claude-sonnet-4.5",
          "fallbacks": ["openai/gpt-5.3"]
        },
        "skills": ["*"],
        "identity": {
          "name": "Juancho",
          "avatar": "🤖"
        },
        "sandbox": {
          "enabled": false
        },
        "tools": {
          "allowlist": ["*"],
          "denylist": []
        }
      }
    ]
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "routing": {
        "default": "juancho"
      }
    }
  }
}
```

### Agent Resolution

**Location:** `src/agents/agent-scope.ts`

- `resolveDefaultAgentId()` → Default agent from config
- `resolveSessionAgentId()` → Agent for specific session
- `resolveAgentWorkspaceDir()` → Workspace directory for agent
- `resolveAgentDir()` → Agent-specific directory (`~/.openclaw/agents/<agentId>/agent`)

### Workspace Directory Structure

```
~/.openclaw/workspace/          # Default workspace (or agent-specific)
├── SOUL.md                    # Persona/tone
├── TOOLS.md                   # User-specific tool notes
├── AGENTS.md                  # Agent instructions (high-level)
├── IDENTITY.md                # User identity
├── USER.md                    # User preferences
├── HEARTBEAT.md               # Heartbeat prompt
├── MEMORY.md                  # Memory notes
└── memory/                    # Memory subdirectory
    └── *.md                   # Additional memory files
```

**For Juancho, add:**

```
~/.openclaw/workspace-juancho/  # Juancho-specific workspace
├── SOUL.md                    # Developer persona
├── TOOLS.md                   # Git config, test commands
├── AGENTS.md                  # GSD workflow instructions
├── .planning/                 # GSD artifacts (NEW)
│   ├── research/             # Research outputs
│   ├── requirements/         # Requirements analysis
│   ├── roadmap/              # Phase roadmaps
│   └── phases/               # Phase execution state
├── memory/                    # Project memory
└── projects/                  # Active projects
```

---

## Data Flow: Telegram Message → Agent → Code → PR

### Typical Task Flow

```
1. User sends Telegram message: "Build a new feature X"
   ↓
2. Telegram channel plugin receives message
   → /extensions/telegram/src/monitor.ts
   ↓
3. Gateway routes to Juancho agent (via routing config)
   → /src/gateway/server-chat.ts
   ↓
4. Auto-reply triggers agent execution
   → /src/auto-reply/reply/agent-runner.ts
   ↓
5. Agent workspace loaded
   → SOUL.md (developer persona)
   → AGENTS.md (GSD workflow instructions)
   → .planning/ directory (previous projects)
   ↓
6. System prompt built with GSD context
   → /src/agents/system-prompt.ts
   "You are a developer. Follow GSD methodology. Before building, research."
   ↓
7. Pi agent executes with tools:
   → exec (git, npm, test commands)
   → browser (research, documentation)
   → memory (project context)
   → Custom GSD tools (orchestrators, researchers)
   ↓
8. Agent outputs:
   → .planning/research/SUMMARY.md
   → .planning/requirements/REQUIREMENTS.md
   → .planning/roadmap/ROADMAP.md
   → Git commits (phase-based)
   ↓
9. Agent replies to Telegram:
   "Phase 1 complete. Review the roadmap at .planning/roadmap/ROADMAP.md"
   ↓
10. User can inspect, approve, or iterate
```

---

## Where GSD Pipeline Integrates

### 1. System Prompt Extension

**File:** `src/agents/system-prompt.ts`

Add GSD workflow instructions to `buildAgentSystemPrompt()`:

```typescript
const gsdSection = buildGSDSection({
  workspaceDir: params.workspaceDir,
  planningDir: path.join(params.workspaceDir, '.planning'),
});
lines.push(...gsdSection);
```

Or inject via workspace files:

**File:** `~/.openclaw/workspace-juancho/AGENTS.md`

```markdown
# Juancho Developer Agent

You are an autonomous AI developer. You build software using GSD methodology:

1. **Research:** Understand the domain before building
2. **Requirements:** Define what to build (not how)
3. **Roadmap:** Break work into phases
4. **Execution:** Build, test, commit per phase
5. **Verification:** Validate each phase before proceeding

Output artifacts to `.planning/` directory. Follow phase structure.
```

### 2. GSD Tools (New)

**Location:** `src/agents/tools/gsd/` (NEW)

Create custom tools:

- `gsd_orchestrator` → Orchestrate GSD pipeline
- `gsd_researcher` → Run research phase
- `gsd_requirements` → Define requirements
- `gsd_roadmap` → Create phase roadmap
- `gsd_phase_executor` → Execute phase
- `gsd_verifier` → Verify phase completion

Register in `src/agents/tools/index.ts` (or as plugin tools).

### 3. Workspace Structure Extension

**Already supported:** OpenClaw loads workspace files dynamically.

**Extension:** Add `.planning/` directory handling:

**File:** `src/agents/workspace.ts`

Add `.planning/` awareness:

```typescript
export const DEFAULT_PLANNING_DIR = ".planning";

export async function loadPlanningState(workspaceDir: string) {
  const planningDir = path.join(workspaceDir, DEFAULT_PLANNING_DIR);
  // Load current project, active phase, etc.
}
```

### 4. Agent Configuration

**File:** `~/.openclaw/config.json`

Add Juancho agent:

```json
{
  "agents": {
    "list": [
      {
        "id": "juancho",
        "name": "Juancho",
        "default": true,
        "workspace": "~/projects/juancho-workspace",
        "model": {
          "primary": "anthropic/claude-sonnet-4.5"
        },
        "skills": ["gsd-orchestrator", "gsd-researcher"],
        "tools": {
          "allowlist": ["exec", "browser", "memory", "gsd_*"]
        }
      }
    ]
  }
}
```

### 5. Telegram Routing

**Already supported:** Telegram channel routes to agent via config.

**File:** `~/.openclaw/config.json`

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "routing": {
        "default": "juancho"
      }
    }
  }
}
```

---

## Architecture Patterns to Follow

### Pattern 1: Workspace-Driven Behavior

**What:** Agent behavior is controlled by workspace files, not hardcoded logic.

**When:** You want configurable, per-agent behavior.

**Example:**

```markdown
# SOUL.md (Developer Persona)

You are Juancho, an autonomous AI developer. You:
- Build software methodically using GSD
- Always research before building
- Document everything
- Commit frequently, in small increments
```

**Why:** Avoids hardcoding prompts in code. Users can customize by editing workspace files.

### Pattern 2: Tool-Based Workflows

**What:** Workflows are implemented as tools that agents call.

**When:** You want reusable, composable workflow steps.

**Example:**

```typescript
// src/agents/tools/gsd/orchestrator.ts
export const gsdOrchestratorTool = {
  name: "gsd_orchestrator",
  description: "Orchestrate GSD pipeline for a project",
  inputSchema: {
    type: "object",
    properties: {
      action: { type: "string", enum: ["start", "next-phase", "verify"] },
      projectName: { type: "string" },
    },
  },
  execute: async (params) => {
    // Load .planning/state.json
    // Determine current phase
    // Return next action
  },
};
```

**Why:** Tools are first-class in OpenClaw. Agents know how to use them.

### Pattern 3: Session-Based State

**What:** State is stored in session files or workspace directories, not in-memory.

**When:** You need state that persists across restarts.

**Example:**

```
~/.openclaw/workspace-juancho/.planning/state.json
{
  "currentProject": "feature-x",
  "currentPhase": "2-requirements",
  "phaseStatus": "in-progress",
  "lastUpdated": "2026-02-14T23:15:00Z"
}
```

**Why:** OpenClaw is stateless by design. State lives in files.

### Pattern 4: Layered Prompts

**What:** System prompt is built from layers (hardcoded + config + workspace + runtime).

**When:** You want flexible, context-aware prompts.

**Example:**

```
System Prompt =
  Hardcoded sections (Safety, Tooling) +
  Config (agent.identity) +
  Workspace files (SOUL.md, AGENTS.md) +
  Runtime info (model, channel, capabilities)
```

**Why:** Prompts adapt to agent, workspace, and runtime context.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Forking Gateway Logic

**What goes wrong:** Creating a parallel Gateway or agent runtime.

**Why bad:** OpenClaw's Gateway is the control plane. Forking it creates maintenance hell.

**Instead:** Extend Gateway via plugins or workspace configuration.

### Anti-Pattern 2: Hardcoding Workflow in Code

**What goes wrong:** Embedding GSD steps directly in TypeScript.

**Why bad:** Inflexible, hard to iterate, not user-configurable.

**Instead:** Implement workflow as tools or workspace-driven prompts.

### Anti-Pattern 3: Bypassing Agent Runtime

**What goes wrong:** Writing custom agent execution logic instead of using `runEmbeddedPiAgent()`.

**Why bad:** Breaks tool execution, session management, streaming, error handling.

**Instead:** Use Pi runtime, extend it via tools or prompt engineering.

### Anti-Pattern 4: Global State in Memory

**What goes wrong:** Storing project state in JavaScript variables.

**Why bad:** Lost on restart, not inspectable, breaks multi-agent scenarios.

**Instead:** Store state in workspace files (`.planning/state.json`).

---

## Scalability Considerations

| Concern | At 1 user | At 10 users | At 100 users |
|---------|-----------|-------------|--------------|
| **Gateway** | Single process, embedded agent runtime | Same (personal assistant model) | Not applicable (not multi-tenant) |
| **Workspace** | Single workspace directory | Multiple agent workspaces via `agents.list` | N/A |
| **Sessions** | File-based sessions in `~/.openclaw/agents/<agentId>/sessions/` | Same (agent-scoped) | N/A |
| **Channels** | Telegram (single bot) | Same (routing via `channels.telegram.routing`) | N/A |
| **Browser** | Single browser instance | Multiple profiles or containers | N/A |

**Key insight:** OpenClaw is designed for **single-user, personal use**. Juancho inherits this model. Multi-user support is out of scope.

---

## Build Order Implications

Based on architecture, suggested build order for Juancho:

1. **Phase 1: System Prompt Rewrite**
   - Modify `SOUL.md`, `AGENTS.md` for developer persona
   - Test with existing OpenClaw setup

2. **Phase 2: Workspace Extension**
   - Add `.planning/` directory handling
   - Create GSD workspace templates

3. **Phase 3: GSD Tools**
   - Implement `gsd_orchestrator`, `gsd_researcher` tools
   - Register tools in agent configuration

4. **Phase 4: Agent Configuration**
   - Create Juancho agent in `config.json`
   - Configure Telegram routing

5. **Phase 5: Onboarding Flow**
   - Customize onboarding wizard for developer context
   - Generate initial `.planning/` structure

6. **Phase 6: Testing & Iteration**
   - End-to-end test with real feature requests
   - Iterate on prompts, tools, workflow

**Rationale:** Start with prompts (lowest risk), then extend workspace, then add tools. Configuration and onboarding come after core mechanics work.

---

## Sources

- OpenClaw codebase: `/Users/didac/Juancho/`
- System prompt builder: `src/agents/system-prompt.ts`
- Agent scope resolution: `src/agents/agent-scope.ts`
- Workspace file loading: `src/agents/workspace.ts`
- Gateway server implementation: `src/gateway/server.impl.ts`
- Pi agent runtime: `src/agents/pi-embedded-runner/run.ts`
- Auto-reply orchestration: `src/auto-reply/reply/agent-runner.ts`
- Configuration schema: `src/config/types.ts`
- Skills example: `.agents/skills/PR_WORKFLOW.md`
- OpenClaw repository README: `README.md`
- AGENTS.md contributor guide: `AGENTS.md`

**Confidence:** HIGH — Based on direct code inspection and architectural patterns evident in the codebase.
