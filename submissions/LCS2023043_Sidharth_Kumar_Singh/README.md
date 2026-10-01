# Data Mining Assignment — Apache Spark Data Lake Preprocessing

**Name:** Sidharth Kumar Singh
**Roll Number:** LCS2023043

## Dataset

`sales.csv` — the raw sales dataset provided in the class repository
(`dataset/sales.csv`): 1000 records, 13 columns. It is used unchanged as the
Bronze layer.

## How to run the notebook

1. Open `Data_Lake_Assignment.ipynb` in Google Colab.
2. Select **Runtime → Run all**.
3. The first cell installs PySpark in the Colab runtime, then the notebook
   downloads the raw dataset, builds the data lake and prints the verification
   output. No local setup is required.

## Data Lake structure

```
data_lake/
|-- bronze/
|   `-- sales.csv                 # raw dataset, unchanged
`-- silver/
    |-- preprocessed_sales/       # Spark CSV output (part files)
    `-- sales_silver.csv          # single-file copy of the Silver output
```

## Preprocessing summary (Bronze → Silver)

- Removed 10 exact duplicate rows (1000 → 990).
- Filled missing `customer_name`, `payment_method` and `city` values with
  `Unknown`.
- Trimmed extra whitespace and normalized the capitalization inconsistencies
  found in `category` and `city`.
- Validated `quantity` (> 0), `unit_price` (> 0) and `discount_percent` (0–100);
  clearly invalid values were imputed with the column median
  (5 / 1850 / 20, computed with Spark, not hard-coded).
- Normalized dates from several text formats to ISO `yyyy-MM-dd`; the 4 values
  that are not valid calendar dates were left empty (the rows are preserved).
- Final Silver output: **990 rows × 13 columns**, no duplicate rows.

All discrepancy counts are produced by Spark operations inside the notebook.
