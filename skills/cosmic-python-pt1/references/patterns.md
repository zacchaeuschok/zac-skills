# Cosmic Python Part 1 — Complete Pattern Reference

Distilled from *Architecture Patterns with Python* by Harry Percival & Bob Gregory. Chapters 1–7 (Part 1: Building an Architecture to Support Domain Modeling).

## Table of Contents
1. [The Core Problem](#1-the-core-problem)
2. [Foundational Principles](#2-foundational-principles)
3. [Domain Model](#3-domain-model)
4. [Repository Pattern](#4-repository-pattern)
5. [Choosing Abstractions](#5-choosing-abstractions)
6. [Service Layer](#6-service-layer)
7. [Testing: High Gear and Low Gear](#7-testing-high-gear-and-low-gear)
8. [Unit of Work](#8-unit-of-work)
9. [Aggregates and Consistency Boundaries](#9-aggregates-and-consistency-boundaries)

---

## 1. The Core Problem

The **Big Ball of Mud** is software's default end-state. Without deliberate architectural effort, domain knowledge scatters across API handlers, business logic couples to I/O, and dependencies tangle. Three forces create it:

- Infrastructure (databases, HTTP) is concrete and immediately useful — it's tempting to call it directly
- Domain objects start as data containers and never gain behavior (the "anemic model")
- Tests are written after the fact, so design is never forced to be testable

The cure is deliberate placement of logic and deliberate direction of dependencies.

---

## 2. Foundational Principles

### Encapsulation
Identify tasks and assign them to well-defined objects. An abstraction hides *how* something works so callers only need to know *what* it does.

### Layering
Divide code into layers with calling rules:
- **Entrypoints** (Flask, FastAPI, CLI) — parse input, call service layer, format output
- **Service Layer** — orchestrate use cases
- **Domain** — business rules and logic
- **Adapters** — database, external APIs, file I/O

Layers call inward only.

### Dependency Inversion Principle (DIP)
- High-level modules (business logic) must not depend on low-level modules (infrastructure). Both depend on abstractions.
- Abstractions should not depend on details. Details depend on abstractions.

**In practice:** your domain model has zero imports from SQLAlchemy, Flask, or any framework. The ORM imports and maps to the domain model, not the other way around.

---

## 3. Domain Model

### Value Objects
Defined by their attributes, not identity. Immutable. Two instances with the same data are equal.

```python
@dataclass(frozen=True)
class OrderLine:
    orderid: str
    sku: str
    qty: int
```

`frozen=True` gives immutability + automatic `__eq__` based on all fields.

**When to use:** Money, coordinates, names, order line items — anything where identity = data.

### Entities
Have persistent identity beyond their attributes. Identity survives attribute changes.

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

    @property
    def allocated_quantity(self) -> int:
        return sum(line.qty for line in self._allocations)

    @property
    def available_quantity(self) -> int:
        return self._purchased_quantity - self.allocated_quantity

    def allocate(self, line: OrderLine):
        if self.can_allocate(line):
            self._allocations.add(line)

    def deallocate(self, line: OrderLine):
        if line in self._allocations:
            self._allocations.remove(line)

    def can_allocate(self, line: OrderLine) -> bool:
        return self.sku == line.sku and self.available_quantity >= line.qty
```

**When to use:** Users, orders, batches, accounts — things that change over time but remain "the same thing."

### Domain Services
Operations that span multiple entities but don't naturally belong to any single one. Use plain functions.

```python
def allocate(line: OrderLine, batches: List[Batch]) -> str:
    try:
        batch = next(b for b in sorted(batches) if b.can_allocate(line))
    except StopIteration:
        raise OutOfStock(f"Out of stock for sku {line.sku}")
    batch.allocate(line)
    return batch.reference
```

### Magic Methods for Domain Semantics
Use `__gt__` to express business ordering rules:

```python
class Batch:
    def __gt__(self, other):
        if self.eta is None:  # warehouse stock (no ETA) sorts first
            return False
        if other.eta is None:
            return True
        return self.eta > other.eta
```

Now `sorted(batches)` respects allocation preference: warehouse stock before shipments, earlier ETAs before later.

### Domain Exceptions
Business failure modes in the language of the domain:

```python
class OutOfStock(Exception):
    pass

class InvalidSku(Exception):
    pass
```

### Rules
- Zero framework imports in domain code
- Test names use domain language: `test_cannot_allocate_if_skus_do_not_match`
- Non-technical stakeholders should recognize their rules in your tests
- Use `@dataclass(frozen=True)` for value objects, explicit `__eq__`/`__hash__` for entities

---

## 4. Repository Pattern

An abstraction over persistent storage that gives the illusion of an in-memory collection.

### Abstract Interface

```python
class AbstractRepository(abc.ABC):
    @abc.abstractmethod
    def add(self, entity):
        raise NotImplementedError

    @abc.abstractmethod
    def get(self, reference) -> Entity:
        raise NotImplementedError
```

Keep it narrow: `add` and `get` are sufficient. Resist adding query methods that belong in read models.

### Classical ORM Mapping

The ORM depends on the model. The model knows nothing about persistence.

```python
# orm.py
from sqlalchemy import Table, Column, Integer, String, Date, ForeignKey, MetaData
from sqlalchemy.orm import mapper, relationship
import model

metadata = MetaData()

order_lines = Table("order_lines", metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("sku", String(255)),
    Column("qty", Integer),
    Column("orderid", String(255)),
)

batches = Table("batches", metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("reference", String(255)),
    Column("sku", String(255)),
    Column("_purchased_quantity", Integer),
    Column("eta", Date, nullable=True),
)

def start_mappers():
    lines_mapper = mapper(model.OrderLine, order_lines)
    mapper(model.Batch, batches, properties={
        "_allocations": relationship(lines_mapper, secondary=allocations)
    })
```

### Concrete Repository

```python
class SqlAlchemyRepository(AbstractRepository):
    def __init__(self, session):
        self.session = session

    def add(self, entity):
        self.session.add(entity)

    def get(self, reference):
        return self.session.query(model.Batch).filter_by(reference=reference).one()

    def list(self):
        return self.session.query(model.Batch).all()
```

### Fake for Testing

```python
class FakeRepository(AbstractRepository):
    def __init__(self, items):
        self._items = set(items)

    def add(self, item):
        self._items.add(item)

    def get(self, reference):
        return next(i for i in self._items if i.reference == reference)

    def list(self):
        return list(self._items)
```

### Ports and Adapters
- **Port:** `AbstractRepository` (the interface)
- **Adapters:** `SqlAlchemyRepository` (production), `FakeRepository` (testing)

---

## 5. Choosing Abstractions

### Five Heuristics
1. Can I represent the state of the external system as a familiar Python data structure?
2. Can I separate *what* I want to happen from *how* it happens?
3. Where can I draw a seam between systems?
4. What implicit concepts can I make explicit?
5. What are the dependencies vs. the core business logic?

### Functional Core, Imperative Shell
Separate stateless logic from I/O. The core function accepts simple data and returns commands:

```python
def determine_actions(source_hashes, dest_hashes, source_folder, dest_folder):
    # Pure function: dict in, actions out
    for sha, filename in source_hashes.items():
        if sha not in dest_hashes:
            yield ("COPY", source_folder / filename, dest_folder / filename)
        elif dest_hashes[sha] != filename:
            yield ("MOVE", dest_folder / dest_hashes[sha], dest_folder / filename)
    for sha, filename in dest_hashes.items():
        if sha not in source_hashes:
            yield ("DELETE", dest_folder / filename)
```

### Dependency Injection Over Mocking
Prefer writing fakes (`FakeRepository`, `FakeUnitOfWork`) over `mock.patch`.

Reasons:
- Abstractions improve design (enable `--dry-run`, alternative backends). Mocks only enable tests.
- Mock tests verify interaction details (brittle). Fake tests verify state outcomes (resilient).
- Explicit dependencies clarify design. Excessive mocking obscures it.

---

## 6. Service Layer

The orchestration layer. Sits between entrypoints and domain logic.

### The Rhythm
Every service function follows: fetch → validate → call domain → persist.

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

### Accept Primitives
Service functions take `str`, `int`, `date` — not domain objects. This decouples the API layer from the domain model.

**Before:** `def allocate(line: OrderLine, repo, session)`
**After:** `def allocate(orderid: str, sku: str, qty: int, uow)`

### Domain Logic vs Orchestration
- **Domain logic** (belongs in model): "allocate to the batch with the earliest ETA that has stock"
- **Orchestration** (belongs in service): "fetch product, call allocate, commit"

If you're writing business `if/else` in a service function, move it to the domain model.

### Thin Entrypoints
Flask/FastAPI handlers do three things: parse input, call a service, format output.

```python
@app.route("/allocate", methods=["POST"])
def allocate_endpoint():
    try:
        batchref = services.allocate(
            request.json["orderid"],
            request.json["sku"],
            request.json["qty"],
            unit_of_work.SqlAlchemyUnitOfWork(),
        )
    except (model.OutOfStock, services.InvalidSku) as e:
        return {"message": str(e)}, 400
    return {"batchref": batchref}, 201
```

---

## 7. Testing: High Gear and Low Gear

### Low Gear — Domain Tests
Best when starting a new project or exploring complex logic. Tests hit domain objects directly.

```python
def test_prefers_warehouse_batches_to_shipments():
    warehouse = Batch("wh", "RETRO-CLOCK", 100, eta=None)
    shipment = Batch("ship", "RETRO-CLOCK", 100, eta=date.today())
    line = OrderLine("o1", "RETRO-CLOCK", 10)
    allocate(line, [warehouse, shipment])
    assert warehouse.available_quantity == 90
    assert shipment.available_quantity == 100
```

### High Gear — Service-Layer Tests
Best for feature work and the bulk of your suite. Tests go through service functions with fakes.

```python
def test_allocate_returns_batch_ref():
    uow = FakeUnitOfWork()
    services.add_batch("b1", "LAMP", 100, None, uow)
    result = services.allocate("o1", "LAMP", 10, uow)
    assert result == "b1"
```

### When to Use Which

| Gear | When | Trade-off |
|------|------|-----------|
| Low (domain) | New project, complex logic, design exploration | High feedback, high coupling |
| High (service) | Feature work, bug fixes, most of your suite | Medium feedback, low coupling |
| E2E (API) | One or two per feature, broad validation | Low feedback, lowest coupling |

### Target Pyramid
- Many service-layer unit tests (fast, decoupled)
- Few integration tests (ORM mapping verification)
- Minimal E2E tests (one or two per feature)

### Signal: Missing Service
If you need domain objects in service-layer tests, you're missing a service function. Add `add_batch()` or similar so tests use only the service API.

---

## 8. Unit of Work

An abstraction over atomic operations. Groups repository access and transaction control.

### Abstract Base

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

### Concrete Implementation

```python
DEFAULT_SESSION_FACTORY = sessionmaker(bind=create_engine(get_postgres_uri()))

class SqlAlchemyUnitOfWork(AbstractUnitOfWork):
    def __init__(self, session_factory=DEFAULT_SESSION_FACTORY):
        self.session_factory = session_factory

    def __enter__(self):
        self.session = self.session_factory()
        self.products = SqlAlchemyRepository(self.session)
        return super().__enter__()

    def __exit__(self, *args):
        super().__exit__(*args)
        self.session.close()

    def commit(self):
        self.session.commit()

    def rollback(self):
        self.session.rollback()
```

### Fake for Testing

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

### Key Properties
- **Safe by default:** no commit = automatic rollback
- **Single entry point:** all repositories accessed via `uow.products`, `uow.orders`, etc.
- **Context manager:** `with uow:` visually groups atomic operations
- **Explicit commits:** only successful paths trigger persistence

---

## 9. Aggregates and Consistency Boundaries

### What is an Aggregate?
A cluster of associated objects treated as a unit for data changes. The **aggregate root** is the sole entry point for modifications.

### Implementation

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

### Rules
1. **One repository per aggregate** — fetch `Product`, never `Batch` directly
2. **All modifications through the root** — `product.allocate(line)`, never `batch.allocate(line)` from outside
3. **Version number for optimistic concurrency** — increment on every state change; the database rejects concurrent modifications
4. **Consistency boundary = transaction boundary** — data within the aggregate is strongly consistent; data across aggregates is eventually consistent

### Choosing Boundaries
Ask: "what data must be consistent within a single transaction?"

- **Too broad** (whole table): poor concurrency, lock contention
- **Too narrow** (individual rows): complex eventual consistency, hard to maintain invariants

The right boundary groups data that shares invariants. `Product` groups all `Batch`es for a SKU because the invariant "an order line can only be allocated to one batch" spans batches within a product.

### Concurrency
The `version_number` enables optimistic locking:
1. Read the aggregate (including version)
2. Modify through the root (version increments)
3. On commit, the database checks the version hasn't changed since the read
4. If another transaction modified the same aggregate, the commit fails → retry

This avoids pessimistic locks while preventing lost updates.
