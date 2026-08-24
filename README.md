# NORDSTORM-CASE-STUDY
# 🛍️ Threadbare Insights: A Customer Shopping Behavior EDA

> *3,900 shoppers. 19 attributes. 10 business questions. One pipeline that takes raw retail data from CSV → cleaned DataFrame → PostgreSQL → boardroom-ready insight.*

This repository walks through a full **exploratory data analysis (EDA) + SQL analytics** project built on a real-world-style e-commerce customer dataset spanning clothing, footwear, accessories, and outerwear. It's designed to mirror how a data analyst would actually work inside a retail company: clean messy data in Python, land it in a relational database, and answer the questions a merchandising or marketing team would actually ask.

---

## 🧭 Why this project exists

Retailers sit on mountains of transactional data but routinely struggle to turn it into decisions. This project simulates that exact workflow — starting from a raw shopping behavior export and ending with concrete, revenue-relevant answers about *who* buys, *what* they buy, *when* they buy, and *whether* discounts and subscriptions are actually working.

A full business case study — modeled on how real apparel retailers like **Nordstrom** use customer analytics to power loyalty programs, personalized offers, and shipping strategy — is included as a PDF in this repo (see [`Problem_Statement_Case_Study.pdf`](./Problem_Statement_Case_Study.pdf)).

---

## 📦 Dataset at a glance

| | |
|---|---|
| **Rows** | 3,900 customers |
| **Raw source file** | `customer_shopping_behavior.csv` |
| **Cleaned/engineered output** | `customer.csv` (post feature-engineering, exported from PostgreSQL) |
| **Total revenue represented** | $233,081 |
| **Avg. purchase amount** | $59.76 |
| **Avg. review rating** | 3.75 / 5 |
| **Categories** | Clothing, Accessories, Footwear, Outerwear |
| **Discount usage** | 43% of transactions |
| **Subscribers** | 1,053 (27%) vs. 2,847 non-subscribers |
| **Geography** | 50 U.S. states |

**Key columns:** `customer_id`, `age`, `gender`, `item_purchased`, `category`, `purchase_amount`, `location`, `size`, `color`, `season`, `review_rating`, `subscription_status`, `shipping_type`, `discount_applied`, `previous_purchases`, `payment_method`, `frequency_of_purchases`, plus two engineered columns: `age_group` and `purchase_frequency_days`.

---

## 🛠️ Tools & tech stack

| Layer | Tool | Purpose |
|---|---|---|
| **Data wrangling** | `pandas`, `numpy` | Null handling, dtype fixes, column renaming |
| **Quick visuals / sanity checks** | `seaborn` | Value distributions during EDA |
| **Feature engineering** | `pandas.qcut`, custom mappings | Built `age_group` (quartile-based) and `purchase_frequency_days` (mapped from purchase frequency labels) |
| **Database** | `PostgreSQL` | Persisted the cleaned dataset for SQL-based analysis |
| **DB connectivity** | `SQLAlchemy`, `psycopg2-binary` | Pushed the cleaned DataFrame into Postgres (`df.to_sql`) and read it back |
| **Analysis language** | `SQL` (PostgreSQL dialect) | Window functions, CTEs, conditional aggregation for all 10 business questions |
| **Notebook environment** | Jupyter (`conda` base env) | `EDA_PROJECT.ipynb` — the full, reproducible workflow |

---

## 🔬 The workflow

```
customer_shopping_behavior.csv
        │
        ▼
┌─────────────────────────────┐
│ 1. Load & Inspect            │  df.head(), df.info(), .isnull().sum()
│    (pandas)                  │  value_counts() on gender, category, season,
│                               │  subscription, shipping, location
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 2. Clean                     │  Impute missing review_rating with the
│                               │  category-level mean
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 3. Standardize                │  Lowercase + underscore column names;
│                               │  rename purchase_amount_(usd) → purchase_amount
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 4. Engineer features          │  age_group (quartile bins: Young Adult /
│                               │  Adult / Middle-aged / Senior)
│                               │  purchase_frequency_days (Weekly → 7,
│                               │  Monthly → 30, Annually → 365, etc.)
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 5. Deduplicate signals         │  Verified discount_applied and
│                               │  promo_code_used were identical columns
│                               │  → dropped the redundant one
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 6. Load to PostgreSQL         │  SQLAlchemy engine → df.to_sql("customer")
└──────────────┬────────────────┘
               ▼
┌─────────────────────────────┐
│ 7. Answer business questions  │  10 SQL queries — insights.sql
│    in SQL                     │
└──────────────┬────────────────┘
               ▼
        customer.csv (final, query-ready export)
```

