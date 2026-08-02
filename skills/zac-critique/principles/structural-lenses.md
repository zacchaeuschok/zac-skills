# Structural Lenses

Use these when you suspect a design is dominated. They help you build the argument.

---

## Boundary quality

A boundary that hides real complexity is valuable. A boundary that merely moves code to a different file is dominated by no boundary at all. Signs of a dominated boundary: the module has a complex interface (many parameters, many exports) but a trivial implementation that mostly delegates elsewhere (shallow module). Functions exist solely to forward calls to another function with near-identical parameters, adding no logic (pass-through). A single function mixes high-level orchestration with low-level I/O in the same body, meaning the boundary was drawn at the wrong altitude (mixed abstraction levels). When you see these, ask: would merging two modules produce a simpler system with identical behavior?

## Knowledge distribution

A concept should live in exactly one place. Signs that it doesn't: a single logical change requires edits across many files (change amplification). A validation rule, business constraint, or data format check is duplicated across modules instead of having a single authority (scattered invariants). A design decision — data format, configuration structure, business rule — is owned by multiple modules (duplicated design decisions). It is ambiguous which module would own a future change to a given feature (unclear ownership). When you see these, the modules were partitioned wrong — the concept was split where it should have been consolidated.

## Interface design

An interface should express the abstraction, not the implementation. Signs of a leaking or incomplete interface: identifiers encode storage mechanisms or data types rather than domain meaning (implementation-leaking names). The code adds if-branches, flags, or conditional paths for edge cases that could be eliminated by choosing a better representation (special-case sprawl). The code breaks naming conventions or structural patterns used elsewhere without justification (inconsistency). A low-level module contains application-specific policy that belongs in a higher layer (policy pushed down instead of up). When you see these, the interface is forcing callers to understand internals or work around gaps.

## Readability

Only relevant after structural issues are resolved. Comments should document intent, constraints, and non-obvious tradeoffs — not restate what the code does. A new reader should be able to understand what the code does and where to make changes without tracing through multiple indirections. If the common case is not quickly inferable from the interface and names, simplify.
