---
title: "Domain-Driven Design: A Practical Introduction"
description: "Discover how DDD helps you build software that reflects your business domain."
slug: domain-driven-design
date: 2024-03-15
categories:
  - Architecture
tags:
  - ddd
  - domain-driven-design
  - software-design
  - bounded-context
---

Domain-Driven Design (DDD) is an approach to software development that centers on the business domain. Let's explore its key concepts.

## Strategic Design

### Bounded Contexts
A bounded context is a boundary within which a particular domain model applies.

```
┌─────────────────────────────────────────────────────┐
│                    E-Commerce                        │
├─────────────────┬─────────────────┬─────────────────┤
│    Ordering     │    Shipping     │    Billing      │
│    Context      │    Context      │    Context      │
│                 │                 │                 │
│  Order          │  Shipment       │  Invoice        │
│  OrderItem      │  Package        │  Payment        │
│  Customer       │  Address        │  Customer       │
└─────────────────┴─────────────────┴─────────────────┘
```

### Ubiquitous Language
A shared language between developers and domain experts:

- Use the same terms in code as in business discussions
- Avoid technical jargon when naming domain concepts
- Document and evolve the language together

## Tactical Design

### Entities
Objects with a unique identity that persists over time:

```python
class Order:
    def __init__(self, order_id: OrderId):
        self.id = order_id
        self.items = []
        self.status = OrderStatus.PENDING

    def add_item(self, product: Product, quantity: int):
        self.items.append(OrderItem(product, quantity))
```

### Value Objects
Immutable objects defined by their attributes:

```python
@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)
```

### Aggregates
A cluster of entities and value objects with a root entity:

```python
class Order:  # Aggregate Root
    def __init__(self):
        self.items: List[OrderItem] = []  # Entity
        self.shipping_address: Address = None  # Value Object
```

### Domain Events
Represent something significant that happened:

```python
class OrderPlaced:
    def __init__(self, order_id: OrderId, customer_id: CustomerId):
        self.order_id = order_id
        self.customer_id = customer_id
        self.occurred_at = datetime.now()
```

## Repository Pattern

Abstracts data access for aggregates:

```python
class OrderRepository(ABC):
    @abstractmethod
    def find_by_id(self, order_id: OrderId) -> Optional[Order]:
        pass

    @abstractmethod
    def save(self, order: Order) -> None:
        pass
```

## Conclusion

DDD is about understanding your domain deeply. Start with strategic design to identify bounded contexts, then use tactical patterns to implement them.
