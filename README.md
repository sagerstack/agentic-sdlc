# Agentic SDLC Framework

A self-contained software development lifecycle automation framework for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Uses agent teams to plan, build, and verify software epic-by-epic with quality gates at every step.

## About

This is a personalised agentic SDLC framework designed for enterprise settings. It draws on my experience as a software engineer and technical lead to demonstrate how agentic frameworks can be adopted to better suit agile and other established SDLC practices used by engineering teams. Most importantly, this framework reflects the ground reality of being agile and adaptive to the growing and changing requirements in the product roadmap.

The framework operates as skills, agents, and slash commands — all living inside a `.claude/` directory. No packages to install, no runtime dependencies. Copy the `.claude/` directory into any project and start using it.

UPCOMING- Packaging as skills to claude plugin marketplace


## How It Works

The SDLC runs as a four-stage pipeline. Each stage must complete before the next begins.

```
  Stage 1              Stage 2              Stage 3              Stage 4
  Code Planning   -->  Phase Planning  -->  Building        -->  Verification
  (solo)               (4-agent team)       (2-agent team)       (solo)

  /sagerstack:         /sagerstack:         /sagerstack:         /sagerstack:
  code-planning        planner              builder              verify
```

| Stage | What It Does | What It Produces |
|-------|-------------|-----------------|
| **Code Planning** | Interactive session to define objective, milestones, key results, and epics | `docs/project-context.md` |
| **Phase Planning** | 4-agent team plans one epic — research, stories with FR/TR/AC, implementation plans, critical review | `docs/phases/epic-{NNN}-{desc}/` with epic, stories, plans, research |
| **Building** | 2-agent team implements one epic via TDD with automated QA validation | `src/`, `tests/`, QA reports |
| **Verification** | Walks you through acceptance criteria one at a time, records pass/fail | `uat-report.md` |

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Claude API access with **Opus** and **Sonnet** models (agents use both)
- Agent teams feature enabled (see Installation)

## Installation

**1. Clone this repository:**

```bash
git clone https://github.com/sagerstack/agent-teams.git
```

**2. Copy the `.claude/` directory into your target project:**

```bash
cp -r agent-teams/.claude/ /path/to/your-project/.claude/
```

**3. Enable agent teams in your project's `.claude/settings.local.json`:**

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  },
  "teammateMode": "in-process"
}
```

That's it. No packages to install. The framework is entirely Claude Code skills, agents, and slash commands.

## Quick Start

Open Claude Code in your project directory and run the pipeline:

```bash
# Step 1: Plan your project (interactive — defines objective, milestones, epics)
/sagerstack:code-planning

# Step 2: Plan the first epic (spawns 4-agent team)
/sagerstack:planner 001

# Step 3: Build the first epic (spawns 2-agent team with TDD + QA)
/sagerstack:builder 001

# Step 4: Verify the first epic (interactive UAT against acceptance criteria)
/sagerstack:verify 001

# Repeat steps 2-4 for each subsequent epic
/sagerstack:planner 002
/sagerstack:builder 002
/sagerstack:verify 002
```

## Commands Reference

| Command | Description |
|---------|------------|
| `/sagerstack:code-planning` | Interactive project planning session. Produces `docs/project-context.md` |
| `/sagerstack:planner [epic]` | Plan one epic with 4-agent team. Pass epic number or `next` for the next unplanned epic |
| `/sagerstack:planner [epic] --skip-research` | Plan without research phase — for straightforward epics with well-understood requirements |
| `/sagerstack:planner manage` | Manage epic structure — insert, remove, reorder, split, or merge epics |
| `/sagerstack:hotfix [description]` | Abbreviated planning for bug fixes — creates a lightweight single-story epic |
| `/sagerstack:builder [epic]` | Build one epic with 2-agent team (Developer + QA) |
| `/sagerstack:verify [epic]` | Interactive UAT — walk through acceptance criteria one at a time |
| `/sagerstack:verify [epic] resume` | Resume a previous UAT session |

## Pipeline Stages

### Stage 1: Code Planning

**Mode**: Solo interactive session

Defines the project's high-level structure using an OKR (Objectives and Key Results) framework. Asks questions one at a time to build up:

- **Objective** — what the project achieves
- **Milestones** — major delivery checkpoints
- **Key Results** — measurable outcomes per milestone
- **Epics** — units of work that deliver key results
- **Architecture decisions** — tech stack, patterns, constraints

**Output**: `docs/project-context.md` — the single source of truth for all subsequent planning.

### Stage 2: Phase Planning

**Mode**: 4-agent team

Plans ONE epic per invocation. The team:

| Agent | Model | Role |
|-------|-------|------|
| Team Lead | Opus | Orchestrates workflow, presents decisions to you |
| Researcher | Sonnet | Investigates requirements, APIs, codebase |
| Business Analyst | Opus | Creates epic and user stories with FR/TR/AC tables |
| Solution Architect | Opus | Generates task-based implementation plans with 100% requirement coverage |
| Critical Analyst | Opus | Reviews plans for alignment, best practices, and gaps |

**Workflow**: Research &#8594; Story proposals &#8594; *User confirms* &#8594; Stories with FR/TR/AC &#8594; *User confirms* &#8594; Technical Q&A &#8594; Implementation plans &#8594; Critical review (max 2 cycles)

Three mandatory checkpoints where you confirm direction before the team proceeds.

**Output per epic**:
```
docs/phases/epic-{NNN}-{desc}/
  epic.md                                    # Epic overview and story portfolio
  research/findings.md                       # Researcher output
  stories/story-{NNN}-{desc}.md              # User stories with FR/TR/AC tables
  plans/story-{NNN}-{desc}-plan.md           # Task-based implementation plans
  plans/story-{NNN}-{desc}-critical-analysis.md  # Critical review
