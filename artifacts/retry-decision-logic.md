# Retry Decision Logic for /dev Command

## Purpose
Define the exact conditions under which /dev retries py-developer invocation, and the retry strategy for each scenario.

---

## Core Principles

### 1. Dual-Source Detection (CRITICAL)

**Detection = Implementation Plan [x] Markers + Developer Log Analysis**

Both sources MUST be analyzed together:

| Source | Information | Purpose |
|--------|-------------|---------|
| **Implementation Plan** | Task completion status: [x] or [ ] | Ground truth for which tasks are complete |
| **Developer Log** | Last activity, quality checks, test results, timestamps | Context for understanding what happened |
| **Combined** | Complete picture of progress and failures | Smart retry decisions |

**Never rely on just one source:**
- ❌ Impl plan only: Don't know WHY task failed or what was being worked on
- ❌ Log only: Don't know WHICH tasks are truly marked complete
- ✅ Both: Know exactly where to resume and why

### 2. Smart Resume (Not Blind Retry)

**Resume from NEXT incomplete task, not retry entire batch**

```
Batch scope: [5.0], [6.0], [7.0]

After timeout, impl plan shows:
  [5.0] [x] ✅
  [6.0] [x] ✅
  [7.0] [ ] ❌

Smart Resume:
  → Invoke py-developer with scope: [7.0] ONLY
  → Do NOT re-implement [5.0], [6.0]
  → Session context includes results from [5.0], [6.0]
```

**Not**:
```
❌ Blind Retry: Re-invoke with full batch [5.0, 6.0, 7.0]
  (Wastes time re-implementing working tasks)
```

### 3. Granular code-qa Remediation

**Target SPECIFIC tasks/issues, not blanket retry**

```
code-qa reports 3 failures:
  1. Coverage issue in bearer_authenticator.py → Created by task [5.0]
  2. Mock violation in test_auth.py → Created by task [7.0]
  3. AC-3 test failure → Created by task [6.0]

Smart Remediation:
  → Map each failure to responsible task
  → If only [5.0] failed: Retry [5.0] ONLY
  → If all 3 failed: Retry [5.0, 6.0, 7.0] with SPECIFIC fixes for each
  → Prompt includes exact file, line, and fix needed per task
```

**Not**:
```
❌ Blanket Retry: "Fix quality issues" (vague, wastes tokens)
✅ Targeted Retry: "Task [5.0]: Add tests for lines 45-62 in bearer_authenticator.py"
```

---

## Retry Triggers (When to Retry)

### Trigger 1: Task Tool Returns TIMEOUT

**Condition**: Task tool times out after 60 minutes (timeout parameter)

**Detection**:
```
<Task subagent_type="py-developer" timeout="3600000">
  ...
</Task>

Returns: TimeoutError (after 3600000ms = 60 min)
```

**Automatic Response**: Analyze progress → Retry decision

---

### Trigger 2: Task Tool Returns ERROR/CRASH

**Condition**: py-developer crashes or encounters unhandled error

**Detection**:
```
<Task subagent_type="py-developer">
  ...
</Task>

Returns: AgentError("Out of memory")
OR
Returns: AgentError("Unexpected termination")
```

**Automatic Response**: Analyze progress → Retry decision

---

### Trigger 3: Quality Check Failure (Success with Failures)

**Condition**: Task tool returns successfully BUT developer log shows quality check failures

**Detection**:
```
Task returns: SUCCESS
Developer log Section 3 shows:
| 2. Coverage | ❌ FAILED | 88% < 95% required | ...
```

**Automatic Response**: Extract failure details → Remediation retry

---

### Trigger 4: code-qa Returns FAIL

**Condition**: code-qa independent validation detects failures py-developer missed

**Detection**:
```
code-qa returns:
{
  "verdict": "FAIL",
  "failures": [
    {"type": "COVERAGE_BELOW_THRESHOLD", "actual": 88.5, "required": 95.0},
    {"type": "MOCK_VIOLATION", "file": "test_auth.py:45"}
  ]
}
```

