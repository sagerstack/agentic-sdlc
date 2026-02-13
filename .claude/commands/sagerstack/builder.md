Invoke the `sagerstack-builder` skill.

This spawns a 2-member agent team (Software Developer, Code QA) to implement a planned phase. Executes implementation plans via TDD with QA validation gates per story.

Requires implementation plans to exist at `docs/phases/phase-N.M/plans/` (created by `/sagerstack:planner`).

User input: $ARGUMENTS

If user input specifies a phase number (e.g., "1.1", "phase 2.3"), build that phase.
If user input includes "--story N", build only that story within the phase.
If user input includes "--continue", auto-chain to the next phase after completion.
If user input is "resume", resume from the last incomplete task.
If user input is "status", report current build progress.
If user input is empty, scan for planned phases and present build status.
