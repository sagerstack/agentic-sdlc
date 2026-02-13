# AI-Powered SDLC: Adaptive Task Batching Architecture

## Overview

This document describes the three-agent architecture for implementing user stories through adaptive task batching with independent quality validation.

### Architecture
```
/dev command (orchestrator)
  ├─→ py-developer (implementation specialist)
  │    └─→ Returns batch report
  ├─→ code-qa (independent validator)
  │    └─→ Returns evidence-based verdict
  └─→ Handles results, retries, progress updates
```

### Benefits
- **Context Reuse**: 60-75% (vs 0% per-task, 100% monolithic)
- **User Visibility**: Progress updates every 30-60 min (vs 2-4 hour black box)
- **Crash Recovery**: Max 1 batch lost (3-5 tasks, 30-60 min work)
- **Token Efficiency**: 150K-250K tokens (vs 615K per-task, 50K monolithic)
- **Quality Assurance**: Independent validation prevents self-reporting bias

---

## Responsibilities Summary

| Agent | Role | Tools | Core Responsibilities | What It Does NOT Do |
|-------|------|-------|----------------------|---------------------|
| **/dev command** | **Orchestrator** | Read, Grep, Glob, Task, Write, TodoWrite | • Parse impl plan into batches (3-5 tasks)<br>• Invoke py-developer per batch<br>• Invoke code-qa for validation<br>• Monitor timeouts/crashes<br>• Handle retry logic (max 2 retries)<br>• Update session context between batches<br>• Post progress to user<br>• Manage checkpoints (.dev-session-*.json)<br>• Escalate failures to user | • Does NOT write code<br>• Does NOT run tests<br>• Does NOT validate quality |
| **py-developer** | **Implementation Specialist** | Read, Write, Edit, MultiEdit, Bash, Grep, Glob, SlashCommand | • Implement batch tasks (3-5 tasks only)<br>• Write code using TDD (Red-Green-Refactor)<br>• Write tests (unit, integration, E2E, acceptance)<br>• Run tests during implementation<br>• Mark tasks [x] AFTER tests pass<br>• Follow Clean Architecture & DDD<br>• Commit & push to feature branch<br>• Return structured batch report | • Does NOT validate own work<br>• Does NOT orchestrate batches<br>• Does NOT invoke other agents<br>• Does NOT execute tasks outside batch scope |
| **code-qa** | **Independent Validator** | Read, Bash, Grep, Glob | • Re-execute ALL tests independently (zero trust)<br>• Measure coverage independently<br>• Scan for mock violations (.env.local enforcement)<br>• Verify acceptance criteria with evidence<br>• Run static analysis (mypy, ruff, bandit)<br>• Return structured verdict: PASS/FAIL/PASS_WITH_WARNINGS<br>• Document all evidence in report | • Does NOT write/edit code<br>• Does NOT fix failures<br>• Does NOT trust py-developer reports<br>• Read-only validation only |

---

## Workflow Phases

### Phase 1: Batch Planning
1. Parse implementation plan to extract all parent tasks with metadata
2. Cluster tasks into batches (3-5 tasks) using:
   - Sequential dependency chains → same batch
   - Same file/module modifications → same batch
   - Same functional category → same batch
   - Max 5 tasks per batch constraint
3. Initialize session context

### Phase 2: Batch Execution Loop (per batch)
1. Post batch start notification to user
2. Invoke py-developer with batch scope + session context (30-60 min timeout)
3. **After Task returns**: Dual-Source Analysis (CRITICAL)
   - **Source 1 - Implementation Plan**: Read impl plan, check [x] or [ ] for each batch task
   - **Source 2 - Developer Log**: Read log file (`US-{story-id}-impl-*.md`), parse workflow steps, quality checks, last activity
   - **Cross-Validate**: Combine both sources for complete picture
4. **Determine success/failure** based on dual-source analysis:
   - SUCCESS: All batch tasks [x] in impl plan + all quality checks ✅ in log
   - PARTIAL: Some tasks [x], some [ ] → Identify NEXT incomplete task to resume
   - FAILURE: All tasks [x] but quality check ❌ in log → Extract specific failure details
   - MANUAL: [MANUAL] marker in log → Save checkpoint, escalate
