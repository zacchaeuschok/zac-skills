---
name: zac-critique
description: Design critique through dominated-design detection and structural complexity analysis. Reads the code, reads the requirements, then asks — is this implementation more complex than the requirements demand? Scans for Ousterhout's 14 red flags (shallow modules, information leakage, pass-through methods, etc.) as evidence of domination. Use when user says "critique", "assess", "evaluate", "review design", "is this clean", "can this be simpler", "is this over-engineered", "why is this so complicated", "what would you simplify", "this feels wrong", "is this proportional", "too many abstractions", "review what I just built", "ousterhout", "complexity review", "is this deep enough", "shallow module", "information leaking", "red flags", "design review", "why is this hard to change", "this code is confusing", or when the user asks you to look at code architecture or design quality.
argument-hint: "[path or component] [--requirements path/to/spec]"
model: opus
effort: high
---

# /zac-critique

Dominated-design detection, not linting.

The question is never "does this code violate principle X." The question is: **is this implementation more complex than the requirements demand?** A design is dominated when a simpler alternative exists that delivers equal or better product experience. Find the dominated designs. Name the dominating alternatives.

## Input

`$ARGUMENTS` — a target and an optional requirements spec.

**Target** (what to review):
- A component name: `/zac-critique the engine`
- A concern: `/zac-critique how tasks are configured`
- A file path: `/zac-critique src/domain/engine.py`
- Nothing: reviews changed files via `! git diff --name-only main`

**Requirements** (what to measure against):
- `--requirements path/to/spec.pdf` — the assignment brief, PRD, or spec. When provided, the critique explicitly measures implementation complexity against what was asked for.
- If omitted, infer requirements from the code itself.

**Output**:
- `--save` — write the critique to a markdown file. Without this flag, output the critique directly in the activity.
- `--save path/to/file.md` — write to a specific path.

## How to think

### Step 1: Understand the requirements

If `--requirements` is provided, read it. If not, infer from the code: what problem is this solving, what are the inputs and outputs, what are the user-facing behaviors?

Write down the requirements in 3-5 sentences. This is the baseline against which complexity is measured. Everything that follows asks: is the implementation proportional to this?

### Step 2: Map the implementation

Read the code, its callers, its neighbours. Document what exists without evaluating it:

- How many modules? What are their names?
- What are the interfaces between them? What data crosses boundaries?
- What is the control flow from entry point to completion?
- How many concepts must a reader hold in memory to trace a single path?

### Step 3: Identify dominated designs

Read `${CLAUDE_SKILL_DIR}/principles/design-philosophy.md` and `${CLAUDE_SKILL_DIR}/principles/dominated-designs.md`.

Scan for Ousterhout's red flags by reading `${CLAUDE_SKILL_DIR}/principles/red-flags.md`. Use them as detection heuristics — each red flag (shallow module, information leakage, temporal decomposition, pass-through method, etc.) is a signal that a dominated design likely exists. Also read `${CLAUDE_SKILL_DIR}/principles/complexity.md` to classify the damage each finding causes (change amplification, cognitive load, or unknown unknowns).

For each significant design choice in the implementation, ask: **does a simpler alternative exist that delivers equal or better product experience?**

For each dominated design found:
1. Name the current choice (D1) — what the code does.
2. Name the dominating alternative (D2) — what would replace it.
3. Show that D2 is strictly simpler — fewer modules, fewer interfaces, less state, fewer concepts.
4. Show that D2 delivers the same or better behavior.
5. Classify the complexity type: **change amplification**, **cognitive load**, or **unknown unknowns**. If a red flag triggered the detection, name it (e.g. "Red flag: shallow module").

Use the lens documents (`${CLAUDE_SKILL_DIR}/principles/structural-lenses.md`, `${CLAUDE_SKILL_DIR}/principles/agent-lenses.md`, `${CLAUDE_SKILL_DIR}/principles/domain-lenses.md`) as supporting evidence for domination arguments. Never cite a lens as a standalone finding.

If the target code is Python, also read `${CLAUDE_SKILL_DIR}/principles/lang/python-idioms.md` for language-specific patterns that may indicate domination. If TypeScript, read `${CLAUDE_SKILL_DIR}/principles/lang/typescript-idioms.md`. These supplement the structural lenses with idiomatic alternatives the language provides natively.

If you cannot name a concrete D2, you have not found domination. Move on.

### Step 4: Ask pointed questions

For each dominated design, formulate 1-2 questions the author would need to answer to justify their choice. These are not suggestions ("have you considered..."). These are questions where the honest answer reveals the problem:

- "How do you decide what belongs in this set, and how do you know when it's done?"
- "What type boundary exists between X and Y?"
- "What is the explicit interface contract between A and B, and how is it enforced?"
- "Which code reads this field, and what behavior changes if you remove it?"

### Step 5: Write the assessment

By default, output the critique directly in the activity session.

If `--save` is passed, write to a markdown file:
- `--save` with no path: write to `<target-dir>/CRITIQUE.md` (or `./CRITIQUE.md` if the target is within the current project).
- `--save path/to/file.md`: write to the specified path.

