# Fetch Take-Home Project***

### Overview
This project involves cleaning, analyzing, and drawing insights from three datasets: users, products, and transactions. The analysis aims to identify data quality issues, highlight trends, and generate actionable insights for business stakeholders.

### Repository Structure
The repository is organized as follows:

#### Datasets
1. Raw Datasets
Users Dataset: Contains user demographic information and join dates.
Products Dataset: Includes details like product barcodes, categories, subcategories, manufacturers, and brands.
Transactions Dataset: Records of all transactions, including purchase dates, store names, user IDs, quantities, and sales amounts.
2. Cleaned Datasets
Cleaned datasets are stored in the data/clean/ directory and include pre-processed versions of the raw data with resolved issues such as:
Removing mismatched USER_ID and BARCODE.
Handling missing or invalid values in MANUFACTURER and BRAND.

#### Key Files
eda.ipynb: Exploratory data analysis (EDA) notebook, documenting the data cleaning process, key findings, and insights.
Fetch Take Home Query Results.md: Results of SQL queries answering specific business-related questions.
Fetch Take Home email.md: Draft email communicating data issues, insights, and requests for action to stakeholders.

#### Highlights
Key Data Quality Issues
1. Mismatch in USER_ID and BARCODE across the three datasets
2. Products with missing BARCODE
3. Products with missing values or invalid terms in MANUFACTURER and BRAND

Minor Data Quality Issues
1. Incorrect data type for certain columns
2. Categorical columns (eg STATE, LANGUAGE, GENDER) with missing values
3. Duplicates across datasets

#### Outstanding Questions 
1. Relationship between PLACEHOLDER MANUFACTURER and PRIVATE LABEL
2. Transactions with missing values or invalid terms in FINAL_QUANTITY & FINAL_SALE 

#### Interesting Trend
User Activity by Day of the Week: Analysis reveals that Saturdays are the most active day for user transactions, while activity slows during weekdays. This insight can be used to optimize marketing campaigns and inventory planning.
