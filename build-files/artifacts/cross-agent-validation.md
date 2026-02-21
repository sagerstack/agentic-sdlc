# Cross-Agent Validation Checklist

## Purpose
This document defines validation rules to prevent common issues that arise from miscommunication between business-analyst, solution-architect, and py-developer agents.

## Business-Analyst → Solution-Architect Handoff

### User Story Validation

#### Measurable Acceptance Criteria Requirements
- [ ] **Quantifiable Assertions**:
  - ✅ MUST use specific numbers: "100-500 tokens" NOT "high-volume tokens"
  - ✅ MUST use comparison operators: ">= $50,000" NOT "sufficient volume"
  - ✅ MUST use measurable metrics: "< 120 seconds" NOT "fast startup"
  - ❌ FORBIDDEN vague language: "appropriate", "sufficient", "reasonable", "high-volume"

- [ ] **Config Value Validation**:
  - IF FR/TR defines config value (e.g., TOKEN_VOLUME_THRESHOLD_USD=50000)
  - THEN at least one AC MUST validate that config value is used
  - Example AC: "All cached tokens have volume_24h_usd >= TOKEN_VOLUME_THRESHOLD_USD"
  - Example AC: "Log shows 'tokens_filtered_by_volume: N' where N > 0"

- [ ] **Business Logic Validation**:
  - ✅ ACs must test OUTCOMES (token count, volume values, cache size)
  - ❌ ACs must NOT only test PROCESS (logs exist, workflow completes)
  - Example GOOD: "Cache contains 100-500 tokens AND all have volume >= $50K"
  - Example BAD: "Cache seeding completes successfully"

#### E2E Test Requirements
- [ ] **E2E Test Requirements**: If any AC requires E2E testing with HTTP endpoints:
  - ✅ MUST include FR for "Presentation layer API endpoints"
  - ✅ MUST specify required routes in FR description
  - ❌ DO NOT assume presentation layer exists elsewhere

#### Live Test Requirements
- [ ] **Live Test Requirements**: If any AC requires real service validation:
  - ✅ MUST specify ".env.local credentials" in Given clause
  - ✅ MUST specify "real [service] (not mocks)" in When clause
  - ✅ MUST define expected non-zero results for liquid instruments

#### Docker Requirements
- [ ] **Docker Requirements**: If any AC requires Docker testing:
  - ✅ MUST include TR for "Docker environment setup"
  - ✅ MUST specify health check endpoints needed

## Solution-Architect → py-developer Handoff

### Implementation Plan Validation

#### Tech Research → Implementation Plan Data Consistency
- [ ] **API Field Coverage Validation**:
  - For each API endpoint documented in tech research:
    - ✅ Identify ALL data fields available in API response
    - ✅ Implementation plan MUST parse all required fields OR explicitly defer with justification
    - ✅ If deferred: Document target story ID, field name, and business reason
    - ✅ **Defer Target Validation**: Target story MUST show explicit task parsing deferred field
    - ❌ FORBIDDEN: Silent omission of available data fields without justification
    - ❌ FORBIDDEN: Orphaned defers where target story assumes field already implemented
  - Example GOOD: "volume24h deferred to US-045" AND US-045 shows "[X.Y] Parse volume24h from API (deferred from US-044)"
  - Example BAD: US-044 says "volume deferred to US-045" BUT US-045 says "integrate with US-044 metadata" (assumes volume already there)

- [ ] **API Response Field Accounting**:
  - IF tech research shows API returns field X (e.g., volume24h, tvlUSD, dayNtlVlm)
  - AND implementation plan parser omits field X
  - THEN plan MUST include one of:
    1. Task to parse field X into domain model
    2. Explicit defer statement: "Field X deferred to US-YYY (reason)"
    3. Explicit exclusion justification: "Field X not needed because..."
  - ❌ DO NOT create implementation gaps where API provides data but parsers silently ignore it

#### Service Integration Task Requirements
- [ ] **New Service Integration Mandate**:
  - IF implementation plan creates new service class (e.g., VolumeFilterPriceService)
  - THEN plan MUST include these tasks IN ORDER:
    1. [{X}.1] Define service interface/class
    2. [{X}.2] Implement core methods
    3. [{X}.3] Write unit tests (isolated, mocked dependencies)
    4. [{X}.4] 🔧 **INTEGRATION**: Import service in main.py or calling module
    5. [{X}.5] 🔧 **INTEGRATION**: Instantiate service with config/dependencies
    6. [{X}.6] 🔧 **INTEGRATION**: Update calling code to invoke service
    7. [{X}.7] E2E test: Verify service is called in production flow
    8. [{X}.8] Live test: Verify service with real dependencies

