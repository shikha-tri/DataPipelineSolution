# DataPipelineSolution


## **1. Introduction**
This document provides a comprehensive solution for handling and processing the given dataset. It includes database design, data pipeline implementation, SQL queries, and PySpark code for efficient processing.

---
## **2. ER Diagram**
The Entity-Relationship diagram represents the database schema and relationships between tables:


https://github.com/shikha-tri/DataPipelineSolution/blob/main/ERDiagram.png

---
## **3. Data Flow Diagram**
The Data Flow Diagram (DFD) represents how data moves through the system:

**DFD Levels:**
1. **Level 0:** Shows an overview of the data flow.
2. **Level 1:** Details interactions between data sources and transformations.
3. **Level 2:** Shows specific operations on data tables.


---
## **4. Database Schema**
**Tables & Relationships:**
- **fact_transactions**: Contains transactional sales data.
- **hier_prod**: Stores product details.
- **hier_possite**: Stores site details.
- **hier_clnd**: Stores calendar details.

**SQL Table Creation Scripts:**
```sql
CREATE TABLE fact_transactions (
    trans_id INT PRIMARY KEY,
    prod_id INT,
    pos_site_id INT,
    fsclwk_id INT,
    sales_units INT,
    sales_dollars DECIMAL(10,2),
    discount_dollars DECIMAL(10,2),
    FOREIGN KEY (prod_id) REFERENCES hier_prod(prod_id),
    FOREIGN KEY (pos_site_id) REFERENCES hier_possite(pos_site_id),
    FOREIGN KEY (fsclwk_id) REFERENCES hier_clnd(fsclwk_id)
);

CREATE TABLE hier_prod (
    prod_id INT PRIMARY KEY,
    prod_name VARCHAR(255)
);

CREATE TABLE hier_possite (
    pos_site_id INT PRIMARY KEY,
    site_name VARCHAR(255)
);

CREATE TABLE hier_clnd (
    fsclwk_id INT PRIMARY KEY,
    week_start_date DATE,
    week_end_date DATE
);
```

---
## **5. Data Processing Pipeline (PySpark Implementation)**
### **Step 1: Read Data from Source**
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("DataPipeline").getOrCreate()

fact_df = spark.read.csv("fact_transactions.csv", header=True, inferSchema=True)
prod_df = spark.read.csv("hier_prod.csv", header=True, inferSchema=True)
possite_df = spark.read.csv("hier_possite.csv", header=True, inferSchema=True)
clnd_df = spark.read.csv("hier_clnd.csv", header=True, inferSchema=True)
```

### **Step 2: Data Cleaning and Transformation**
```python
from pyspark.sql.functions import col

fact_df = fact_df.withColumn("sales_dollars", col("sales_dollars").cast("double"))
```

### **Step 3: Join Tables**
```python
final_df = fact_df.join(prod_df, "prod_id").join(possite_df, "pos_site_id").join(clnd_df, "fsclwk_id")
```

### **Step 4: Save Processed Data**
```python
final_df.write.parquet("processed_data.parquet")
```

---
## **6. Queries for Analysis**
### **Total Sales per Product**
```sql
SELECT p.prod_name, SUM(f.sales_dollars) AS total_sales 
FROM fact_transactions f
JOIN hier_prod p ON f.prod_id = p.prod_id
GROUP BY p.prod_name;
```

### **Total Discount Given per Site**
```sql
SELECT s.site_name, SUM(f.discount_dollars) AS total_discount 
FROM fact_transactions f
JOIN hier_possite s ON f.pos_site_id = s.pos_site_id
GROUP BY s.site_name;
```

---
## **7. Deployment Instructions**
- Install **PySpark**, **SQL database**, and required dependencies.
- Execute the provided scripts.
- Verify results using SQL queries.

---
## **8. Conclusion**
This solution efficiently handles large datasets using SQL and PySpark for seamless ETL and analytics. It ensures data integrity and supports scalable processing.

