# Data Mining Assignment — Apache Spark Data Lake Preprocessing

**Name:** Aryan Kadian
**Roll Number:** LCS2023051

## Dataset

`sales.csv` — 1000 sales order records with 13 columns:
`order_id, customer_id, customer_name, product, category, quantity, unit_price, discount_percent, payment_method, city, state, order_date, order_status`

## Files in this folder

| Path | Description |
|------|-------------|
| `Data_Lake_Assignment.ipynb` | Complete Colab notebook with the PySpark workflow (executed, with outputs) |
| `bronze/sales.csv` | Bronze layer — raw dataset, unchanged (byte-identical to `dataset/sales.csv`) |
| `silver/sales_silver.csv` | Silver layer — preprocessed output (the single Spark part file) |

## How to run

1. Open `Data_Lake_Assignment.ipynb` in [Google Colab](https://colab.research.google.com/) (File → Upload notebook).
2. Click **Runtime → Run all**.
   - The first cell installs PySpark (`pip install pyspark`).
   - If `sales.csv` has been uploaded to the Colab session it is used; otherwise the notebook downloads it
     automatically from the class repository.
3. The notebook creates the `data_lake/` folders, runs all checks and preprocessing with Spark, writes the
   Silver output and reads it back for verification.

It can also be run locally with Jupyter (requires Java 8/11/17/21 and Python 3.9+).

## Data Lake structure

```
data_lake/
|-- bronze/
|   `-- sales.csv                  <- raw data, copied without modification (MD5 checked)
`-- silver/
    |-- preprocessed_sales/        <- Spark CSV output folder (part-00000-*.csv, _SUCCESS)
    `-- sales_silver.csv           <- copy of the single part file, for submission
```

## Discrepancies identified (via Spark)

- Exact duplicate rows, and `order_id` values reused by different orders
- Missing values in `customer_name`, `payment_method` and `city`
- Extra leading/trailing spaces and inconsistent capitalization (e.g. `beauty` / `SPORTS`, `DELHI` / ` Hyderabad `)
- Invalid numbers: quantity ≤ 0, unit price ≤ 0, discount outside 0–100
- Dates in other formats (`2025/07/14`, `09-15-2026`) and invalid dates (`31/02/2026`, `2025-02-30`, `2026-13-05`, `not-a-date`)

All counts are computed in the notebook with Spark operations — none are hard-coded.

## Preprocessing summary (Silver layer)

1. **Duplicates** — exact duplicate rows removed with `dropDuplicates()`.
2. **Text cleanup** — trimmed spaces, collapsed repeated spaces, blank strings → null; `initcap` for
   `customer_name`, `category`, `city`, `state`, `order_status`; `customer_id` upper-cased.
   `product` and `payment_method` are only trimmed (values like `USB-C Hub`, `UPI` are already consistent).
3. **Missing values** — `customer_name` and `payment_method` → `"Unknown"`; missing `city` filled from `state`
   when that state has only one city in the data, otherwise `"Unknown"`.
4. **Numeric validation** — `quantity` cast to int, `unit_price` / `discount_percent` to double; clearly invalid
   values are set to null while the rest of the record is kept.
5. **Dates** — converted to `DateType` in `yyyy-MM-dd`; recognizable alternate formats are converted and
   impossible / unparseable dates are set to null.
6. **Record preservation** — only duplicates are removed (1000 → 990 rows). A `dq_issues` column flags rows with
   `invalid_quantity`, `invalid_unit_price`, `invalid_discount`, `invalid_date` or `duplicate_order_id`.
7. **Verification** — row counts before/after are shown, the Silver data is read back with Spark, the checks are
   re-run on it, and the Bronze file's MD5 is confirmed unchanged.
