<objective>
Create the /sagerstack:planner Claude Code skill (SKILL.md) that spawns a 4-member agent team to plan one phase at a time. The team researches the phase, generates an epic with user stories, creates implementation plans, and critically reviews them — all with user confirmation at key decision points.
</objective>

<context>
Read these files first:
- `./design/agent-teams-architecture.md` — Full workflow architecture (Section 1: Planner Team)
- `./research/agent-teams-synthesis.md` — Extracted specs and artifact templates from old design

This skill is invoked directly by the user via `/sagerstack:planner`. It receives its input from `docs/code_context.md` (produced by `/sagerstack:code-planning`), which contains Milestones and Phases.

The skill spawns a Claude Code agent team using TeamCreate with 4 members:
1. **Researcher** — Investigates phase requirements
2. **Business Analyst (BA)** — Creates epics and user stories
3. **Solution Architect** — Generates implementation plans with cost awareness
4. **Critical Analyst** — Reviews implementation plans against user stories + best practices

Existing skills referenced:
- `/sagerstack:code-planning` produces `docs/code_context.md` (milestones + phases with: Delivers, Definition of Done, Success Criteria)
- `/sagerstack:software-engineering` defines architecture patterns the Solution Architect should align with
- `/sagerstack:local-testing` defines infrastructure patterns the Solution Architect should consider

User preferences:
- Zero to low cost implementation plans preferred. Flag costs to user.
- User confirms at: proposal stage, story breakdown, technical direction
- BA can manage phases: insert, remove, reorder
- Critical Analyst reviews IMPLEMENTATION PLANS (not stories), AFTER generation
- Validates (a) alignment with user story FR/TR/AC and (b) industry best practices
</context>

<requirements>

## Skill Structure

Create the skill at: `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/SKILL.md`

Follow the exact SKILL.md structure of existing sagerstack skills.

## Essential Principles

### 1. Phase-at-a-Time Processing
- Read `docs/code_context.md` to identify available phases
- Present phase list to user, let them select which to plan
- Process ONE phase completely before offering the next
- Each phase produces a complete artifact set (epic, stories, impl plans)

### 2. Research-First Approach
- Researcher investigates BEFORE BA creates anything
- Research covers: domain knowledge, technical landscape, existing patterns in codebase
- Research findings are shared with BA and Solution Architect

### 3. User Q&A at Decision Points
The Team Lead (orchestrator) manages Q&A sessions with the user using AskUserQuestion:
- **Proposal stage**: BA generates 2-3 approach proposals. User selects direction.
- **Story breakdown**: BA presents user stories with FR/TR/AC. User confirms or adjusts.
- **Technical direction**: Solution Architect presents options with cost implications. User confirms approach.

### 4. Cost-Conscious Architecture
Solution Architect MUST:
- Prefer zero-cost solutions (open source, free tiers, local-only)
- When cost is unavoidable, flag it explicitly during Q&A
- Present cost comparison: free alternative vs paid option with trade-offs
- Never assume user will accept paid services

### 5. Critical Review After Implementation Plans
Critical Analyst reviews AFTER Solution Architect generates impl plans:
- (a) Does the impl plan cover ALL FR, TR, and AC from the user story?
- (b) Does it follow industry best practices for the technology choices?
- (c) Are there better alternatives the Solution Architect missed?
- If issues found → Solution Architect revises (max 2 revision cycles)

### 6. Phase Management
BA supports these operations on `docs/code_context.md`:
- **Insert phase**: Add a new phase between existing ones (renumber as needed)
- **Remove phase**: Delete a phase and renumber
- **Reorder phases**: Change phase sequence with dependency check

## Team Configuration

```
Team name: planner-phase-{N.M}
Members:
  - name: researcher
    subagent_type: general-purpose
    role: Phase requirements investigation
    tools: WebSearch, WebFetch, Read, Grep, Glob, Bash

  - name: business-analyst
    subagent_type: general-purpose
    role: Epic and user story generation, phase management
    tools: Read, Write, Edit, Grep, Glob

  - name: solution-architect
    subagent_type: general-purpose
    role: Implementation plan generation with cost awareness
    tools: Read, Write, Edit, WebSearch, WebFetch, Grep, Glob, Bash

  - name: critical-analyst
    subagent_type: general-purpose
    role: Implementation plan review against stories + best practices
    tools: Read, Grep, Glob, WebSearch
```

## Workflow Sequence

### Step 1: Intake
- Read `docs/code_context.md`
- List all phases with their status (planned, in-progress, complete)
- Ask user which phase to plan (or offer next unplanned phase)

### Step 2: Team Setup
- TeamCreate with team name based on phase
- Create task list for the phase

### Step 3: Research
- Assign Researcher to investigate the phase
- Researcher reads: phase description from code_context.md, existing codebase, external resources
- Researcher produces: `docs/phases/phase-N.M/research/findings.md`

### Step 4: Proposal Generation
- Assign BA to generate 2-3 approach proposals (using research findings)
- Team Lead presents proposals to user via AskUserQuestion
- User selects preferred direction

### Step 5: Epic + User Story Generation
- BA generates epic document: `docs/phases/phase-N.M/epic.md`
- BA generates user stories: `docs/phases/phase-N.M/stories/story-1.md`, etc.
- Each story has: Title, Description, FR list, TR list, AC list with specific scenarios
- Team Lead presents story breakdown to user for confirmation
- User confirms or requests adjustments

### Step 6: Technical Direction Q&A
- Solution Architect reviews stories and research findings
- Solution Architect identifies key technical decisions needed
- Team Lead presents options to user via AskUserQuestion (with cost flags)
- User confirms technical direction

