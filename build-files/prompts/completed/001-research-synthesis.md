<objective>
Extract valuable specifications from the old agent-teams design artifacts and synthesize them with Claude Code's current team/subagent capabilities to produce a research document that informs the design of two new Claude Code skills: /sagerstack:planner and /sagerstack:builder.
</objective>

<context>
This project is redesigning an AI-powered SDLC framework. The old design used a custom three-agent architecture (orchestrator, py-developer, code-qa). The new design leverages Claude Code's native capabilities: TeamCreate, Task tool with subagent_type, SendMessage, parallel task execution, and shared task lists.

The old design artifacts are in `./artifacts/` (21 files). Three existing Claude Code skills will integrate with the new workflow:
- `/sagerstack:code-planning` — Produces `docs/project-context.md` with Milestones and Phases
- `/sagerstack:software-engineering` — Python architecture (Vertical Slice + DDD, TDD, 90% coverage, CamelCase)
- `/sagerstack:local-testing` — Docker-first local execution, pytest fixtures, deployment scripts

Two new skills will be created:
1. `/sagerstack:planner` — Agent team (BA, Researcher, Solution Architect, Critical Analyst) that takes phases from project-context.md and produces epics, user stories, and implementation plans
2. `/sagerstack:builder` — Agent team (Software Developer, Code QA) that executes implementation plans and validates against acceptance criteria + UAT
</context>

<research_tasks>

## Task 1: Extract Specifications from Old Artifacts

Read every file in `./artifacts/` thoroughly. For each file, extract:

1. **Artifact templates** — The structure/format of MVPs, Epics, User Stories, Tech Research, Implementation Plans. Capture the exact fields, sections, and formatting patterns that provide traceability.

2. **Quality standards** — From `code-quality-standards.md`, `quality-checks.md`: SOLID principles, data policies (NO_MOCK_DATA, NO_HARDCODING, NO_ARTIFICIAL_LIMITS, COMPLETE_API_DATA, FILTER_BEFORE_LIMIT), coverage requirements, naming conventions.

3. **Process patterns** — From `ai-sdlc.md`, `process-implementation-plan.md`: Batch processing logic, checkpoint management, retry/recovery patterns, dual-source analysis, task completion criteria.

4. **Validation patterns** — From `critical-analysis.md`, `cross-agent-validation.md`: How independent validation works, what gets checked, remediation workflows.

5. **Complexity scoring** — From `ai-complexity-scoring-framework.md`: How to assess task complexity for batching decisions.

6. **Requirement traceability** — From `user-story-artifact.md`, `implementation-plan.md`: How FR/TR/AC map to tasks, how tasks map to tests, the complete traceability chain.

For each extraction, note:
- What to KEEP (valuable for the new design)
- What to ADAPT (good concept but needs updating for Claude Code teams)
- What to DROP (replaced by Claude Code native capabilities)

## Task 2: Study Claude Code Team Capabilities

Research Claude Code's current agent team features by examining:
- How TeamCreate works (team config, task lists, member discovery)
- How Task tool spawns subagents (subagent_type options, available tools per type)
- How SendMessage enables agent communication (DM, broadcast, shutdown)
- How TaskCreate/TaskList/TaskUpdate coordinate shared work
- How parallel task execution works (multiple Task calls in single message)
- Permission modes for spawned agents (plan, acceptEdits, bypassPermissions)

Document the capabilities and constraints that affect workflow design.

## Task 3: Study Existing Skills Integration Points

Read these three skill files and identify integration points:

1. `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-planning/SKILL.md`
   - What does project-context.md contain? (milestones, phases, E2E tests)
   - What format are phases in? (fields: Delivers, Definition of Done, Success Criteria)
   - Where does code-planning END and planner should BEGIN?

2. `/Users/sagarpratapsingh/.claude/skills/sagerstack-software-engineering/SKILL.md`
   - What patterns must the Software Developer agent follow?
   - What verification checklist applies after coding?
   - How does TDD workflow integrate with batch task execution?

3. `/Users/sagarpratapsingh/.claude/skills/sagerstack-local-testing/SKILL.md`
   - What testing infrastructure does the Developer agent need to set up?
   - How do .env files, Docker Compose, and deployment scripts work?
   - What verification steps confirm local environment is working?

</research_tasks>

<output>
Save the research synthesis to: `./research/agent-teams-synthesis.md`

Structure the output as:

```markdown
# Agent Teams Research Synthesis
Date: [today]

## 1. Specifications to Keep (from old artifacts)

### Artifact Templates
[Extracted templates with field definitions]

### Quality Standards
[Extracted standards and policies]

### Process Patterns
[Batch processing, checkpoints, retry logic]

### Validation Patterns
[Independent validation approach]

### Complexity Scoring
[Assessment framework]

### Requirement Traceability
[FR/TR/AC → Task → Test chain]

## 2. Specifications to Adapt

[Concepts that need updating for Claude Code teams, with notes on what changes]

## 3. Specifications to Drop

[Things replaced by Claude Code native capabilities, with reasoning]

## 4. Claude Code Team Capabilities

### TeamCreate & Configuration
[How teams work]

### Task Tool & Subagents
[Available agent types, tools, constraints]

### Communication Protocol
[SendMessage patterns]

### Task Coordination
[Shared task list patterns]

### Parallel Execution
[How to parallelize work]

## 5. Skill Integration Points

### code-planning → planner handoff
[What planner receives, format, fields]

### planner → builder handoff
[What builder receives: impl plans, user stories with AC]

### software-engineering integration
[How Developer agent uses the skill]

### local-testing integration
[How Developer agent sets up test infrastructure]

## 6. Key Design Constraints

[Constraints that must be respected in the architecture design]
```
</output>

<verification>
Before completing, verify:
- [ ] All 21 artifact files read and analyzed
- [ ] Clear KEEP/ADAPT/DROP categorization for each specification
- [ ] Claude Code team capabilities documented with constraints
- [ ] All three existing skills analyzed for integration points
- [ ] Handoff formats defined (code-planning → planner → builder)
- [ ] No speculation — only documented capabilities and explicit artifact content
</verification>
