# Project Structure: Clean Architecture + Domain-Driven Design

This document defines the standard project structure for greenfield Python projects adopting Clean Architecture and Domain-Driven Design (DDD) principles.

## Complete Directory Structure

```
project-root/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── cd.yml
│       ├── lint.yml
│       └── static-analysis.yml
│
├── src/
│   ├── domain/                          # Domain Layer (Enterprise Business Rules)
│   │   ├── __init__.py
│   │   ├── entities/                    # Domain entities
│   │   │   ├── __init__.py
│   │   │   ├── user.py
│   │   │   ├── order.py
│   │   │   └── product.py
│   │   ├── value_objects/               # Immutable value objects
│   │   │   ├── __init__.py
│   │   │   ├── email.py
│   │   │   ├── money.py
│   │   │   └── address.py
│   │   ├── aggregates/                  # Aggregate roots
│   │   │   ├── __init__.py
│   │   │   └── order_aggregate.py
│   │   ├── events/                      # Domain events
│   │   │   ├── __init__.py
│   │   │   ├── user_registered.py
│   │   │   └── order_placed.py
│   │   ├── repositories/                # Repository interfaces
│   │   │   ├── __init__.py
│   │   │   ├── user_repository.py
│   │   │   └── order_repository.py
│   │   ├── services/                    # Domain services
│   │   │   ├── __init__.py
│   │   │   ├── pricing_service.py
│   │   │   └── inventory_service.py
│   │   ├── exceptions/                  # Domain exceptions
│   │   │   ├── __init__.py
│   │   │   ├── business_rule_violation.py
│   │   │   └── entity_not_found.py
│   │   └── specifications/              # Business rule specifications
│   │       ├── __init__.py
│   │       └── eligible_for_discount.py
│   │
│   ├── application/                     # Application Layer (Use Cases)
│   │   ├── __init__.py
│   │   ├── use_cases/                   # Use case implementations
│   │   │   ├── __init__.py
│   │   │   ├── user/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── register_user.py
│   │   │   │   ├── update_user_profile.py
│   │   │   │   └── get_user_by_id.py
│   │   │   └── order/
│   │   │       ├── __init__.py
│   │   │       ├── create_order.py
│   │   │       ├── cancel_order.py
│   │   │       └── get_order_history.py
│   │   ├── dtos/                        # Data Transfer Objects
│   │   │   ├── __init__.py
│   │   │   ├── user_dto.py
│   │   │   ├── order_dto.py
│   │   │   └── product_dto.py
│   │   ├── mappers/                     # DTO <-> Domain mappers
│   │   │   ├── __init__.py
│   │   │   ├── user_mapper.py
│   │   │   └── order_mapper.py
│   │   ├── interfaces/                  # Application service interfaces
│   │   │   ├── __init__.py
│   │   │   ├── email_service.py
│   │   │   ├── payment_gateway.py
│   │   │   └── event_bus.py
│   │   └── validators/                  # Input validation
│   │       ├── __init__.py
│   │       ├── user_validator.py
│   │       └── order_validator.py
│   │
│   ├── infrastructure/                  # Infrastructure Layer (External Concerns)
│   │   ├── __init__.py
│   │   ├── persistence/                 # Data access implementations
│   │   │   ├── __init__.py
│   │   │   ├── database.py
│   │   │   ├── models/                  # ORM models
│   │   │   │   ├── __init__.py
│   │   │   │   ├── user_model.py
│   │   │   │   └── order_model.py
│   │   │   ├── repositories/            # Repository implementations
│   │   │   │   ├── __init__.py
│   │   │   │   ├── sqlalchemy_user_repository.py
│   │   │   │   └── sqlalchemy_order_repository.py
│   │   │   ├── migrations/              # Database migrations
│   │   │   │   ├── alembic.ini
│   │   │   │   ├── env.py
│   │   │   │   └── versions/
│   │   │   │       └── 001_initial_schema.py
│   │   │   └── seeders/                 # Data seeders
│   │   │       ├── __init__.py
│   │   │       └── seed_initial_data.py
│   │   ├── messaging/                   # Message queue implementations
│   │   │   ├── __init__.py
│   │   │   ├── rabbitmq_event_bus.py
│   │   │   └── kafka_event_bus.py
│   │   ├── external_services/           # Third-party service clients
│   │   │   ├── __init__.py
│   │   │   ├── stripe_payment_gateway.py
│   │   │   ├── sendgrid_email_service.py
│   │   │   └── aws_s3_storage.py
│   │   ├── caching/                     # Cache implementations
│   │   │   ├── __init__.py
│   │   │   ├── redis_cache.py
│   │   │   └── in_memory_cache.py
│   │   ├── logging/                     # Logging configuration
│   │   │   ├── __init__.py
│   │   │   └── logger.py
│   │   └── config/                      # Configuration management
│   │       ├── __init__.py
│   │       ├── settings.py
│   │       └── dependency_injection.py
│   │
│   ├── presentation/                    # Presentation Layer (Interface Adapters)
│   │   ├── __init__.py
│   │   ├── api/                         # REST API
│   │   │   ├── __init__.py
│   │   │   ├── v1/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── routes/
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── users.py
│   │   │   │   │   ├── orders.py
│   │   │   │   │   └── health.py
│   │   │   │   ├── schemas/             # Pydantic schemas
│   │   │   │   │   ├── __init__.py
│   │   │   │   │   ├── user_schema.py
│   │   │   │   │   └── order_schema.py
│   │   │   │   ├── dependencies.py      # Route dependencies
│   │   │   │   └── middleware.py        # API middleware
│   │   │   └── main.py                  # FastAPI application
│   │   ├── graphql/                     # GraphQL API (optional)
│   │   │   ├── __init__.py
│   │   │   ├── schema.py
│   │   │   ├── resolvers/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── user_resolver.py
│   │   │   │   └── order_resolver.py
│   │   │   └── mutations/
│   │   │       ├── __init__.py
│   │   │       └── user_mutations.py
│   │   ├── cli/                         # Command-line interface
│   │   │   ├── __init__.py
│   │   │   ├── commands/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── migrate.py
│   │   │   │   └── seed.py
│   │   │   └── main.py
│   │   └── workers/                     # Background workers
│   │       ├── __init__.py
│   │       ├── celery_app.py
│   │       └── tasks/
│   │           ├── __init__.py
│   │           └── email_tasks.py
│   │
│   └── shared/                          # Shared utilities and common code
│       ├── __init__.py
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── date_utils.py
│       │   └── string_utils.py
│       ├── constants.py
│       └── types.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                      # Pytest configuration and fixtures
│   ├── unit/                            # Unit tests (isolated, fast)
│   │   ├── __init__.py
│   │   ├── domain/
│   │   │   ├── __init__.py
│   │   │   ├── entities/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_user.py
│   │   │   │   └── test_order.py
│   │   │   ├── value_objects/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── test_email.py
│   │   │   │   └── test_money.py
│   │   │   └── services/
│   │   │       ├── __init__.py
│   │   │       └── test_pricing_service.py
│   │   ├── application/
│   │   │   ├── __init__.py
│   │   │   └── use_cases/
│   │   │       ├── __init__.py
│   │   │       ├── user/
│   │   │       │   ├── __init__.py
│   │   │       │   └── test_register_user.py
│   │   │       └── order/
│   │   │           ├── __init__.py
│   │   │           └── test_create_order.py
│   │   └── infrastructure/
│   │       ├── __init__.py
│   │       └── persistence/
│   │           ├── __init__.py
│   │           └── repositories/
│   │               ├── __init__.py
│   │               └── test_user_repository.py
│   │
│   ├── integration/                     # Integration tests (multiple components)
│   │   ├── __init__.py
│   │   ├── conftest.py                  # Integration test fixtures
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── test_user_endpoints.py
│   │   │   └── test_order_endpoints.py
│   │   ├── persistence/
│   │   │   ├── __init__.py
│   │   │   └── test_database_operations.py
│   │   └── external_services/
│   │       ├── __init__.py
│   │       └── test_payment_gateway.py
│   │
│   ├── e2e/                             # End-to-end tests (full system)
│   │   ├── __init__.py
│   │   ├── conftest.py                  # E2E test fixtures
│   │   ├── test_user_registration_flow.py
│   │   ├── test_order_placement_flow.py
│   │   └── test_payment_processing_flow.py
│   │
│   ├── fixtures/                        # Test data fixtures
│   │   ├── __init__.py
│   │   ├── user_fixtures.py
│   │   └── order_fixtures.py
│   │
│   └── factories/                       # Test data factories
│       ├── __init__.py
│       ├── user_factory.py
│       └── order_factory.py
│
├── docker/
│   ├── Dockerfile                       # Production image
│   ├── Dockerfile.dev                   # Development image
│   ├── docker-compose.yml               # Multi-container orchestration
│   ├── docker-compose.dev.yml           # Development environment
│   ├── docker-compose.test.yml          # Test environment
│   └── entrypoint.sh                    # Container startup script
│
├── scripts/                             # Utility scripts
│   ├── setup.sh                         # Project setup
│   ├── run_tests.sh                     # Test execution
│   ├── lint.sh                          # Code linting
│   ├── format.sh                        # Code formatting
│   ├── type_check.sh                    # Type checking
│   ├── security_check.sh                # Security analysis
│   └── deploy.sh                        # Deployment
│
├── docs/                                # Documentation
│   ├── architecture/
│   │   ├── clean-architecture.md
│   │   ├── domain-model.md
│   │   ├── infrastructure.md
│   │   └── architecture-decisions.md    # Architecture decisions per user story
│   ├── releases/
│   │   └── CHANGELOG.md                 # Product-wide version history
│   ├── test-coverage.md                 # Overall test coverage report
│   ├── api/
│   │   └── openapi.yaml
│   └── development/
│       ├── setup.md
│       └── contributing.md
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── cd.yml
│       ├── lint.yml
│       └── static-analysis.yml
│
├── .gitignore
├── .dockerignore
├── .env.example                         # Example environment variables
├── .env.local                           # Local environment (gitignored)
├── .env.test                            # Test environment
│
├── pyproject.toml                       # Poetry dependencies & project config
├── poetry.lock                          # Locked dependencies (auto-generated)
├── poetry.toml                          # Poetry-specific configuration
│
├── pytest.ini                           # Pytest configuration
├── .coveragerc                          # Coverage configuration
├── coverage.xml                         # Coverage report (gitignored)
├── htmlcov/                             # HTML coverage report (gitignored)
│
├── mypy.ini                             # MyPy type checking configuration
├── .mypy_cache/                         # MyPy cache (gitignored)
│
├── .flake8                              # Flake8 linting configuration
├── .pylintrc                            # Pylint configuration
├── pyrightconfig.json                   # Pyright type checker configuration
│
├── .black                               # Black formatter configuration (optional)
├── .isort.cfg                           # isort import sorting configuration
├── setup.cfg                            # Alternative config file for tools
│
├── .bandit                              # Bandit security linting config
├── .safety-policy.yml                   # Safety dependency security config
│
├── .pre-commit-config.yaml              # Pre-commit hooks configuration
├── .editorconfig                        # Editor configuration
│
├── .ruff.toml                           # Ruff linter configuration
│
├── sonar-project.properties             # SonarQube static analysis config
├── .codeclimate.yml                     # Code Climate configuration
│
├── alembic.ini                          # Database migration config
│
├── README.md                            # Project overview and setup guide
└── LICENSE                              # License file
```

