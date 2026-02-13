# Popular AI Prompts for Python Backend Development

This document curates the most popular and effective AI prompts for Python backend development, Clean Architecture, and Domain-Driven Design based on web research conducted in 2025.

## Table of Contents
- [Python Backend Developer Prompts](#python-backend-developer-prompts)
- [Clean Architecture Prompts](#clean-architecture-prompts)
- [Domain-Driven Design (DDD) Prompts](#domain-driven-design-ddd-prompts)
- [Test-Driven Development (TDD) Prompts](#test-driven-development-tdd-prompts)
- [Key GitHub Repositories](#key-github-repositories)
- [Best Practices](#best-practices)

---

## Python Backend Developer Prompts

### 1. High-Quality Python Development Prompt
**Source**: internetworking.dev/my-ultimate-python-prompt

```
Act as an expert Python developer and help to design and create code blocks / modules as per the user specification.

RULES:
- MUST provide clean, production-grade, high quality code.
- ASSUME the user is using python version 3.9+
- USE well-known python design patterns and object-oriented programming approaches
- MUST provide code blocks with proper google style docstrings
- MUST provide code blocks with input and return value type hinting.
- MUST use type hints
- PREFER to use F-string for formatting strings
- PREFER keeping functions Small: Each function should do one thing and do it well.
- USE @property: For getter and setter methods.
- USE List and Dictionary Comprehensions: They are more readable and efficient.
- USE generators for large datasets to save memory.
- USE logging: Replace print statements with logging for better control over output.
- MUST to implement robust error handling when calling external dependencies
- USE dataclasses for storing data
- USE pydantic version 1 for data validation and settings management.
- Ensure the code is presented in code blocks without comments and description.
- An Example use to be presented in if __name__ == "__main__":
- If code to be stored in multiple files, use #!filepath to signal that in the same code block.
```

### 2. FastAPI Backend Development Prompt

```
Create a production-ready REST API using Python's FastAPI framework with the following requirements:
- Implement full CRUD operations
- Use Pydantic models for request/response validation
- Include proper error handling with custom exceptions
- Set up database connection with async SQLAlchemy
- Implement dependency injection for database sessions
- Add comprehensive logging
- Include OpenAPI documentation
- Use type hints throughout
- Follow RESTful naming conventions
- Implement proper HTTP status codes
```

### 3. Python Microservices Prompt

```
Design a Python microservice using clean architecture principles with:
- Clear separation of concerns (domain, application, infrastructure, presentation layers)
- Async/await patterns for I/O operations
- Message queue integration (RabbitMQ/Kafka)
- Redis caching layer
- Health check endpoints
- Prometheus metrics
- Structured logging with correlation IDs
- Graceful shutdown handling
- Configuration management via environment variables
- Docker containerization
```

### 4. Python AWS Lambda Serverless Prompt

```
Write a serverless function in Python for AWS Lambda that:
- Processes incoming data from API Gateway/S3/SQS
- Uses type hints and Pydantic for validation
- Implements proper error handling and retries
- Logs structured JSON logs
- Returns properly formatted responses
- Handles cold starts efficiently
- Uses AWS SDK (boto3) with proper error handling
- Implements timeout and memory optimization
```

---

## Clean Architecture Prompts

### 1. AI Architecture Prompts (Eskil Steenberg Philosophy)
**Source**: github.com/Alexanderdunlop/ai-architecture-prompts

**Core Philosophy:**
> "It's faster to write five lines of code today than to write one line today and then have to edit it in the future."

**Prompt Principles:**
```
Transform this codebase into modular "black box" systems with:
- Clean, replaceable interfaces
- Framework-agnostic components
- Single responsibility principle adherence
- Constant developer velocity focus

Guidelines:
1. Create interfaces that can be swapped without affecting other components
2. Each module should have a clear, minimal API surface
3. Implement adapters for external dependencies (DOM, HTTP, Database)
4. Design for replaceability - any component should be removable/replaceable
5. Focus on behavior contracts, not implementation details
```

### 2. Clean Architecture Python Implementation Prompt

```
Implement this feature using Clean Architecture principles:

STRUCTURE:
src/
├── domain/           # Enterprise business rules
│   ├── entities/     # Business objects with identity
│   ├── value_objects/  # Immutable domain concepts
│   ├── repositories/ # Data access interfaces
│   └── services/     # Domain logic
├── application/      # Application business rules
│   ├── use_cases/    # Application-specific operations
│   ├── dtos/         # Data transfer objects
│   └── interfaces/   # External service contracts
├── infrastructure/   # External concerns
│   ├── persistence/  # Database implementations
│   ├── external_services/  # Third-party integrations
│   └── config/       # Configuration
└── presentation/     # User interface
    └── api/          # REST/GraphQL endpoints

RULES:
- Dependencies point inward (domain has no dependencies)
- Domain layer is framework-agnostic
- Infrastructure implements interfaces from inner layers
- Use dependency injection throughout
- Each use case handles one business operation
- All business logic in domain layer
- Presentation layer only handles I/O
```

### 3. Hexagonal Architecture (Ports and Adapters) Prompt

```
Design this system using Hexagonal Architecture:

CORE CONCEPTS:
1. Application Core (Domain + Use Cases)
   - Contains all business logic
   - No framework dependencies
   - Defines ports (interfaces) for external systems

2. Ports (Interfaces)
   - Primary Ports: Used by external actors to interact with application
   - Secondary Ports: Used by application to interact with external systems

3. Adapters
   - Primary Adapters: REST API, GraphQL, CLI, Workers
   - Secondary Adapters: Database, Message Queue, Email Service

IMPLEMENTATION:
- Define port interfaces in application layer
- Implement adapters in infrastructure layer
- Use dependency injection to wire adapters to ports
- Keep domain logic pure and testable
- Make infrastructure swappable
```

### 4. Layered Architecture with Strict Boundaries Prompt

```
Create a layered architecture with strict dependency rules:

LAYERS (from inner to outer):
1. Domain Layer
   - Pure Python, no framework imports
   - Entities, Value Objects, Domain Events
   - Business rules and invariants
   - Repository interfaces

2. Application Layer
   - Use Cases (orchestration)
   - Application Services
   - DTOs and Mappers
   - Depends only on Domain Layer

3. Infrastructure Layer
   - Database implementations
   - External API clients
   - Messaging implementations
   - Implements interfaces from Application/Domain

4. Presentation Layer
   - API Controllers
   - Request/Response models
   - Middleware
   - Depends on Application Layer

RULES:
- Each layer can only depend on layers below it
- No skipping layers (Presentation cannot directly access Domain)
- Use interfaces to invert dependencies when needed
- Test each layer independently
```

---

## Domain-Driven Design (DDD) Prompts

### 1. DDD Bounded Context Definition Prompt

```
Define a Bounded Context for [domain area] following DDD principles:

CONTEXT MAPPING:
1. Identify the Bounded Context boundaries
   - What is included/excluded?
   - What is the ubiquitous language?
   - Who are the domain experts?

2. Define Core Domain Concepts
   - Entities (objects with identity)
   - Value Objects (immutable concepts)
   - Aggregates (consistency boundaries)
   - Domain Events (significant occurrences)

3. Identify Relationships
   - Upstream/Downstream contexts
   - Shared Kernel
   - Customer/Supplier
   - Anticorruption Layer

4. Define Context Integration
   - Published Language (API contracts)
   - Open Host Service
   - Translation layers

OUTPUT:
- Bounded Context canvas
- Ubiquitous language glossary
- Context map diagram
- Entity/Value Object definitions
```

### 2. DDD Aggregate Design Prompt

```
Design an Aggregate for [business concept] using DDD tactical patterns:

AGGREGATE CHARACTERISTICS:
1. Aggregate Root
   - Entity with global identity
   - Enforces invariants
   - Controls access to internals

2. Aggregate Boundaries
   - What entities/value objects are inside?
   - What are the consistency boundaries?
   - What business rules must always be enforced?

3. Domain Events
   - What significant events occur?
   - When should events be published?
   - What data should events contain?

IMPLEMENTATION RULES:
- Only reference aggregate root from outside
- Use IDs to reference other aggregates
- Keep aggregates small
- Enforce invariants in root entity
- Publish domain events for state changes
- Make state changes transactional

PYTHON STRUCTURE:
```python
class AggregateRoot(Entity):
    def __init__(self):
        self._domain_events: List[DomainEvent] = []

    def business_operation(self, params):
        # Validate invariants
        # Update state
        # Record domain event
        self._domain_events.append(SomethingHappened(...))
```
```

### 3. DDD Entity and Value Object Prompt

```
Distinguish between Entities and Value Objects for this domain concept:

ENTITY CHECKLIST:
□ Has unique identity that persists over time
□ Has lifecycle (created, modified, deleted)
□ Equality based on ID, not attributes
□ Mutable state
□ Usually persisted to database

VALUE OBJECT CHECKLIST:
□ No identity - defined by attributes
□ Immutable
□ Equality based on all attributes
□ Can be freely shared/copied
□ Represents a domain concept (Money, Email, Address)

IMPLEMENTATION PATTERNS:

Entity:
```python
@dataclass
class User(Entity):
    id: UserId
    email: Email  # Value Object
    name: str
    created_at: datetime

    def change_email(self, new_email: Email) -> None:
        if not new_email.is_valid():
            raise ValueError("Invalid email")
        self.email = new_email
```

Value Object:
```python
@dataclass(frozen=True)
class Email:
    value: str

    def __post_init__(self):
        if not self._is_valid(self.value):
            raise ValueError(f"Invalid email: {self.value}")

    @staticmethod
    def _is_valid(email: str) -> bool:
        return "@" in email  # Simplified
```
```

### 4. DDD Repository Pattern Prompt

```
Implement the Repository pattern following DDD principles:

REPOSITORY RESPONSIBILITIES:
1. Provide collection-like interface for aggregates
2. Encapsulate persistence mechanism
3. Maintain aggregate consistency
4. Return fully reconstituted aggregates

INTERFACE DEFINITION (in Domain Layer):
```python
from abc import ABC, abstractmethod
from typing import Optional, List

class UserRepository(ABC):
    @abstractmethod
    def find_by_id(self, user_id: UserId) -> Optional[User]:
        """Retrieve user by ID or None if not found"""
        pass

    @abstractmethod
    def find_by_email(self, email: Email) -> Optional[User]:
        """Retrieve user by email"""
        pass

    @abstractmethod
    def save(self, user: User) -> None:
        """Persist user aggregate"""
        pass

    @abstractmethod
    def delete(self, user: User) -> None:
        """Remove user aggregate"""
        pass

    @abstractmethod
    def find_all(self, spec: Specification) -> List[User]:
        """Query users matching specification"""
        pass
```

IMPLEMENTATION (in Infrastructure Layer):
```python
class SQLAlchemyUserRepository(UserRepository):
    def __init__(self, session: Session):
        self._session = session

    def find_by_id(self, user_id: UserId) -> Optional[User]:
        model = self._session.query(UserModel).filter_by(
            id=str(user_id)
        ).first()
        return self._to_domain(model) if model else None

    def save(self, user: User) -> None:
        model = self._to_model(user)
        self._session.merge(model)
        self._session.commit()

    def _to_domain(self, model: UserModel) -> User:
        # Map ORM model to domain entity
        pass

    def _to_model(self, user: User) -> UserModel:
        # Map domain entity to ORM model
        pass
```

RULES:
- Define interface in domain layer
- Implement in infrastructure layer
- Use domain objects (entities, value objects) not ORM models
- Repository works with aggregate roots only
- Encapsulate query logic using Specifications
```
```

### 5. DDD Domain Service Prompt

```
Create a Domain Service for business logic that doesn't fit in entities:

WHEN TO USE DOMAIN SERVICES:
- Operation involves multiple aggregates
- Stateless operation
- Business logic that doesn't naturally belong to one entity
- Complex calculations or transformations

STRUCTURE:
```python
class PricingService:
    """Domain service for complex pricing calculations"""

    def calculate_order_total(
        self,
        order: Order,
        customer: Customer,
        promotion: Optional[Promotion] = None
    ) -> Money:
        """
        Calculate total order price considering:
        - Line item prices
        - Customer discounts
        - Promotions
        - Tax rules
        """
        subtotal = sum(item.calculate_price() for item in order.items)

        customer_discount = customer.calculate_discount(subtotal)
        subtotal = subtotal - customer_discount

        if promotion and promotion.applies_to(order):
            subtotal = promotion.apply_discount(subtotal)

        tax = self._calculate_tax(subtotal, customer.region)

        return subtotal + tax

    def _calculate_tax(self, amount: Money, region: Region) -> Money:
        # Tax calculation logic
        pass
```

CHARACTERISTICS:
- Stateless (no instance variables)
- Named after domain activity (not CRUD)
- Uses ubiquitous language
- Coordinates multiple domain objects
- Contains pure business logic
```
```

### 6. Complete DDD Implementation Prompt

```
Implement a complete DDD solution for [domain problem]:

STEP 1: Strategic Design
1. Identify Bounded Contexts
2. Create Context Map
3. Define Ubiquitous Language
4. Identify Core Domain vs Supporting Subdomains

STEP 2: Tactical Patterns
1. Define Aggregates with clear boundaries
2. Create Entities with identity
3. Model Value Objects for domain concepts
4. Design Domain Events for state changes
5. Define Repository interfaces
6. Identify Domain Services

STEP 3: Application Layer
1. Create Use Cases for each business operation
2. Define DTOs for data transfer
3. Implement Application Services for orchestration
4. Define interfaces for external services

STEP 4: Infrastructure Layer
1. Implement Repository persistence
2. Create ORM mappings
3. Implement external service adapters
4. Set up dependency injection

STEP 5: Presentation Layer
1. Create API endpoints
2. Map requests to DTOs
3. Handle errors appropriately
4. Return proper responses

PROJECT STRUCTURE:
src/
├── domain/
│   ├── aggregates/
│   ├── entities/
│   ├── value_objects/
│   ├── events/
│   ├── repositories/
│   ├── services/
│   └── specifications/
├── application/
│   ├── use_cases/
│   ├── dtos/
│   └── interfaces/
├── infrastructure/
│   ├── persistence/
│   ├── messaging/
│   └── services/
└── presentation/
    └── api/

PRINCIPLES:
- Ubiquitous language in code
- Domain layer has no dependencies
- Rich domain models (not anemic)
- Aggregate consistency boundaries
- Domain events for side effects
- Repository per aggregate root
```

---

## Test-Driven Development (TDD) Prompts

### 1. TDD Red-Green-Refactor Cycle Prompt

```
Implement [feature] using strict Test-Driven Development:

PROCESS:
1. RED: Write failing test first
   - Write test for smallest behavior increment
   - Test should fail (no implementation yet)
   - Verify test actually fails

2. GREEN: Write minimal code to pass test
   - Only write enough to make test pass
   - Don't worry about perfect design yet
   - Get to green quickly

3. REFACTOR: Improve design
   - Remove duplication
   - Improve names
   - Extract methods/classes
   - All tests still pass

RULES:
- Never write production code without failing test
- Write only enough test to fail
- Write only enough code to pass test
- Use pytest framework
- Aim for >95% coverage
- Test behavior, not implementation
- Mock external dependencies

TEST STRUCTURE:
```python
# test_user_registration.py
import pytest
from domain.entities import User
from domain.value_objects import Email
from application.use_cases import RegisterUserUseCase

class TestUserRegistration:
    def test_register_new_user_with_valid_email(self):
        # Arrange
        use_case = RegisterUserUseCase(user_repo=MockUserRepository())
        email = "test@example.com"
        name = "Test User"

        # Act
        user = use_case.execute(email=email, name=name)

        # Assert
        assert user.email.value == email
        assert user.name == name
        assert user.id is not None

    def test_register_user_with_invalid_email_raises_error(self):
        # Arrange
        use_case = RegisterUserUseCase(user_repo=MockUserRepository())

        # Act & Assert
        with pytest.raises(ValueError, match="Invalid email"):
            use_case.execute(email="invalid", name="Test User")
```
```

### 2. Unit Testing Layers Prompt

```
Write comprehensive unit tests for each architectural layer:

DOMAIN LAYER TESTS (No mocks needed - pure logic):
```python
# Test Entities
def test_user_can_change_email():
    user = User(id=UserId.generate(), email=Email("old@example.com"))
    new_email = Email("new@example.com")

    user.change_email(new_email)

    assert user.email == new_email

# Test Value Objects
def test_money_addition():
    money1 = Money(amount=10.00, currency="USD")
    money2 = Money(amount=5.00, currency="USD")

    result = money1 + money2

    assert result.amount == 15.00
    assert result.currency == "USD"

# Test Domain Services
def test_pricing_service_calculates_total_with_discount():
    service = PricingService()
    order = create_test_order(subtotal=100.00)
    customer = create_test_customer(discount_percent=10)

    total = service.calculate_order_total(order, customer)

    assert total.amount == 90.00
```

APPLICATION LAYER TESTS (Mock repositories):
```python
@pytest.fixture
def mock_user_repo():
    return Mock(spec=UserRepository)

def test_register_user_use_case(mock_user_repo):
    # Arrange
    use_case = RegisterUserUseCase(user_repo=mock_user_repo)
    mock_user_repo.find_by_email.return_value = None

    # Act
    user = use_case.execute(email="test@example.com", name="Test")

    # Assert
    mock_user_repo.save.assert_called_once()
    assert isinstance(user, User)
```

INTEGRATION TESTS (Real database):
```python
@pytest.fixture
def db_session():
    # Create test database
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    session = Session(engine)
    yield session
    session.close()

def test_repository_can_save_and_retrieve_user(db_session):
    # Arrange
    repo = SQLAlchemyUserRepository(db_session)
    user = User(
        id=UserId.generate(),
        email=Email("test@example.com"),
        name="Test User"
    )

    # Act
    repo.save(user)
    retrieved = repo.find_by_id(user.id)

    # Assert
    assert retrieved.id == user.id
    assert retrieved.email == user.email
```

TEST PYRAMID:
- 50% Unit Tests (fast, isolated)
- 30% Integration Tests (components working together)
- 20% E2E Tests (full system)
```
```

### 3. TDD for AI/LLM Applications Prompt

```
Implement Test-Driven Development for LLM/AI features:

CHALLENGES:
- Non-deterministic outputs
- External API dependencies
- Latency and cost considerations

STRATEGIES:
1. Test behavior contracts, not exact outputs
2. Use fixtures for common LLM responses
3. Mock LLM calls in unit tests
4. Use real LLM in integration tests with caching

EXAMPLE:
```python
# Test using recorded responses
@pytest.fixture
def llm_summary_response():
    return {
        "summary": "User wants to implement a payment feature",
        "key_points": ["payment", "integration", "security"],
        "confidence": 0.95
    }

def test_ticket_analyzer_extracts_requirements(
    mock_llm_client,
    llm_summary_response
):
    # Arrange
    mock_llm_client.generate.return_value = llm_summary_response
    analyzer = TicketAnalyzer(llm_client=mock_llm_client)
    ticket_text = "We need to add Stripe payment integration..."

    # Act
    result = analyzer.analyze(ticket_text)

    # Assert
    assert "payment" in result.requirements
    assert result.confidence > 0.9
    mock_llm_client.generate.assert_called_once()

# Integration test with real LLM (use VCR cassettes)
@pytest.mark.vcr()
def test_ticket_analyzer_with_real_llm():
    analyzer = TicketAnalyzer(llm_client=OpenAIClient())

    result = analyzer.analyze("Add user authentication")

    assert "authentication" in result.requirements.lower()
```

TOOLS:
- pytest-vcr for recording/replaying HTTP
- pytest-recording for LLM responses
- hypothesis for property-based testing
```
```

---

## Key GitHub Repositories

### 1. AI Architecture Prompts
**URL**: https://github.com/Alexanderdunlop/ai-architecture-prompts
**Focus**: Clean, modular architecture prompts for Claude/Cursor
**Key Features**:
- Black box interface design
- Framework-agnostic components
- Constant developer velocity
- Based on Eskil Steenberg's principles

### 2. Awesome Claude Prompts
**URL**: https://github.com/langgptai/awesome-claude-prompts
**Focus**: Curated collection of Claude prompts
**Key Features**:
- Python development prompts
- Code generation examples
- Various domain-specific prompts

### 3. Awesome Claude Code
**URL**: https://github.com/hesreallyhim/awesome-claude-code
**Focus**: Commands, files, workflows for Claude Code
**Key Features**:
- AB Method for incremental development
- Project management workflows
- Sub-agent patterns
- Code quality hooks

### 4. AI System Prompts Collection
**URL**: https://github.com/dontriskit/awesome-ai-system-prompts
**Focus**: System prompts for top AI tools
**Includes**: ChatGPT, Claude, Claude-Code, Cursor, Windsurf, and more

### 5. Python DDD Examples
**URL**: https://github.com/qu3vipon/python-ddd
**URL**: https://github.com/pgorecki/python-ddd
**Focus**: Python Domain-Driven Design implementations
**Key Features**:
- FastAPI integration
- Clean Architecture
- Tactical DDD patterns

---

## Best Practices

### 1. Prompt Engineering for Development

**Effective Prompt Structure:**
```
[ROLE] Act as [expert role]

[CONTEXT] For [project/domain context]

[TASK] Implement/Design/Create [specific task]

[CONSTRAINTS]
- MUST: Required rules
- PREFER: Preferred approaches
- USE: Tools/patterns to use
- AVOID: Anti-patterns

[OUTPUT FORMAT]
- Code structure
- Documentation requirements
- Testing requirements
```

### 2. Memory Banks for AI-First Development

Create structured context files:

```markdown
# PROJECT_CONTEXT.md

## Tech Stack
- Python 3.11+
- FastAPI
- PostgreSQL with SQLAlchemy
- Redis for caching
- Celery for background tasks

## Architecture
- Clean Architecture
- Domain-Driven Design
- Repository Pattern
- CQRS for reads/writes

## Coding Standards
- Type hints required
- Google-style docstrings
- >95% test coverage
- Black formatter
- Ruff linter

## Domain Language
- User → Customer
- Order → Booking
- Payment → Transaction
```

### 3. Iterative Refinement

Start broad, then refine:
1. Generate initial structure
2. Review and provide feedback
3. Refine implementation
4. Add tests
5. Optimize and refactor

### 4. Context Tools

Use tools to provide codebase context:
- **repomix** - JavaScript/TypeScript projects
- **onefilellm** - Python projects
- **aider** - AI pair programming

### 5. Testing Strategy

```
Test Pyramid:
┌─────────────┐
│  E2E Tests  │  20% - Full system flows
├─────────────┤
│ Integration │  30% - Components together
│    Tests    │
├─────────────┤
│    Unit     │  50% - Isolated components
│    Tests    │
└─────────────┘
```

### 6. Pre-commit Hooks

Automate quality checks:
```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: black
        name: black
        entry: poetry run black
        language: system
        types: [python]

      - id: ruff
        name: ruff
        entry: poetry run ruff check
        language: system
        types: [python]

      - id: mypy
        name: mypy
        entry: poetry run mypy
        language: system
        types: [python]

      - id: pytest
        name: pytest
        entry: poetry run pytest
        language: system
        pass_filenames: false
```

---

## References

### Articles
1. [Creating a High-Quality Python Development Prompt](https://internetworking.dev/my-ultimate-python-prompt/)
2. [Modern Test-Driven Development in Python](https://testdriven.io/blog/modern-tdd/)
3. [Clean Architecture with Python](https://medium.com/@shaliamekh/clean-architecture-with-python-d62712fd8d4f)
4. [From Python to Prompts: AI-First Development](https://www.jit.io/blog/from-python-to-prompts-becoming-an-ai-first-developer)

### Books
1. **Architecture Patterns with Python** - Harry Percival & Bob Gregory
2. **Clean Architecture** - Robert C. Martin
3. **Domain-Driven Design** - Eric Evans
4. **Test-Driven Development with Python** - Harry Percival

### Courses
1. [Clean Architecture in Python](https://www.educative.io/courses/clean-architecture-python) - Educative
2. [ChatGPT Prompt Engineering for Developers](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/) - DeepLearning.AI
3. [Modern TDD, Microservices](https://testdriven.io/) - TestDriven.io

---

## Document Metadata

- **Created**: 2025-10-03
- **Last Updated**: 2025-10-03
- **Purpose**: Reference guide for AI-assisted Python backend development
- **Maintained By**: GlobalDex Engineering Team
- **Version**: 1.0.0
