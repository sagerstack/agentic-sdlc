---
name: sagerstack:code-planning
description: Mandatory planning workflow before writing any code. Use when starting new features, projects, or any implementation work. Ensures alignment on requirements, architecture, and approach before coding begins.
---

<essential_principles>

## How Code Planning Works

This workflow is MANDATORY before writing ANY code. No exceptions.

### 1. Plan Before Code

Every implementation starts with planning:
1. Understand the objective (what and why)
2. Clarify requirements (full lifecycle: pilot → testing → deployment)
3. Confirm architecture decisions
4. Define success criteria
5. THEN write code

### 2. Question-Driven Discovery

Use the question bank in `references/questions.md` to systematically explore:
- Project objectives
- Code structure
- Domain modeling
- Configuration
- Infrastructure
- Testing
- Deployment

### 3. Preference Lookup

For each question:
1. Check if a preference exists in `@persona/` files
2. If yes → present as suggested answer
3. If no → ask user directly
4. Record new preferences via detect+confirm protocol

### 4. Output: project-context.md

Planning produces `docs/project-context.md` in the current project:
- Summary of all decisions made
- Architecture diagram (text-based)
- Implementation plan
- E2E test definitions

</essential_principles>

<intake>
**What would you like to plan?**

1. New project from scratch
2. New feature for existing project
3. Refactor existing code
4. Continue previous planning session

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "new project", "start fresh" | `workflows/plan-new-project.md` |
| 2, "feature", "add", "implement" | `workflows/plan-feature.md` |
| 3, "refactor", "restructure" | `workflows/plan-refactor.md` |
| 4, "continue", "resume" | `workflows/continue-planning.md` |
</routing>

<workflow_steps>

## Planning Process

### Step 1: Load Questions
Read `references/questions.md` for the comprehensive question bank.

### Step 2: Objective Questions
Ask project-specific questions:
- What is the project type?
- Who is the end user?
- What problem does this solve?
- What is the deployment target?

### Step 3: Technical Questions
For each category, check persona files for existing preferences:

| Category | Check File |
|----------|------------|
| Architecture | @persona/values.md (no hardcoded values) |
| Code style | Skill: software-engineering |
| Testing | Skill: build-test-execute |
| Deployment | Skill: deploy-aws |

### Step 4: Confirm Understanding
Before proceeding, summarize:
- What we're building
- Why we're building it
- How we'll build it
- How we'll know it's done

Get explicit confirmation.

### Step 5: Phase Breakdown (Iterative)

**Read `references/phase-patterns.md` before proposing phases.**

Break the implementation into a two-level hierarchy:
1. **Milestones** (larger, ~week-sized) - significant deliverables
2. **Phases** (smaller, 1-2 days) - concrete steps within milestones

Present ONE milestone at a time, then break it into phases. Wait for confirmation before proceeding to the next milestone.

**Key Principles (from phase-patterns.md):**
- First milestone produces something runnable locally
- Each phase is a vertical slice (not horizontal layer)
- Core value delivered before infrastructure concerns (auth, logging, etc.)
- Local deployment mimics cloud deployment 100%
- AWS deployment is a separate, later milestone

#### Step 5a: Select and Adapt Template

Before proposing phases, follow this process:

1. **Identify project type** from Step 2 responses:
   | Project Type | Template in phase-patterns.md |
   |--------------|-------------------------------|
   | New project from scratch | "New Project from Scratch" |
   | New feature for existing project | "New Feature for Existing Project" |
   | Refactor/restructure | "Refactor/Restructure" |
   | Integration with external system | "Integration with External System" |

2. **Read the matching template** in `references/phase-patterns.md`

3. **Adapt the template** based on understanding gathered:
   - Replace generic names with actual domain entities identified
   - Replace "core capability" with the specific value-add identified
   - Adjust infrastructure phases based on deployment target
   - Add/remove phases based on scope

4. **Verify adaptation** against anti-patterns:
   - No "big bang" phases (too much in one phase)
   - No horizontal layer phases (all repos, then all handlers)
   - Infrastructure concerns deferred appropriately
   - First milestone produces runnable local app

#### Step 5b: Iterative Proposal

**Process:**
1. Based on the adapted template, propose Milestone 1 with its phases
2. Wait for user confirmation or edits
3. Only after confirmation, propose next milestone
4. Continue until user confirms all milestones are complete

**For each milestone, include:**

```
**Milestone [N]: [Name]**
- **Delivers**: [Major capability or outcome]
- **Why Now**: [Why this milestone before others]
- **Success Criteria**: [How to verify milestone completion]

**Phases:**

**Phase [N.1]: [Name]**
- **Delivers**: [Specific output]
- **Definition of Done**:
  - [Observable outcome 1]
  - [Observable outcome 2]
- **Success Criteria**:
  - [Verification step 1]
  - [Verification step 2]

**Phase [N.2]: [Name]**
...

Does this look right, or would you like to adjust?
```

