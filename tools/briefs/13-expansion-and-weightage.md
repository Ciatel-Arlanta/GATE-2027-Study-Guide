# Brief: expand thin chapters and mark PYQ weightage (2026-10)

Repo: `C:\Files\GATE\Gate-plan` (GATE 2027 CS + DA study guide; the reader writes both papers).

## Read first
1. `tools/AUTHORING-GUIDE.md` — chapter anatomy, style rules, and the "PYQ weightage marking" section. Follow it exactly.
2. `tools/pyq-weightage-data.md` — real PYQ question counts per subject/subtopic. Use only these numbers for weightage.
3. One model chapter at the target depth: `07-data-structures/heaps.md` (also good: `10-computer-organization/pipelining.md`).
4. `STRUCTURE.md` for valid relative link paths.

## PYQ source text
(Temporary `pdftotext -layout` extracts of `C:\Files\GATE\PYQs Qbs\`, used for the October 2026 run and deleted afterwards.)
Plain-text PYQ books (questions with full text, year headers like `1.2.3 Subtopic: GATE CSE 2021 Set 1 | Question: 40`):
`C:\Files\GATE\pyq-text\GATE_PYQ_PART_1.txt` (Discrete maths, engg maths, aptitude), `..._PART_2.txt` (Algorithms, Compiler, DS, C, TOC), `..._PART_3.txt` (COA, CN, DBMS, Digital logic, OS), `GATE_DA_PYQ.txt` (DA paper + shared subjects). Files are large: use grep/ripgrep on subtopic names, then read only the matching ranges. Use them to see what GATE actually asks, and model worked examples and practice questions on those patterns. Do not copy questions verbatim; do not cite a specific year unless the header shows it.

## Expansion target (for chapters you are told to expand)
- Rewrite the chapter in place to the full anatomy: Quick glance, numbered concept sections (intuition → definition → worked example with every step), "Formulas and facts to memorise" table, GATE traps, Connections, Practice.
- Target 250–450 lines. Cover every plan topic in the header and every frequently asked PYQ subtopic that belongs to this chapter, giving the most space to High-weightage subtopics.
- At least one fully worked GATE-style numerical per numbered section where the topic is numerical (Gantt charts, page-fault tables, subnet calculations, DP tables, MST traces, etc.).
- Practice: 6–8 questions, MCQ/MSQ/NAT mix, easy → hard, full solutions in `<details>` blocks.
- Keep existing correct content and links; fix anything wrong.
- Verify every numeric answer by recomputing it (you may run Python to check).

## Weightage marking (for every chapter you touch)
Apply the three markings from the authoring guide (header line, `[PYQ: X]` tags on numbered headings, `**Exam focus:**` Quick glance bullet), using counts from `tools/pyq-weightage-data.md`.

## Boundaries
- Edit only the files assigned to you. Do not edit README/CHEATSHEET/CHECKPOINT files, root files, or other chapters unless explicitly assigned.
- Do not run git commands that change state (no commit, checkout, reset, stash).
- At the end, reply with: files changed, line counts before/after, the weightage line you added to each chapter, and anything you were unsure about.

## Marking-only assignments
For chapters you are told to MARK (not expand): keep the content, add the three weightage markings, and check coverage against the PYQ data. If a subtopic that is High weightage for this chapter is missing or covered in under ~10 lines, add a concise numbered section for it (intuition, rule, one fully worked GATE-style example) and one practice question. Do not shrink or rewrite existing correct material.
