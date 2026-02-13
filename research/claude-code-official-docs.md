# Claude Code Official Documentation Reference
Date: 2026-02-09
Sources:
- https://code.claude.com/docs/en/agent-teams
- https://code.claude.com/docs/en/sub-agents

## Agent Teams

### Enable
Experimental, disabled by default. Requires:
```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```
Requires Claude Code 1.0.34+.

### Architecture

| Component     | Role                                                                                       |
|:------------- |:-------------------------------------------------------------------------------------------|
| **Team lead** | Main Claude Code session that creates team, spawns teammates, coordinates work              |
| **Teammates** | Separate Claude Code instances, each with own context window, tools, conversation history   |
| **Task list** | Shared list of work items that teammates claim and complete                                 |
| **Mailbox**   | Messaging system for communication between agents                                          |

### Key Constraints
- **One team per session**: Lead can only manage one team at a time. Clean up before starting new one.
- **No nested teams**: Teammates cannot spawn their own teams or teammates. Only lead manages team.
- **Lead is fixed**: Session that creates team is lead for lifetime. Can't promote teammate.
- **Permissions set at spawn**: All teammates start with lead's permission mode. Can change after spawning but can't set per-teammate at spawn.
- **No session resumption**: `/resume` and `/rewind` do not restore in-process teammates.
- **Context isolation**: Teammates load same project context (CLAUDE.md, MCP servers, skills) but do NOT inherit lead's conversation history. Task files on disk and SendMessage are the ONLY coordination channels.

### Display Modes
- **In-process** (default): All teammates in main terminal. Shift+Up/Down to select.
- **Split panes**: Each teammate gets own pane. Requires tmux or iTerm2.

Setting: `"teammateMode": "in-process"` or `"tmux"` or `"auto"` (default).

### Team Configuration Storage
- Team config: `~/.claude/teams/{team-name}/config.json`
- Task list: `~/.claude/tasks/{team-name}/`
- Config contains `members` array: name, agentId, agentType per teammate.

### Communication

**SendMessage types:**
- `message`: DM to one specific teammate (by name)
- `broadcast`: Send to all teammates (use sparingly, costs scale with team size)
- `shutdown_request`: Ask teammate to gracefully shut down
- `shutdown_response`: Teammate approves or rejects shutdown
- `plan_approval_response`: Approve/reject teammate's plan

**Automatic delivery:**
- Messages delivered automatically when teammate is idle
- Idle notifications auto-sent when teammate's turn ends
- Shared task list visible to all agents

### Task Coordination

**Task states:** pending → in_progress → completed
**Dependencies:** Tasks can depend on other tasks. Blocked tasks auto-unblock when dependencies complete.
**Claiming:** File locking prevents race conditions. Lead can assign or teammates self-claim.

### Delegate Mode
Press Shift+Tab to enable. Restricts lead to coordination-only tools — spawning, messaging, shutting down teammates, managing tasks. Prevents lead from implementing tasks itself.

### Hooks for Quality Gates
- `TeammateIdle`: Runs when teammate about to go idle. Exit code 2 sends feedback and keeps teammate working.
- `TaskCompleted`: Runs when task being marked complete. Exit code 2 prevents completion with feedback.

### Graceful Shutdown
1. Lead sends `shutdown_request` to each teammate via SendMessage
2. Teammates finalize work, confirm with `shutdown_response`
3. Lead runs `TeamDelete` after all teammates acknowledged
4. Always use lead to clean up (not teammates)

### When to Use Agent Teams vs Subagents

|                   | Subagents                                        | Agent teams                                         |
|:------------------|:-------------------------------------------------|:----------------------------------------------------|
| **Context**       | Own context window; results return to caller     | Own context window; fully independent               |
| **Communication** | Report results back to main agent only           | Teammates message each other directly               |
| **Coordination**  | Main agent manages all work                      | Shared task list with self-coordination             |
| **Best for**      | Focused tasks where only the result matters      | Complex work requiring discussion and collaboration |
| **Token cost**    | Lower: results summarized back to main context   | Higher: each teammate is separate Claude instance   |

### Best Practices
- Give teammates enough context in spawn prompt (they don't inherit lead's conversation)
- Size tasks appropriately: not too small (coordination overhead) or too large (wasted effort)
- 5-6 tasks per teammate keeps everyone productive
- Start with research/review before parallel implementation
- Avoid file conflicts: each teammate owns different files
- Monitor and steer: check in on progress, redirect as needed

---

## Subagents (Custom)

### Definition Format
Markdown files with YAML frontmatter at:
- `.claude/agents/` (project-level, priority 2)
- `~/.claude/agents/` (user-level, priority 3)
- `--agents` CLI flag (session-level, priority 1)
- Plugin's `agents/` directory (priority 4)

### Supported Frontmatter Fields