## Layer Descriptions

### Domain Layer (`src/domain/`)
**Purpose**: Core business logic and rules, completely independent of external concerns.

**Components**:
- **Entities**: Objects with identity that persist over time (e.g., User, Order)
- **Value Objects**: Immutable objects defined by their attributes (e.g., Email, Money)
- **Aggregates**: Cluster of entities and value objects with a single root entity
- **Domain Events**: Events that represent something significant in the domain
- **Repository Interfaces**: Contracts for data persistence (implementations in infrastructure)
- **Domain Services**: Business logic that doesn't naturally fit in entities
- **Specifications**: Encapsulated business rules for validation and querying
- **Exceptions**: Domain-specific error types

**Principles**:
- No dependencies on outer layers
- Pure business logic
- Framework-agnostic
- Highly testable

### Application Layer (`src/application/`)
**Purpose**: Orchestrates domain objects to perform application use cases.

**Components**:
- **Use Cases**: Application-specific business rules (e.g., RegisterUser, CreateOrder)
- **DTOs**: Simple data structures for transferring data between layers
- **Mappers**: Convert between DTOs and domain entities
- **Interfaces**: Contracts for external services (email, payment, etc.)
- **Validators**: Input validation logic

**Principles**:
- Depends only on domain layer
- Coordinates domain objects
- Contains application-specific logic
- No framework dependencies