**Conversation flow example:**

```
Claude: "Let me suggest Milestone 1 for this implementation:

**Milestone 1: Local Foundation**
- Delivers: Minimal app running locally with core order creation E2E
- Why Now: Need runnable foundation before adding features
- Success Criteria: Can create an order via CLI and see it persisted

**Phases:**

**Phase 1.1: Project Skeleton**
- Delivers: Project structure, dependencies, basic config
- Definition of Done:
  - Poetry project initialized
  - Vertical slice folder structure
  - Basic .env.local setup
- Success Criteria:
  - `poetry install` succeeds
  - Empty app runs without errors

**Phase 1.2: Order Domain Model**
- Delivers: Core Order aggregate with value objects
- Definition of Done:
  - Order entity with ID, customerId, status
  - OrderLine value object
  - Money value object
- Success Criteria:
  - Unit tests pass
  - Domain layer has no infrastructure imports

**Phase 1.3: Repository + Handler**
...

Does this look right, or would you like to adjust?"

User: "Looks good, but add OrderStatus enum to Phase 1.2"

Claude: "Got it. Phase 1.2 updated to include OrderStatus enum.

Milestone 1 confirmed. Here's Milestone 2:

**Milestone 2: Core Capability Complete**
..."
```

**Continue until user says:** "All milestones defined" or similar confirmation.

### Step 6: E2E Test Definition

After phases are confirmed, explicitly define the end-to-end tests that must pass for the project to be complete.

**Process:**
1. Ask: "What end-to-end tests should pass when this project is complete?"
2. Guide thinking toward:
   - User journeys / happy paths
   - Critical error cases
   - Integration points between systems
3. Capture structured test definitions
4. Confirm all E2E tests before proceeding

**For each E2E test, capture:**

```
### E2E Test: [Test Name]

**Preconditions:**
- [System state required before test]
- [Data that must exist]

**Steps:**
1. [Action 1]
2. [Action 2]
3. [Action 3]

**Expected Outcome:**
- [What should happen]
- [What data should be created/modified]
- [What response should be returned]
```

**Guiding questions:**
- "If a user completes the happy path, what should we verify?"
- "What's the most critical failure mode we need to test?"
- "Where does this system integrate with others, and what could go wrong?"

**Continue until user confirms:** "All E2E tests defined" or similar.

### Step 7: Generate project-context.md
Create `docs/project-context.md` with:
```markdown
# Code Context: [Project Name]
Date: [date]

## Objective
[What and why]

## Architecture
[Text diagram and key decisions]

## Implementation Plan

### Milestone 1: [Name]
- **Delivers**: [Major capability]
- **Why Now**: [Ordering rationale]
- **Success Criteria**: [Milestone verification]

#### Phase 1.1: [Name]
- **Delivers**: [Output]
- **Definition of Done**:
  - [Outcome 1]
  - [Outcome 2]
- **Success Criteria**:
  - [Verification 1]
  - [Verification 2]

#### Phase 1.2: [Name]
...

### Milestone 2: [Name]
...

[Continue for all confirmed milestones and phases]

## E2E Tests

### E2E Test: [Test Name]
**Preconditions:**
- [Required state]

**Steps:**
1. [Action]

**Expected Outcome:**
- [Result]

[Continue for all defined E2E tests]

## Decisions Made
[Key choices and rationale]
```

### Step 8: Proceed to Implementation
After planning is complete:
1. Invoke `/sagerstack:software-engineering` for architecture
2. Invoke `/sagerstack:build-test-execute` for TDD implementation
3. Invoke `/sagerstack:deploy-aws` if infrastructure needed

</workflow_steps>

<reference_index>
## References

All in `references/`:

| File | Purpose |
|------|---------|
| questions.md | Comprehensive question bank by category |
| phase-patterns.md | Phase structuring patterns, ordering principles, milestone templates |
</reference_index>

<success_criteria>

Planning is complete when:
- [ ] All project-specific questions answered
- [ ] Technical preferences confirmed or captured
- [ ] User confirmed understanding summary
- [ ] Milestones defined with phases (two-level hierarchy, one milestone at a time)
- [ ] Phase patterns applied (local-first, vertical slices, core value before infrastructure)
- [ ] E2E tests defined for project completion
- [ ] `docs/project-context.md` created with milestones, phases, and E2E tests
- [ ] Ready to proceed with implementation skills

</success_criteria>
