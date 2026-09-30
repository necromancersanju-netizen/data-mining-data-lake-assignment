# Data Mining Lab 1: Spark Data Lake Preprocessing

**Name:** Piyush Kumar

**Roll number:** LIT2023016

Completed PySpark workflow for `sales.csv` from the
[class repository](https://github.com/necromancersanju-netizen/data-mining-data-lake-assignment).
The executed notebook includes explanations, Spark-generated discrepancy counts,
preprocessing, CSV output, and read-back verification.

## Run in Google Colab

1. Open [Google Colab](https://colab.research.google.com/) and upload `Data_Lake_Assignment.ipynb`.
2. The first Markdown cell identifies Piyush Kumar (LIT2023016).
3. Choose **Runtime > Run all**. The notebook installs PySpark 4.0.1 and downloads the original dataset automatically.
4. Inspect the printed tables and verification result. Download the executed notebook with **File > Download > Download .ipynb**.
5. Use Colab's Files panel to download `silver/sales_silver.csv`.

Internet access is needed for the first dependency installation and dataset download.
The submitted notebook was run from top to bottom in Google Colab with
PySpark 4.0.1. All 10 code cells contain successful saved outputs, including
the `/content/data_lake/silver/preprocessed_sales` output location and final
verification result. The downloaded Silver CSV exactly matches the independently
verified output. The workflow also passed a separate local Linux execution.
Other Jupyter environments need Python 3.9+ and Java 17+.

## Runtime folder structure

The submission contains the executed notebook, this README, the unchanged
Bronze CSV, and `silver/sales_silver.csv`. Running the notebook also creates the
Spark output directory shown below.

```text
Data_Lake_Assignment.ipynb
README.md
data_lake/
  bronze/
    sales.csv
  silver/
    preprocessed_sales/
      part-....csv
      _SUCCESS
silver/
  sales_silver.csv
```

Bronze contains the exact source bytes. The submission-local `.gitattributes`
prevents Git from converting CSV line endings, including on Windows. Spark creates a directory for its Silver
CSV output. `silver/sales_silver.csv` is a copy of the complete Spark data part,
provided under the filename requested for submission. It contains all 990 retained records.

## What preprocessing does

- Remove duplicate rows only when every original field matches.
- Trim surrounding spaces and standardize capitalization in names, categories,
  cities, states, and order status. Preserve product spellings and the acronym UPI.
- Replace missing descriptive text with `Unknown`.
- Require positive whole-number quantity, positive unit price, and discount
  percentage between 0 and 100 inclusive. Invalid values become null.
- Parse `yyyy-MM-dd`, `yyyy/MM/dd`, `dd/MM/yyyy`, and `MM-dd-yyyy` dates.
  Write valid dates as `yyyy-MM-dd`; invalid calendar dates become null.
- Retain useful records and flag missing/invalid information in `data_quality_issues`.
- Save Silver as CSV, reload it with Spark, compare its complete contents, and
  confirm the Bronze checksum is unchanged.

Zero quantity and zero price are treated as invalid for this sales exercise.
No values are invented for a missing number or date. CSV nulls are empty fields.
No arbitrary outlier removal, sales aggregation, model training, or Gold layer is used.

## Verified results for this source revision

These are a snapshot of the executed results. The notebook calculates its counts
from the DataFrame; none of these discrepancy counts is hard-coded in its workflow.

| Measure | Result |
|---|---:|
| Bronze rows | 1,000 |
| Exact duplicate rows removed | 10 |
| Silver rows written and read back | 990 |
| Retained rows with a quality flag | 58 |
| Order IDs with multiple different retained rows | 8 |
| Missing customer names filled with Unknown | 10 |
| Missing payment methods filled with Unknown | 8 |
| Missing cities filled with Unknown | 8 |
| Invalid quantities set to null | 12 |
| Invalid prices set to null | 8 |
| Invalid discounts set to null | 8 |
| Invalid dates set to null | 4 |

Two valid nonstandard date strings become ISO dates:
`2025/07/14` becomes `2025-07-14`, and `09-15-2026` becomes `2026-09-15`.
Examples such as `31/02/2026` and `2025-02-30` are invalid dates and become null.
The 8 repeated order IDs are retained because their rows have different contents.

All 10 code cells ran successfully in Google Colab. Spark verified row counts, valid numerical ranges,
unchanged Bronze bytes, and exact read-back contents including row multiplicity.
An independent Python check also matched every Silver record to the documented rules.

## How to explain the lab

**Bronze** is the original evidence. **Silver** is the version prepared for later
use. We remove redundant copies and fix simple formats, but do not pretend to know
missing or impossible values. A null and a quality flag make those problems visible
while preserving the remaining information in the transaction.

`dropDuplicates()` compares complete rows. `trim()` and `initcap()` clean text.
`when()` applies validation rules. `try_cast()` and `try_to_timestamp()` safely parse
numbers and dates. `write.csv()` saves Silver, and `spark.read.csv()` verifies it.

## Class submission

The second page of the assignment specifies the class repository submission layout:

```text
submissions/LIT2023016_Piyush_Kumar/
  Data_Lake_Assignment.ipynb
  silver/sales_silver.csv
```

Fork the class repository and put these files in your own submission folder.
To include the README and Bronze copy requested on page 1, place them inside
that same student folder. Do not replace the class repository's root README or
dataset, or alter another student's folder. Open a pull request with the title:

`Data Mining Assignment - LIT2023016 - Piyush Kumar`

All submitted files are contained in `submissions/LIT2023016_Piyush_Kumar/`.
The repository-level README and dataset are unchanged.

## Source and reproducibility

- Dataset: [sales.csv at the pinned source revision](https://github.com/necromancersanju-netizen/data-mining-data-lake-assignment/blob/81539d8aee71b8ce4c428365cea26e6b272723c6/dataset/sales.csv)
- Source revision: `81539d8aee71b8ce4c428365cea26e6b272723c6`
- Bronze SHA-256: `1b2c7e0acea6250c992b890975ed0947b5bececa6a2880b707e4523e0ee22eb8`
- [PySpark 4.0.1 installation requirements](https://spark.apache.org/docs/4.0.1/api/python/getting_started/install.html)
- [Spark Column.try_cast](https://spark.apache.org/docs/4.0.1/api/python/reference/pyspark.sql/api/pyspark.sql.Column.try_cast.html)