- [ ] **Config Usage Validation**:
  - IF user story defines config value (e.g., TOKEN_VOLUME_THRESHOLD_USD)
  - THEN implementation plan MUST include task showing config → service wiring
  - Example: "Pass settings.TOKEN_VOLUME_THRESHOLD_USD to VolumeFilterPriceService()"
  - Example: "Verify grep 'TOKEN_VOLUME_THRESHOLD_USD' src/ returns usage in service"

- [ ] **Service Dependency Verification**:
  - Before marking implementation plan complete, CHECK:
    - [ ] Service class file created?
    - [ ] Service imported somewhere in src/ (not just tests)?
    - [ ] Service instantiated in dependency injection?
    - [ ] Service method called from application code?
  - IF any check fails → Add missing integration task

#### FR Intent Preservation Validation (CRITICAL)
- [ ] **Approach/Method Validation**:
  - IF FR specifies HOW to implement (e.g., "query APIs", "pre-populate before X", "subscribe to events")
  - THEN implementation plan MUST preserve that approach OR explicitly justify substitution
  - ❌ FORBIDDEN: Silent substitution of implementation method without justification

  - **Action Verb Preservation** (validates WHAT action to take):
    - "query", "retrieve", "fetch", "call" → Proactive API calls (PULL data) → Tasks MUST include HTTP GET/POST to endpoints
    - "subscribe", "track", "listen", "monitor" → Reactive event handling (RECEIVE data) → Tasks MUST include event handler registration
    - "pre-populate", "seed", "initialize" → Upfront data loading BEFORE workflow starts → Tasks MUST complete before primary workflow activates
    - "transform", "normalize", "validate" → Data processing operations → Tasks MUST include processing logic
    - Changing verb type (proactive ↔ reactive) requires explicit justification

  - **Temporal Sequence Preservation** (validates WHEN action happens):
    - "before X", "after Y completes", "upon completion of Z" → Strict ordering constraints
    - "during startup", "prior to activation", "immediately after" → Timing requirements
    - Implementation plan tasks MUST preserve temporal order through task dependencies and phase sequencing
    - Example VIOLATION: FR says "pre-populate BEFORE WebSocket activates" but tasks say "track events FROM WebSocket"

  - **Data Source Preservation** (validates WHERE data comes from):
    - FR specifies "query Solana API" → Tasks MUST call Solana HTTP endpoints (not WebSocket events)
    - FR specifies "subscribe to WebSocket" → Tasks MUST register WebSocket event handlers (not API polling)
    - FR specifies "from blockchain sources (Solana, BSC, Hyperliquid)" → Tasks MUST query ALL specified sources
    - Changing data source requires explicit justification

- [ ] **FR-to-Task Mapping Validation**:
  - For EACH FR, architect MUST create task section: `[X.0][FR-Y] <FR Title>`
  - solution-architect MUST read FR description word-by-word to extract:
    - [ ] Action verbs (query, retrieve, pre-populate, track, subscribe)
    - [ ] Temporal constraints (before, after, during, upon completion)
    - [ ] Data sources (API endpoints, WebSocket streams, database, cache)
    - [ ] Sequencing requirements (step 1 → step 2 → step 3)
  - Validate implementation plan tasks preserve EACH extracted constraint

- [ ] **Substitution Justification Requirement**:
  - IF implementation plan changes FR approach (different verb, different source, different sequence)
  - THEN plan MUST include "FR Interpretation Notes" section with:
    ```markdown
    ### FR-X Approach Substitution
    - **Original FR Intent**: [Exact quote from FR]
    - **Alternative Approach**: [What implementation plan does instead]
    - **Reason for Substitution**: [Technical justification]
    - **Trade-offs**: [What is gained/lost]
    - **Validation**: [How to verify new approach meets FR intent]
    ```
  - ❌ FORBIDDEN: Changing FR approach without documented justification

