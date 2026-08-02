# Design Philosophy

These are not rules to check. They are a coherent worldview for detecting whether a simpler correct decomposition exists.

---

**Prevent information leakage aggressively.** Every module must hide all knowledge of its internal representation, invariants, policies, and edge cases. If a caller needs to "know how it works" to use it correctly, the abstraction has failed. Information leakage is the dominant root cause of long-term complexity because it creates invisible, brittle coupling.

**Make modules deep by design, not small by habit.** A module should encapsulate as much related behavior and policy as possible behind a minimal interface. Splitting functionality prematurely creates shallow abstractions that push complexity outward instead of absorbing it. Fewer, deeper modules dominate many small, "clean" ones in long-term systems.

**Push complexity downward into modules, never upward into callers.** If code is hard, the hard part belongs inside a module, not duplicated or reasoned about at call sites. Callers should express what they want, not how to do it safely. Any design that forces callers to handle special cases is leaking complexity.

**Design interfaces to be complete and consistent, not minimal.** A "minimal" interface that requires workarounds or extra coordination is worse than a slightly larger one that covers real usage cleanly. Interfaces should support the common cases directly and uniformly, so users do not invent ad-hoc logic outside the abstraction. Completeness reduces accidental complexity more than sparsity.

**Consolidate related knowledge; avoid temporal and spatial coupling.** If a concept requires actions to occur in a specific order, or data to be split across modules, the design is flawed. Temporal coupling ("you must call A before B") and split responsibility are explicit design smells. One module should own sequencing, validation, and lifecycle whenever possible.

**Evaluate every design choice in the joint space of software complexity and product experience.** Good design minimises software complexity AND maximises a chosen dimension of product experience. A design is bad if it is *dominated*: higher software complexity with no corresponding gain in product experience. A design is bad if it increases software complexity without delivering a meaningful increase in product experience. When two designs deliver equivalent experience, the simpler one wins unconditionally.

---

The question is never "does this violate tenet X." The question is: given these tenets, is the current decomposition the simplest correct one? If a simpler decomposition exists that satisfies all tenets equally well, the current design is dominated.
