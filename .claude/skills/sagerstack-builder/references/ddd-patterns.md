# Domain-Driven Design (DDD) Pattern Catalog

Comprehensive reference for implementing DDD tactical patterns in Python. Used by the Software Developer agent during implementation to ensure proper DDD pattern application.

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
    amount: Decimal
    currency: str = "USD"

    def __post_init__(self):
        if self.amount < 0:
            raise ValueError("Money amount cannot be negative")
        if not self.currency or len(self.currency) != 3:
            raise ValueError("Currency must be 3-letter ISO code")

    def add(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)

    def multiply(self, factor: Decimal) -> "Money":
        return Money(self.amount * factor, self.currency)
```

**Email Value Object**:
```python
import re
from dataclasses import dataclass

@dataclass(frozen=True)
class Email:
    value: str

    def __post_init__(self):
        if not self._isValid(self.value):
            raise ValueError(f"Invalid email format: {self.value}")

    @staticmethod
    def _isValid(email: str) -> bool:
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return bool(re.match(pattern, email))

    def domain(self) -> str:
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

@dataclass
class User:
    id: UUID = field(default_factory=uuid4)
    email: Email
    name: str
    createdAt: datetime = field(default_factory=datetime.utcnow)
    isActive: bool = True
    _domainEvents: list[DomainEvent] = field(default_factory=list, init=False, repr=False)

    def changeEmail(self, newEmail: Email) -> None:
        oldEmail = self.email
        self.email = newEmail
        self._domainEvents.append(
            UserEmailChanged(userId=self.id, oldEmail=oldEmail, newEmail=newEmail)
        )

    def deactivate(self) -> None:
        if not self.isActive:
            raise ValueError("User is already inactive")
        self.isActive = False
        self._domainEvents.append(UserDeactivated(userId=self.id))

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
from uuid import UUID

@dataclass
class OrderItem:
    productId: UUID
    quantity: int
    unitPrice: Money

    def total(self) -> Money:
        return self.unitPrice.multiply(Decimal(self.quantity))

@dataclass
class Order:
    id: UUID = field(default_factory=uuid4)
    customerId: UUID
    items: list[OrderItem] = field(default_factory=list)
    status: OrderStatus = OrderStatus.PENDING
    _domainEvents: list[DomainEvent] = field(default_factory=list, init=False, repr=False)

    def addItem(self, productId: UUID, quantity: int, price: Money) -> None:
        if self.status != OrderStatus.PENDING:
            raise InvalidOrderStateError("Cannot modify submitted order")
        if len(self.items) >= 10:
            raise OrderLimitExceededError("Cannot add more than 10 items")
        item = OrderItem(productId=productId, quantity=quantity, unitPrice=price)
        self.items.append(item)
        self._domainEvents.append(OrderItemAdded(orderId=self.id, item=item))

    def removeItem(self, productId: UUID) -> None:
        if self.status != OrderStatus.PENDING:
            raise InvalidOrderStateError("Cannot modify submitted order")
        self.items = [item for item in self.items if item.productId != productId]
        self._domainEvents.append(OrderItemRemoved(orderId=self.id, productId=productId))

    def submit(self) -> None:
        if not self.items:
            raise EmptyOrderError("Cannot submit empty order")
        if self.status != OrderStatus.PENDING:
            raise InvalidOrderStateError("Order already submitted")
        self.status = OrderStatus.SUBMITTED
        self._domainEvents.append(OrderSubmitted(orderId=self.id, total=self.total()))

    def total(self) -> Money:
        if not self.items:
            return Money(Decimal("0.00"))
        return sum((item.total() for item in self.items), Money(Decimal("0.00")))

    def domainEvents(self) -> list[DomainEvent]:
        events = self._domainEvents.copy()
        self._domainEvents.clear()
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
from typing import Protocol, Optional

