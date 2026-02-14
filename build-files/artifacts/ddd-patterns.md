# Domain-Driven Design (DDD) Pattern Catalog

## Purpose

Comprehensive reference for implementing Domain-Driven Design tactical patterns in Python. Used by py-developer agent during implementation to ensure proper DDD pattern application.

---

## Strategic Design Patterns

### Bounded Contexts
- Define clear boundaries around domain models
- Each bounded context has its own ubiquitous language
- Minimize dependencies between contexts
- Use context mapping to manage relationships

### Context Mapping
- **Upstream/Downstream**: Identify which context influences which
- **Shared Kernel**: Common model shared between contexts (use sparingly)
- **Customer/Supplier**: Downstream context depends on upstream APIs
- **Conformist**: Downstream conforms to upstream model
- **Anticorruption Layer**: Translate between contexts to protect domain integrity

### Ubiquitous Language
- Use business terminology in code (class names, method names, variables)
- Ensure domain experts and developers speak the same language
- Reflect language in documentation, tests, and conversations

---

## Tactical Patterns

### Entities vs Value Objects

**Use Entities when**:
- Identity matters (User, Order, Account)
- Object has a lifecycle with state changes
- Two objects with same attributes are still different if IDs differ
- Object needs to be tracked over time

**Use Value Objects when**:
- No identity, only attributes matter (Email, Money, Address)
- Immutable (once created, never changes)
- Side-effect free operations (methods return new instances)
- Two objects with same attributes are considered equal

---

## Pattern Implementations

### Value Object Pattern

**Characteristics**:
- Immutable (`@dataclass(frozen=True)`)
- Validation in `__post_init__`
- Methods return new instances (no mutation)
- Equality based on attributes, not identity

**Implementation**:
```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass(frozen=True)
class Money:
    """Represents a monetary amount (Value Object).

    Immutable value object ensuring currency consistency.
    """
    amount: Decimal
    currency: str = "USD"

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("Money amount cannot be negative")
        if not self.currency or len(self.currency) != 3:
            raise ValueError("Currency must be 3-letter ISO code")

    def add(self, other: "Money") -> "Money":
        """Add two Money instances (must have same currency)."""
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)

    def multiply(self, factor: Decimal) -> "Money":
        """Multiply money by a factor."""
        return Money(self.amount * factor, self.currency)

# Usage
price = Money(Decimal("10.00"))
tax = price.multiply(Decimal("0.08"))
total = price.add(tax)  # Money(10.80, "USD")
```

**Example: Email Value Object**:
```python
import re
from dataclasses import dataclass

@dataclass(frozen=True)
class Email:
    """Email address value object with validation."""
    value: str

    def __post_init__(self):
        if not self._is_valid(self.value):
            raise ValueError(f"Invalid email format: {self.value}")

    @staticmethod
    def _is_valid(email: str) -> bool:
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, email))

    def domain(self) -> str:
        """Extract domain from email."""
        return self.value.split('@')[1]
```

---

### Entity Pattern

**Characteristics**:
- Has unique identity (ID field)
- Mutable state (attributes can change)
- Lifecycle management (created, modified, deleted)
- Equality based on ID, not attributes
- Emits domain events for significant state changes

**Implementation**:
```python
from dataclasses import dataclass, field
from uuid import UUID, uuid4
from datetime import datetime
from typing import List

@dataclass
class User:
    """User entity (aggregate root)."""
    id: UUID = field(default_factory=uuid4)
    email: Email
    name: str
    created_at: datetime = field(default_factory=datetime.utcnow)
    is_active: bool = True
    _domain_events: List[DomainEvent] = field(default_factory=list, init=False, repr=False)

    def change_email(self, new_email: Email) -> None:
        old_email = self.email
        self.email = new_email
        self._domain_events.append(UserEmailChanged(user_id=self.id, old_email=old_email, new_email=new_email))

    def deactivate(self) -> None:
        if not self.is_active:
            raise ValueError("User is already inactive")
        self.is_active = False
        self._domain_events.append(UserDeactivated(user_id=self.id))

    def __eq__(self, other):
        return isinstance(other, User) and self.id == other.id

    def __hash__(self):
        return hash(self.id)
```

