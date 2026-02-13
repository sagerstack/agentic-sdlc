<objective>
Design the complete workflow architecture for two Claude Code agent team skills — /sagerstack:planner and /sagerstack:builder — using the research synthesis as input. Produce a detailed architecture document that serves as the blueprint for building both skills and a supporting code-qa skill.
</objective>

<context>
Read `./research/agent-teams-synthesis.md` first. It contains extracted specifications from the old agent-teams design, Claude Code team capabilities, and integration points with existing skills.

The architecture must define two independent agent teams that form a pipeline:

```
/sagerstack:code-planning (existing, user-driven)
    → docs/code_context.md (Milestones + Phases)
        → /sagerstack:planner (Team 1: plans one phase at a time)
            → docs/phases/phase-N/ (epic, user stories, impl plans)
                → /sagerstack:builder (Team 2: executes impl plans)
                    → Implemented + validated code
```

User preferences that MUST be respected:
- Zero to low cost implementation plans preferred. Costs must be flagged to user.
- Configurable execution: one phase at a time (default) or auto-chain with QA gates (--continue flag)
- User confirms at: proposal stage, user story breakdown, and technical direction
- All configuration from .env files, no hardcoded values
- CamelCase naming, Vertical Slice + DDD, 90%+ test coverage, TDD
- Git workflow: feature branches + PRs, never push to main directly
</context>

<design_requirements>

## 1. Planner Team Architecture

Design the /sagerstack:planner agent team with 4 members:

### Team Lead (Orchestrator)
- Manages workflow progression through the phase
- Coordinates between team members
- Manages user Q&A sessions (uses AskUserQuestion)
- Tracks phase progress via shared task list

### Researcher
- Investigates the phase requirements (domain knowledge, technical landscape)
- Uses WebSearch, WebFetch, codebase exploration
- Produces research findings for BA and Solution Architect to consume
- subagent_type: general-purpose or Explore (decide which is better)

### Business Analyst (BA)
- Reads phase from code_context.md
- Generates proposals for user (multiple options when appropriate)
- Creates epic document (phase-level) with user stories
- Each user story has: FR (Functional Requirements), TR (Technical Requirements), AC (Acceptance Criteria)
- Uses artifact templates extracted from old design (adapted for new format)
- subagent_type: general-purpose

### Solution Architect
- Generates implementation plans for each user story AFTER stories are confirmed
- Conducts Q&A with user for technical direction before researching
- Flags any cost implications to user
- Preference for zero/low cost solutions
- Researches technical approaches (libraries, patterns, APIs)
- subagent_type: general-purpose

### Critical Analyst
- Reviews IMPLEMENTATION PLANS (not user stories) AFTER they are generated
- Validates: (a) alignment with user story requirements (FR/TR/AC), (b) industry best practices
- Flags gaps, risks, better alternatives
- If issues found → Solution Architect revises
- subagent_type: general-purpose

### Planner Workflow Sequence
Define the exact step-by-step workflow:
1. Team Lead reads phase from code_context.md
2. Team Lead assigns research task to Researcher
3. Researcher investigates → returns findings
4. Team Lead assigns story creation to BA (with research findings)
5. BA generates proposals → Team Lead presents to user via Q&A
6. User confirms direction
7. BA generates epic + user stories
8. Team Lead presents story breakdown to user for confirmation
9. User confirms stories
10. Team Lead assigns impl plan generation to Solution Architect
11. Solution Architect Q&A with user (via Team Lead) for technical direction + cost flags
12. Solution Architect researches and generates impl plans per user story
13. Team Lead assigns review to Critical Analyst
14. Critical Analyst reviews impl plans against user stories + best practices
15. If issues → Solution Architect revises (loop back to step 12)
16. If approved → Phase planning complete, artifacts saved

### Planner Artifacts (define exact formats)
Design the artifact formats:
- `docs/phases/phase-N.M/epic.md` — Phase-level epic document
- `docs/phases/phase-N.M/stories/story-N.md` — Individual user story with FR/TR/AC
- `docs/phases/phase-N.M/plans/story-N-plan.md` — Implementation plan per user story
- `docs/phases/phase-N.M/research/findings.md` — Research output

### Phase Management
The BA should support phase operations:
- Insert new phase
- Remove existing phase
- Reorder phases
Define how these operations modify code_context.md.

## 2. Builder Team Architecture

Design the /sagerstack:builder agent team with 2 members:

### Team Lead (Orchestrator)
- Reads implementation plans from planner output
- Assigns tasks to Developer and QA
- Manages execution flow and retry logic
- Configurable: single phase or auto-chain with QA gates

### Software Developer
- Executes implementation plan tasks
- MUST use /sagerstack:software-engineering patterns (Vertical Slice + DDD, TDD, CamelCase)
- MUST use /sagerstack:local-testing for test infrastructure
- Works through tasks sequentially within a user story
- Commits after each logical unit of work
- subagent_type: general-purpose

