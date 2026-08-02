# Agent Lenses

Use these when critiquing LLM-based or agent-based systems. They help detect dominated designs in AI/agent architectures.

---

## Premature structure

Creating categories, options, and taxonomies before usage justifies them is a dominated design. Signs: named states, labels, enums, or fields that never drive branching, routing, or observable behavior — naming something is not architecture (named-variable-theater). Elaborate intent enums, classification hierarchies, or state machines defined before usage patterns validate them — early taxonomies become constraints you cannot test (premature-taxonomy). Many configuration knobs, modes, strategies, or tool selections exposed before it is known which ones matter — premature optionality makes the simple path unclear (optionality-explosion). APIs designed around one imagined workflow rather than the smallest stable contract callers actually need — overfitted interfaces break when the workflow changes. When you see these, the simpler alternative is: start with one concrete path and let structure emerge from observed usage.

## Brittle control

Using low-fidelity signals for high-stakes decisions is a dominated design when higher-fidelity alternatives exist. Signs: keyword lists, regexes, or string-contains checks gating meaning-heavy decisions like intent classification or safety filtering — lexical heuristics are brittle proxies that create false positives and miss paraphrases (semantic-control-via-lexical-heuristics). Terminal actions (block, escalate, refuse) enforced based on detectors with unknown error rates — treating probabilistic signals as deterministic policy turns noise into incorrect enforcement (hard-guarantees-on-soft-signals). Rules or constraints expressed only as comments or docstrings without runtime enforcement — principles that exist only as text are aspirational, not operational (policy-as-text). When you see these, ask: is the signal's fidelity proportional to the decision's cost?

## Unbounded cost

Spending compute without knowing whether it helps is a dominated design when a bounded alternative delivers the same outcome. Signs: extra LLM calls for decide/check/critique/verify steps without evidence the additional pass improves correctness proportionally to cost (double-loop-inflation). Free-form planning or self-directed loops without iteration limits or termination conditions (unbounded-autonomy). Code shipped without a measurable success criterion — without a defined metric, you cannot tell whether changes improve the system or just change it (evaluation-deferred). When you see these, the dominating alternative is: set a hard ceiling, measure before adding loops, define "better" before building.