---

### Aggregate Pattern

**Characteristics**:
- Cluster of entities and value objects with single root
- Enforces invariants (business rules that must always be true)
- External objects reference aggregate by root ID only
- All changes go through aggregate root
- Maintains consistency boundary

**Design Rules**:
1. Keep aggregates small (ideally single entity + value objects)
2. Reference other aggregates by ID only (no object references)
3. Use eventual consistency between aggregates
4. One repository per aggregate root

**Implementation**:
```python
from dataclasses import dataclass, field
from typing import List
from uuid import UUID

@dataclass
class OrderItem:
    """Value object within Order aggregate."""
    product_id: UUID
    quantity: int
    unit_price: Money

    def total(self) -> Money:
        """Calculate line item total."""
        return self.unit_price.multiply(Decimal(self.quantity))

@dataclass
class Order:
    """Order aggregate root.

    Enforces invariants:
    - Maximum 10 items per order
    - Cannot submit empty order
    - Cannot modify submitted order
    """
    id: UUID = field(default_factory=uuid4)
    customer_id: UUID  # Reference to Customer aggregate by ID
    items: List[OrderItem] = field(default_factory=list)
    status: OrderStatus = OrderStatus.PENDING
    _domain_events: List[DomainEvent] = field(default_factory=list, init=False, repr=False)

    def add_item(self, product_id: UUID, quantity: int, price: Money) -> None:
        """Add item to order (enforces 10-item limit)."""
        if self.status != OrderStatus.PENDING:
            raise ValueError("Cannot modify submitted order")

        if len(self.items) >= 10:
            raise OrderLimitExceededError("Cannot add more than 10 items")

        item = OrderItem(product_id=product_id, quantity=quantity, unit_price=price)
        self.items.append(item)
        self._domain_events.append(OrderItemAdded(order_id=self.id, item=item))

    def remove_item(self, product_id: UUID) -> None:
        """Remove item from order."""
        if self.status != OrderStatus.PENDING:
            raise ValueError("Cannot modify submitted order")

        self.items = [item for item in self.items if item.product_id != product_id]
        self._domain_events.append(OrderItemRemoved(order_id=self.id, product_id=product_id))

    def submit(self) -> None:
        """Submit order for processing (enforces non-empty rule)."""
        if not self.items:
            raise EmptyOrderError("Cannot submit empty order")

        if self.status != OrderStatus.PENDING:
            raise ValueError("Order already submitted")

        self.status = OrderStatus.SUBMITTED
        self._domain_events.append(OrderSubmitted(order_id=self.id, total=self.total()))

    def total(self) -> Money:
        """Calculate order total."""
        if not self.items:
            return Money(Decimal("0.00"))

        return sum((item.total() for item in self.items), Money(Decimal("0.00")))

    def domain_events(self) -> List[DomainEvent]:
        """Retrieve and clear domain events."""
        events = self._domain_events.copy()
        self._domain_events.clear()
        return events
```

---

### Repository Interface Pattern

**Characteristics**:
- One repository per aggregate root
- Interface (port) in domain or application layer
- Implementation (adapter) in infrastructure layer
- Returns domain objects, not ORM models
- Hides persistence details from domain

**Implementation**:
```python
from abc import ABC, abstractmethod
from typing import Optional, List, Protocol
from uuid import UUID

class UserRepository(Protocol):
    """Repository interface for User aggregate (port)."""

    def get_by_id(self, user_id: UUID) -> Optional[User]:
        """Retrieve user by ID (None if not found)."""
        ...

    def get_by_email(self, email: Email) -> Optional[User]:
        """Retrieve user by email (None if not found)."""
        ...

    def find_all_active(self) -> List[User]:
        """Retrieve all active users."""
        ...

    def save(self, user: User) -> None:
        """Persist user (insert or update)."""
        ...

    def delete(self, user_id: UUID) -> None:
        """Remove user from persistence."""
        ...

    def exists(self, email: Email) -> bool:
        """Check if user with email exists."""
        ...
```

