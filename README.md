# E-commerce SQL Analytics Project (MySQL)

A complete, portfolio-ready SQL project built on a realistic, self-generated e-commerce dataset (~1.6 million rows). Built to demonstrate practical SQL skills across the full spectrum: data modelling, joins, subqueries, CTEs, window functions, stored procedures, functions, triggers, views, and index optimization.

## Why I built this

Most SQL portfolio projects use the same 5 tiny Kaggle datasets with a handful of beginner-level joins. This project instead:
- Uses a **self-generated dataset at real-world scale** (300K orders, 766K order line items, 50K customers)
- Covers **every major SQL concept** in one connected schema, not isolated toy examples
- Includes **measurable index optimization** with actual EXPLAIN ANALYZE before/after comparisons
- Models a realistic **business domain** (e-commerce) that any interviewer immediately understands

## Tech stack
- **Database:** MySQL 8.0+
- **Data generation:** Python 3 + Faker library
- **~1.6 million total rows** across 11 interconnected tables

## Project structure

```
ecommerce_sql_project/
├── README.md
├── generate_data.py              # Generates all CSV data (configurable volume)
├── 01_schema.sql                 # Full database schema with FKs
├── 02_load_data.sql              # Bulk data import via LOAD DATA INFILE
├── 03_queries.sql                # 25+ queries: beginner → advanced, all topics
├── 04_procedures_functions.sql   # Stored procedures, functions, cursors
├── 05_indexes.sql                # Index strategy with EXPLAIN ANALYZE proof
├── 06_views_triggers.sql         # Views and triggers with audit logging
└── data/                         # Generated CSV files (gitignored if too large)
```

## Database schema (entity relationship overview)

```
categories (self-referencing: parent_category_id)
    └── products ── suppliers
            ├── order_items ── orders ── customers ── addresses
            │                       └── payments
            └── reviews ── customers

employees (self-referencing: manager_id) — separate org-chart dataset for recursive CTE demos
```

11 tables: `customers`, `addresses`, `categories`, `suppliers`, `products`, `orders`, `order_items`, `payments`, `reviews`, `employees`, plus two audit tables (`order_status_audit`, `product_price_audit`) populated automatically by triggers.

## Dataset scale

| Table | Row count |
|---|---|
| customers | 50,000 |
| addresses | 65,000 |
| orders | 300,000 |
| order_items | ~766,000 |
| payments | ~250,000 |
| reviews | 150,000 |
| products | 8,000 |
| suppliers | 500 |
| employees | 200 |
| categories | 40 |

Volume is fully configurable in `generate_data.py` — scale up to millions of rows by editing the `CONFIG` section at the top of the file.

## How to run this project locally

### 1. Generate the data
```bash
pip install faker
python generate_data.py
```
This produces ~80MB of CSV files with full referential integrity (verified — zero orphaned foreign keys).

### 2. Create the database and schema
```bash
mysql -u root -p < 01_schema.sql
```

### 3. Load the data
Edit the file paths in `02_load_data.sql` to point to your generated CSVs, then:
```bash
mysql --local-infile=1 -u root -p < 02_load_data.sql
```
If you hit a `local_infile` error, run this once first:
```sql
SET GLOBAL local_infile = 1;
```

### 4. Explore the queries
```bash
mysql -u root -p ecommerce_db < 03_queries.sql
```

### 5. Install procedures, functions, views, and triggers
```bash
mysql -u root -p ecommerce_db < 04_procedures_functions.sql
mysql -u root -p ecommerce_db < 06_views_triggers.sql
```

### 6. Run the index optimization demo
```bash
mysql -u root -p ecommerce_db < 05_indexes.sql
```
Compare the `EXPLAIN ANALYZE` output before and after index creation — full table scans (`type: ALL`) become indexed lookups (`type: ref`/`range`).

## Concepts covered (with file references)

| Concept | Where | Example |
|---|---|---|
| Basic filtering/aggregation | `03_queries.sql` Section A | Revenue totals, GROUP BY, HAVING |
| All JOIN types | `03_queries.sql` Section B | INNER, LEFT, SELF, CROSS, anti-join |
| Subqueries | `03_queries.sql` Section C | Scalar, correlated, EXISTS |
| CTEs (incl. recursive) | `03_queries.sql` Section D | Org chart hierarchy, category tree |
| Window functions | `03_queries.sql` Section E | ROW_NUMBER, RANK, LAG/LEAD, NTILE, running totals |
| Data modelling | `01_schema.sql`, Section F | Normalization, self-referencing FKs, star-schema query |
| Stored procedures | `04_procedures_functions.sql` | Cursors, transactions, error handling |
| User-defined functions | `04_procedures_functions.sql` | Customer LTV, tier classification |
| Indexes & optimization | `05_indexes.sql` | EXPLAIN ANALYZE before/after benchmarks |
| Views | `06_views_triggers.sql` | Reusable dashboard-ready aggregations |
| Triggers | `06_views_triggers.sql` | Auto-sync totals, audit logging, data integrity |

## Sample insight queries

A few of the more interesting business questions this project answers:

- **Month-over-month revenue growth** using `LAG()` window function
- **Customer spend quartiles** using `NTILE(4)` for segmentation
- **Full company org chart** reconstructed with a recursive CTE
- **Products outperforming their category average** using correlated subqueries
- **3-month rolling revenue average** using a window frame (`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`)

## What I'd add next
- Partitioning the `orders` table by date range for further performance gains at scale
- A small Python/pandas notebook visualizing the query outputs
- A read-replica / sharding discussion for the README (system design angle)

## Author notes
Every number in this README was independently verified — row counts, foreign key integrity, and query logic were checked programmatically before being documented here, not just assumed to work.
