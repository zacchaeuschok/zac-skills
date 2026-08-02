# Dominated Designs

## Definition

A design choice D1 is **dominated** by an alternative D2 when:

- D2 has equal or lower software complexity, AND
- D2 delivers equal or better product experience.

A dominated design is never justified. It is strictly worse. The task of critique is to find these.

## How to Argue Domination

You must name both sides:

1. **D1** (current design): what the code does now.
2. **D2** (dominating alternative): what would replace it.
3. **Complexity comparison**: show that D2 is strictly simpler — fewer modules, fewer interfaces, less state, fewer lines, fewer concepts a reader must hold in memory.
4. **Experience comparison**: show that D2 delivers the same or better behavior from the user's perspective.

If you cannot name D2 concretely, you have not found domination — you have found a preference. Preferences are not findings.

## Common Patterns of Domination

### Open-ended input where constrained input suffices

```python
YES_WORDS = {"y", "yes", "yeah", "yep", "sure", "ok", "okay", "affirmative"}
NO_WORDS = {"n", "no", "nope", "nah", "negative"}
```

D1: Accept a synonym list for a binary decision. The input set is open-ended, never "done," and every addition permanently expands the behavior surface. This is an unbounded classification problem for a binary outcome.

D2: Constrained input — numbered options (`1 = Yes`, `2 = No`), or exact tokens (`yes` or `no` only) with a hard rejection of anything else. The input space is finite, closed, and final. Validation is constant-time and complete.

Why D2 dominates: software complexity drops (no synonym list, no fuzzy matching, no maintenance tax). Product experience improves (the user sees exactly what is expected and cannot "almost" answer incorrectly). D1 is strictly dominated.

### Multiple modules where one suffices

D1: Split a concept across files/classes that share all their data and change together. Each module has a thin interface and delegates to the others.

D2: One module that owns the concept end-to-end.

Why D2 dominates: fewer interfaces, zero coupling between the removed modules, same behavior. The split existed for organizational reasons (file size, "separation of concerns"), not because it hid real complexity.

### Pass-through abstraction layers

D1: A service calls a repository that calls a store. The repository adds no logic — it re-exposes the store's interface with different names.

D2: The service calls the store directly.

Why D2 dominates: one fewer abstraction layer, one fewer interface to learn, same behavior. The intermediate layer was defensive architecture for a future that never arrived.

### Premature generalization

D1: A config-driven, plugin-capable, strategy-pattern framework that currently has exactly one implementation.

D2: A direct implementation of the one case.

Why D2 dominates: the framework is more complex (config parsing, plugin loading, strategy dispatch), and the one case works identically either way. The generalization is speculative — it pays complexity cost now for flexibility that may never be needed.

### Configuration fields that are never read

D1: A data class declares fields (`requires_intent_confirmation`, `allows_delivery`, `successful_end_message`) that no code ever accesses. They imply a generic, configurable system.

D2: Remove the dead fields. The code is honest about what it does.

Why D2 dominates: fewer fields, less conceptual surface, identical runtime behavior. The dead fields are not architecture — they are aspirational comments disguised as data.

### State machines for linear flows

D1: An explicit FSM with a transition table, conditions, hooks, and triggers for a flow that is strictly sequential with no branching, no re-entry, and no concurrent states.

D2: A sequence of function calls.

Why D2 dominates: the FSM adds representation overhead (state enum, transition table, trigger dispatch) without enabling behavior the linear version cannot express. The FSM is only justified if the flow genuinely branches, loops, or requires state inspection by external callers.

Note: if the assignment *requires* an FSM, the FSM is not dominated — but the implementation should be proportional to the branching the FSM actually enables. An FSM that encodes one linear path through 15 states is dominated by an FSM with fewer, more meaningful states.

## What Domination Is Not

- "I would have done it differently" is not domination. You must show D2 is strictly simpler AND at least as good.
- "This is verbose" is not domination. Verbosity with clarity is not dominated by brevity with obscurity.
- "This violates principle X" is not domination. Principles are lenses for finding domination, not findings in themselves.