**Infrastructure Implementation Example**:
```python
from typing import Optional, List
from sqlalchemy.orm import Session

class PostgresUserRepository:
    """PostgreSQL implementation of UserRepository (adapter)."""

    def __init__(self, session: Session):
        self._session = session

    def get_by_id(self, user_id: UUID) -> Optional[User]:
        from infrastructure.orm.models import UserModel
        model = self._session.query(UserModel).filter_by(id=user_id).first()
        return self._to_domain(model) if model else None

    def save(self, user: User) -> None:
        from infrastructure.orm.models import UserModel
        model = self._session.query(UserModel).filter_by(id=user.id).first()
        if model:
            model.email, model.name, model.is_active = user.email.value, user.name, user.is_active
        else:
            model = UserModel(id=user.id, email=user.email.value, name=user.name, is_active=user.is_active)
            self._session.add(model)
        self._session.commit()

    def _to_domain(self, model: UserModel) -> User:
        return User(id=model.id, email=Email(model.email), name=model.name,
                    created_at=model.created_at, is_active=model.is_active)
```

---

### Domain Event Pattern

**Characteristics**:
- Immutable (`@dataclass(frozen=True)`)
- Past-tense naming (UserRegistered, OrderSubmitted)
- Represents significant state change
- Carries minimal necessary data
- Used for cross-aggregate communication

**Implementation**:
```python
@dataclass(frozen=True)
class UserRegistered:
    """Domain event: User registration completed."""
    user_id: UUID
    email: str
    registered_at: datetime = field(default_factory=datetime.utcnow)
    event_type: str = "UserRegistered"

@dataclass(frozen=True)
class OrderSubmitted:
    """Domain event: Order submitted for processing."""
    order_id: UUID
    customer_id: UUID
    total: Money
    submitted_at: datetime = field(default_factory=datetime.utcnow)
```

**Event Publisher Interface**:
```python
from typing import List, Protocol

class EventPublisher(Protocol):
    """Interface for publishing domain events."""

    def publish(self, event: DomainEvent) -> None:
        """Publish single domain event."""
        ...

    def publish_all(self, events: List[DomainEvent]) -> None:
        """Publish multiple domain events."""
        ...
```

---

### Domain Service Pattern

**Characteristics**:
- Stateless operations on domain objects
- Use when operation doesn't naturally belong to an entity
- Named with verbs (CalculateShippingCost, ValidateOrderConstraints)
- Orchestrates multiple entities/value objects
- Business logic that spans aggregates

**Implementation**:
```python
from dataclasses import dataclass

@dataclass
class PricingService:
    """Domain service for pricing calculations.

    Stateless service for complex pricing rules spanning multiple aggregates.
    """

    def calculate_order_total(
        self,
        items: List[OrderItem],
        customer: Customer,
        promotions: List[Promotion]
    ) -> Money:
        """Calculate total order price with discounts.

        Business rule: Apply customer tier discount, then promotion codes.
        """
        subtotal = sum((item.total() for item in items), Money(Decimal("0.00")))

        # Apply customer tier discount
        tier_discount = self._calculate_tier_discount(subtotal, customer.tier)
        subtotal = subtotal.add(tier_discount)

        # Apply promotion codes
        for promo in promotions:
            promo_discount = self._calculate_promotion_discount(subtotal, promo)
            subtotal = subtotal.add(promo_discount)

        return subtotal

    def _calculate_tier_discount(self, amount: Money, tier: CustomerTier) -> Money:
        """Calculate tier-based discount (negative amount)."""
        discount_rate = {
            CustomerTier.BRONZE: Decimal("0.00"),
            CustomerTier.SILVER: Decimal("0.05"),
            CustomerTier.GOLD: Decimal("0.10"),
        }[tier]

        discount_amount = amount.multiply(discount_rate)
        return Money(-discount_amount.amount, amount.currency)

    def _calculate_promotion_discount(self, amount: Money, promo: Promotion) -> Money:
        """Calculate promotion code discount."""
        if promo.type == PromotionType.PERCENTAGE:
            discount_amount = amount.multiply(promo.value)
            return Money(-discount_amount.amount, amount.currency)
        elif promo.type == PromotionType.FIXED:
            return Money(-promo.value, amount.currency)
        else:
            raise ValueError(f"Unknown promotion type: {promo.type}")
```

