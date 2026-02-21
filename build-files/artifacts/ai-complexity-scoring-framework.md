# AI Complexity Scoring Framework

## Purpose & Overview

The AI Complexity Score provides a standardized 1-10 metric for estimating the complexity of AI-assisted code generation for user stories, epics, and implementation plans. Unlike traditional human-effort estimation (hours/days/weeks), this framework measures factors that directly impact AI agent performance: research depth required, code generation complexity, integration challenges, and testing automation scope.

### Key Objectives
- **Standardized Complexity Assessment**: Consistent scoring across all artifacts and team members
- **AI Generation Planning**: Predict number of refinement iterations needed
- **Architect-Critic Workflow**: Determine when architect-critic collaboration is required
- **Research Depth Planning**: Estimate web research cycles needed for solution-architect

### Integration Points
- **Used by**: business-analyst (epic/story sizing), solution-architect (implementation planning)
- **Referenced in**: Epic artifact, User Story artifact, Implementation Plan artifact
- **Outputs**: AI Complexity Score (1-10), Expected iterations, Research depth level

---

## Core Scoring Formula

```
AI Complexity Score = Research Depth Factor + Code Complexity Factor + Integration Factor + Testing Factor

Total Range: 0-10 points

Factor Contributions:
- Research Depth Factor: 0-3 points
- Code Complexity Factor: 0-3 points
- Integration Complexity Factor: 0-2 points
- Testing Scope Factor: 0-2 points
```

---

## Factor 1: Research Depth (0-3 points)

Measures the web research effort required for solution-architect to generate implementation plans.

### Scoring Matrix

| Score | Level | Research Cycles | Characteristics | Examples |
|-------|-------|----------------|-----------------|----------|
| **0** | None | 0 cycles | Well-known patterns, no external research needed, standard library usage | Basic CRUD operations, Simple data models, Standard REST endpoints |
| **1** | Standard | 1-2 cycles | Single official documentation source, well-documented APIs, common patterns | FastAPI endpoint setup, PostgreSQL connection, Redis caching |
| **2** | Deep | 4-6 cycles | Multiple sources, architect-critic refinement needed, some ambiguity in docs | Third-party API integration, Rate limiting algorithms, OAuth2 implementation |
| **3** | Extensive | 8+ cycles | Novel patterns, incomplete documentation, multiple competing approaches | Custom sentiment analysis, Distributed systems patterns, WebSocket with fallbacks |

### Scoring Indicators (Add +1 for each)

**Documentation Quality**:
- ❌ No official SDK available (requires manual HTTP client implementation) = +1
- ❌ API documentation incomplete, outdated, or contradictory = +1
- ❌ No official examples or integration guides = +1

**Pattern Novelty**:
- 🆕 Novel integration pattern not commonly documented = +1
- 🆕 Multiple competing approaches requiring trade-off analysis = +1
- 🆕 Custom algorithm design required (not library-based) = +1

**Research Breadth**:
- 📚 Requires researching 4+ different technologies/services = +1
- 📚 Performance benchmarking needed across options = +1
- 📚 Security research for compliance requirements = +1

### Research Cycle Definition

**1 Research Cycle** = solution-architect performs:
1. WebSearch for relevant documentation/examples
2. WebFetch to read 2-3 official docs or implementation guides
3. Analyze and synthesize findings into implementation guidance
4. Document specific API endpoints, configuration, integration patterns

---

## Factor 2: Code Complexity (0-3 points)

Measures the structural complexity of code generation based on file count, lines of code, and architectural patterns.

### Scoring Matrix

| Score | Level | File Count | LOC Range | Architectural Pattern | Examples |
|-------|-------|-----------|-----------|----------------------|----------|
| **0** | Trivial | 1-2 files | <100 LOC | Single function/script, no architecture | Configuration file, Simple utility function |
| **1** | Low | 3-5 files | 100-500 LOC | Simple layered (routes + service) | REST endpoint with service layer, Basic data model |
| **2** | Medium | 6-15 files | 500-2000 LOC | Clean architecture (domain + infra + app layers) | Feature module with repository pattern, Background task processor |
| **3** | High | 16+ files | >2000 LOC | Event-driven, microservices, distributed patterns | Real-time data pipeline, Multi-service orchestration |

