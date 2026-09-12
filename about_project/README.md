# Customer Behavior Analytics — End-to-End Data Analytics Project

## Overview
This project is an end-to-end data analytics pipeline built to analyze customer purchasing behavior, spending patterns, product performance, discounts, subscriptions, and demographics.

The project demonstrates the complete analytics workflow — from raw CSV data preparation in Python to SQL analysis in PostgreSQL, DAX calculations in Power BI, and the development of an interactive Power BI dashboard and analytical report.

## Project Pipeline
```sql
Raw Customer Dataset
        ↓
Python / Jupyter Notebook
        ↓
Data Cleaning & Transformation
        ↓
PostgreSQL Database
        ↓
SQL Analysis & Business Questions
        ↓
Power BI + DAX
        ↓
Interactive Dashboard
        ↓
Insights & Business Report
```

## Dataset
The project uses a Customer Behavior dataset containing customer-level purchase information.

Key fields include:
-  Customer ID
-  Age
-  Gender
-  Category
-  Item Purchased
-  Purchase Amount
-  Review Rating
-  Subscription Status
-  Discount Applied
-  Promo Code Used
-  Shipping Type
-  Frequency of Purchases
-  Previous Purchases
-  Age Group
The dataset was initially provided as a CSV file and prepared using Python before being loaded into PostgreSQL.

## Tools & Technologies
-  Tool	                Purpose
-  Python	              Data loading, cleaning, transformation and EDA
-  Pandas	              Data manipulation and preprocessing
-  NumPy	              Data analysis and numerical operations
-  Jupyter Notebook	    Data analysis pipeline and documentation
-  PostgreSQL	          Database storage and SQL analysis
-  pgAdmin 4	          PostgreSQL database management
-  SQL	                Business analysis and customer segmentation
-  Power BI	            Data visualization and dashboard development
-  DAX	                Measures and analytical calculations

## Project Steps
### 1. Data Loading
The raw CSV dataset was imported into a Jupyter Notebook using Pandas.
```sql
import pandas as pd
import numpy as np
df = pd.read_csv("customer_behavior.csv")
df.info()
df.describe()
```
Initial checks were performed to understand:

Dataset structure
Data types
Numerical statistics
Missing values
Number of records and columns

### 2. Exploratory Data Analysis
EDA was performed to understand customer behavior and identify potential data-quality issues.
Examples include:
```sql
df.info()
df.describe()
df.isnull().sum()
```
The analysis focused on:
-  Missing values
-  Data types
-  Purchase amounts
-  Customer demographics
-  Product categories
-  Review ratings
-  Subscription behavior
-  Discount usage
-  Purchase frequency

### 3. Data Cleaning
-  Handling Missing Values
Missing review ratings were filled using the median review rating within each category.
```sql
df['Review Rating'] = (
    df.groupby('Category')['Review Rating']
      .transform(lambda x: x.fillna(x.median()))
)
```
This approach preserves differences between product categories rather than replacing all missing values with one overall median.

-  Standardizing Column Names
Column names were converted to lowercase and spaces were replaced with underscores.

df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(' ', '_')

The purchase amount column was also renamed:
```sql
df = df.rename(
    columns={'purchase_amount_(usd)': 'purchase_amount'}
)
```

### 4. Feature Engineering
Age Group
A new age_group column was created using age quartiles.

labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
```sql
df['age_group'] = pd.qcut(
    df['age'],
    q=4,
    labels=labels
)
```
This allowed customer revenue and sales to be analyzed across different age segments.

Purchase Frequency
A numerical purchase-frequency field was created by mapping frequency descriptions to the approximate number of days between purchases.
```sql
frequency_mapping = {
    'Fortnightly': 14,
    'Weekly': 7,
    'Monthly': 30,
    'Quarterly': 90,
    'Bi-Weekly': 14,
    'Annually': 365,
    'Every 3 Months': 90
}

df['purchase_frequency_days'] = (
    df['frequency_of_purchases'].map(frequency_mapping)
)
```
- Removing Redundant Data
The relationship between discount_applied and promo_code_used was checked:
```sql
(df['discount_applied'] == df['promo_code_used']).all()

Since both columns contained equivalent information, promo_code_used was removed:

df = df.drop('promo_code_used', axis=1)
```

### 5. Loading Data into PostgreSQL
After cleaning and transformation, the final DataFrame was loaded into PostgreSQL.

Required packages:
```sql
pip install psycopg2-binary sqlalchemy

Connection to PostgreSQL was established using SQLAlchemy:

from sqlalchemy import create_engine

username = "postgres"
password = "YOUR_PASSWORD"
host = "localhost"
port = "5432"
database = "customer_behavior"

engine = create_engine(
    f"postgresql+psycopg2://{username}:{password}@{host}:{port}/{database}"
)

table_name = "customer_behavior"

df.to_sql(
    table_name,
    engine,
    if_exists="replace",
    index=False
)

print(
    f"Data successfully loaded into table '{table_name}' "
    f"in database '{database}'."
)
```

```sql
Database Flow
Jupyter Notebook
      ↓
Clean Pandas DataFrame
      ↓
SQLAlchemy
      ↓
PostgreSQL
      ↓
customer_behavior table
      ↓
SQL Analysis in pgAdmin 4
```

### 6. SQL Analysis
Several business questions were answered using PostgreSQL.

## Key SQL Questions
### 1. Revenue by gender
Compared total revenue generated by male and female customers.

