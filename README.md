<p align="center">
  <img src="upatta_logo.png" alt="Upatta Data Analytics" width="220"/>
</p>

<h1 align="center">Kusto Query Language (KQL)</h1>

<p align="center">
  <strong>A comprehensive KQL vs SQL tutorial with NYC Taxi data</strong><br/>
  Author: <strong>Zia Ahmed</strong> · <strong>Upatta Analytics</strong>
</p>

---

## Overview

**Kusto Query Language (KQL)** is a read-only query language developed by Microsoft for querying large datasets in near real-time. It is the primary language used in:

- **[Azure Data Explorer (ADX)](https://azure.microsoft.com/en-us/products/data-explorer/)** — Microsoft's big data analytics service
- **[Microsoft Sentinel](https://azure.microsoft.com/en-us/products/microsoft-sentinel/)** — cloud-native SIEM for security analytics
- **[Azure Monitor / Log Analytics](https://azure.microsoft.com/en-us/products/monitor/)** — infrastructure and application telemetry


KQL is designed for **telemetry, logs, time-series data, and observability** — querying billions of rows of machine-generated data in seconds.

This repository contains an interactive Jupyter notebook that teaches **KQL and SQL side-by-side** using a synthetic NYC Taxi dataset. KQL runs natively on Azure Data Explorer and Microsoft Sentinel; the notebook simulates KQL's pipeline syntax with Pandas method-chaining and executes real SQL via **DuckDB**.

## Tutorial

| File | Description |
|------|-------------|
| [`kql_sql_nyc_taxi_tutorial.ipynb`](kql_sql_nyc_taxi_tutorial.ipynb) | Full hands-on tutorial — 16 sections with side-by-side KQL and SQL examples |
| [`kql_sql_nyc_taxi_tutorial.html`](kql_sql_nyc_taxi_tutorial.html) | Static HTML export — open in any browser (no Jupyter required) |

## Table of Contents

1. Setup & Dependencies
2. Dataset Generation
3. Language Philosophy
4. Basic Filtering
5. Column Selection (`project` / `SELECT`)
6. Derived / Computed Columns (`extend` / computed `SELECT`)
7. Aggregation (`summarize` / `GROUP BY`)
8. Sorting & Limiting (`order by` + `take` / `ORDER BY LIMIT`)
9. String Operations
10. Date/Time Operations
11. Joins
12. Subqueries vs `let` Statements
13. Window Functions
14. NULL Handling
15. Complex Multi-Step Analytics
16. Performance Tips & Cheat Sheet

## Dataset

The notebook generates **5,000 synthetic NYC taxi trips** spanning January–February 2024. The schema mirrors the real [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).

| Column | Type | Description |
|--------|------|-------------|
| `trip_id` | int | Unique trip identifier |
| `pickup_datetime` | datetime | Trip start timestamp |
| `dropoff_datetime` | datetime | Trip end timestamp |
| `passenger_count` | int | Number of passengers (has NULLs) |
| `trip_distance` | float | Distance in miles |
| `fare_amount` | float | Base fare ($) |
| `tip_amount` | float | Tip amount (has NULLs) |
| `payment_type` | str | Credit Card / Cash / … |
| `pickup_borough` | str | NYC borough |
| `dropoff_borough` | str | NYC borough |
| `vendor_id` | str | CMT / VTS / DDS |
| `rate_code` | str | Standard / JFK / Newark / … |
| `total_amount` | float | fare + tip + surcharges |
| `trip_duration_min` | float | Duration in minutes |

## Getting Started

### Prerequisites

- Python 3.11+
- Jupyter Notebook or JupyterLab

### Install & Run

```bash
pip install pandas numpy duckdb matplotlib jupyter
jupyter notebook kql_sql_nyc_taxi_tutorial.ipynb
```

Or install dependencies from within the notebook:

```python
%pip install pandas numpy duckdb matplotlib --quiet
```

## KQL Syntax: The Pipe Model

KQL uses a **left-to-right pipeline** style. You start with a table and chain operations using the `|` (pipe) operator:

```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| summarize FailedLogins = count() by Account
| order by FailedLogins desc
| take 10
```

Reading this like a sentence: *"From SecurityEvent, filter to the last 24 hours, keep only failed login events, count them per account, sort descending, show the top 10."*

### Core KQL Operators

| Operator | Purpose | Example |
|---|---|---|
| `where` | Filter rows | `where Level == "Error"` |
| `project` | Select/rename columns | `project Name, Age, City` |
| `summarize` | Aggregate & group | `summarize count() by Region` |
| `extend` | Add computed columns | `extend Duration = EndTime - StartTime` |
| `order by` | Sort results | `order by Timestamp desc` |
| `take` / `limit` | Limit row count | `take 100` |
| `join` | Join two tables | `T1 \| join T2 on Key` |
| `union` | Combine tables | `union T1, T2` |
| `parse` | Extract from strings | `parse msg with * "user=" User ","` |
| `mv-expand` | Expand arrays | `mv-expand tags` |
| `render` | Visualize results | `render timechart` |

## KQL vs SQL — Key Differences

| Dimension | KQL | SQL |
|---|---|---|
| **Style** | Pipeline (left-to-right) | Declarative (clause-based) |
| **Read/Write** | Read-only | Read + Write + DDL |
| **Primary use case** | Logs, telemetry, time-series | Relational/transactional data |
| **Time functions** | First-class (`ago()`, `bin()`, `startofday()`) | Bolted on, varies by dialect |
| **Filtering** | `where` early and often | `WHERE` clause |
| **Aggregation** | `summarize ... by` | `GROUP BY` |
| **Column selection** | `project` | `SELECT` |
| **String parsing** | `parse`, `extract()`, `split()` | Limited, dialect-specific |
| **Schema flexibility** | Dynamic columns, JSON nesting | Strict schema |
| **Visualization** | Built-in `render` operator | None |

### Same query, two languages

**SQL:**
```sql
SELECT Account, COUNT(*) AS FailedLogins
FROM SecurityEvent
WHERE TimeGenerated > DATEADD(hour, -24, GETUTCDATE())
  AND EventID = 4625
ORDER BY FailedLogins DESC
LIMIT 10;
```

**KQL:**
```kql
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| summarize FailedLogins = count() by Account
| order by FailedLogins desc
| take 10
```

## Language Philosophy

| | KQL | SQL |
|---|---|---|
| **Mental model** | *"Transform this table step by step"* | *"Describe the result set I want"* |
| **Flow** | Top → bottom pipeline | Clauses in non-execution order |
| **Exploration** | Natural for iterative analysis | Powerful but clause ordering can feel counterintuitive |

## Concepts Covered

| Category | KQL key operators | SQL equivalents |
|---|---|---|
| Filter | `where` | `WHERE`, `HAVING` |
| Select | `project`, `project-away` | `SELECT`, `EXCLUDE` |
| Derive | `extend` | computed `SELECT` |
| Aggregate | `summarize … by` | `GROUP BY` |
| Sort/Limit | `order by`, `take`, `top` | `ORDER BY`, `LIMIT` |
| Strings | `contains`, `strcat`, `toupper` | `LIKE`, `\|\|`, `UPPER` |
| Dates | `bin()`, `ago()`, `hourofday()` | `DATE_TRUNC`, `INTERVAL`, `EXTRACT` |
| Joins | `join kind=…` (6 kinds) | `JOIN` types |
| Variables | `let` | CTEs (`WITH`) |
| Windows | `row_rank()`, `prev()`, `row_cumsum()` | `RANK()`, `LAG()`, `SUM() OVER` |
| Nulls | `isnull`, `coalesce`, `countif` | `IS NULL`, `COALESCE`, `CASE WHEN` |

## When to Use KQL

- Security investigations & threat hunting
- Application performance monitoring
- Infrastructure log analysis
- Time-series analytics at scale
- Azure-native observability stacks

If you're working in **Microsoft Sentinel, Log Analytics, or Azure Data Explorer**, KQL is the go-to. For traditional relational databases and transactional systems, SQL remains the standard.

## Next Steps

- **Try real KQL** → [Azure Data Explorer playground](https://dataexplorer.azure.com/clusters/help/databases/Samples)
- **KQL in Jupyter** → `%pip install kqlmagic` then `%kql adx://cluster.region/db`
- **DuckDB on your own files** → `duckdb.read_csv("file.csv")` or `duckdb.read_parquet("file.parquet")` — zero setup!

---

<p align="center">
  <sub>© Zia Ahmed · Upatta Analytics · A complete Data Solution</sub>
</p>