### Scoring Indicators (Add +1 for each)

**Code Patterns**:
- ⚡ Async/concurrent programming required (asyncio, threading) = +1
- 🔄 State management across multiple components = +1
- 🎨 Multiple design patterns in single story (Factory + Observer + Strategy) = +1

**Cross-Cutting Concerns**:
- 📊 Structured logging with context propagation = +1
- 🔐 Security layer (authentication, authorization, encryption) = +1
- 📈 Monitoring/observability instrumentation = +1

**Data Flow Complexity**:
- 🔀 Complex data transformations (multi-stage ETL) = +1
- 💾 Multiple database interactions with transactions = +1
- 🌊 Stream processing with backpressure handling = +1

---

## Factor 3: Integration Complexity (0-2 points)

Measures complexity of external system integrations (APIs, databases, message queues).

### Scoring Matrix

| Score | Level | Integration Count | Integration Types | Examples |
|-------|-------|------------------|------------------|----------|
| **0** | None | 0 external APIs | Self-contained logic only | Pure business logic, Data transformation utils |
| **1** | Simple | 1-2 external APIs | Well-documented REST APIs, simple auth (API key) | Single third-party API, PostgreSQL database |
| **2** | Complex | 3+ external APIs OR complex integration | WebSockets, webhooks, OAuth2, custom protocols | Multiple APIs with correlation, Real-time event streams |

### Scoring Indicators (Add +1 for each)

**Authentication Complexity**:
- 🔑 OAuth2/OIDC flow implementation = +1
- 🎫 Token refresh and session management = +1
- 🔐 Multiple authentication mechanisms (API key + OAuth) = +1

**Communication Patterns**:
- 📡 Real-time data streams (WebSocket, Server-Sent Events) = +1
- 🪝 Webhook handling with signature verification = +1
- 🔄 Bidirectional communication with state sync = +1

**Error Handling**:
- ♻️ Retry logic with exponential backoff = +1
- 🚦 Circuit breaker pattern for fault tolerance = +1
- 📦 Message queuing for reliability = +1

**Data Coordination**:
- 🔗 Data transformation between multiple services = +1
- 🎯 Request correlation across API calls = +1
- ⏱️ Rate limiting coordination across services = +1

---

## Factor 4: Testing Scope (0-2 points)

Measures the breadth and depth of automated testing required.

### Scoring Matrix

| Score | Level | Test Types | Coverage Target | Examples |
|-------|-------|-----------|-----------------|----------|
| **0** | None | No automated tests | N/A (prototype only) | Throwaway spike, Manual testing only |
| **1** | Basic | Unit tests only | >80% unit coverage | Unit tests for business logic, Mock external dependencies |
| **2** | Comprehensive | Unit + Integration + E2E | >95% overall coverage | Full test pyramid, Performance tests, Contract tests |

### Scoring Indicators (Add +1 for each)

**Test Complexity**:
- 🎭 Mocking external services with complex behaviors = +1
- 🔄 State management testing (setup/teardown complexity) = +1
- ⚡ Async/concurrent code testing = +1

**Test Types Required**:
- 🔗 Integration tests with real external services = +1
- 🏁 End-to-end workflow tests = +1
- ⚡ Performance/load testing with benchmarks = +1
- 📜 API contract testing (OpenAPI validation) = +1

---

## Score Interpretation Guide

### Complexity Levels

| Score Range | Complexity Level | AI Generation Characteristics | Expected Iterations | Architect-Critic |
|-------------|-----------------|------------------------------|---------------------|------------------|
| **1-2** | Trivial | Single-pass generation, minimal research, standard patterns | 1 iteration | Not needed |
| **3-4** | Low | Standard research (1-2 cycles), minor refinements | 1-2 iterations | Helpful |
| **5-6** | Medium | Deep research (4-6 cycles), architect-critic review recommended | 2-3 iterations | Recommended |
| **7-8** | High | Extensive research (8+ cycles), architect-critic required | 3-4 iterations | Required |
| **9-10** | Very High | Novel patterns, multiple refinement cycles, high uncertainty | 4+ iterations | Essential |

