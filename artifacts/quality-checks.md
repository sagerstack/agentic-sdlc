# Quality Check Procedures

## Purpose

Defines standard quality checks executed after completing each parent task during py-developer implementation. Ensures all code meets production-grade quality standards before proceeding.

---

## Execution Timing

Run these checks **after completing ALL subtasks in a parent task**.

**Execution Order**: MUST run sequentially (1 → 2 → 3 → ... → 8)

**Failure Handling**: If ANY check fails → STOP, fix issues, re-run from Check 1

---

## Quality Checks

### Check 1: Complete Test Suite

**Command**: `poetry run pytest tests/ -v`

**Purpose**: Run all tests together to validate entire suite passes without conflicts

**Success**: Exit code 0, all tests pass

**Why Run All Tests Together**:
- Individual tests already run during task execution (TDD workflow)
- This check validates: no test isolation issues, no shared state conflicts, suite-level integration works

**Prerequisites**: `.env.local` with valid credentials for integration/E2E tests

**If Fails**: STOP → display errors → fix → re-run from Check 1

---

### Check 2: Test Coverage Report

**Command**:
```bash
poetry run pytest tests/ --cov=src --cov-report=term-missing --cov-report=html --cov-fail-under=95
```

**Purpose**: Ensure minimum 95% code coverage

**Success Criteria**:
- Coverage ≥ 95%
- Exit code: 0

**Output Format**:
```
🔵 [HH:MM:SS] Generating test coverage report (target: ≥95%)...

---------- coverage: platform darwin, python 3.11.5 ----------
Name                                    Stmts   Miss  Cover   Missing
---------------------------------------------------------------------
src/domain/entities/user.py               42      0   100%
src/domain/value_objects/email.py         18      0   100%
src/application/use_cases/register.py     35      1    97%   45
src/infrastructure/repos/user_repo.py     28      0   100%
---------------------------------------------------------------------
TOTAL                                    123      1    99%

Coverage HTML written to dir htmlcov

✅ [HH:MM:SS] Coverage: 99% (exceeds 95% requirement)
```

**If Coverage < 95%**:
- ❌ Report: `"Coverage below threshold: X% (target: 95%)"`
- ❌ Display uncovered lines
- ❌ STOP workflow
- ❌ Add missing tests
- ❌ Re-run from Check 1

---

### Check 3: Static Type Checking

**Command**:
```bash
poetry run mypy src/ --strict
```

**Purpose**: Verify all type hints are correct (strict mode)

**Success Criteria**:
- Exit code: 0
- No type errors
- All functions have type hints

**Output Format**:
```
🔵 [HH:MM:SS] Running MyPy static type checking (strict mode)...
Success: no issues found in 15 source files
✅ [HH:MM:SS] MyPy passed: 0 type errors
```

**If Type Errors Found**:
```
❌ [HH:MM:SS] MyPy failed: 3 type errors detected

src/domain/entities/user.py:42: error: Missing return statement
src/application/use_cases/register.py:18: error: Argument 1 has incompatible type "str"; expected "Email"
src/infrastructure/repos/user_repo.py:25: error: Function is missing a type annotation

Found 3 errors in 3 files (checked 15 source files)
```

- ❌ STOP workflow
- ❌ Fix type errors
- ❌ Re-run from Check 1

---

### Check 4: Code Linting & Formatting

**Commands**:
```bash
poetry run ruff check src/ tests/ --fix
poetry run black src/ tests/ --check
```

**Purpose**: Ensure code follows style guidelines

**Success Criteria**:
- Ruff: 0 violations after auto-fix
- Black: All files already formatted (or formatted successfully)

**Output Format**:
```
🔵 [HH:MM:SS] Running Ruff linting and Black formatting...

Ruff check:
All checks passed! (15 files)

Black check:
All done! ✨ 🍰 ✨
15 files would be left unchanged.

✅ [HH:MM:SS] Linting passed: 0 violations
```

**If Violations Remain After Auto-Fix**:
```
❌ [HH:MM:SS] Ruff failed: 2 violations found

src/domain/entities/user.py:42:1: F401 [*] `datetime` imported but unused
src/application/use_cases/register.py:18:80: E501 Line too long (95 > 100 characters)
```

- ❌ STOP workflow
- ❌ Fix violations manually
- ❌ Re-run from Check 1

---

### Check 5: Security Scanning

**Command**:
```bash
poetry run bandit -r src/ -ll
```

**Purpose**: Detect security vulnerabilities

**Success Criteria**:
- No HIGH or MEDIUM severity issues
- LOW severity issues acceptable (but log for review)

**Output Format**:
```
🔵 [HH:MM:SS] Running Bandit security scan...

[bandit]
Run started: 2025-10-03 14:40:45

Test results:
    No issues identified.

Code scanned:
    Total lines of code: 1234
    Total lines skipped (#nosec): 0

✅ [HH:MM:SS] Security scan passed: 0 high/medium severity issues
```

