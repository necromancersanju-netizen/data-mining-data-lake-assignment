# Data Mining Assignment — Apache Spark Data Lake Preprocessing

**Student Name:** Mayank Mishra  
**Roll Number:** LCS2023047  
**Course:** Data Mining  
**Dataset:** `sales.csv`  
**Environment:** Google Colab + Apache Spark (PySpark)  

---

## 1. Dataset Overview

The dataset `sales.csv` contains **1,000 sales transactions** across 13 columns:
- `order_id`: Unique identifier for an order (String / Integer)
- `customer_id`: Identifier for the purchasing customer (String)
- `customer_name`: Name of the customer (String)
- `product`: Product name/item purchased (String)
- `category`: Category of product (String)
- `quantity`: Quantity of items purchased (Integer)
- `unit_price`: Unit price of product (Double)
- `discount_percent`: Discount applied (Double, 0–100%)
- `payment_method`: Mode of payment (String)
- `city`: Delivery city (String)
- `state`: Delivery state (String)
- `order_date`: Date the order was placed (Date / String)
- `order_status`: Current lifecycle status of order (String)

---

## 2. Directory Structure

```
submissions/LCS2023047_Mayank_Mishra/
|-- Data_Lake_Assignment.ipynb         # Complete executed Colab/Jupyter notebook with PySpark workflow
|-- README.md                          # Assignment documentation and run guide
|-- bronze/
|   `-- sales.csv                      # Raw bronze dataset (strictly immutable, MD5 verified)
`-- silver/
    `-- sales_silver.csv               # Preprocessed, cleaned silver dataset
```

Within the Google Colab execution runtime, the Data Lake is structured as:
```
data_lake/
|-- bronze/
|   `-- sales.csv                      # Raw data copied without modification
`-- silver/
    |-- preprocessed_sales/            # Spark distributed CSV output directory
    `-- sales_silver.csv               # Single consolidated silver CSV file
```

---

## 3. How to Run the Notebook

### In Google Colab:
1. Open [Google Colab](https://colab.research.google.com/).
2. Click **File** → **Upload notebook** and select `submissions/LCS2023047_Mayank_Mishra/Data_Lake_Assignment.ipynb`.
3. Click **Runtime** → **Run all** (`Ctrl + F9`).
4. The notebook automatically:
   - Installs PySpark (`!pip install -q pyspark`).
   - Creates the `data_lake/bronze/` and `data_lake/silver/` hierarchy.
   - Copies or downloads the raw `sales.csv` into `data_lake/bronze/`.
   - Executes all discrepancy checks using Spark operations.
   - Applies the preprocessing transformations.
   - Saves the silver dataset and re-reads it for verification.

### Locally (Jupyter Lab / VS Code):
- Requirements: Python 3.9+, Java 8/11/17/21, and `pyspark`.
- Run all cells sequentially from top to bottom.

---

## 4. Discrepancies Identified via Apache Spark

All counts were programmatically obtained using Apache Spark SQL functions without hardcoding:

| Discrepancy Category | Spark Finding | Affected Count |
|---|---|---|
| **Exact Duplicates** | Identical rows duplicated across all columns | **10 rows** |
| **Reused `order_id`s** | Different transactions sharing the same order ID | **8 pairs (16 rows)** |
| **Missing Values** | Null or whitespace-only blank values | `customer_name`: **10**, `payment_method`: **8**, `city`: **8** |
| **Text Whitespace** | Irregular leading/trailing or multiple consecutive spaces | `customer_name`: **3**, `city`: **3** |
| **Capitalization** | Inconsistent casing in categories and cities | `beauty`, `SPORTS`, `CLOTHING`, `ELECTRONICS`, `DELHI`, `delhi`, `JAIPUR`, etc. |
| **Invalid Quantity** | Quantities `<= 0` or negative numbers (`0`, `-1`, `-2`) | **12 rows** (in Bronze) |
| **Invalid Unit Price** | Unit prices `<= 0` or negative values (`0`, `-500`, `-250`) | **8 rows** (in Bronze) |
| **Invalid Discount** | Discount percentages outside the valid range `[0, 100]` (`-5`, `120`) | **8 rows** (in Bronze) |
| **Non-Standard Dates** | Valid dates using alternative formats (`yyyy/MM/dd`, `MM-dd-yyyy`) | **2 rows** (`2025/07/14`, `09-15-2026`) |
| **Invalid Dates** | Impossible calendar dates or unparseable text strings | **4 rows** (`31/02/2026`, `not-a-date`, `2025-02-30`, `2026-13-05`) |

---

## 5. Preprocessing Summary (Silver Layer)

1. **Exact Duplicate Removal:**
   - Applied `bronze_df.dropDuplicates()`, removing **10 duplicate rows** and reducing the dataset from **1,000 to 990 rows**.
   - Distinct transactions that shared an `order_id` were preserved.

2. **Text Cleaning & Capitalization Standardization:**
   - Stripped leading/trailing spaces and collapsed repeated whitespace using `F.trim(F.regexp_replace(col, "\\s+", " "))`.
   - Standardized `customer_name`, `category`, `city`, `state`, and `order_status` to Title Case via `F.initcap()`.
   - Converted `customer_id` to uppercase via `F.upper()`.
   - Preserved valid acronyms and product names (e.g. `UPI`, `USB-C Hub`).

3. **Missing Value Imputation:**
   - **City Imputation from State:** Imputed missing cities when the state uniquely maps to a single city in the dataset (Karnataka → Bengaluru, Andhra Pradesh → Vijayawada, Delhi → Delhi, Rajasthan → Jaipur, Telangana → Hyderabad, Uttar Pradesh → Lucknow, West Bengal → Kolkata).
   - Ambiguous states (e.g. Maharashtra with Mumbai & Pune) and other missing fields (`customer_name`, `payment_method`) were imputed with `"Unknown"`.

4. **Numerical Validation:**
   - Validated domain rules: `quantity > 0`, `unit_price > 0`, and `0 <= discount_percent <= 100`.
   - Clearly invalid numbers were set to `null` while preserving the surrounding transaction record.

5. **Date Normalization:**
   - Standardized all valid dates into the ISO standard `yyyy-MM-dd` format.
   - Non-standard formats were converted to standard format.
   - Impossible calendar dates (`31/02/2026`, `2025-02-30`, `2026-13-05`) and invalid text (`not-a-date`) were safely set to `null`.

6. **Preservation of Records:**
   - Useful transaction records were kept intact (990 rows preserved).

---

## 6. Verification Results

- **Bronze Layer Row Count:** `1,000`
- **Silver Layer Row Count:** `990`
- **Exact Duplicates in Silver:** `0`
- **Remaining Missing Customer Names / Payment Methods / Cities:** `0` (all imputed or marked `"Unknown"`)
- **Invalid Numerical Values in Silver:** `0`
- **Bronze File Integrity:** Byte-for-byte unmodified (MD5: `2b7d6737635600be42aa639140cd6ea9`).