**Automatic Response**: Re-invoke py-developer with remediation prompt (separate retry counter: qa_retry_count)

---

## Do NOT Retry Conditions

### No Retry 1: MANUAL Task Detected

**Condition**: py-developer returns MANUAL_BLOCK

**Detection**:
```
Developer log shows:
| Step H | [18.0][MANUAL] | LunarCrush API subscription required | ...

OR

Task returns with batch report:
{
  "blocking_task": "[18.0][MANUAL]",
  "reason": "API credentials required"
}
```

**Response**: Save checkpoint, escalate to user, EXIT (no retry)

---

### No Retry 2: Max Retries Exhausted

**Condition**: retry_count >= 2 (already retried twice)

**Detection**:
```
retry_count = 2 (started at 0, incremented twice)
```

**Response**: Escalate to user with options, EXIT (no more retries)

---

### No Retry 3: Complete Success

**Condition**: All batch tasks marked [x] AND all quality checks passed

**Detection**:
```
Impl plan shows: [5.0] [x], [6.0] [x], [7.0] [x] (all batch tasks)
Developer log Section 3 shows: All checks ✅ PASSED
```

**Response**: Proceed to code-qa validation (no retry needed)

---

### No Retry 4: code-qa Max Retries Exhausted

**Condition**: qa_retry_count >= 2 (code-qa failed twice after remediation)

**Detection**:
```
qa_retry_count = 2
code-qa returns: FAIL (still failing after 2 py-developer remediation attempts)
```

**Response**: Escalate QA failures to user, EXIT

---

## Retry Decision Matrix

| Task Result | Log Analysis | Impl Plan [x] | Decision | Strategy |
|-------------|--------------|---------------|----------|----------|
| **TIMEOUT** | Last activity <5 min ago | 67% (2/3) | ✅ RETRY | Partial retry (incomplete tasks) |
| **TIMEOUT** | Last activity >30 min ago | 0% (0/3) | ✅ RETRY | Full retry (likely hung) |
| **TIMEOUT** | N/A | 100% (3/3) | ❌ NO RETRY | Actually complete, proceed to code-qa |
| **ERROR** | Quality check failed | 100% (3/3) | ✅ RETRY | Remediation retry (fix quality issue) |
| **ERROR** | N/A | 33% (1/3) | ✅ RETRY | Partial retry (incomplete tasks) |
| **SUCCESS** | Quality check ❌ | 100% (3/3) | ✅ RETRY | Remediation retry |
| **SUCCESS** | Quality check ✅ | 100% (3/3) | ❌ NO RETRY | Complete success, proceed to code-qa |
| **MANUAL_BLOCK** | N/A | Any% | ❌ NO RETRY | Escalate to user |
| **Any** | N/A | Any% | ❌ NO RETRY | If retry_count >= 2 |

---

## Detailed Retry Decision Algorithm

