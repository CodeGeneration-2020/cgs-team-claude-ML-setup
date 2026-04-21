# Notebook Workflow Guide

Practical guide for ML/AI work in Jupyter notebooks using Claude Code.

---

## 1. When to Use Notebook Workflow

**Good fit:**
- Exploratory data analysis — inspecting data shape, distributions, and anomalies
- Feature engineering iterations where intermediate outputs matter
- One-off model comparisons or metric evaluations
- Prototyping a pipeline before converting it to a script

**Not a good fit:**
- Production training runs — use a script instead
- Anything that needs to run unattended or be scheduled
- Work that requires a persistent kernel state across many Claude turns

---

## 2. Recommended Workflow Step by Step

### Step 1 — Orient Claude to the notebook

Read the notebook first to surface structure, variable names, and any obvious issues before making changes.

```
Read my notebook at notebooks/eda.ipynb and tell me:
- what dataset it operates on
- what the main analysis steps are
- any obvious issues (missing imports, broken references, unclear sections)
```

### Step 2 — State a concrete goal for the session

Be specific. Vague goals produce vague edits.

```
I want to add a section after the "Load Data" cell that:
1. Checks for nulls per column and prints a summary
2. Plots a histogram of the target variable
3. Flags any columns with >20% missing values
```

### Step 3 — Edit cells explicitly

Describe what to change by section name or surrounding content rather than cell index.

```
Add a new code cell after the cell that defines `df`. It should compute
value_counts() for every categorical column and display the top 5 for each.
```

### Step 4 — Run and inspect outputs

If `jupyter` is available, Claude Code can execute the notebook via Bash and report errors.

```
Run the notebook and tell me if any cells errored. If they did, show me
the traceback and fix the most likely cause.
```

### Step 5 — Iterate on a specific cell

Target individual cells for refinement once outputs are visible.

```
The histogram in cell 7 is unreadable because the bins are too wide.
Change it to use 50 bins and add axis labels.
```

### Step 6 — Clean up before committing

Before finishing, ask Claude to review the notebook for hygiene.

```
Review the notebook for:
- cells that print redundant intermediate outputs
- imports that are duplicated or unused
- markdown cells that are missing or out of date
List what you'd change and ask before editing.
```

---

## 3. Example Prompts

**Load and inspect a dataset**
```
Read the CSV at data/raw/train.csv into a dataframe. Show shape, dtypes,
null counts, and the first 5 rows. Add this as a new section called
"Initial Inspection" at the top of the notebook.
```

**Diagnose a failing cell**
```
Cell 12 raises a KeyError on column "user_id". Read the notebook and find
where that column is dropped or renamed upstream, then fix cell 12.
```

**Add evaluation metrics**
```
After the model is trained in cell 20, add cells that compute and display:
- classification report (sklearn)
- confusion matrix as a heatmap (seaborn)
- ROC-AUC score
Use the existing `y_test` and `y_pred` variables.
```

**Refactor a section into a function**
```
Cells 8–14 do feature engineering. Extract that logic into a single function
called `engineer_features(df)` and add it to a new cell at the top of the
"Feature Engineering" section. Replace cells 8–14 with a single call to it.
```

**Explain what the notebook does**
```
Summarize this notebook in plain language: what problem it solves, what data
it uses, what the output is, and whether any steps look incomplete.
```

---

## 4. Best Practices

**Name sections with markdown cells.** Using `## Feature Engineering` as a section header makes it easier to give precise edit instructions rather than referring to cell numbers.

**Keep cells short and single-purpose.** Long cells that do many things are harder to edit safely without breaking adjacent logic.

**Clear outputs before committing.** Unless the outputs are the artifact (e.g., a report), committed outputs inflate diffs and cause merge conflicts.

**Tell Claude the variable names that matter.** If your dataframe is `df_train` not `df`, say so upfront — ambiguity leads to wrong edits.

---

## 5. Current Limitations

**No live kernel connection.** Claude reads the saved `.ipynb` file, which only contains outputs from the last run. If you execute a cell in Jupyter but don't save, Claude won't see the result.

**Execution re-runs the full notebook.** Running via `nbconvert --execute` starts from scratch. There is no way to run a single cell in isolation through Claude Code, which can be slow for notebooks with expensive steps.

**Large notebooks can be harder to work with.** Notebooks with many cells or large embedded outputs (images, wide dataframes) take longer to read and may cause Claude to lose track of structure. Splitting by stage can help.

**Claude re-reads from disk each turn.** There is no persistent in-memory state between turns. Save after each meaningful change so Claude picks it up on the next turn.

**Plot outputs require a re-run.** Claude cannot see a rendered image without executing the notebook and reading the saved output.

---

## 6. Suggested Next Steps for This Repo

- `notebooks/examples/eda-template.ipynb` — a minimal starter notebook demonstrating the section structure recommended in this guide
- `prompts/notebook/` — reusable prompt templates for common tasks (EDA, feature engineering, model evaluation)
- `checklists/ml-review.md` — a pre-commit checklist for notebook quality (outputs cleared, sections named, no dead cells)
- `docs/limitations.md` — a living document tracking Claude Code notebook gaps as they are discovered or resolved
