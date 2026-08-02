---
name: cosmic-python-pt1
description: "Write Python backend code following the architecture patterns from Part 1 of Architecture Patterns with Python (Cosmic Python) — Domain Model, Repository, Service Layer, Unit of Work, and Aggregates. Use this skill whenever building a new Python service, adding a feature to an existing backend, scaffolding domain-driven layers, writing allocation/ordering/inventory logic, or when the user says 'cosmic python', 'domain driven', 'DDD', 'clean architecture', 'hexagonal', 'ports and adapters', 'scaffold a service', 'add a domain model', 'create a repository', 'set up unit of work', 'build the service layer', or asks how to structure a Python backend so business logic stays separate from infrastructure. Also trigger when the user is writing any Python backend that touches a database and wants the domain logic testable without infrastructure. This is the GENERATIVE counterpart to the cosmic-python review skill — use this one when writing code, use the review skill when evaluating existing code."
---

# Cosmic Python Part 1 — Building Guide

Guide for writing Python backend services where business logic stays sovereign and infrastructure stays peripheral. Based on *Architecture Patterns with Python* by Harry Percival & Bob Gregory.

The existing `cosmic-python` skill reviews code against these patterns. This skill helps you **write** code that follows them from the start.

## When to use this

- Scaffolding a new Python service from scratch
- Adding a feature that needs domain logic, persistence, and an API endpoint
- Refactoring an existing service toward clean architecture
- Any time you're about to put business logic in a Flask/FastAPI handler or SQLAlchemy model and want a better path

## The architecture at a glance

```
Entrypoints (Flask/FastAPI)     ← parses HTTP, calls service layer
    ↓
Service Layer                   ← orchestrates use cases, manages UoW
    ↓
Domain Model                    ← business rules, pure Python, no imports from infrastructure
    ↑
Repository + Unit of Work       ← abstractions that infrastructure implements
    ↑
Adapters (SQLAlchemy, etc.)     ← concrete implementations, depend on domain
```

Dependencies point **inward**. Domain never imports infrastructure. The ORM depends on the model, not the other way around.

## Layer-by-layer guide

Read `references/patterns.md` for the full theory and all code examples. Below is the decision-making guide for each layer.

### 1. Domain Model — start here

The domain model is the most valuable part of your system. Get this right first.

**Value Objects** — things identified by their data, not an ID:
```python
@dataclass(frozen=True)
class OrderLine:
    orderid: str
    sku: str
    qty: int
```
Use `@dataclass(frozen=True)` for immutability and automatic equality. Two `OrderLine`s with the same fields are the same thing.

**Entities** — things with persistent identity:
```python
class Batch:
    def __init__(self, ref: str, sku: str, qty: int, eta: Optional[date]):
        self.reference = ref
        self.sku = sku
        self.eta = eta
        self._purchased_quantity = qty
        self._allocations: Set[OrderLine] = set()

    def __eq__(self, other):
        if not isinstance(other, Batch):
            return False
        return other.reference == self.reference

    def __hash__(self):
        return hash(self.reference)
```
Implement `__eq__` and `__hash__` on the identity field. Attributes can change; identity persists.

**Domain services** — operations that span multiple entities:
```python
def allocate(line: OrderLine, batches: List[Batch]) -> str:
    try:
        batch = next(b for b in sorted(batches) if b.can_allocate(line))
    except StopIteration:
        raise OutOfStock(f"Out of stock for sku {line.sku}")
    batch.allocate(line)
    return batch.reference
```
Use plain functions. Not everything needs to be a method on an entity.

**Domain exceptions** — business failure modes in domain language:
```python
class OutOfStock(Exception):
    pass
```

**Key rules:**
- Zero imports from SQLAlchemy, Flask, or any framework in domain code
- Use Python magic methods (`__gt__`, `__eq__`, `__hash__`) to make domain objects work idiomatically with `sorted()`, sets, dicts
- Name tests in domain language: `test_cannot_allocate_if_skus_do_not_match`
- A non-technical stakeholder should recognize business rules when reading your tests

### 2. Repository — abstract persistence

The repository gives you the illusion of an in-memory collection. Define a narrow interface:

```python
class AbstractRepository(abc.ABC):
    @abc.abstractmethod
    def add(self, entity):
        raise NotImplementedError

    @abc.abstractmethod
    def get(self, reference) -> Entity:
        raise NotImplementedError
```

**The inversion:** the ORM imports and maps to your domain objects. Your domain objects never know they're being persisted.

```python
# orm.py — the ORM depends on the model, not vice versa
from sqlalchemy.orm import mapper
import model

metadata = MetaData()
order_lines = Table("order_lines", metadata,
    Column("id", Integer, primary_key=True),
    Column("sku", String(255)),
    Column("qty", Integer),
)

def start_mappers():
    mapper(model.OrderLine, order_lines)
```