```markdown
## /dev instruction (after py-developer Task returns):

**Step 1: Check for MANUAL Block (Immediate Exit)**
1. Read developer log, search for "[MANUAL]" in Task ID column
2. If found:
   - Extract blocking task ID and reason
   - Post MANUAL block message to user
   - Save checkpoint to .dev-session-{story-id}.json
   - EXIT (no retry)

**Step 2: Check Retry Counter (Prevent Infinite Loops)**
1. Check retry_count from session context
2. If retry_count >= 2:
   - Post escalation message to user (max retries exhausted)
   - Save checkpoint
   - EXIT (no more retries)

**Step 3: Dual-Source Progress Analysis (CRITICAL)**

**A. Read Implementation Plan**:
1. Read impl plan file: US-{story-id}-impl-plan.md
2. For EACH task in current batch scope:
   - Task [5.0]: Search for "| [x] | [5.0] |" or "| [ ] | [5.0] |"
   - Task [6.0]: Search for "| [x] | [6.0] |" or "| [ ] | [6.0] |"
   - Task [7.0]: Search for "| [x] | [7.0] |" or "| [ ] | [7.0] |"
3. Record completion status:
   ```
   completed_tasks = ["[5.0]", "[6.0]"]  # Found [x]
   incomplete_tasks = ["[7.0]"]          # Found [ ]
   progress = 2/3 = 67%
   ```

**B. Read Developer Log**:
1. Find log file: Glob US-{story-id}-impl-*.md
2. Parse Section 2 (Workflow Steps Table):
   - Extract all rows with Step = "Task"
   - Get LAST task row → Most recent activity
   - Extract: timestamp, task ID, description, metrics
3. Parse Section 3 (Quality Checks Summary):
   - Check for ❌ FAILED status
   - Extract failure details if any
4. Parse Header:
   - Status field (✅ Complete / ⚠️ Partial / ❌ Failed)
   - Duration

**C. Cross-Validate (Detect Inconsistencies)**:
```
If impl plan shows [5.0] [x], [6.0] [x], [7.0] [ ]:
  AND log last task = [6.0]:
    → Consistent: Stopped after completing [6.0]

If impl plan shows [5.0] [x], [6.0] [ ], [7.0] [ ]:
  AND log last task = [6.2] (subtask):
    → Working on [6.0], crashed mid-task
    → Resume from [6.0] (not [7.0])

If impl plan shows ALL [x]:
  AND log shows quality check ❌ FAILED:
    → Implementation complete, quality issue
    → Need remediation, not task retry
```

**Step 4: Determine Retry Strategy Based on Combined Analysis**

**Case A: 100% Complete + All Quality Checks Passed**
```
If progress == 100% AND all quality checks ✅:
  Post: "✅ Batch complete, all quality checks passed"
  Proceed to code-qa validation
  Do NOT retry
```

**Case B: 100% Complete + Quality Check Failed**
```
If progress == 100% AND any quality check ❌:
  Extract failure details from log Section 3
  Post: "⚠️ Batch complete but quality check failed: {details}"
  Increment retry_count
  Retry Strategy: REMEDIATION
    - Re-invoke py-developer with same batch tasks
    - Add remediation prompt: "Fix quality check failure: {details}"
    - Focus: Address specific failure (e.g., "Add tests for 95% coverage")
```

**Case C: Partial Complete (1-99%) - SMART RESUME**
```
If 0 < progress < 100%:

  1. Identify next incomplete task from impl plan:
     - Scan batch tasks in order: [5.0], [6.0], [7.0]
     - Find first task with [ ] marker
     - Example: [5.0] [x], [6.0] [x], [7.0] [ ] → Next = [7.0]

  2. Cross-check with developer log:
     - If log shows "last task = [6.2]" (subtask of [6.0])
       AND impl plan shows [6.0] [ ]:
         → py-developer was working on [6.0], crashed mid-task
         → Resume from [6.0] (re-attempt entire parent task)

     - If log shows "last task = [6.0]" (parent task complete)
       AND impl plan shows [6.0] [x]:
         → py-developer finished [6.0] cleanly
         → Resume from [7.0] (next task)

  3. Post progress analysis:
     "⚠️ Batch 2 timeout after 60 min

      Impl Plan Status:
        ✅ [5.0] Complete (Bearer authenticator)
        ✅ [6.0] Complete (API client base)
        ⏸️ [7.0] Incomplete (Endpoint testing)

      Developer Log Analysis:
        - Last activity: 21:55:42 (Task [6.0] complete)
        - Tests passed: 20/20
        - Coverage: 91%

      Resume Point: Task [7.0] (next incomplete task)"

  4. Increment retry_count

  5. Retry Strategy: SMART RESUME
     - Batch scope: ONLY incomplete tasks (e.g., [7.0])
     - Do NOT re-implement completed tasks [5.0], [6.0]
     - Session context: Include results from [5.0], [6.0]
     - Prompt: "Resume from task [7.0], reuse components from [5.0], [6.0]"