```

**Flags**:
- `--skip-research` — Skips the Researcher agent and Architect web research. Use for epics where requirements are well understood.

### Stage 3: Building

**Mode**: 2-agent team

Implements ONE epic from the planning artifacts produced in Stage 2.

| Agent | Model | Role |
|-------|-------|------|
| Team Lead | Opus | Assigns tasks, manages remediation |
| Software Developer | Sonnet | Implements code via TDD (red-green-refactor) |
| Code QA | Opus | Validates acceptance criteria, runs 9-check quality pipeline |

**QA pipeline (9 checks)**: test suite, coverage (>=90%), type checking (mypy strict), linting (ruff), formatting, security (bandit), Docker build, CHANGELOG, git status.

If QA finds failures, the Developer gets targeted remediation tasks (max 2 retries per story).

**Output**: `src/`, `tests/`, and per-story QA reports in `docs/phases/epic-{NNN}-{desc}/qa/`.

### Stage 4: Verification

**Mode**: Solo interactive session

Walks you through each acceptance criterion (Given/When/Then) one at a time. You mark each as pass, fail, or skip. The skill:

- Infers severity from AC context (never asks you to rate)
- Writes results incrementally (supports `resume` if you stop midway)
- Routes failures back to `/sagerstack:builder` for remediation

**Output**: `docs/phases/epic-{NNN}-{desc}/qa/uat-report.md`

## Architecture

### Framework Structure

Everything lives in the `.claude/` directory:

```
.claude/
  skills/                              # Skill definitions (SKILL.md + references + workflows)
    sagerstack-code-planning/          # Project planning (OKR framework)
    sagerstack-planner/                # Epic planning orchestration
    sagerstack-builder/                # Epic building orchestration
    sagerstack-verify/                 # Interactive UAT verification
    sagerstack-code-qa/                # QA validation methodology (preloaded by QA agent)
    sagerstack-software-engineering/   # Python architecture standards (preloaded by Developer agent)
    sagerstack-local-testing/          # Testing infrastructure (preloaded by Developer agent)
    sagerstack-deploy-aws/             # AWS Terraform deployment
    project-memory/                    # Cross-session knowledge tracking

  agents/                              # Agent definitions
    planner-researcher.md              # Sonnet — codebase + web research
    planner-ba.md                      # Opus  — stories with FR/TR/AC
    planner-architect.md               # Opus  — implementation plans
    planner-critic.md                  # Opus  — critical review
    builder-developer.md               # Sonnet — TDD implementation
    builder-qa.md                      # Opus  — quality validation

  commands/sagerstack/                 # Slash commands (entry points)
    code-planning.md
    planner.md
    builder.md
    verify.md
    hotfix.md
```

### How Skills Connect to Agents

Skills are preloaded into agents so each agent has the domain knowledge it needs:

| Agent | Preloaded Skills |
|-------|-----------------|
| `builder-developer` | `sagerstack-software-engineering`, `sagerstack-local-testing`, `project-memory` |
| `builder-qa` | `sagerstack-code-qa`, `project-memory` |
| `planner-ba` | `sagerstack-code-planning`, `project-memory` |
| `planner-architect` | `project-memory` |
| `planner-critic` | `project-memory` |

### Artifact Structure

When the pipeline runs against a project, it produces this structure:

```
docs/
  project-context.md                   # From Stage 1 (objective, milestones, epics)
  phases/
    epic-001-{desc}/
      epic.md                          # Epic overview
      research/findings.md             # Research output
      stories/story-001-{desc}.md      # User stories with FR/TR/AC
      plans/story-001-{desc}-plan.md   # Implementation plans
      plans/story-001-{desc}-critical-analysis.md
      qa/story-001-{desc}-qa-report.md # QA validation report
      qa/uat-report.md                 # UAT results from verification
    epic-002-{desc}/
      ...
  project_notes/                       # Cross-session memory
    bugs.md                            # Bug log with solutions
    decisions.md                       # Architectural Decision Records
    key_facts.md                       # Project configuration
    issues.md                          # Work history
```

## Quality Standards

These standards are embedded in the skills and automatically enforced by the agents:

| Area | Standard |
|------|----------|
| **Architecture** | Vertical Slice + DDD, strict domain purity (no infrastructure imports in domain) |
| **Naming** | CamelCase everywhere (classes, functions, variables, tests) |
| **Testing** | TDD mandatory (red-green-refactor), coverage >= 90% |
| **Configuration** | No hardcoded values — all config from `.env` files |
| **QA** | Zero-trust validation, 9-check pipeline, AC-driven testing |
| **Planning** | 100% FR/TR/AC coverage mapping, cost flagging, max 2 refinement cycles |
| **Git** | Feature branches per story, latest main merged before push |

## Customization

The framework is designed to be adapted to your team's standards:

- **Quality standards** — Edit the skills in `.claude/skills/sagerstack-software-engineering/` and `.claude/skills/sagerstack-code-qa/` to change architecture patterns, naming conventions, coverage thresholds, or QA checks.
- **Agent models** — Change the `model` field in agent definitions under `.claude/agents/` to use different Claude models (e.g., switch all Opus agents to Sonnet for lower cost).
- **Pipeline stages** — Each stage is independent. You can use the planner without the builder, or skip verification for internal tooling.
- **Templates** — Story, epic, and implementation plan templates are in `.claude/skills/sagerstack-planner/references/artifact-templates.md`.

## Acknowledgments

- **[project-memory](https://github.com/SpillwaveSolutions/project-memory)** by [SpillwaveSolutions](https://github.com/SpillwaveSolutions) — the `project-memory` skill in this project is based on their work.
