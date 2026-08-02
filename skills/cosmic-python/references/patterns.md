# Cosmic Python — Part 1 Pattern Reference

Full theory distilled from *Architecture Patterns with Python* (Percival & Gregory). Use this as the authoritative source when writing findings.

## Table of Contents
1. [The Core Problem: Big Ball of Mud](#1-the-core-problem)
2. [Foundational Principles](#2-foundational-principles)
3. [Domain Model Pattern](#3-domain-model-pattern)
4. [Repository Pattern](#4-repository-pattern)
5. [Abstractions and Coupling](#5-abstractions-and-coupling)
6. [Service Layer Pattern](#6-service-layer-pattern)
7. [TDD: High Gear and Low Gear](#7-tdd-high-gear-and-low-gear)
8. [Unit of Work Pattern](#8-unit-of-work-pattern)
9. [Aggregates and Consistency Boundaries](#9-aggregates-and-consistency-boundaries)
10. [Signs of a Healthy Architecture](#10-signs-of-a-healthy-architecture)
11. [Common Violations Catalog](#11-common-violations-catalog)

---

## 1. The Core Problem

**The Big Ball of Mud** is software's default end-state. Without deliberate effort, domain knowledge scatters across API handlers, business logic couples to I/O operations, and dependencies tangle. This happens because:

- Infrastructure (databases, HTTP frameworks) is concrete and immediately useful — it's tempting to call it directly
- Domain objects start as data containers and never gain behavior
- Tests are written after the fact, so design is never forced to be testable

The result: every change is risky, tests require a running database, and onboarding a new engineer takes weeks just to understand where logic lives.

**The cure is not complexity** — it's deliberate placement of logic and deliberate direction of dependencies. All patterns in this book serve one goal: keep business logic sovereign and infrastructure peripheral.

---

## 2. Foundational Principles

### Encapsulation
Identify tasks and assign them to well-defined objects. An abstraction hides how something works so callers only need to know what it does. The right level of abstraction makes code expressive and testable.

### Layering
Divide code into discrete layers with defined calling rules:
- **Entrypoints** (Flask, CLI, workers) — parse input, call service layer, format output
- **Service Layer** — orchestrate use cases
- **Domain** — business rules and logic
- **Infrastructure/Adapters** — database, external APIs, file I/O

Layers call inward only. Infrastructure never imports domain. Domain never imports infrastructure.

### Dependency Inversion Principle (DIP)
High-level modules (business logic) must not depend on low-level modules (infrastructure). Both should depend on abstractions.

In practice: your domain model should have zero imports from SQLAlchemy, Flask, requests, or any other framework. If it does, business logic is coupled to infrastructure — a change in ORM version or database technology requires touching domain code.

**The inversion:** rather than `OrderLine` inheriting from `SQLAlchemy.Base`, the ORM mapping should depend on `OrderLine`. The domain object doesn't know it's being persisted.

---

## 3. Domain Model Pattern

### What it is
A dedicated layer where business logic lives, expressed in the language of the business. Objects in the domain model are designed around behavior and rules, not around database schemas or API payloads.

### Entities vs. Value Objects

**Value Objects** are identified solely by their data:
- Immutable — changing any attribute makes it a different object
- Equality based on all attributes
- Use frozen dataclasses in Python
- Examples: `OrderLine(orderid, sku, qty)`, `Money(amount, currency)`

```python
@dataclass(frozen=True)
class OrderLine:
    orderid: str
    sku: str
    qty: int
```

**Entities** have persistent identity that survives attribute changes:
- Mutable
- Equality based on unique identifier
- Implement `__eq__` and `__hash__` based on the identity field
- Examples: `Batch`, `Order`, `User`

```python
class Batch:
    def __eq__(self, other):
        if not isinstance(other, Batch):
            return False
        return other.reference == self.reference

    def __hash__(self):
        return hash(self.reference)
```

The critical distinction: a `Batch` with 18 allocated units is still the same `Batch` as when it had 20 — the reference is its identity. An `OrderLine` with qty=3 is a fundamentally different object from one with qty=2.

### Domain Services
Some business operations don't belong on any single entity or value object. Express these as standalone functions in the domain layer.

```python
def allocate(line: OrderLine, batches: List[Batch]) -> str:
    batch = next(b for b in sorted(batches) if b.can_allocate(line))
    batch.allocate(line)
    return batch.reference
```

This is a domain service: it works on domain objects, expresses a business rule, and has no infrastructure concerns.

### Domain Exceptions
Use domain-specific exceptions to communicate business conditions:

```python
class OutOfStock(Exception):
    pass
```

Not `ValueError` or `HTTPException` — those are infrastructure concepts.

### Ubiquitous Language
Domain objects, method names, test names, and exception names should all use the language that business stakeholders use. If your code says `batch.can_allocate(line)` and your stakeholders say "can we fulfill this order from this shipment," you're aligned. If your code says `batch.check_qty_gte(line.qty)` you've lost the translation.

### The Anemic Domain Model anti-pattern
An anemic domain model is a class that looks like a domain object but has no methods — just getters, setters, and data. All the business logic lives somewhere else (usually in "service" classes that reach into the model's fields).

Signs:
- Classes with only `__init__` and properties
- Service functions that manipulate domain object internals directly
- Business rules expressed as if/else chains in API handlers or service functions
- Domain objects that inherit from ORM base classes and add nothing but columns

An anemic model defeats the purpose of a domain model — the logic is still scattered, just scattered into service classes instead of endpoint handlers.

### Domain model independence
The domain model should have no imports from infrastructure. It should be possible to run all domain logic tests with zero database connections, zero HTTP calls, and zero framework imports.

---

## 4. Repository Pattern

### What it is
An abstraction over data storage that makes the persistence layer swappable and the domain layer ignorant of how data is stored. The repository presents a simple interface (add, get) that makes storage look like an in-memory collection.

### The abstraction

```python
class AbstractRepository(abc.ABC):
    @abc.abstractmethod
    def add(self, batch: model.Batch):
        raise NotImplementedError

    @abc.abstractmethod
    def get(self, reference: str) -> model.Batch:
        raise NotImplementedError
```

The domain layer and service layer depend on `AbstractRepository`. Concrete implementations (`SqlAlchemyRepository`, `FakeRepository`) are only wired in at the edges of the system.

### ORM mapping inversion
Traditional ORMs (Django ORM, SQLAlchemy declarative base) couple domain models to infrastructure:

```python
# WRONG — domain model inherits from ORM
class OrderLine(Base):
    __tablename__ = 'order_lines'
    id = Column(Integer, primary_key=True)
    sku = Column(String)
```

The right approach uses SQLAlchemy's classical mapping — the ORM depends on the domain model, not vice versa:

```python
# Correct — pure domain object
class OrderLine:
    def __init__(self, orderid, sku, qty):
        self.orderid = orderid
        self.sku = sku
        self.qty = qty

# ORM mapping defined separately, in adapters/
mapper_registry.map_imperatively(model.OrderLine, order_lines_table)
```

This is "persistence ignorance" — the domain model knows nothing about how it's loaded or saved.

### Fake Repository
The most valuable property of the Repository pattern is how easy it makes testing:

```python
class FakeRepository(AbstractRepository):
    def __init__(self, batches):
        self._batches = set(batches)

    def add(self, batch):
        self._batches.add(batch)

    def get(self, reference):
        return next(b for b in self._batches if b.reference == reference)
```

If this is hard to write, the abstraction is too complicated. The ease of writing a fake is a design signal.

### Ports and Adapters vocabulary
- **Port**: The interface/abstraction (e.g., `AbstractRepository`)
- **Adapter**: A concrete implementation of that port (e.g., `SqlAlchemyRepository`, `FakeRepository`)

The domain and service layer only see ports. Adapters are wired in at the application boundary.

### When to apply
The Repository pattern pays off when:
- The domain is complex (business rules, invariants, multi-object operations)
- You want to test business logic without a database
- You may need to swap persistence implementations (different databases, caching layers)

It may not be worth it for simple CRUD applications where the "domain" is just storing and retrieving records.

---

## 5. Abstractions and Coupling

### The coupling paradox
Local coupling is good — tightly cohesive components that work together. Global coupling (unrelated parts depending on each other) kills systems. Abstractions reduce global coupling by hiding complexity behind simpler interfaces.

The rule: **the number of dependencies should grow proportionally to the system's size, not superlinearly.** When every module knows about every other module, the graph explodes. Abstractions cap this growth.

### Functional Core, Imperative Shell
One of the most powerful patterns for testability:

- **Functional core**: Pure logic, no I/O, no side effects — just transforms inputs to outputs
- **Imperative shell**: Gathers real-world state, calls the functional core, applies outputs to the world

```
Shell: read filesystem → functional core → write filesystem
Core: given {source: {hash: path}, dest: {hash: path}} → return [(action, src, dst), ...]
```

The core is trivially testable with dictionaries. The shell is thin and needs only integration tests.

### Fakes vs. Mocks

**Fakes** are real (minimal) implementations of an interface:
- Assert on end state: "did the right thing end up in the repository?"
- Don't break when internal behavior changes
- Force good design — if a fake is hard to write, the interface is wrong
- Support additional use cases (dry-run mode, FTP backend, etc.)

**Mocks** (from `unittest.mock`) record calls and assert on behavior:
- Verify intermediate steps: "did we call `shutil.copy` with these args?"
- Brittle — break when implementation changes even if behavior is identical
- Don't improve design — `mock.patch(os.walk)` doesn't make the code more extensible
- Can be overused to test implementation details rather than outcomes

Prefer fakes. Use mocks only at system boundaries you don't control (third-party APIs, time, randomness).

### Finding the right abstraction
Ask:
1. Can the messy external system's state be represented as a standard Python data structure (dict, list)?
2. Can a single function return that state?
3. Can the "intended effects" be represented as data (a list of action tuples) rather than direct calls?
4. Where is the natural seam — the cleanest place to insert an abstraction?

---

## 6. Service Layer Pattern

### What it is
A thin orchestration layer that sits between entrypoints (HTTP handlers, CLI) and the domain. Each service function represents exactly one use case. It fetches domain objects from the repository, calls domain logic, and commits results.

### The three concerns that must be separated

| Concern | Lives in | Example |
|---------|----------|---------|
| Interface | Entrypoints | Parse JSON, set HTTP status codes, handle routing |
| Orchestration | Service Layer | Fetch batch, validate SKU, call allocate, commit |
| Business rules | Domain | "Can't allocate if available qty < order qty" |

A service function looks like this:

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

Note: takes primitive types (strings, ints), not domain objects or HTTP request objects. Uses an abstract UoW, not a real database session.

### What belongs in the service layer vs. domain

**Service layer** — orchestration only:
- Fetching objects from the repository
- Input validation that requires data lookup (e.g., "does this SKU exist?")
- Calling domain logic
- Committing the transaction
- Exception handling / mapping to application errors

**Domain** — business rules:
- Invariant enforcement ("available qty >= 0")
- Allocation algorithm ("pick earliest ETA batch")
- Domain-specific exceptions (`OutOfStock`)
- Complex state transformations on entities

**Common violation:** business rules creeping into the service layer. When a service function contains if/else chains that implement business logic rather than just orchestrating domain calls, the domain model is anemic.

### Flask endpoint after service layer
The endpoint becomes thin:

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

If an endpoint is doing more than: parse request → call service → format response, it's doing too much.

### Application Service vs. Domain Service
Confusing terminology:
- **Application Service** (service layer function): orchestrates a workflow — boring sequencing
- **Domain Service**: business logic that doesn't fit naturally on any entity (e.g., a tax calculator, an allocation function)

Both are "services" but they live in different layers and have different responsibilities.

### Project layout
```
src/
  domain/
    model.py          # Entities, value objects, domain exceptions
  service_layer/
    services.py       # Use case functions
    unit_of_work.py   # UoW abstractions
  adapters/
    orm.py            # SQLAlchemy mappings
    repository.py     # Repository implementations
  entrypoints/
    flask_app.py      # HTTP handlers
tests/
  unit/               # Domain + service layer tests (fast, no DB)
  integration/        # ORM + repository tests (need DB)
  e2e/               # Full HTTP tests (need DB + running server)
```

---

## 7. TDD: High Gear and Low Gear

### The test pyramid
```
        /\
       /  \   E2E (2-3 tests per feature)
      /----\
     /      \ Integration (DB, ORM, repository)
    /--------\
   /          \ Unit (domain + service layer) — the bulk
  /____________\
```

A healthy project has many fast unit tests, few integration tests, and very few E2E tests. An inverted pyramid (lots of E2E, few unit tests) produces slow feedback, brittle suites, and a fear of refactoring.

### Low gear: domain-layer tests
Use when:
- Designing core business logic for the first time
- Solving complex architectural problems
- You need design feedback ("is this hard to test because it's wrong?")

Trade-offs:
- High design feedback — test failures tell you your domain model is awkward
- Tightly coupled to implementation — brittle during refactoring
- Read like business documentation

### High gear: service-layer tests
Use for most feature work:

```python
def test_allocates_to_earliest_batch():
    uow = FakeUnitOfWork()
    services.add_batch("batch1", "LAMP", 100, today, uow)
    services.add_batch("batch2", "LAMP", 100, tomorrow, uow)
    result = services.allocate("o1", "LAMP", 10, uow)
    assert result == "batch1"
```

Trade-offs:
- Lower coupling — resilient to domain model refactoring
- Fewer tests needed for same coverage
- Less design feedback

### Fully decoupled service tests
Ideal service-layer tests use only:
- Primitive types as inputs (strings, ints, dates)
- `FakeUnitOfWork` (not a real database)
- Service functions only (no direct domain object instantiation)

If your service tests are importing domain classes directly and instantiating them, ask: is there a service function missing? The service layer might be incomplete.

### Rules of thumb
- One E2E test per feature (verifies integration)
- Many service-layer tests per feature (exhaustive coverage of cases)
- Small focused set of domain tests (high-value design verification)
- Treat error handling as its own feature category — one E2E unhappy path, multiple unit tests for error logic

---

## 8. Unit of Work Pattern

### What it is
An abstraction for atomic operations. If the Repository abstracts storage, the Unit of Work abstracts a transaction — it groups operations so they all succeed or all roll back.

It also serves as a single entry point to repositories, so callers don't need to juggle repository and session objects separately.

### Interface

```python
class AbstractUnitOfWork(abc.ABC):
    products: AbstractProductRepository

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.rollback()  # Safe default — uncommitted work is rolled back

    @abc.abstractmethod
    def commit(self):
        raise NotImplementedError

    @abc.abstractmethod
    def rollback(self):
        raise NotImplementedError
```

Usage:

```python
with uow:
    product = uow.products.get(sku)
    product.allocate(line)
    uow.commit()
```

### Explicit commit philosophy
The default behavior on `__exit__` is **rollback**. Committing requires an explicit call. This makes the system "safe by default" — any exception, early return, or forgotten commit automatically undoes the operation. There is exactly one code path that leads to state changes: a complete, successful execution followed by explicit `uow.commit()`.

This is better than implicit auto-commit because:
- Failures are safe by default
- Transaction boundaries are visible in code
- Multiple operations can be grouped atomically

### SQLAlchemy implementation

```python
class SqlAlchemyUnitOfWork(AbstractUnitOfWork):
    def __init__(self, session_factory=DEFAULT_SESSION_FACTORY):
        self.session_factory = session_factory

    def __enter__(self):
        self.session = self.session_factory()
        self.products = SqlAlchemyProductRepository(self.session)
        return super().__enter__()

    def __exit__(self, *args):
        super().__exit__(*args)
        self.session.close()

    def commit(self):
        self.session.commit()

    def rollback(self):
        self.session.rollback()
```

### Fake UoW for testing

```python
class FakeUnitOfWork(AbstractUnitOfWork):
    def __init__(self):
        self.products = FakeProductRepository([])
        self.committed = False

    def commit(self):
        self.committed = True

    def rollback(self):
        pass
```

`uow.committed` lets tests verify that the service function actually committed.

### Don't mock SQLAlchemy's Session
The "don't mock what you don't own" principle: mocking SQLAlchemy's `Session` directly couples tests to a third-party API's internals. If SQLAlchemy changes its internal structure, tests break even if your code works correctly. By abstracting your own UoW, you only need to fake *your own* interface.

### The UoW replaces raw session passing
Before UoW, service functions accept a `session` parameter — a raw SQLAlchemy concept. This leaks infrastructure into the service layer.

After UoW, service functions accept an `AbstractUnitOfWork` — a concept your codebase owns. Sessions are managed internally by the UoW.

---

## 9. Aggregates and Consistency Boundaries

### The concurrency problem
Without aggregates, enforcing an invariant like "available quantity >= 0" under concurrent load either requires locking an entire table (kills performance) or risks race conditions (violates the invariant). Two requests can both read `qty=10`, both decide allocation is valid, and both commit — resulting in `qty=-10`.

The insight: CHAIR allocations don't conflict with DESK allocations. We only need consistency within a product's stock, not across the entire batches table. We need a **consistency boundary** narrower than "everything."

### What is an aggregate?
An aggregate is a cluster of domain objects treated as a unit for data changes:
- Has a **root entity** (the single entry point for all modifications)
- All invariants within the boundary are enforced together
- External code only touches the root, never child objects directly

### Choosing the right aggregate
The allocation system candidates:
- `Shipment`: too coarse — groups unrelated SKUs
- `Warehouse`: even coarser — everything in stock
- `Product` (by SKU): correct — all batches for one SKU form a natural consistency boundary

The right aggregate is the smallest unit that can enforce all its invariants internally.

### One repository per aggregate
Only aggregates get repositories. Repositories should never return child objects (e.g., individual `Batch` objects) directly — callers must go through the aggregate root (`Product`).

```python
# WRONG — returns a non-aggregate
class BatchRepository:
    def get(self, reference) -> Batch: ...

# CORRECT — returns the aggregate root
class ProductRepository:
    def get(self, sku) -> Product: ...
```

This enforces the convention that aggregates are the only public interface into the domain model. If you're writing `BatchRepository`, you probably haven't identified your aggregates yet.

### Version numbers for optimistic concurrency

```python
class Product:
    def __init__(self, sku: str, batches: List[Batch], version_number: int = 0):
        self.sku = sku
        self.batches = batches
        self.version_number = version_number

    def allocate(self, line: OrderLine) -> str:
        try:
            batch = next(b for b in sorted(self.batches) if b.can_allocate(line))
            batch.allocate(line)
            self.version_number += 1  # Increment on every mutation
            return batch.reference
        except StopIteration:
            raise OutOfStock(f"Out of stock for sku {line.sku}")
```

The database enforces: if two concurrent transactions both read `version=3` and both try to write `version=4`, only one succeeds. The other gets a concurrency error and must retry.

**Optimistic vs. pessimistic locking:**
- **Optimistic** (version numbers): assumes conflicts are rare. Detect and retry on failure. Best for high-read, low-conflict workloads.
- **Pessimistic** (`SELECT FOR UPDATE`): prevents conflicts. One transaction waits for the other. Best for high-conflict workloads. No retries needed but serializes throughput.

### Bounded contexts
The same word can mean different things in different parts of the system. "Product" in an allocation service contains only `sku` and `batches`. "Product" in an e-commerce service contains `price`, `description`, `image_url`.

Domain models should include only the data they need for their calculations. Don't build a universal "Product" that combines all meanings — build focused models per bounded context. This maps naturally to microservices, where each service owns its own domain model.

---

## 10. Signs of a Healthy Architecture

When these are all true, the architecture is sound:

**Domain model:**
- [ ] Business rules are expressed as methods on entities and value objects
- [ ] No imports of ORM, HTTP frameworks, or infrastructure in `domain/`
- [ ] Exceptions are domain-specific (`OutOfStock`, `InvalidSku`)
- [ ] Entities implement `__eq__` and `__hash__` based on identity
- [ ] Value objects are frozen dataclasses

**Repository:**
- [ ] `AbstractRepository` defines a narrow interface (add, get)
- [ ] `FakeRepository` is trivial to write and use in tests
- [ ] ORM mapping is defined separately from domain objects (classical mapping)
- [ ] Domain code never references SQLAlchemy columns or Base

**Service layer:**
- [ ] Each service function corresponds to one use case
- [ ] Service functions take primitives (str, int) not HTTP request objects
- [ ] Service functions take `AbstractUnitOfWork`, not a database session
- [ ] Entrypoints are thin: parse input → call service → format output
- [ ] No business rules in service functions (only orchestration)

**Unit of Work:**
- [ ] `AbstractUnitOfWork` is defined in the service layer
- [ ] Commit is explicit; rollback is the safe default
- [ ] `FakeUnitOfWork` used in unit tests, never raw SQLAlchemy sessions
- [ ] Service functions use `with uow:` blocks

**Aggregates:**
- [ ] Consistency boundaries are identified and named
- [ ] One repository per aggregate root
- [ ] Child objects (e.g., `Batch`) are not returned by repositories directly
- [ ] Version numbers increment on every mutation for optimistic concurrency

**Testing:**
- [ ] Business logic can be tested without database, HTTP client, or frameworks
- [ ] Test pyramid is healthy (many unit, few integration, very few E2E)
- [ ] Service layer tests use `FakeUnitOfWork` and primitives only
- [ ] Fakes, not mocks, for code you own

---

## 11. Common Violations Catalog

### Business logic in endpoint handlers
**What it looks like:** Flask/FastAPI route functions containing if/else chains that implement rules, SQL queries, or complex state calculations.
**Why it matters:** Logic cannot be tested without spinning up HTTP. Business rules are hidden inside infrastructure code. Different endpoints may duplicate logic.
**Fix:** Extract to a service function (if it's orchestration) or a domain method (if it's a business rule).

### Anemic domain model
**What it looks like:** Domain classes that are pure data containers. All logic in service classes that reach into domain object fields.
**Why it matters:** The domain model provides no value — it's just a struct. Business rules scatter across service functions.
**Fix:** Move logic onto domain objects. `product.allocate(line)` instead of `allocate_service.do_allocate(product, line)`.

### ORM classes as domain model
**What it looks like:** Domain entities inherit from `SQLAlchemy.Base` or `django.db.models.Model`.
**Why it matters:** Changing the ORM requires touching domain code. Domain objects carry infrastructure baggage (session state, lazy loading). Domain model tests require database setup.
**Fix:** Use SQLAlchemy classical mapping (`mapper_registry.map_imperatively`). Domain objects become plain Python classes.

### Raw session in service layer
**What it looks like:** Service functions accept a `session: Session` parameter and call `session.query(...)` directly.
**Why it matters:** Service layer is coupled to SQLAlchemy's API. Testing requires a real or mocked SQLAlchemy session. The service layer "knows" how data is stored.
**Fix:** Introduce `AbstractUnitOfWork`. Service functions call `uow.products.get(sku)`, not `session.query(Product)`.

### No abstraction between service and infrastructure
**What it looks like:** Service functions import `SqlAlchemyRepository` directly rather than receiving an `AbstractRepository`.
**Why it matters:** Cannot swap implementations without changing service code. Tests require a database.
**Fix:** Inject repository/UoW through function parameters. Wire concrete implementations at the application boundary.

### Multiple repositories for related objects
**What it looks like:** `BatchRepository`, `OrderLineRepository`, `AllocationRepository` all existing separately.
**Why it matters:** No clear aggregate root. Consistency cannot be enforced — callers can modify `Batch` directly without going through `Product`. Concurrent modifications are uncontrolled.
**Fix:** Identify the aggregate root. Create one repository for the aggregate. Reach child objects through the aggregate.

### Missing service layer (logic in endpoints)
**What it looks like:** All use-case logic in Flask view functions / FastAPI route handlers.
**Why it matters:** No way to reuse use cases from CLI, workers, or other entrypoints. Testing requires HTTP. Business logic changes require touching web framework code.
**Fix:** Extract use cases to service functions. Make endpoints thin wrappers.

### Test mocking infrastructure you own
**What it looks like:** `@patch('myapp.repository.SqlAlchemyRepository')` or `mock.patch('sqlalchemy.orm.Session')`.
**Why it matters:** Tests verify implementation details, not behavior. Tests break when internal implementation changes. No design benefit.
**Fix:** Use `FakeRepository` and `FakeUnitOfWork` instead. Mock only third-party APIs you don't control.

### Over-engineering simple CRUD
**What it looks like:** Full DDD stack (aggregates, UoW, repositories) for a feature that's just storing and reading records with no business rules.
**Why it matters:** Adds complexity with no payoff. The book says simple apps don't need these patterns.
**Fix:** Use the framework's built-in ORM for simple CRUD. Reach for these patterns when business logic complexity justifies it.