**If Issues Found**:
```
❌ [HH:MM:SS] Bandit failed: 2 security issues detected

>> Issue: [B608:hardcoded_sql_expressions] Possible SQL injection vector through string-based query construction
   Severity: Medium   Confidence: Low
   Location: src/infrastructure/repos/user_repo.py:45

>> Issue: [B105:hardcoded_password_string] Possible hardcoded password
   Severity: Medium   Confidence: Medium
   Location: src/infrastructure/config.py:12
```

- ❌ STOP workflow
- ❌ Review and fix security issues
- ❌ Re-run from Check 1

**Note**: Can skip LOW severity issues, but should log for future review

---

### Check 6: Docker Build Verification

**Command**:
```bash
docker build -f docker/Dockerfile -t app:latest .
```

**Purpose**: Ensure application builds successfully in Docker

**Success Criteria**:
- Build completes with exit code 0
- Image created successfully

**Output Format**:
```
🔵 [HH:MM:SS] Building Docker image...

[+] Building 45.2s (12/12) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 1.23kB
 => [internal] load .dockerignore
 => [1/6] FROM docker.io/library/python:3.11-slim
 => [2/6] WORKDIR /app
 => [3/6] COPY pyproject.toml poetry.lock ./
 => [4/6] RUN pip install poetry && poetry install --no-dev
 => [5/6] COPY src/ ./src/
 => [6/6] RUN poetry run pytest tests/ --cov=src
 => exporting to image
 => => writing image sha256:abc123...
 => => naming to docker.io/library/app:latest

✅ [HH:MM:SS] Docker build successful
```

**If Build Fails**:
```
❌ [HH:MM:SS] Docker build failed

ERROR [4/6] RUN pip install poetry && poetry install --no-dev
 > RUN pip install poetry && poetry install --no-dev:
#8 12.34 ERROR: Could not find a version that satisfies the requirement httpx>=0.27.0
#8 12.34 ERROR: No matching distribution found for httpx>=0.27.0
```

- ❌ STOP workflow
- ❌ Fix Dockerfile or dependencies
- ❌ Re-run from Check 1

**Note**: For projects without Docker, SKIP this check

---

### Check 7: Update CHANGELOG.md

**Purpose**: Document changes for version tracking

