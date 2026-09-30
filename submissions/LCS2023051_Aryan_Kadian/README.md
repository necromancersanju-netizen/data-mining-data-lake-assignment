# Apache Spark Data Lake Preprocessing

**Name:** Aryan Kadian
**Roll Number:** LCS2023051
**Dataset:** `sales.csv` (1,000 sales orders, 13 columns: order, customer, product, quantity, price, discount, payment, location, date, status)
**Environment:** Google Colab + PySpark (tested on PySpark 3.5 and 4.x)

## Files

| File | Description |
|------|-------------|
| `Data_Lake_Assignment.ipynb` | Full Spark workflow, with outputs |
| `bronze/sales.csv` | Raw dataset, byte-for-byte identical to `dataset/sales.csv` |
| `silver/sales_silver.csv` | Preprocessed Silver-layer output (single CSV) |

## How to run the notebook

1. Open `Data_Lake_Assignment.ipynb` in Google Colab (**File → Upload notebook**, or open it from GitHub).
2. Click **Runtime → Run all**.
   - PySpark is installed automatically if it is missing.
   - The notebook looks for `sales.csv` locally (`sales.csv` or `dataset/sales.csv`). If it is not found, the file is downloaded from the class repository, so no manual upload is needed.
3. The notebook creates the `data_lake/` folders, runs the preprocessing, writes the Silver output and reads it back to check it.

The notebook runs from top to bottom. Every discrepancy count is computed with Spark, not hard-coded.

## Data Lake structure

```
data_lake/
|-- bronze/
|   `-- sales.csv                 # raw data, unchanged (md5 is checked at the end)
`-- silver/
    |-- preprocessed_sales/       # Spark CSV output (part-00000-*.csv + _SUCCESS)
    `-- sales_silver.csv          # the same data copied to one file
```

## Discrepancies found (by Spark)

| Issue | Count |
|-------|------:|
| Exact duplicate rows | 10 |
| `order_id` reused by different (non-identical) rows | 8 |
| Missing `customer_name` / `payment_method` / `city` | 10 / 8 / 8 |
| Text values with extra spaces | 6 |
| Text values with inconsistent capitalization (e.g. `ELECTRONICS`, `delhi`) | 12 |
| `quantity` <= 0 | 12 |
| `unit_price` <= 0 | 8 |
| `discount_percent` outside 0–100 (`-5`, `120`) | 8 |
| Non-standard or impossible `order_date` | 6 |

## Preprocessing performed (Silver layer)

1. **Duplicates:** exact duplicate rows were removed with `dropDuplicates()`. Rows that share an `order_id` but have different content were kept, because they are separate orders.
2. **Missing values:** a missing `customer_name` was filled from other rows with the same `customer_id` (this worked for all 10). Missing `payment_method` and `city` were filled with `"Unknown"`. No rows were dropped.
3. **Text cleanup:** leading, trailing and repeated spaces were removed. `customer_name`, `category`, `city`, `state` and `order_status` were converted to Title Case. `product` and `payment_method` were only trimmed, so acronyms like `UPI` and `USB-C` stay intact.
4. **Numeric validation:** `quantity` became an integer and must be > 0. `unit_price` must be > 0. `discount_percent` must be between 0 and 100. Invalid values were set to NULL and the rest of the record was kept.
5. **Dates:** `yyyy-MM-dd`, `yyyy/MM/dd`, `dd/MM/yyyy` and `MM-dd-yyyy` were parsed into one date column, written as `yyyy-MM-dd`. Impossible dates (`not-a-date`, `2026-13-05`, `2025-02-30`, `31/02/2026`) were set to NULL and the rows were kept.
6. **Records kept:** only the 10 exact duplicates were removed. Unusual but possible values, such as high prices, were not changed.

**Row count:** 1,000 in Bronze → 990 in Silver.