### Code QA
- Uses the new code-qa skill (to be created)
- Validates against acceptance criteria from user story
- Runs flexible UAT: reads project config to determine launch method (docker-compose or local process)
- Reports: pass/fail per AC, coverage metrics, UAT results
- If fail → Developer remediation loop (targeted, not full retry)
- subagent_type: general-purpose

### Builder Workflow Sequence
1. Team Lead reads impl plan for current user story
2. Team Lead creates task list from impl plan tasks
3. Team Lead assigns tasks to Software Developer
4. Developer implements using TDD (red-green-refactor per task)
5. Developer commits and reports completion
6. After all tasks complete → Team Lead assigns validation to Code QA
7. Code QA runs tests, checks coverage, validates AC, runs UAT
8. If QA fails → Team Lead creates remediation tasks, assigns back to Developer
9. Developer fixes → Code QA re-validates (max 2 retries before escalation)
10. If QA passes → User story marked complete
11. Move to next user story in phase
12. After all stories → Phase complete
13. If --continue flag → move to next phase (back to planner if needed)

### Builder Artifacts
- `docs/phases/phase-N.M/qa/story-N-qa-report.md` — QA validation report per user story
- Developer log entries in implementation plan (task completion markers)

## 3. Code QA Skill Design

Design the code-qa skill that the Code QA agent uses:

### Acceptance Criteria Validation
- Parse AC from user story
- Map each AC to test scenarios
- Execute tests and report pass/fail per AC
- Report coverage metrics

### Flexible UAT
- Read project configuration to determine launch method
- If docker-compose.yml exists → launch via docker-compose, test via HTTP
- If local process config exists → start app locally, test via HTTP
- Run UAT scenarios derived from AC
- Capture results with evidence (HTTP responses, log output)

### QA Report Format
Define the exact report structure: summary, AC results, coverage, UAT results, recommendations.

## 4. Communication Protocol

Define how agents communicate:
- Team Lead ↔ User: via AskUserQuestion (for Q&A sessions)
- Team Lead → Members: via SendMessage + TaskUpdate (task assignment)
- Members → Team Lead: via SendMessage (status reports, findings, questions)
- Cross-team: planner artifacts on disk → builder reads them (no direct communication)

## 5. Error Handling & Recovery

Define error handling for:
- Agent crash mid-task (checkpoint/resume strategy)
- QA validation failure (remediation loop with max retries)
- User rejects proposal (BA revises)
- Critical Analyst rejects impl plan (Solution Architect revises)
- Phase management conflicts (concurrent modifications)

## 6. Configuration

Define what's configurable:
- Execution mode: single phase vs auto-chain (--continue flag)
- Max QA retries before escalation (default: 2)
- Artifact output directory (default: docs/phases/)
- Whether to auto-commit after each task

</design_requirements>

<output>
Save the architecture document to: `./design/agent-teams-architecture.md`

Structure as:

```markdown
# Agent Teams Workflow Architecture
Date: [today]

## Overview
[High-level diagram and description]

## 1. Planner Team (/sagerstack:planner)

### Team Configuration
[TeamCreate config, member roles, subagent types]

### Workflow
[Step-by-step sequence with decision points]

### Artifact Formats
[Templates for each artifact type]

### Phase Management
[Insert, remove, reorder operations]

### User Interaction Points
[When and how user is consulted]

## 2. Builder Team (/sagerstack:builder)

### Team Configuration
[TeamCreate config, member roles, subagent types]

### Workflow
[Step-by-step sequence with QA gates]

### Execution Modes
[Single phase vs auto-chain]

### Remediation Loop
[Failure handling, max retries, escalation]

## 3. Code QA Skill

### AC Validation
[How acceptance criteria are validated]

### Flexible UAT
[Launch detection, test execution, evidence capture]

### Report Format
[QA report template]

## 4. Communication Protocol
[Inter-agent communication patterns]

## 5. Error Handling & Recovery
[Failure modes and recovery strategies]

## 6. Configuration
[All configurable parameters with defaults]

## 7. Artifact Directory Structure
[Complete file/folder layout]

## 8. Integration Map
[How skills connect: code-planning → planner → builder]
```
</output>

<verification>
Before completing, verify:
- [ ] Both team architectures fully defined with member roles and subagent types
- [ ] Workflow sequences are complete with no ambiguous steps
- [ ] All artifact formats have concrete templates (not just descriptions)
- [ ] User interaction points clearly defined (when Q&A happens)
- [ ] Error handling covers all failure modes
- [ ] Critical Analyst reviews impl plans (not user stories) AFTER generation
- [ ] Solution Architect flags costs and prefers zero/low cost options
- [ ] Configurable execution mode documented (single phase vs --continue)
- [ ] Integration with all three existing skills is explicit
- [ ] No Claude Code capability assumptions — only documented features used
</verification>