Follow the **writing rules** below, then use this structure:

```markdown
# Design Critique: <target>

## Score: <N>/10

<One sentence: the verdict. Why this score and not higher.>

## Requirements

<What was asked for. One sentence per requirement. No more than 5.>

## Findings

> The following analysis is sequential. Each section follows the
> call chain from where the previous section left off.

### Finding N: <Verdict as a sentence — what is wrong>

<1-2 sentences: what the problem is and why it matters.>

<Then a sequential step-by-step trace through the actual code path
that demonstrates the problem. See Writing Rules below for format.>

#### What replaces it

<Show the replacement code. Concrete. Then state the cost delta
(what is removed) and behavior delta (identical / better).>

**Complexity type:** <change amplification | cognitive load | unknown unknowns>
**Red flag:** <if applicable — name the Ousterhout red flag that signaled this>

**Question:** <One pointed question.>

### Finding N+1: ...

## Proportionality

<Is the implementation proportional to the requirements?
One paragraph, max 4 sentences.>
```

## Writing Rules

### Core principle: trace the call chain

Each finding is a **sequential walkthrough** of the actual execution path that demonstrates the problem. The reader should be able to follow the flow from entry point to the point where the design mistake manifests, without opening the codebase.

### How to write a finding

1. **Start with the verdict.** The finding heading is a sentence that states what is wrong. Not a label. Wrong: "Task routing approach." Right: "Task routing is hardcoded, making the configuration fields dead code."

2. **Follow with 1-2 sentences** explaining what the problem is and why it matters.

3. **Then trace the execution path step by step.** Each step is a numbered sub-heading (`#### Step 1.`, `#### Step 2.`, etc.) that shows:
   - The function signature or code block at that point in the chain
   - A short paragraph (1-3 sentences) explaining what this code does and where control goes next
   - What data is produced and where it is stored

4. **Each step picks up where the previous step left off.** Step 2 is called by Step 1. Step 3 is called by Step 2. The reader follows a single thread of execution. If two steps are not sequential, say explicitly how control gets from one to the other.

5. **Show the actual code.** Use fenced code blocks with the real function signatures and bodies (trimmed to the relevant parts). Do not describe code in prose when you can show it. Annotate with inline comments (`# ← explanation`) to highlight the key lines.

6. **End each finding with "What replaces it."** Show the replacement code in a code block. State the cost delta as a table or one-liner (e.g. "11 fields and 11 methods removed"). State the behavior delta ("identical behavior").

### How to write prose

7. **Lead with the conclusion.** The first sentence of every section is the answer. Everything after it is evidence.

8. **Score goes at the top.** The reader's first question is "how did it go?" Answer immediately.

9. **Short sentences.** Maximum 25 words. If a sentence needs a semicolon, it is two sentences.

10. **No hedge words.** Do not write "somewhat", "arguably", "notably", "it could be said that." State the fact.

11. **No paragraphs longer than 4 sentences.**

12. **Use tables for inventories.** Module lists, field lists, cost summaries — tables, not inline enumeration. Never list more than 3 items inside a sentence.

13. **Group related findings.** If three findings share a root cause, say so and group them. MECE: no overlap, no gaps.

## Scoring Calibration

- **9-10**: The simplest correct decomposition, or within one small refactor of it. No dominated designs found.
- **7-8**: Sound architecture with minor dominated choices. The right bones. A few unnecessary indirections or dead abstractions.
- **5-6**: The implementation works but contains significant dominated designs. Not the decomposition you would choose from scratch. The architecture is defensible but not optimal.
- **3-4**: The decomposition is wrong. Multiple dominated designs. The code works but the architecture fights the problem rather than fitting it.
- **1-2**: Fundamental misunderstanding of the problem or the solution space. More complex than a correct solution AND delivers worse behavior.

**Working code with the wrong decomposition is a 4-5, not a 7-8.** Correctness is necessary but not sufficient. The question is whether the decomposition is the simplest correct one.

**Do not score above 6** if the implementation contains more components, boundaries, or abstraction layers than the requirements demand.

## When no dominated designs are found

If the implementation is the simplest correct decomposition — or close to it — say so directly. Omit the Findings section entirely. Write the Requirements and Proportionality sections, score 9-10, and explain why no simpler alternative exists. Inventing weak findings to fill a template undermines the critique's credibility.

## Constraints

- By default, output the critique in the activity. Only write to a file when `--save` is passed.
- Every finding must trace the execution path with actual code — so the reader never needs to open the codebase to understand the problem.
- Every finding must show replacement code and state what is removed. Without a concrete replacement, it is not a finding — it is a preference.
- Show code in fenced blocks. Prose descriptions of code waste the reader's time when the code itself is clearer.
- Do not soften findings with praise. The author wants the truth, not encouragement.
- Do not list principle violations as standalone findings. Principles are lenses for detecting domination, not findings in themselves.
- Do not flag verbose-but-obvious code. Clarity beats brevity.
