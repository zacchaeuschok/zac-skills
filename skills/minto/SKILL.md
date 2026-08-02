---
name: minto
description: Transforms any plan, proposal, memo, or writing into structured communication using the Minto Pyramid Principle. Restructures around a single governing thought with SCQA introductions, MECE groupings, and inductive reasoning. Use when user says "minto", "pyramid", "structure this", "restructure", "make this clearer", "rewrite this memo", "tighten this up", "SCQA", "this is all over the place", "organize my thinking", "help me structure this", or provides a draft that needs structural clarity.
argument-hint: "[path to document or paste text] [--tone considered|direct|concerned] [--save]"
model: opus
effort: high
---

# /minto

Restructure writing into pyramid form. Not "improve the writing." Replace the structure.

The output is a pyramid document where every heading is a conclusion, every group is MECE, every level answers the question raised by the level above, and the reader can extract the full argument by reading headings alone.

## Input

`$ARGUMENTS` — a document and optional flags.

**Source** (what to restructure):
- A file path: `/minto docs/proposal.md`
- A pasted block of text in the conversation
- A concern: `/minto the investor update`
- Nothing: asks the user what to restructure

**Flags**:
- `--tone considered|direct|concerned` — controls SCQA ordering (default: `direct`)
  - `considered`: Situation → Complication → Question → Answer
  - `direct`: Answer → Situation → Complication
  - `concerned`: Complication → Situation → Answer
- `--save` — write the restructured document to a file
- `--save path/to/file.md` — write to a specific path

## Procedure

### Step 1: Extract the governing thought

Read the input. Find the single point. Write it as one sentence. This sentence is the apex. Everything in the output exists to support it.

If the input has no single governing thought, say so and stop.

If the input is very short (under ~50 words or a bare bullet list), ask the user for more context before restructuring — there is not enough material to build a pyramid.

If the input is already well-structured with clear conclusions and logical groupings, say so. Do not restructure for the sake of restructuring.

### Step 2: Build the SCQA introduction

Read `${CLAUDE_SKILL_DIR}/references/scqa.md`.

Write exactly four labeled components:

- **S** — one to two sentences the reader already knows
- **C** — one to two sentences about what changed or threatens
- **Q** — one sentence: the question the complication raises
- **A** — one sentence: the governing thought (= the answer)

Order per `--tone`. Default is `direct` (A first, then S, then C).

The introduction is two to three paragraphs. No more.

### Step 3: Build the key line (level 1)

Read `${CLAUDE_SKILL_DIR}/references/mece.md`.

Identify three to five points that together prove the governing thought. These points ARE the key line.

**Mandatory checks before proceeding:**

1. **Same-kind test.** Label all points with one plural noun (reasons, requirements, risks, steps, etc.). If no single noun fits, the grouping mixes types. Fix it.
2. **MECE test.** No point overlaps another. All points together cover the full argument. No gap a reader could object to.
3. **Ordering test.** Read `${CLAUDE_SKILL_DIR}/references/ordering.md`. Choose one ordering principle (time, structure, importance). Apply it. State which ordering you chose.

### Step 4: Build supporting levels (level 2+)

For each key-line point, ask: "what question does this raise?" Answer with another MECE group of three to five sub-points.

Apply the same three checks at every level: same-kind, MECE, ordering.

Stop when a statement needs no further proof — the reader would accept it as given.

### Step 5: Write the pyramid document

Read `${CLAUDE_SKILL_DIR}/references/writing-rules.md`.

The output has two parts: the **scaffold** (a planning artifact for the author) and the **document** (the final writing the audience reads).

---

#### Part 1: Scaffold (for the author only)

Output this block first inside a collapsible `<details>` tag. It is a planning artifact — not part of the final document. The author uses it to verify the pyramid's logic before reading the prose.

```html
<details>
<summary>Pyramid scaffold (planning artifact — not part of the document)</summary>

GOVERNING THOUGHT: [one sentence]

KEY LINE ([ordering principle] order; same-kind noun: "[plural noun]"):
1. [conclusion sentence]
   a. [sub-point]
   b. [sub-point]
   c. [sub-point]
2. [conclusion sentence]
   a. [sub-point]
   b. [sub-point]
   c. [sub-point]
3. [conclusion sentence]
   a. [sub-point]
   b. [sub-point]
   c. [sub-point]

MECE CHECK:
- Mutual exclusivity: [one sentence confirming no overlap, or naming the overlap]
- Collective exhaustiveness: [one sentence confirming full coverage, or naming the gap]

SCQA:
- S: [situation]
- C: [complication]
- Q: [question]
- A: [answer = governing thought]

TONE: [considered | direct | concerned]

</details>
```

