# 🦄 Unicorn Companies — Industry Trend Analysis

> A SQL + Python data analysis project exploring high-growth unicorn companies to identify the best-performing industries for investment insights.

---

## 📌 Project Overview

This project was completed as part of a DataCamp data analysis challenge. The goal was to support an investment firm by analyzing trends in high-growth companies — specifically identifying which industries are producing the most unicorns and the highest valuations.

---

## 🗃️ Database Schema

The project uses a PostgreSQL database called `unicorns` with the following tables:

### `dates`
| Column | Description |
|---|---|
| `company_id` | Unique ID for the company |
| `date_joined` | Date the company became a unicorn |
| `year_founded` | Year the company was founded |

### `funding`
| Column | Description |
|---|---|
| `company_id` | Unique ID for the company |
| `valuation` | Company value in USD |
| `funding` | Amount of funding raised in USD |
| `select_investors` | Key investors in the company |

### `industries`
| Column | Description |
|---|---|
| `company_id` | Unique ID for the company |
| `industry` | Industry the company operates in |

### `companies`
| Column | Description |
|---|---|
| `company_id` | Unique ID for the company |
| `company` | Company name |
| `city` | Headquarters city |
| `country` | Headquarters country |
| `continent` | Headquarters continent |

---

## 🎯 Objective

1. Identify the **top 3 best-performing industries** by number of new unicorns created in **2019, 2020, and 2021 combined**
2. For those industries, return:
   - `industry` — the industry name
   - `year` — the year they became a unicorn
   - `num_unicorns` — number of unicorns per industry per year
   - `average_valuation_billions` — average valuation in billions (rounded to 2 decimal places)
3. Sort results by **year** and **num_unicorns**, both descending

---

## 💻 Solution

### Cell 1 — SQL Query (stored as DataFrame `df`)

```python
%%sql
df <<
WITH top_industries AS (
    SELECT 
        i.industry,
        COUNT(*) AS num_unicorns
    FROM industries i
    JOIN dates d ON i.company_id = d.company_id
    WHERE EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry
    ORDER BY num_unicorns DESC
    LIMIT 3
),

yearly_stats AS (
    SELECT
        i.industry,
        EXTRACT(YEAR FROM d.date_joined) AS year,
        COUNT(*) AS num_unicorns,
        ROUND(AVG(f.valuation / 1000000000.0), 2) AS average_valuation_billions
    FROM industries i
    JOIN dates d ON i.company_id = d.company_id
    JOIN funding f ON i.company_id = f.company_id
    WHERE i.industry IN (SELECT industry FROM top_industries)
      AND EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry, year
)

SELECT
    industry,
    year,
    num_unicorns,
    average_valuation_billions
FROM yearly_stats
ORDER BY year DESC, num_unicorns DESC;
```

### Alternative — Pure Python + SQLAlchemy

```python
import pandas as pd

df = pd.read_sql("""
WITH top_industries AS (
    SELECT 
        i.industry,
        COUNT(*) AS num_unicorns
    FROM industries i
    JOIN dates d ON i.company_id = d.company_id
    WHERE EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry
    ORDER BY num_unicorns DESC
    LIMIT 3
),
yearly_stats AS (
    SELECT
        i.industry,
        EXTRACT(YEAR FROM d.date_joined) AS year,
        COUNT(*) AS num_unicorns,
        ROUND(AVG(f.valuation / 1000000000.0), 2) AS average_valuation_billions
    FROM industries i
    JOIN dates d ON i.company_id = d.company_id
    JOIN funding f ON i.company_id = f.company_id
    WHERE i.industry IN (SELECT industry FROM top_industries)
      AND EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry, year
)
SELECT industry, year, num_unicorns, average_valuation_billions
FROM yearly_stats
ORDER BY year DESC, num_unicorns DESC;
""", con)

df
```

---

## 📊 Expected Output Format

| industry | year | num_unicorns | average_valuation_billions |
|---|---|---|---|
| industry1 | 2021 | --- | --- |
| industry2 | 2020 | --- | --- |
| industry3 | 2019 | --- | --- |
| ... | ... | ... | ... |

---

## 🛠️ Tools & Technologies

- **PostgreSQL** — relational database
- **SQL** — CTEs, JOINs, aggregations, window filtering
- **Python** — pandas, SQLAlchemy
- **Jupyter Notebook** — DataCamp workspace

---

## 📁 Files

| File | Description |
|---|---|
| `notebook.ipynb` | Main Jupyter notebook with solution |
| `calculator.jpg` | Project asset image |
| `README.md` | This file |

---

## 🧠 Key SQL Concepts Used

- **Common Table Expressions (CTEs)** — modular query building
- **Subqueries** — filtering with `IN (SELECT ...)`
- **Aggregate Functions** — `COUNT()`, `AVG()`, `ROUND()`
- **Multi-table JOINs** — joining 3 tables on `company_id`
- **Date Extraction** — `EXTRACT(YEAR FROM date_column)`
- **Unit Conversion** — dividing valuation by `1,000,000,000` for billions
- **ORDER BY multiple columns** — sorting by year and count descending

---

*Project completed via DataCamp — Analyzing Unicorn Companies*
