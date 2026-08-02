# Python Idioms

Does the code use non-idiomatic Python patterns where the language offers better alternatives?

**Typing and interfaces.** Use `Protocol` for structural subtyping at module boundaries — accept a `Repository` protocol, not a `PostgresRepo` class. Keep protocols small: one capability per protocol (`Readable`, `Closable`), not monoliths. Use `dataclass` or `NamedTuple` for plain data; reserve `TypedDict` for unstructured dict interop (JSON payloads, legacy APIs). Prefer `@dataclass(frozen=True)` for value objects. Avoid `Any` — use `object` when the type is genuinely unknown, `Any` defeats static analysis. Use `Optional[X]` or `X | None` explicitly rather than relying on `None` default inference.

**Error handling.** EAFP (try/except) over LBYL (if-check-then-act) when the check and the action can race or when the check duplicates the operation. But keep try blocks narrow — only the line that can raise, not the whole function body. Catch specific exceptions, never bare `except:` or `except Exception`. Never silence exceptions with `pass` unless the intent is documented.

**Iteration.** Use `enumerate()` over manual index tracking. Use `zip()` to iterate parallel sequences. Prefer list/dict/set comprehensions for simple transforms — but do not nest comprehensions beyond one level; use a loop or extract a function. Use `itertools` for lazy pipelines over large sequences. Use `for/else` only when the `else` meaning (no `break`) is genuinely clearer than a flag variable.

**Resource management.** Always use `with` for resources that need cleanup (files, connections, locks). Write custom context managers via `@contextmanager` when setup/teardown logic is reused. Never rely on `__del__` for cleanup.

**Mutability.** Avoid mutable default arguments (`def f(x=[])`) — use `None` sentinel with `if x is None: x = []`. Prefer `tuple` over `list` for fixed-length heterogeneous data. Use `frozenset` when the set should not change after construction. Mark dataclass fields `field(default_factory=...)` for mutable defaults.

**Naming.** Use `snake_case` for functions, methods, variables, modules. Use `PascalCase` for classes. Use `UPPER_SNAKE_CASE` for module-level constants. Prefix private attributes with `_`, never `__` (name mangling) unless avoiding subclass collisions. Name booleans as predicates (`is_valid`, `has_access`), not nouns (`valid`, `access`).

**Imports and structure.** Prefer absolute imports. Avoid wildcard imports (`from x import *`). Group imports: stdlib, third-party, local — separated by blank lines. Do not import inside functions unless avoiding circular imports (and fix the circular import instead). Use `__all__` to declare public API when the module has internal helpers.

**Modern Python (3.10+).** Use `match/case` for multi-branch dispatch on structure, not chains of `isinstance` checks. Use `X | Y` union syntax over `Union[X, Y]`. Use `@dataclass(slots=True)` for memory-efficient data classes. Use `tomllib` for config parsing over hand-rolled solutions.