```

**Case D: No Progress (0%)**
```
If progress == 0%:
  Check last activity timestamp in log

  If last_activity < 5 min ago:
    Post: "⚠️ Batch timeout, recent activity detected (likely slow, not stuck)"
    Increment retry_count
    Retry Strategy: FULL RETRY
      - Re-invoke py-developer with full batch
      - Timeout: 60 min (same as original)

  Else (last_activity > 30 min ago):
    Post: "⚠️ Batch timeout, no recent activity (likely hung)"
    Increment retry_count
    Retry Strategy: FULL RETRY WITH SHORTER TIMEOUT
      - Re-invoke py-developer with full batch
      - Timeout: 30 min (shorter to detect hang faster)
```

**Step 5: Execute Retry (If Applicable)**
1. Post retry notification to user
2. Update session context with retry_count
3. Construct retry prompt (full batch / partial / remediation)
4. Invoke py-developer again
5. Return to Step 1 (check MANUAL, check retry_count, analyze, decide)
```

---

## Retry Strategy Details

### Strategy 1: Full Batch Retry

**When**: 0% progress, recent activity detected

**Prompt**:
```markdown
You are implementing Batch 2 (RETRY 1/2).

Previous attempt timed out after 60 min with 0% progress.
Log shows last activity 3 min before timeout (likely working, just slow).

**Batch Scope**: [5.0], [6.0], [7.0] (same as original)
**Session Context**: {previous batch learnings}

Continue with same tasks. Focus on efficiency.
```

**Timeout**: 60 min (unchanged)

---

### Strategy 2: Partial Batch Retry

**When**: 1-99% progress (some tasks complete, others incomplete)

**Prompt**:
```markdown
You are implementing Batch 2 CONTINUATION (RETRY 1/2).

Previous attempt completed 2/3 tasks before timeout.

**Completed** (do NOT re-implement):
  - [5.0] Bearer authenticator ✅
  - [6.0] API client base ✅

**Incomplete** (implement these):
  - [7.0] Endpoint testing

**Batch Scope**: [7.0] ONLY
**Session Context**:
  {previous batch learnings}
  {results from [5.0], [6.0] - files created, tests passed}

Resume implementation from [7.0]. Reuse components from [5.0], [6.0].
```

**Timeout**: 30 min (estimated remaining work, not full 60 min)

---

### Strategy 3: Remediation Retry

**When**: 100% complete BUT quality check failed

**Prompt**:
```markdown
You are implementing Batch 2 REMEDIATION (QA_RETRY 1/2).

All tasks completed, but quality check failed.

**Quality Check Failure**:
  - Check 2 (Coverage): 88% < 95% required
  - Missing coverage: Error handling branches in auth.py

**Batch Scope**: [5.0], [6.0], [7.0] (re-implement to fix quality issue)
**Session Context**: {previous implementation results}

Fix the quality check failure:
1. Add tests for error handling branches
2. Target: 95% coverage (currently 88%, need +7%)
3. Focus files: infrastructure/auth/bearer_authenticator.py

Do NOT change working code unnecessarily. Only add missing tests.
```

**Timeout**: 30 min (targeted fix, not full reimplementation)

---

### Strategy 4: code-qa Remediation Retry (GRANULAR TARGETING)

**When**: code-qa validation detects failures after batch completion

**Step 1: Parse code-qa Report for Issue Location**

