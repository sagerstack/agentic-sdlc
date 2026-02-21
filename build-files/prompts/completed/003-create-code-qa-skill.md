<objective>
Create the /sagerstack:code-qa Claude Code skill (SKILL.md) that the Code QA agent uses within the /sagerstack:builder team. This skill provides acceptance criteria validation, test execution, coverage verification, and flexible UAT (User Acceptance Testing) capabilities.
</objective>

<context>
Read these files first to understand the design and integration requirements:
- `./design/agent-teams-architecture.md` — Full workflow architecture (Section 3: Code QA Skill)
- `./research/agent-teams-synthesis.md` — Extracted specs and validation patterns from old design

This skill is used by the Code QA agent (subagent_type: general-purpose) spawned by the /sagerstack:builder team. It is NOT invoked directly by the user — it is loaded by the Code QA agent to know HOW to validate work.

The skill must integrate with:
- `/sagerstack:software-engineering` — Knows the quality standards (CamelCase, Vertical Slice + DDD, 90% coverage, TDD)
- `/sagerstack:local-testing` — Knows how to launch local environment (Docker Compose, LocalStack)
- User story artifacts at `docs/phases/phase-N.M/stories/story-N.md` — Contains FR/TR/AC to validate against
- Implementation plan at `docs/phases/phase-N.M/plans/story-N-plan.md` — Contains task completion markers

User preferences:
- Acceptance criteria focus (not full zero-trust re-execution)
- Flexible UAT: skill reads project config to determine launch method
- QA report must be actionable — map failures to specific tasks for targeted remediation
- No hardcoded values, CamelCase naming throughout
</context>

<requirements>

## Skill Structure

Create the skill at: `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/SKILL.md`

Follow the exact SKILL.md structure used by existing sagerstack skills:
1. YAML frontmatter (name, description)
2. `<essential_principles>` — Core validation approach
3. `<intake>` — What the QA agent needs to start validation
4. `<routing>` — Route to appropriate workflow
5. `<workflow_steps>` — Detailed validation process
6. `<reference_index>` — Supporting reference files
7. `<verification>` — Post-validation checklist

## Essential Principles

### 1. Acceptance Criteria-Driven Validation
- Parse AC from user story file
- Each AC maps to one or more verification steps
- Report pass/fail PER individual AC (not aggregate)
- Every AC must have a verification result

### 2. Test Execution
- Run the project's test suite: `poetry run pytest --cov`
- Verify coverage meets 90% threshold
- Check that tests follow TDD patterns (test file exists for each implementation file)
- Run specific test categories: unit → integration → e2e

### 3. Code Quality Checks
- Verify CamelCase naming throughout new code
- Verify domain layer has no infrastructure imports
- Verify no hardcoded values (check for magic numbers, string literals that should be config)
- Verify structured logging (no print statements)
- Run linting: `poetry run ruff check`
- Run type checking: `poetry run mypy`

### 4. Flexible UAT
The skill must determine how to launch and test the app:

```
IF docker-compose.yml exists in project root:
    → Launch via: docker-compose up -d
    → Wait for health checks to pass
    → Test via: HTTP requests to container endpoints
    → Cleanup: docker-compose down

ELIF pyproject.toml has [tool.poetry.scripts] or main entry point:
    → Launch via: poetry run [entry-point] &
    → Wait for process to be ready (health endpoint or port open)
    → Test via: HTTP requests to localhost
    → Cleanup: kill process

ELSE:
    → Report: "No launch configuration found. UAT skipped."
    → Suggest how to configure launch method
```

UAT scenarios are derived from acceptance criteria:
- Each AC with observable behavior → UAT scenario
- Execute the scenario against running app
- Capture evidence: HTTP response, status code, response body
- Compare against expected outcome from AC

### 5. Project Memory Integration
The Code QA skill integrates with `/project-memory` to build institutional knowledge:

**During validation (READ):**
- Read `docs/project_notes/bugs.md` — check if known bugs are being reintroduced
- Read `docs/project_notes/decisions.md` — verify code follows established architectural decisions

**After validation (WRITE):**
- Write newly discovered bugs to `docs/project_notes/bugs.md` (date, issue, root cause, solution, prevention)
- Log story completion to `docs/project_notes/issues.md` (date, story ID, status, description)

This creates a compound learning effect: each QA cycle contributes to project memory, making future cycles faster and more thorough.

### 6. Granular Failure Mapping
When validation fails:
- Map each failure to the specific implementation plan task that produced it
- Provide specific remediation guidance (what to fix, where)
- Categorize: test failure, coverage gap, code quality violation, UAT failure
- This enables targeted remediation (Developer fixes specific tasks, not everything)

