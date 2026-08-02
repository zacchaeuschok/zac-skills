# KonMari Principles Applied to Code

The full mapping from Marie Kondo's philosophy to software.

## The 6 Rules, Translated

### 1. Commit to tidying
Half-hearted cleanup produces half-hearted results and leads to rebound. A refactor that stops halfway leaves the codebase worse — two patterns where there should be one. When you start, finish the category completely.

### 2. Envision your ideal codebase
Before changing anything: what should this code feel like to work in? What does a new team member see when they open this module? What does "done" look like? Without this vision, you'll optimize locally but miss the whole.

### 3. Discard first, organize second
Remove dead code, unused dependencies, and speculative abstractions **before** refactoring. Organizing clutter is still clutter. The most common failure mode: creating elaborate folder structures and abstractions around code that should have been deleted.

### 4. Tidy by category, not by location
Don't review file-by-file. Review by concern:
- First pass: all unused imports across the codebase
- Second pass: all dead code paths
- Third pass: all speculative abstractions

This reveals patterns invisible in file-by-file review. Seeing 14 unused utility functions across 8 files tells a different story than finding 1-2 per file.

### 5. Follow the right order
Start with easy, low-attachment items (unused imports). End with sentimental items (legacy architecture). The order builds your judgment. If you start with the hardest decisions, you'll keep everything.

**The order:**
1. Clothes → obvious dead code, unused imports
2. Books → "should use someday" code, commented-out blocks
3. Papers → bureaucratic overhead, redundant comments
4. Komono → mixed concerns, sprawl, inconsistency
5. Sentimental → beloved legacy code, clever-but-opaque solutions

### 6. Ask: does it spark joy?
For code, joy means: **Does this serve the system's purpose? Is it clear? Is it intentional?**

Joy is not:
- Cleverness (clever code sparks the author's joy, not the reader's)
- Conciseness for its own sake (terse code can be hostile)
- Following patterns religiously (patterns are tools, not goals)

Joy is:
- A function whose name tells you what it does
- A module with a clear boundary and a reason to exist
- A test that explains the "why" of a behavior
- Code that a new team member can read without asking questions

## Deep Principles

### "Might need it someday" is the engine of code clutter
Every codebase has speculative code — abstractions for flexibility never used, parameters for config never varied, branches for cases that never occur. This code was kept out of **anxiety about the future**, not service to the present.

Kondo's observation: the feared scenario almost never materializes. And when it does, the old code is usually wrong for the actual need anyway. Git history exists. Delete with confidence.

### Thanking code before removing it
This sounds strange for software. It isn't.

When you remove legacy code, acknowledge what it solved. Put it in the commit message: "Remove the original caching layer — this kept us running through the 2023 traffic spike. The new system supersedes it." This:
- Honors the author's work
- Creates a historical record of *why* it existed
- Makes the removal feel like graduation, not destruction
- Processes the team's emotional attachment

### The rebound effect
Codebases that are "cleaned up" without a clear vision rebound. New clutter fills the vacuum because there's no filter for what belongs. A style guide isn't enough — you need a shared understanding of what this codebase is *for* and what working in it should feel like.

### Respect your code
Kondo personifies objects. For code: treat it with care.
- Name things honestly (a function named `process` doesn't respect the reader)
- Format consistently (crumpled code is disrespected code)
- Keep it visible (deep nesting and hidden side effects are stacking, not filing)
- Don't let it rot in a corner unvisited (stale modules attract more staleness)

### Vertical storage — everything visible
Kondo stores items upright so nothing is hidden under a stack. In code:
- Prefer flat structures over deep nesting
- Prefer explicit over implicit
- Prefer early returns over nested conditionals
- Every branch of logic should be visible at a glance, not buried four levels deep

### The click point
The moment a module reaches its natural, minimal, correct state. You feel it — there's nothing to add, nothing to remove. The code does exactly what it should, no more. Not every module reaches this. But knowing it exists changes how you evaluate code.

### Storage is not the solution
Kondo is skeptical of organizational products. The storage industry profits from people who organize without discarding.

In code: tooling, wrappers, and framework abstractions don't fix architectural clutter. A linter that auto-fixes formatting doesn't address a module that shouldn't exist. A wrapper around a bad API is still a bad API. Fix the root cause.

### You can only tidy your own belongings
Kondo forbids tidying someone else's stuff — it's a boundary violation. You inspire by example.

In code: if you're reviewing someone else's code, identify what doesn't spark joy, but don't unilaterally rewrite their module. Suggest. Explain. Show. The author does the tidying.

## The Joy Spectrum

Not all code needs to spark the same kind of joy:

| Code Type | What Joy Looks Like |
|---|---|
| Business logic | Clear intent, reads like a specification |
| Infrastructure | Invisible — you never think about it, it just works |
| Tests | Confidence — you trust them to catch regressions |
| APIs | Predictable — callers know what to expect without reading source |
| Configuration | Minimal — only what actually varies is configurable |
| Error handling | Honest — tells the user/developer exactly what went wrong |

## Anti-Patterns: Code That Never Sparks Joy

- **The "just in case" abstraction**: An interface with one implementation, created for "flexibility"
- **The guilt closet**: A `utils/` or `helpers/` folder where orphaned code goes to hide
- **The memorial**: Code preserved because someone important wrote it
- **The anxiety drawer**: Defensive code for scenarios that can't happen in practice
- **The aspirational library**: Code written for a future that never came
- **The borrowed identity**: Patterns copied from other projects without understanding why they exist here