- [ ] **Real-World Example - FR-12 Price Cache Warmup**:

  **FR-12 Requirement**:
  > "After token metadata cache seeding completes (US-045), pre-populate price cache with initial prices for high-volume tokens from blockchain sources (query Solana for Raydium/Orca prices, BSC for PancakeSwap prices, Hyperliquid L1 for native prices) before WebSocket updates activate"

  **Constraint Extraction Checklist**:
  - [ ] Temporal dependency: "After US-045 completes" → Task must wait for US-045
  - [ ] Action verb: "pre-populate" → Proactive cache seeding (not reactive tracking)
  - [ ] Method: "query blockchain sources" → Direct API calls (not event listening)
  - [ ] Data sources: Solana (Raydium + Orca), BSC (PancakeSwap), Hyperliquid → 3-4 API calls
  - [ ] Data target: "initial prices for high-volume tokens" → Specific filtered dataset
  - [ ] Temporal sequence: "before WebSocket updates activate" → Pre-population BEFORE subscriptions start

  **CORRECT Implementation** (preserves all constraints):
  ```
  [14.0][FR-12] Cache Warmup on Startup with Blockchain Sources
    [14.1] Wait for US-045 token metadata cache seeding completion
    [14.2] Query Solana Raydium API for top 100 high-volume token prices
    [14.3] Query Solana Orca API for top 100 high-volume token prices
    [14.4] Query BSC PancakeSwap API for top 100 high-volume token prices
    [14.5] Query Hyperliquid L1 API for top 100 high-volume token prices
    [14.6] Pre-populate price cache with retrieved prices (batch insert)
    [14.7] Verify price cache contains minimum 100 initial prices
    [14.8] Proceed to WebSocket subscription activation (AFTER verification)
  ```

  **VIOLATION Example** (what actually happened in US-050):
  ```
  ❌ WRONG:
  [14.1] Track warmup events from WebSocket (Raydium/Orca via Solana WebSocket)
  [14.2] Track warmup events from BSC WebSocket (PancakeSwap)
  [14.3] Track warmup events from Hyperliquid WebSocket (native prices)
  [14.4] Warmup completion detection (all DEXs reach target)

  ISSUES:
  - ❌ Action verb changed: "query" → "track" (proactive → reactive)
  - ❌ Method changed: "API calls" → "WebSocket events"
  - ❌ Temporal sequence violated: No tasks execute BEFORE WebSocket activation
  - ❌ No justification provided for approach substitution
  - ❌ FR intent lost: "immediate cache hit capability" not achievable with event-driven approach
  ```

#### Task Order Validation
- [ ] **Presentation Layer Before E2E**:
  - If AC E2E tests use `curl http://localhost:8000/endpoint`
  - THEN presentation layer creation task MUST come BEFORE E2E test tasks
  - Task should specify: "Create FastAPI routes: GET /status, POST /test-event, etc."

#### Docker Task Clarity
- [ ] **Docker Setup Tasks**:
  - ✅ Include explicit task: "Verify Docker daemon running"
  - ✅ Include explicit task: "Measure docker-compose build time"
  - ✅ Specify: "Docker build typically <60s with cache"
  - ❌ DO NOT mark Docker tasks as ">10 min" without measurement

#### Docker Redeployment Before Testing
- [ ] **CRITICAL: Always Rebuild Containers Before E2E/Live Tests**:
  - ✅ MUST rebuild Docker containers with latest code BEFORE running E2E tests
  - ✅ MUST rebuild Docker containers with latest code BEFORE running live tests
  - ✅ Use: `docker-compose up -d --build` to ensure latest code is deployed
  - ✅ Wait 5 seconds after rebuild for services to be ready
  - ❌ DO NOT run E2E/live tests against stale containers (tests will fail or give false results)

#### Live Test Instructions
- [ ] **Live Test Tasks** (AC-X.7):
  - ✅ MUST state: "Use .env.local credentials"
  - ✅ MUST state: "Connect to real [service] endpoint"
  - ✅ MUST state: "Monitor for X minutes, expect Y events"
  - ✅ MUST state: "Validate non-zero results (liquid instruments)"
  - ❌ DO NOT mark as "deferred to integration phase"

#### Test Coverage Requirements
- [ ] **Every AC MUST have**:
  - [{X}.4] Unit tests (mocked)
  - [{X}.5] Integration tests (test doubles)
  - [{X}.6] E2E tests (Docker + curl)
  - [{X}.7] Live verification (real services)

