# Data Mining Assignment - Data Lake Preprocessing

## Dataset Name
`sales.csv`

## How to Run the Notebook
1. Open the `Data_Lake_Assignment.ipynb` notebook in Google Colab.
2. Upload the `sales.csv` dataset to your Colab session storage (the default `/content/` directory).
3. Run the notebook from top to bottom (Runtime > Run all). 

## Data Lake Structure
The notebook automatically creates the following structure within the Colab environment to simulate a Data Lake:
data_lake/
|-- bronze/
|   `-- sales.csv (Raw dataset)
`-- silver/
    `-- sales_silver.csv (Processed output file)

## Summary of Preprocessing Performed
The PySpark script applies the following basic preprocessing steps to transition data from the Bronze to the Silver layer:
1. **Duplicate Removal:** Exact duplicate rows were identified and removed.
2. **Missing Values Handling:** Handled missing/null values by replacing null numerics with `0` and null strings with `'Unknown'`.
3. **Text Formatting:** Applied trimming to remove extra spaces and converted text fields to Initial Capitalization for consistency.
4. **Data Validation:** Filtered out clearly invalid numerical values such as quantities less than or equal to `0`, negative prices, and discount percentages outside the `0-100` range.
5. **Date Standardization:** Standardized date formats to a consistent format (`yyyy-MM-dd`) using Spark's `to_date` function, gracefully handling invalid dates (like `31/02/2026`) by treating them as `NULL`.
