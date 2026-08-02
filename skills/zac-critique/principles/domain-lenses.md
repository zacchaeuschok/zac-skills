# Domain Lenses

Use these when critiquing domain modelling. They help detect dominated designs in how business logic is structured.

---

## Model integrity

Domain objects should express business behavior and rules, not merely mirror database tables or passive data containers. Methods should enforce business invariants. In Python, this means plain classes with business methods, not ORMs or dataclasses treated as the domain. Identifiers — class names, methods, variables, modules — should use the same vocabulary as the business domain. If developers say "item" but the domain says "order", or the code says "context" when the domain says "shipment", the translation creates friction. Names should come from the domain, not from developer convention. When model integrity is violated, the code forces developers to mentally translate between two vocabularies and reason about business rules that are scattered rather than expressed directly on the objects they govern.

## Infrastructure isolation

The domain layer must have zero infrastructure imports — no ORMs, database drivers, HTTP clients, or framework base classes. The ORM imports the model, never the reverse. Repositories should present simple collection semantics (add, get) hiding all persistence mechanics. If you cannot swap the storage implementation without changing the service layer, the abstraction is leaking. Service layers should fetch, delegate, and persist — nothing more. If a service function contains business rules, validation logic, or domain calculations, that complexity belongs in the domain model, not the orchestration layer. When infrastructure isolation is violated, domain logic becomes untestable without infrastructure, and infrastructure changes ripple into business logic.
