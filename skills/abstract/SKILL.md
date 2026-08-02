---
name: abstract
description: Write ML/research paper abstracts using Neel Nanda's cold-start structure with built-in AI-writing decontamination. Use when user says "write an abstract", "draft abstract", "abstract for", "help with my abstract", "improve this abstract", "review my abstract", or provides a paper draft needing an abstract.
---

# Abstract Writer

Write research paper abstracts that orient cold-start readers fast, make specific claims backed by concrete evidence, and read like a human wrote them.

## Sources

- Neel Nanda, "Highly Opinionated Advice on How to Write ML Papers"
- Wikipedia, "Signs of AI writing"

## Trigger

Run when user invokes `/abstract` or asks to write/review/improve an abstract.

## Input

One of:
- A paper draft, partial or complete
- A description of the work (claims, results, method)
- An existing abstract to improve

If input is unclear, ask: "What did you do, what did you find, and why does it matter?"

## The Structure (6 sentences, give or take)

Follow Nanda's sentence-by-sentence blueprint:

### Sentence 1 — Orient the reader
Something uncontroversially true that places the reader in the right subfield. Not a grand statement about AI or science. Specific.

- Good: "Chain-of-thought prompting improves performance on multi-step reasoning tasks."
- Bad: "Large language models have revolutionized artificial intelligence."

### Sentence 2 — The gap or need
What's unknown, broken, or missing. This is the tension that justifies the paper.

- Good: "However, it remains unclear whether these chains faithfully reflect the model's internal computation."
- Bad: "Despite significant progress, challenges remain in this rapidly evolving field."

### Sentence 3 — Your contribution
State what you did and why it matters. Be specific. Name the method or finding. Fold in key definitions for jargon the reader needs.

One sentence, one idea. If you need two medium-long sentences, question that choice.

### Sentences 4-5 — Evidence
Your most concrete result. A number, a theorem, a comparison. Make the result real and substantial. Fold evidence into the claim sentence where possible rather than separating "we did X" from "we found Y."

- Good: "We find a single direction in residual stream space whose ablation reduces refusal rates from 97% to 3% across all tested harmful prompts."
- Bad: "Our experiments demonstrate promising results across several benchmarks."

### Sentence 6 — So what
Why this matters beyond your paper. Implications. How it connects to the broader research agenda. Signal your confidence level: is this preliminary, suggestive, or establishing a best practice?

## The Decontamination Rules

Every draft MUST be checked against these. They are not optional.

### Banned vocabulary (when clustered, any one in isolation is fine)
Do not use these words unless they are the technically precise term with no substitute:

`additionally, boasts, bolstered, comprehensive, crucial, cutting-edge, delve, diverse array, emphasizing, enduring, enhancing, ensures, evolving landscape, exemplifies, exploring, facilitating, fostering, furthermore, garnered, groundbreaking, highlighting, in the heart of, indelible mark, innovative, intricate/intricacies, it's worth noting, key (as adjective), landscape (metaphorical), leveraging, meticulous/meticulously, moreover, multifaceted, nestled, notable/notably, nuanced, pivotal, plays a crucial role, profound, renowned, rich tapestry, seamless, serves as, setting the stage, showcasing, significance, stands as, streamlining, testament, transformative, underscores, valuable, vibrant`

If you catch yourself writing two or more of these in the same abstract, rewrite from scratch.

### Banned structures
- **"Not just X, but also Y"** — sounds like a press release
- **Rule of three adjectives** — "robust, scalable, and efficient" is a tell
- **"Serves as / stands as / marks"** instead of "is" — just say "is"
- **"Boasts / features / maintains / offers"** instead of "has" — just say "has"
- **Present participle tacking** — "highlighting the importance of..." dangling off a sentence
- **Vague attribution** — "researchers have shown" (who? cite them or don't claim it)
- **Hedging stacks** — "It is important to note that this potentially suggests" — commit or don't
- **Significance escalation** — don't inflate. If it's a small result, say so honestly.

### Positive style rules
- Prefer "is" and "has" over elaborate substitutes
- Prefer short sentences. Vary length — a 6-word sentence after a 20-word sentence creates rhythm.
- Use concrete nouns and verbs. "We train a probe" not "We leverage a probing methodology."
- Write in active voice by default. Passive is fine when the agent is genuinely unimportant.
- One idea per sentence. Period. New sentence. New idea.
- Read it aloud. If you stumble, rewrite.

## Process

1. **Extract** — Read the input. Identify: subfield, gap, method, key result (with numbers), implication.
2. **Draft** — Write 5-7 sentences following the structure above.
3. **Decontaminate** — Run every sentence against the banned vocabulary and banned structures. Replace any violations.
4. **Read aloud test** — Read the draft as if you're explaining your paper to a colleague at lunch. Does every sentence land? Rewrite any that don't.
5. **Present** — Show the abstract with brief annotations explaining each sentence's role.

## Output Format

```
[The abstract — 5-7 sentences, no heading, no formatting, plain paragraph]

---
Annotations:
- S1 (orient): [why this sentence works]
- S2 (gap): ...
- S3 (contribution): ...
- S4-5 (evidence): ...
- S6 (so what): ...

Decontamination check: [list any flagged words that were removed/replaced, or "clean"]
```

## Reviewing an Existing Abstract

When the user provides an abstract to improve:

1. Identify which of the 6 sentence roles are present, missing, or muddled
2. Flag any AI-writing tells (banned vocab, structures, hedging)
3. Flag vague claims that should be concrete
4. Rewrite, preserving the author's voice where it's already good
5. Show before/after with annotations

## Rules

1. **No grand openings.** Never start with the importance of AI, ML, or science in general.
2. **Numbers or it didn't happen.** Every abstract needs at least one concrete metric.
3. **Jargon is fine when precise.** Don't dumb down technical terms that your audience knows. Don't use jargon to sound smart.
4. **Lose nuance gracefully.** An abstract can't capture everything. That's OK. Capture the one thing that matters most.
5. **Signal confidence honestly.** "We find preliminary evidence" and "we establish" mean different things. Use the right one.
6. **The decontamination rules are load-bearing.** An abstract that reads like AI wrote it undermines the author's credibility regardless of content quality.