### 7. QA Report Generation
Output a structured report at `docs/phases/phase-N.M/qa/story-N-qa-report.md`:

```markdown
# QA Report: [Story Title]
Date: [timestamp]
Story: [story file path]
Implementation Plan: [plan file path]
Verdict: PASS | FAIL | PARTIAL

## Acceptance Criteria Results

| AC ID | Description | Status | Evidence |
|-------|-------------|--------|----------|
| AC-1  | [description] | PASS/FAIL | [test name or UAT result] |
| AC-2  | [description] | PASS/FAIL | [test name or UAT result] |

## Test Results

### Unit Tests
- Total: N | Passed: N | Failed: N
- Coverage: N%
- Failed tests: [list with file:line references]

### Integration Tests
- Total: N | Passed: N | Failed: N

### E2E Tests
- Total: N | Passed: N | Failed: N

## Code Quality

| Check | Status | Details |
|-------|--------|---------|
| CamelCase naming | PASS/FAIL | [violations if any] |
| Domain purity | PASS/FAIL | [violations if any] |
| No hardcoded values | PASS/FAIL | [violations if any] |
| Linting (ruff) | PASS/FAIL | [error count] |
| Type checking (mypy) | PASS/FAIL | [error count] |

## UAT Results

| Scenario | Method | Expected | Actual | Status |
|----------|--------|----------|--------|--------|
| [scenario] | [HTTP method + URL] | [expected response] | [actual response] | PASS/FAIL |

## Failure Remediation Map

| Failure | Related Task | Remediation |
|---------|-------------|-------------|
| [failure description] | [task ID from impl plan] | [specific fix guidance] |

## Summary
[Overall assessment and recommendation: approve, remediate, or escalate]
```

## Workflows

Create workflow files for:

### workflows/validate-story.md
The main validation workflow:
1. Read user story (parse FR/TR/AC)
2. Read implementation plan (check task completion markers)
3. Run test suite
4. Check coverage
5. Run code quality checks
6. Run UAT (if launch config available)
7. Generate QA report
8. Return verdict

### workflows/run-uat.md
UAT-specific workflow:
1. Detect launch method
2. Start application
3. Wait for readiness
4. Execute UAT scenarios
5. Capture evidence
6. Stop application
7. Return results

### workflows/remediation-check.md
After developer fixes:
1. Read previous QA report
2. Re-run ONLY failed checks (not full suite)
3. Update report
4. Return updated verdict

## References

Create reference files by extracting and adapting content from old artifacts and existing skills:
- `references/ac-parsing.md` — How to parse acceptance criteria from user story format (extract from `./artifacts/user-story-artifact.md`)
- `references/uat-patterns.md` — Common UAT scenario patterns (CRUD, auth, error handling)
- `references/quality-checklist.md` — Complete quality check reference (extract from `./artifacts/quality-checks.md` and `./artifacts/code-quality-standards.md`)
- `references/cross-agent-validation.md` — Validation methodology (extract from `./artifacts/cross-agent-validation.md`)

IMPORTANT: These reference files should contain the FULL content adapted for the new workflow, not just placeholders. The artifacts folder will be deleted after skills are created — the skills must be self-contained.

</requirements>

<implementation>
Use the `/taches-cc-resources:create-agent-skills` skill pattern to create the SKILL.md. Follow the exact structure of existing sagerstack skills (read the three skills in `/Users/sagarpratapsingh/.claude/skills/` for reference).

The SKILL.md should be self-contained — the Code QA agent loads it and knows exactly what to do without additional context.

Workflow files and reference files should be created as empty placeholders with TODO comments indicating what content goes in each. The SKILL.md is the priority deliverable.
</implementation>

<output>
Create these files:
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/SKILL.md` — Main skill file (complete)
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/workflows/validate-story.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/workflows/run-uat.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/workflows/remediation-check.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/references/ac-parsing.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/references/uat-patterns.md` — Placeholder
- `/Users/sagarpratapsingh/.claude/skills/sagerstack-code-qa/references/quality-checklist.md` — Placeholder
</output>

<verification>
Before completing, verify:
- [ ] SKILL.md follows exact structure of existing sagerstack skills (YAML frontmatter, XML tags)
- [ ] All 6 essential principles are in `<essential_principles>`
- [ ] QA report format is complete with all sections
- [ ] Flexible UAT detection logic covers docker-compose and local process
- [ ] Failure remediation maps failures to specific impl plan tasks
- [ ] AC parsing reads from the user story artifact format
- [ ] No hardcoded values in the skill (paths are relative, thresholds configurable)
- [ ] Workflow files created (even as placeholders)
- [ ] Skill integrates with software-engineering quality standards
</verification>
