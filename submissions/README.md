# Data Mining Assignment: Apache Spark Data Lake Preprocessing

**Student Roll Number:** LCB2023039  
**Student Name:** Prashik Humane  
**Dataset Name:** sales.csv  
**Environment:** Google Colab + PySpark  

---

## 1. Data Lake Structure

```
data_lake/
├── bronze/
│   └── sales.csv             # Raw, unmodified dataset
└── silver/
    └── preprocessed_sales/   # Preprocessed Silver layer dataset
```

Submissions directory layout:
```
submissions/LCB2023039_Prashik_Humane/
├── Data_Lake_Assignment.ipynb
├── README.md
└── silver/
    └── sales_silver.csv
```

---

## 2. Preprocessing Operations Summary

The raw `sales.csv` dataset in the **Bronze layer** contained 1,000 records. Using PySpark operations, the following preprocessing transformations were applied to produce the clean **Silver layer**:

1. **Exact Duplicate Removal**: Identified and dropped exact duplicate rows.
2. **Missing Value Imputation**: Imputed missing values in text fields (`customer_name`, `payment_method`, `city`) with `'Unknown'`.
3. **Text Standardization**: Trimmed extra leading/trailing whitespace and standardized text fields (`customer_name`, `product`, `category`, `payment_method`, `city`, `state`, `order_status`) to Title Case (`initcap`).
4. **Numerical Validation**:
   - Fixed negative `discount_percent` values (e.g. `-5` converted to `5`).
   - Capped `discount_percent` values exceeding 100% at `100`.
   - Filtered out clearly corrupted records where `quantity <= 0` or `unit_price <= 0`.
5. **Date Standardization & Filtering**:
   - Converted valid dates in various formats (`yyyy-MM-dd`, `dd/MM/yyyy`, `yyyy/MM/dd`, `MM-dd-yyyy`) to a single unified format (`YYYY-MM-DD`).
   - Filtered out unparseable/invalid calendar dates (e.g., `31/02/2026`, `not-a-date`).

---

## 3. Verification & Row Counts

- **Bronze Layer (Before Preprocessing):** `1,000` rows
- **Silver Layer (After Preprocessing):** `967` rows
- **Records Filtered / Cleaned:** `33` rows

---

## 4. How to Run the Notebook in Google Colab

1. Open [Google Colab](https://colab.research.google.com/).
2. Upload `Data_Lake_Assignment.ipynb`.
3. Ensure `dataset/sales.csv` (or `sales.csv`) is present in the Colab working environment.
4. Execute cells sequentially from top to bottom (**Runtime > Run all**).
5. The notebook will automatically build `data_lake/bronze/` and `data_lake/silver/` layers and produce `silver/sales_silver.csv`.