## py-developer Execution Validation

### Pre-Execution Checks

#### Environment Validation
- [ ] **Before marking any task SKIPPED**:
  - ✅ CHECK: Does .env.local exist? → Use it for live tests
  - ✅ CHECK: Is Docker running? → Start it (takes <30s)
  - ✅ CHECK: Do E2E tests need endpoints? → Build presentation layer

#### Time Measurement Requirements
- [ ] **Before skipping for ">10 min"**:
  - ✅ MEASURE: Run `time docker-compose build`
  - ✅ MEASURE: Run `time docker-compose up -d`
  - ✅ ONLY skip if measured time > 10 min
  - ❌ DO NOT assume based on perception

### Test Execution Validation

#### Suspicious Results Detection
- [ ] **Red Flags Requiring Investigation**:
  - "0 swap events detected" → Liquid pairs ALWAYS have events
  - "No transactions in 100 blocks" → Mainnet ALWAYS active
  - Test passes with empty results → Likely connection issue
  - Test skipped when .env.local exists → Configuration error

#### E2E Test Blockers
- [ ] **If E2E test requires endpoints that don't exist**:
  - ✅ BUILD the presentation layer first
  - ✅ Create FastAPI routes as needed
  - ❌ DO NOT skip E2E test
  - Presentation layer is PART of story scope if tests need it

- [ ] **Before running E2E tests**:
  - ✅ MUST rebuild Docker containers: `docker-compose up -d --build`
  - ✅ MUST wait for services to be ready (sleep 5 seconds)
  - ❌ DO NOT run tests against stale containers (will test old code)

#### Live Test Execution
- [ ] **When .env.local exists**:
  - ✅ EXECUTE all live tests
  - ✅ Connect to real services
  - ✅ Monitor for actual events
  - ❌ DO NOT skip because "requires live connection"

### Token Budget Rules
- [ ] **Session Management**:
  - ❌ NEVER stop due to "token budget"
  - ❌ NEVER ask about continuing
  - ✅ Continue until 100% complete OR blocking MANUAL task
  - User directive: "budget not be a consideration"

## Common Anti-Patterns to Avoid

### Business-Analyst Anti-Patterns
- ❌ Creating ACs with E2E tests but no FR for presentation layer
- ❌ Vague live test requirements ("verify with real data")
- ❌ Missing Docker setup requirements when E2E tests need it
- ❌ **Vague acceptance criteria** ("high-volume tokens" instead of "100-500 tokens")
- ❌ **Testing process not outcomes** ("seeding completes" instead of "token count < 1000")
- ❌ **Defining config values without validation ACs** (TOKEN_VOLUME_THRESHOLD_USD defined but no AC validates usage)

### Solution-Architect Anti-Patterns
- ❌ E2E test tasks before presentation layer creation tasks
- ❌ Marking Docker tasks as ">10 min" without evidence
- ❌ Vague live test instructions ("use real endpoint")
- ❌ Missing explicit ".env.local" usage instructions
- ❌ **Creating service without integration tasks** (VolumeFilterPriceService implemented but never wired)
- ❌ **No config → service mapping** (config value defined but not passed to service)
- ❌ **No import verification task** (service created but never imported in production code)
- ❌ **Orphaned defers** (US-044 defers volume to US-045, but US-045 doesn't add volume parsing)

### py-developer Anti-Patterns
- ❌ Skipping E2E tests due to missing presentation layer (build it!)
- ❌ Skipping live tests when .env.local exists
- ❌ Accepting 0 results from liquid instruments
- ❌ Stopping due to token budget concerns
- ❌ Assuming Docker operations >10 min without measuring
- ❌ **Running E2E/live tests without rebuilding Docker containers** (tests stale code, not latest changes)

## Enforcement

### Pre-Generation Validation
- business-analyst MUST validate user story before submission
- solution-architect MUST validate impl plan task order
- py-developer MUST validate environment before execution

### Runtime Validation
- py-developer MUST log suspicious results
- py-developer MUST measure actual times
- py-developer MUST attempt before skipping

### Post-Execution Validation
- Review implementation logs for anti-patterns
- Update agents if new patterns emerge
- Document lessons learned