### 2. Discount users with high spending
Identified customers who used discounts but still spent at or above the overall average purchase amount using a subquery.

### 3. Top-rated products
Found the top five products based on average review rating.

### 4. Shipping analysis
Compared average purchase amounts between Standard and Express shipping.

### 5. Subscription analysis
Compared subscribers and non-subscribers based on:
Number of customers
Average spend
Total revenue

### 6. Discount analysis
Identified the five products with the highest percentage of discounted purchases.

### 7. Customer segmentation
Segmented customers into:
New
Returning
Loyal
based on previous purchases.

### 8. Top products by category
Used a window function with ROW_NUMBER() to identify the top three products within each category.

### 9. Repeat buyers and subscriptions
Analyzed whether customers with more than five previous purchases were more likely to subscribe.

### 10. Revenue by age group
Compared total revenue generated by different age groups.

### SQL Techniques Used
```sql
GROUP BY
ORDER BY
WHERE
CASE WHEN
Subqueries
CTEs
Aggregate Functions
Window Functions
ROW_NUMBER()
CAST / ::numeric
ROUND()
```

### 7. Power BI & DAX
The cleaned PostgreSQL data was connected to Power BI for visualization and further analysis.

DAX was used to create analytical measures such as:
-        Total Customers
-        Total Revenue
-        Average Purchase Amount
-        Average Review Rating
-        Customer counts by subscription status
-        Revenue by category
-        Sales by category
-        Revenue by age group
-        Sales by age group

Example DAX measures can be maintained in a dedicated DAX_Measures file.

### Total Customers =
DISTINCTCOUNT(customer_behavior[customer_id])

### Total Revenue =
SUM(customer_behavior[purchase_amount])

### Average Purchase Amount =
AVERAGE(customer_behavior[purchase_amount])

### Average Review Rating =
AVERAGE(customer_behavior[review_rating])

### 8. Power BI Dashboard
The final dashboard was designed to provide a concise view of customer behavior and business performance.

### Dashboard KPIs
The dashboard highlights:

3.9K Customers
$59.76 Average Purchase Amount
3.75 Average Review Rating
Dashboard Visualizations

The report includes:
-        Subscription Status donut chart
-        Revenue by Category
-        Sales by Category
-        Revenue by Age Group
-        Sales by Age Group
-        Customer filtering by:
-        Subscription Status
-        Gender
-        Category
-        Shipping Type
-        Dashboard Design
The dashboard provides interactive filtering so users can drill down into customer behavior based on different dimensions.

                 CUSTOMER BEHAVIOR DASHBOARD
 ┌───────────────────────────────────────────────────┐
 │ Customers │ Avg Purchase │ Avg Review Rating      │
 ├───────────────────────────────────────────────────┤
 │ Subscription │ Revenue by Category │ Sales by Cat.│
 ├───────────────────────────────────────────────────┤
 │ Revenue by Age Group │ Sales by Age Group         │
 └───────────────────────────────────────────────────┘


### 9. Business Insights
The analysis can be used to answer questions such as:
-        Which customer groups generate the most revenue?
-        Do subscribers spend more than non-subscribers?
-        Which product categories generate the highest revenue?
-        Which products receive the highest ratings?
-        Which products are most frequently purchased with discounts?
-        How does spending vary across age groups?
-        Which shipping methods are associated with higher average purchases?
-        Are repeat customers more likely to subscribe?
-        Which products are the top sellers within each category?
These insights provide a foundation for decisions around customer retention, subscription strategy, promotions, product management, and marketing.

### Results
The completed project delivers a full analytics workflow:
-        Data Engineering
-        Imported raw CSV data
-        Investigated data quality
-        Handled missing values
-        Standardized column names
-        Removed redundant columns
-        Created analytical features
-        Data Analysis
-        Performed exploratory data analysis in Python
-        Created customer segments
-        Analyzed purchasing frequency
-        Investigated customer and product behavior
-        SQL Analytics
-        Loaded cleaned data into PostgreSQL
-        Created business-focused SQL queries
-        Used aggregations, subqueries, CTEs and window functions
-        Business Intelligence
-        Connected PostgreSQL data to Power BI
-        Created DAX measures
-        Built an interactive dashboard
-        Presented customer and sales KPIs
-        Reporting
-        Converted analytical findings into business-oriented insights and recommendations.

### End-to-End Architecture
                    ┌──────────────────┐
                    │   CSV Dataset     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Python / Pandas   │
                    │      + EDA        │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Data Cleaning &   │
                    │ Feature Engineering│
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   PostgreSQL      │
                    │     pgAdmin 4     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   SQL Analysis    │
                    │ Business Questions│
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │     Power BI      │
                    │       DAX         │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Interactive       │
                    │ Dashboard         │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Business Report   │
                    │ & Insights        │
                    └──────────────────┘

Key Skills Demonstrated
Python • Pandas • NumPy • Exploratory Data Analysis • Data Cleaning • Feature Engineering • PostgreSQL • SQL • pgAdmin 4 • CTEs • Subqueries • Window Functions • Power BI • DAX • Data Visualization • Dashboard Development • Business Analysis • Reporting

Conclusion
This project demonstrates a complete end-to-end data analytics workflow, connecting data preparation, database management, SQL analysis, business intelligence, visualization, and reporting into a single pipeline.

It showcases the ability to take a raw dataset and transform it into actionable business insights using Python, PostgreSQL and Power BI.