---

### Specification Pattern

**Characteristics**:
- Encapsulates business rules as objects
- Composable with AND, OR, NOT operations
- Reusable across queries and validation
- Testable in isolation

**Implementation**:
```python
from abc import ABC, abstractmethod
from typing import Any

class Specification(ABC):
    """Base specification for business rules."""

    @abstractmethod
    def is_satisfied_by(self, candidate: Any) -> bool:
        """Check if candidate satisfies specification."""
        pass

    def and_(self, other: "Specification") -> "Specification":
        return AndSpecification(self, other)

    def or_(self, other: "Specification") -> "Specification":
        return OrSpecification(self, other)

    def not_(self) -> "Specification":
        return NotSpecification(self)


class AndSpecification(Specification):
    """Composite specification: Both must be satisfied."""

    def __init__(self, left: Specification, right: Specification):
        self.left = left
        self.right = right

    def is_satisfied_by(self, candidate: Any) -> bool:
        return self.left.is_satisfied_by(candidate) and self.right.is_satisfied_by(candidate)


class OrSpecification(Specification):
    """Composite specification: Either must be satisfied."""

    def __init__(self, left: Specification, right: Specification):
        self.left = left
        self.right = right

    def is_satisfied_by(self, candidate: Any) -> bool:
        return self.left.is_satisfied_by(candidate) or self.right.is_satisfied_by(candidate)


class NotSpecification(Specification):
    """Composite specification: Negation."""

    def __init__(self, spec: Specification):
        self.spec = spec

    def is_satisfied_by(self, candidate: Any) -> bool:
        return not self.spec.is_satisfied_by(candidate)


class AdultUserSpecification(Specification):
    """User must be 18+ years old."""

    def is_satisfied_by(self, user: User) -> bool:
        age = (datetime.now() - user.birth_date).days // 365
        return age >= 18


class ActiveUserSpecification(Specification):
    """User must have active account."""

    def is_satisfied_by(self, user: User) -> bool:
        return user.is_active


class PremiumUserSpecification(Specification):
    """User must have premium subscription."""

    def is_satisfied_by(self, user: User) -> bool:
        return user.subscription_tier in [SubscriptionTier.PREMIUM, SubscriptionTier.ENTERPRISE]


# Usage: Composing specifications
eligible_for_discount = (
    AdultUserSpecification()
    .and_(ActiveUserSpecification())
    .and_(PremiumUserSpecification())
)

if eligible_for_discount.is_satisfied_by(user):
    apply_discount(user)
```

---

### Factory Pattern

**Characteristics**:
- Encapsulates complex object creation
- Enforces invariants during construction
- Centralizes creation logic
- Useful when object construction has business rules

**Implementation**:
```python
@dataclass
class UserFactory:
    """Factory for creating User entities with business rule validation."""

    def create_user(self, email: str, name: str, password: str, user_repository: UserRepository) -> User:
        """Create user with validation (email uniqueness, password strength)."""
        email_vo = Email(email)
        if user_repository.exists(email_vo):
            raise ValueError(f"User with email {email} already exists")
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")

        user = User(
            email=email_vo, name=name, password_hash=self._hash_password(password),
            subscription_tier=SubscriptionTier.FREE, created_at=datetime.utcnow(), is_active=True
        )
        user._domain_events.append(UserRegistered(user_id=user.id, email=email))
        return user

    def _hash_password(self, password: str) -> str:
        import bcrypt
        return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()
```

