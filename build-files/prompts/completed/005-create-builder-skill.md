<objective>
Create the /sagerstack:builder Claude Code skill (SKILL.md) that spawns a 2-member agent team to execute implementation plans produced by /sagerstack:planner. The Software Developer implements using TDD with existing sagerstack skills, and Code QA validates against acceptance criteria with flexible UAT.
</objective>

<context>
Read these files first:
- `./design/agent-teams-architecture.md` — Full workflow architecture (Section 2: Builder Team)
- `./research/agent-teams-synthesis.md` — Extracted specs, batch processing, retry patterns

This skill is invoked by the user via `/sagerstack:builder`. It reads implementation plans from `docs/phases/phase-N.M/plans/` and user stories from `docs/phases/phase-N.M/stories/` (produced by /sagerstack:planner).

The skill spawns a Claude Code agent team using TeamCreate with 2 members:
1. **Software Developer** — Implements tasks using /sagerstack:software-engineering + /sagerstack:local-testing
2. **Code QA** — Validates using /sagerstack:code-qa skill

Existing skills the Developer agent MUST use:
- `/sagerstack:software-engineering` — Vertical Slice + DDD, TDD, CamelCase, 90% coverage
- `/sagerstack:local-testing` — Docker-first, pytest fixtures, .env files

The Code QA agent uses:
- `/sagerstack:code-qa` — Acceptance criteria validation + flexible UAT (created by prompt 003)

User preferences:
- Configurable: one phase at a time (default) or auto-chain with QA gates (--continue)
- Max 2 QA retries before escalation to user
- Targeted remediation: fix specific failing tasks, not full re-implementation
- Git workflow: feature branch per story, commits after logical units
</context>

<requirements>

## Skill Structure

Create the skill at: `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/SKILL.md`

Follow the exact SKILL.md structure of existing sagerstack skills.

## Essential Principles

### 1. Implementation Plan-Driven Execution
- Read impl plan tasks in order
- Each task is atomic: has a clear deliverable and associated test
- Developer marks tasks complete in the impl plan as they go ([x])
- Never skip tasks or reorder without explicit reason

### 2. TDD is Non-Negotiable
Developer MUST follow red-green-refactor for every task:
1. Write failing test first (RED)
2. Write minimum code to pass (GREEN)
3. Refactor for quality (REFACTOR)
4. Commit

The Developer agent should load `/sagerstack:software-engineering` to enforce this.

### 3. Skill-Enforced Quality
Developer agent loads these skills before coding:
- `/sagerstack:software-engineering` → architecture patterns, TDD, naming, domain purity
- `/sagerstack:local-testing` → test fixtures, Docker setup, .env management

Code QA agent loads:
- `/sagerstack:code-qa` → validation workflow, UAT, report generation

### 4. QA Gate per User Story
After Developer completes all tasks for a user story:
- Code QA runs validation (AC check, tests, coverage, quality, UAT)
- If PASS → story marked complete, move to next
- If FAIL → remediation loop (max 2 retries)
- If still failing after 2 retries → escalate to user with QA report

### 5. Targeted Remediation
When QA fails:
- QA report maps failures to specific impl plan tasks
- Developer only re-works the specific failing tasks
- QA re-runs only the previously failing checks (not full suite)
- This prevents wasted effort on already-passing work

### 6. Project Memory Integration
Both builder team members integrate with `/project-memory`:

**Software Developer (READ before coding, WRITE when learning):**
- Read `docs/project_notes/decisions.md` — follow established architectural patterns
- Read `docs/project_notes/key_facts.md` — use correct config values, endpoints, ports
- Read `docs/project_notes/bugs.md` — when encountering errors, check for known solutions first
- Write bug entries when solving new bugs (date, issue, root cause, solution, prevention)
- Update `key_facts.md` with newly discovered config values

**Code QA (READ during validation, WRITE after validation):**
- Read `docs/project_notes/bugs.md` — check for regressions of known bugs
- Read `docs/project_notes/decisions.md` — verify code follows established patterns
- Write newly found bugs to `bugs.md`
- Log story completion to `docs/project_notes/issues.md`

Add `project-memory` to the `skills:` field of both the Developer and Code QA subagent definitions.

### 7. Configurable Execution Mode
- **Default (single phase)**: Execute one phase, stop after all stories validated
- **Auto-chain (--continue)**: After phase completes, check if next phase has impl plans ready. If yes, continue. Pause at each QA gate for user approval.

## Team Configuration

```
Team name: builder-phase-{N.M}
Members:
  - name: software-developer
    subagent_type: general-purpose
    role: TDD implementation of plan tasks
    mode: bypassPermissions
    tools: Read, Write, Edit, Bash, Grep, Glob
    skills_to_load:
      - /sagerstack:software-engineering
      - /sagerstack:local-testing

  - name: code-qa
    subagent_type: general-purpose
    role: Acceptance criteria validation and UAT
    tools: Read, Write, Edit, Bash, Grep, Glob
    skills_to_load:
      - /sagerstack:code-qa
```

## Workflow Sequence

### Step 1: Intake
- Read `docs/project-context.md` to identify the current phase
- Read `docs/phases/phase-N.M/` to find available impl plans
- List user stories with their impl plan status
- Ask user which phase/story to build (or auto-select next unbuilt)
- Check for --continue flag in args

### Step 2: Team Setup
- TeamCreate with team name based on phase
- Create task list from impl plan tasks

### Step 3: Pre-Implementation Setup
- Team Lead checks if local environment exists (Docker Compose, .env files)
- If not → assign Developer to set up using /sagerstack:local-testing patterns
- Verify: `docker-compose ps` shows healthy containers, `poetry install` succeeds

