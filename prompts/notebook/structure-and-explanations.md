# Notebook Structure and Explanations

Internal prompt templates for improving notebook structure and documentation using Claude Code.
Based on findings from testing against `build_sandbox_dataset.ipynb` and `notebooks/examples/notebook-analysis-test.ipynb`.

---

## Prompt Templates

### 1. Analyze Notebook Structure

Use this before making any edits. Read-only — no changes.

```
Read [notebook path] and report:
- What the notebook does and what data it operates on
- How it is currently sectioned (or where sections are missing)
- Any obvious issues: missing imports, broken references, dead cells, unclear ordering

Do not make any edits. List your findings only.
```

---

### 2. Add Section Headers

Adds structural markdown headers only. No interpretation, no body text.

```
Add section headers to [notebook path] to reflect its logical structure.

Rules:
- Add markdown cells with ## headers only — no explanatory text beneath them
- Do not touch any code cells
- Do not add more than one markdown cell per section
- If this requires more than 3 insertions, list what you would add and where before making any edits
```

---

### 3. Add Short Explanation Cells

Adds brief interpretation cells, but only where output is non-obvious.

```
Add a short explanation cell after cells that produce important or non-obvious outputs in [notebook path].

Rules:
- Only add cells where the output requires interpretation — not after every step
- Each explanation must be a single sentence (two at most)
- Do not explain what the code does — explain what the result means
- Do not touch any code cells
- List what you plan to add before making any edits
```

---

### 4. Reduce Over-Explaining / Clean Up Markdown

Trims markdown without touching code. Default to deletion over rewriting.

```
Review the markdown cells in [notebook path] and reduce over-annotation.

Rules:
- Delete markdown cells that restate what the code already makes obvious
- Shorten cells that are longer than 2–3 lines unless the length is clearly justified
- Prefer deletion over rewriting — if a cell can be cut entirely, cut it
- Do not touch any code cells
- List every cell you would change and what you would do to it before making any edits
```

---

### 5. Pre-Commit Review

Strict read-only checklist. No fixes applied.

```
Review [notebook path] before I commit it and report any issues across these areas:

- Outputs: are cell outputs cleared, or are there large/sensitive outputs that should not be committed?
- Imports: any duplicated or unused imports?
- Dead cells: any empty cells or cells that are commented out entirely?
- Markdown coverage: are there long stretches of code with no section header?
- Markdown overload: are there sections where the markdown is longer than the code it describes?

Do not make any changes. Return a list of issues only, grouped by area.
```

---

### 6. Fix Section Ordering / Renumber Headers

Use after structural edits have shifted section numbers out of order.

```
The section headers in [notebook path] are out of order or incorrectly numbered after recent edits.

Rules:
- Renumber or reorder ## headers to reflect the current logical flow
- Reference sections by their name and content, not by cell index
- Do not change header text other than numbering
- Do not touch any code cells or non-header markdown cells
- List the before/after header sequence before applying any changes
```

---

## Prompting Tips

These come directly from the testing sessions — not general advice.

**Separate structure passes from interpretation passes.** Adding headers and interpretation cells in the same step produces over-annotated notebooks. Do headers first, review, then decide if interpretation cells are needed at all.

**Always ask for a plan before edits touch more than 2–3 cells.** The "list what you'd change before editing" constraint caught scope issues early and was cheaper than cleaning up after a heavy-handed pass.

**Reference sections by name, not cell index.** Cell indices shift as cells are added. Section names and nearby content are stable anchors.

**Run the notebook before writing interpretation cells.** Explanation cells written from code alone (not actual output) will be directional at best. If outputs are not saved, run first, save, then annotate.

**Use two separate steps when inserting interleaved cells.** If you need a section header followed immediately by an interpretation cell, insert the header first, save, then insert the interpretation cell after the same anchor — it will land in the right order.

**Incremental passes beat one large edit.** The cleaner result in every test came from a targeted second pass, not from getting the first pass right.