---

## Best Practices

### Aggregate Design
1. **Keep aggregates small**: Ideally single entity + value objects
2. **Reference by ID**: Don't hold direct references to other aggregates
3. **Eventual consistency**: Use domain events for cross-aggregate updates
4. **Single transaction**: All changes within aggregate happen in one transaction

### Domain Events
1. **Past tense naming**: UserRegistered (not RegisterUser)
2. **Immutable**: Use `@dataclass(frozen=True)`
3. **Minimal data**: Only what's necessary for consumers
4. **Clear events**: Collect via `_domain_events` field, publish after persistence

### Repository Pattern
1. **One per aggregate root**: Not per entity
2. **Return domain objects**: Never leak ORM models to domain layer
3. **Interface in domain**: Implementation in infrastructure
4. **Hide persistence**: Domain layer has zero knowledge of database

### Value Objects
1. **Immutability**: Always use `frozen=True`
2. **Validation in __post_init__**: Fail fast on invalid state
3. **No identity**: Equality based on attributes
4. **Side-effect free**: Methods return new instances

### Entities
1. **Identity**: ID field (UUID recommended)
2. **Mutable**: State can change over time
3. **Domain events**: Emit for significant state changes
4. **Equality by ID**: Override `__eq__` and `__hash__`

---

## Common Pitfalls

### ❌ Anemic Domain Model
**Problem**: Entities with only getters/setters, logic in services
```python
# BAD: Anemic entity
class Order:
    def __init__(self):
        self.items = []
        self.status = "pending"

# Business logic in service instead of entity
class OrderService:
    def add_item(self, order, item):
        if len(order.items) >= 10:
            raise Error()
        order.items.append(item)
```

**Solution**: Rich domain model with behavior
```python
# GOOD: Rich entity
class Order:
    def add_item(self, item):
        if len(self.items) >= 10:
            raise OrderLimitExceededError()
        self.items.append(item)
```

### ❌ Large Aggregates
**Problem**: Trying to maintain consistency across too many entities
```python
# BAD: Huge aggregate
class Order:
    customer: Customer  # Full object
    shipping_address: Address
    billing_address: Address
    payment: Payment  # Full object
    items: List[Product]  # Full objects
```

**Solution**: Reference by ID, use eventual consistency
```python
# GOOD: Small aggregate
class Order:
    customer_id: UUID  # Reference only
    items: List[OrderItem]  # Value objects within aggregate
    # Use domain events to notify Payment bounded context
```

### ❌ Repositories Returning ORM Models
**Problem**: Infrastructure leaking into domain
```python
# BAD: Returns SQLAlchemy model
def get_user(self, user_id):
    return session.query(UserModel).filter_by(id=user_id).first()
```

**Solution**: Always return domain entities
```python
# GOOD: Returns domain entity
def get_user(self, user_id):
    model = session.query(UserModel).filter_by(id=user_id).first()
    return self._to_domain(model)
```

---

## Quick Reference

| Pattern | When to Use | Key Trait |
|---------|-------------|-----------|
| **Value Object** | No identity, immutable concepts | `@dataclass(frozen=True)` |
| **Entity** | Has identity, mutable lifecycle | `id: UUID`, `__eq__` by ID |
| **Aggregate** | Enforce invariants across entities | Single root, transactional boundary |
| **Repository** | Abstract persistence | One per aggregate root |
| **Domain Event** | Significant state change occurred | Immutable, past-tense |
| **Domain Service** | Cross-entity business logic | Stateless, verb-named |
| **Specification** | Reusable business rules | Composable with AND/OR/NOT |
| **Factory** | Complex object creation | Encapsulates construction rules |

---

## References

- **Domain-Driven Design** by Eric Evans (Blue Book)
- **Implementing Domain-Driven Design** by Vaughn Vernon (Red Book)
- **Python implementation patterns**: Scott Millett, Nick Tune