class UserRepository(Protocol):
    def getById(self, userId: UUID) -> Optional[User]: ...
    def getByEmail(self, email: Email) -> Optional[User]: ...
    def findAllActive(self) -> list[User]: ...
    def save(self, user: User) -> None: ...
    def delete(self, userId: UUID) -> None: ...
    def exists(self, email: Email) -> bool: ...
```

**Infrastructure Implementation**:
```python
from sqlalchemy.orm import Session

class PostgresUserRepository:
    def __init__(self, session: Session):
        self._session = session

    def getById(self, userId: UUID) -> Optional[User]:
        from infrastructure.orm.models import UserModel
        model = self._session.query(UserModel).filter_by(id=userId).first()
        return self._toDomain(model) if model else None

    def save(self, user: User) -> None:
        from infrastructure.orm.models import UserModel
        model = self._session.query(UserModel).filter_by(id=user.id).first()
        if model:
            model.email = user.email.value
            model.name = user.name
            model.isActive = user.isActive
        else:
            model = UserModel(
                id=user.id, email=user.email.value,
                name=user.name, isActive=user.isActive
            )
            self._session.add(model)
        self._session.commit()

    def _toDomain(self, model) -> User:
        return User(
            id=model.id, email=Email(model.email),
            name=model.name, createdAt=model.createdAt,
            isActive=model.isActive
        )
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
    userId: UUID
    email: str
    registeredAt: datetime = field(default_factory=datetime.utcnow)
    eventType: str = "UserRegistered"

@dataclass(frozen=True)
class OrderSubmitted:
    orderId: UUID
    customerId: UUID
    total: Money
    submittedAt: datetime = field(default_factory=datetime.utcnow)
```

**Event Publisher Interface**:
```python
class EventPublisher(Protocol):
    def publish(self, event: DomainEvent) -> None: ...
    def publishAll(self, events: list[DomainEvent]) -> None: ...
```

---

### Domain Service Pattern

**Characteristics**:
- Stateless operations on domain objects
- Use when operation does not naturally belong to an entity
- Named with verbs (CalculateShippingCost, ValidateOrderConstraints)
- Orchestrates multiple entities/value objects
- Business logic that spans aggregates

**Implementation**:
```python
@dataclass
class PricingService:
    def calculateOrderTotal(
        self,
        items: list[OrderItem],
        customer: Customer,
        promotions: list[Promotion]
    ) -> Money:
        subtotal = sum(
            (item.total() for item in items),
            Money(Decimal("0.00"))
        )
        tierDiscount = self._calculateTierDiscount(subtotal, customer.tier)
        subtotal = subtotal.add(tierDiscount)
        for promo in promotions:
            promoDiscount = self._calculatePromotionDiscount(subtotal, promo)
            subtotal = subtotal.add(promoDiscount)
        return subtotal

    def _calculateTierDiscount(self, amount: Money, tier: CustomerTier) -> Money:
        discountRate = {
            CustomerTier.BRONZE: Decimal("0.00"),
            CustomerTier.SILVER: Decimal("0.05"),
            CustomerTier.GOLD: Decimal("0.10"),
        }[tier]
        discountAmount = amount.multiply(discountRate)
        return Money(-discountAmount.amount, amount.currency)

    def _calculatePromotionDiscount(self, amount: Money, promo: Promotion) -> Money:
        if promo.type == PromotionType.PERCENTAGE:
            discountAmount = amount.multiply(promo.value)
            return Money(-discountAmount.amount, amount.currency)
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
    @abstractmethod
    def isSatisfiedBy(self, candidate: Any) -> bool:
        pass

    def and_(self, other: "Specification") -> "Specification":
        return AndSpecification(self, other)

    def or_(self, other: "Specification") -> "Specification":
        return OrSpecification(self, other)

    def not_(self) -> "Specification":
        return NotSpecification(self)

class AndSpecification(Specification):
    def __init__(self, left: Specification, right: Specification):
        self.left = left
        self.right = right

    def isSatisfiedBy(self, candidate: Any) -> bool:
        return self.left.isSatisfiedBy(candidate) and self.right.isSatisfiedBy(candidate)

