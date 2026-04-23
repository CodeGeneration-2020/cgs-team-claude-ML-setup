# Claude Code Notebook Limitations

Observed limitations and failure patterns from testing notebook editing via Claude Code.
Covers notebook reading, markdown structuring, explanation cells, and iterative cleanup.
Does not cover general Claude Code limitations or ML platform concerns.

---

## 1. Known Limitations Observed in Practice

**No live kernel connection.**
Claude reads the saved `.ipynb` file on disk. If you execute a cell in Jupyter but do not save, Claude will not see the result. There is no way to query the live kernel state.

**Execution re-runs the full notebook.**
Running via `nbconvert --execute` starts from cell 0. There is no single-cell execution path through Claude Code. This is slow for notebooks with expensive steps. `nbconvert` must also be installed separately (`pip install nbconvert`) — it is not available in all environments by default.

**Interpretation cells are written from code, not from actual output.**
When outputs are not saved in the notebook, Claude writes interpretation based on what the code should produce. Observations and values will be directional. Notebooks with non-obvious or data-dependent outputs require a run-first approach before annotating.

**Issue detection is static only.**
Claude can identify suspicious patterns by reading — hardcoded paths, no-op entries, broken references — but cannot confirm whether they cause runtime failures without executing the notebook.

**Large notebooks with embedded outputs are harder to navigate.**
Notebooks with many cells or large embedded outputs (images, wide dataframes) take longer to read and increase the risk of Claude losing track of structure. Splitting large notebooks by stage reduces this.

**Cell indices are not reliable anchors during editing.**
Indices shift as cells are inserted or deleted. This is an editing constraint, not a blocker — section names and nearby content are stable alternatives and should be used instead.

---

## 2. Common Failure Patterns

**Over-annotation on the first pass.**
Adding section headers and interpretation cells in the same step consistently produced notebooks that felt over-annotated. A second cleanup pass was needed every time. The root cause is that Claude defaults to comprehensive coverage when scope is not constrained.

**Wrong cell insertion order.**
When more than one cell needs to be inserted between the same two existing cells, the order of operations must be planned explicitly. Inserting in a single step can produce the wrong sequence. The two-phase pattern (insert the header first, then insert the interpretation after the same anchor) is reliable but not obvious.

**Hardcoded and environment-specific paths.**
Static reading of notebooks surfaces absolute paths (e.g., `/Users/name/project/data/file.csv`) that would break in any other environment. These are easy to miss in a manual review and were flagged reliably by Claude as a class of issue — making path review a practical item to include in any pre-commit check.

**Unverifiable issues.**
Claude correctly identified a no-op `VENDOR_MAP` entry and a `credit_debit_indicator` inconsistency in `build_sandbox_dataset.ipynb` by reading alone. Whether these caused actual runtime problems could not be confirmed without executing the notebook. Static findings should be treated as candidates, not confirmed bugs.

---

## 3. Practical Workarounds

| Limitation | Workaround |
|---|---|
| No live kernel connection | Save after every meaningful change so Claude picks up the latest state on the next turn |
| Full notebook re-execution | Split expensive steps into separate notebooks, or accept that execution-based debugging is slow |
| Interpretation from code only | Run the notebook and save outputs before asking Claude to write interpretation cells |
| Static issue detection only | Use Claude findings as a review checklist; confirm runtime behavior by executing |
| Large notebooks | Split by stage; for structure work, read and edit one section at a time |
| Cell index drift | Always reference cells by section name or surrounding content, never by index |

---

## 4. Prompting Recommendations

**Ask for a plan before editing more than 2–3 cells** — mitigates wrong insertion order and over-annotation. Reviewing a list of proposed changes is faster than cleaning up after a heavy-handed pass.

**Separate structure passes from interpretation passes** — mitigates over-annotation. One pass for headers, review, then a separate decision on whether interpretation cells are needed at all.

**Constrain scope explicitly in every edit prompt** — mitigates Claude's default toward comprehensive coverage. "Headers only, no body text" and "one sentence max" are constraints that need to be stated; Claude will not infer them from context.

**Include path review in pre-commit checks** — mitigates hardcoded paths slipping through. Claude reliably flags absolute and environment-specific paths when asked; this is not checked unless explicitly requested.

---

## 5. What Still Needs Validation

These are planned test cases from `docs/notebook-testing-findings.md` that have not been run yet:

- **Add a section as a unit** — insert a markdown header and a new code cell together in a single step
- **Large notebook reliability** — verify read accuracy and edit reliability at 50–100 cells
- **Saved plot outputs** — check whether Claude can read and interpret saved image outputs from a prior run
