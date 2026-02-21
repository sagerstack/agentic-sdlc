# Task Batching Algorithm for /cook Command

## Purpose
Define the algorithm for batching implementation plan tasks into executable groups following **strict sequential order**.

---

## Core Principle

**Tasks MUST be batched in the exact order they appear in the implementation plan.**

❌ **NEVER reorder tasks** based on category, file affinity, or dependencies
✅ **ALWAYS follow sequential order**: 0.1 → 1.X → 2.X → 3.X → ...

---

## Batching Rules (Applied in Order)

### Rule 1: Sequential Order (Primary Rule)

**Tasks are processed in their implementation plan sequence**

Example from impl plan:
```markdown
### 0. Initiation
  - [ ] [0.1] Create `.env.local` file

### 1. Manual Prerequisites
  - [ ] [1.1][MANUAL] Navigate to pricing page
  - [ ] [1.2][MANUAL] Subscribe to plan
  - [ ] [1.3][MANUAL] Generate API key
  ...
  - [ ] [1.9][MANUAL] Confirm 200 OK response

### 2. Environment & Setup
  - [ ] [2.1] Verify Python 3.11+ installed
  - [ ] [2.2] Verify Poetry installed
  ...
  - [ ] [2.7] Install test dependencies
```

**Sequential processing order**: 0.1, then 1.1-1.9, then 2.1-2.7 (never reorder)

---

### Rule 2: Max Batch Size (5 Tasks)

**No batch exceeds 5 tasks**

If consecutive tasks exceed 5, split into multiple batches:

Example:
```markdown
Tasks: [2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7] (7 consecutive tasks)

Split into:
→ Batch 3: [2.1, 2.2, 2.3, 2.4, 2.5] (5 tasks)
→ Batch 4: [2.6, 2.7] (2 tasks)
```

---

### Rule 3: Manual Task Isolation

**Tasks with `[MANUAL]` markers MUST be isolated from automated tasks**

Manual tasks cannot be mixed with automated tasks in the same batch.

Example:
```markdown
Sequence: [0.1], [1.1][MANUAL], [1.2][MANUAL], [2.1], [2.2]

Batching:
→ Batch 1: [0.1] (automated - 1 task)
→ Batch 2: [1.1][MANUAL], [1.2][MANUAL] (manual - 2 tasks)
→ Batch 3: [2.1], [2.2] (automated - 2 tasks)
```

---

### Rule 4: Manual Task Batching

**Multiple consecutive `[MANUAL]` tasks can batch together (max 5)**

Example:
```markdown
Tasks: [1.1][MANUAL], [1.2][MANUAL], ..., [1.9][MANUAL] (9 manual tasks)

Split into:
→ Batch 2: [1.1, 1.2, 1.3, 1.4, 1.5][MANUAL] (5 tasks)
→ Batch 3: [1.6, 1.7, 1.8, 1.9][MANUAL] (4 tasks)
```

---

## Algorithm Execution Steps

```markdown
Step 1: Parse Implementation Plan
  - Read impl plan document
  - Extract all tasks in sequential order
  - Preserve exact order: [0.1, 1.1, 1.2, ..., 1.9, 2.1, 2.2, ...]
  - Tag each task: MANUAL or AUTOMATED

Step 2: Initialize Batch Counter
  - batch_number = 1
  - current_batch = []
  - current_batch_type = null (MANUAL or AUTOMATED)

Step 3: Iterate Through Tasks Sequentially
  For each task in sequential order:
    a. Determine task type (MANUAL or AUTOMATED)

    b. Check batch compatibility:
       - If current_batch is empty:
           → Add task to current_batch
           → Set current_batch_type = task type

       - Else if task type matches current_batch_type AND len(current_batch) < 5:
           → Add task to current_batch

       - Else (type mismatch OR batch full):
           → Finalize current_batch as Batch N
           → Create new batch with current task
           → Set new batch_type = task type
           → Increment batch_number

Step 4: Finalize Last Batch
  - Add remaining tasks in current_batch as final batch

Step 5: Assign Batch Metadata
  For each batch:
    - batch_id: Sequential number (1, 2, 3, ...)
    - batch_type: MANUAL or AUTOMATED
    - tasks: List of task IDs in batch
    - task_count: Number of tasks
    - category_hint: Derived from first task's section (e.g., "Initiation", "Setup", "FR", "TR", "AC")

Step 6: Output Batch Plan
  Return ordered list of batches with metadata
```

---

## Example: US-025 Sequential Batching

**Implementation Plan Task Sequence**:
```
0.1 (automated)
1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7, 1.8, 1.9 (all MANUAL)
2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7 (automated)
3.1, 3.2, 3.3, 3.4, 3.5 (automated)
4.1, 4.2, 4.3, 4.4, 4.5, 4.6 (automated)
...
```

**Batching Result** (following sequential order):

