# Notebook Testing Findings

---

## 1. Goal

Validate the notebook workflow described in `docs/notebook-workflow.md` against two real notebooks:
a real-world data preparation notebook (`build_sandbox_dataset.ipynb`) and a small synthetic test notebook (`notebook-analysis-test.ipynb`).

---

## 2. Test Notebooks

**`build_sandbox_dataset.ipynb`** — 35-cell production notebook. Loads a NetSuite vendor payment CSV, anonymises it, and exports a `sandbox_dataset.json` for app testing. Used for read and issue-identification testing only; no edits except section renumbering.

**`notebooks/examples/notebook-analysis-test.ipynb`** — 11-cell synthetic notebook. Simple EDA over a small employee dataframe. Used for full edit cycle: add markdown cells, review, clean up.

---

## 3. What We Tested

- Reading a notebook and summarising its structure and purpose
- Identifying issues without running any code
- Editing markdown cells by cell ID (replace, insert, delete)
- Running multiple edits in parallel
- Two-phase insertion strategy for ordering multiple cells between the same two existing cells
- Iterative cleanup: adding markdown then reducing it in a second pass

---

## 4. What Worked Well

**Reading and summarising** — Claude accurately described section structure, identified the main purpose, and flagged real issues (no-op `VENDOR_MAP` entry, broken section numbering, hardcoded absolute paths) without executing the notebook.

**Markdown editing by cell ID** — precise and reliable. Edits landed exactly where intended, with no impact on adjacent code cells.

**Parallel edits** — batching independent `NotebookEdit` calls in a single step worked consistently. The 8-cell section renumber on `build_sandbox_dataset.ipynb` and the 12-cell addition on the test notebook both completed without error.

**Two-phase insertion** — inserting section headers first, then interpretation cells after the same anchors, correctly interleaved the cells in the right order.

**Plan before edit** — showing the proposed changes before applying them caught scope issues early (the first markdown pass was too heavy).

---

## 5. What Did Not Work Well

**First markdown pass was over-generated.** Adding section headers, purpose lines, and interpretation cells in one step produced a notebook that felt over-annotated. A second cleanup pass was needed. The better approach is to add structure incrementally rather than all at once.

**Two-phase insertion adds friction.** When more than one cell needs to be inserted between the same two existing cells, the order of operations must be planned carefully. This is not obvious and easy to get wrong.

**No way to validate code without running the notebook.** Issue identification was limited to static reading — we could not confirm whether the no-op `VENDOR_MAP` entry or the `credit_debit_indicator` inconsistency actually caused runtime problems without executing the notebook.

**Interpretation cells written before seeing output.** All interpretation cells were written based on the code, not the actual output. Values and observations are directional. Any notebook with non-obvious outputs would require a run-first approach before writing interpretations.

---

## 6. Prompting Guidance

**Be specific about what to edit.** "Add a section header before the null check" is better than "add section headers". Broad instructions produce over-annotated notebooks.

**Separate structure from interpretation.** Add section headers in one step, review, then add interpretation cells. Mixing both in one pass is hard to calibrate.

**Show the plan first.** For any edit touching more than 2–3 cells, ask Claude to list what it would change before applying. Easier to cut before writing than after.

**Reference by section name, not cell index.** Cell indices shift as cells are added. Section names and content anchors are stable.

**Use the two-phase pattern for interleaved insertions.** If you need [output → interpretation → next section header], insert the section header first, then insert the interpretation after the same anchor — it will land between the output and the header.

---

## 7. Next Test Cases

- **Run the notebook** via `nbconvert --execute` and have Claude report errors and fix the failing cell (`unknown_column`)
- **Edit a code cell** — change a function or fix a bug, verify surrounding cells are unaffected
- **Add a new section** — insert both a markdown header and a code cell together as a unit
- **Test on a larger notebook** — verify read accuracy and edit reliability at 50–100 cells
- **Test with cell outputs present** — check whether Claude can read and interpret saved plot outputs or dataframe previews
