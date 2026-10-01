# Data Mining: Spark Data Lake Assignment

**Student:** Abhay Prasad

**Roll number:** LIT2023006

**Dataset:** `sales.csv` from the class repository's `dataset/` folder

## Run in Google Colab

1. Open `Data_Lake_Assignment.ipynb` in [Google Colab](https://colab.research.google.com/github/abhay9494/data-mining-data-lake-assignment/blob/main/submissions/LIT2023006_Abhay_Prasad/Data_Lake_Assignment.ipynb).
2. Use a standard Python CPU runtime and choose **Runtime > Run all**.
3. The first cell uses the runtime's installed PySpark, or installs PySpark 3.5.8 if absent. This submission was executed with Spark 4.0.4 in Google Colab. The notebook downloads the original dataset from a pinned class-repository commit, without requiring a file upload or Google Drive mount.
4. Inspect the schema, source rows, Spark-generated discrepancy counts, before/after row counts, Silver read-back, and verification results.
5. The complete submission output is `silver/sales_silver.csv`. Download it from the Colab Files panel. Download the executed notebook through **File > Download > Download .ipynb**.

The notebook can also run in Jupyter with Python 3.10-3.12, Java 17, and PySpark 3.5.8. Run from this submission folder to use the included Bronze file without downloading it. Google Colab is the intended environment.

## Structure

Committed submission:

```text
submissions/LIT2023006_Abhay_Prasad/
├── Data_Lake_Assignment.ipynb
├── README.md
├── bronze/
│   └── sales.csv                 # byte-for-byte copy of the provided data
├── silver/
│   └── sales_silver.csv          # complete processed data, not just a sample
└── verification_summary.json    # counts calculated by Spark
```

Created in the Colab runtime:

```text
data_lake/
├── bronze/
│   └── sales.csv
└── silver/
    └── preprocessed_sales/
        ├── part-....csv
        └── _SUCCESS
silver/
└── sales_silver.csv
```

Spark writes the Silver directory first. Since this dataset is small, the notebook uses one output part and copies that complete CSV to the filename required for submission. All transformation and validation operations use Spark; Python only handles files, dependency installation, and the source checksum.

## Basic preprocessing

- Remove exact duplicate source rows across all columns before any normalization. Do not deduplicate on order ID alone.
- Trim surrounding spaces and convert blank strings to null. Fill missing descriptive text with `Unknown`; preserve missing identifiers as null.
- Use title case for names, categories, cities, states, and order statuses. For product and payment labels, use the most frequent trimmed spelling in each case-insensitive group, with a deterministic lexicographic tie-break. This preserves meaningful acronyms and product units.
- Require positive integral quantity, positive finite unit price, and finite discount percentage in `[0, 100]`. Invalid or missing values become null, preserving the rest of the record. No invented numerical imputation is applied.
- Parse `yyyy-MM-dd`, `yyyy/MM/dd`, `MM-dd-yyyy`, and `dd/MM/yyyy` with Spark's corrected date parser. Valid dates are written as `yyyy-MM-dd`; invalid dates become null. Year-last slash dates are interpreted day-first.
- Keep plausible unusual values, future dates, and cancelled/returned orders. No Gold layer or detailed business analysis is included.

Null numeric/date values are exported as empty CSV fields and mean unknown or unusable, **not zero**. Numeric columns become integer quantity and double price/discount; order date becomes Spark `DateType`. Initial raw columns are strings so malformed source values remain visible during inspection.

## Verification and provenance

The submitted notebook was executed from top to bottom in Google Colab on October 1, 2026. All 10 code cells completed without errors. Spark calculated **1,000 input rows, 10 exact duplicates removed, and 990 Silver rows**. It identified 12 invalid quantities, 8 invalid prices, 8 invalid discounts, 4 invalid dates, and 2 valid dates requiring format normalization. These results are saved outputs, not hard-coded discrepancy counts.

All discrepancy counts come from Spark operations at runtime. Assertions verify that:

- Bronze has the same SHA-256 as the original provided CSV before and after processing.
- The only removed rows are exact original duplicates.
- Both the Spark Silver directory and the submission CSV read back with the same schema, row count, and complete row multiset as the processed DataFrame.
- Non-null numeric values satisfy the validation rules and descriptive text is normalized.

Source: [class dataset at commit `81539d8`](https://github.com/necromancersanju-netizen/data-mining-data-lake-assignment/blob/81539d8aee71b8ce4c428365cea26e6b272723c6/dataset/sales.csv). The notebook records the expected source checksum, and `verification_summary.json` records the actual calculated results.

All submitted changes are confined to this student's folder. The class README, shared dataset, and other submissions are unchanged.