```
Batch 1 (AUTOMATED): [0.1]
  - Category: Initiation
  - Tasks: 1
  - Type: AUTOMATED

Batch 2 (MANUAL): [1.1, 1.2, 1.3, 1.4, 1.5]
  - Category: Manual Prerequisites
  - Tasks: 5
  - Type: MANUAL

Batch 3 (MANUAL): [1.6, 1.7, 1.8, 1.9]
  - Category: Manual Prerequisites
  - Tasks: 4
  - Type: MANUAL

Batch 4 (AUTOMATED): [2.1, 2.2, 2.3, 2.4, 2.5]
  - Category: Environment & Setup
  - Tasks: 5
  - Type: AUTOMATED

Batch 5 (AUTOMATED): [2.6, 2.7, 3.1, 3.2, 3.3]
  - Category: Environment & Setup / Project Config
  - Tasks: 5
  - Type: AUTOMATED

Batch 6 (AUTOMATED): [3.4, 3.5, 4.1, 4.2, 4.3]
  - Category: Project Config / Docker
  - Tasks: 5
  - Type: AUTOMATED

Batch 7 (AUTOMATED): [4.4, 4.5, 4.6, 5.1, 5.2]
  - Category: Docker / FR-1
  - Tasks: 5
  - Type: AUTOMATED

... (continue in sequential order)
```

**Key Point**: Tasks are batched in exact sequence (0.1 → 1.X → 2.X → 3.X), not grouped by category (all FR together, all TR together).

---

## Edge Cases

### Case 1: Single Manual Task

**Input**: [0.1], [1.1][MANUAL], [2.1], [2.2]

**Output**:
```
Batch 1: [0.1] (automated)
Batch 2: [1.1] (manual - isolated)
Batch 3: [2.1, 2.2] (automated)
```

Manual tasks are isolated even if there's only one.

---

### Case 2: Large Manual Block (>5 tasks)

**Input**: [1.1][MANUAL] through [1.15][MANUAL] (15 manual tasks)

**Output**:
```
Batch 2: [1.1, 1.2, 1.3, 1.4, 1.5] (manual)
Batch 3: [1.6, 1.7, 1.8, 1.9, 1.10] (manual)
Batch 4: [1.11, 1.12, 1.13, 1.14, 1.15] (manual)
```

Split into multiple manual batches, each ≤5 tasks.

---

### Case 3: Alternating Manual/Automated

**Input**: [1.1], [1.2][MANUAL], [1.3], [1.4][MANUAL], [1.5]

**Output**:
```
Batch 1: [1.1] (automated)
Batch 2: [1.2] (manual - isolated)
Batch 3: [1.3] (automated)
Batch 4: [1.4] (manual - isolated)
Batch 5: [1.5] (automated)
```

Each type change triggers new batch, even if result is single-task batches.

---

### Case 4: No Manual Tasks

**Input**: All tasks are automated [0.1] through [41.8]

**Output**:
```
Batch 1: [0.1, 1.1, 1.2, 1.3, 1.4] (automated - 5 tasks)
Batch 2: [1.5, 1.6, 1.7, 1.8, 1.9] (automated - 5 tasks)
Batch 3: [2.1, 2.2, 2.3, 2.4, 2.5] (automated - 5 tasks)
... (continue in groups of 5)
```

Simply chunk into consecutive groups of ≤5 tasks.

---

## Anti-Patterns (DO NOT DO)

### ❌ Anti-Pattern 1: Grouping by Category

**WRONG**:
```
Batch 1: All FR tasks [5.1-13.4]
Batch 2: All TR tasks [14.1-20.3]
Batch 3: All AC tasks [21.1-32.7]
```

**Reason**: Violates sequential order (skips over tasks to group by category)

---

### ❌ Anti-Pattern 2: Grouping by File Affinity

**WRONG**:
```
Batch 1: All tasks touching lunarcrush_client.py [6.1, 7.1, 8.1, 11.1]
Batch 2: All tasks touching settings.py [12.1, 14.1, 18.1]
```

**Reason**: Violates sequential order (jumps around to group by file)

---

### ❌ Anti-Pattern 3: Dependency-Based Reordering

**WRONG**:
```
Sequence: [5.1], [6.1] (depends on 5.1), [7.1] (independent)
Batching: Batch 1: [5.1, 6.1], Batch 2: [7.1]
```

**Reason**: While this respects dependencies, if the impl plan has 7.1 between 5.1 and 6.1 in sequence, you must follow that order.

**CORRECT**:
Follow impl plan sequence even if it seems suboptimal for dependencies.

---

## Benefits of Sequential Batching

1. **Predictable Progress**: User sees tasks completed in documented order
2. **Impl Plan Integrity**: Respects author's intended execution flow
3. **Simpler Logic**: No complex dependency/affinity analysis needed
4. **Fewer Errors**: Avoids reordering bugs and missed tasks
5. **Audit Trail**: Orchestration log matches impl plan section-by-section

---

## Implementation in /cook Command

```markdown
# /cook orchestrator pseudocode:

1. Read impl plan
2. Extract tasks in order: tasks = [0.1, 1.1, 1.2, ..., 41.8]
3. Initialize: batches = [], current_batch = [], current_type = null
4. For each task in tasks:
     if current_batch is empty:
       current_batch.append(task)
       current_type = task.type
     elif task.type == current_type AND len(current_batch) < 5:
       current_batch.append(task)
     else:
       batches.append(current_batch)
       current_batch = [task]
       current_type = task.type
5. batches.append(current_batch)  # Final batch
6. Execute batches sequentially
```

---

## Summary

| Aspect | Rule |
|--------|------|
| **Primary Principle** | Sequential order (NEVER reorder) |
| **Batch Size** | Max 5 tasks per batch |
| **Manual Tasks** | Isolated from automated tasks |
| **Manual Batching** | Multiple consecutive manual tasks batch together (max 5) |
| **Category/File Grouping** | ❌ NOT USED (causes reordering) |
| **Dependency Chains** | ❌ NOT USED (causes reordering) |