---

## ❓ Business questions answered (`insights.sql`)

Each question below was answered with a dedicated SQL query against the `customer` table in Postgres — aggregate functions, `CASE` logic, CTEs, and a `ROW_NUMBER()` window function are all used across the set.

| # | Question | SQL technique used |
|---|---|---|
| 1 | Total revenue generated by male vs. female customers | `GROUP BY` + `SUM` |
| 2 | Which customers used a discount but still spent above the average purchase amount? | Correlated scalar subquery (`AVG` in `WHERE`) |
| 3 | Top 5 products by average review rating | `GROUP BY` + `AVG` + `ORDER BY ... LIMIT` |
| 4 | Standard vs. Express shipping — average purchase amount | Filtered `GROUP BY` |
| 5 | Do subscribers spend more than non-subscribers? (avg spend & total revenue) | Multi-metric `GROUP BY` |
| 6 | Top 5 products by % of purchases with a discount applied | Conditional aggregation (`CASE WHEN` inside `SUM`) |
| 7 | Segment customers into New / Returning / Loyal by purchase history | CTE + `CASE` segmentation |
| 8 | Top 3 best-selling products *within each* category | CTE + `ROW_NUMBER() OVER (PARTITION BY ...)` |
| 9 | Are repeat buyers (5+ past purchases) more likely to be subscribers? | Filtered `GROUP BY` |
| 10 | Revenue contribution by age group | `GROUP BY` + `SUM` + `ORDER BY` |

> 💡 **Sample finding:** Male customers significantly outnumber female customers in this dataset (2,652 vs. 1,248), which materially skews the gender revenue split in Q1 — worth flagging in any report built on top of this data.

---

## 📁 Repository structure

```
.
├── EDA_PROJECT.ipynb                  # End-to-end cleaning + feature engineering notebook
├── customer_shopping_behavior.csv     # Raw source data
├── customer.csv                       # Cleaned, feature-engineered, query-ready dataset
├── insights.sql                       # 10 business questions answered in SQL
├── Problem_Statement_Case_Study.pdf   # Real-world business framing (case-study style)
└── README.md                          # You are here
```

---

## ▶️ How to reproduce this

1. **Clone the repo** and install the Python dependencies:
   ```bash
   pip install pandas numpy seaborn sqlalchemy psycopg2-binary
   ```
2. **Spin up PostgreSQL** locally (or point at a hosted instance) and create a database.
3. **Run `EDA_PROJECT.ipynb`** top to bottom — it loads the raw CSV, cleans it, engineers `age_group` and `purchase_frequency_days`, and writes the result into your `customer` table via SQLAlchemy.
   > ⚠️ Before running, replace the hardcoded database credentials in the notebook with your own — ideally loaded from environment variables or a `.env` file rather than committed to the notebook. Never commit real credentials to a public repo.
4. **Run the queries in `insights.sql`** against your `customer` table (psql, DBeaver, pgAdmin, or any SQL client) to reproduce all 10 insights.

---

## 📈 What this project demonstrates

- End-to-end data pipeline thinking: raw file → cleaned data → relational database → SQL analytics
- Practical `pandas` cleaning (missing-value imputation strategy chosen per-column, not blanket dropna)
- Thoughtful feature engineering (quartile-based segmentation, mapping categorical frequency to numeric recency)
- Real SQL fluency: subqueries, CTEs, conditional aggregation, and window functions — not just `SELECT *`
- Business framing: every query maps to a question a merchandising, marketing, or retention team would actually ask

---

## 🚀 Possible next steps

- Build a Tableau/Power BI dashboard on top of the `customer` table
- Add a churn/RFM (Recency-Frequency-Monetary) segmentation layer
- A/B test whether discount usage actually drives incremental revenue vs. cannibalizing full-price sales
- Model subscription conversion likelihood based on shopping behavior

---

## 📄 License

This project is shared for educational and portfolio purposes. The dataset is a synthetic/sample retail dataset used for learning SQL and EDA workflows.
