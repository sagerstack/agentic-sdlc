# Agent Teams Research Synthesis

## Document Purpose

This document synthesizes findings from three research areas to inform the design of two new Claude Code skills: `/sagerstack:planner` and `/sagerstack:builder`. The old design used a custom three-agent architecture (orchestrator/dev-supervised, py-developer, code-qa). The new design leverages Claude Code's native team capabilities.

**Context**: Redesigning an AI-powered SDLC framework. Two new skills will replace the old custom agent architecture:
- `/sagerstack:planner` -- agent team: Business Analyst, Researcher, Solution Architect, Critical Analyst
- `/sagerstack:builder` -- agent team: Software Developer, Code QA

---

## Section 1: Artifact Specifications (KEEP / ADAPT / DROP)

### 1.1 Product Artifacts (Templates for SDLC Outputs)

| # | Artifact | File | Verdict | Rationale |
|---|----------|------|---------|-----------|
| 1 | MVP Iteration | `mvp-artifact.md` | **KEEP** | Defines MVP scope, epic themes, success criteria. Product-level artifact unchanged by architecture shift. Template structure (Metadata, Goal, In Scope, Out of Scope, Epics table, Success Criteria, Market Research Validation) is agent-agnostic. |
| 2 | Epic | `epic-artifact.md` | **KEEP** | Defines capability groupings, dependency types (Blocking/Prerequisite/Informational), sequencing priorities (P1-P3). Contains data flow dependency patterns (Data Foundation, Integration, Business Logic). Agent-agnostic. |
| 3 | User Story | `user-story-artifact.md` | **KEEP** | Core template with FR/TR/AC traceability. FR categories (Capability, Workflow, Data Validation, UI/UX), TR categories (Performance, Security, Reliability, Data Processing, Storage, Privacy, Compliance), AC types (Functional subtypes + Technical subtypes). The "Validates" column linking ACs to FR/TR IDs is a critical traceability mechanism. Requirements Clarifications and Technical Guidance sections remain relevant. |
| 4 | Tech Research | `tech-research.md` | **KEEP** | Research companion to implementation plan. API Research section (REQUIRED for external data sources) with executable scripts, actual JSON responses, field extraction mapping. NO_MOCK_DATA_POLICY embedded. Relationship: Tech Research = WHAT/WHY, Implementation Plan = HOW. |
| 5 | Implementation Plan | `implementation-plan.md` | **ADAPT** | Task-based execution guide. Core template structure stays: Metadata, Quick Reference, Requirements Coverage Validation (mandatory 100% FR/TR/AC mapping), Task-Based Plan. **Adaptations needed**: (1) Remove batch-specific markers that assumed old orchestrator -- the `[X.0][CATEGORY]` parent task and `[X.Y]` subtask format is still useful but batch execution logic moves to skill orchestration. (2) The 4 test levels per AC (Unit, Integration, E2E, Live Verification) are valuable and should remain. (3) API Field Extraction Task Requirements (CRITICAL section) should remain as-is -- it enforces field-specific extraction per tech research. (4) BLOCKING FAILURE conditions should remain as validation rules for the Solution Architect agent. |
| 6 | Critical Analysis | `critical-analysis.md` | **KEEP** | Independent review of implementation plans. Technology Stack Alignment, Architecture Pattern Compliance, Budget Compliance, Critical Issues (Blockers), High-Risk Assumptions, Scalability Analysis, Recommendations Priority Matrix. Has a "No Issues" minimal template variant. The 1:1 relationship with implementation plan and refinement tracking (1st/2nd analysis iterations) is critical for the planner's architect-critic loop. |
| 7 | Developer Log | `developer-log.md` | **ADAPT** | 6-section structured log. **Adaptations needed**: (1) Remove py-developer/dev-supervised responsibility split -- in new design, the builder skill's Software Developer agent writes all sections. (2) Retain the Section 2 Implementation Tasks table (Timestamp, Phase, Task, Files, Approach, Key Decisions, Tests). (3) Retain Section 3 Quality Checks Summary. (4) Simplify Sections 4-6 (Summary, Issues, Recommendations) into a single post-implementation report since there is no separate orchestrator agent. |
| 8 | Changelog Template | `changelog-template.md` | **KEEP** | Keep a Changelog format per user story. Simple, agent-agnostic. Created during implementation, summarized into product-wide CHANGELOG.md. |

### 1.2 Quality & Validation Artifacts

