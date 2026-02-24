# Dataset of SQL solutions with derived code-complexity metrics (Data in Brief

**DOI:** 10.5281/zenodo.18755578

This repository provides the following files:

- **`SQL_codes_dataset.csv`** — the original dataset with SQL code solutions.
- **`metrics_dataset.csv`** — the same dataset enriched with automatically computed size/complexity metrics.
- **`SQL_code_metrics.ipynb`** — a Jupyter notebook that computes the metrics and exports `metrics_dataset.csv`.

### Quick statistics

- **Rows (SQL statements):** 669  
- **Sources:**  
  - 189 student solutions (`Model = "student"`, anonymized student IDs 1–30)  
  - 480 LLM-generated solutions (`Model = {ChatGPT5.2, CortexAI, Claude_4.5_Sonnet}`, 4 variants per task)
- **Tasks indexing:** 
  - `TG` = task group (A–D)
  - `X` = task number (1–10)

## File format notes 

Both CSV files are:

- **semicolon-separated** (`sep=';'`)
- **UTF-8 encoded**
- contain SQL code with **line breaks inside the `AXC` field**
- `metrics_dataset.csv` uses **comma as decimal separator** for some numeric values (`decimal=','`)

### Loading with Python (pandas)

```python
import pandas as pd

sql_df = pd.read_csv("SQL_codes_dataset.csv", sep=";", encoding="utf-8")
metrics_df = pd.read_csv("metrics_dataset.csv", sep=";", decimal=",", encoding="utf-8")
```

## Dataset schemas

### 1) `SQL_codes_dataset.csv` (original)

| Column   | Description |
|----------|-------------|
| `ID`     | Unique row identifier |
| `TG`     | Task group (A–D) |
| `Student`| Student ID (1–30) for student solutions, `-1` for LLM-generated solutions |
| `Model`  | Source of the solution (`student`, `ChatGPT5.2`, `CortexAI`, `Claude_4.5_Sonnet`) |
| `Variant`| LLM generation variant (0–3) for LLMs, `-1` for students |
| `X`      | Task number (1–10) within the task group |
| `AXC`    | SQL code (Snowflake dialect used for parsing in the notebook) |

### 2) `metrics_dataset.csv` (dataset + metrics)

Contains all columns from `SQL_codes_dataset.csv` plus the following computed metrics:

#### Size metrics
- `lines_of_code` — number of lines (split by newline)
- `character_length` — total characters
- `token_count` — token count from notebook tokenizer

#### Halstead-style metrics (computed from tokenized SQL)
- `n1_distinct_operators`, `n2_distinct_operands`
- `N1_total_operators`, `N2_total_operands`
- `vocabulary`, `program_length`
- `volume`, `difficulty`, `effort`, `time_to_program`  
  (`time_to_program` is computed as `effort / 18`)

#### SQL construct counts
- `subquery_count`
- `window_function_count`
- `aggregate_function_count`
- `cte_count`

#### Nesting and cognitive-style metrics
- `max_nesting_depth` — parenthesis nesting depth
- `cognitive_complexity` — heuristic score based on subqueries, logical operators, window functions, and nesting depth

#### Additional composite metrics
- `sqlshare_complexity` — weighted heuristic using table/column/operator/expression counts and character length
- `weighted_complexity` — weighted score based on select/subquery/window/aggregate/CTE presence

#### AST-based metrics (via `sqlglot`, Snowflake dialect)
- `ast_node_count`
- `ast_height`
- `ast_branching_factor`
- `leaf_node_ratio`

> Note: If an SQL statement cannot be parsed by `sqlglot`, AST features are set to zero for that row.

## Reproducing the metrics

### Requirements

- Python 3.10+ recommended
- Packages:
  - `pandas`
  - `sqlglot`
  - (optional) `jupyter`

### Install dependencies

```bash
python -m pip install pandas sqlglot jupyter
```

Open and run:

- `SQL_code_metrics.ipynb`

The notebook reads `SQL_codes_dataset.csv` and writes `metrics_dataset.csv`.
