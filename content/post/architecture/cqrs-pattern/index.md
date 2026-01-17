---
title: "CQRS: Separating Reads from Writes"
description: "Learn how Command Query Responsibility Segregation can simplify complex systems."
slug: cqrs-pattern
date: 2024-04-18
categories:
  - Architecture
tags:
  - cqrs
  - design-patterns
  - scalability
  - event-sourcing
---

CQRS (Command Query Responsibility Segregation) separates read and write operations into different models. Let's explore when and how to use it.

## The Problem with CRUD

Traditional CRUD operations use the same model for reads and writes:

```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────┐
│  Single  │
│  Model   │
└────┬─────┘
     │
     ▼
┌──────────┐
│ Database │
└──────────┘
```

This works until:
- Read and write patterns differ significantly
- You need different scaling for reads vs writes
- Complex queries slow down writes

## The CQRS Solution

Separate your models:

```
┌──────────┐                      ┌──────────┐
│  Client  │                      │  Client  │
└────┬─────┘                      └────┬─────┘
     │ Command                         │ Query
     ▼                                 ▼
┌──────────┐                      ┌──────────┐
│  Write   │                      │  Read    │
│  Model   │─────────────────────▶│  Model   │
└────┬─────┘     Sync/Events      └────┬─────┘
     │                                 │
     ▼                                 ▼
┌──────────┐                      ┌──────────┐
│ Write DB │                      │ Read DB  │
└──────────┘                      └──────────┘
```

## Commands and Queries

### Commands (Write Side)
Commands change state and return nothing:

```python
class CreateOrderCommand:
    customer_id: str
    items: List[OrderItem]

class OrderCommandHandler:
    def handle(self, command: CreateOrderCommand):
        order = Order.create(
            command.customer_id,
            command.items
        )
        self.repository.save(order)
        self.event_bus.publish(OrderCreated(order.id))
```

### Queries (Read Side)
Queries return data and change nothing:

```python
class GetOrderQuery:
    order_id: str

class OrderQueryHandler:
    def handle(self, query: GetOrderQuery) -> OrderDTO:
        return self.read_db.find_order(query.order_id)
```

## Synchronization Strategies

### Synchronous
Update read model immediately after write:
- Simple but couples the models
- Read model always consistent

### Asynchronous (Event-Based)
Use events to update read model:
- Loose coupling
- Eventual consistency
- Better scalability

```python
class OrderCreatedHandler:
    def handle(self, event: OrderCreated):
        self.read_db.insert({
            "order_id": event.order_id,
            "customer_name": self.get_customer_name(event.customer_id),
            "total": event.total,
            "status": "created"
        })
```

## When to Use CQRS

| Use CQRS | Avoid CQRS |
|----------|------------|
| High read/write ratio disparity | Simple CRUD apps |
| Complex read queries | Small datasets |
| Need independent scaling | Tight consistency required |
| Event sourcing | Small teams |

## Conclusion

CQRS adds complexity but provides powerful benefits for the right use cases. Start simple and adopt CQRS only when you hit limitations with traditional patterns.