code-qa returns detailed failure report:
```json
{
  "verdict": "FAIL",
  "failures": [
    {
      "type": "COVERAGE_BELOW_THRESHOLD",
      "actual": 88.5,
      "required": 95.0,
      "uncovered_files": ["infrastructure/auth/bearer_authenticator.py"],
      "uncovered_lines": "45-62",
      "related_task": "[5.0]"  // If code-qa can map to task
    },
    {
      "type": "MOCK_VIOLATION",
      "file": "tests/integration/test_auth.py",
      "line": 45,
      "related_task": "[7.0]"  // Task that created this test
    },
    {
      "type": "AC_VALIDATION_FAILED",
      "ac_id": "AC-3",
      "test_file": "tests/integration/test_authentication.py",
      "reason": "Test passes in py-developer run but fails in code-qa re-run",
      "related_task": "[6.0]"
    }
  ]
}
```

**Step 2: Map Failures to Specific Tasks**

Analyze which tasks need remediation:
```
Failure 1 (Coverage): bearer_authenticator.py → Created in task [5.0]
Failure 2 (Mock): test_auth.py → Created in task [7.0]
Failure 3 (AC-3): test_authentication.py → Created in task [6.0]

Tasks needing fixes: [5.0], [6.0], [7.0]
```

**Step 3: Determine Remediation Scope**

**Option A: Single Task Issue** (1 failure)
```
If only [5.0] has issues:
  Retry scope: [5.0] ONLY
  Do NOT touch [6.0], [7.0]
```

**Option B: Multiple Task Issues** (2+ failures)
```
If [5.0], [6.0], [7.0] all have issues:
  Retry scope: All three tasks [5.0, 6.0, 7.0]
  But with SPECIFIC fixes for each
```

**Option C: Cross-Task Issue** (coverage spans multiple files)
```
If coverage issue affects files from [5.0] AND [6.0]:
  Retry scope: [5.0, 6.0]
  Leave [7.0] untouched
```

**Step 4: Construct Targeted Remediation Prompt**

**Example 1: Single Task Remediation** (Only [5.0] failed)
```markdown
You are implementing CODE-QA REMEDIATION for Task [5.0] (QA_RETRY 1/2).

Task [5.0] completed implementation but failed code-qa validation.

**code-qa Failures for [5.0]**:
  - Coverage: 88.5% < 95.0% required
  - Uncovered: bearer_authenticator.py lines 45-62 (error handling branches)

**Remediation Scope**: Task [5.0] ONLY
**Do NOT modify**: Tasks [6.0], [7.0] (passed code-qa validation)

**Session Context**:
  - [5.0] created: infrastructure/auth/bearer_authenticator.py
  - [5.0] tests: tests/unit/infrastructure/auth/test_bearer_authenticator.py

**Fix Required**:
1. Add tests for error handling branches in bearer_authenticator.py lines 45-62
2. Target coverage: 95% (currently 88.5%, need +6.5%)
3. Focus files:
   - infrastructure/auth/bearer_authenticator.py (add error handling tests)
   - tests/unit/infrastructure/auth/test_bearer_authenticator.py (add test cases)

**Do NOT**:
- Modify API client code (task [6.0])
- Modify endpoint tests (task [7.0])
- Change working code unnecessarily

Re-run tests after fixes. Verify coverage ≥95% for bearer_authenticator.py.
```

**Example 2: Multi-Task Remediation** (All 3 tasks have issues)
```markdown
You are implementing CODE-QA REMEDIATION for Batch 2 (QA_RETRY 1/2).

Multiple tasks failed code-qa validation. Fix issues for each task.

**code-qa Failures**:

Task [5.0] Issues:
  ✗ Coverage: 88.5% < 95% (bearer_authenticator.py lines 45-62)
  Fix: Add error handling tests

Task [6.0] Issues:
  ✗ AC-3 validation failed: test_authentication.py passes locally but fails in code-qa
  Fix: Investigate test isolation issue, ensure test doesn't depend on previous test state

Task [7.0] Issues:
  ✗ Mock violation: tests/integration/test_auth.py:45 uses @patch
  Fix: Remove @patch, use real API (.env.local exists)

**Remediation Scope**: Tasks [5.0], [6.0], [7.0]

**Session Context**: {all previous implementation details}

**Fixes Required** (in order):
1. [5.0]: Add tests for coverage (bearer_authenticator.py lines 45-62)
2. [6.0]: Fix AC-3 test isolation (test_authentication.py)
3. [7.0]: Remove mock, use real API (test_auth.py:45)

Work through each task fix. Re-run ALL tests after each fix.
Mark each task [x] in impl plan after fix verified.
```