Every key-line point and every sub-point must appear in the scaffold. If a sub-point has its own sub-points, indent further. The scaffold is the complete pyramid outline.

---

#### Part 2: Document (the actual output)

The document is what the audience reads. **It must read as natural, authoritative prose — not as a framework exercise.** No Minto jargon. No visible MECE labels. No "key line" references. No "governing thought" labels. No "SCQA" markers. The pyramid structure is invisible scaffolding — the reader feels clarity and logical flow, not a template.

The scaffold determines the document's heading hierarchy, but the document stands alone. A reader who never sees the scaffold must find the document clear, direct, and persuasive.

**How the scaffold maps to prose:**

| Scaffold element | Document element |
|-----------------|-----------------|
| Governing thought | `#` title (a conclusion sentence, not a label) |
| SCQA | Opening paragraphs (2-3 max). No S/C/Q/A labels. The components flow as natural narrative. |
| Key-line point | `##` heading (a conclusion sentence) |
| Sub-point | `###` heading (a conclusion sentence) |
| Sub-sub-point | `####` heading (a conclusion sentence) |

**Document template:**

```markdown
# [Title — the governing thought as a natural heading]

[Opening paragraphs: the SCQA components woven into 2-3 paragraphs
of natural prose. No labels. The situation anchors the reader, the
complication creates tension, and the answer resolves it. The reader
should not know this is an SCQA structure — they should just feel
oriented, then concerned, then directed.]

## [Conclusion sentence — what is true about this aspect]

[Prose that supports the heading. First sentence sharpens or
restates the claim. Remaining sentences are evidence: data, facts,
examples. Max 4 sentences per paragraph. Carry a word from the
previous section's end into this section's opening.]

### [Conclusion sentence — sub-point as natural prose]

[Evidence. Max 4 sentences.]

### [Conclusion sentence]

[Evidence. Max 4 sentences.]

## [Conclusion sentence — next key-line point]

[Prose. Same rules. The transition from the previous section
connects on substance, not with "Furthermore."]

### [Conclusion sentence]

[Evidence.]

...

## [Conclusion sentence — final key-line point]

...
```

**The invisible structure test:** Give the document to someone who has never heard of Minto. They should find it clear and well-argued. They should not think "this looks like it was generated from a template." If the structure is visible, the writing has failed.

## Format Rules

The detailed prose rules (sentence length, paragraph length, hedge words, transitions, tables) are in `${CLAUDE_SKILL_DIR}/references/writing-rules.md`, loaded in Step 5. The rules below are specific to the pyramid output and are not repeated in that file.

1. **The document reads as natural prose.** Framework terminology ("governing thought," "key line," "SCQA," "MECE," "complication") belongs in the scaffold only. The reader should feel clarity, not a methodology. If the structure is visible, the writing has failed.

2. **Every heading is a conclusion sentence.** A label ("Market Analysis") forces the reader to scan the body for the point. A conclusion sentence ("The addressable market is $2B and growing 15% annually") lets the reader extract the full argument from headings alone.

3. **Every group is MECE.** Overlapping points confuse the reader about where an idea lives. Gaps let a skeptic dismiss the argument as incomplete. If you cannot achieve MECE, name the gap in the scaffold so the author can decide.

4. **Every group is same-kind.** Mixing reasons with caveats or steps with outcomes breaks the reader's mental model of the group. Label all items with one plural noun — if no noun fits, the grouping is wrong.

5. **Every group is ordered.** Arbitrary sequences feel random. State the ordering principle in the scaffold (time, structure, or importance) and apply it consistently within each group.

6. **No content invention.** Restructure what the author wrote. Adding claims or data the author did not provide turns restructuring into ghostwriting.

7. **Scaffold and document must match structurally.** Every scaffold entry appears as a heading. Every heading traces back to a scaffold entry. The document's headings are natural conclusion sentences — not copies of scaffold shorthand.

## Constraints

- By default, output in the conversation. Only write to a file when `--save` is passed.
- If `--save` with no path: write to `./PYRAMID.md`.
- The governing thought must be one sentence. If you cannot compress it, the argument is unclear — say so and stop.
- Never exceed five items at any pyramid level. Three is optimal.
- Do not preserve the original's structure out of politeness. If it is wrong, replace it entirely.
- Do not qualify findings. Do not soften. Do not add "however" clauses that walk back the point.
- The scaffold is mandatory. Never skip it. Never merge it into the document.
- **No framework jargon in the document.** The scaffold is where you show your work. The document is where you show the result. The reader of the document should never know a framework was used.