5. **Smart Retry** based on analysis:
   - Partial: Resume from NEXT incomplete task (not retry entire batch)
   - Failure: Target SPECIFIC quality issue (not blanket retry)
   - Max 2 retries per batch
6. Escalate to user if max retries exhausted

### Phase 3: QA Validation (after successful batch)
1. Check if batch has FRs/TRs (skip QA if setup-only batch)
2. Invoke code-qa with user story + code location + batch scope
3. **Parse code-qa verdict** (PASS/FAIL/PASS_WITH_WARNINGS)
4. If FAIL: **Granular Remediation Analysis**
   - Parse code-qa failure report (detailed JSON with file, line, task mapping)
   - Map each failure to responsible task(s): Coverage issue → [5.0], Mock violation → [7.0], AC failure → [6.0]
   - Determine remediation scope: Single task [5.0] OR multiple tasks [5.0, 6.0, 7.0]
   - Re-invoke py-developer with SPECIFIC fixes per task (not blanket "fix quality issues")
   - Max 2 QA retries (qa_retry_count)
5. If PASS or PASS_WITH_WARNINGS: Proceed to next batch
6. Escalate QA failures if max retries exhausted (qa_retry_count >= 2)

### Phase 4: Completion
1. Post final summary (tasks, coverage, quality checks)
2. Delete checkpoint file
3. Done

---

## Complete Flow Diagram

```mermaid
flowchart TD
    Start([User runs /dev US-025]) --> ParsePlan[Parse Implementation Plan]
    ParsePlan --> ClusterTasks[Cluster Tasks into Batches<br/>3-5 tasks per batch]
    ClusterTasks --> InitContext[Initialize Session Context]
    InitContext --> BatchLoop{More batches?}

    BatchLoop -->|Yes| PostBatchStart[Post: Starting Batch N/M]
    BatchLoop -->|No| FinalSummary[Post Final Summary]

    PostBatchStart --> InvokePyDev[Invoke py-developer<br/>Batch scope + Session context<br/>Timeout: 60 min]

    InvokePyDev --> PyDevResult{py-developer result}

    PyDevResult -->|SUCCESS| ProcessBatchReport[Process Batch Report]
    PyDevResult -->|TIMEOUT 60 min| CheckProgress
    PyDevResult -->|CRASH/Error| CheckProgress
    PyDevResult -->|MANUAL BLOCK| ManualHandler

    CheckProgress[Analyze Developer Log<br/>Read impl plan [x] markers<br/>Parse quality checks] --> CalcProgress{Progress?}

    CalcProgress -->|100% complete| ProcessBatchReport
    CalcProgress -->|1-99% partial| PartialRetry{retry_count < 2?}
    CalcProgress -->|0% no progress| FullRetry{retry_count < 2?}

    PartialRetry -->|Yes| RetryIncomplete[Retry incomplete tasks only<br/>retry_count++]
    PartialRetry -->|No| EscalateUser[Escalate to User<br/>Save checkpoint<br/>Exit]

    FullRetry -->|Yes| RetryFull[Retry full batch<br/>retry_count++]
    FullRetry -->|No| EscalateUser

    RetryIncomplete --> InvokePyDev
    RetryFull --> InvokePyDev

    ManualHandler[MANUAL Block Handler<br/>Save checkpoint<br/>Post action required] --> ExitManual([Exit - User Action Required])

    ProcessBatchReport --> UpdateSession[Update Session Context<br/>with batch results]

    UpdateSession --> CheckQA{Batch has<br/>FRs/TRs?}

    CheckQA -->|No - Setup only| SaveCheckpoint
    CheckQA -->|Yes| InvokeCodeQA[Invoke code-qa<br/>User story + Code location<br/>Timeout: 15 min]

    InvokeCodeQA --> CodeQAResult{code-qa result}

    CodeQAResult -->|TIMEOUT/CRASH| QARetryCheck{qa_retry < 2?}
    QARetryCheck -->|Yes| InvokeCodeQA
    QARetryCheck -->|No| LogQAError[Log QA error<br/>Skip validation] --> SaveCheckpoint

    CodeQAResult -->|PASS| PostSuccess[Post: Batch N complete<br/>QA verified]
    CodeQAResult -->|PASS_WITH_WARNINGS| PostWarnings[Post: Batch N complete<br/>QA verified with warnings]
    CodeQAResult -->|FAIL| QAFailHandler{qa_retry_count < 2?}

    QAFailHandler -->|Yes| Remediate[Re-invoke py-developer<br/>with QA fixes<br/>qa_retry_count++]
    QAFailHandler -->|No| EscalateQA[Escalate QA Failure<br/>Save checkpoint<br/>Exit]

    Remediate --> InvokePyDev

    PostSuccess --> SaveCheckpoint
    PostWarnings --> SaveCheckpoint

    SaveCheckpoint[Save .dev-session-STORY.json] --> BatchLoop

    FinalSummary --> DeleteCheckpoint[Delete checkpoint file]
    DeleteCheckpoint --> Done([Done ✅])

    EscalateUser --> ExitFail([Exit - Manual Intervention])
    EscalateQA --> ExitQAFail([Exit - QA Failure])

    style Start fill:#e1f5e1
    style Done fill:#e1f5e1
    style ExitManual fill:#fff4e1
    style ExitFail fill:#ffe1e1
    style ExitQAFail fill:#ffe1e1
    style InvokePyDev fill:#e1e5ff
    style InvokeCodeQA fill:#ffe1f5
    style EscalateUser fill:#ffe1e1
    style EscalateQA fill:#ffe1e1
    style ManualHandler fill:#fff4e1
```