**Fake for testing:**
```python
class FakeRepository(AbstractRepository):
    def __init__(self, items):
        self._items = set(items)
    def add(self, item):
        self._items.add(item)
    def get(self, reference):
        return next(i for i in self._items if i.reference == reference)
```

**One repository per aggregate** — never create a `BatchRepository`. Fetch the aggregate root (`Product`) and let it manage its children.

### 3. Service Layer — orchestrate use cases

Services follow a consistent rhythm: fetch from repo → validate → call domain logic → persist.

```python
def allocate(orderid: str, sku: str, qty: int, uow: AbstractUnitOfWork) -> str:
    line = OrderLine(orderid, sku, qty)
    with uow:
        product = uow.products.get(sku=sku)
        if product is None:
            raise InvalidSku(f"Invalid sku {sku}")
        batchref = product.allocate(line)
        uow.commit()
    return batchref
```

**Accept primitives, not domain objects** — service functions take `str`, `int`, `date`, not `OrderLine`. This decouples the API layer from the domain model and makes testing easier.

**Domain logic vs orchestration logic:**
- Domain: "allocate to the batch with the earliest ETA that has stock" → belongs in the domain model
- Orchestration: "fetch product, call allocate, commit" → belongs in the service layer

If you're writing `if/else` about business rules in a service function, that logic belongs in the domain model.

### 4. Unit of Work — manage transactions

The UoW groups repository access and transaction control into one context manager:

```python
class AbstractUnitOfWork(abc.ABC):
    products: AbstractRepository

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.rollback()

    @abc.abstractmethod
    def commit(self):
        raise NotImplementedError

    @abc.abstractmethod
    def rollback(self):
        raise NotImplementedError
```

**Safe by default:** exiting without calling `commit()` automatically rolls back. Explicit commits only.

**Fake for testing:**
```python
class FakeUnitOfWork(AbstractUnitOfWork):
    def __init__(self):
        self.products = FakeRepository([])
        self.committed = False
    def commit(self):
        self.committed = True
    def rollback(self):
        pass
```

### 5. Aggregates — consistency boundaries

An aggregate is a cluster of objects treated as a single unit for data changes. The aggregate root is the only entry point.

```python
class Product:  # aggregate root
    def __init__(self, sku: str, batches: List[Batch], version_number: int = 0):
        self.sku = sku
        self.batches = batches
        self.version_number = version_number

    def allocate(self, line: OrderLine) -> str:
        try:
            batch = next(b for b in sorted(self.batches) if b.can_allocate(line))
        except StopIteration:
            raise OutOfStock(f"Out of stock for sku {line.sku}")
        batch.allocate(line)
        self.version_number += 1
        return batch.reference
```

**Rules:**
- One repository per aggregate (not per entity)
- All modifications go through the aggregate root
- The `version_number` enables optimistic concurrency — the database rejects concurrent writes to the same aggregate
- Choose aggregate boundaries by asking: "what data must be consistent within a single transaction?"

### 6. Testing strategy

**Low gear** (domain tests) — use when exploring a new domain or solving complex logic:
```python
def test_prefers_warehouse_batches_to_shipments():
    warehouse = Batch("wh", "RETRO-CLOCK", 100, eta=None)
    shipment = Batch("ship", "RETRO-CLOCK", 100, eta=date.today())
    line = OrderLine("o1", "RETRO-CLOCK", 10)
    allocate(line, [warehouse, shipment])
    assert warehouse.available_quantity == 90
    assert shipment.available_quantity == 100
```

**High gear** (service-layer tests) — use for feature work and the bulk of your test suite:
```python
def test_allocate_returns_batch_ref():
    uow = FakeUnitOfWork()
    services.add_batch("b1", "LAMP", 100, None, uow)
    result = services.allocate("o1", "LAMP", 10, uow)
    assert result == "b1"
```

**Target pyramid:** many service-layer unit tests, few integration tests (ORM mapping), minimal E2E tests (one or two per feature).

**Prefer dependency injection over mocking.** Write fakes (`FakeRepository`, `FakeUnitOfWork`) instead of `mock.patch`. Fakes test behavior; mocks test implementation details.

## Scaffolding a new service

When asked to build a new service from scratch, create this structure:

```
src/
├── domain/
│   └── model.py          # entities, value objects, domain services, exceptions
├── adapters/
│   ├── orm.py             # SQLAlchemy classical mappings
│   └── repository.py      # AbstractRepository + SqlAlchemyRepository
├── service_layer/
│   ├── services.py        # use-case functions
│   └── unit_of_work.py    # AbstractUoW + SqlAlchemyUoW
└── entrypoints/
    └── flask_app.py       # thin HTTP layer
tests/
├── unit/
│   ├── test_model.py      # domain logic tests (low gear)
│   └── test_services.py   # service layer tests with fakes (high gear)
├── integration/
│   └── test_repository.py # ORM mapping verification
└── e2e/
    └── test_api.py        # one or two happy-path API tests
```

Start with the domain model. Write tests first. Add layers outward only as needed.