### AI Generation Approach by Score

#### Score 1-2 (Trivial)
- **Research Phase**: Skip or minimal (use existing knowledge)
- **Code Generation**: Single-pass with py-developer
- **Review Cycles**: 0-1 (direct implementation)
- **Success Rate**: >95% first-pass success

#### Score 3-4 (Low)
- **Research Phase**: Standard (1-2 official documentation reads)
- **Code Generation**: Single-pass with minor adjustments
- **Review Cycles**: 1-2 (implementation + adjustment)
- **Success Rate**: 85-95% first-pass success

#### Score 5-6 (Medium)
- **Research Phase**: Deep (4-6 sources, architect generates impl plan)
- **Code Generation**: Multi-pass (initial + refinements)
- **Review Cycles**: 2-3 (impl plan + code + refinement)
- **Success Rate**: 70-85% first-pass success
- **Architect-Critic**: Recommended for implementation plan review

#### Score 7-8 (High)
- **Research Phase**: Extensive (8+ sources, multiple pattern evaluations)
- **Code Generation**: Iterative with architect-critic collaboration
- **Review Cycles**: 3-4 (research + impl plan + critic + code + refinement)
- **Success Rate**: 50-70% first-pass success
- **Architect-Critic**: Required (2 refinement cycles minimum)

#### Score 9-10 (Very High)
- **Research Phase**: Novel pattern research, multiple competing approaches
- **Code Generation**: Highly iterative with multiple refinement cycles
- **Review Cycles**: 4+ (extensive research + impl plan + critic cycles + code + multiple refinements)
- **Success Rate**: <50% first-pass success
- **Architect-Critic**: Essential (3+ refinement cycles expected)
- **Consider**: Breaking into smaller stories (complexity reduction)

---

## Practical Scoring Examples

### Example 1: Basic Database Model (Score: 2 - Trivial)

**Story**: "Create SQLAlchemy models for cryptocurrency sentiment data"

#### Factor Breakdown
```
Research Depth: 0 (Well-known patterns, standard TimescaleDB usage)
Code Complexity: 1 (3 files, ~200 LOC, simple layered architecture)
Integration Complexity: 0 (Database-only, no external APIs)
Testing Scope: 1 (Unit tests, >80% coverage)
```

**Total Score: 2 (Trivial)**

**AI Approach**: 1 iteration, skip research, direct implementation, no architect-critic

---

### Example 2: LunarCrush API Client (Score: 6 - Medium)

**Story**: "Implement LunarCrush API v4 client with rate limiting and error handling"

#### Factor Breakdown
```
Research Depth: 2 (No official SDK +1, API v4 docs research +1, rate limiting patterns)
Code Complexity: 2 (8 files, ~800 LOC, clean architecture, async HTTP +1)
Integration Complexity: 1 (1 external API, REST + API key, rate limiting coordination)
Testing Scope: 1 (Unit + integration tests, rate limiter tests, >85% coverage)
```

**Total Score: 6 (Medium)**

**AI Approach**: 2-3 iterations, deep research (4-6 cycles), architect-critic recommended, 1-2 refinement cycles

---

### Example 3: Real-time Sentiment Analysis Pipeline (Score: 9 - Very High)

**Story**: "Build end-to-end sentiment analysis pipeline with CryptoBERT and real-time signal generation"

#### Factor Breakdown
```
Research Depth: 3 (CryptoBERT integration +1, stream processing patterns +1, ML inference optimization +1)
Code Complexity: 3 (25+ files across collectors/transformers/analyzers/signals/publishers, ~3500 LOC,
                     event-driven architecture +1, async stream processing +1, multiple patterns +1)
Integration Complexity: 2 (4 external APIs, WebSocket +1, message queue +1, OAuth2 +1, multi-source correlation +1)
Testing Scope: 2 (Unit >90%, integration, E2E, performance tests +1, contract tests +1)
```