### Step 4: Story Execution Loop
For each user story in the phase:

#### 4a: Create Feature Branch
```bash
git checkout -b feature/phase-N.M-story-N
```

#### 4b: Task Execution
- Team Lead assigns impl plan tasks to Developer
- Developer loads /sagerstack:software-engineering
- For each task:
  1. Read task description from impl plan
  2. Write failing test (RED)
  3. Implement minimum code (GREEN)
  4. Refactor (REFACTOR)
  5. Commit with descriptive message
  6. Mark task [x] in impl plan
- Developer reports completion to Team Lead

#### 4c: QA Validation
- Team Lead assigns validation to Code QA
- Code QA loads /sagerstack:code-qa
- Code QA runs validate-story workflow:
  1. Parse AC from user story
  2. Run test suite, check coverage
  3. Run code quality checks
  4. Run UAT (if launch config available)
  5. Generate QA report at `docs/phases/phase-N.M/qa/story-N-qa-report.md`
- Code QA returns verdict to Team Lead

#### 4d: Remediation (if QA fails)
- Team Lead reads QA report, identifies failing tasks
- Team Lead assigns specific remediation tasks to Developer
- Developer fixes targeted issues
- Code QA re-validates (remediation-check workflow — only re-runs failed checks)
- Max 2 remediation cycles
- If still failing → escalate to user with report

#### 4e: Story Completion
- If QA passes:
  - Story status updated to complete
  - Merge feature branch (or create PR based on git workflow preference)
  - Move to next story

### Step 5: Phase Completion
- All stories in phase validated and complete
- Summary report generated
- Phase status updated in project-context.md (ready → complete)
- If --continue flag:
  - Check next phase for ready impl plans
  - If ready → start Step 1 for next phase
  - If not ready → inform user, suggest running /sagerstack:planner for next phase

### Step 6: Team Shutdown
- All tasks resolved
- Team Lead sends shutdown_request to all members
- TeamDelete to clean up

## Builder Artifacts

The builder produces:
- Modified source code (in vertical slice structure)
- Test files (unit, integration, e2e)
- Updated impl plan with [x] completion markers
- QA reports per story: `docs/phases/phase-N.M/qa/story-N-qa-report.md`
- Git commits (one per logical task unit)

## Error Handling

### Developer Agent Crash
- Task list preserves progress (completed tasks stay marked)
- Respawn Developer, resume from last incomplete task
- Impl plan [x] markers show what's already done

### QA Agent Crash
- Respawn QA, re-run validation from scratch (stateless operation)

### Test Suite Failure During Development
- Developer should fix the failing test before proceeding
- If stuck after 3 attempts on same test → escalate to user

### Environment Issues
- If Docker containers fail → Developer tries `docker-compose down && docker-compose up -d`
- If persistent → escalate to user

## Workflows

Create workflow files for:
- `workflows/execute-phase.md` — Main execution workflow (steps 1-6)
- `workflows/execute-story.md` — Single story execution (step 4)
- `workflows/remediation-loop.md` — QA failure remediation process
- `workflows/setup-environment.md` — Pre-implementation environment check

## References

Create reference files by extracting and adapting content from old artifacts:
- `references/git-workflow.md` — Branch naming, commit conventions, PR creation
- `references/task-execution.md` — How to read and execute impl plan tasks (extract from `./artifacts/process-implementation-plan.md`)
- `references/escalation-protocol.md` — When and how to escalate to user
- `references/developer-log.md` — Developer logging format (extract from `./artifacts/developer-log.md`)
- `references/ddd-patterns.md` — DDD implementation patterns (extract from `./artifacts/ddd-patterns.md`)

IMPORTANT: These reference files should contain the FULL content adapted for the new workflow, not just placeholders. The artifacts folder will be deleted after skills are created — the skills must be self-contained.

</requirements>

<implementation>
Read existing sagerstack skills for structural reference:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-planning/SKILL.md`
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-software-engineering/SKILL.md`
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-local-testing/SKILL.md`

Also read the code-qa skill created by prompt 003:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/SKILL.md`

The SKILL.md must be complete and self-contained. When loaded by Claude, it should:
1. Set up the agent team
2. Read impl plans and user stories
3. Orchestrate Developer → QA → Remediation loop
4. Handle configurable execution (single phase vs --continue)
5. Manage git workflow (feature branches, commits)
6. Generate completion reports

Workflow files and reference files should be created as placeholders with clear TODO descriptions.
</implementation>

<output>
Create these files:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/SKILL.md` — Main skill file (complete)
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/workflows/execute-phase.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/workflows/execute-story.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/workflows/remediation-loop.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/workflows/setup-environment.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/references/git-workflow.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/references/task-execution.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-builder/references/escalation-protocol.md` — Placeholder
</output>

<verification>
Before completing, verify:
- [ ] SKILL.md matches structure of existing sagerstack skills
- [ ] Team configuration specifies both members with subagent_type and skills_to_load
- [ ] Developer agent loads /sagerstack:software-engineering and /sagerstack:local-testing
- [ ] Code QA agent loads /sagerstack:code-qa
- [ ] TDD workflow enforced (red-green-refactor per task)
- [ ] QA gate runs after all tasks in a story complete
- [ ] Remediation loop is targeted (specific failing tasks, not full re-impl)
- [ ] Max 2 retries before escalation
- [ ] Configurable execution mode (single phase default, --continue for auto-chain)
- [ ] Git workflow: feature branch per story, commits per logical task unit
- [ ] Error handling covers agent crash, test failure, environment issues
- [ ] Input reads from docs/phases/phase-N.M/ (planner output)
- [ ] Workflow and reference placeholder files created
</verification>
