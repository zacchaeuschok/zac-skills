---
name: strategy
description: Evaluates whether a strategy, plan, proposal, or product direction is genuinely good or merely dressed-up ambition. Uses Rumelt's kernel (diagnosis, guiding policy, coherent action) and the four hallmarks of bad strategy to produce a verdict. Use when user says "strategy", "is this a good strategy", "evaluate this plan", "strategy review", "is this bad strategy", "stress test this", "challenge this proposal", "review our roadmap", "does this plan make sense", "tear this apart", "what's wrong with this strategy", or presents a plan/strategy/roadmap/deck for critique.
argument-hint: "[path to document or paste text] [--save]"
model: opus
effort: high
---

# /strategy

Strategy evaluation, not cheerleading.

The question is never "is this plan ambitious enough." The question is: **does this strategy contain a real kernel — a diagnosis of the challenge, a guiding policy that addresses it, and coherent actions that execute it?** If any element is missing, it is bad strategy. If all three are present, evaluate whether each element is sound.

## Input

`$ARGUMENTS` — a strategy, plan, proposal, or product direction.

**Source** (what to evaluate):
- A file path: `/strategy docs/strategy.md`
- A pasted block of text in the conversation
- A concern: `/strategy our go-to-market plan`
- Nothing: asks the user what to evaluate

**Flags**:
- `--save` — write the assessment to a file
- `--save path/to/file.md` — write to a specific path

## How to think

### Step 1: Search for the kernel

Read `${CLAUDE_SKILL_DIR}/references/kernel.md` for the full framework — definitions, tests, and examples for each element.

Search the input for the three kernel elements: **diagnosis** (names the actual challenge), **guiding policy** (specifies an approach that rules out alternatives), and **coherent actions** (coordinated steps that reinforce each other).

If any element is missing, the strategy is incomplete at best. State which element is absent and what the consequences are.

### Step 2: Test for the four hallmarks of bad strategy

Read `${CLAUDE_SKILL_DIR}/references/bad-strategy.md` for the full detection criteria and examples.

Apply each test: **fluff** (buzzwords masking absence of thought), **failure to face the challenge** (no named obstacle), **goals mistaken for strategy** (desired outcomes without a mechanism), **bad strategic objectives** (infeasible or unfocused).

For each hallmark found, cite the specific text that triggers it.

### Step 3: Evaluate sources of power

Read `${CLAUDE_SKILL_DIR}/references/sources-of-power.md`.

If the strategy passes Steps 1 and 2, evaluate whether it leverages real sources of power:

- **Leverage** — Does the strategy concentrate effort on pivot points where small actions yield outsized impact? Or does it spread resources thinly across many fronts?
- **Proximate objectives** — Are the near-term targets achievable and concrete? Or are they as hard as the original problem?
- **Chain-link systems** — Does the strategy address the weakest link? Strengthening strong links while ignoring the bottleneck is wasted effort.
- **Coherence as advantage** — Do the actions form a reinforcing system that is hard to replicate piecemeal? Or are they independent initiatives a competitor could copy one at a time?
- **Advantage** — Does the strategy exploit a genuine asymmetry? What isolating mechanisms protect the advantage?

### Step 4: Apply the sniff tests

Read `${CLAUDE_SKILL_DIR}/references/tests.md`.

Run these tests against the strategy:

- **The Opposite Test** — Would a reasonable competitor choose the opposite? If not, it is too generic to be strategic.
- **The Bridge Test** — Does the strategy build a bridge between challenge and action? If the objectives are as hard as the original problem, no bridge exists.
- **The No Test** — Can you identify what the strategy says "no" to? If nothing is excluded, no real choice was made.
- **The Simplicity Test** — Good strategy looks simple and obvious in hindsight. Complexity masks weak thinking.
- **The Coherence Test** — Do actions reinforce each other, or are they independent initiatives?
- **The Focus Test** — Does the strategy specify what the organization will NOT do?

### Step 5: Write the assessment

By default, output in the conversation.

If `--save` is passed, write to a file:
- `--save` with no path: write to `./STRATEGY-REVIEW.md`
- `--save path/to/file.md`: write to the specified path

Use this structure:

```markdown
# Strategy Review: <target>

## Verdict: <Good Strategy | Bad Strategy | Incomplete Strategy>

<One sentence: the core judgment. What is present and what is missing.>

## The Kernel

### Diagnosis
<Is there a diagnosis? Quote it. Is it a real diagnosis (names the challenge, identifies root causes, simplifies complexity) or a pseudo-diagnosis (restates goals, describes symptoms, ignores obstacles)?>

### Guiding Policy
<Is there a guiding policy? Quote it. Does it specify an approach and rule out alternatives, or is it a goal/vision statement anyone would agree with?>

### Coherent Actions
<Are there coherent actions? List them. Do they reinforce each other, or are they independent initiatives?>

## Bad Strategy Hallmarks

| Hallmark | Found? | Evidence |
|----------|--------|----------|
| Fluff | Yes/No | <quote or "—"> |
| Failure to face the challenge | Yes/No | <quote or "—"> |
| Goals mistaken for strategy | Yes/No | <quote or "—"> |
| Bad strategic objectives | Yes/No | <quote or "—"> |

## Sources of Power

<Which sources of power does the strategy leverage? Which does it ignore? Is the advantage real and defensible?>

## Sniff Tests

| Test | Pass/Fail | Note |
|------|-----------|------|
| Opposite | | <Would a competitor choose the opposite?> |
| Bridge | | <Does it connect challenge to action?> |
| No | | <What does it say no to?> |
| Simplicity | | |
| Coherence | | |
| Focus | | |

## What Would Make This Good Strategy

<If the strategy is bad or incomplete, state what would fix it. Be specific: name the missing diagnosis, the absent guiding policy, or the incoherent actions. Do not suggest improvements the author did not imply — just name what is missing.>
```

## Edge Cases

**Good strategy with no issues.** If the kernel is complete, no hallmarks are found, and real sources of power are leveraged — say so. Omit the "What Would Make This Good Strategy" section. Do not manufacture findings to fill the template.

**Non-strategy input.** If the input is a to-do list, a project plan, a technical spec, or a set of goals without a kernel, say so upfront. Explain what distinguishes strategy from planning: a plan is a schedule of actions; a strategy is a diagnosis + guiding policy + coherent action.

**Partial or fragmentary input.** If only a fragment is provided (e.g., a single slide, a paragraph), evaluate what exists but note what is missing due to incomplete input vs. genuinely absent from the strategy.

**Scope.** This framework is rooted in competitive/business strategy (Rumelt's examples are IBM, IKEA, retail banks). It applies well to product strategy, go-to-market plans, and organizational strategy. For purely technical decisions ("should we rewrite in Rust?"), the kernel structure still works but the sources-of-power analysis may be less relevant.

## Writing Rules

1. **Lead with the verdict.** The reader's first question is "is this good or bad?" Answer immediately.

2. **Quote the source.** Every claim about the strategy must cite the specific text that supports it. Do not paraphrase — quote, then evaluate.

3. **Short sentences.** Maximum 25 words. If it needs a semicolon, it is two sentences.

4. **No hedge words.** Delete "somewhat," "arguably," "it could be said." State the fact.

5. **No paragraphs longer than 4 sentences.**

6. **Tables for inventories.** Three or more comparable items belong in a table.

7. **Do not soften the verdict.** Bad strategy is bad strategy. Do not praise effort, intent, or ambition. Evaluate the strategy, not the strategist.

8. **Do not add strategy.** The skill evaluates what exists. It does not propose new strategy. The "What Would Make This Good Strategy" section names what is missing — it does not fill the gap.

## Constraints

- By default, output in the conversation. Only write to a file when `--save` is passed.
- Every hallmark claim must cite specific text from the input.
- If the kernel is complete and no hallmarks are found, say so — do not manufacture findings.
- Do not evaluate execution feasibility beyond what is stated. Evaluate the strategy as written.
- Do not confuse strategy with planning. A plan is a schedule of actions. A strategy is a diagnosis + guiding policy + coherent action. A plan without a kernel is not strategy.
- Do not write sentences longer than 25 words.
- Do not write paragraphs longer than 4 sentences.
- Headings are verdicts, not labels where possible.