**Total Score: 10 (Very High - capped at 10)**

**AI Approach**: 4+ iterations, extensive research (12+ cycles), architect-critic essential (3+ refinement cycles)
**Recommendation**: Break into 3-4 smaller stories to reduce complexity

---

## Standardized Documentation Template

Use this template in Epic, User Story, and Implementation Plan artifacts:

```markdown
## AI Complexity Assessment

### Complexity Factors
| Factor | Score | Justification |
|--------|-------|---------------|
| **Research Depth** | [0-3] | [Research needs, documentation gaps, pattern novelty] |
| **Code Complexity** | [0-3] | [File count, LOC range, architectural pattern, async requirements] |
| **Integration Complexity** | [0-2] | [API count, integration patterns, auth complexity] |
| **Testing Scope** | [0-2] | [Test types, coverage targets, mocking complexity] |

### Overall AI Complexity Score: **[1-10]**

**Complexity Level**: [Trivial / Low / Medium / High / Very High]

**AI Generation Strategy**:
- **Expected Iterations**: [1 / 1-2 / 2-3 / 3-4 / 4+]
- **Research Phase**: [Skip / Standard (1-2) / Deep (4-6) / Extensive (8+) cycles]
- **Architect-Critic Review**: [Not needed / Helpful / Recommended / Required / Essential]

**Complexity Drivers**: [Specific factors increasing score beyond base]
**Risk Factors**: [Dependencies or unknowns that could increase complexity]

**Mitigation Strategies**: [How to reduce complexity or manage risks]
```

---

## Calibration Guidelines

### 1. Baseline with Reference Examples
Use the 3 examples as calibration anchors: Score 2 (Database Model), 6 (API Client), 9 (ML Pipeline)

### 2. Score Independently, Then Converge
Business-analyst and solution-architect score independently; if scores differ by >2 points, discuss and converge

### 3. Start Conservative
Start with base assumptions (0 for each factor); only add points when indicators are clearly present

### 4. Document Edge Cases
When a story doesn't fit the matrix cleanly: score using best judgment, document why, add to calibration repository

### 5. Retrospective Calibration
Compare predicted vs actual complexity after completion; update scoring indicators if systematic bias detected

### 6. Maintain Calibration Examples Repository
Create `/domain/resources/ai-complexity-examples.md` with real scored stories, actual vs predicted iterations, lessons learned

---

## Integration with SDLC Workflow

### Epic Artifact
- Business-analyst generates epic
- Scores overall epic complexity (sum of anticipated story scores)
- Uses score to determine epic prioritization and breakdown strategy

### User Story Artifact
- Business-analyst scores story during creation
- Documents in "AI Complexity Assessment" section
- Helps solution-architect understand research depth needed

### Implementation Plan Artifact
- Solution-architect re-scores after deep research phase
- May adjust score up/down based on research findings
- Documents actual complexity discovered during research

### Architect-Critic Workflow
- Complexity score 7+ triggers mandatory architect-critic collaboration
- Score determines number of refinement cycles (1 cycle per 3 points above 4)
- Example: Score 8 → 2 refinement cycles recommended

---

## Common Scoring Mistakes to Avoid

| Mistake | Wrong Approach | Right Approach |
|---------|---------------|----------------|
| **Business vs AI Complexity** | "Strategically important = high score" | Score AI generation factors only, not business value |
| **Uncertainty vs Complexity** | "Unknown requirements = complex" | Document unknowns separately, score known requirements |
| **Unfamiliar Technology** | "Team hasn't used it = complex" | If well-documented, AI can research it (score stays low) |
| **Documentation Quality** | "Just one API = score 1" | No SDK + poor docs → Research Depth +2 |
| **Static Scoring** | Keep original score | Solution-architect adjusts after research phase |

---

## Version History

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2025-01-XX | 1.0 | Product Team | Initial AI Complexity Scoring Framework creation |

---

**Reference**: This framework is the single source of truth for AI complexity estimation in the GlobalDex AI-assisted SDLC process. All agents (business-analyst, solution-architect, solution-critic) reference this framework when assessing implementation complexity.
