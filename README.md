# Basic HR Analytics with SQL — README

This README describes the notebook **`Basic data analysis using SQL queries .ipynb`**. It shows how to answer HR-style business questions with **SQL** and load the results into **pandas** for quick analysis.

## What this notebook does
- Connects to a **PostgreSQL** database (read-only) using **SQLAlchemy** and **psycopg2**, and pulls tables into pandas with `pd.read_sql`.
- Works with four main tables:
  - `hr_dataset` — employee-level records (e.g., performance score, manager, department, age, employment status, days employed, pay rate, marriage and gender indicators).
  - `production_staff` — production KPIs (e.g., weekly throughput, daily error rate, 90‑day complaints).
  - `recruiting_costs` — recruiting spend by employment source.
  - `salary_grid` — hourly min/mid/max by position.
- Frames business questions as **hypotheses** and answers them with SQL:
  - **H1**: Marriage relates to performance quality (share married by performance group).
  - **H2**: The firm is male-dominated (gender mix overall and by performance group).
  - **H4–H6**: Relations between performance and **age**, **tenure** (days employed), and **pay**.
  - Top **manager(s)** per performance group (ranked) and top **employee source** per department.
  - Departments with many **low-performing but active** employees (bounce candidates).
  - **Worst production staff** by error rate and complaints, controlling for throughput.
  - **Recruiting cost per hire** by employment source by joining `recruiting_costs` with observed hires.
  - Alignment of **position pay** (observed average) with the **salary grid** bands.
- Presents answers as inline pandas DataFrames.

## SQL features demonstrated
- Selection and filtering: `SELECT`, `WHERE`, `DISTINCT`, `IN`, `BETWEEN`, `LIKE/ILIKE`, `LIMIT`.
- Aggregation: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, with `GROUP BY` and `HAVING`.
- Type and numeric helpers: `CAST`, `ROUND`, `COALESCE`.
- Joins: `LEFT JOIN`, `FULL OUTER JOIN` to merge headcount with costs and grids.
- **CTEs**: `WITH` for composable step‑by‑step logic (including nested CTEs).
- **Window functions**: `RANK() OVER (PARTITION BY … ORDER BY …)` to find top managers or sources.
- Text aggregation: `STRING_AGG` to collect names per group.
- Case logic and bucketing: `CASE WHEN` patterns to group categories.
- Unioning segments: `UNION` where needed to stack results.

## Environment and prerequisites
- Python 3.9+
- Jupyter
- Packages: `pandas`, `SQLAlchemy`, `psycopg2` (notebook also imports `urllib.request` and `json`; `pymongo` is imported but not used).
  
```bash
pip install pandas sqlalchemy psycopg2-binary jupyter
```

## How to run
1. Open the notebook in Jupyter.
2. **Database access:** The notebook expects a PostgreSQL URL. Replace the example connection string in the first connection cell with your own, or set an environment variable and read it:
   ```python
   import os, sqlalchemy
   conn = os.environ.get("DB_URL")  # e.g., postgresql+psycopg2://user:pass@host:5432/db
   engine = sqlalchemy.create_engine(conn)
   connect = engine.connect()
   ```
3. Run cells from top to bottom. Each section poses a question, defines a SQL query string, and loads results via `pd.read_sql(query, connect)`.

## Notebook structure (high level)
- **Setup:** Imports, DB connection, quick sample pulls from the four tables.
- **Hypotheses 1–2:** Marriage vs performance; gender mix overall and by performance group.
- **Descriptives by performance:** Mean age, tenure, and pay rate by performance group.
- **Leaders and sources:** Rank top **manager(s)** per performance group; top **employee source** per department using window functions and `STRING_AGG`.
- **Risk flags:** Find departments with high shares of active low-performers (“bounce candidates”).
- **Production focus:** Rank worst production staff by error rate and complaints with throughput controls.
- **Cost and pay benchmarking:** Compute **recruiting cost per hire** by employment source using `FULL OUTER JOIN`; compare observed pay vs the **salary grid** via `LEFT JOIN` and CTEs.
- **Notes:** Results are shown as DataFrames and can be exported with `df.to_csv(...)` if needed.

## Reusing on your data
- Replace table names and column references to match your schema, or create views that mirror:
  - `hr_dataset(employee_id, department, manager_name, performance_score, employment_status, pay_rate, age, days_employed, genderid, marriedid, employee_source, position, …)`
  - `production_staff(employee_name, abutments_hour_wk1, abutments_hour_wk2, daily_error_rate, complaints_90d, …)`
  - `recruiting_costs("Employment Source", "Total", …)`
  - `salary_grid("Position", "Hourly Min", "Hourly Mid", "Hourly Max", …)`
- Keep the window-function ranking and `STRING_AGG` patterns; they adapt well to other KPIs.

## Troubleshooting
- **Auth/connection errors**: Confirm your `DB_URL` and that the host allows inbound connections. If SSL is required, include options in the URL.
- **Identifier quoting**: The source uses quoted identifiers like `"Performance Score"`. If your schema uses snake_case, remove the quotes and adjust names.
- **Performance**: For large tables, add predicates in `WHERE`, and consider materializing intermediate CTEs as temp tables.

---

**Author:** Aleksandr Bystrov  
**Purpose:** Demonstrate practical SQL for HR analytics in a Jupyter workflow.