**Timeout**: 30 min (for 1 task) or 60 min (for multiple tasks)

---

## Retry Counter Management

### Two Independent Counters

| Counter | Scope | Max | Incremented When | Reset When |
|---------|-------|-----|------------------|------------|
| `retry_count` | Per batch (py-developer execution failures) | 2 | Timeout, crash, quality check self-detected failure | Batch succeeds with all quality checks ✅ |
| `qa_retry_count` | Per batch (code-qa validation failures) | 2 | code-qa returns FAIL | code-qa returns PASS or PASS_WITH_WARNINGS |

### Counter Interaction Example

```
Batch 2 execution:
  1. py-developer attempt 1: TIMEOUT → retry_count = 1
  2. py-developer attempt 2: SUCCESS (quality checks ✅) → retry_count reset to 0
  3. code-qa validation: FAIL (coverage 88%) → qa_retry_count = 1
  4. py-developer remediation attempt 1: SUCCESS → qa_retry_count stays 1
  5. code-qa validation retry: PASS → qa_retry_count reset to 0
  6. Batch 2 complete ✅

Batch 3 execution:
  - retry_count = 0 (fresh start)
  - qa_retry_count = 0 (fresh start)
```

---

## Escalation Messages

### Escalation 1: Max py-developer Retries

**When**: retry_count >= 2

**Message**:
```
❌ Batch 2 failed after 3 attempts (max retries exhausted)

Last Attempt:
  - Progress: 33% (1/3 tasks)
  - Completed: [5.0]
  - Failed: [6.0], [7.0]
  - Last error: Quality Check 2 (Coverage) failed

Attempts History:
  1. Timeout at 0% → Retry
  2. Timeout at 33% (1/3) → Partial retry
  3. Timeout at 33% (1/3) → Max retries exceeded

Options:
  1. Manual intervention: Debug why tasks [6.0], [7.0] failing
  2. Reduce batch size: /dev US-025 --resume --batch-size=2
  3. Skip batch: /dev US-025 --resume --skip-batch=2
  4. Fallback to monolithic: /dev US-025 --no-batching

Session saved to: .dev-session-US-025.json
Resume command: /dev US-025 --resume
```

### Escalation 2: Max code-qa Retries

**When**: qa_retry_count >= 2

**Message**:
```
❌ Batch 2 QA validation failed after 2 remediation attempts

Last QA Result: FAIL
  - Coverage: 91.2% (required: 95%, gap: 3.8%)
  - AC-3: Test failing when re-run by code-qa

Remediation History:
  1. py-developer fix → code-qa: FAIL (coverage 89%)
  2. py-developer fix → code-qa: FAIL (coverage 91.2%)

Options:
  1. Manual intervention: Fix coverage gaps manually
  2. Adjust user story: Lower coverage requirement to 90%
  3. Skip QA: /dev US-025 --resume --skip-qa-batch=2
  4. Stop and debug: Investigate why AC-3 failing

Session saved to: .dev-session-US-025.json
```

---

## Summary

**Retry triggers**: Timeout, Crash, Quality check failure, code-qa FAIL

**No retry**: MANUAL block, Max retries (2), Complete success

**Retry strategies**: Full retry, Partial retry, Remediation retry, code-qa remediation

**Counters**: retry_count (max 2), qa_retry_count (max 2), independent per batch

**Decision logic**: Analyze log + impl plan → Determine progress → Choose strategy → Execute retry OR escalate
