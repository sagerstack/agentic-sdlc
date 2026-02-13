# Agent Teams Architecture Design

Date: 2026-02-13
Status: Draft
Author: Design Agent

---

## Table of Contents

1. [Pipeline Overview](#1-pipeline-overview)
2. [Planner Team Architecture](#2-planner-team-architecture)
3. [Builder Team Architecture](#3-builder-team-architecture)
4. [Code QA Skill Design](#4-code-qa-skill-design)
5. [Communication Protocol](#5-communication-protocol)
6. [Error Handling](#6-error-handling)
7. [Configuration](#7-configuration)
8. [Custom Subagent Definitions](#8-custom-subagent-definitions)

---

## 1. Pipeline Overview

### End-to-End Flow

```
User Request ("let's build X")
    |
    v
/sagerstack:code-planning (existing skill, user-driven, interactive)
    |
    v
docs/code_context.md
    (Contains: Milestones with Phases, E2E Tests, Architecture Decisions)
    |
    v
/sagerstack:planner (Agent Team 1)
    Processes ONE phase at a time from code_context.md
    |
    v
docs/phases/phase-N.M/
    epic.md
    stories/story-N.md
    plans/story-N-plan.md
    research/findings.md
    |
    v
/sagerstack:builder (Agent Team 2)
    Executes implementation plans for ONE phase
    |
    v
Implemented + validated code on feature branch
    (Tests passing, coverage met, QA approved)
```

### Pipeline Boundaries

| Boundary | Input | Output | Trigger |
|----------|-------|--------|---------|
| User -> code-planning | Feature request (natural language) | `docs/code_context.md` | User invokes `/sagerstack:code-planning` |
| code_context.md -> planner | Phase reference from `code_context.md` | Phase artifacts in `docs/phases/phase-N.M/` | User invokes `/sagerstack:planner` |
| Phase artifacts -> builder | Impl plans from `docs/phases/phase-N.M/plans/` | Code on feature branch | User invokes `/sagerstack:builder` |
| Builder -> next phase | Completed phase code | Next phase input (or prompt user) | `--continue` flag or user invocation |

### Skill Invocation Chain

Two coexisting paths. The CLAUDE.md routing table selects the path based on user intent.

**Path A -- Full SDLC (agent teams):**
```
code-planning -> planner -> builder [-> builder -> ...] -> deploy-aws
```

**Path B -- Direct Implementation (existing chain, unchanged):**
```
code-planning -> software-engineering -> local-testing -> deploy-aws
```

### Phase Addressing Scheme

Phases are addressed using the milestone-phase notation from `code_context.md`:

- `phase-1.1` = Milestone 1, Phase 1
- `phase-1.2` = Milestone 1, Phase 2
- `phase-2.1` = Milestone 2, Phase 1

This notation carries through to the artifact directory structure:
```
docs/phases/
  phase-1.1/
    epic.md
    stories/
    plans/
    research/
  phase-1.2/
    ...
```

---

## 2. Planner Team Architecture

### 2.1 Team Composition

| Role | Agent Type | Permission Mode | Model | Purpose |
|------|-----------|----------------|-------|---------|
| Team Lead | `planner-lead` (custom) | `delegate` | `opus` | Orchestrates workflow, manages user Q&A, coordinates teammates |
| Researcher | `planner-researcher` (custom) | `plan` | `sonnet` | Investigates phase requirements, researches APIs/services, produces findings |
| Business Analyst | `planner-ba` (custom) | `acceptEdits` | `opus` | Creates epics + user stories with FR/TR/AC, manages phase lifecycle |
| Solution Architect | `planner-architect` (custom) | `acceptEdits` | `opus` | Generates impl plans, flags costs, researches technical approaches |
| Critical Analyst | `planner-critic` (custom) | `plan` | `opus` | Reviews impl plans for alignment with user stories + best practices |

**Model Rationale:**
- Opus for Lead, BA, Architect, Critic: These roles require deep reasoning about requirements, architecture, and quality assessment.
- Sonnet for Researcher: Research tasks are primarily information gathering and synthesis. Sonnet provides sufficient capability at lower cost.

### 2.2 Workflow (16 Steps)

```
Step 1: Read Phase
    |
Step 2: Assign research ──> Step 3: Researcher investigates
    |
Step 4: Assign story creation ──> Step 5: BA generates proposals
    |
Step 6: User confirms direction (Q&A Point 1)
    |
Step 7: BA generates epic + user stories
    |
Step 8: User confirms story breakdown (Q&A Point 2)
    |
Step 9: BA generates stories (confirmed)
    |
Step 10: Assign impl plan generation
    |
Step 11: Solution Architect Q&A with user (Q&A Point 3)
    |
Step 12: Solution Architect researches + generates impl plans
    |
Step 13: Assign review to Critical Analyst
    |
Step 14: Critical Analyst reviews impl plans
    |
    +--- Issues found? ──> Step 15: Solution Architect revises (max 2 cycles, back to Step 12)
    |
Step 16: Phase planning complete, artifacts saved
```

#### Step-by-Step Detail

**Step 1: Team Lead reads phase from code_context.md**

Team Lead reads `docs/code_context.md`, identifies the target phase (specified by user or next unplanned phase), and extracts:
- Phase name and description
- Definition of Done criteria
- Success Criteria
- Milestone context (what this phase contributes to)
- Any relevant E2E test definitions

Team Lead creates the team and spawns all 4 teammates with full phase context in their spawn prompts.

```
Action: TeamCreate(team_name="planner-phase-N-M", description="Planning phase N.M: {phase name}")
Action: Spawn 4 teammates via Task tool with subagent_type matching custom agent definitions
```

**Step 2: Team Lead assigns research task to Researcher**

Team Lead creates a task in the shared task list and sends a message to the Researcher with:
- Phase context (full phase description from code_context.md)
- What to investigate (technologies, APIs, services, patterns mentioned in the phase)
- Specific research questions derived from the phase scope

```
Action: TaskCreate(subject="Research phase N.M requirements", description="...")
Action: TaskUpdate(taskId="1", owner="researcher")
Action: SendMessage(type="message", recipient="researcher", content="...", summary="Research phase N.M")
```

**Step 3: Researcher investigates and returns findings**

Researcher uses read-only tools (Glob, Grep, Read, WebSearch, WebFetch) to:
- Search the existing codebase for relevant patterns and existing implementations
- Research external APIs, libraries, and services mentioned in the phase
- Identify technical constraints and dependencies
- Document findings following the tech research template structure

Researcher saves findings to `docs/phases/phase-N.M/research/findings.md` and messages Team Lead.

```
Action: Write findings to docs/phases/phase-N.M/research/findings.md
Action: SendMessage(type="message", recipient="planner-lead", content="Research complete. Key findings: ...", summary="Research findings ready")
Action: TaskUpdate(taskId="1", status="completed")
```

**Step 4: Team Lead assigns story creation to BA (with research findings)**

Team Lead sends a message to BA containing:
- Full phase context from code_context.md
- Research findings from Step 3 (file path reference)
- Instruction to generate initial proposals (NOT full stories yet)

```
Action: TaskCreate(subject="Generate epic and story proposals for phase N.M", description="...")
Action: TaskUpdate(taskId="2", owner="business-analyst")
Action: SendMessage(type="message", recipient="business-analyst", content="...", summary="Create story proposals")
```

**Step 5: BA generates proposals and Team Lead presents to user (Q&A Point 1)**

BA drafts a high-level proposal containing:
- Proposed epic theme and scope
- Proposed user story titles with brief descriptions
- Estimated complexity per story (using AI Complexity Scoring Framework)
- Cost implications (any paid services, subscriptions, or infrastructure costs flagged)

BA sends proposals to Team Lead. Team Lead presents proposals to user and waits for confirmation.

```
BA Action: SendMessage(type="message", recipient="planner-lead", content="Proposals: ...", summary="Story proposals ready")
Lead Action: Present proposals to user, ask "Does this direction look right? Any adjustments?"
```

**Step 6: User confirms direction**

User reviews proposals and either:
- Confirms direction as-is
- Requests adjustments (scope changes, story additions/removals, priority changes)

If adjustments requested, Team Lead sends feedback to BA, who revises proposals. This mini-loop continues until user confirms.

**Step 7: BA generates epic + user stories**

With confirmed direction, BA generates:
- Epic document following `epic-artifact.md` template
- User story documents following `user-story-artifact.md` template
- Each story includes: FR/TR/AC tables, dependencies, risk assessment, Requirements Clarifications section

BA saves artifacts:
- `docs/phases/phase-N.M/epic.md`
- `docs/phases/phase-N.M/stories/story-1.md`
- `docs/phases/phase-N.M/stories/story-2.md`
- ...

```
Action: Write epic and stories to docs/phases/phase-N.M/
Action: SendMessage(type="message", recipient="planner-lead", content="Epic and stories generated. N stories created.", summary="Stories generated")
Action: TaskUpdate(taskId="2", status="completed")
```

**Step 8: Team Lead presents story breakdown to user for confirmation (Q&A Point 2)**

Team Lead presents to user:
- Story count and titles
- Story dependency graph
- FR/TR/AC summary per story
- Any open Requirements Clarifications ([INSERT_USER_INPUT] placeholders)

User reviews and either confirms or requests changes. If changes needed, Team Lead sends feedback to BA for revision.

**Step 9: User confirms stories**

User confirms the story breakdown. BA updates story status from Draft to Final after incorporating any user feedback. Any Requirements Clarifications placeholders are resolved.

**Step 10: Team Lead assigns impl plan generation to Solution Architect**

Team Lead creates tasks for each user story's implementation plan and assigns to Solution Architect:
- User story file paths
- Research findings file path
- Technical Guidance section from user stories (if populated)
- Instruction to flag any costs and prefer zero/low-cost approaches

```
Action: TaskCreate(subject="Generate impl plan for story N", description="...") for each story
Action: TaskUpdate(taskId="N", owner="solution-architect")
Action: SendMessage(type="message", recipient="solution-architect", content="...", summary="Generate impl plans")
```

**Step 11: Solution Architect Q&A with user via Team Lead (Q&A Point 3)**

Before generating plans, Solution Architect may need technical direction on:
- Technology choices (e.g., which database, which API client library)
- Cost trade-offs (e.g., "Option A is free but limited, Option B costs $X/month")
- Architecture decisions not covered in code_context.md
- Infrastructure preferences

Solution Architect sends questions to Team Lead, who presents them to user. User responses flow back through Team Lead.

```
Architect Action: SendMessage(type="message", recipient="planner-lead", content="Technical questions: ...", summary="Need technical direction")
Lead Action: Present questions to user, wait for answers
Lead Action: SendMessage(type="message", recipient="solution-architect", content="User answers: ...", summary="Technical direction provided")
```

**Cost Flagging Protocol:**
Solution Architect MUST flag any non-zero cost implications using this format in messages to Team Lead:

```
COST FLAG:
- Item: {service/tool/subscription name}
- Monthly cost: ${amount}
- Alternative: {zero-cost or lower-cost alternative, if any}
- Trade-off: {what is lost by choosing the cheaper option}
- Recommendation: {architect's recommendation}
```

Team Lead presents cost flags to user for explicit approval before proceeding.

**Step 12: Solution Architect researches and generates impl plans per user story**

For each user story, Solution Architect:
1. Reads the user story FR/TR/AC
2. Reads research findings
3. Generates tech research document (if external APIs/services involved)
4. Generates implementation plan following `implementation-plan.md` template
5. Ensures 100% FR/TR/AC mapping in Requirements Coverage Validation section
6. Includes API Field Extraction tasks where applicable (per tech research)
7. Flags all costs in the plan

Artifacts saved:
- `docs/phases/phase-N.M/plans/story-N-plan.md`
- `docs/phases/phase-N.M/research/story-N-tech-research.md` (if applicable)

```
Action: Write impl plans and tech research
Action: SendMessage(type="message", recipient="planner-lead", content="Impl plans complete for N stories.", summary="Impl plans ready")
Action: TaskUpdate for each plan task -> completed
```

**Step 13: Team Lead assigns review to Critical Analyst**

Team Lead creates review tasks and assigns to Critical Analyst:
- Impl plan file paths for all stories in this phase
- User story file paths (for FR/TR/AC cross-reference)
- Instruction: Review impl plans (NOT user stories) for alignment with user stories AND industry best practices

```
Action: TaskCreate(subject="Review impl plans for phase N.M", description="...")
Action: TaskUpdate(taskId="N", owner="critical-analyst")
Action: SendMessage(type="message", recipient="critical-analyst", content="...", summary="Review impl plans")
```

**Step 14: Critical Analyst reviews impl plans**

Critical Analyst reviews EACH implementation plan against:
1. **Alignment with User Story FR/TR/AC:**
   - Every FR has a corresponding parent task in the impl plan
   - Every TR has a corresponding parent task
   - Every AC has 4 test levels (unit, integration, E2E, live verification)
   - Requirements Coverage Validation shows 100% mapping
2. **Industry Best Practices:**
   - Architecture patterns appropriate for the problem domain
   - Security considerations addressed
   - Performance requirements achievable
   - Error handling comprehensive
   - No anti-patterns (hardcoded values, missing validation, etc.)
3. **Cost Compliance:**
   - Budget constraints from Technical Guidance respected
   - No unexpected cost escalation
4. **Cross-Story Integration:**
   - Dependencies between stories properly handled
   - No contract mismatches between story outputs/inputs

Critical Analyst generates a critical analysis document per `critical-analysis.md` template:
- `docs/phases/phase-N.M/plans/story-N-critical-analysis.md`

If no issues: Uses "No Issues" minimal template, sets Confidence Level = High.
If issues found: Uses full template with Critical Issues, High-Risk Assumptions, Recommendations.

```
Action: Write critical analysis documents
Action: SendMessage(type="message", recipient="planner-lead", content="Review complete. {N} issues found across {M} plans.", summary="Critical review complete")
Action: TaskUpdate -> completed
```

**Step 15: Refinement loop (if issues found, max 2 cycles)**

If Critical Analyst found issues:

Cycle 1:
1. Team Lead sends Critical Analyst's findings to Solution Architect
2. Solution Architect reads critical analysis, revises impl plans
3. Team Lead sends revised plans back to Critical Analyst for re-review
4. Critical Analyst updates critical analysis (Analysis Iteration = 2nd Analysis)

Cycle 2 (if still issues after Cycle 1):
1. Same flow as Cycle 1
2. After Cycle 2, proceed regardless with documented risks
3. Critical Analyst updates Refinement Tracking section with remaining concerns

```
Lead Action: SendMessage(type="message", recipient="solution-architect", content="Critical issues found: {summary}. Please revise.", summary="Revise impl plans")
Architect Action: Revise plans, SendMessage -> Lead
Lead Action: SendMessage(type="message", recipient="critical-analyst", content="Revised plans ready for re-review.", summary="Re-review revised plans")
Critic Action: Re-review, update critical analysis, SendMessage -> Lead
```

**Step 16: Phase planning complete, artifacts saved**

Team Lead:
1. Verifies all artifacts exist and are complete
2. Logs completion summary
3. Shuts down all teammates via `shutdown_request`
4. Deletes team via `TeamDelete`

Final artifact tree:
```
docs/phases/phase-N.M/
  epic.md
  stories/
    story-1.md
    story-2.md
    ...
  plans/
    story-1-plan.md
    story-1-critical-analysis.md
    story-2-plan.md
    story-2-critical-analysis.md
    ...
  research/
    findings.md
    story-1-tech-research.md (if applicable)
    story-2-tech-research.md (if applicable)
```

### 2.3 User Q&A Points Summary

| Q&A Point | Step | Purpose | Who Answers | Blocking? |
|-----------|------|---------|-------------|-----------|
| Q&A 1 | Step 5-6 | Confirm epic/story direction | User | Yes -- BA cannot proceed without direction confirmation |
| Q&A 2 | Step 8-9 | Confirm story breakdown (FR/TR/AC, dependencies) | User | Yes -- Architect cannot plan without confirmed stories |
| Q&A 3 | Step 11 | Technical direction + cost approvals | User | Yes -- Architect cannot generate plans without tech decisions |

### 2.4 Phase Management

The BA supports phase lifecycle operations on `docs/code_context.md`:

| Operation | Description | Trigger |
|-----------|-------------|---------|
| Insert phase | Add a new phase between existing phases, renumber subsequent | User requests during planning |
| Remove phase | Remove a phase, renumber subsequent | User decides phase is unnecessary |
| Reorder phases | Change phase execution order within a milestone | User changes priority |
| Split phase | Divide one phase into two smaller phases | Phase is too large |
| Merge phases | Combine two phases into one | Phases are too small |

Phase management is performed by the BA at the direction of the user, communicated through the Team Lead. Changes are written directly to `docs/code_context.md` and propagate to subsequent planner invocations.

---

## 3. Builder Team Architecture

### 3.1 Team Composition

| Role | Agent Type | Permission Mode | Model | Purpose |
|------|-----------|----------------|-------|---------|
| Team Lead | `builder-lead` (custom) | `delegate` | `opus` | Orchestrates build workflow, manages task assignment, handles QA results |
| Software Developer | `builder-developer` (custom) | `bypassPermissions` | `sonnet` | Implements code via TDD, follows software-engineering + local-testing skills |
| Code QA | `builder-qa` (custom) | `plan` | `opus` | Validates AC pass/fail, runs quality checks, flexible UAT |

**Model Rationale:**
- Opus for Lead and QA: Coordination requires deep reasoning. QA needs thorough analytical capability for accurate validation.
- Sonnet for Developer: Code implementation is well-structured by impl plans. Sonnet handles TDD cycles effectively at lower cost.

**Permission Mode Rationale:**
- `delegate` for Lead: Prevents lead from writing code; forces pure coordination.
- `bypassPermissions` for Developer: Must write/edit files, run tests, execute shell commands without prompts.
- `plan` for QA: Read-only access. QA must NEVER modify code. Validates by reading source, running tests via Bash, inspecting outputs.

### 3.2 Workflow

```
Step 1: Read impl plans
    |
Step 2: Create task list
    |
Step 3: Assign tasks to Developer
    |
Step 4: Developer implements (TDD: red-green-refactor)
    |
Step 5: Developer commits after each logical unit
    |
Step 6: After all tasks -> Assign QA validation
    |
Step 7: QA validates (AC check, tests, coverage, UAT)
    |
    +--- QA fails? -> Step 8: Targeted remediation (max 2 retries, back to Step 6)
    |
Step 9: Story complete, move to next story
    |
Step 10: After all stories -> Phase complete
    |
Step 11: If --continue -> Next phase (or prompt user to run planner)
```

#### Step-by-Step Detail

**Step 1: Team Lead reads impl plans for current phase**

Team Lead reads all implementation plans from `docs/phases/phase-N.M/plans/`:
- Identifies all stories with their impl plans
- Reads critical analysis documents for any documented risks
- Determines story execution order based on dependencies
- Reads user stories for FR/TR/AC reference

```
Action: Read docs/phases/phase-N.M/plans/story-*-plan.md
Action: Read docs/phases/phase-N.M/stories/story-*.md
Action: Identify story dependency order
```

**Step 2: Team Lead creates task list from impl plans**

For each story (in dependency order), Team Lead:
1. Creates the team: `TeamCreate(team_name="builder-phase-N-M", description="Building phase N.M")`
2. Spawns Developer and QA teammates
3. Extracts parent tasks from the impl plan's Task-Based Implementation Plan section
4. Creates a task per parent task in the shared TaskList
5. Sets up task dependencies (QA tasks blocked by all developer tasks for that story)

Task creation follows impl plan structure:
- `[MANUAL]` tasks: Created as blocked, Team Lead notifies user
- `[SETUP]` tasks: Assigned to Developer first
- `[FR-N]`, `[TR-N]`, `[AC-N]` tasks: Assigned to Developer in impl plan order
- QA validation task: Created as blocked by all story impl tasks

```
Action: TeamCreate(team_name="builder-phase-N-M")
Action: Spawn developer and qa teammates
Action: For each parent task in impl plan:
    TaskCreate(subject="[Story N] {parent task description}", description="{full subtask list}")
Action: TaskCreate(subject="[Story N] QA Validation", description="Validate all AC for story N")
Action: TaskUpdate(qaTaskId, addBlockedBy=[all story N impl task IDs])
```

**Step 3: Team Lead assigns tasks to Developer**

Team Lead assigns implementation tasks to Developer sequentially per story:
- Sends message with full context: impl plan reference, user story reference, tech research reference
- Instructs Developer to follow TDD: write failing test first, then minimal code to pass, then refactor
- Developer must follow `/sagerstack:software-engineering` standards (preloaded via skills field)
- Developer must follow `/sagerstack:local-testing` standards (preloaded via skills field)

```
Action: TaskUpdate(taskId, owner="developer")
Action: SendMessage(type="message", recipient="developer", content="Implement {task}. Follow impl plan: {path}. TDD required.", summary="Implement task {N}")
```

**Step 4: Developer implements via TDD**

For each assigned task, Developer:
1. Reads the impl plan subtasks
2. For each subtask:
   a. Writes a failing test (Red)
   b. Writes minimal code to make test pass (Green)
   c. Refactors for quality (Refactor)
3. Runs full test suite to confirm no regressions
4. Marks task as completed in TaskList

Developer follows these standards (preloaded via skills):
- Vertical Slice + DDD structure
- CamelCase naming everywhere
- Strict domain purity (no infrastructure imports in domain)
- Custom exceptions + Result pattern
- Structured logging
- No hardcoded values (all config from .env files)

```
Developer Action: Implement subtasks following TDD
Developer Action: TaskUpdate(taskId, status="completed")
Developer Action: SendMessage(type="message", recipient="builder-lead", content="Task complete. Tests passing.", summary="Task {N} complete")
```

**Step 5: Developer commits after each logical unit**

After completing each parent task (all subtasks done):
1. Developer runs the quality check pipeline (Checks 1-5 from quality-checks.md):
   - Check 1: Full test suite (`poetry run pytest tests/ -v`)
   - Check 2: Coverage report (`poetry run pytest --cov --cov-fail-under=90`)
   - Check 3: Static type checking (`poetry run mypy src/`)
   - Check 4: Linting (`poetry run ruff check src/ tests/`)
   - Check 5: Security scan (`poetry run bandit -r src/`)
2. If all checks pass, commits to feature branch with descriptive message
3. Continues to next task

```
Developer Action: Run quality checks
Developer Action: git add {relevant files} && git commit -m "{descriptive message}"
```

**Step 6: After all impl tasks for a story -> Team Lead assigns QA validation**

When all implementation tasks for a story are completed:
1. The QA validation task auto-unblocks (dependency chain resolved)
2. Team Lead assigns the QA task to Code QA
3. Team Lead sends QA the full context: story path, impl plan path, what to validate

```
Action: TaskUpdate(qaTaskId, owner="code-qa")
Action: SendMessage(type="message", recipient="code-qa", content="Validate story N. Story: {path}. Plan: {path}.", summary="QA validate story N")
```

**Step 7: QA validates**

Code QA performs validation (see Section 4 for full QA skill design):
1. AC-driven validation: Parse AC from user story, test each one, report pass/fail
2. Quality pipeline: Run all 9 quality checks
3. Coverage verification: Confirm >= 90% coverage
4. UAT: If docker-compose exists, spin up and test via HTTP
5. Code quality: Verify CamelCase, domain purity, no hardcoded values

QA generates a QA Report and sends to Team Lead.

```
QA Action: Run validation suite
QA Action: SendMessage(type="message", recipient="builder-lead", content="QA Report: ...", summary="QA results for story N")
QA Action: TaskUpdate(qaTaskId, status="completed") -- only if all pass
```

**Step 8: Remediation loop (if QA fails, max 2 retries)**

If QA finds failures:

1. QA sends granular failure report to Team Lead (see Section 4 for report format)
2. Team Lead creates targeted remediation tasks from QA's failure-to-task mapping
3. Team Lead assigns remediation tasks to Developer
4. Developer fixes issues, commits
5. Team Lead re-assigns QA validation
6. If still failing after 2 retries, Team Lead escalates to user with:
   - What failed
   - What was attempted
   - Remaining issues
   - Recommended manual intervention

```
Lead Action: TaskCreate(subject="[Story N] Remediation: {failure description}", description="{specific fix needed}")
Lead Action: TaskUpdate(remediationTaskId, owner="developer")
Lead Action: SendMessage(type="message", recipient="developer", content="Fix needed: {QA findings}", summary="Fix QA failures")
-- After developer fixes:
Lead Action: SendMessage(type="message", recipient="code-qa", content="Re-validate story N after fixes.", summary="Re-validate story N")
```

**Step 9: Story complete, move to next story**

When QA passes for a story:
1. Team Lead marks the story as complete
2. Developer marks all `[x]` in the impl plan for completed tasks
3. Move to next story in dependency order
4. Repeat Steps 3-8 for each story

**Step 10: After all stories -> Phase complete**

When all stories in the phase are implemented and QA-approved:
1. Team Lead runs a final full test suite across the entire codebase
2. Team Lead verifies all impl plan tasks are marked `[x]`
3. Team Lead shuts down teammates and deletes team
4. Phase is marked as complete

```
Action: SendMessage(type="shutdown_request", recipient="developer")
Action: SendMessage(type="shutdown_request", recipient="code-qa")
-- After shutdown confirmations:
Action: TeamDelete
```

**Step 11: Phase continuation**

Two modes based on configuration:

**Default (single phase):** Builder completes and returns to user. User can invoke builder again for the next phase, or invoke planner first if the next phase is not yet planned.

**With `--continue` flag:** After phase completes, builder checks if the next phase's impl plans exist:
- If next phase plans exist: Automatically starts building the next phase (creates new team)
- If next phase plans do NOT exist: Prompts user to run planner for the next phase first, then pauses

### 3.3 Git Workflow

All builder work follows the git constraints:
- Never push directly to main
- Always work on feature branches
- Branch naming: `feature/phase-N.M-story-{story-number}`
- Commits: After each parent task completion (with quality checks passing)
- PR: Created after all stories in a phase pass QA (or per-story if preferred by user)
- Merge latest from main before pushing feature branch

---

## 4. Code QA Skill Design

### 4.1 Overview

The Code QA skill (`/sagerstack:code-qa`) defines the validation methodology for the `builder-qa` agent. It operates as a zero-trust validator: it re-runs all tests independently, never trusts developer assertions, and reports granular pass/fail results.

### 4.2 AC-Driven Validation

For each user story, QA parses the Acceptance Criteria table and validates each AC independently.

**Parsing Process:**
1. Read user story file at `docs/phases/phase-N.M/stories/story-N.md`
2. Extract the AC table (Given/When/Then/Type/Validates/Priority)
3. For each AC, determine validation method based on Type:

| AC Type | Validation Method |
|---------|-------------------|
| Functional - Happy Path | Run corresponding test, verify expected outcome |
| Functional - Failure Scenario | Run corresponding test, verify error handling |
| Functional - Edge Case | Run corresponding test, verify edge behavior |
| Functional - Error Handling | Run corresponding test, verify error response |
| Functional - Integration | Run integration test with real or mocked services |
| Functional - End-to-End | Run E2E test via docker-compose + curl |
| Technical - Performance | Run performance test, verify threshold |
| Technical - Security | Run security scan, verify no vulnerabilities |
| Technical - Reliability | Run reliability test, verify SLA |

**AC Report Format (per AC):**

```markdown
### AC-{N}: {AC Description}
- **Status**: PASS / FAIL
- **Validates**: {FR/TR IDs}
- **Evidence**:
  - Test: {test file}:{test function}
  - Result: {pass/fail with output}
  - Notes: {any observations}
```

### 4.3 Quality Pipeline (9 Checks)

QA runs the adapted quality checks sequentially:

| Check | Command | Threshold | Failure Action |
|-------|---------|-----------|----------------|
| 1. Full Test Suite | `poetry run pytest tests/ -v` | All pass | STOP, report failures |
| 2. Coverage | `poetry run pytest --cov=src --cov-fail-under=90` | >= 90% | STOP, report uncovered lines |
| 3. Type Checking | `poetry run mypy src/ --strict` | Zero errors | STOP, report type errors |
| 4. Linting | `poetry run ruff check src/ tests/` | Zero violations | STOP, report lint issues |
| 5. Formatting | `poetry run ruff format --check src/ tests/` | All formatted | STOP, report unformatted files |
| 6. Security | `poetry run bandit -r src/` | No high/critical | STOP, report vulnerabilities |
| 7. Docker Build | `docker-compose build` | Builds successfully | STOP, report build errors |
| 8. CHANGELOG | Verify entry exists | Entry present | STOP, report missing entry |
| 9. Git Status | Verify clean working tree | No uncommitted changes | STOP, report dirty files |

### 4.4 Flexible UAT

QA performs User Acceptance Testing by detecting the application's execution model and testing accordingly.

**Detection Logic:**
1. Check for `docker-compose.yml` or `docker-compose.yaml` in project root
   - If found: UAT via Docker
2. Check for application entry point (e.g., `main.py`, FastAPI app)
   - If found: UAT via local process
3. If neither found: Skip UAT, note in report

**UAT via Docker:**
```bash
# 1. Build and start
docker-compose build
docker-compose up -d

# 2. Wait for health check
for i in {1..30}; do
  curl -sf http://localhost:{port}/health && break
  sleep 1
done

# 3. Run E2E test scenarios from impl plan
# For each AC of type "Functional - End-to-End":
response=$(curl -s http://localhost:{port}/{endpoint})
status=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:{port}/{endpoint})

# 4. Assert expected outcomes
test "$status" = "200" || echo "FAIL: Expected 200, got $status"

# 5. Tear down
docker-compose down
```

**UAT via Local Process:**
```bash
# 1. Start application
poetry run python -m src.main &
APP_PID=$!

# 2. Wait for startup
sleep 3

# 3. Run test scenarios
# Same curl-based assertions as Docker UAT

# 4. Stop application
kill $APP_PID
```

### 4.5 Granular Failure Mapping

When QA finds failures, it maps each failure back to specific impl plan tasks. This enables targeted remediation instead of broad rework.

**Failure Mapping Process:**
1. For each test failure, identify the source file and function
2. Map the source file to the impl plan task that created/modified it
3. Map the test to the AC it validates
4. Produce a remediation task list

**Failure Report Format:**

```markdown
# QA Report: Story {N} - {Story Title}

## Summary
- **Overall Status**: FAIL
- **AC Results**: {X}/{Y} passed
- **Coverage**: {N}%
- **Quality Checks**: {X}/{9} passed
- **UAT**: PASS / FAIL / SKIPPED

## AC Results
| AC ID | Description | Status | Evidence |
|-------|-------------|--------|----------|
| AC-1 | {desc} | PASS | test_file:test_func |
| AC-2 | {desc} | FAIL | test_file:test_func - AssertionError: ... |
| AC-3 | {desc} | PASS | test_file:test_func |

## Quality Check Results
| Check | Status | Details |
|-------|--------|---------|
| Test Suite | PASS | 45/45 tests pass |
| Coverage | FAIL | 87% (target: 90%). Uncovered: src/orders/infrastructure/repo.py:45-62 |
| Type Check | PASS | 0 errors |
| ... | ... | ... |

## UAT Results
| Scenario | Status | Details |
|----------|--------|---------|
| Health check | PASS | 200 OK |
| Create order | FAIL | Expected 201, got 500. Response: {"error": "..."} |

## Failure-to-Task Mapping
| Failure | Source | Impl Plan Task | Remediation |
|---------|--------|----------------|-------------|
| AC-2 test failure | src/orders/domain/order.py:23 | [5.0][FR-1] subtask [5.2] | Fix validation logic for empty cart |
| Coverage gap | src/orders/infrastructure/repo.py:45-62 | [7.0][FR-3] subtask [7.3] | Add tests for repository error paths |
| UAT: Create order 500 | src/orders/api/routes.py:15 | [8.0][AC-2] subtask [8.1] | Fix endpoint handler exception |

## Remediation Tasks (for Team Lead)
1. Fix validation logic in order domain - relates to [5.0][FR-1]
2. Add repository error path tests - relates to [7.0][FR-3]
3. Fix endpoint exception handling - relates to [8.0][AC-2]
```

### 4.6 QA Pass Report Format

When all validations pass:

```markdown
# QA Report: Story {N} - {Story Title}

## Summary
- **Overall Status**: PASS
- **AC Results**: {Y}/{Y} passed (100%)
- **Coverage**: {N}% (>= 90%)
- **Quality Checks**: 9/9 passed
- **UAT**: PASS

## AC Results
| AC ID | Description | Status | Evidence |
|-------|-------------|--------|----------|
| AC-1 | {desc} | PASS | test_file:test_func |
| AC-2 | {desc} | PASS | test_file:test_func |
| ... | ... | ... | ... |

## Quality Check Results
All 9 checks passed. See details below.
| Check | Status |
|-------|--------|
| Test Suite | PASS (45/45) |
| Coverage | PASS (95%) |
| Type Check | PASS (0 errors) |
| Linting | PASS (0 violations) |
| Formatting | PASS |
| Security | PASS (0 issues) |
| Docker Build | PASS |
| CHANGELOG | PASS |
| Git Status | PASS |

## UAT Results
All scenarios passed.

## Recommendation
Story is ready for merge. All acceptance criteria validated, quality standards met.
```

---

## 5. Communication Protocol

### 5.1 Channel Matrix

| From | To | Channel | Purpose |
|------|-----|---------|---------|
| Team Lead | User | Direct output (stdout) | Q&A, proposals, status updates, escalations |
| User | Team Lead | User input (stdin) | Confirmations, answers, direction |
| Team Lead | Teammate | `SendMessage` type: `message` | Task assignment, context delivery, feedback |
| Teammate | Team Lead | `SendMessage` type: `message` | Completion reports, questions, findings |
| Team Lead | All Teammates | `SendMessage` type: `broadcast` | Critical announcements only (use sparingly) |
| Team Lead | Teammate | `SendMessage` type: `shutdown_request` | Graceful shutdown |
| Teammate | Team Lead | `SendMessage` type: `shutdown_response` | Shutdown acknowledgment |
| Planner -> Builder | Disk artifacts | Phase artifact files in `docs/phases/` | Cross-team handoff |

### 5.2 Message Format Standards

**Task Assignment Message (Lead -> Teammate):**
```
TASK ASSIGNMENT:
- Task ID: {taskId from TaskList}
- Subject: {task subject}
- Context:
  - Phase: {phase reference}
  - Story: {story file path} (if applicable)
  - Impl Plan: {plan file path} (if applicable)
  - Research: {research file path} (if applicable)
- Instructions: {specific action required}
- Constraints: {any special constraints}
- Expected Output: {what to produce}
```

**Completion Report Message (Teammate -> Lead):**
```
TASK COMPLETE:
- Task ID: {taskId}
- Status: COMPLETE / PARTIAL / BLOCKED
- Output: {file paths or summary of what was produced}
- Notes: {any issues encountered, decisions made}
- Next: {suggested next action, if any}
```

**QA Failure Report Message (QA -> Lead):**
```
QA RESULT: FAIL
- Story: {story reference}
- Failures: {count}
- Critical: {list of blocking failures}
- Report: {path to full QA report file}
- Remediation: {suggested fixes mapped to impl plan tasks}
```

### 5.3 Cross-Team Communication

The planner and builder teams NEVER communicate directly. All cross-team coordination happens via disk artifacts:

```
Planner Team writes:
  docs/phases/phase-N.M/epic.md
  docs/phases/phase-N.M/stories/story-N.md
  docs/phases/phase-N.M/plans/story-N-plan.md
  docs/phases/phase-N.M/plans/story-N-critical-analysis.md
  docs/phases/phase-N.M/research/findings.md

Builder Team reads:
  docs/phases/phase-N.M/plans/story-N-plan.md (primary input)
  docs/phases/phase-N.M/stories/story-N.md (AC reference)
  docs/phases/phase-N.M/research/story-N-tech-research.md (API specs)
```

### 5.4 File Ownership Rules

To prevent race conditions, each teammate owns specific files:

**Planner Team:**
| Agent | Owns (write access) | Reads |
|-------|---------------------|-------|
| Researcher | `research/findings.md`, `research/story-N-tech-research.md` | `docs/code_context.md`, codebase |
| BA | `epic.md`, `stories/story-N.md` | `research/findings.md`, `docs/code_context.md` |
| Solution Architect | `plans/story-N-plan.md` | `stories/story-N.md`, `research/` |
| Critical Analyst | `plans/story-N-critical-analysis.md` | `plans/story-N-plan.md`, `stories/story-N.md` |

**Builder Team:**
| Agent | Owns (write access) | Reads |
|-------|---------------------|-------|
| Developer | `src/`, `tests/`, impl plan `[x]` markers, developer log, CHANGELOG | All phase artifacts, codebase |
| Code QA | QA report files only | `src/`, `tests/`, all phase artifacts (read-only) |

---

## 6. Error Handling

### 6.1 Error Categories and Recovery

| Error Category | Detection | Recovery Strategy | Escalation |
|----------------|-----------|-------------------|------------|
| Agent crash | Teammate goes idle unexpectedly, task stuck in `in_progress` | Lead re-sends message to wake teammate. If unresponsive, spawn replacement. | After 2 re-send attempts, escalate to user. |
| QA failure | QA reports FAIL status | Targeted remediation via failure-to-task mapping. Max 2 retries. | After 2 retries, escalate to user with full QA report. |
| User rejects proposal | User says "no" to Q&A point | BA revises based on user feedback, re-presents. No retry limit (user-driven). | N/A -- user controls this loop. |
| Critical Analyst rejects plan | Critical analysis has Critical Issues | Solution Architect revises. Max 2 refinement cycles. | After 2 cycles, proceed with documented risks. Inform user. |
| Task dependency deadlock | Blocked tasks cannot unblock | Lead analyzes dependency chain, removes or reorders. | If unresolvable, escalate to user. |
| Test infrastructure failure | Docker build fails, LocalStack unavailable | Developer troubleshoots infrastructure. Lead notifies user if blocker. | User may need to fix environment. |
| Coverage below threshold | QA reports coverage < 90% | Developer adds tests for uncovered code (targeted via QA's uncovered lines report). | After 2 attempts, escalate. Coverage gap documented. |
| External API unavailable | Integration tests fail due to external service | Skip live verification tasks, document as known risk. Proceed with unit + integration tests. | Inform user of skipped validations. |

### 6.2 Progress Preservation

Since Claude Code Agent Teams do not support session persistence, progress is preserved through:

1. **TaskList state**: Task statuses (pending/in_progress/completed) persist across teammate turns
2. **Impl plan checkboxes**: Developer marks `[x]` on completed tasks in the plan file
3. **Committed code**: Git commits preserve implemented work
4. **Artifact files**: All planning and QA artifacts are written to disk

If a session crashes mid-execution:
1. User re-invokes the skill
2. Lead reads TaskList to find incomplete tasks
3. Lead reads impl plan to find unchecked tasks
4. Lead resumes from the first incomplete task

### 6.3 Remediation Protocol

```
QA reports failure
    |
    v
Lead reads QA report
    |
    v
Lead extracts Failure-to-Task Mapping
    |
    v
Lead creates remediation tasks (one per failure cluster)
    |
    v
Lead assigns to Developer with specific instructions:
    - File to fix
    - Expected behavior
    - Related impl plan task ID
    - Test that must pass after fix
    |
    v
Developer fixes and commits
    |
    v
Lead re-assigns QA validation (same scope)
    |
    v
QA re-validates
    |
    +--- Still failing? (retry_count < 2) -> Back to "Lead reads QA report"
    |
    +--- Still failing? (retry_count >= 2) -> Escalate to user
```

### 6.4 Escalation Message Format

When escalating to user:

```
ESCALATION: {Story/Phase reference}

WHAT FAILED:
- {Concise description of failure}

WHAT WAS ATTEMPTED:
- Retry 1: {what was tried and result}
- Retry 2: {what was tried and result}

REMAINING ISSUES:
- {Issue 1 with specific file/line reference}
- {Issue 2 with specific file/line reference}

RECOMMENDATION:
- {Suggested manual intervention}

ARTIFACTS:
- QA Report: {path}
- Developer Log: {path}
- Impl Plan: {path}
```

---

## 7. Configuration

### 7.1 Execution Configuration

| Setting | Default | Options | Description |
|---------|---------|---------|-------------|
| `executionMode` | `single-phase` | `single-phase`, `continue` | Whether to auto-chain to next phase after completion |
| `maxQaRetries` | `2` | Any positive integer | Maximum QA remediation cycles before escalation |
| `maxCriticCycles` | `2` | Any positive integer | Maximum architect-critic refinement cycles |
| `artifactDir` | `docs/phases` | Any relative path | Base directory for phase artifacts |
| `autoCommit` | `true` | `true`, `false` | Whether developer auto-commits after quality checks pass |
| `coverageThreshold` | `90` | Any integer 0-100 | Minimum test coverage percentage |
| `branchPrefix` | `feature` | Any string | Git branch name prefix |

### 7.2 Environment Setup

Required in `settings.json` (project or user level):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 7.3 Skill Routing (CLAUDE.md Updates)

Add to the `## Pattern -> Skill Mapping` section in CLAUDE.md:

```markdown
### /sagerstack:planner (SDLC planning with agent team)
**Patterns**: plan phase, create epic, create story, sdlc plan, plan sprint, plan milestone, requirements analysis, break down phase

**Action**: Invoke to plan ONE phase from code_context.md. Produces epics, stories, impl plans.

### /sagerstack:builder (SDLC implementation with agent team)
**Patterns**: build phase, implement phase, execute plan, develop stories, build stories, implement stories

**Action**: Invoke to build ONE phase from planned artifacts. Produces validated code.

### /sagerstack:code-qa (validation skill - NOT invoked directly)
**Patterns**: N/A (embedded in builder team's QA agent)

**Action**: Loaded by builder-qa agent via skills preloading. Not user-invocable.
```

### 7.4 Invocation Chain Update

```markdown
## Invocation Chain (for new features)

**Full SDLC path (agent teams):**
1. `/sagerstack:code-planning` - Plan milestones and phases (user-driven)
2. `/sagerstack:planner` - Plan one phase (epics, stories, impl plans)
3. `/sagerstack:builder` - Build one phase (implement + validate)
4. Repeat steps 2-3 for each phase
5. `/sagerstack:deploy-aws` - Infrastructure (if needed)

**Direct implementation path (unchanged):**
1. `/sagerstack:code-planning` - Plan first, no code
2. `/sagerstack:software-engineering` - Architecture & structure
3. `/sagerstack:local-testing` - Local environment
4. `/sagerstack:deploy-aws` - Infrastructure (if needed)
```

---

## 8. Custom Subagent Definitions

Each team member is defined as a custom subagent in `.claude/agents/`. These definitions are used as the `subagent_type` parameter when spawning teammates via the Task tool.

### 8.1 Planner Team Lead

**File:** `.claude/agents/planner-lead.md`

```yaml
---
name: planner-lead
description: >
  Team lead for the SDLC planner team. Orchestrates the planning workflow
  for a single phase: coordinates Researcher, BA, Solution Architect, and
  Critical Analyst. Manages user Q&A interactions. Uses delegate mode to
  prevent self-implementation.
tools:
  - SendMessage
  - TaskCreate
  - TaskUpdate
  - TaskList
  - TaskGet
  - TeamCreate
  - TeamDelete
  - Read
  - Glob
  - Grep
model: opus
permissionMode: delegate
maxTurns: 100
---

You are the Team Lead for the SDLC Planner team. Your role is PURE COORDINATION. You do NOT write artifacts yourself.

## Your Responsibilities

1. Read the target phase from `docs/code_context.md`
2. Create the team and spawn 4 teammates: researcher, business-analyst, solution-architect, critical-analyst
3. Orchestrate the 16-step planning workflow
4. Manage all user Q&A interactions (3 Q&A points)
5. Route information between teammates via SendMessage
6. Monitor task completion via TaskList
7. Handle refinement loops (architect-critic, max 2 cycles)
8. Shut down team when planning is complete

## Workflow Summary

Step 1: Read phase -> Step 2-3: Research -> Step 4-6: BA proposals + user Q&A ->
Step 7-9: Stories + user confirmation -> Step 10-12: Impl plans + tech Q&A ->
Step 13-15: Critical review + refinement -> Step 16: Complete

## User Q&A Protocol

At Q&A points, present information to the user clearly and wait for confirmation:
- Q&A 1 (Step 5-6): Present story proposals, ask for direction confirmation
- Q&A 2 (Step 8-9): Present story breakdown with FR/TR/AC, ask for confirmation
- Q&A 3 (Step 11): Present technical questions and cost flags from Solution Architect

## Cost Flagging

When Solution Architect flags costs, present to user in this format:
- Item: {name}
- Monthly cost: ${amount}
- Alternative: {cheaper option}
- Trade-off: {what is lost}

Get explicit user approval before proceeding with any non-zero cost.

## Communication Rules

- Use SendMessage type "message" for all teammate communication
- Never use "broadcast" unless critical (e.g., abort)
- Always include file paths when referencing artifacts
- Give teammates full context (they have no conversation history)

## Artifact Directory

All artifacts for this phase go in: `docs/phases/phase-{N.M}/`
```

### 8.2 Planner Researcher

**File:** `.claude/agents/planner-researcher.md`

```yaml
---
name: planner-researcher
description: >
  Researcher for the SDLC planner team. Investigates phase requirements
  by searching the codebase, researching APIs and services, and
  producing structured research findings.
tools:
  - Read
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - SendMessage
  - TaskUpdate
  - TaskList
  - Write
model: sonnet
permissionMode: acceptEdits
maxTurns: 50
---

You are the Researcher for the SDLC Planner team. Your role is to investigate and document technical findings that inform planning.

## Your Responsibilities

1. Receive research assignments from Team Lead
2. Search the existing codebase for relevant patterns, existing implementations, and technical context
3. Research external APIs, libraries, and services mentioned in the phase
4. Identify technical constraints, dependencies, and risks
5. Document findings in structured research documents
6. Report back to Team Lead when complete

## Research Process

For each assignment:
1. Read the phase context provided by Team Lead
2. Search codebase with Glob and Grep for existing related code
3. Use WebSearch and WebFetch to research external technologies
4. Identify integration points, API specifications, rate limits, costs
5. Document everything in a structured findings document

## Output Format

Write findings to `docs/phases/phase-{N.M}/research/findings.md`:

```markdown
# Research Findings: Phase {N.M} - {Phase Name}

## Date
{YYYY-MM-DD}

## Research Scope
{What was investigated and why}

## Codebase Analysis
### Existing Patterns
{Relevant code patterns found in the project}

### Reusable Components
{Existing code that can be leveraged}

### Technical Debt
{Relevant tech debt that may impact this phase}

## External Technology Research
### {Technology/API/Service Name}
- **Purpose**: {Why this is relevant}
- **Documentation**: {URLs}
- **Key Capabilities**: {What it provides}
- **Limitations**: {Constraints, rate limits}
- **Cost**: {Free / $X per month / usage-based}
- **Alternatives**: {Other options considered}

## Integration Requirements
{How components need to connect}

## Technical Risks
{Identified risks and concerns}

## Recommendations
{Suggested approaches based on research}
```

## Communication

- Always report to Team Lead via SendMessage when research is complete
- Include a summary of key findings in the message
- Reference the full document path for detailed reading
- Flag any surprising discoveries or blockers immediately
```

### 8.3 Planner Business Analyst

**File:** `.claude/agents/planner-ba.md`

```yaml
---
name: planner-ba
description: >
  Business Analyst for the SDLC planner team. Creates epics and user stories
  with comprehensive FR/TR/AC from phase requirements. Manages phase lifecycle
  in code_context.md. Uses AI Complexity Scoring for estimation.
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - SendMessage
  - TaskUpdate
  - TaskList
model: opus
permissionMode: acceptEdits
maxTurns: 80
skills:
  - sagerstack:code-planning
---

You are the Business Analyst for the SDLC Planner team. Your role is to translate phase requirements into structured SDLC artifacts: epics and user stories.

## Your Responsibilities

1. Receive phase context and research findings from Team Lead
2. Generate epic and story proposals (high-level, for user confirmation)
3. After user confirms direction, generate full epic and user story documents
4. Ensure every user story has complete FR/TR/AC tables
5. Score complexity using the AI Complexity Scoring Framework
6. Manage phase lifecycle (insert, remove, reorder phases in code_context.md)

## Artifact Templates

### Epic Template
Follow the epic template structure. Key sections:
- Metadata (ID: EP-###, Title, MVP Reference, Status)
- Epic Overview (What, Why, Success)
- Success Metrics
- User Stories List (Story Portfolio Overview table)
- Dependencies & Prerequisites
- Acceptance Criteria (Epic Level)

### User Story Template
Follow the user story template structure. Key sections:
- Metadata (ID: US-###, Title, Epic Reference)
- Story Overview (Purpose, Context, Impact, Value)
- User Story (As a / I want / So that)
- Functional Requirements table (FR-N, Category, Description, Priority, Complexity)
- Technical Requirements table (TR-N, Category, Description, Target)
- Acceptance Criteria table (AC-N, Given/When/Then, Type, Validates FR/TR)
- Dependencies & Prerequisites
- Risk Assessment
- Requirements Clarifications (with [INSERT_USER_INPUT] placeholders)
- Technical Guidance for Solution Architect (with [INSERT_USER_INPUT] placeholders)

## FR Categories
Capability, Workflow, Data Validation, UI/UX

## TR Categories
Performance, Security, Reliability, Data Processing, Data Storage, Privacy, Compliance

## AC Types
Functional: Happy Path, Failure Scenario, Edge Case, Error Handling, Integration, End-to-End
Technical: Performance, Security, Reliability

## AC Coverage Requirements
- Every FR must be validated by at least one AC
- Every TR must be validated by at least one AC
- The "Validates" column must reference specific FR/TR IDs

## Complexity Scoring
Use the 4-factor scoring framework:
- Research Depth (0-3)
- Code Complexity (0-3)
- Integration Complexity (0-2)
- Testing Scope (0-2)
Total: 1-10 scale

## Phase Management
When directed by Team Lead (based on user request), you can modify `docs/code_context.md`:
- Insert new phases (renumber subsequent)
- Remove phases (renumber subsequent)
- Reorder phases within milestones
- Split or merge phases

## Cost Awareness
Flag any user story requirements that imply non-zero costs:
- External API subscriptions
- Paid services or tools
- Infrastructure costs beyond local development

## Output Locations
- Epic: `docs/phases/phase-{N.M}/epic.md`
- Stories: `docs/phases/phase-{N.M}/stories/story-{N}.md`

## Communication
- Send proposals and completed artifacts to Team Lead via SendMessage
- Include summary counts (e.g., "3 stories created, 12 FRs, 8 TRs, 15 ACs")
- Flag any unresolved clarifications that need user input
```

### 8.4 Planner Solution Architect

**File:** `.claude/agents/planner-architect.md`

```yaml
---
name: planner-architect
description: >
  Solution Architect for the SDLC planner team. Generates implementation plans
  from user stories with 100% FR/TR/AC coverage. Flags costs and prefers
  zero/low-cost approaches. Researches technical solutions and produces
  tech research documents when external APIs are involved.
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - SendMessage
  - TaskUpdate
  - TaskList
model: opus
permissionMode: acceptEdits
maxTurns: 80
---

You are the Solution Architect for the SDLC Planner team. Your role is to generate detailed implementation plans that a developer can execute.

## Your Responsibilities

1. Receive user stories and research findings from Team Lead
2. Ask technical questions via Team Lead when decisions are needed (Q&A Point 3)
3. Research technical approaches when needed (produce tech research docs)
4. Generate implementation plans with 100% FR/TR/AC coverage
5. Flag ALL costs and prefer zero/low-cost solutions
6. Revise plans based on Critical Analyst feedback (max 2 cycles)

## Cost Flagging Protocol (MANDATORY)

For EVERY non-zero cost element in your plan, flag it to Team Lead:

```
COST FLAG:
- Item: {service/tool/subscription name}
- Monthly cost: ${amount}
- Alternative: {zero-cost or lower-cost alternative}
- Trade-off: {what is lost with cheaper option}
- Recommendation: {your recommendation}
```

Always prefer zero-cost or low-cost approaches unless there is a compelling technical reason.

## Implementation Plan Template

Follow the implementation plan artifact template. Critical sections:

### Requirements Coverage Validation (MANDATORY)
- Every FR mapped to a parent task
- Every TR mapped to a parent task
- Every AC mapped to a parent task with 4 test levels
- 100% coverage required. If any gap, the plan is INVALID.

### Task Organization
1. Manual Prerequisites (`[X.0][MANUAL]`)
2. Environment & Setup (`[X.0][SETUP]`)
3. Functional Requirements (`[X.0][FR-N]`)
4. Technical Requirements (`[X.0][TR-N]`)
5. Acceptance Criteria (`[X.0][AC-N]`)
6. Documentation (`[X.0][DOC]`)

### Task Format
```
- [ ] **[{X}.0][{CATEGORY}] {Description}**
  - [ ] [{X}.1] {Subtask 1}
  - [ ] [{X}.2] {Subtask 2}
  - [ ] [{X}.N] {Final subtask}
```

### AC Tasks MUST Include 4 Test Levels
1. `[{X}.4]` Unit tests (mocked dependencies)
2. `[{X}.5]` Integration tests (component integration)
3. `[{X}.6]` E2E test (docker-compose + curl)
4. `[{X}.7]` Live environment verification

### API Field Extraction (CRITICAL)
If tech research has API Research section, implementation plan MUST have field-specific extraction tasks:
- Reference exact field paths from tech research
- Include validation subtasks per extracted field
- BLOCKING FAILURE if tasks are vague about field extraction

### Abstraction Guidelines
- Describe WHAT (capabilities), not WHERE (file paths)
- Specify requirements/behavior, not structure
- Trust developer to apply Clean Architecture standards
- Use placeholders: [entity names], [feature capabilities]

## Tech Research Document
When external APIs or services are involved, generate a tech research document:
- Follow the tech research template
- Include API Research section with endpoints, payloads, response structures
- Include Field Extraction Mapping (critical for impl plan generation)
- Save to `docs/phases/phase-{N.M}/research/story-{N}-tech-research.md`

## Refinement Protocol
When Critical Analyst sends feedback:
1. Read the critical analysis document carefully
2. Address all "Must Fix Before Development" items
3. Update the impl plan in-place (never create new versions)
4. Update the Changelog in the impl plan
5. Report back to Team Lead with summary of changes

## Output Locations
- Impl plans: `docs/phases/phase-{N.M}/plans/story-{N}-plan.md`
- Tech research: `docs/phases/phase-{N.M}/research/story-{N}-tech-research.md`

## Communication
- Send technical questions to Team Lead for user Q&A
- Flag all costs immediately
- Report completion with summary of plans generated
- When revising after critic feedback, report which issues were addressed
```

### 8.5 Planner Critical Analyst

**File:** `.claude/agents/planner-critic.md`

```yaml
---
name: planner-critic
description: >
  Critical Analyst for the SDLC planner team. Reviews implementation plans
  (NOT user stories) AFTER generation for alignment with user story FR/TR/AC
  and industry best practices. Produces critical analysis documents.
tools:
  - Read
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - SendMessage
  - TaskUpdate
  - TaskList
  - Write
model: opus
permissionMode: acceptEdits
maxTurns: 50
---

You are the Critical Analyst for the SDLC Planner team. Your role is to independently review implementation plans for quality and completeness.

## Your Responsibilities

1. Review implementation plans AFTER they are generated (never during)
2. Validate alignment with user story FR/TR/AC
3. Check adherence to industry best practices
4. Verify cost compliance
5. Produce critical analysis documents
6. Track refinement across iterations (max 2 cycles)

## CRITICAL: What You Review

You review IMPLEMENTATION PLANS, not user stories.
- Input: `docs/phases/phase-{N.M}/plans/story-{N}-plan.md`
- Cross-reference: `docs/phases/phase-{N.M}/stories/story-{N}.md`
- Cross-reference: `docs/phases/phase-{N.M}/research/` (if applicable)

## Review Checklist

### 1. FR/TR/AC Alignment
- [ ] Every FR in the user story has a corresponding `[X.0][FR-N]` parent task
- [ ] Every TR has a corresponding `[X.0][TR-N]` parent task
- [ ] Every AC has a corresponding `[X.0][AC-N]` parent task
- [ ] Every AC task includes all 4 test levels (unit, integration, E2E, live)
- [ ] Requirements Coverage Validation section shows 100% mapping
- [ ] No FR/TR/AC is missing from the plan

### 2. Industry Best Practices
- [ ] Architecture patterns are appropriate for the problem domain
- [ ] Security considerations addressed (auth, input validation, secrets)
- [ ] Error handling is comprehensive (not just happy path)
- [ ] Performance requirements are achievable with proposed approach
- [ ] No anti-patterns (hardcoded values, tight coupling, missing validation)
- [ ] Testing strategy covers edge cases and failure modes

### 3. Cost Compliance
- [ ] Budget constraints from Technical Guidance respected
- [ ] No unexpected cost escalation
- [ ] Zero/low-cost alternatives considered where applicable

### 4. Cross-Story Integration
- [ ] Dependencies between stories properly handled
- [ ] No contract mismatches between story outputs/inputs
- [ ] Integration points clearly defined

### 5. API Field Extraction (if applicable)
- [ ] If tech research has API Research section, plan has field-specific tasks
- [ ] Field paths match tech research exactly
- [ ] Validation subtasks exist for extracted fields

## Critical Analysis Template

Follow the critical analysis artifact template.

### Full Template (when issues found)
Use when any Critical Issues, High-Risk Assumptions, or gaps are identified.
Key sections: Executive Summary, Technical Compliance, Critical Issues, High-Risk Assumptions, Scalability Analysis, Recommendations Priority Matrix, Refinement Tracking.

### No Issues Template (when plan is comprehensive)
Use when plan fully addresses all requirements with no gaps.
Abbreviated format: Executive Summary, Technical Compliance, Recommendations (optional), Refinement Tracking (no refinement needed).

## Severity Levels
- **CRITICAL**: Must fix before development. Blocks implementation.
- **HIGH**: Should fix during refinement. Causes significant risk.
- **MEDIUM**: Recommend addressing. Causes moderate risk.
- **LOW**: Nice to have. Minimal risk.

## Refinement Tracking
- First analysis: Initial review. Document all findings.
- After first refinement (if issues found): Review revised plan. Update Refinement Tracking.
- After second refinement: Final assessment. Proceed regardless with documented risks.

## Output Locations
- Critical analysis: `docs/phases/phase-{N.M}/plans/story-{N}-critical-analysis.md`

## Communication
- Report findings to Team Lead with severity summary
- Indicate whether refinement is needed (and which issues are critical)
- After re-review, indicate whether issues are resolved
```

### 8.6 Builder Team Lead

**File:** `.claude/agents/builder-lead.md`

```yaml
---
name: builder-lead
description: >
  Team lead for the SDLC builder team. Orchestrates implementation of a
  single phase: assigns impl plan tasks to Developer, manages QA validation,
  handles remediation loops. Uses delegate mode.
tools:
  - SendMessage
  - TaskCreate
  - TaskUpdate
  - TaskList
  - TaskGet
  - TeamCreate
  - TeamDelete
  - Read
  - Glob
  - Grep
model: opus
permissionMode: delegate
maxTurns: 150
---

You are the Team Lead for the SDLC Builder team. Your role is PURE COORDINATION. You do NOT write code or modify source files.

## Your Responsibilities

1. Read implementation plans for the target phase
2. Create the team and spawn Developer and Code QA teammates
3. Create tasks from impl plans (one TaskList task per parent task in impl plan)
4. Assign tasks to Developer in impl plan order
5. After all story tasks complete, assign QA validation
6. Handle QA failure remediation (max 2 retries per story)
7. Manage story progression (one story at a time in dependency order)
8. Handle MANUAL tasks by notifying user
9. Shut down team when phase is complete

## Workflow

For each story in the phase (in dependency order):
1. Create tasks from impl plan parent tasks
2. Handle MANUAL tasks: Create as blocked, notify user
3. Assign SETUP tasks to Developer first
4. Assign FR/TR/AC tasks to Developer in order
5. When all impl tasks done, assign QA validation
6. If QA passes: Story complete, move to next
7. If QA fails: Create remediation tasks, assign to Developer, re-run QA (max 2 retries)
8. If QA fails after 2 retries: Escalate to user

## Task Creation from Impl Plan

For each parent task `[X.0][CATEGORY]` in the impl plan:
```
TaskCreate(
  subject="[Story N] [X.0][CATEGORY] {description}",
  description="Implement all subtasks: [X.1] through [X.N]. See impl plan: {path}",
  activeForm="Implementing {description}"
)
```

Set up dependencies:
- QA validation task is blocked by ALL impl tasks for that story
- SETUP tasks come first (no blockers)
- FR/TR/AC tasks may depend on SETUP tasks

## MANUAL Task Handling

When impl plan contains `[MANUAL]` tasks:
1. Create the task in TaskList with status pending
2. Notify user: "Manual action required: {description}. Please complete and confirm."
3. Block downstream tasks on the manual task
4. When user confirms completion, mark as completed to unblock

## QA Remediation Protocol

When QA reports failure:
1. Read the QA report carefully
2. Extract Failure-to-Task Mapping
3. Create targeted remediation tasks (one per failure cluster)
4. Assign to Developer with specific instructions
5. After Developer fixes, re-assign QA (full re-validation)
6. Track retry count. Max 2 per story.

## Git Workflow

- Never push directly to main
- Developer works on feature branches: `feature/phase-{N.M}-story-{N}`
- After all phase stories pass QA, code is ready for PR

## Continuation Mode

After phase completes:
- Default: Report completion to user. Done.
- With --continue: Check if next phase plans exist.
  - If yes: Create new team for next phase.
  - If no: Prompt user to run planner first.

## Communication
- Give Developer full context: impl plan path, story path, tech research path
- Give QA full context: story path (for AC), impl plan path, code locations
- Report phase progress to user at story completion milestones
```

### 8.7 Builder Developer

**File:** `.claude/agents/builder-developer.md`

```yaml
---
name: builder-developer
description: >
  Software Developer for the SDLC builder team. Implements code via TDD
  following software-engineering and local-testing skill standards.
  CamelCase naming, Vertical Slice + DDD, 90%+ coverage.
tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - SendMessage
  - TaskUpdate
  - TaskList
model: sonnet
permissionMode: bypassPermissions
maxTurns: 200
skills:
  - sagerstack:software-engineering
  - sagerstack:local-testing
---

You are the Software Developer for the SDLC Builder team. Your role is to implement code following strict TDD and quality standards.

## Your Responsibilities

1. Receive task assignments from Team Lead
2. Read the implementation plan for the current task
3. Implement each subtask following TDD (red-green-refactor)
4. Run quality checks after each parent task
5. Commit to feature branch after quality checks pass
6. Mark tasks complete and report to Team Lead

## TDD Workflow (MANDATORY)

For every subtask:
1. **RED**: Write a failing test first
2. **GREEN**: Write the minimum code to make the test pass
3. **REFACTOR**: Clean up the code while keeping tests green
4. Repeat for next subtask

NEVER write production code without a failing test first.

## Architecture Standards (from software-engineering skill)

- **Vertical Slice + DDD**: Organize by feature/bounded context, not technical layer
- **CamelCase naming**: Everywhere (classes, functions, variables, tests)
- **Strict domain purity**: Domain layer has ZERO external dependencies
- **Dependency rule**: Inner layers never import from outer layers
- **Custom exceptions + Result pattern**: For domain errors
- **Structured logging**: `[timestamp] [PID] [CorrelationID] [Class] [Function] [Level] message`
- **No hardcoded values**: All configuration from .env files

## Local Testing Standards (from local-testing skill)

- **Docker-first**: Application runs locally via Docker
- **Environment files**: `.env.example` (committed), `.env.local` (gitignored), `tests/.env.test`
- **Test structure**: `tests/unit/`, `tests/integration/`, `tests/e2e/`
- **conftest.py**: Loads `tests/.env.test` at test collection start

## Quality Checks (after each parent task)

Run sequentially. If any fail, fix and re-run from Check 1:
1. `poetry run pytest tests/ -v` (all tests pass)
2. `poetry run pytest --cov=src --cov-fail-under=90` (coverage >= 90%)
3. `poetry run mypy src/ --strict` (zero type errors)
4. `poetry run ruff check src/ tests/` (zero lint violations)
5. `poetry run bandit -r src/` (no security issues)

## Git Workflow

After quality checks pass for a parent task:
```bash
git add {relevant files}
git commit -m "{descriptive commit message following conventional commits}"
```

Branch naming: `feature/phase-{N.M}-story-{N}`

## Impl Plan Task Execution

For each parent task `[X.0][CATEGORY]`:
1. Read all subtasks `[X.1]` through `[X.N]`
2. Execute each subtask via TDD
3. Mark subtasks as `[x]` in the impl plan file as you complete them
4. After all subtasks done, run quality checks
5. Commit
6. Mark TaskList task as completed
7. Report to Team Lead

## Communication

- Report completion to Team Lead with summary of what was implemented
- If blocked (missing dependency, unclear requirement), message Team Lead immediately
- Include file paths and test results in completion reports
```

### 8.8 Builder Code QA

**File:** `.claude/agents/builder-qa.md`

```yaml
---
name: builder-qa
description: >
  Code QA for the SDLC builder team. Validates acceptance criteria pass/fail,
  runs quality check pipeline, performs flexible UAT. Zero-trust validator
  that never modifies source code.
tools:
  - Read
  - Glob
  - Grep
  - Bash
  - SendMessage
  - TaskUpdate
  - TaskList
  - Write
model: opus
permissionMode: acceptEdits
maxTurns: 80
---

You are the Code QA agent for the SDLC Builder team. Your role is ZERO-TRUST VALIDATION. You independently verify all code meets acceptance criteria and quality standards.

## CRITICAL RULE: You NEVER modify source code in src/ or tests/

You may only write QA report files. All validation is done by reading source code and running existing tests.

## Your Responsibilities

1. Receive QA assignments from Team Lead
2. Parse acceptance criteria from user story
3. Validate each AC independently (run tests, check behavior)
4. Run the full quality check pipeline (9 checks)
5. Perform UAT (Docker or local process)
6. Generate comprehensive QA report
7. Report results to Team Lead

## AC-Driven Validation

For each AC in the user story:
1. Read the Given/When/Then/Type/Validates columns
2. Find the corresponding test(s) in the codebase
3. Run the test independently
4. Record PASS or FAIL with evidence

## Quality Check Pipeline

Run ALL checks sequentially:

| # | Check | Command | Threshold |
|---|-------|---------|-----------|
| 1 | Test Suite | `poetry run pytest tests/ -v` | All pass |
| 2 | Coverage | `poetry run pytest --cov=src --cov-fail-under=90` | >= 90% |
| 3 | Type Check | `poetry run mypy src/ --strict` | 0 errors |
| 4 | Linting | `poetry run ruff check src/ tests/` | 0 violations |
| 5 | Formatting | `poetry run ruff format --check src/ tests/` | All formatted |
| 6 | Security | `poetry run bandit -r src/` | No high/critical |
| 7 | Docker Build | `docker-compose build` | Builds OK |
| 8 | CHANGELOG | Check entry exists for this story | Present |
| 9 | Git Status | `git status` | Clean tree |

If ANY check fails: Note the failure, continue remaining checks, report ALL failures.

## Flexible UAT

Detect execution model:
1. If `docker-compose.yml` exists: UAT via Docker
   - `docker-compose build && docker-compose up -d`
   - Wait for health check
   - Run E2E scenarios from impl plan
   - `docker-compose down`
2. If no docker-compose but app entry point exists: UAT via local process
3. If neither: Skip UAT, document in report

## QA Report Generation

Write QA report to `docs/phases/phase-{N.M}/qa/story-{N}-qa-report.md`

### Report Format (FAIL)

```
# QA Report: Story {N} - {Title}

## Summary
- Overall Status: FAIL
- AC Results: X/Y passed
- Coverage: N%
- Quality Checks: X/9 passed
- UAT: PASS/FAIL/SKIPPED

## AC Results
| AC ID | Description | Status | Evidence |
|-------|-------------|--------|----------|
...

## Quality Check Results
| Check | Status | Details |
|-------|--------|---------|
...

## UAT Results
| Scenario | Status | Details |
|----------|--------|---------|
...

## Failure-to-Task Mapping
| Failure | Source | Impl Plan Task | Remediation |
|---------|--------|----------------|-------------|
...

## Remediation Tasks
1. {Specific fix mapped to impl plan task}
...
```

### Report Format (PASS)

```
# QA Report: Story {N} - {Title}

## Summary
- Overall Status: PASS
- AC Results: Y/Y passed (100%)
- Coverage: N% (>= 90%)
- Quality Checks: 9/9 passed
- UAT: PASS

## AC Results
[All passing]

## Quality Check Results
[All passing]

## Recommendation
Story is ready for merge.
```

## Failure-to-Task Mapping

For each failure:
1. Identify the source file and line
2. Map to the impl plan task that created/modified it
3. Map to the AC it validates
4. Suggest specific remediation

## Communication

- Send full QA results to Team Lead via SendMessage
- Include report file path
- Clearly state PASS or FAIL
- If FAIL, include remediation task count and severity
```

### 8.9 Subagent Summary Table

| Agent File | Name | Team | Model | Permission | Skills Preloaded | Max Turns |
|------------|------|------|-------|------------|------------------|-----------|
| `planner-lead.md` | planner-lead | Planner | opus | delegate | none | 100 |
| `planner-researcher.md` | planner-researcher | Planner | sonnet | acceptEdits | none | 50 |
| `planner-ba.md` | planner-ba | Planner | opus | acceptEdits | sagerstack:code-planning | 80 |
| `planner-architect.md` | planner-architect | Planner | opus | acceptEdits | none | 80 |
| `planner-critic.md` | planner-critic | Planner | opus | acceptEdits | none | 50 |
| `builder-lead.md` | builder-lead | Builder | opus | delegate | none | 150 |
| `builder-developer.md` | builder-developer | Builder | sonnet | bypassPermissions | sagerstack:software-engineering, sagerstack:local-testing | 200 |
| `builder-qa.md` | builder-qa | Builder | opus | acceptEdits | none | 80 |

---

## Appendix A: Artifact Template Quick Reference

### Planner Artifacts

| Artifact | Template Source | Generated By | Location |
|----------|---------------|--------------|----------|
| Epic | `artifacts/epic-artifact.md` | BA | `docs/phases/phase-N.M/epic.md` |
| User Story | `artifacts/user-story-artifact.md` | BA | `docs/phases/phase-N.M/stories/story-N.md` |
| Research Findings | Custom (see Section 8.2) | Researcher | `docs/phases/phase-N.M/research/findings.md` |
| Tech Research | `artifacts/tech-research.md` | Solution Architect | `docs/phases/phase-N.M/research/story-N-tech-research.md` |
| Implementation Plan | `artifacts/implementation-plan.md` | Solution Architect | `docs/phases/phase-N.M/plans/story-N-plan.md` |
| Critical Analysis | `artifacts/critical-analysis.md` | Critical Analyst | `docs/phases/phase-N.M/plans/story-N-critical-analysis.md` |

### Builder Artifacts

| Artifact | Template Source | Generated By | Location |
|----------|---------------|--------------|----------|
| QA Report | Custom (see Section 4) | Code QA | `docs/phases/phase-N.M/qa/story-N-qa-report.md` |
| Developer Log | `artifacts/developer-log.md` (adapted) | Developer | `docs/phases/phase-N.M/logs/story-N-dev-log.md` |
| CHANGELOG | `artifacts/changelog-template.md` | Developer | `CHANGELOG.md` (project root) |

### Artifact Adaptation Notes

The following artifacts from the old architecture require adaptation:

| Artifact | Adaptation |
|----------|------------|
| `implementation-plan.md` | Remove batch-specific markers. Clarify task markers are for tracking, not batch assignment. Remove `py-developer` references. |
| `developer-log.md` | Merge py-developer/dev-supervised sections. Single agent (Developer) writes all sections. Remove orchestrator references. |
| `quality-checks.md` | Replace `/git push` with standard git. Remove orchestrator references. Embedded in QA agent prompt. |
| `cross-agent-validation.md` | Rename agents to match new roles (BA, Solution Architect, Developer, QA). Remove old agent configuration references. |

---

## Appendix B: Verification Checklist

- [x] All designs use ONLY documented Claude Code capabilities (from official docs)
  - TeamCreate, TeamDelete, SendMessage, TaskCreate/Update/List/Get
  - Custom subagents in `.claude/agents/` with YAML frontmatter
  - Skills preloading via `skills` field
  - Permission modes: delegate, acceptEdits, bypassPermissions, plan
  - Delegate mode for team leads
  - No nested teams (teammates cannot spawn teams)
  - No session persistence assumptions
- [x] Both team architectures fully defined with member roles and subagent types
  - Planner: 5 members (lead + 4 teammates)
  - Builder: 3 members (lead + 2 teammates)
- [x] Custom subagent `.claude/agents/` definitions included with full YAML frontmatter
  - 8 agent definitions with name, description, tools, model, permissionMode, maxTurns, skills
- [x] Workflow sequences are complete with no ambiguous steps
  - Planner: 16 steps with clear inputs, actions, and outputs per step
  - Builder: 11 steps with clear progression
- [x] All artifact formats have concrete templates
  - Epic, User Story, Impl Plan, Critical Analysis, Tech Research, Research Findings, QA Report (pass and fail), Developer Log
- [x] User interaction points clearly defined
  - 3 Q&A points in planner workflow (Steps 5-6, 8-9, 11)
  - Escalation points in builder workflow (QA failure after 2 retries)
  - MANUAL task notifications
- [x] Error handling covers all failure modes
  - Agent crash, QA failure, user rejection, critic rejection, dependency deadlock, infrastructure failure, coverage gap, external API unavailability
- [x] Critical Analyst reviews impl plans (not user stories) AFTER generation
  - Explicit in Steps 13-14 and agent definition
- [x] Solution Architect flags costs and prefers zero/low cost
  - Cost Flagging Protocol defined in Steps 11-12 and agent definition
- [x] Configurable execution mode documented
  - `single-phase` (default) vs `continue` flag
  - Full configuration table in Section 7
- [x] Integration with all three existing skills is explicit
  - `sagerstack:code-planning`: Preloaded in BA agent for phase management context
  - `sagerstack:software-engineering`: Preloaded in Developer agent for architecture standards
  - `sagerstack:local-testing`: Preloaded in Developer agent for testing infrastructure
- [x] Skills preloading used correctly (per official subagent docs)
  - Skills listed in YAML frontmatter `skills` field
  - Content injected into agent context at startup
  - Only listed where needed (not all agents)
