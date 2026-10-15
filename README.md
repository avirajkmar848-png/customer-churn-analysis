# Customer Churn Analysis

Exploratory analysis of a small customer-churn dataset stored in SQLite. The notebook
loads three related tables, cleans them, engineers churn-related features, and surfaces
a set of retention KPIs plus visualizations.

## Dataset

`customer_churn.db` (SQLite) with three tables joined on `customerid`:

| Table | Description | Key columns |
|-------|-------------|-------------|
| `db_customer` | Customer demographics | `customerid`, `name`, `country`, `state`, `gender`, `dob` |
| `db_subscription` | Subscription / billing | `subscription_start_date`, `subscription_type`, `plan_type`, `contract_type`, `cancellation_date`, `monthly_charges`, `cltv`, `churn_score` |
| `db_support` | Support contacts | `complaint_date`, `escalations`, `csat_score` |

The data is **synthetic** (fictional customer names) and contains **21 customers**.

## Pipeline

1. **Load** – discover tables via `sqlite_master` and read each into a DataFrame.
2. **Clean**
   - Rename `name` → `customer_name`; drop `interests` / `pincode`.
   - Parse `dob` and all date columns to `datetime`.
   - Standardize `gender` (`Men` → `Male`, `Women` → `Female`).
   - Fill missing `country` from a `state → country` mapping.
3. **Feature engineering**
   - `churn_flag` = 1 when `cancellation_date` is present.
   - `complaint_count` per customer (before de-duplicating support records).
   - De-duplicate support to the most recent complaint per customer.
   - `tenure_days`, `customer_age`, `churn_risk` (low / med / high from `churn_score`).
   - `escalation_flag` (binary) kept alongside the raw `escalations` value.
4. **Analysis** – churn / retention rate, churn by plan, state and subscription type,
   ARPU, average tenure and age, revenue at risk, escalation rate and correlation.
5. **Visualization** – monthly churn trend, churn by plan type and state, correlation
   heatmap (ordinal encoding) and a faceted `catplot`.

All date-dependent metrics use a fixed `REPORT_DATE = 2025-04-01` so results are
reproducible.

## Key results

| Metric | Value |
|--------|-------|
| Churn rate | 28.57 % |
| Retention rate | 71.43 % |
| ARPU (monthly charges) | 18.85 |
| Average tenure | 1159 days |
| Average age | 36 years |
| Revenue at risk (monthly) | 73.94 |
| Escalation rate | 19.05 % |
| Avg complaints per user | 0.43 |
| Escalation ↔ churn correlation | 0.77 |

Churn by plan type:

| Plan | Churn rate |
|------|-----------|
| Basic | 60.00 % |
| Premium | 14.29 % |
| Standard | 22.22 % |

Churn by subscription type:

| Type | Churn rate |
|------|-----------|
| Organic | 0.00 % |
| Paid | 16.67 % |
| Refferal | 83.33 % |

## Caveat

> **n = 21.** Every rate and correlation here is illustrative only. Per-state churn of
> 0 % or 100 % reflects 1–3 customers, and the 0.77 escalation↔churn correlation has a
> very wide confidence interval at this sample size. Treat this as a workflow
> demonstration, not a statistically reliable finding.

## Getting started

```bash
pip install -r requirements.txt
jupyter notebook churn_analysis.ipynb
```

The database is read with a relative path, so run the notebook from the repository root.

## Project structure

```
customer-churn-analysis/
├── churn_analysis.ipynb   # analysis notebook
├── customer_churn.db      # SQLite source data (synthetic)
├── requirements.txt
├── .gitignore
└── README.md
```