| Field             | Required | Description                                                |
|:------------------|:---------|:-----------------------------------------------------------|
| `name`            | Yes      | Unique identifier, lowercase letters and hyphens           |
| `description`     | Yes      | When Claude should delegate to this subagent               |
| `tools`           | No       | Tools the subagent can use (inherits all if omitted)       |
| `disallowedTools` | No       | Tools to deny                                              |
| `model`           | No       | `sonnet`, `opus`, `haiku`, or `inherit` (default)          |
| `permissionMode`  | No       | `default`, `acceptEdits`, `delegate`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns`        | No       | Max agentic turns before stopping                          |
| `skills`          | No       | Skills to preload into context at startup                  |
| `mcpServers`      | No       | MCP servers available to subagent                          |
| `hooks`           | No       | Lifecycle hooks scoped to this subagent                    |
| `memory`          | No       | Persistent memory: `user`, `project`, or `local`           |

### Skills Preloading
```yaml
skills:
  - api-conventions
  - error-handling-patterns
```
Full skill content injected into subagent's context at startup. Subagents DON'T inherit skills from parent — must list explicitly.

### Persistent Memory
```yaml
memory: user  # or project, local
```
- `user`: `~/.claude/agent-memory/<name>/` — across all projects
- `project`: `.claude/agent-memory/<name>/` — project-specific, version controllable
- `local`: `.claude/agent-memory-local/<name>/` — project-specific, not version controlled

When enabled:
- System prompt includes read/write instructions for memory directory
- First 200 lines of MEMORY.md injected into context
- Read, Write, Edit tools auto-enabled

### Permission Modes

| Mode                | Behavior                                     |
|:--------------------|:---------------------------------------------|
| `default`           | Standard permission checking with prompts    |
| `acceptEdits`       | Auto-accept file edits                       |
| `dontAsk`           | Auto-deny permission prompts                 |
| `delegate`          | Coordination-only mode for team leads        |
| `bypassPermissions` | Skip all permission checks                   |
| `plan`              | Read-only exploration                        |

### Built-in Subagent Types

| Type              | Model    | Tools        | Purpose                                    |
|:------------------|:---------|:-------------|:-------------------------------------------|
| **Explore**       | Haiku    | Read-only    | Fast codebase search and analysis           |
| **Plan**          | Inherit  | Read-only    | Research for planning                       |
| **general-purpose** | Inherit | All tools    | Complex multi-step tasks                    |
| **Bash**          | Inherit  | Bash         | Terminal commands in separate context        |

### Key Constraints
- Subagents CANNOT spawn other subagents (no nesting)
- Subagents receive only their system prompt + environment details, NOT full Claude Code system prompt
- Background subagents auto-deny unpre-approved permissions
- MCP tools NOT available in background subagents
- Resumed subagents retain full conversation history

### Task Tool Spawning Control
```yaml
# Allow spawning only specific subagents
tools: Task(worker, researcher), Read, Bash

# Allow spawning any subagent
tools: Task, Read, Bash

# No Task = cannot spawn subagents
tools: Read, Bash
```
Only applies to agents running as main thread with `claude --agent`. Subagents cannot spawn subagents regardless.

### Hooks in Subagents
```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

### Project-Level Hooks for Subagent Events
```json
{
  "hooks": {
    "SubagentStart": [{ "matcher": "db-agent", "hooks": [...] }],
    "SubagentStop": [{ "hooks": [...] }]
  }
}
```

---

## Design Implications for Agent Teams Skills

### For /sagerstack:planner (Agent Team)
1. Team lead creates team with TeamCreate, spawns 4 teammates
2. Each teammate is a separate Claude Code instance with own context
3. Teammates MUST be given full context in spawn prompt (no history inheritance)
4. Communication via SendMessage (DM between teammates, reports to lead)
5. Shared task list coordinates work sequentially
6. Lead manages Q&A with user (teammates can't interact with user directly in standard flow)
7. Delegate mode prevents lead from doing implementation work
8. Quality gates via TaskCompleted hooks

### For /sagerstack:builder (Agent Team)
1. Team lead spawns Developer and QA as teammates
2. Developer needs bypassPermissions or acceptEdits to write code
3. QA needs Bash access for test execution and UAT
4. Task dependencies: QA tasks blocked by Developer tasks
5. Remediation loop: lead creates new tasks when QA fails
6. Graceful shutdown after all stories complete

### For Custom Subagent Definitions
Each team member can be defined as a subagent in `.claude/agents/`:
- With specific `skills` preloaded (e.g., software-engineering for Developer)
- With `tools` restricted to what they need
- With `permissionMode` appropriate to their role
- With `memory` for cross-session learning (optional)
- With `hooks` for quality enforcement

### Critical Design Decision: Agent Teams vs Subagents
The planner and builder skills use **Agent Teams** (not subagents) because:
- Team members need to communicate with each other (e.g., BA shares stories with Solution Architect)
- Shared task list enables coordinated workflow progression
- Lead can manage user Q&A while teammates work
- Persistent context per teammate enables multi-step work within a session

However, within a team, some quick tasks could still use **subagents** (e.g., a teammate spawning an Explore subagent for quick codebase search). But teammates cannot spawn other teammates or create nested teams.
