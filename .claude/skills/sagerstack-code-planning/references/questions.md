# Planning Question Bank

Comprehensive questions for the planning workflow. Organized by category.

## Category Mapping

| Category | Persona File | Skill |
|----------|--------------|-------|
| Values/Quality | persona/values.md | - |
| Constraints | persona/constitution.md | - |
| Architecture | - | software-engineering |
| Testing | - | build-test-execute |
| Deployment | - | deploy-aws |

---

## Objective (Project-Specific)

### What is the project type?
- Purpose: Understand the nature of what we're building
- Examples: CLI tool, web API, scheduled job, library

### Who is the end user?
- Purpose: Understand who benefits from this
- Examples: Internal team, external customers, other developers

### What problem does this solve?
- Purpose: Understand the "why" behind the project
- Follow-up: What happens if we don't build this?

### What is the deployment target?
- Purpose: Understand where this will run
- Examples: Lambda, EKS, local only

---

## Code Structure

### Project structure pattern?
- Check: skill:software-engineering (Vertical Slice + DDD)
- Follow-up: Any deviations needed for this project?

### Where should tests go?
- Check: skill:build-test-execute (tests/ folder)
- Follow-up: Unit, integration, e2e separation?

---

## Domain Modeling

### What are the core domain entities?
- Purpose: Identify the main concepts
- Follow-up: Relationships between entities?

### What domain events matter?
- Purpose: Identify state changes worth tracking
- Follow-up: Who consumes these events?

---

## Configuration

### What configuration is needed?
- Check: persona/values.md (no hardcoded values)
- Follow-up: Which values change between environments?

### What secrets are required?
- Check: skill:deploy-aws (Secrets Manager for prod)
- Follow-up: API keys, credentials, tokens?

---

## Infrastructure

### What AWS services are needed?
- Check: skill:deploy-aws
- Common: Lambda, S3, SNS, SQS, Secrets Manager, EKS

### What external APIs are called?
- Purpose: Identify external dependencies
- Follow-up: Rate limits, authentication, error handling?

---

## Testing

### What E2E tests must pass locally?
- Purpose: Define success criteria
- Follow-up: Happy path + edge cases

### What should be mocked vs real?
- Check: skill:build-test-execute (mock for unit, LocalStack for integration)
- Follow-up: Any exceptions?

---

## Deployment

### What triggers deployment?
- Check: skill:deploy-aws (GitHub Actions)
- Follow-up: Manual approval needed?

### What monitoring is needed?
- Purpose: How will we know it's working?
- Follow-up: Alerts, dashboards, logs?

---

## Entry Point

### What is the application entry point?
- Purpose: Understand how the code is invoked
- Examples: Lambda handler, CLI main, API router

---

## Custom Questions

_Add project-specific questions discovered during planning sessions._

<!--
Format:

### [Question text]
- Purpose: [why we ask this]
- Check: [persona file or skill, if applicable]
- Added: [date]
- Context: [why this question was needed]
-->

---

_This question bank is non-exhaustive. Generate follow-up questions based on responses._
