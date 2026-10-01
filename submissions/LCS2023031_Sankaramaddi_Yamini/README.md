# Apache Spark Data Lake Preprocessing Assignment

**Course:** B.Tech 7th Semester Data Mining Lab  
**Student Name:** Sankaramaddi Yamini  
**Roll Number:** LCS2023031  
**Dataset:** `sales.csv` (1,000 raw transaction records)  
**Environment:** Google Colab / Apache Spark (PySpark 3.x/4.x)  

---

## 1. Overview & Data Lake Architecture

This project implements a two-tier Medallion Data Lake architecture using PySpark to preprocess raw e-commerce sales transactions:

```
data_lake/
├── bronze/
│   └── sales.csv                   <- Raw, unmodified source dataset
└── silver/
    └── preprocessed_sales/         <- Cleaned, validated, and standardized dataset (CSV)
```

- **Bronze Layer (`data_lake/bronze/`):** Contains the raw `sales.csv` dataset ingested without any modifications or transformations.
- **Silver Layer (`data_lake/silver/`):** Contains the preprocessed and validated dataset where duplicates, missing values, text format inconsistencies, numerical corruptions, and invalid dates have been resolved.

---

## 2. How to Run the Notebook in Google Colab

1. **Upload Notebook:**
   - Open [Google Colab](https://colab.research.google.com/).
   - Click **File > Upload notebook** and select `Data_Lake_Assignment.ipynb`.

2. **Run All Cells:**
   - Click **Runtime > Run all** (or `Ctrl+F9`).
   - The first code cell installs `pyspark` and automatically detects or fetches `sales.csv` into `data_lake/bronze/sales.csv`.
   - The notebook executes top-to-bottom without manual intervention, dynamically calculating all discrepancy metrics through Spark operations.

3. **Output Files Generated:**
   - Bronze layer: `data_lake/bronze/sales.csv`
   - Silver layer: `data_lake/silver/preprocessed_sales/`
   - Submission CSV: `submissions/<ROLL_NUMBER>_<NAME>/silver/sales_silver.csv`

---

## 3. Discrepancies Identified via Spark Operations

The dataset was profiled using Apache Spark operations (no hard-coded numbers):

| Discrepancy Type | Spark Detection Method | Found |
| :--- | :--- | :--- |
| **Exact Duplicates** | `bronze_df.count() - bronze_df.dropDuplicates().count()` | 10 duplicate rows |
| **Missing Values** | `F.col(col).isNull() | (F.trim(F.col(col)) == "")` | `customer_name`: 10, `payment_method`: 8, `city`: 8 |
| **Whitespace Inconsistencies** | `F.col(col) != F.trim(F.col(col))` | `customer_name`: 3 rows, `city`: 3 rows |
| **Capitalization Inconsistencies** | `groupBy(col).count()` | Inconsistent casing in `category` (`sports`, `SPORTS`, `beauty`, `ELECTRONICS`, `CLOTHING`) and `city` (`DELHI`, `HYDERABAD`, `coimbatore`, `JAIPUR`, `delhi`) |
| **Invalid Numerical Values** | Boundary condition filters (`quantity <= 0`, `unit_price <= 0`, `discount < 0 or > 100`) | `quantity <= 0`: 12 rows<br>`unit_price <= 0`: 8 rows<br>`discount_percent < 0 or > 100`: 8 rows |
| **Inconsistent & Invalid Dates** | Regex check `^\d{4}-\d{2}-\d{2}$` and multi-format `coalesce(to_date(...))` | Non-standard valid dates: 2 (`2025/07/14`, `09-15-2026`)<br>Impossible dates: 4 (`31/02/2026`, `not-a-date`, `2025-02-30`, `2026-13-05`) |

---

## 4. Preprocessing Operations Performed (Silver Layer)

1. **Deduplication:**
   - Executed `dropDuplicates()` on the Bronze DataFrame to remove exact duplicate rows (reduced from 1,000 to 990 rows).
2. **Missing Value Handling:**
   - Imputed missing text values in `customer_name`, `payment_method`, and `city` with `'Unknown'` via `fillna(...)`. This preserved 26 valid financial transactions without dropping revenue records.
3. **Text Standardization:**
   - Trimmed leading and trailing whitespace across all string fields using `F.trim()`.
   - Standardized capitalization to Title Case using `F.initcap()` on `category`, `city`, `state`, and `order_status`.
   - Preserved standard banking acronym `UPI` for payment methods.
4. **Numerical Validation:**
   - Applied business validation rules: filtered records where `quantity > 0`, `unit_price > 0`, and `0 <= discount_percent <= 100`. Removed 28 corrupted records.
5. **Date Standardization & Validation:**
   - Parsed mixed date patterns (`yyyy-MM-dd`, `yyyy/MM/dd`, `MM-dd-yyyy`, `dd/MM/yyyy`) and standardized them into uniform `yyyy-MM-dd` format.
   - Filtered out 4 unparseable / calendar-impossible dates.
6. **Record Preservation:**
   - Preserved valid transactions with non-standard business statuses (`Cancelled`, `Returned`) and legitimate edge values (large orders, high unit prices).

---

## 5. Verification Results

- **Row Count Before Preprocessing (Bronze):** `1,000`
- **Row Count After Preprocessing (Silver):** `958`
- **Total Records Dropped:** `42` (10 duplicates + 12 invalid quantities + 8 invalid prices + 8 invalid discounts + 4 invalid dates)
- **Null Count in Silver Layer:** `0` across all 13 columns
- **Date Format in Silver Layer:** 100% compliant `yyyy-MM-dd`
