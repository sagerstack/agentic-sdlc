# Python Code Quality Standards

## Purpose

Comprehensive code quality standards for Python backend development. Used by py-developer agent to ensure production-grade, maintainable code following SOLID principles and modern Python best practices.

---

## Table of Contents

**Core Principles & Standards**
1. [SOLID Principles](#solid-principles) - SRP, OCP, LSP, ISP, DIP
2. [Type Hints](#type-hints-required) - Required for all functions, protocols, generics
3. [Docstrings](#docstrings-required-for-public-apis) - Google-style documentation
4. [Error Handling](#error-handling) - Custom exceptions, validation patterns
5. [Logging](#logging-not-print) - Structured logging with correlation IDs
6. [Immutability](#immutability-value-objects) - Value objects and frozen dataclasses
7. [Dependency Injection](#dependency-injection) - Constructor injection patterns
8. [Best Practices](#best-practices) - Small functions, composition, async/await
9. [Code Formatting & Linting](#code-formatting--linting) - Black, Ruff, MyPy

**Data Quality & API Integration Policies**
10. [NO_MOCK_DATA_POLICY](#no_mock_data_policy) - Use real API data, verify with production
11. [NO_HARDCODING_POLICY](#no_hardcoding_policy---dynamic-discovery) - Dynamic discovery from DEX APIs
12. [NO_ARTIFICIAL_LIMITS_POLICY](#no_artificial_limits_policy---justify-all-data-limits) - Justify all numeric limits
13. [COMPLETE_API_DATA_POLICY](#complete_api_data_policy---dont-discard-api-fields) - Don't discard API response fields
14. [FILTER_BEFORE_LIMIT_POLICY](#filter_before_limit_policy---business-rules-first) - Business rules before technical limits

**Configuration Management**
15. [Configuration Management Standards](#configuration-management-standards) - 12-factor app, secrets vs configuration

**Testing & Validation**
16. [Testing Standards](#testing-standards) - Minimum 95% coverage, test naming conventions
17. [Pre-commit Hooks](#pre-commit-hooks) - Automated quality checks

---

## SOLID Principles

Apply these principles throughout all code:

### 1. Single Responsibility Principle (SRP)
**Rule**: Each class/function has one reason to change

**Good Example**:
```python
# GOOD: Separate responsibilities
class User:
    def change_email(self, new_email: Email) -> None:
        self.email = new_email

class EmailNotificationService:
    def send_email_changed_notification(self, user: User) -> None:
        # Send notification logic
```

**Bad Example**:
```python
# BAD: User class handles notifications
class User:
    def change_email(self, new_email: Email) -> None:
        self.email = new_email
        self._send_notification()  # Mixed responsibility!
```

---

### 2. Open/Closed Principle (OCP)
**Rule**: Open for extension, closed for modification

**Good Example**:
```python
# GOOD: Extend via inheritance/composition
class DiscountCalculator(ABC):
    @abstractmethod
    def calculate(self, amount: Money) -> Money:
        pass

class PercentageDiscount(DiscountCalculator):
    def __init__(self, rate: Decimal):
        self.rate = rate

    def calculate(self, amount: Money) -> Money:
        return amount.multiply(self.rate)
```

**Bad Example**:
```python
# BAD: Modify existing class for new behavior
class DiscountCalculator:
    def calculate(self, amount: Money, discount_type: str) -> Money:
        if discount_type == "percentage":
            # logic
        elif discount_type == "fixed":
            # logic
        elif discount_type == "tiered":  # Added later - violates OCP
            # logic
```

---

### 3. Liskov Substitution Principle (LSP)
**Rule**: Subtypes must be substitutable for base types

**Good Example**:
```python
# GOOD: Subtype preserves base contract
class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount: Money) -> PaymentResult:
        pass

class StripePaymentProcessor(PaymentProcessor):
    def process(self, amount: Money) -> PaymentResult:
        # Always returns PaymentResult, never raises unexpected errors
        return PaymentResult(success=True, transaction_id="...")
```

**Bad Example**:
```python
# BAD: Subtype violates base contract
class BrokenPaymentProcessor(PaymentProcessor):
    def process(self, amount: Money) -> str:  # Wrong return type!
        return "processed"  # Violates LSP
```

---

### 4. Interface Segregation Principle (ISP)
**Rule**: Many focused interfaces over one general interface

**Good Example**:
```python
# GOOD: Focused interfaces
class Readable(Protocol):
    def read(self) -> bytes:
        ...

class Writable(Protocol):
    def write(self, data: bytes) -> None:
        ...

class FileStream(Readable, Writable):
    # Implements both when needed
    pass

class ReadOnlyFile(Readable):
    # Only implements read
    pass
```

**Bad Example**:
```python
# BAD: Fat interface
class FileOperations(Protocol):
    def read(self) -> bytes: ...
    def write(self, data: bytes) -> None: ...
    def compress(self) -> None: ...
    def encrypt(self) -> None: ...
    # ReadOnlyFile forced to implement write, compress, encrypt!
```

---

### 5. Dependency Inversion Principle (DIP)
**Rule**: Depend on abstractions, not concretions

**Good Example**:
```python
# GOOD: Depends on interface
class RegisterUserUseCase:
    def __init__(
        self,
        user_repository: UserRepository,  # Interface (Protocol)
        event_publisher: EventPublisher   # Interface (Protocol)
    ):
        self._user_repository = user_repository
        self._event_publisher = event_publisher
```

**Bad Example**:
```python
# BAD: Depends on concrete implementation
class RegisterUserUseCase:
    def __init__(self):
        self._user_repository = PostgresUserRepository()  # Concrete!
        self._event_publisher = RabbitMQPublisher()      # Concrete!
```

---

## Type Hints (Required)

### All Functions Must Have Type Hints

**Required**:
- Parameter types
- Return types
- Optional types where applicable
- Generic types for collections

**Good Example**:
```python
from typing import Optional, List
from decimal import Decimal

def calculate_discount(amount: Decimal, rate: Decimal, max: Optional[Decimal] = None) -> Decimal:
    discount = amount * rate
    return min(discount, max) if max else discount

def find_users_by_tier(tier: SubscriptionTier) -> List[User]:
    pass
```

### Protocol for Interfaces

Use `Protocol` from `typing` for interface definitions:

```python
from typing import Protocol, Optional

class UserRepository(Protocol):
    """Repository interface for User aggregate."""

    def get_by_id(self, user_id: UUID) -> Optional[User]:
        """Retrieve user by ID."""
        ...

    def save(self, user: User) -> None:
        """Persist user."""
        ...
```

### Generic Types

Use generics for reusable containers:

```python
from typing import TypeVar, Generic, List

T = TypeVar('T')

class Result(Generic[T]):
    """Generic result type for success/failure."""

    def __init__(self, value: Optional[T] = None, error: Optional[str] = None):
        self.value = value
        self.error = error

    def is_success(self) -> bool:
        return self.error is None

# Usage
def register_user(command: RegisterUserCommand) -> Result[User]:
    # Returns Result[User] on success, Result with error on failure
    pass
```

---

## Docstrings (Required for Public APIs)

### Google-Style Docstrings

**Required sections**:
- Summary line (imperative mood)
- Detailed description (if needed)
- Args (parameter descriptions)
- Returns (return value description)
- Raises (exceptions raised)
- Examples (for complex functions)

**Example**:
```python
def register_user(email: str, name: str, birth_date: Optional[datetime] = None) -> User:
    """Register a new user in the system.

    Args:
        email: User's email address. Must be valid format.
        name: User's full name (2-100 characters).
        birth_date: Optional date of birth. Must be 18+.

    Returns:
        Newly created User entity with UUID.

    Raises:
        InvalidEmailError: Invalid email format or already registered.
        ValidationError: Name doesn't meet length requirements.
        UnderageError: User under 18 years old.
    """
    pass
```

### Class Docstrings

```python
class User:
    """User aggregate root managing authentication and profile.

    Attributes:
        id: Unique identifier (UUID v4).
        email: Validated email (Email value object).
        name: Full name (2-100 characters).
        subscription_tier: Current subscription level.

    Invariants:
        - Email unique across users
        - Active users have valid subscription
    """
```

### Module Docstrings

```python
"""User domain module with aggregate root, value objects, and domain events.

Business rules: Email uniqueness, password strength, age 18+.
"""
```

---

## Error Handling

### Custom Exception Hierarchies

Create domain-specific exceptions:

```python
class DomainException(Exception):
    """Base exception for all domain errors."""

class ValidationError(DomainException):
    """Domain validation failures."""

class InvalidEmailError(ValidationError):
    """Invalid email format."""

class BusinessRuleViolation(DomainException):
    """Business rule violations."""

class EntityNotFoundError(DomainException):
    """Entity not found in repository."""
```

### Validation in Value Objects

**Always validate in `__post_init__`**:

```python
@dataclass(frozen=True)
class Email:
    value: str

    def __post_init__(self):
        if not self._is_valid(self.value):
            raise InvalidEmailError(f"Invalid email: {self.value}")

    @staticmethod
    def _is_valid(email: str) -> bool:
        return bool(re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email))
```

### Error Messages

Provide actionable error messages:

```python
# GOOD: Actionable error message
if len(password) < 8:
    raise ValidationError(
        f"Password must be at least 8 characters. "
        f"Provided password length: {len(password)}"
    )

# BAD: Vague error message
if len(password) < 8:
    raise ValidationError("Invalid password")
```

---

## Logging (Not Print)

### Structured Logging with Correlation IDs

**Always use `logging` module, never `print`**:

```python
import logging
import uuid

logger = logging.getLogger(__name__)

def execute(self, command: RegisterUserCommand) -> Result[UserDTO]:
    correlation_id = str(uuid.uuid4())

    logger.info("Executing RegisterUser", extra={
        "correlation_id": correlation_id, "email": command.email, "operation": "user_registration"
    })

    try:
        user = self._create_user(command)
        logger.info("User registered", extra={"correlation_id": correlation_id, "user_id": str(user.id)})
        return Result(value=user)
    except ValidationError as e:
        logger.warning("Validation failed", extra={"correlation_id": correlation_id, "error": str(e)})
        return Result(error=str(e))
    except Exception as e:
        logger.error("Unexpected error", extra={
            "correlation_id": correlation_id, "error_type": type(e).__name__
        }, exc_info=True)
        raise
```

### Log Levels

| Level | Use Case | Example |
|-------|----------|---------|
| **DEBUG** | Detailed diagnostic info | Variable values, internal state |
| **INFO** | General informational events | Use case execution, domain events |
| **WARNING** | Recoverable issues | Validation failures, retries |
| **ERROR** | Errors requiring attention | Unexpected exceptions, integration failures |
| **CRITICAL** | System failure | Database unreachable, fatal errors |

### Sanitize Sensitive Data

**Never log passwords, tokens, or PII without redaction**:

```python
# GOOD: Redact sensitive data
logger.info("User authenticated", extra={"user_id": user.id, "email": self._redact_email(user.email)})

# BAD: Exposes sensitive data
logger.info(f"User: {user.email}, password: {password}")  # NEVER!

def _redact_email(email: str) -> str:
    user, domain = email.split('@')
    return f"{user}@*****.{domain.split('.')[-1]}"  # user@*****.com
```

---

## Immutability (Value Objects)

### Use `frozen=True` for Value Objects

```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass(frozen=True)
class Money:
    """Immutable money value object."""
    amount: Decimal
    currency: str = "USD"

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("Money amount cannot be negative")

    def add(self, other: "Money") -> "Money":
        """Return new Money instance (no mutation)."""
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)

# Usage
price = Money(Decimal("10.00"))
tax = price.multiply(Decimal("0.08"))
total = price.add(tax)  # Creates NEW Money instance
# price is unchanged (immutable)
```

### Avoid Mutable Default Arguments

```python
# BAD: Mutable default argument
def add_item(self, items=[]):  # DANGEROUS!
    items.append("new")
    return items

# GOOD: Use None and create new list
def add_item(self, items: Optional[List[str]] = None) -> List[str]:
    if items is None:
        items = []
    items.append("new")
    return items
```

---

## Dependency Injection

### Constructor Injection Throughout

**Always inject dependencies via constructor**:

```python
# Application layer use case
class RegisterUserUseCase:
    """Use case for user registration."""

    def __init__(
        self,
        user_repository: UserRepository,    # Interface (port)
        event_publisher: EventPublisher,    # Interface (port)
        email_service: EmailService,        # Interface (port)
        password_hasher: PasswordHasher     # Interface (port)
    ):
        self._user_repository = user_repository
        self._event_publisher = event_publisher
        self._email_service = email_service
        self._password_hasher = password_hasher

    def execute(self, command: RegisterUserCommand) -> Result[UserDTO]:
        """Execute user registration."""
        # Use injected dependencies
        if self._user_repository.exists(Email(command.email)):
            return Result(error="Email already registered")

        # ... business logic
```

### Avoid Service Locator Pattern

```python
# BAD: Service locator
class RegisterUserUseCase:
    def execute(self, command):
        ServiceLocator.get("UserRepository").save(user)  # AVOID!

# GOOD: Dependency injection
class RegisterUserUseCase:
    def __init__(self, user_repository: UserRepository):
        self._user_repository = user_repository
```

---

## Best Practices

### Small, Focused Functions

**Keep functions under 20 lines, single responsibility**:

```python
# GOOD: Small, focused functions
def validate_email(email: str) -> Email:
    if not _is_valid_email_format(email):
        raise InvalidEmailError(email)
    return Email(email)

def create_user(email: Email, name: str, password_hash: str) -> User:
    return User(email=email, name=name, password_hash=password_hash,
                subscription_tier=SubscriptionTier.FREE, is_active=True)

# BAD: One giant function combining validation, hashing, creation, event emission
```

### Prefer Composition Over Inheritance

```python
# GOOD: Composition
class OrderPricing:
    def __init__(self, tax_calculator: TaxCalculator, discount_calculator: DiscountCalculator):
        self._tax_calculator = tax_calculator
        self._discount_calculator = discount_calculator

    def calculate_total(self, order: Order) -> Money:
        subtotal = order.subtotal()
        discount = self._discount_calculator.calculate(subtotal)
        return subtotal - discount + self._tax_calculator.calculate(subtotal - discount)

# AVOID: Deep inheritance like DiscountedTaxableOrder(TaxableOrder, DiscountableOrder)
```

### Use Dataclasses/Pydantic for Data Structures

```python
from dataclasses import dataclass
from pydantic import BaseModel, EmailStr

# Simple data transfer object
@dataclass
class UserDTO:
    id: str
    email: str
    name: str
    subscription_tier: str

# Pydantic for API validation
class RegisterUserRequest(BaseModel):
    email: EmailStr
    name: str
    password: str
```

### Generators for Large Datasets

```python
# GOOD: Generator for memory efficiency
def find_active_users(repository: UserRepository) -> Iterator[User]:
    """Stream users without loading all into memory."""
    batch_size, offset = 100, 0
    while batch := repository.find_active(limit=batch_size, offset=offset):
        for user in batch:
            yield user
        offset += batch_size

# Usage: process one at a time
for user in find_active_users(repo):
    process(user)
```

### Async/Await for I/O-Bound Operations

```python
import httpx

async def fetch_user_data(user_id: UUID) -> Dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/users/{user_id}")
        return response.json()

async def enrich_users(users: List[User]) -> List[Dict]:
    return await asyncio.gather(*[fetch_user_data(u.id) for u in users])
```

---

## Code Formatting & Linting

### Black (Code Formatter)

**Auto-format all code**:
```bash
poetry run black src/ tests/
```

**Configuration** (pyproject.toml):
```toml
[tool.black]
line-length = 100
target-version = ['py311']
```

### Ruff (Linter)

**Replaces flake8, isort, pylint**:
```bash
poetry run ruff check src/ tests/ --fix
```

**Configuration** (.ruff.toml):
```toml
line-length = 100
target-version = "py311"
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
]
```

### MyPy (Static Type Checker)

**Run in strict mode**:
```bash
poetry run mypy src/ --strict
```

**Configuration** (mypy.ini):
```ini
[mypy]
python_version = 3.11
strict = True
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
```

---

## Data Quality & API Integration Standards

### NO_MOCK_DATA_POLICY

**Purpose**: Ensure implementation uses real data from external APIs as specified in technical research, preventing silent substitution with mock/placeholder data.

#### When to Use Real Data vs Mocks

| Test Type | Data Source | Rationale |
|-----------|-------------|-----------|
| **Unit Tests** | Mock external dependencies | Isolated component testing, fast execution |
| **Integration Tests** | Real APIs (if .env.local exists) | Verify actual integration contracts |
| **Acceptance Tests** | Real APIs (if .env.local exists) | Validate end-to-end workflows |
| **Live Tests** | Real APIs (REQUIRED) | Production-like validation |
| **E2E Tests** | Real APIs OR seeded test data | Complete workflow verification |

#### Production Code Requirements

**CRITICAL**: Application and Infrastructure layers MUST use real API data extraction:

1. **Follow Tech Research Field Mappings**: Extract exact fields documented in technical research
   ```python
   # GOOD: Follows tech research field mapping
   # Tech research documented: data[1][i].midPx → direct_price
   async def _cache_hyperliquid_pool(self, meta: Dict, asset_ctx: Dict) -> None:
       mid_px = asset_ctx.get("midPx")  # ✅ Exact field from research
       if mid_px is None:
           return
       mid_px_decimal = Decimal(str(mid_px))
       # ... use mid_px_decimal

   # BAD: Ignores documented field, uses different data
   async def _cache_hyperliquid_pool(self, meta: Dict, asset_ctx: Dict) -> None:
       volume = asset_ctx.get("dayNtlVlm")  # ❌ Wrong field - research said midPx!
       # ... missing price extraction
   ```

2. **No Placeholder Data in Production Paths**:
   ```python
   # BAD: Placeholder data in production code
   def get_price(self, token: str) -> Decimal:
       return Decimal("100.0")  # ❌ FORBIDDEN - placeholder price

   # GOOD: Real API data extraction
   async def get_price(self, token: str) -> Decimal:
       response = await self._client.get(f"/api/prices/{token}")
       data = response.json()
       return Decimal(str(data["price"]))  # ✅ Real API data
   ```

3. **Validation Tasks Required**:
   - Implementation plan MUST include: "Verify extracted fields match API research documentation"
   - Add assertions in tests: `assert extracted_field == expected_from_api_research`
   - Log field extraction for debugging: `logger.debug("Extracted midPx", midPx=mid_px, source="asset_ctx")`

#### Integration Test Data Strategy

**When .env.local exists with credentials**:

```python
# GOOD: Use real API when credentials available
import os
import pytest

@pytest.mark.integration
async def test_hyperliquid_price_fetcher_integration():
    """Integration test with REAL Hyperliquid API."""
    # Use real API client (not mock)
    api_key = os.getenv("HYPERLIQUID_API_KEY")
    if not api_key:
        pytest.skip("HYPERLIQUID_API_KEY not in .env.local")

    fetcher = HyperliquidPriceFetcher(api_key=api_key)

    # Call REAL API
    prices = await fetcher.fetch_prices()

    # Verify real data structure matches tech research
    assert len(prices) > 0, "Should fetch real prices from live API"
    assert all("midPx" in p for p in prices), "API should return midPx field per tech research"
```

```python
# BAD: Mock when credentials exist
@pytest.mark.integration
async def test_hyperliquid_price_fetcher_integration(mocker):
    # ❌ WRONG: Mocking when .env.local has real credentials
    mocker.patch("httpx.AsyncClient.post", return_value=mock_response)
    # This is a UNIT test disguised as integration test!
```

#### Field Extraction Validation

**REQUIRED**: When tech research documents field mappings, implementation MUST extract those exact fields:

**Tech Research Documentation Example**:
```markdown
**Field Extraction Mapping**:
- `data[0].universe[i].name` → `base_token`
- `data[1][i].midPx` → `direct_price`
- `data[1][i].dayNtlVlm` → `liquidity_usd`
```

**Implementation MUST Match**:
```python
async def _cache_hyperliquid_pool(self, meta: Dict[str, Any], asset_ctx: Dict[str, Any]) -> None:
    # ✅ Extract exact fields from tech research
    symbol = meta.get("name")  # Matches: data[0].universe[i].name → base_token
    mid_px = asset_ctx.get("midPx")  # Matches: data[1][i].midPx → direct_price
    liquidity = asset_ctx.get("dayNtlVlm")  # Matches: data[1][i].dayNtlVlm → liquidity_usd

    # Validate extraction matches research
    if mid_px is None:
        logger.warning("midPx field missing - expected per tech research", symbol=symbol)
        return

    # Convert to domain types
    direct_price = Decimal(str(mid_px))
    liquidity_usd = Decimal(str(liquidity)) if liquidity else Decimal("0")
```

#### Anti-Patterns to Avoid

**❌ Silent Field Substitution**:
```python
# Tech research said: Extract data[1][i].midPx for price
# Implementation did: Extract data[1][i].dayNtlVlm for volume only
# RESULT: Price data missing, user story failed silently
```

**❌ Vague Implementation Tasks**:
```python
# BAD: "Query Hyperliquid API for prices"
# GOOD: "Extract midPx from data[1][i] per tech research section 2.3"
```

**❌ Mock Data in Integration Tests**:
```python
# BAD: .env.local exists but test uses mocks
# GOOD: .env.local exists → test calls REAL API
```

---

### NO_HARDCODING_POLICY - Dynamic Discovery

**Purpose**: Prevent hardcoded assumptions about token pairings, quote tokens, or market structure that break when DEX data doesn't match expectations.

#### Forbidden Patterns

**❌ Hardcoded Quote Tokens**:
```python
# BAD: Assumes all tokens pair with specific quote token
price_seeding_service = PriceSeedingService(
    quote_symbol="WSOL"  # ❌ FORBIDDEN - hardcoded assumption
)

# User queries BTC price
# System tries: BTC/WSOL (doesn't exist)
# Result: No price found, but BTC/USD pool exists in cache!
```

**❌ Hardcoded Token Pairs**:
```python
# BAD: Hardcoded list of pairs to query
TRADING_PAIRS = [
    ("BTC", "USDC"),
    ("ETH", "USDC"),
    ("SOL", "USDC"),
]

for base, quote in TRADING_PAIRS:
    pool = await fetch_pool(base, quote)  # ❌ Assumes these pairs exist
```

**❌ Token-First Data Flow**:
```python
# BAD: Fetch tokens, then try to find pools
async def seed_prices():
    tokens = await fetch_all_tokens()  # Step 1: Get tokens
    for token in tokens:
        # Step 2: Assume token pairs with USDC
        pool = await get_pool(token.symbol, "USDC")  # ❌ Hardcoded quote
        if pool:
            price = pool.calculate_price()
```

#### Correct Patterns

**✅ Pool-First Data Flow**:
```python
# GOOD: Fetch all pools, derive everything else from pool data
async def seed_prices():
    # Step 1: Fetch ALL pools from DEX APIs (no pair filtering)
    all_pools = await fetch_all_pools_from_dex()

    # Step 2: Filter pools by volume/liquidity criteria
    filtered_pools = [p for p in all_pools if p.liquidity_usd >= MIN_VOLUME]

    # Step 3: Cache pools (with direct_price from API)
    for pool in filtered_pools:
        await pool_cache.set_pool(pool)

    # Step 4: Extract unique tokens from pool pairs
    unique_tokens = set()
    for pool in filtered_pools:
        unique_tokens.add(pool.base_token)
        unique_tokens.add(pool.quote_token)

    # Step 5: Seed prices directly from pool data
    for pool in filtered_pools:
        pair = f"{pool.base_token}:{pool.quote_token}"
        price = pool.direct_price or pool.calculate_price()
        await price_cache.set_price(pair, price)
```

**✅ Dynamic Quote Token Discovery**:
```python
# GOOD: Discover quote tokens from actual pool data
async def discover_price(base_token: str):
    # Try common quote tokens dynamically (from pool cache analysis)
    quote_tokens = await get_available_quote_tokens_from_cache()

    for quote_token in quote_tokens:
        pool = await pool_cache.get_pool(base_token, quote_token)
        if pool:
            return pool.direct_price or pool.calculate_price()

    # If no direct pair, try reverse pairs (US-052)
    for quote_token in quote_tokens:
        reverse_pool = await pool_cache.get_pool(quote_token, base_token)
        if reverse_pool:
            reverse_price = reverse_pool.direct_price or reverse_pool.calculate_price()
            return Decimal(1) / reverse_price  # Invert

    return None  # No pool found for any pairing
```

**✅ Data Flow Architecture**:
```python
# CORRECT FLOW:
# 1. DEX API → ALL pools/pairs
# 2. Filter by volume/liquidity
# 3. Cache pools (primary data source)
# 4. Derive tokens from pools (secondary)
# 5. Derive prices from pools (secondary)

class PoolBasedDataAggregator:
    """Aggregates data starting from pool/pair data, not tokens."""

    async def initialize(self):
        # Fetch all pools from DEX APIs
        raydium_pools = await self.fetch_raydium_pools()  # All pairs
        orca_pools = await self.fetch_orca_pools()        # All pairs
        hyperliquid_pools = await self.fetch_hyperliquid_pools()  # All perps

        all_pools = raydium_pools + orca_pools + hyperliquid_pools

        # Filter by volume
        high_volume_pools = [p for p in all_pools if p.liquidity_usd >= 50000]

        # Cache pools
        for pool in high_volume_pools:
            await self.pool_cache.set_pool(pool)

        # Extract unique tokens from pools
        unique_tokens = self.extract_tokens_from_pools(high_volume_pools)
        for token in unique_tokens:
            await self.token_cache.set_token(token)

        # Seed prices from pools
        for pool in high_volume_pools:
            pair = f"{pool.base_token}:{pool.quote_token}"
            price = pool.direct_price or pool.calculate_price()
            await self.price_cache.set_price(pair, price)
```

#### Validation Checklist

Before deploying data aggregation code, verify:

- [ ] No hardcoded quote tokens in configuration (e.g., `quote_symbol="WSOL"`)
- [ ] No hardcoded token pairs in code (e.g., `PAIRS = [("BTC", "USDC")]`)
- [ ] Pools are fetched BEFORE tokens (pool-first data flow)
- [ ] Volume filtering applied to POOLS, not tokens
- [ ] Prices derived from pool data (direct_price or calculate_price())
- [ ] Token extraction happens FROM pools, not separate metadata APIs
- [ ] No assumptions about which tokens pair with which quote tokens

#### Real-World Example: Price Seeding Failure

**What Went Wrong** (actual production issue):
```python
# main.py - WRONG IMPLEMENTATION
price_seeding_service = PriceSeedingService(
    quote_symbol="WSOL"  # ❌ Hardcoded assumption
)

# Price seeding queries:
# - BTC/WSOL → NOT FOUND (pool is BTC/USD)
# - ETH/WSOL → NOT FOUND (pool is ETH/USD)
# - SOL/WSOL → NOT FOUND (pool is SOL/USDC)
# Result: 0 prices seeded out of 272 tokens
```

**Correct Implementation**:
```python
# Fetch all pools first
pools = await fetch_all_pools_from_dexes()
filtered_pools = [p for p in pools if p.liquidity_usd >= 50000]

# Cache pools
for pool in filtered_pools:
    await pool_cache.set_pool(pool)

# Seed prices directly from pool data
for pool in filtered_pools:
    pair = f"{pool.base_token}:{pool.quote_token}"
    price = pool.direct_price  # From API (e.g., midPx for Hyperliquid)
    await price_cache.set_price(pair, price)

# Result: 354 prices seeded from real pool data
```

---

### NO_ARTIFICIAL_LIMITS_POLICY - Justify All Data Limits

**Purpose**: Prevent arbitrary numeric limits that artificially restrict data processing without technical justification.

**Extension to NO_HARDCODING_POLICY**: Hardcoding includes not just strings, but also numeric limits and pagination defaults.

#### Forbidden Patterns

**❌ Hardcoded Data Limits Without Justification**:
```python
# BAD: Arbitrary limit without explanation
max_pools_per_dex = 200  # ❌ Why 200? Fetches 0.03% of available 702K pools

# BAD: Hidden hardcoding in default parameters
def fetch_pools(self, max_pools: int = 50):  # ❌ Default masks hardcoding
    return pools[:max_pools]

# BAD: Artificial pagination
def get_items(self, page_size: int = 100):  # ❌ No justification for limit
    return items[:page_size]
```

**❌ Limit Before Filter (Wrong Order)**:
```python
# BAD: Apply limit BEFORE business rules
sorted_pools = sorted(all_pools, key=lambda p: p.liquidity, reverse=True)
limited = sorted_pools[:200]  # ❌ Limit first!
filtered = [p for p in limited if p.volume >= 50000]  # Filter second ❌
```

#### Correct Patterns

**✅ Unlimited with Business Rule Filtering**:
```python
# GOOD: No artificial limit, filter by business criteria only
max_pools_per_dex = None  # Unlimited, filter by $50K liquidity criterion

async def fetch_pools(self, min_liquidity_usd: float = 50000.0):
    all_pools = await self._fetch_all_from_api()
    filtered = [p for p in all_pools if p.liquidity >= min_liquidity_usd]
    return filtered  # Returns however many meet criteria
```

**✅ Technical Limit with Justification**:
```python
# GOOD: Limit with inline comment explaining technical constraint
MAX_POOLS = 10000  # Technical limit: API rate limit 100 req/min × 100 pools/req

async def fetch_pools(self, min_liquidity_usd: float):
    all_pools = await self._fetch_all_from_api()
    filtered = [p for p in all_pools if p.liquidity >= min_liquidity_usd]
    return filtered[:MAX_POOLS]  # Apply technical limit last
```

**✅ Filter Before Limit (Correct Order)**:
```python
# GOOD: Filter → Sort → Limit
# Step 1: Filter by business criteria FIRST
filtered = [p for p in all_pools if p.volume >= 50000]

# Step 2: Sort by priority
sorted_pools = sorted(filtered, key=lambda p: p.liquidity, reverse=True)

# Step 3: Limit ONLY if needed (with justification)
selected = sorted_pools[:10000]  # Memory constraint: 1GB heap max
```

#### Real-World Example: Artificial Pool Limit

**What Went Wrong**:
```python
# Raydium API returns 702,224 pools
# Code fetched only 200 pools (0.03% coverage!)
pool_data_fetcher = PoolDataFetcherService(
    max_pools_per_dex=200  # ❌ No justification, arbitrary limit
)

# Result: Missed 702,024 pools that could have met $50K threshold
```

**Correct Implementation**:
```python
# Fetch ALL pools, filter by business criteria
pool_data_fetcher = PoolDataFetcherService(
    max_pools_per_dex=None,  # Unlimited, filter by criteria only
    min_liquidity_usd=50000.0,  # Business rule: $50K threshold
)

# Result: 1,709 pools cached (properly filtered by $50K rule)
```

---

### COMPLETE_API_DATA_POLICY - Don't Discard API Fields

**Purpose**: Ensure API response data is used completely, not partially extracted and discarded.

#### Forbidden Patterns

**❌ Extracting Only Metadata, Discarding Entities**:
```python
# BAD: API returns complete pool data, code extracts only token metadata
def _parse_pair_to_metadata(self, pair: dict) -> TokenMetadata:
    """
    API response:
    {
      "name": "USDT/USDC",        // Trading pair
      "baseMint": "0x123...",      // Base token
      "quoteMint": "0x456...",     // Quote token ← DISCARDED!
      "price": 0.5,                // Direct price ← DISCARDED!
      "liquidity": 50000000,       // Pool liquidity ← DISCARDED!
      "volume24h": 10000000        // Volume (extracted ✓)
    }
    """
    # Extract ONLY token metadata
    return TokenMetadata(
        symbol=pair["name"].split("/")[0],
        volume_24h_usd=pair["volume24h"],  # ← Only field extracted!
    )
    # ❌ quote_token DISCARDED
    # ❌ price DISCARDED
    # ❌ pool_address DISCARDED
    # ❌ reserves DISCARDED
```

**❌ Breaking Data Relationships**:
```python
# BAD: Pool contains tokens + prices, but only tokens cached
pools = await fetch_pools_from_api()  # Returns complete pool data

# Extract tokens only, discard pools ❌
tokens = [Token(symbol=p.base_token) for p in pools]
await token_cache.set_many(tokens)

# Later: Try to seed prices
# ERROR: No pool data! Can't calculate prices!
```

#### Correct Patterns

**✅ Cache Complete Entities, Then Derive**:
```python
# GOOD: Cache pools FIRST (source of truth), derive tokens/prices FROM pools
async def initialize(self):
    # Step 1: Fetch complete pool data
    pools = await fetch_pools_from_api()

    # Step 2: Cache COMPLETE pools (all fields)
    for pool in pools:
        await self._pool_cache.set_pool(
            pool_address=pool.pool_address,
            base_token=pool.base_token,     # ✓
            quote_token=pool.quote_token,   # ✓
            direct_price=pool.price,        # ✓
            reserves_base=pool.reserves[0], # ✓
            reserves_quote=pool.reserves[1],# ✓
            liquidity_usd=pool.liquidity,   # ✓
        )

    # Step 3: Extract tokens FROM cached pools
    unique_tokens = set()
    for pool in pools:
        unique_tokens.add(pool.base_token)
        unique_tokens.add(pool.quote_token)

    # Step 4: Seed prices FROM cached pools
    for pool in pools:
        price = pool.direct_price  # Available because we cached it!
        await price_cache.set_price(f"{pool.base_token}:{pool.quote_token}", price)
```

**✅ Use ALL Relevant API Fields**:
```python
# GOOD: Extract all relevant fields from API response
async def _cache_pool_from_api(self, pool_data: dict):
    """Cache complete pool data from API response."""
    await self._pool_cache.seed_pool(
        pool_address=pool_data.get("ammId"),           # ✓
        dex="raydium",                                  # ✓
        chain="solana",                                 # ✓
        base_token=pool_data["name"].split("/")[0],    # ✓
        quote_token=pool_data["name"].split("/")[1],   # ✓ Not discarded!
        reserves_base=pool_data.get("tokenAmountCoin"),# ✓
        reserves_quote=pool_data.get("tokenAmountPc"), # ✓
        liquidity_usd=pool_data.get("liquidity"),      # ✓
        fee_tier=Decimal("0.0025"),                    # ✓
        direct_price=pool_data.get("price"),           # ✓ Not discarded!
    )
```

#### Real-World Example: Pool Data Discarded

**What Went Wrong**:
```python
# raydium_metadata_repository.py
# Fetched 702,224 pools from API
pools = await fetch_raydium_pairs()  # ← Complete pool data

# Parsed pools → Extracted ONLY volume for tokens ❌
for pair in pools:
    token = TokenMetadata(
        symbol=pair["name"].split("/")[0],
        volume_24h_usd=pair["volume24h"],
    )
    # ❌ Discarded: quote_token, price, reserves, liquidity, pool_address

# Result: 272 tokens cached, 0 prices seeded (no pool data!)
```

**Correct Implementation**:
```python
# pool_data_fetcher_service.py
# Fetch pools → Cache COMPLETE pools → Derive tokens/prices FROM pools
pools = await fetch_raydium_pairs()

# Step 1: Cache complete pools
for pool in pools:
    await self._cache_raydium_pool(pool)  # ← ALL fields cached

# Step 2: Extract tokens FROM pools
cached_pools = await pool_cache.get_all_pools()
unique_tokens = {token for p in cached_pools for token in [p.base_token, p.quote_token]}

# Step 3: Seed prices FROM pools
for pool in cached_pools:
    price = pool.direct_price  # ← Available because we cached it!
    await price_cache.set_price(f"{pool.base_token}:{pool.quote_token}", price)

# Result: 1,709 pools cached, 924 tokens extracted, 1,000+ prices seeded ✓
```

---

### FILTER_BEFORE_LIMIT_POLICY - Business Rules First

**Purpose**: Ensure business criteria filters are applied BEFORE technical limits.

**Core Principle**: The order matters - filter by business rules FIRST, then apply technical limits.

#### Forbidden Patterns

**❌ Limit Before Filter**:
```python
# BAD: Sort → Limit → Filter (WRONG ORDER)
sorted_pools = sorted(all_pools, key=lambda p: p.liquidity, reverse=True)
limited_pools = sorted_pools[:200]  # ❌ Apply limit BEFORE business filter
filtered_pools = [p for p in limited_pools if p.volume >= 50000]

# Result: Business rule ($50K) only applied to 200 pools, not all 702K pools!
```

**❌ No Justification for Limit**:
```python
# BAD: Technical limit without explaining why
selected_pools = filtered_pools[:1000]  # ❌ Why 1000? No comment explaining constraint
```

#### Correct Patterns

**✅ Filter → Sort → Limit (Correct Order)**:
```python
# GOOD: Filter by business criteria FIRST
# Step 1: Apply business rule to ALL data
filtered_pools = [p for p in all_pools if p.volume >= 50000]

# Step 2: Sort by priority metric
sorted_pools = sorted(filtered_pools, key=lambda p: p.liquidity, reverse=True)

# Step 3: Apply technical limit LAST (with justification)
selected_pools = sorted_pools[:10000]  # Memory constraint: 1GB heap max
```

**✅ Justified Technical Limits**:
```python
# GOOD: Every limit has inline comment explaining constraint
MAX_POOLS = 10000  # API rate limit: 100 req/min, 100 pools/req = 10K max

MAX_CACHE_ENTRIES = 50000  # Memory constraint: 2GB heap, 40KB per entry

MAX_CONCURRENT = 100  # Connection pool limit: DB max_connections=100
```

**✅ No Limit When Not Needed**:
```python
# GOOD: No artificial limit if business filter reduces dataset sufficiently
all_pools = await fetch_all_pools()  # 702,224 pools
filtered_pools = [p for p in all_pools if p.volume >= 50000]  # → 1,709 pools

# No limit needed - business filter already reduced from 702K → 1.7K
await pool_cache.set_many(filtered_pools)  # Cache all filtered pools
```

#### Real-World Example: Wrong Filter Order

**What Went Wrong**:
```python
# pool_data_fetcher_service.py (ORIGINAL VERSION)
pools = await fetch_all_pools()  # 702,224 pools

# ❌ WRONG ORDER: Sort → Limit → Filter
sorted_pools = sorted(pools, key=lambda p: p.liquidity, reverse=True)
top_200 = sorted_pools[:200]  # ❌ Limit FIRST to 200 pools (0.03%)
filtered = [p for p in top_200 if p.volume >= 50000]  # Filter SECOND (too late!)

# Result: $50K filter applied to only 200 pools, not 702,224 pools
```

**Correct Implementation**:
```python
# pool_data_fetcher_service.py (FIXED VERSION)
pools = await fetch_all_pools()  # 702,224 pools

# ✅ CORRECT ORDER: Filter → Sort → Limit
# Step 1: Filter by business criteria FIRST
filtered_pools = [p for p in pools if p.liquidity >= 50000]  # → 1,709 pools

# Step 2: Sort by priority
sorted_pools = sorted(filtered_pools, key=lambda p: p.liquidity, reverse=True)

# Step 3: Limit only if needed (not needed here - 1.7K is manageable)
selected_pools = sorted_pools[:999999]  # Unlimited in practice

# Result: 1,709 pools properly filtered from full dataset (not just 200)
```

---

## Configuration Management Standards

**Purpose**: Enforce 12-factor app configuration principles to prevent mixing secrets with configuration and ensure maintainability.

**Core Principle**: Separate secrets (environment-specific, never in code) from configuration (code with defaults, optionally overridden via environment).

### The 12-Factor App Configuration Philosophy

**Rule**: Store config in the environment, but distinguish between **secrets** and **configuration**:

| Category | Storage Location | Characteristics | Examples |
|----------|------------------|----------------|----------|
| **Secrets** | `.env.local` (never committed) | Environment-specific, sensitive, no defaults | API keys, database passwords, private keys, endpoint URLs |
| **Configuration** | `app_config.py` (in code with defaults) | Application behavior, safe defaults, overridable | Feature flags, timeouts, thresholds, retry limits, batch sizes |

### Forbidden Patterns

**❌ Configuration in .env.local**:
```bash
# BAD: Feature flags and configuration in .env.local
ENABLE_WEBSOCKET_SOLANA=true
ENABLE_WEBSOCKET_BSC=true
WEBSOCKET_RECONNECT_MAX_ATTEMPTS=10
WEBSOCKET_PING_INTERVAL=30
CACHE_TTL_SECONDS=3600
MAX_RETRY_ATTEMPTS=5
RAYDIUM_POOL_VOLUME_THRESHOLD=50000

# Problems:
# 1. Configuration mixed with secrets
# 2. No defaults in code (deployment requires full .env.local)
# 3. Hard to understand application behavior without .env.local
# 4. Can't override individual settings (all-or-nothing)
```

**❌ Secrets in Code**:
```python
# BAD: API keys hardcoded in code
class HyperliquidClient:
    def __init__(self):
        self.api_key = "pk_live_1234567890abcdef"  # ❌ FORBIDDEN - secret in code!
        self.endpoint = "https://api.hyperliquid.xyz"  # ❌ Endpoint should be in .env
```

**❌ Mixed Secrets and Configuration**:
```python
# BAD: Mixing secrets with configuration in dataclass
@dataclass
class Settings:
    # Secrets (OK - from .env.local)
    CHAINSTACK_SOLANA_API_KEY: str

    # Configuration (WRONG - should have defaults)
    ENABLE_WEBSOCKET_SOLANA: bool  # ❌ No default - requires .env.local
    WEBSOCKET_PING_INTERVAL: int   # ❌ No default - requires .env.local
```

### Correct Patterns

**✅ Secrets ONLY in .env.local**:
```bash
# GOOD: .env.local contains ONLY secrets (API keys, endpoints, passwords)
# ============================================
# Chainstack API Endpoints and Keys
# ============================================

# Chainstack Solana Mainnet (for Raydium + Orca)
CHAINSTACK_SOLANA_ENDPOINT=https://solana-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID
CHAINSTACK_SOLANA_API_KEY=YOUR_SOLANA_API_KEY_HERE
CHAINSTACK_SOLANA_WSS_URL=wss://solana-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID

# Chainstack BSC Mainnet (for PancakeSwap)
CHAINSTACK_BSC_ENDPOINT=https://bsc-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID
CHAINSTACK_BSC_API_KEY=YOUR_BSC_API_KEY_HERE
CHAINSTACK_BSC_WSS_URL=wss://bsc-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID

# BitQuery API Key (for BSC token metadata)
BITQUERY_API_KEY=YOUR_BITQUERY_API_KEY_HERE

# Hyperliquid Private Key (PLACEHOLDER - Update with real credentials)
HYPERLIQUID_PRIVATE_KEY=0x0000000000000000000000000000000000000000000000000000000000000000

# ============================================
# Notes
# ============================================
# 1. This file should contain ONLY secrets (API keys) and endpoints
# 2. All configuration (enable/disable flags, timeouts, etc.) is in app_config.py
# 3. NEVER commit this file to version control
```

**✅ Configuration with Defaults in Code**:
```python
# GOOD: app_config.py with secrets loaded from .env.local + configuration with defaults
from dataclasses import dataclass
from dotenv import load_dotenv
import os

@dataclass
class Settings:
    """Application settings loaded from environment variables.

    Configuration Philosophy:
    - API keys and endpoints: Load from .env.local (secrets)
    - Feature flags and tuning: Defined here with defaults (configuration)
    """

    # ========================================
    # SECRETS (from .env.local - required)
    # ========================================
    CHAINSTACK_SOLANA_ENDPOINT: str
    CHAINSTACK_SOLANA_API_KEY: str
    CHAINSTACK_SOLANA_WSS_URL: str | None
    CHAINSTACK_BSC_ENDPOINT: str
    CHAINSTACK_BSC_API_KEY: str
    CHAINSTACK_BSC_WSS_URL: str | None
    BITQUERY_API_KEY: str
    HYPERLIQUID_PRIVATE_KEY: str

    # ========================================
    # CONFIGURATION (with defaults - optional env override)
    # ========================================

    # Venue enable/disable flags (configuration defaults, can be overridden via env)
    ENABLE_VENUE_RAYDIUM: bool = True
    ENABLE_VENUE_ORCA: bool = True
    ENABLE_VENUE_PANCAKESWAP: bool = True
    ENABLE_VENUE_HYPERLIQUID: bool = True

    # WebSocket enable/disable flags (configuration defaults, can be overridden via env)
    ENABLE_WEBSOCKET_SOLANA: bool = True
    ENABLE_WEBSOCKET_BSC: bool = True
    ENABLE_WEBSOCKET_HYPERLIQUID: bool = True

    # WebSocket settings (unified for all clients)
    WEBSOCKET_RECONNECT_DELAY: int = 1
    WEBSOCKET_RECONNECT_MAX_DELAY: int = 60
    WEBSOCKET_RECONNECT_MAX_ATTEMPTS: int = 10
    WEBSOCKET_RECONNECT_BACKOFF_BASE: int = 1
    WEBSOCKET_PING_INTERVAL: int = 30
    WEBSOCKET_PONG_TIMEOUT: int = 5

    # Cache configuration
    CACHE_TTL_SECONDS: int = 3600
    CACHE_SEEDING_CONCURRENCY: int = 10
    CACHE_SEEDING_TIMEOUT_SECONDS: int = 120
    TOKEN_VOLUME_THRESHOLD_USD: int = 50000

    # Raydium configuration
    RAYDIUM_AMM_PROGRAM_ID: str = "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
    RAYDIUM_POOL_VOLUME_THRESHOLD: int = 50000
    EVENT_BUFFER_SIZE: int = 100

    # Hyperliquid configuration
    HYPERLIQUID_NETWORK: str = "mainnet"
    HYPERLIQUID_MAX_RETRIES: int = 3
    HYPERLIQUID_TIMEOUT_SECONDS: int = 10
    HYPERLIQUID_RATE_LIMIT_PER_SECOND: int = 100

    # Rate limiting configuration
    MAX_RETRY_ATTEMPTS: int = 5
    RATE_LIMIT_BACKOFF_BASE: int = 1

    # Performance monitoring
    ENABLE_LATENCY_TRACKING: bool = True
    ENABLE_COST_TRACKING: bool = True
    ENABLE_SUCCESS_RATE_TRACKING: bool = True

    # Application
    APP_HOST: str = "0.0.0.0"
    APP_PORT: int = 8000
    ENVIRONMENT: str = "development"
    LOG_LEVEL: str = "INFO"


def _load_settings() -> Settings:
    """Load settings from environment variables with defaults."""
    # Load .env.local if it exists
    load_dotenv(".env.local")

    return Settings(
        # ========================================
        # SECRETS (from .env.local - required)
        # ========================================
        CHAINSTACK_SOLANA_ENDPOINT=os.environ["CHAINSTACK_SOLANA_ENDPOINT"],
        CHAINSTACK_SOLANA_API_KEY=os.getenv("CHAINSTACK_SOLANA_API_KEY", ""),
        CHAINSTACK_SOLANA_WSS_URL=os.getenv("CHAINSTACK_SOLANA_WSS_URL"),
        CHAINSTACK_BSC_ENDPOINT=os.environ["CHAINSTACK_BSC_ENDPOINT"],
        CHAINSTACK_BSC_API_KEY=os.getenv("CHAINSTACK_BSC_API_KEY", ""),
        CHAINSTACK_BSC_WSS_URL=os.getenv("CHAINSTACK_BSC_WSS_URL"),
        BITQUERY_API_KEY=os.environ["BITQUERY_API_KEY"],
        HYPERLIQUID_PRIVATE_KEY=os.getenv("HYPERLIQUID_PRIVATE_KEY", ""),

        # ========================================
        # CONFIGURATION (optional env override)
        # ========================================
        ENABLE_VENUE_RAYDIUM=os.getenv("ENABLE_VENUE_RAYDIUM", "true").lower() in ("true", "1", "yes"),
        ENABLE_VENUE_ORCA=os.getenv("ENABLE_VENUE_ORCA", "true").lower() in ("true", "1", "yes"),
        ENABLE_WEBSOCKET_SOLANA=os.getenv("ENABLE_WEBSOCKET_SOLANA", "true").lower() in ("true", "1", "yes"),
        WEBSOCKET_PING_INTERVAL=int(os.getenv("WEBSOCKET_PING_INTERVAL", "30")),
        CACHE_TTL_SECONDS=int(os.getenv("CACHE_TTL_SECONDS", "3600")),
        RAYDIUM_POOL_VOLUME_THRESHOLD=int(os.getenv("RAYDIUM_POOL_VOLUME_THRESHOLD", "50000")),
        MAX_RETRY_ATTEMPTS=int(os.getenv("MAX_RETRY_ATTEMPTS", "5")),
        LOG_LEVEL=os.getenv("LOG_LEVEL", "INFO"),
        ENVIRONMENT=os.getenv("ENVIRONMENT", "development"),
        # ... (all other configuration with defaults)
    )
```

### Configuration Categorization Guidelines

**Decision Tree**: For each variable, ask:

1. **Is this value different per environment (dev/staging/prod)?**
   - YES → `.env.local` (secret)
   - NO → Continue to step 2

2. **Does this value contain sensitive data (password, key, token)?**
   - YES → `.env.local` (secret)
   - NO → Continue to step 3

3. **Can this value have a sensible default?**
   - YES → `app_config.py` with default (configuration)
   - NO → `.env.local` (likely environment-specific)

**Examples**:

| Variable | Category | Rationale | Location |
|----------|----------|-----------|----------|
| `CHAINSTACK_SOLANA_API_KEY` | Secret | Sensitive, env-specific | `.env.local` |
| `CHAINSTACK_SOLANA_ENDPOINT` | Secret | Environment-specific URL | `.env.local` |
| `ENABLE_WEBSOCKET_SOLANA` | Configuration | Boolean flag, default `true` | `app_config.py` (default: `True`) |
| `WEBSOCKET_PING_INTERVAL` | Configuration | Timeout setting, default `30` | `app_config.py` (default: `30`) |
| `RAYDIUM_AMM_PROGRAM_ID` | Configuration | Blockchain constant | `app_config.py` (default: `"675kPX9..."`) |
| `CACHE_TTL_SECONDS` | Configuration | Performance tuning, default `3600` | `app_config.py` (default: `3600`) |
| `DATABASE_PASSWORD` | Secret | Sensitive credential | `.env.local` |
| `LOG_LEVEL` | Configuration | Logging verbosity, default `INFO` | `app_config.py` (default: `"INFO"`) |

### Naming Conventions

**Consistent naming prevents configuration duplication**:

**✅ GOOD: Consistent naming across settings**:
```python
# Unified WebSocket settings for all clients
WEBSOCKET_RECONNECT_DELAY: int = 1
WEBSOCKET_RECONNECT_MAX_DELAY: int = 60
WEBSOCKET_PING_INTERVAL: int = 30
WEBSOCKET_PONG_TIMEOUT: int = 5

# Used by:
# - SolanaWebSocketClient (Raydium + Orca)
# - BSCWebSocketClient (PancakeSwap)
# - HyperliquidWebSocketClient
```

**❌ BAD: Inconsistent naming creates duplication**:
```python
# BAD: Two sets of WebSocket settings with similar purpose but different names
WS_RECONNECT_DELAY_SECONDS: int = 1              # Used by BSC client
WS_PING_INTERVAL_SECONDS: int = 30               # Used by BSC client
WEBSOCKET_RECONNECT_DELAY: int = 1               # Used by Solana client
WEBSOCKET_PING_INTERVAL: int = 30                # Used by Solana client

# Result: Duplicate settings, confusion about which to use
```

**Naming Rules**:
1. **Use consistent prefixes** for related settings:
   - `WEBSOCKET_*` for all WebSocket settings
   - `CACHE_*` for cache settings
   - `HYPERLIQUID_*` for Hyperliquid-specific settings
   - `RAYDIUM_*` for Raydium-specific settings

2. **Avoid abbreviations** unless industry-standard:
   - ✅ `WEBSOCKET_PING_INTERVAL` (clear)
   - ❌ `WS_PING_INT` (ambiguous)

3. **Use descriptive suffixes** for units:
   - `_SECONDS` for time durations
   - `_USD` for dollar amounts
   - `_PERCENT` for percentages
   - `_MS` for milliseconds

### .env.local.example Template

**Always provide a template** showing ONLY secrets:

```bash
# .env.local.example
# Copy this file to .env.local and fill in your actual credentials

# ============================================
# Chainstack API Endpoints and Keys
# ============================================

# Chainstack Solana Mainnet (for Raydium + Orca)
CHAINSTACK_SOLANA_ENDPOINT=https://solana-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID
CHAINSTACK_SOLANA_API_KEY=YOUR_SOLANA_API_KEY_HERE
CHAINSTACK_SOLANA_WSS_URL=wss://solana-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID

# Chainstack BSC Mainnet (for PancakeSwap)
CHAINSTACK_BSC_ENDPOINT=https://bsc-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID
CHAINSTACK_BSC_API_KEY=YOUR_BSC_API_KEY_HERE
CHAINSTACK_BSC_WSS_URL=wss://bsc-mainnet.core.chainstack.com/YOUR_ENDPOINT_ID

# BitQuery API Key (for BSC token metadata)
BITQUERY_API_KEY=YOUR_BITQUERY_API_KEY_HERE

# Hyperliquid Private Key (PLACEHOLDER - Update with real credentials)
HYPERLIQUID_PRIVATE_KEY=0x0000000000000000000000000000000000000000000000000000000000000000

# ============================================
# Notes
# ============================================
# 1. Copy this file to .env.local and fill in your actual credentials
# 2. NEVER commit .env.local to version control
# 3. This file should contain ONLY secrets (API keys) and endpoints
# 4. All configuration (enable/disable flags, timeouts, etc.) is in app_config.py
```

### Real-World Failure Example

**What Went Wrong** (WebSocket configuration issue):

```python
# ORIGINAL IMPLEMENTATION (WRONG)

# .env.local (77 lines with mixed secrets + configuration) ❌
CHAINSTACK_SOLANA_API_KEY=...
CHAINSTACK_SOLANA_WSS_ENDPOINT=...  # Wrong naming (should be _WSS_URL)
ENABLE_WEBSOCKET_SOLANA=true        # ❌ Configuration in .env.local
WEBSOCKET_PING_INTERVAL=30          # ❌ Configuration in .env.local
RAYDIUM_POOL_VOLUME_THRESHOLD=50000 # ❌ Configuration in .env.local
EVENT_BUFFER_SIZE=100               # ❌ Configuration in .env.local
# ... 40+ more configuration variables

# app_config.py (without defaults) ❌
@dataclass
class Settings:
    # All loaded from .env.local, no defaults
    ENABLE_WEBSOCKET_SOLANA: bool  # ❌ No default - requires .env.local
    WEBSOCKET_PING_INTERVAL: int   # ❌ No default - requires .env.local
    WS_PING_INTERVAL_SECONDS: int  # ❌ Duplicate with different name!

# Problems:
# 1. Configuration mixed with secrets (40+ config vars in .env.local)
# 2. No defaults in code (deployment impossible without full .env.local)
# 3. Inconsistent naming (WEBSOCKET_PING_INTERVAL vs WS_PING_INTERVAL_SECONDS)
# 4. Duplicate settings with similar purpose
```

**Correct Implementation**:

```python
# FIXED IMPLEMENTATION (CORRECT)

# .env.local (37 lines with ONLY secrets) ✅
CHAINSTACK_SOLANA_API_KEY=...
CHAINSTACK_SOLANA_WSS_URL=...      # ✅ Consistent naming (_WSS_URL)
CHAINSTACK_BSC_API_KEY=...
CHAINSTACK_BSC_WSS_URL=...
BITQUERY_API_KEY=...
HYPERLIQUID_PRIVATE_KEY=...
# (Only 8 variables - all secrets)

# app_config.py (with defaults for all configuration) ✅
@dataclass
class Settings:
    # Secrets (from .env.local)
    CHAINSTACK_SOLANA_API_KEY: str
    CHAINSTACK_SOLANA_WSS_URL: str | None

    # Configuration (with defaults)
    ENABLE_WEBSOCKET_SOLANA: bool = True  # ✅ Default in code
    WEBSOCKET_PING_INTERVAL: int = 30     # ✅ Default in code, unified naming
    RAYDIUM_POOL_VOLUME_THRESHOLD: int = 50000  # ✅ Default in code
    EVENT_BUFFER_SIZE: int = 100          # ✅ Default in code

# Benefits:
# 1. Clear separation: .env.local = secrets, app_config.py = configuration
# 2. Defaults in code: deployable without full .env.local
# 3. Consistent naming: WEBSOCKET_* for all WebSocket settings
# 4. No duplicates: consolidated to single unified settings
```

### Validation Checklist

Before committing configuration code, verify:

- [ ] `.env.local` contains ONLY secrets (API keys, endpoints, passwords)
- [ ] `.env.local.example` exists and shows template for secrets
- [ ] All configuration has sensible defaults in `app_config.py`
- [ ] Configuration can be overridden via environment variables (for testing)
- [ ] No hardcoded secrets in code (all loaded from `.env.local`)
- [ ] Naming is consistent across related settings (no duplicates)
- [ ] Comments explain what belongs in `.env.local` vs `app_config.py`
- [ ] `.env.local` is in `.gitignore` (never committed)

### Benefits of Proper Configuration Management

1. **Deployment Flexibility**: Code works with defaults, no full `.env.local` required
2. **Maintainability**: Configuration documented in code, not scattered across env files
3. **Security**: Secrets isolated in `.env.local`, never committed to version control
4. **Testability**: Easy to override configuration for tests without modifying `.env.local`
5. **Clarity**: Clear separation between "what changes per environment" (secrets) and "how the app behaves" (configuration)
6. **Reduced Duplication**: Unified naming prevents duplicate settings with similar purposes

---

## Testing Standards

### Minimum 95% Coverage

```bash
poetry run pytest tests/ --cov=src --cov-report=term-missing --cov-fail-under=95
```

### Test Naming Convention

```python
def test_should_{behavior}_when_{condition}_given_{context}():
    """
    Pattern: test_should_X_when_Y_given_Z
    - behavior: What should happen
    - condition: Under what circumstances
    - context: Given what setup
    """
    pass

# Examples
def test_should_reject_email_when_invalid_format_given_missing_at_symbol():
    with pytest.raises(InvalidEmailError):
        Email("invalid-email.com")

def test_should_emit_event_when_user_registered_given_valid_data():
    user = User(email=Email("test@example.com"), name="Test")
    events = user.domain_events()
    assert any(isinstance(e, UserRegistered) for e in events)
```

---

## Pre-commit Hooks

**Install pre-commit hooks** (.pre-commit-config.yaml):
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black

  - repo: https://github.com/charliermarsh/ruff-pre-commit
    rev: v0.0.270
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.3.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]

  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.5
    hooks:
      - id: bandit
        args: [-r, src/]
```

---

## Summary Checklist

Before committing code, verify:

**Code Quality**:
- [ ] All functions have type hints
- [ ] All public APIs have Google-style docstrings
- [ ] SOLID principles applied
- [ ] Custom exceptions for domain errors
- [ ] Structured logging (no print statements)
- [ ] Value objects are immutable (`frozen=True`)
- [ ] Dependencies injected via constructor
- [ ] Functions are small and focused (<20 lines)

**Configuration Management**:
- [ ] `.env.local` contains ONLY secrets (API keys, endpoints, passwords)
- [ ] `.env.local.example` exists showing template for secrets
- [ ] All configuration has defaults in `app_config.py`
- [ ] Naming is consistent (no duplicates like `WS_*` and `WEBSOCKET_*`)
- [ ] No hardcoded secrets in code
- [ ] Comments explain secrets vs configuration separation

**Code Formatting & Quality**:
- [ ] Black formatting applied
- [ ] Ruff linting passed
- [ ] MyPy strict mode passed
- [ ] Test coverage ≥95%
- [ ] Pre-commit hooks installed and passing
