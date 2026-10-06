# Authoring guide

How every chapter in this guide is built, so that any page can be read cold and revised at a glance. Read this before adding or editing a chapter.

## Goals

1. **Self-sufficient.** A reader who has never studied the topic must understand it from this page alone. Define every term before using it. Build intuition first, then the formal statement, then a worked example.
2. **Glanceable.** A reader revising the night before must recover the topic in 2–3 minutes from the Quick glance box, tables, and bolded key facts.
3. **Connected.** Every chapter states where its ideas come from and where they reappear, with relative links.
4. **Exam-shaped.** Every concept is tied to how GATE actually asks it (MCQ / MSQ / NAT), with the traps that cost marks.

## Chapter anatomy (use these headings, in this order)

```markdown
# <Chapter title>

> **Paper:** CS | DA | CS+DA · **Priority:** P0 | P1 | P2 · **Plan topics:** <exact topic names from the Master Study Plan covered here>
> **Prerequisites:** [link](../xx/file.md) · **Leads to:** [link](../yy/file.md)

## Quick glance
<5–10 bullets that compress the entire chapter: definitions, the key formula, the key algorithm/idea, the #1 trap. Someone reading only this box should be able to answer an easy GATE question.>

## <Concept sections, numbered: "1. ...", "2. ...">
For each concept: intuition (plain language, an analogy if it helps) → precise definition/statement → **worked example** with every step shown → summary table where useful.

## Formulas and facts to memorise
<A table: Item | Formula/fact | When to use>

## GATE traps
<Bulleted list of the specific mistakes that cost marks, each with the correct reasoning in one line.>

## Connections
<Bullets: "[Linked chapter](relative/path.md) — one line explaining HOW the idea connects", both backward (built on) and forward (used in). Cross subject boundaries generously.>

## Practice
<5–8 GATE-style questions mixing MCQ, MSQ and NAT, ordered easy → hard. Each answer and full solution goes inside a folded block:>

<details><summary>Answer</summary>

**Answer:** ...  
**Solution:** step-by-step...

</details>
```

## Section files

- **`README.md`** — title; prerequisites (links); which paper(s) and approximate GATE weightage; a short "Why this subject matters / mental model" paragraph; a reading-order table `Chapter | Priority | Plan topics covered`; a small mermaid `flowchart` of how the chapters inside the section depend on each other; a "Connections to other subjects" list; links to `CHEATSHEET.md` and `CHECKPOINT.md`; footer `[Roadmap](../ROADMAP.md) · [Connections](../CONNECTIONS.md) · [Progress](../PROGRESS.md)`.
- **`CHEATSHEET.md`** — one dense page for final revision: every formula, table, algorithm complexity, definition and "remember this" rule from the section. Tables and bullets only; no long prose.
- **`CHECKPOINT.md`** — 10–15 mixed GATE-style questions spanning the whole section (several combining two or more chapters), easy → hard, each with a folded solution. End with a scoring line: e.g. "≥ 80% → move on; < 60% → re-read the chapters listed against the questions you missed."

## Style rules

- Markdown that renders on GitHub, VS Code and Obsidian. Use `$...$` / `$$...$$` LaTeX for formulas that need it, plain text for trivial ones. Use mermaid (` ```mermaid `) for flowcharts, state machines, trees and graphs where a picture helps; use ` ```text ` ASCII diagrams for memory layouts, tables of trace values, K-maps, pipelines and the like.
- Worked examples use concrete numbers and show every intermediate step, the way a strong topper writes on rough paper. Prefer examples in the style of real GATE PYQs (do not claim a specific year unless certain).
- Code: C examples in ` ```c `, Python in ` ```python `, SQL in ` ```sql `. Every code-tracing example shows the output and why.
- Bold the single most important fact in each subsection. Keep paragraphs short (≤ 4 lines).
- Correctness above all: these are exam notes. Double-check every formula, every numeric answer and every claim. If a convention varies between textbooks (e.g. tree height from 0 or 1, B-tree order definitions, Bayes vs. frequentist CI wording), state the convention you use and how GATE usually phrases it.
- Relative links only, using exactly the file paths in [STRUCTURE.md](../STRUCTURE.md). Link to files, not to heading anchors.
- No emojis. No filler ("In this chapter we will..."). No motivational fluff.