### Infrastructure Layer (`src/infrastructure/`)
**Purpose**: Implements technical capabilities required by outer layers.

**Components**:
- **Persistence**: Database connections, ORM models, repository implementations
- **Messaging**: Message queue implementations (RabbitMQ, Kafka)
- **External Services**: Third-party API clients (Stripe, SendGrid, AWS)
- **Caching**: Cache implementations (Redis, in-memory)
- **Logging**: Logging configuration and utilities
- **Config**: Configuration management and dependency injection

**Principles**:
- Implements interfaces from inner layers
- Contains all technical details
- Framework-specific code lives here
- Swappable implementations

### Presentation Layer (`src/presentation/`)
**Purpose**: Exposes application functionality to external consumers.

**Components**:
- **API**: REST/GraphQL endpoints
- **Schemas**: Input/output validation schemas (Pydantic)
- **CLI**: Command-line interface
- **Workers**: Background job processors
- **Middleware**: Cross-cutting concerns (auth, logging)

**Principles**:
- Thin layer that delegates to application layer
- Handles HTTP/protocol concerns
- Input validation and serialization
- No business logic

## Testing Strategy

### Unit Tests (`tests/unit/`)
- Test individual components in isolation
- Mock all dependencies
- Fast execution (< 1 second per test)
- High coverage target (>90%)
- Test all business logic in domain and application layers

