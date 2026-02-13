# Phase Patterns

Guidance for structuring implementation phases. Read this during planning to inform phase breakdown.

---

## Core Philosophy

### Local-First Development
The objective for initial phases is to set up minimal foundation to run the app locally. Deliver one capability at a time with complete E2E running successfully on local machine first.

### Value-Driven Prioritization
Phases should target core capability and value add FIRST, before bells and whistles (authentication, access control, observability, etc.).

### Cloud-Ready from Day One
Develop with a view of eventual cloud deployment. Local deployment strategy should mimic eventual cloud deployment 100%. This means:
- Same containerization approach
- Same service boundaries
- Same configuration patterns
- LocalStack for AWS services

### Cloud Deployment After Core Works
Plan for AWS deployment only AFTER core capability works locally E2E. Don't deploy half-baked features to cloud.

---

## Two-Level Phase Hierarchy

### Level 1: Milestones
Larger chunks representing significant deliverables. Typically week-sized.

```
Milestone [N]: [Name]
- Delivers: [Major capability or outcome]
- Why Now: [Why this milestone before others]
- Success Criteria: [How to verify milestone completion]
```

### Level 2: Phases (within Milestones)
Smaller chunks within milestones. Typically 1-2 day sized.

```
Phase [M.N]: [Name]
- Delivers: [Specific output]
- Definition of Done:
  - [Observable outcome 1]
  - [Observable outcome 2]
- Success Criteria:
  - [Verification step 1]
  - [Verification step 2]
```

---

## Phase Ordering Principles

### 1. Minimal Runnable First
First phases should produce something that runs locally, even if minimal.

**Good**: "Phase 1: CLI skeleton that prints hello world"
**Bad**: "Phase 1: Complete domain model for all entities"

### 2. Vertical Slices Over Horizontal Layers
Each phase delivers a complete vertical slice (domain → application → infrastructure → API) for ONE capability, not all layers for all capabilities.

**Good**: "Phase 2: Create order flow (domain + repo + handler + endpoint)"
**Bad**: "Phase 2: All domain models for the entire system"

### 3. Core Value Before Infrastructure
Defer non-core requirements until core capability works.

**Defer until later:**
- Authentication / Authorization
- Access control
- Logging infrastructure
- Monitoring / Alerting
- Rate limiting
- Caching optimization

**Do early:**
- Core business logic
- Primary user journey
- Data persistence (minimal)
- Local execution

### 4. One Capability at a Time
Don't parallelize capabilities in early phases. Complete one E2E before starting next.

### 5. Cloud Deployment as Separate Milestone
AWS deployment is its own milestone AFTER local E2E works.

---

## Typical Milestone Patterns

### New Project from Scratch

```
Milestone 1: Local Foundation
- Delivers: Minimal app running locally with one core capability E2E
- Phases:
  1.1: Project skeleton (structure, dependencies, config)
  1.2: Domain model for core capability
  1.3: Repository + in-memory implementation
  1.4: Application handler
  1.5: Entry point (CLI/API) + local execution
  1.6: Docker Compose + LocalStack setup

Milestone 2: Core Capability Complete
- Delivers: Full core capability working locally
- Phases:
  2.1-N: Additional vertical slices for core capability

Milestone 3: Secondary Capabilities
- Delivers: Supporting features
- Phases: [Vertical slices for each]

Milestone 4: Production Readiness
- Delivers: Non-functional requirements
- Phases:
  4.1: Authentication/Authorization
  4.2: Error handling + logging
  4.3: Configuration management
  4.4: Health checks

Milestone 5: AWS Deployment
- Delivers: Running in AWS
- Phases:
  5.1: Terraform modules
  5.2: Secrets Manager setup
  5.3: Lambda/EKS deployment
  5.4: CI/CD pipeline
```

### New Feature for Existing Project

```
Milestone 1: Feature Foundation
- Delivers: New capability working locally E2E
- Phases:
  1.1: Domain model (new slice or extend existing)
  1.2: Repository interface + implementation
  1.3: Application handler
  1.4: API endpoint
  1.5: Integration with existing code

Milestone 2: Feature Complete
- Delivers: All feature requirements
- Phases: [Additional vertical slices]

Milestone 3: Deploy to AWS
- Delivers: Feature live in production
- Phases:
  3.1: Terraform updates (if infra changes)
  3.2: Deploy + verify
```

### Refactor/Restructure

```
Milestone 1: Safety Net
- Delivers: Tests covering existing behavior
- Phases:
  1.1: Characterization tests for existing code
  1.2: Identify boundaries and dependencies

Milestone 2: Incremental Migration
- Delivers: Code migrated to new structure
- Phases:
  2.1: New structure alongside old
  2.2: Migrate slice by slice (one at a time)
  2.3: Update consumers
  2.4: Remove old code

Milestone 3: Verification
- Delivers: Confidence in refactor
- Phases:
  3.1: E2E tests pass
  3.2: Performance validation
  3.3: Deploy + monitor
```

### Integration with External System

```
Milestone 1: Interface Definition
- Delivers: Contract with external system
- Phases:
  1.1: Understand external API (research)
  1.2: Define domain interface (abstraction)
  1.3: Mock implementation for local dev

Milestone 2: Real Implementation
- Delivers: Working integration locally
- Phases:
  2.1: Infrastructure client implementation
  2.2: Error handling + retries
  2.3: Local E2E with real calls (sandbox/test env)

Milestone 3: Production Integration
- Delivers: Live integration
- Phases:
  3.1: Secrets setup
  3.2: Deploy + verify
  3.3: Monitoring for integration health
```

---

## Anti-Patterns to Avoid

### Big Bang Phases
**Bad**: "Phase 1: Implement entire domain model"
**Good**: "Phase 1: Implement Order aggregate only"

### Horizontal Layer Phases
**Bad**: "Phase 1: All repositories, Phase 2: All handlers"
**Good**: "Phase 1: Order slice (domain + repo + handler)"

### Infrastructure Before Value
**Bad**: "Phase 1: Set up Terraform and CI/CD"
**Good**: "Phase 1: Core capability running locally"

### Premature Cloud Deployment
**Bad**: "Phase 3: Deploy to AWS" (before local E2E works)
**Good**: "Milestone 5: AWS Deployment" (after local is solid)

### Skipping Local Verification
**Bad**: Jump to cloud testing
**Good**: Full E2E locally first, then cloud

---

## Checklist Before Finalizing Phases

- [ ] First milestone produces something runnable locally
- [ ] Each phase is a vertical slice (not horizontal layer)
- [ ] Core value delivered before infrastructure concerns
- [ ] Local deployment mimics cloud deployment
- [ ] AWS deployment is a separate, later milestone
- [ ] Phase sizes are 1-2 days (not weeks)
- [ ] Milestone sizes are ~1 week (not months)
- [ ] No capability parallelization in early phases