| # | Artifact | File | Verdict | Rationale |
|---|----------|------|---------|-----------|
| 9 | Code Quality Standards | `code-quality-standards.md` | **KEEP** | Comprehensive standards: SOLID principles, type hints, Google-style docstrings, custom exception hierarchies, structured logging with correlation IDs, immutable value objects, constructor DI. 5 Data Policies (NO_MOCK_DATA, NO_HARDCODING, NO_ARTIFICIAL_LIMITS, COMPLETE_API_DATA, FILTER_BEFORE_LIMIT). Configuration Management (secrets in .env.local, config in app_config.py). Testing standards (95% coverage, test naming convention). Pre-commit hooks (Black, Ruff, MyPy, Bandit). These standards are agent-agnostic and should be embedded in the builder skill. |
| 10 | Quality Checks | `quality-checks.md` | **ADAPT** | 9 sequential quality checks after each parent task. **Adaptations needed**: (1) Remove `/git push` slash command reference in Check 8 -- use standard git operations. (2) Remove the old orchestrator references. (3) Retain the sequential check pipeline: Tests -> Coverage (95%) -> MyPy strict -> Ruff/Black -> Bandit -> Docker Build -> CHANGELOG -> Git Commit -> Task Completion marking. (4) The "failure = STOP, fix, re-run from Check 1" pattern is valuable for the builder's Code QA agent. |
| 11 | Cross-Agent Validation | `cross-agent-validation.md` | **ADAPT** | Validation rules between BA -> Solution Architect -> Developer handoffs. **Adaptations needed**: (1) Rename agents to match new roles (business-analyst -> BA agent, solution-architect -> Solution Architect agent, py-developer -> Software Developer agent). (2) Core validation rules remain: Measurable AC requirements, Config Value Validation, Business Logic Validation (test OUTCOMES not PROCESS), E2E/Live/Docker test requirements. (3) The FR Intent Preservation Validation (action verb, temporal sequence, data source preservation) is critical for Solution Architect quality. (4) Service Integration Task Requirements (8-step mandate for new services) should remain. (5) Docker Redeployment Before Testing rule should remain. |
| 12 | Cross-Story Integration Pattern | `cross-story-integration-pattern.md` | **KEEP** | Universal pattern for preventing contract mismatch and workflow incompleteness gaps across user stories. 6-Step Prevention Framework (Dependency Contract Mapping, Integration Gap Detection, Workflow Completeness Validation, Integration Layer Task Creation, Integration Accountability Matrix, Self-Validation Checklist). BLOCKING VALIDATION FAILURE if dependencies exist but no Cross-Story Integration Analysis. Agent-agnostic pattern. |

### 1.3 Process & Architecture Artifacts