### Integration Tests (`tests/integration/`)
- Test multiple components working together
- Use real dependencies (database, cache) in test containers
- Test repository implementations with actual database
- Test API endpoints with full request/response cycle
- Moderate execution time (< 30 seconds per test suite)

### E2E Tests (`tests/e2e/`)
- Test complete user flows through the system
- All real services running (docker-compose)
- Simulate real user scenarios
- Slower execution (minutes)
- Fewer tests, focused on critical paths

## Configuration Files

### Poetry (`pyproject.toml`)
- Dependency management
- Build system configuration
- Project metadata
- Tool configurations (pytest, black, mypy, etc.)

### Static Analysis & Linting
- **mypy**: Type checking
- **flake8**: Style guide enforcement (PEP 8)
- **pylint**: Advanced linting and code quality
- **black**: Code formatting
- **isort**: Import sorting
- **bandit**: Security vulnerability scanning
- **safety**: Dependency security checking
- **ruff**: Fast Python linter (combines multiple tools)

### Pre-commit Hooks
Automatically run checks before commits:
- Format code with black
- Sort imports with isort
- Lint with flake8/ruff
- Type check with mypy
- Security scan with bandit

## Poetry Command Reference

```bash
# Project setup
poetry install                          # Install all dependencies
poetry install --no-dev                 # Install only production dependencies

# Dependency management
poetry add fastapi sqlalchemy pydantic  # Add production dependency
poetry add --group dev pytest mypy      # Add development dependency
poetry remove package-name              # Remove dependency
poetry update                           # Update all dependencies
poetry show                             # List installed packages

# Running commands
poetry run python src/presentation/api/main.py
poetry run pytest
poetry run mypy src/
poetry run black src/ tests/
poetry run flake8 src/ tests/
poetry run isort src/ tests/
poetry run bandit -r src/

# Shell
poetry shell                            # Activate virtual environment
exit                                    # Deactivate

# Build and publish
poetry build                            # Build distribution packages
poetry publish                          # Publish to PyPI
```

## Docker Support

### Development Environment
```bash
docker-compose -f docker-compose.dev.yml up
```

### Test Environment
```bash
docker-compose -f docker-compose.test.yml up
poetry run pytest
```

### Production Environment
```bash
docker-compose up -d
```

## Key Architectural Principles

1. **Dependency Rule**: Dependencies point inward. Outer layers depend on inner layers, never the reverse.

2. **Interface Adapters**: Use interfaces to invert dependencies (Dependency Inversion Principle).

3. **Single Responsibility**: Each layer has a single, well-defined responsibility.

4. **Test-Driven Development**: Write tests first, especially for domain logic.

5. **Immutability**: Prefer immutable value objects and data structures.

6. **Domain-First**: Start with domain modeling, add infrastructure last.

7. **Explicit Architecture**: Make architectural boundaries clear through folder structure.

8. **Configuration as Code**: All configuration in version control (except secrets).


## Getting Started

1. Copy this structure to your new project
2. Configure `pyproject.toml` with your project details
3. Run `poetry install` to set up dependencies
4. Start with domain modeling in `src/domain/`
5. Write tests in `tests/unit/domain/`
6. Implement use cases in `src/application/`
7. Add infrastructure implementations
8. Build presentation layer last

## Best Practices

- **Always write tests first** for domain logic
- **Keep domain layer pure** - no external dependencies
- **Use dependency injection** for all cross-layer dependencies
- **Make interfaces explicit** - define contracts in application layer
- **Keep use cases focused** - one use case = one business operation
- **Use value objects** for domain concepts (Email, Money, etc.)
- **Validate at boundaries** - presentation and application layers
- **Use DTOs** for data transfer between layers
- **Run static analysis** before every commit
- **Maintain high test coverage** (>90% for domain/application layers)

## References

- Clean Architecture by Robert C. Martin
- Domain-Driven Design by Eric Evans
- Implementing Domain-Driven Design by Vaughn Vernon
- Python Clean Architecture: https://github.com/iktakahiro/dddpy
