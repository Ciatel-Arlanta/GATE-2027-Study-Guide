# Brief: README weightage columns and fact-check review (2026-10)

Repo: `C:\Files\GATE\Gate-plan` (GATE 2027 CS + DA study guide). Read `tools/AUTHORING-GUIDE.md` first, especially "PYQ weightage marking".

## Task A: section README weightage
For each assigned section `NN-name/README.md`:
1. In the reading-order table, add a `PYQ weightage` column. Fill it from each chapter's `> **PYQ weightage:**` header line (level + "N Qs since 2015"), e.g. `High · 22`.
2. Directly under the title/blockquote, add one line: `**PYQ weightage:** ~X Qs per CSE paper (2021–26), trend T; DA: D Qs in 2024–26.` Take X, T, D from the section's row in the root `WEIGHTAGE.md`.
3. Add a short "Study priority" list: the section's chapters ordered High → Low, one line each saying what to master first.
Do not change anything else in the README. Edit README files only.

## Task B: fact-check review of expanded chapters
For each assigned chapter:
1. Recompute every worked example and every Practice answer (use Python where it helps). Check formulas, complexities, definitions and GATE conventions.
2. Fix definite errors in place with minimal edits. Do not restyle or shorten content.
3. Check that relative links resolve to files that exist.
4. Check the file has no control characters and that `<details>` blocks are balanced.

## Boundaries
- Edit only your assigned files. No git state-changing commands (no commit, checkout, reset, stash).
- Final reply: per file, a list of the errors you fixed (old → new, with one-line reason) and anything you could not resolve.