class OrSpecification(Specification):
    def __init__(self, left: Specification, right: Specification):
        self.left = left
        self.right = right

    def isSatisfiedBy(self, candidate: Any) -> bool:
        return self.left.isSatisfiedBy(candidate) or self.right.isSatisfiedBy(candidate)

class NotSpecification(Specification):
    def __init__(self, spec: Specification):
        self.spec = spec

    def isSatisfiedBy(self, candidate: Any) -> bool:
        return not self.spec.isSatisfiedBy(candidate)

# Example specifications
class AdultUserSpecification(Specification):
    def isSatisfiedBy(self, user: User) -> bool:
        age = (datetime.now() - user.birthDate).days // 365
        return age >= 18

class ActiveUserSpecification(Specification):
    def isSatisfiedBy(self, user: User) -> bool:
        return user.isActive

# Composing specifications
eligibleForDiscount = (
    AdultUserSpecification()
    .and_(ActiveUserSpecification())
)

if eligibleForDiscount.isSatisfiedBy(user):
    applyDiscount(user)
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
    def createUser(
        self, email: str, name: str, password: str,
        userRepository: UserRepository
    ) -> User:
        emailVo = Email(email)
        if userRepository.exists(emailVo):
            raise ValueError(f"User with email {email} already exists")
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters")

        user = User(
            email=emailVo, name=name,
            passwordHash=self._hashPassword(password),
            subscriptionTier=SubscriptionTier.FREE,
            createdAt=datetime.utcnow(), isActive=True
        )
        user._domainEvents.append(
            UserRegistered(userId=user.id, email=email)
        )
        return user

    def _hashPassword(self, password: str) -> str:
        import bcrypt
        return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()
```

---

## Best Practices

### Aggregate Design
1. **Keep aggregates small**: Ideally single entity + value objects
2. **Reference by ID**: Do not hold direct references to other aggregates
3. **Eventual consistency**: Use domain events for cross-aggregate updates
4. **Single transaction**: All changes within aggregate happen in one transaction

### Domain Events
1. **Past tense naming**: UserRegistered (not RegisterUser)
2. **Immutable**: Use `@dataclass(frozen=True)`
3. **Minimal data**: Only what is necessary for consumers
4. **Clear events**: Collect via `_domainEvents` field, publish after persistence

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

### Anemic Domain Model
**Problem**: Entities with only getters/setters, logic in services.
```python
# BAD: Anemic entity
class Order:
    def __init__(self):
        self.items = []
        self.status = "pending"

# Business logic in service instead of entity
class OrderService:
    def addItem(self, order, item):
        if len(order.items) >= 10:
            raise Error()
        order.items.append(item)
```

**Solution**: Rich domain model with behavior.
```python
# GOOD: Rich entity
class Order:
    def addItem(self, item):
        if len(self.items) >= 10:
            raise OrderLimitExceededError()
        self.items.append(item)
```

### Large Aggregates
**Problem**: Trying to maintain consistency across too many entities.
```python
# BAD: Huge aggregate
class Order:
    customer: Customer  # Full object
    payment: Payment    # Full object
    items: list[Product]  # Full objects
```

**Solution**: Reference by ID, use eventual consistency.
```python
# GOOD: Small aggregate
class Order:
    customerId: UUID      # Reference only
    items: list[OrderItem]  # Value objects within aggregate
```

### Repositories Returning ORM Models
**Problem**: Infrastructure leaking into domain.
```python
# BAD: Returns SQLAlchemy model
def getUser(self, userId):
    return session.query(UserModel).filter_by(id=userId).first()
```

**Solution**: Always return domain entities.
```python
# GOOD: Returns domain entity
def getUser(self, userId):
    model = session.query(UserModel).filter_by(id=userId).first()
    return self._toDomain(model)
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