| # | Artifact | File | Verdict | Rationale |
|---|----------|------|---------|-----------|
| 13 | AI SDLC | `ai-sdlc.md` | **DROP** | Core SDLC process with three-agent architecture (/dev orchestrator, py-developer, code-qa). Batch processing logic, checkpoint management, session context structure, mermaid flow diagram -- all specific to the old custom architecture. Claude Code's native team capabilities replace all of this. The workflow phases (Batch Planning, Batch Execution Loop, QA Validation, Completion) are replaced by TeamCreate + TaskList coordination. |
| 14 | Process Implementation Plan | `process-implementation-plan.md` | **DROP** | Cursor-era implementation plan management ("one sub-task at a time, ask user for permission"). Superseded by the implementation-plan.md artifact template and Claude Code's native task execution. |
| 15 | Retry Decision Logic | `retry-decision-logic.md` | **DROP** | Retry patterns specific to the old /dev orchestrator: timeout detection, dual-source progress analysis (impl plan [x] markers + developer log), retry counters (retry_count, qa_retry_count), escalation messages. Claude Code handles agent failures natively through team coordination and message passing. |
| 16 | Task Batching Algorithm | `task-batching-algorithm.md` | **DROP** | Sequential batching algorithm (max 5 tasks, MANUAL task isolation, no reordering). This was the heart of the old orchestrator's batch planning. In the new design, the builder skill orchestrates task execution through Claude Code's TaskList without needing a custom batching algorithm. |
| 17 | AI Complexity Scoring | `ai-complexity-scoring-framework.md` | **KEEP** | 4-factor scoring (Research Depth 0-3, Code Complexity 0-3, Integration Complexity 0-2, Testing Scope 0-2). Score interpretation (1-2 Trivial to 9-10 Very High). Determines architect-critic collaboration needs. Agent-agnostic, used by BA and Solution Architect during planning. |
| 18 | Artifacts File Management | `artifacts-file-mgt.md` | **KEEP** | Folder structure standards (domain/execution/mvp{N}-{goal}/), naming conventions (EP-###, US-###), auto-derivation logic for IDs (product-wide uniqueness). Critical for artifact organization. Agent-agnostic. |

### 1.4 Code Pattern Artifacts

| # | Artifact | File | Verdict | Rationale |
|---|----------|------|---------|-----------|
| 19 | DDD Patterns | `ddd-patterns.md` | **KEEP** | Comprehensive DDD pattern catalog: Value Objects (frozen dataclass), Entities (UUID identity), Aggregates (10-item limit example), Repository (Protocol interface), Domain Events (frozen, past-tense), Domain Services (stateless), Specification Pattern (composable AND/OR/NOT), Factory Pattern. Common pitfalls (Anemic Domain Model, Large Aggregates, ORM model leakage). Should be embedded in builder skill as reference. |
| 20 | Project Structure | `project-structure.md` | **KEEP** | Clean Architecture + DDD directory structure (src/domain, src/application, src/infrastructure, src/presentation, tests/unit, tests/integration, tests/e2e). Layer descriptions, testing strategy, configuration files, Poetry commands. Agent-agnostic reference. |
| 21 | Popular AI Prompts | `popular-ai-prompts.md` | **DROP** | Curated prompts for Python backend development (FastAPI, Clean Architecture, DDD, TDD). These are reference prompts from external sources, not operational artifacts. The patterns they describe are already captured in the DDD Patterns, Code Quality Standards, and Project Structure artifacts. No value for skill design. |

### 1.5 Verdict Summary

| Verdict | Count | Artifacts |
|---------|-------|-----------|
| **KEEP** | 11 | MVP, Epic, User Story, Tech Research, Critical Analysis, Changelog, Code Quality Standards, Cross-Story Integration, AI Complexity Scoring, Artifacts File Management, DDD Patterns, Project Structure |
| **ADAPT** | 4 | Implementation Plan, Developer Log, Quality Checks, Cross-Agent Validation |
| **DROP** | 6 | AI SDLC, Process Implementation Plan, Retry Decision Logic, Task Batching Algorithm, Popular AI Prompts, (none -- corrected to 5) |

**Correction**: 11 KEEP + 4 ADAPT + 6 DROP = 21 total. The counts above include all 21 artifacts.

---

## Section 2: Claude Code Team Capabilities

### 2.1 Team Infrastructure

| Capability | Tool | Key Parameters | Notes |
|------------|------|----------------|-------|
| Create team | `TeamCreate` | `team_name`, `description` | Creates team file at `~/.claude/teams/{name}.json` and task list at `~/.claude/tasks/{name}/` |
| Delete team | `TeamDelete` | (none) | Removes team and task directories. Fails if active members remain. |
| Send message | `SendMessage` | `type` (message/broadcast/shutdown_request/shutdown_response), `recipient`, `content`, `summary` | Direct messages and broadcasts. Recipients identified by NAME. |
| Create task | `TaskCreate` | `subject`, `description`, `activeForm` | Tasks created with status `pending`, no owner |
| Update task | `TaskUpdate` | `taskId`, `status`, `owner`, `addBlocks`, `addBlockedBy` | Status workflow: pending -> in_progress -> completed. Supports dependency chains. |
| List tasks | `TaskList` | (none) | Returns id, subject, status, owner, blockedBy |
| Get task detail | `TaskGet` | `taskId` | Full details including description, blocks, blockedBy |

### 2.2 Agent Spawning

Agents are spawned via the `Task` tool (not listed in the function catalog above, but referenced in skill documentation). Key parameters:

- `subagent_type`: Determines agent capabilities. Options include `general-purpose` (full tool access), read-only types (Explore, Plan), and custom agents defined in `.claude/agents/`.
- `team_name` and `name`: Join the agent to a team with a human-readable name.
- `plan_mode_required`: When true, agent must get plan approved before executing.
- Parallel spawning: Multiple Task tool calls in a single message execute in parallel.

### 2.3 Communication Patterns

| Pattern | Mechanism | Use Case |
|---------|-----------|----------|
| Leader -> Teammate | `SendMessage` type: "message" | Assign work, provide context, give feedback |
| Teammate -> Leader | `SendMessage` type: "message" | Report completion, ask questions, raise blockers |
| Broadcast | `SendMessage` type: "broadcast" | Critical team-wide announcements (use sparingly) |
| Shutdown | `SendMessage` type: "shutdown_request" / "shutdown_response" | Graceful teammate termination |
| Plan approval | `SendMessage` type: "plan_approval_response" | Approve/reject teammate plans when plan_mode_required |

### 2.4 Task Coordination

- **Shared TaskList**: All team members read/write the same task list.
- **Dependency chains**: `addBlocks` and `addBlockedBy` fields prevent premature task execution.
- **Ownership**: Tasks assigned via `TaskUpdate` with `owner` parameter.
- **Status workflow**: `pending` -> `in_progress` -> `completed` (or `deleted`).
- **Idle state**: Teammates go idle after each turn -- this is normal. Sending a message wakes them.
- **Auto-delivery**: Messages from teammates are automatically delivered; no manual inbox checking.

### 2.5 Capability Constraints

| Constraint | Detail |
|------------|--------|
| No real-time monitoring | Leader analyzes results AFTER agent returns, not during execution |
| No custom retry logic | Unlike old architecture's retry_count/qa_retry_count, retries are manual (leader sends new message) |
| No session persistence | No `.dev-session-*.json` checkpoints. State lives in TaskList and messages. |
| No batching algorithm | Tasks are individually created and assigned, not algorithmically batched |
| Agent tool restrictions | Read-only agents cannot edit files. Tool restrictions per agent type. |
| Message costs | Broadcasts are expensive (N messages for N teammates). Prefer direct messages. |

### 2.6 Comparison: Old Architecture vs Claude Code Teams

| Aspect | Old Architecture | Claude Code Teams |
|--------|------------------|-------------------|
| Agent creation | Custom agent definitions in `.claude/agents/` | `TeamCreate` + Task tool spawning |
| Task distribution | Algorithmic batching (max 5 tasks) | Manual TaskCreate + TaskUpdate ownership |
| Progress tracking | Dual-source (impl plan [x] + developer log) | TaskList status + message passing |
| Retry logic | Automated (retry_count, qa_retry_count, max 2) | Manual leader decision + new message |
| Session recovery | `.dev-session-*.json` checkpoint files | TaskList persists across turns |
| Quality validation | Independent code-qa agent (zero trust) | Code QA teammate with read-only tools |
| Communication | None (agents don't talk to each other) | SendMessage (direct + broadcast) |
| Parallelism | Sequential batch execution | Parallel Task tool calls in single message |

---

## Section 3: Existing Skill Integration Points

### 3.1 `/sagerstack:code-planning`

**Purpose**: Mandatory planning workflow before any code. Produces `docs/code_context.md`.

**Workflow**: Question-driven discovery -> Preference lookup -> Iterative milestone/phase proposal -> E2E test definition -> Generate code_context.md

**Key Outputs**:
- Milestones (week-sized deliverables) containing Phases (1-2 day concrete steps)
- E2E Test definitions (Preconditions, Steps, Expected Outcome)
- Architecture decisions and technical choices

**Integration with Planner Skill**:
- The planner skill operates at a DIFFERENT level of abstraction. Code-planning produces `docs/code_context.md` with milestones/phases. The planner skill produces SDLC artifacts (MVP, Epic, User Story, Tech Research, Implementation Plan, Critical Analysis).
- **Non-overlapping**: code-planning is for when a user says "let's build X" and needs milestone/phase breakdown. The planner skill is for when SDLC artifacts need to be generated from requirements.
- **Potential conflict**: Both could be triggered by "new feature" or "let's build". The CLAUDE.md routing table should disambiguate: code-planning for direct-to-code planning, planner for full SDLC artifact generation.

**Handoff points**:
- code-planning's `docs/code_context.md` could serve as INPUT to the planner skill (providing milestones/phases as context for epic/story breakdown)
- OR planner skill could REPLACE code-planning for projects using the full SDLC flow

### 3.2 `/sagerstack:software-engineering`

**Purpose**: Python code architecture with Vertical Slice + DDD and Clean Architecture.

**Key Standards Enforced**:
- Vertical Slice structure: Feature slices own full stack (domain/application/infrastructure/api)
- CamelCase naming convention everywhere (non-standard for Python but enforced)
- Strict domain purity: No infrastructure imports in domain layer
- TDD mandatory: Red-green-refactor, 90%+ coverage
- Custom exceptions + Result pattern
- Structured logging format
- Constructor dependency injection

**Integration with Builder Skill**:
- The builder skill's Software Developer agent MUST invoke/follow software-engineering standards during implementation.
- The skill's architecture patterns (Vertical Slice + DDD) should be embedded in the Software Developer agent's system prompt.
- CamelCase enforcement, domain purity rules, and TDD requirements flow from this skill into the builder's quality gates.
- Coverage threshold: software-engineering says 90%, code-quality-standards.md says 95%. Need reconciliation (likely 95% as the higher bar).

**Handoff points**:
- Builder skill's Software Developer agent operates WITHIN software-engineering constraints
- Code QA agent validates AGAINST software-engineering standards

### 3.3 `/sagerstack:local-testing`

**Purpose**: Testing infrastructure, local environment simulation, Docker-first execution.

**Key Standards Enforced**:
- Docker-first local execution with LocalStack for AWS services
- Environment file management: `.env.example` (committed), `.env.local` (gitignored), `tests/.env.test` (test config)
- Local deployment scripts (scripts/local/)
- Horizontal test structure (tests/unit/, tests/integration/, tests/e2e/)
- Docker Compose for multi-container orchestration
- Minikube for Kubernetes local testing

**Integration with Builder Skill**:
- The builder skill's Software Developer agent should set up local testing infrastructure per this skill's standards.
- Docker Compose files, .env file management, and local deployment scripts follow this skill.
- The Code QA agent's quality checks include Docker Build verification (Check 6 in quality-checks.md).

**Handoff points**:
- Builder skill references local-testing for Docker setup, env file management, test infrastructure
- Quality check pipeline (quality-checks.md) integrates with local-testing's Docker requirements

### 3.4 Skill Invocation Chain (Updated for New Skills)

Current chain from CLAUDE.md:
```
code-planning -> software-engineering -> local-testing -> deploy-aws
```

Proposed updated chain with new skills:
```
Option A (Full SDLC):
  planner -> builder
  (planner internally uses: code-planning concepts for milestone/phase planning)
  (builder internally uses: software-engineering, local-testing standards)

Option B (Direct Implementation):
  code-planning -> software-engineering -> local-testing -> deploy-aws
  (unchanged, for projects not using full SDLC)
```

The two chains should coexist. CLAUDE.md routing decides which path based on user intent.

---

## Section 4: Handoff Formats

### 4.1 User -> Planner Skill

**Input**: User provides one of:
- Feature request (natural language)
- Product requirements document
- Existing MVP/Epic/User Story artifacts to refine

**Expected format**: Free-form text. Planner skill's BA agent asks clarifying questions.

### 4.2 Planner Skill Internal Handoffs

#### BA Agent -> Researcher Agent

**Artifact**: User Story (draft) with FR/TR/AC tables
**Format**: Markdown file following `user-story-artifact.md` template
**Key fields for researcher**: Technical Guidance section, FR descriptions mentioning external APIs/services, TR performance targets
**Trigger**: User Story has Technical Guidance with `[INSERT_USER_INPUT]` values filled in

#### Researcher Agent -> Solution Architect Agent

**Artifact**: Tech Research document
**Format**: Markdown file following `tech-research.md` template
**Key fields for architect**: API Research section (endpoints, payloads, response structures, field extraction mapping), Solution Options Analysis, Architecture Recommendations
**Trigger**: Tech Research status = Final

#### Solution Architect Agent -> Critical Analyst Agent

**Artifact**: Implementation Plan
**Format**: Markdown file following `implementation-plan.md` template
**Key fields for critic**: Requirements Coverage Validation table (must show 100% FR/TR/AC coverage), Task structure, API Field Extraction tasks (must reference exact fields from Tech Research)
**Trigger**: Implementation Plan first draft complete

#### Critical Analyst Agent -> Solution Architect Agent (Refinement Loop)

**Artifact**: Critical Analysis
**Format**: Markdown file following `critical-analysis.md` template
**Key fields for architect**: Critical Issues (Blockers), High-Risk Assumptions, Recommendations Priority Matrix ("Must Fix Before Development")
**Trigger**: Critical Analysis identifies issues requiring plan refinement
**Loop**: Max 2 refinement cycles. After 2nd cycle, proceed regardless with documented risks.

### 4.3 Planner Skill -> Builder Skill

**Artifacts passed** (the "handoff bundle"):

1. **User Story** (`US-###-{title}.md`) -- FR/TR/AC definitions, business context
2. **Tech Research** (`US-###-tech-research.md`) -- API specs, field mappings, architecture decisions
3. **Implementation Plan** (`US-###-impl-plan.md`) -- Task list with requirement traceability
4. **Critical Analysis** (`US-###-critical-analysis.md`) -- Reviewed risks and sign-off

**Validation before handoff**:
- Implementation Plan Requirements Coverage shows 100% FR/TR/AC mapping
- Critical Analysis Confidence Level = High (or documented exceptions)
- No unresolved Critical Issues in Critical Analysis
- All API Field Extraction tasks reference exact field paths from Tech Research

### 4.4 Builder Skill Internal Handoffs

#### Software Developer Agent -> Code QA Agent

**Artifacts**:
- Updated Implementation Plan with `[x]` markers on completed tasks
- Developer Log (Sections 1-2 at minimum)
- Code changes on feature branch
- Test results (pytest output)

**Format**: Code QA agent reads implementation plan, developer log, and source code directly. No special handoff format needed -- Code QA operates as zero-trust validator.

#### Code QA Agent -> Software Developer Agent (Remediation)

**Artifact**: QA Failure Report
**Format**: Message via SendMessage containing:
- Specific failures (file, line, issue type)
- Affected tasks (mapped back to implementation plan task IDs)
- Required fixes (actionable remediation steps)

**Trigger**: QA validation detects failures in coverage, type checking, security, or AC validation.

### 4.5 Builder Skill -> User

**Output**: Completed user story implementation
- Feature branch with all code changes
- All quality checks passed (9-check pipeline)
- Implementation Plan with all tasks marked `[x]`
- Developer Log documenting implementation
- CHANGELOG entry added

---

## Section 5: Gap Analysis

### 5.1 Gaps Between Old Artifacts and New Architecture

| Gap | Description | Impact | Resolution |
|-----|-------------|--------|------------|
| **No retry automation** | Old architecture had automated retry logic (retry_count, qa_retry_count). Claude Code teams have no built-in retry mechanism. | QA failures or agent crashes require manual leader intervention | Planner/Builder skill instructions should include explicit retry guidance: "If agent reports failure, assess the issue and resend with targeted fix instructions." |
| **No session persistence** | Old architecture saved `.dev-session-*.json` checkpoints. Claude Code teams rely on TaskList persistence only. | Mid-session crashes could lose context not captured in TaskList | Mitigation: Ensure all progress is tracked in TaskList tasks (not just agent memory). Implementation Plan `[x]` markers serve as secondary checkpoint. |
| **No batch planning** | Old architecture's batching algorithm (max 5 tasks, sequential order) is dropped. | Tasks must be individually managed through TaskList | Not a real gap -- TaskList with dependencies replaces batching. Builder skill leader creates tasks from implementation plan and assigns them. |
| **Coverage threshold discrepancy** | software-engineering says 90%, code-quality-standards says 95%, quality-checks says 95% | Inconsistent quality gate | Reconcile to single value. Recommend: 95% as the production standard, documented in builder skill. |
| **No orchestrator agent definition** | Old architecture had `/dev command` as orchestrator. New architecture needs skill-level orchestration. | Leader agent behavior must be defined in skill instructions | Each skill (planner, builder) needs a leader workflow defined in its SKILL.md that specifies how to create team, create tasks, assign work, and handle results. |
| **Agent tool restrictions unclear** | Old code-qa was explicitly read-only (Read, Bash, Grep, Glob). Claude Code agent types need explicit tool specification. | Code QA agent could accidentally modify code | Builder skill must specify Code QA agent with read-only subagent_type or explicit tool restrictions. |
| **No developer log v2 adaptation** | Developer log assumes py-developer/dev-supervised split. New architecture has single Software Developer agent. | Log format references wrong agent names | Adapt developer-log.md: Single agent writes all sections. Remove "dev-supervised appends" language. |

### 5.2 Gaps Between Existing Skills and New Skills

| Gap | Description | Resolution |
|-----|-------------|------------|
| **code-planning vs planner scope overlap** | Both could be triggered by "new feature". code-planning produces milestones/phases; planner produces SDLC artifacts. | CLAUDE.md routing: "let's build" + direct implementation -> code-planning. "create MVP/epic/story" or full SDLC -> planner. |
| **software-engineering embedded in builder** | Builder skill's Software Developer must follow software-engineering standards, but they are a separate skill. | Builder skill instructions should explicitly state: "Follow all standards from /sagerstack:software-engineering". Reference, don't duplicate. |
| **local-testing embedded in builder** | Builder's Docker/env setup must follow local-testing standards. | Same approach: reference, don't duplicate. Builder skill instructions reference local-testing for Docker setup and env management. |
| **deploy-aws not in scope** | deploy-aws is invoked after builder completes, not part of either new skill. | Document in handoff: "After builder completes, invoke /sagerstack:deploy-aws if infrastructure needed." |

### 5.3 Missing Capabilities

| Missing Capability | Description | Recommendation |
|--------------------|-------------|----------------|
| **Planner skill leader workflow** | No existing artifact defines how the planner team leader orchestrates BA, Researcher, Solution Architect, and Critical Analyst | Design a leader workflow in planner SKILL.md: sequential pipeline BA -> Researcher -> Architect -> Critic -> (refinement loop) -> handoff |
| **Builder skill leader workflow** | No existing artifact defines how the builder team leader orchestrates Software Developer and Code QA | Design a leader workflow in builder SKILL.md: Implementation Plan -> task creation -> assign Software Developer -> on completion, assign Code QA -> handle results |
| **Parallel agent execution guidance** | Old architecture was strictly sequential. Claude Code supports parallel Task spawning. | Planner: BA and Researcher could work in parallel if user story and tech research have no dependencies. Builder: Software Developer and Code QA are inherently sequential (QA validates developer output). |
| **Inter-skill communication** | No mechanism for planner skill to directly invoke builder skill | Planner skill produces artifacts in the file system. Builder skill reads them. File-based handoff is sufficient. CLAUDE.md invocation chain manages the sequence. |

---

## Section 6: Recommendations

### 6.1 Planner Skill Design Recommendations

1. **Leader Agent Workflow**: Define a sequential pipeline in SKILL.md:
   - Step 1: Create team with `TeamCreate`
   - Step 2: Spawn BA agent, create user story (or refine existing)
   - Step 3: Spawn Researcher agent after user story is finalized -- produce tech research
   - Step 4: Spawn Solution Architect agent after tech research is final -- produce implementation plan
   - Step 5: Spawn Critical Analyst agent after impl plan draft -- produce critical analysis
   - Step 6: If critical issues found, send refinement feedback to Solution Architect (max 2 cycles)
   - Step 7: Handoff bundle complete, shut down team

2. **Agent Definitions**: Each agent should have specific instructions embedded in the Task prompt:
   - BA Agent: Follow `user-story-artifact.md` template. Use `ai-complexity-scoring-framework.md` for scoring. Follow `cross-agent-validation.md` BA section rules.
   - Researcher Agent: Follow `tech-research.md` template. Execute API research with real calls (NO_MOCK_DATA_POLICY). Use web search for documentation.
   - Solution Architect Agent: Follow `implementation-plan.md` template. Use `cross-agent-validation.md` architect section rules. Apply `cross-story-integration-pattern.md` for dependency analysis. Reference `artifacts-file-mgt.md` for file placement.
   - Critical Analyst Agent: Follow `critical-analysis.md` template. Use "No Issues" template when plan is comprehensive. Track refinement iterations.

3. **Artifact References**: Planner SKILL.md should list all KEEP/ADAPT artifacts as reference documents available to agents. Store adapted versions in the skill's `references/` directory.

4. **CLAUDE.md Routing**: Add planner to the pattern-skill mapping:
   - Patterns: "create mvp", "create epic", "create story", "create plan", "sdlc", "requirements"
   - Action: Invoke planner FIRST, then builder for implementation

### 6.2 Builder Skill Design Recommendations

1. **Leader Agent Workflow**: Define an implementation pipeline in SKILL.md:
   - Step 1: Read handoff bundle (user story, tech research, impl plan, critical analysis)
   - Step 2: Create team with `TeamCreate`
   - Step 3: Create tasks in TaskList from implementation plan (one task per parent task `[X.0]`)
   - Step 4: Assign tasks to Software Developer agent (sequentially or in small groups)
   - Step 5: After Software Developer completes a batch, assign Code QA agent for validation
   - Step 6: If QA fails, send remediation instructions to Software Developer (max 2 cycles per batch)
   - Step 7: When all tasks complete and QA passes, shut down team

2. **Agent Definitions**:
   - Software Developer Agent (full tool access): Follow `software-engineering` skill standards. Apply `ddd-patterns.md` patterns. Follow `code-quality-standards.md`. Use `project-structure.md` for directory layout. TDD mandatory. Write developer log per adapted `developer-log.md`.
   - Code QA Agent (read-only tools): Follow `quality-checks.md` pipeline. Zero-trust validation (re-run all tests independently). Validate against `cross-agent-validation.md` developer section rules. Report failures with specific file/line/task mapping.

3. **Quality Gate Integration**: The 9-check quality pipeline from `quality-checks.md` should be the Code QA agent's checklist. Adapt Check 8 (Git Commit) to use standard git operations instead of the old `/git push` slash command.

4. **Coverage Threshold**: Reconcile to 95% (the higher bar from `code-quality-standards.md` and `quality-checks.md`). Update `software-engineering` skill reference to match.

5. **CLAUDE.md Routing**: Add builder to the pattern-skill mapping:
   - Patterns: "implement story", "build story", "execute plan", "develop US-###"
   - Action: Invoke builder with implementation plan path

### 6.3 Artifact Adaptation Priorities

| Priority | Artifact | Adaptation Required |
|----------|----------|---------------------|
| P1 | `implementation-plan.md` | Remove batch-specific language. Clarify that task markers are for tracking, not batch assignment. |
| P1 | `cross-agent-validation.md` | Rename agents to match new roles. Remove old agent configuration file references. |
| P2 | `developer-log.md` | Merge py-developer/dev-supervised sections. Single agent writes all sections. |
| P2 | `quality-checks.md` | Replace `/git push` with standard git. Remove orchestrator references. |
| P3 | Coverage threshold | Reconcile 90% vs 95% across all references. Standardize on 95%. |

### 6.4 File Organization for New Skills

```
~/.claude/skills/
  sagerstack-planner/
    SKILL.md                          # Leader workflow, agent definitions, routing
    references/
      mvp-artifact.md                 # KEEP as-is
      epic-artifact.md                # KEEP as-is
      user-story-artifact.md          # KEEP as-is
      tech-research.md                # KEEP as-is
      implementation-plan.md          # ADAPTED version
      critical-analysis.md            # KEEP as-is
      ai-complexity-scoring.md        # KEEP as-is
      artifacts-file-mgt.md           # KEEP as-is
      cross-agent-validation.md       # ADAPTED version (planner sections only)
      cross-story-integration.md      # KEEP as-is

  sagerstack-builder/
    SKILL.md                          # Leader workflow, agent definitions, routing
    references/
      implementation-plan.md          # ADAPTED version (shared with planner)
      code-quality-standards.md       # KEEP as-is
      quality-checks.md              # ADAPTED version
      developer-log.md               # ADAPTED version
      changelog-template.md          # KEEP as-is
      ddd-patterns.md                # KEEP as-is
      project-structure.md           # KEEP as-is
      cross-agent-validation.md      # ADAPTED version (builder sections only)
```

### 6.5 Key Design Decisions Still Needed

1. **Should planner and builder be separate skills or one combined skill?** Recommendation: Separate. They have different agent compositions, different trigger patterns, and can be invoked independently. A user might use planner but implement manually, or skip planner and use builder with hand-written artifacts.

2. **Should the planner's BA agent ask the user clarification questions, or should clarifications be gathered before invoking the skill?** Recommendation: BA agent should ask. This is consistent with the user-story-artifact.md template which has Requirements Clarifications section with `[INSERT_USER_INPUT]` placeholders.

3. **How should the builder handle MANUAL tasks?** Recommendation: Builder leader reads implementation plan, identifies `[MANUAL]` markers, creates TaskList items as blocked, and informs the user which manual steps are required before automated tasks can proceed.

4. **Should artifacts be stored in skill references/ or in the project's domain/ folder?** Recommendation: Templates go in skill `references/`. Generated artifacts (actual MVPs, epics, stories) go in the project's `domain/execution/` per `artifacts-file-mgt.md`.

---

## Appendix: Artifact-to-Agent Mapping

| Artifact | Planner Agent(s) | Builder Agent(s) |
|----------|-------------------|-------------------|
| MVP | BA (creates) | -- |
| Epic | BA (creates) | -- |
| User Story | BA (creates, refines) | Software Developer (reads FR/TR/AC) |
| Tech Research | Researcher (creates) | Software Developer (reads API specs) |
| Implementation Plan | Solution Architect (creates) | Software Developer (executes tasks), Code QA (validates completion) |
| Critical Analysis | Critical Analyst (creates) | -- |
| Code Quality Standards | -- | Software Developer (follows), Code QA (validates) |
| Quality Checks | -- | Code QA (executes pipeline) |
| Developer Log | -- | Software Developer (writes) |
| Changelog | -- | Software Developer (creates per story) |
| DDD Patterns | Solution Architect (references) | Software Developer (applies) |
| Project Structure | Solution Architect (references) | Software Developer (applies) |
| Cross-Agent Validation | All planner agents (validation rules) | All builder agents (validation rules) |
| AI Complexity Scoring | BA (scores), Solution Architect (re-scores) | -- |
| Artifacts File Management | All planner agents (file placement) | -- |
| Cross-Story Integration | Solution Architect (applies pattern) | -- |