---

## Conditional Scenarios

| Failure Mode | Detection | Recovery | User Visibility |
|--------------|-----------|----------|-----------------|
| **py-developer Crash** | Task tool returns error | Check progress → Retry batch up to 2x | "❌ Batch X crashed. Retrying incomplete tasks..." |
| **py-developer Timeout** | Task tool timeout (60 min) | Check progress → Retry incomplete tasks (partial retry) | "⚠️ Batch X timeout at Y% complete. Retrying Z remaining tasks..." |
| **py-developer Hang** | Task tool timeout (60 min) | Same as timeout | "⚠️ Batch X timeout. Retrying..." |
| **Partial Completion** | Read impl plan [x] markers | Retry only incomplete tasks | "Batch X was 60% complete (2/3 tasks). Retrying task [7.0]..." |
| **No Progress** | All tasks unmarked after timeout/crash | Full batch retry up to 2x | "⚠️ Batch X made no progress (0%). Retrying full batch..." |
| **Max Retries Exhausted** | retry_count ≥ 2 | Escalate to user, save checkpoint | "❌ Batch X failed after 3 attempts. Options: 1) Manual debug 2) Skip batch 3) Reduce batch size 4) Fallback to --no-batching" |
| **MANUAL Task Block** | py-developer returns MANUAL_BLOCK | Save checkpoint, no retry (user action required) | "⚠️ Blocked by MANUAL task [X]. Action required: ... Resume with: /dev US-025 --resume" |
| **code-qa FAIL** | code-qa verdict: FAIL | Re-invoke py-developer with remediation (max 2 QA retries) | "❌ QA validation failed. Failures: ... Remediation attempt 1/2..." |
| **code-qa Max Retries** | qa_retry_count ≥ 2 | Escalate to user | "❌ QA validation failed after 2 remediation attempts. Options: 1) Manual fix 2) Adjust user story 3) Skip QA" |
| **code-qa PASS_WITH_WARNINGS** | code-qa verdict: PASS_WITH_WARNINGS | Log warnings, proceed to next batch | "✅ QA validated: PASS (with warnings). ⚠️ Linting issues found. Address before final PR." |
| **code-qa Crash/Timeout** | Task tool error or 15 min timeout | Retry code-qa up to 2x, then skip validation | "⚠️ QA validation timeout. Retry 1/2..." |

---

## Retry Counter Management

| Counter | Scope | Max Value | Incremented When | Reset When |
|---------|-------|-----------|------------------|------------|
| `retry_count` | Per batch (py-developer failures) | 2 | py-developer timeout/crash | Batch succeeds OR escalated to user |
| `qa_retry_count` | Per batch (code-qa failures) | 2 | code-qa returns FAIL | Batch QA passes OR escalated to user |

