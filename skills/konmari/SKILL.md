---
name: konmari
description: Code review through Marie Kondo's KonMari principles — spark joy, discard before organizing, tidy by category, vision-driven design. Use when user says "konmari", "spark joy", "does this code spark joy", "tidy this code", "declutter", "what can I remove", "is this code joyful", "kondo my code", or wants to evaluate whether code is intentional, minimal, and purposeful.
---

# KonMari Code Review

Apply Marie Kondo's tidying philosophy to code. This is not about minimalism — it is about **choosing deliberately what belongs** in the codebase you are building.

## Trigger

Run when user invokes `/konmari` or asks to evaluate code through KonMari principles.

## Input

- A file, module, directory, or PR diff to review
- If no target specified, ask: "What code would you like to tidy?"

## Process

Follow the KonMari order strictly. Each phase completes before the next begins.

### Phase 0: Vision — "What life do you want to live?"

Before touching code, answer: **What should this codebase feel like to work in?**

Read the target code. Read surrounding context (tests, callers, types). Then state in 2-3 sentences what the code's ideal state looks like — its purpose, its natural shape, what "done" means. This vision anchors every decision.

### Phase 1: Clothes — The Obvious (Low Attachment)

Scan for items no one is emotionally attached to:

- [ ] Unused imports
- [ ] Unused variables and parameters
- [ ] Dead code (unreachable branches, impossible conditions)
- [ ] Redundant type assertions the compiler already knows
- [ ] Empty blocks, no-op catch clauses
- [ ] Console.log / debug print statements left behind

These are the worn-out socks. Thank them, remove them.

### Phase 2: Books — The "I Should" Code (Intellectual Guilt)

Code kept because someone *should* use it, not because anyone does:

- [ ] Commented-out code ("I might need this later")
- [ ] TODO/FIXME comments older than 3 months with no linked ticket
- [ ] Speculative abstractions — interfaces with one implementation, config for one value
- [ ] Premature generalizations — "just in case" parameters nobody passes
- [ ] Over-engineered utilities used exactly once

Kondo's insight: the book you never read already served its purpose — the moment you acquired it. The code you never call already taught you something. Let it go.

### Phase 3: Papers — The Bureaucratic (No One Loves Papers)

The organizational overhead that accumulates silently:

- [ ] Redundant or stale comments that restate the code
- [ ] Boilerplate that could be eliminated by a better abstraction
- [ ] Wrapper functions that just pass through
- [ ] Layers of indirection that add no value
- [ ] Configuration files for features that are always on or always off

Kondo's default for papers: **discard everything.** Keep only what is currently in use, needed for a limited period, or legally required.

### Phase 4: Komono — The Miscellaneous Sprawl

The largest category. Code that individually seems fine but collectively creates clutter:

- [ ] Functions doing two things (violation of single responsibility)
- [ ] Deep nesting (more than 3 levels) — violates vertical storage principle (everything visible)
- [ ] Inconsistent naming within the same module
- [ ] Mixed abstraction levels in the same function
- [ ] Dependencies that are used for one trivial operation
- [ ] Test helpers duplicated across files

Apply the category rule: address each type of issue *completely* across the scope, not file-by-file.

### Phase 5: Sentimental — The Code Someone Loved

The hardest. Code that carries emotional weight:

- [ ] Legacy implementations someone worked hard on but that are now superseded
- [ ] Clever solutions that are hard to read (the author's joy, not the reader's)
- [ ] Architectural decisions that made sense once but no longer do
- [ ] "Don't touch this, it works" code that everyone routes around

**Thank it before removing it.** Acknowledge what it solved. Note it in the commit message. Then let it go. The purpose of code is not permanence — it's to serve the system at a moment in time.

## Output Format

Write the review as a markdown document with this structure:

```markdown
# KonMari Review: [target]

## Vision
[2-3 sentences: what this code should feel like when it's right]

## Findings

### Phase 1: Clothes (obvious removals)
- [item]: [file:line] — [what to remove and why]

### Phase 2: Books (intellectual guilt)
- [item]: [file:line] — [what it taught, why it can go]

### Phase 3: Papers (bureaucratic overhead)
- [item]: [file:line] — [what to simplify]

### Phase 4: Komono (accumulated sprawl)
- [item]: [file:line] — [what to consolidate]

### Phase 5: Sentimental (the hard ones)
- [item]: [file:line] — [gratitude note + recommendation]

## The Click Point
[Describe what "done" looks like — when this code reaches its natural minimal state]

## Joy Check
[Final assessment: does this code, as-is, spark joy? What's the gap?]
```

Omit empty phases. If a phase has no findings, the code already sparks joy in that dimension.

## Principles (Reference)

See [PRINCIPLES.md](PRINCIPLES.md) for the full KonMari-to-code mapping.

## Rules

1. **Discard before organizing.** Never suggest refactoring until removal is complete.
2. **Tidy by category, not location.** Group findings by type, not by file.
3. **"Might need it someday" is not a reason to keep code.** If it's needed, it can be rewritten. If it's important, it's in git history.
4. **Thank before removing.** Every sentimental removal gets a gratitude note.
5. **The vision anchors everything.** If you can't articulate what "done" looks like, you're not ready to tidy.
6. **Joy is not pleasure — it's resonance.** Joyful code isn't clever or pretty. It's code that belongs.