**Actions**:
1. Open `CHANGELOG.md` file (create if doesn't exist)
2. Locate "## [Unreleased]" section (create if doesn't exist)
3. Add entry under appropriate subsection (Added/Changed/Fixed/Removed)
4. Format: `- [Category] Brief description (US-###)`
5. Include files modified
6. Save changes

**Format**:
```markdown
## [Unreleased]

### Added
- User authentication with email/password validation (US-025)
  - Added Email value object with regex validation
  - Added User entity with authentication methods
  - Added RegisterUserUseCase in application layer
  - Modified: src/domain/value_objects/email.py, src/domain/entities/user.py, src/application/use_cases/register_user.py
  - Tests: tests/domain/test_email.py, tests/application/test_register_user.py

### Changed
- Improved error messages for invalid email format (US-025)
  - Modified: src/domain/value_objects/email.py

### Fixed
- Fixed race condition in user repository (US-024)
  - Modified: src/infrastructure/repos/user_repo.py
```

**Output Format**:
```
🔵 [HH:MM:SS] Updating CHANGELOG.md with implementation details...

Added to CHANGELOG.md:
- [Added] User authentication with email/password validation (US-025)

✅ [HH:MM:SS] CHANGELOG.md updated successfully
```

---

### Check 8: Commit and Push Changes

**Command**:
```bash
/git push
```

**Purpose**: Commit all changes and push to remote feature branch

**Actions** (performed by `/git push` slash command):
1. Stage all changes (`git add .`)
2. Create commit with meaningful message
3. Push to remote repository

**Commit Message Format**:
```
feat(US-###): Brief description of parent task

- Detailed change 1
- Detailed change 2
- Detailed change 3

Files modified:
- src/path/to/file1.py
- src/path/to/file2.py
- tests/path/to/test_file.py

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Output Format**:
```
🔵 [HH:MM:SS] Committing and pushing changes to remote...

[feature/US-025-lunarcrush-authentication-setup abc1234] feat(US-025): Implement LunarCrush authentication module
 5 files changed, 342 insertions(+), 12 deletions(-)
 create mode 100644 src/integrations/lunarcrush/auth.py
 create mode 100644 src/integrations/lunarcrush/client.py
 create mode 100644 tests/integrations/lunarcrush/test_auth.py

Enumerating objects: 15, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 8 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 3.45 KiB | 3.45 MiB/s, done.
Total 9 (delta 6), reused 0 (delta 0), pack-reused 0
To github.com:user/repo.git
   def5678..abc1234  feature/US-025-lunarcrush-authentication-setup -> feature/US-025-lunarcrush-authentication-setup

✅ [HH:MM:SS] Changes committed and pushed successfully
```

**If Commit/Push Fails**:
```
❌ [HH:MM:SS] Git push failed

error: failed to push some refs to 'github.com:user/repo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
```

- ❌ STOP workflow
- ❌ Pull latest changes: `git pull --rebase`
- ❌ Resolve conflicts if any
- ❌ Re-run Check 8

---

### Check 9: Implementation Plan Task Completion

**Purpose:** Verify all completed tasks marked [x] in implementation plan

**Actions:**
1. Open implementation plan file (path stored in Step D)
2. Count tasks: Total, Marked [x], Remaining [ ]
3. Verify: Each task py-developer implemented has `[x]` checkbox

**Success Criteria:**
- Every implemented task marked `[x]`
- If any task completed but not marked → FAIL (py-developer skipped task marking)

**Output Format:**
```
🔵 [HH:MM:SS] Validating implementation plan task completion...

Tasks Review:
- Total tasks: X
- Completed [x]: Y
- Remaining [ ]: Z (manual or not implemented)

✅ [HH:MM:SS] All completed tasks marked [x] in implementation plan
```

**If Fails:** Mark completed tasks [x], verify accuracy

---

## Quality Check Result Summary

After completing all 9 checks:

### ✅ ALL Checks Pass

```
📊 Quality Check Summary
━━━━━━━━━━━━━━━━━━━━━━━━
✅ Check 1: Test Suite - PASSED (45 tests, 0 failures)
✅ Check 2: Coverage - PASSED (99%, exceeds 95% target)
✅ Check 3: Type Checking - PASSED (0 errors)
✅ Check 4: Linting - PASSED (0 violations)
✅ Check 5: Security - PASSED (0 high/medium issues)
✅ Check 6: Docker Build - PASSED
✅ Check 7: CHANGELOG - UPDATED
✅ Check 8: Git Push - COMMITTED & PUSHED
✅ Check 9: Task Completion - ALL TASKS MARKED [x]

🎉 All quality gates passed - ready for next parent task
```

**Action**: Mark parent task `[x]` complete in implementation plan, proceed to next parent task

---

### ❌ ANY Check Fails

```
📊 Quality Check Summary
━━━━━━━━━━━━━━━━━━━━━━━━
✅ Check 1: Test Suite - PASSED
✅ Check 2: Coverage - PASSED
❌ Check 3: Type Checking - FAILED (3 type errors)
⏸️  Check 4: Linting - SKIPPED (blocked by Check 3)
⏸️  Check 5: Security - SKIPPED (blocked by Check 3)
⏸️  Check 6: Docker Build - SKIPPED (blocked by Check 3)
⏸️  Check 7: CHANGELOG - SKIPPED (blocked by Check 3)
⏸️  Check 8: Git Push - SKIPPED (blocked by Check 3)

❌ Quality checks failed - resolve issues and re-run from Check 1
```

**Action**: STOP, fix issues, re-run quality checks from Check 1

---

## Quick Reference Table

| Check | Command | Threshold | Failure Action |
|-------|---------|-----------|----------------|
| **1. Test Suite** | `pytest tests/ -v` | 0 failures | Fix tests, re-run from Check 1 |
| **2. Coverage** | `pytest --cov=src --cov-fail-under=95` | ≥95% | Add tests, re-run from Check 1 |
| **3. Type Checking** | `mypy src/ --strict` | 0 errors | Fix types, re-run from Check 1 |
| **4. Linting** | `ruff check --fix` + `black --check` | 0 violations | Fix style, re-run from Check 1 |
| **5. Security** | `bandit -r src/ -ll` | 0 high/medium | Fix issues, re-run from Check 1 |
| **6. Docker Build** | `docker build -f docker/Dockerfile` | Exit code 0 | Fix Dockerfile, re-run from Check 1 |
| **7. CHANGELOG** | Manual edit | Entry added | Add entry, continue |
| **8. Git Push** | `/git push` | Exit code 0 | Resolve conflicts, re-run Check 8 |
| **9. Task Completion** | Review impl plan | All [x] | Mark completed tasks, verify |

---

## Logging Requirements

Each quality check MUST be logged to:
1. **Console output** (for user visibility)
2. **Implementation log file** (for audit trail)

**Log Format** (per `developer-log.md`):
```
[HH:MM:SS] | py-developer | Check X | Running {check name}
[HH:MM:SS] | py-developer | Check X | {Result}: {summary}
```

**Example**:
```
[14:35:00] | py-developer | Check 1 | Running test suite
[14:35:05] | py-developer | Check 1 | PASSED: 45 tests, 0 failures
[14:35:05] | py-developer | Check 2 | Generating coverage report
[14:35:10] | py-developer | Check 2 | PASSED: 99% coverage
```

---

## Notes

- **Sequential execution required**: Cannot skip ahead if earlier check fails
- **Auto-fix where possible**: Ruff and Black have `--fix` flags to auto-correct
- **Manual intervention**: Some failures require developer judgment (security issues, type errors)
- **Docker optional**: Projects without Dockerfile skip Check 6
- **Coverage exceptions**: Can exclude specific files from coverage (e.g., `# pragma: no cover`)
