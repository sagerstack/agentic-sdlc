# Manage Phases Workflow

TODO: Phase management operations on project-context.md.

## Purpose
Allows the user to modify the phase structure in docs/project-context.md without
running a full planning workflow. Supports insert, remove, reorder, split, and merge.

## Operations

### Insert Phase
- User specifies where to insert (e.g., "after phase 1.2")
- BA creates new phase entry with name, description, DoD
- All subsequent phases renumbered (e.g., old 1.3 becomes 1.4)
- Update milestone totals

### Remove Phase
- User specifies phase to remove (e.g., "remove phase 2.1")
- BA removes phase entry
- All subsequent phases renumbered
- Update milestone totals

### Reorder Phases
- User specifies new order within a milestone
- BA reorders and renumbers
- Validate no dependency violations

### Split Phase
- User specifies phase to split (e.g., "split phase 1.3 into two")
- BA creates two new phases from original
- Renumber subsequent phases

### Merge Phases
- User specifies phases to merge (e.g., "merge 1.2 and 1.3")
- BA combines into single phase
- Renumber subsequent phases

## Key Behaviors
- Always confirm with user before modifying project-context.md
- Renumber ALL subsequent phases after any modification
- Validate milestone integrity after changes
- No team spawn needed -- BA operates directly

## References
- SKILL.md essential principles: Phase Management
- docs/project-context.md (from /sagerstack:code-planning)
