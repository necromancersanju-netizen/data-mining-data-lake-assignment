# Data Mining - Spark Data Lake Assignment

## Dataset
**Name**: `sales.csv`

## How to Run the Notebook
1. Open Google Colab (or any Jupyter environment).
2. Upload the `Data_Lake_Assignment.ipynb` notebook.
3. Run the notebook from top to bottom. It will automatically set up the Bronze and Silver folders and download the raw data required for preprocessing.

## Data Lake Structure
- **Bronze Layer**: Contains the raw, untouched dataset (`bronze/sales.csv`).
- **Silver Layer**: Contains the preprocessed dataset (`silver/sales_silver.csv`), cleansed and standardized using Apache Spark.

## Preprocessing Performed (Silver Layer)
1. **Duplicate Removal**: Dropped exact duplicate rows.
2. **Missing Values**: Dropped records where the core identifier (`order_id`) was missing.
3. **Text Standardization**: Trimmed leading/trailing whitespaces from all text fields and standardized capitalization using `initcap` (e.g., for `city`, `state`, `category`, `product`).
4. **Validation**: Removed invalid records where `quantity` $\le 0$ or `unit_price` $< 0$. Bounded `discount_percent` to a valid range of $0$ to $100$.
5. **Date Formatting**: Converted `order_date` to a standard timestamp and dropped records with clearly malformed/invalid dates (e.g., February 31st).