### Step 7: Implementation Plan Generation
- Solution Architect generates impl plans per user story
- Each plan at: `docs/phases/phase-N.M/plans/story-N-plan.md`
- Plan contains: numbered tasks, subtasks, test requirements per task
- Aligns with /sagerstack:software-engineering patterns (Vertical Slice + DDD)
- Aligns with /sagerstack:local-testing infrastructure patterns

### Step 8: Critical Review
- Critical Analyst reviews each impl plan
- Checks: (a) covers all FR/TR/AC from story, (b) follows best practices
- Produces review notes
- If issues found → Solution Architect revises (max 2 cycles)
- If approved → impl plans finalized

### Step 9: Completion
- All artifacts saved
- Phase status updated in code_context.md (planned → ready)
- Summary presented to user
- Team shutdown

## Artifact Formats

### Epic Document (docs/phases/phase-N.M/epic.md)
```markdown
# Epic: [Phase Name]
Phase: [N.M]
Milestone: [N]
Date: [date]
Status: planning | ready | in-progress | complete

## Objective
[What this phase delivers and why it matters]

## Scope
### In Scope
- [Capability 1]
- [Capability 2]

### Out of Scope
- [Explicitly excluded]

## User Stories
| ID | Title | Complexity | Status |
|----|-------|-----------|--------|
| 1  | [title] | [low/med/high] | draft | ready | complete |

## Dependencies
- [What must be done before this phase]
- [What this phase enables]

## Success Criteria
[From code_context.md phase definition]
```

### User Story (docs/phases/phase-N.M/stories/story-N.md)
```markdown
# Story [N]: [Title]
Phase: [N.M]
Epic: [epic title]
Complexity: [low | medium | high]
Status: draft | ready | in-progress | complete

## Description
[As a [role], I want [capability] so that [benefit]]

## Functional Requirements
- **FR-1**: [Requirement description]
- **FR-2**: [Requirement description]

## Technical Requirements
- **TR-1**: [Requirement description]
- **TR-2**: [Requirement description]

## Acceptance Criteria
- **AC-1**: Given [precondition], when [action], then [expected outcome]
- **AC-2**: Given [precondition], when [action], then [expected outcome]

## Notes
[Any additional context from research or Q&A]
```

### Implementation Plan (docs/phases/phase-N.M/plans/story-N-plan.md)
```markdown
# Implementation Plan: Story [N] - [Title]
Story: [story file path]
Date: [date]
Status: draft | approved | in-progress | complete

## Technical Approach
[Brief description of how this will be implemented]
[Key library/framework choices with rationale]

## Cost Implications
[None | Description of any costs with alternatives]

## Tasks

### [1.0][FR-1] [Task Title]
- [ ] [1.1] [Subtask description]
  - Test: [test to write first]
- [ ] [1.2] [Subtask description]
  - Test: [test to write first]

### [2.0][FR-2] [Task Title]
- [ ] [2.1] [Subtask description]
  - Test: [test to write first]

### [3.0][TR-1] [Task Title]
- [ ] [3.1] [Subtask description]

## Test Coverage Map
| Requirement | Test Type | Test File |
|-------------|-----------|-----------|
| FR-1 | unit | tests/unit/[slice]/test[Name].py |
| AC-1 | e2e | tests/e2e/test[Scenario].py |

## Architecture Alignment
- Vertical slice: [which slice this belongs to]
- Domain entities: [entities to create/modify]
- Infrastructure: [repos, external services]
```

## Workflows

Create workflow files for:
- `workflows/plan-phase.md` — Main planning workflow (steps 1-9)
- `workflows/manage-phases.md` — Insert, remove, reorder operations
- `workflows/generate-stories.md` — BA story generation process
- `workflows/generate-impl-plan.md` — Solution Architect plan generation
- `workflows/critical-review.md` — Critical Analyst review process

## References

Create reference files for:
- `references/story-patterns.md` — User story writing patterns and anti-patterns
- `references/impl-plan-patterns.md` — Implementation plan best practices
- `references/cost-assessment.md` — How to evaluate and flag costs

</requirements>

<implementation>
Read existing sagerstack skills for structural reference:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-planning/SKILL.md`
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-software-engineering/SKILL.md`
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-local-testing/SKILL.md`

The SKILL.md must be complete and self-contained. When loaded by Claude, it should provide everything needed to:
1. Set up the agent team
2. Run the full planning workflow
3. Generate all artifacts in correct format
4. Manage user Q&A sessions
5. Handle phase management operations

Workflow files and reference files should be created as placeholders with clear TODO descriptions.
</implementation>

<output>
Create these files:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/SKILL.md` — Main skill file (complete)
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/workflows/plan-phase.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/workflows/manage-phases.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/workflows/generate-stories.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/workflows/generate-impl-plan.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/workflows/critical-review.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/references/story-patterns.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/references/impl-plan-patterns.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-planner/references/cost-assessment.md` — Placeholder
</output>

<verification>
Before completing, verify:
- [ ] SKILL.md matches structure of existing sagerstack skills (YAML, XML tags)
- [ ] Team configuration specifies all 4 members with subagent_type and tools
- [ ] Workflow has 9 steps with clear Team Lead orchestration
- [ ] User Q&A happens at 3 points: proposals, story breakdown, technical direction
- [ ] Critical Analyst reviews IMPLEMENTATION PLANS (not stories), AFTER generation
- [ ] Cost flagging is explicit in Solution Architect's workflow
- [ ] Phase management operations defined (insert, remove, reorder)
- [ ] All artifact templates match the formats specified above
- [ ] Input reads from code_context.md (code-planning output)
- [ ] Output artifacts at docs/phases/phase-N.M/ structure
- [ ] Workflow and reference placeholder files created
</verification>
