# The Nature of Complexity

Complexity is anything related to the structure of a software system that makes it hard to understand and modify. If a system is easy to understand and modify, it is simple. If it is hard, it is complex.

---

## Three symptoms

### Change amplification

A seemingly simple change requires modifications in many different places. This happens when related knowledge is scattered across modules instead of consolidated. The number of places that must change is the direct measure of amplification.

### Cognitive load

How much a developer must know to complete a task. High cognitive load means more time learning, more risk of missing something, and more bugs. Cognitive load arises from many sources: complex APIs, global variables, inconsistencies, dependencies between modules, and inadequate documentation.

More lines of code does not always mean more complexity. Sometimes an approach that requires more lines is simpler because it reduces cognitive load. Dense, clever code that fits in fewer lines can be harder to understand.

### Unknown unknowns

It is not obvious which pieces of code must be modified to complete a task, or what information a developer needs to have. This is the worst form of complexity because you do not know what you do not know. Unknown unknowns cause the most insidious bugs — the ones that appear long after a change was made, in a place no one expected.

## Two causes

### Dependencies

Code that cannot be understood or modified in isolation. A dependency exists whenever a piece of code relates to another piece of code. Some dependencies are fundamental and cannot be eliminated. The goal is to reduce the number of dependencies and make the remaining ones simple and obvious.

### Obscurity

Important information is not obvious. This includes non-obvious dependencies, inconsistent naming, missing documentation, and APIs where the meaning of parameters is unclear. Obscurity creates unknown unknowns.

## The accumulation model

Complexity is not caused by a single catastrophic decision. It accumulates in hundreds of small increments — each shortcut, each leaky abstraction, each piece of scattered knowledge adds a little. The only defense is a zero-tolerance approach: treat every increment of complexity as a cost and eliminate it when possible.

## The formula

Total complexity = sum of (complexity of each part × fraction of time developers spend on that part). Isolating complexity in a rarely-touched module is almost as good as eliminating it.

## Strategic vs. tactical programming

**Tactical programming** focuses on getting features working quickly. Each shortcut adds a small amount of complexity. The "tactical tornado" — a developer who cranks out code faster than anyone else — leaves a wake of complexity behind.

**Strategic programming** treats design as a primary goal. Working code is necessary but not sufficient. Invest 10-20% of development time in design improvements. This investment starts paying off within months.