### Example Retry Flow
```
Batch 2:
  1. py-developer timeout → retry_count = 1
  2. py-developer retry: Success → retry_count reset to 0
  3. code-qa: FAIL → qa_retry_count = 1
  4. py-developer remediation: Success
  5. code-qa retry: PASS → qa_retry_count reset to 0
  6. Batch 2 complete ✅

Batch 3:
  - Fresh counters: retry_count = 0, qa_retry_count = 0
```

---

## Session Context Structure

Session context maintains state between batches and enables crash recovery:

```json
{
  "user_story_id": "US-025",
  "impl_plan_path": "domain/execution/.../US-025-impl-plan.md",
  "src_path": "services/lunarcrush/src",
  "total_batches": 12,
  "completed_batches": [
    {
      "batch_id": "1",
      "batch_name": "Setup & Foundation",
      "tasks": ["[2.0]", "[3.0]", "[4.0]"],
      "completed_at": "2025-10-06T10:35:00Z"
    },
    {
      "batch_id": "2",
      "batch_name": "Authentication Core",
      "tasks": ["[5.0]", "[6.0]", "[7.0]"],
      "completed_at": "2025-10-06T11:20:00Z"
    }
  ],
  "completed_tasks": ["[2.0]", "[3.0]", "[4.0]", "[5.0]", "[6.0]", "[7.0]"],
  "created_files": [
    ".env.example",
    "pyproject.toml",
    "domain/entities/api_key.py",
    "infrastructure/auth/bearer_authenticator.py",
    "infrastructure/http/lunarcrush_client.py"
  ],
  "architectural_decisions": [
    "Using httpx.AsyncClient for all HTTP operations",
    "API key stored as domain value object (APIKey)",
    "Bearer token authentication via BearerAuthenticator class",
    "Environment variables via python-dotenv"
  ],
  "test_coverage_status": {
    "unit": 23.7,
    "integration": 18.5,
    "e2e": 12.0
  },
  "next_batch_hints": "Batch 3 should extend LunarCrushClient with timeout handling, reuse BearerAuthenticator"
}
```

**Saved to**: `.dev-session-{user-story-id}.json` after each batch
**Used for**: Resume functionality (`/dev US-025 --resume`)

---

## Post-Execution Monitoring Strategy

**Simple and Effective**: /dev analyzes py-developer work AFTER Task tool returns (no real-time monitoring needed).

### Developer Log Analysis (After py-developer Returns)

**When Task tool returns** (success/timeout/error), /dev performs comprehensive analysis:

**Step 1: Find Log File**
- Use Glob tool: `domain/execution/.../US-{story-id}/US-{story-id}-impl-*.md`
- Get most recent log file by timestamp in filename

**Step 2: Parse Log Sections**

Read entire log file and extract:

**A. Header (Lines 1-10)**:
```markdown
**Status**: ✅ Complete / ⚠️ Partial / ❌ Failed
**Duration**: 35 minutes
```
→ Quick status check

**B. Section 2: Workflow Steps Table**:
```markdown
| Timestamp | Step | Task ID | Description | Changes Summary | Metrics |
| 21:55:42 | Task | 6.2 | Integration test... | test_auth.py | 3 tests |
```
→ Extract LAST row for most recent activity

**C. Section 3: Quality Checks Summary**:
```markdown
| Check | Status | Details | Timestamp |
| 2. Coverage | ❌ FAILED | 91% < 95% required | 21:56:10 |
```
→ Identify which quality gate failed

**Step 3: Cross-Check Impl Plan**
- Read implementation plan file
- Check [x] markers for batch tasks
- Confirm: Log says "task 6.0 complete" → impl plan shows `[6.0] [x]`

**Step 4: Smart Decision Logic**

Combine log + impl plan insights:

```
If all batch tasks marked [x] AND all quality checks ✅:
  → SUCCESS: Proceed to code-qa validation

If some tasks marked [x] AND log shows recent activity:
  → PARTIAL: Retry incomplete tasks with context from log

If quality check ❌ in log:
  → FAILURE: Extract failure reason, retry with targeted fix

If log shows "MANUAL task detected":
  → MANUAL BLOCK: Save checkpoint, escalate to user

If no recent activity in log (last entry >30 min ago):
  → HUNG: Likely stuck, retry with shorter timeout
```

**Step 5: Enhanced User Feedback**

Post comprehensive analysis to user:

```
⚠️ Batch 2 timeout after 60 min

📋 Developer Log Analysis:
  Last Activity: 21:55:42 (4 min before timeout)
  Working On: [6.2] Integration test for authentication
  Progress: 67% (2/3 tasks marked [x])

  ✅ Completed:
    - [5.0] Bearer authenticator (15 min, 12 tests passed)
    - [6.0] API client base (23 min, 8 tests passed)

  🔄 In Progress:
    - [7.0] Endpoint testing (started 21:08, last activity 21:55)

  📊 Metrics:
    - Tests: 20/20 passed
    - Coverage: 91% (target: 95%, gap: 4%)
    - Last Change: Added integration test (test_auth.py:45)

  ❌ Quality Check Failure:
    - Check 2 (Coverage): 91% < 95% required
    - Missing coverage: Error handling branches

🎯 Retry Strategy:
  Resume task [7.0] with focus on error handling tests
  Expected coverage gain: +5% → 96% (meets requirement)
  Timeout: 30 min (estimated remaining work)
```

### Why This Works

| Data Source | Information | Enables |
|-------------|------------|---------|
| **Developer Log** | Task timeline, test results, quality checks, last activity | Understand what py-developer did and where it stopped |
| **Impl Plan [x]** | Task completion status | Confirm which tasks are truly complete |
| **Combined** | Complete picture of batch execution | Smart retry decisions with targeted fixes |

**No complex monitoring infrastructure needed** - just read files after Task returns.

---

## Key Workflow Rules

1. **Batch Scope Enforcement**: py-developer executes ONLY tasks in batch scope (3-5 tasks), never the entire impl plan
2. **Task Marking**: py-developer marks tasks [x] in impl plan ONLY AFTER tests pass during TDD
3. **Dual-Source Detection**: /dev ALWAYS analyzes both impl plan [x] markers AND developer log for complete picture
4. **Smart Resume**: After timeout/crash, /dev resumes from NEXT incomplete task (not retry entire batch)
5. **Granular Remediation**: code-qa failures mapped to specific tasks → targeted fixes (not blanket retry)
6. **Zero Trust Validation**: code-qa re-executes ALL tests independently, never trusts py-developer's self-reported results
7. **Context Propagation**: Session context passed to each py-developer invocation contains learnings from all previous batches
8. **Independent QA**: code-qa runs AFTER py-developer completes batch, validates against user story requirements
9. **Smart QA Skipping**: QA validation skipped for setup-only batches (no FRs/TRs to verify)
10. **Checkpoint Persistence**: Session context saved after every batch to enable crash recovery
11. **Escalation Protocol**: After 2 failed retries (py-developer or code-qa), escalate to user with options
12. **MANUAL Task Handling**: No retry on MANUAL tasks - save checkpoint and wait for user action

---

## Command Usage

```bash
# Basic usage (default: 60 min timeout, 2 retries, auto batch size)
/dev US-025

# Custom timeout (fail faster)
/dev US-025 --timeout=30 --max-retries=3

# Smaller batches (more frequent updates)
/dev US-025 --batch-size=3

# Resume from checkpoint after crash/MANUAL task
/dev US-025 --resume

# Fallback to monolithic mode (single long run)
/dev US-025 --no-batching

# Skip QA for specific batch (after manual fix)
/dev US-025 --resume --skip-qa-batch=2
```

---

## Benefits vs Alternatives

| Approach | Context Reuse | User Visibility | Crash Recovery | Token Cost | Quality Assurance |
|----------|---------------|-----------------|----------------|------------|-------------------|
| **Adaptive Batching (This)** | 60-75% | ✅ Every 30-60 min | ✅ Max 1 batch lost | 150K-250K | ✅ Independent QA |
| **Monolithic** | 100% | ❌ 2-4 hour black box | ❌ Restart from scratch | 50K-75K | ⚠️ Self-validation only |
| **Per-Task** | 0-5% | ✅ Every 15-45 min | ✅ Max 1 task lost | 615K-1M | ✅ Independent QA |

**Why Adaptive Batching Wins**:
- Balanced context reuse (not too little, not too wasteful)
- Regular progress updates without constant interruption
- Reasonable crash recovery (30-60 min work lost max)
- Acceptable token cost (not 10x worse than monolithic)
- Independent quality validation catches issues early
