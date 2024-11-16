# **E-Commerce Data Processing Pipeline**

This repository contains an end-to-end data pipeline project that processes e-commerce data (users, buyers, sellers, and countries) using Azure Data Lake Storage, Azure Data Factory, and Databricks. The project implements a scalable Medallion architecture with Bronze, Silver, and Gold layers for optimized data processing and analytics.

---

## **Project Overview**

This project is designed to:
- Process raw CSV files (users, buyers, sellers, and countries).
- Convert data to efficient Parquet format for storage.
- Implement a structured Medallion architecture using Delta tables.
- Automate data processing workflows with triggers for seamless execution.

---

## **Architecture Flow**

![ECommerce ETL Pipeline Architecture](/images/ECommerce_ETL_Data_Pipeline_Architecture_Diagram.jpg)

1. **Raw Data Upload**  
   - Four CSV files: `users`, `buyers`, `sellers`, and `countries` are manually uploaded to **Azure Data Lake Storage (ADLS)** under the `landing-zone-1` folder.
  
**Azure Data Lake Storage:**

![ECommerce ETL Pipeline Architecture](/images/adls.png)

2. **Azure Data Factory Pipelines**  
   - **Users Pipeline**:  
     Triggered automatically whenever new files are uploaded to `landing-zone-1`. It processes the `users` CSV (split into 10 chunks) and converts it to Parquet format. The output is stored in `landing-zone-2`.

![ECommerce ETL Pipeline Architecture](/images/pipeline_1_user_data.png)

   - **Buyers, Sellers, Countries Pipeline**:  
     Runs weekly using a schedule trigger to process these three CSV files and convert them to Parquet format. The output is stored in `landing-zone-2`.

![ECommerce ETL Pipeline Architecture](/images/pipeline_2_other_data.png)

3. **Data Processing in Databricks**  
   - **Bronze Layer**:  
     Reads Parquet files from `landing-zone-2` and stores them as Delta tables.
   - **Silver Layer**:  
     Applies transformations to the Delta tables from the Bronze layer, refining and cleaning the data. The refined data is stored as Delta tables in the Silver layer.
   - **Gold Layer**:  
     Combines all Silver Delta tables into a single comprehensive Delta table (`ecom_one_big_table`) for business analytics using join operations on the `country` column.

4. **Automation with Triggers**  
   - A third pipeline in Azure Data Factory triggers the execution of Databricks notebooks whenever new files are added to `landing-zone-2`.
  
![ECommerce ETL Pipeline Architecture](/images/pipeline_3_databricks.png)

---

## **Folder Structure**

### Azure Data Lake Storage (ADLS)
- **`landing-zone-1`**: Stores raw CSV files uploaded manually.

  ![ECommerce ETL Pipeline Architecture](/images/adls_landing_zone_1.png)

  ![ECommerce ETL Pipeline Architecture](/images/csv_data_users_raw.png)


- **`landing-zone-2`**: Stores Parquet files processed by Azure Data Factory.

  ![ECommerce ETL Pipeline Architecture](/images/adls_landing_zone_2.png)

  ![ECommerce ETL Pipeline Architecture](/images/parquet_data_users_raw.png)


---

## **Implementation Details**

### Azure Data Factory Pipelines
- **Pipeline 1: Users Data Processing**  
  - **Trigger**: Storage event trigger (Blob Created).  
  - **Action**: Converts `users` CSV chunks to Parquet and stores them in `landing-zone-2`.  

- **Pipeline 2: Buyers, Sellers, and Countries Data Processing**  
  - **Trigger**: Weekly schedule trigger.  
  - **Action**: Converts `buyers`, `sellers`, and `countries` CSV files to Parquet and stores them in `landing-zone-2`.  

- **Pipeline 3: Databricks Processing**  
  - **Trigger**: Storage event trigger (Blob Created in `landing-zone-2`).  
  - **Action**: Executes Databricks notebooks for the Bronze, Silver, and Gold layers.

**Triggers:**

![ECommerce ETL Pipeline Architecture](/images/trigger.png)

**Pipeline Runs History:**

![ECommerce ETL Pipeline Architecture](/images/pipeline_runs_history.png)

---

### Databricks Medallion Architecture
1. **Bronze Layer**  
   - Reads Parquet files from ADLS using PySpark:  
     ```python
     spark.read.format("parquet").load("adls_path/landing-zone-2/file_name.parquet")
     ```  
   - Stores data in Delta table format:  
     ```python
     df.write.format("delta").save("/mnt/delta/tables/bronze/table_name")
     ```

2. **Silver Layer**  
   - Reads Delta tables from the Bronze layer.  
   - Performs transformations to refine the data.  
   - Saves the refined data as Delta tables:  
     ```python
     refined_df.write.format("delta").mode("overwrite").save("/mnt/delta/tables/silver/table_name")
     ```

3. **Gold Layer**  
   - Combines all Silver Delta tables into a single Delta table:  
     ```python
     comprehensive_user_table = silver_users \
         .join(silver_countries, ["country"], "outer") \
         .join(silver_buyers, ["country"], "outer") \
         .join(silver_sellers, ["country"], "outer")
     ```  
   - Stores the final table:  
     ```python
     comprehensive_user_table.write.format("delta").mode("overwrite").option("mergeSchema", "true").save("/mnt/delta/tables/gold/ecom_one_big_table")
     ```

**Gold Layer Execution:**

![ECommerce ETL Pipeline Architecture](/images/pipeline_3_result_databricks.png)


---

## **Technologies Used**
- **Azure Data Lake Storage (ADLS)**: Storage for raw CSV and processed Parquet files.
- **Azure Data Factory (ADF)**: Data ingestion and transformation pipelines.
- **Databricks**: Medallion architecture implementation using PySpark and Delta Lake.
- **Parquet**: Optimized file format for faster data processing.
- **Delta Lake**: Provides ACID transactions and scalable data processing.

![ECommerce ETL Pipeline Architecture](/images/resource_group.png)

---

## **How It Works**
1. Manually upload raw data (CSV files) to the `landing-zone-1` folder in ADLS.
2. Pipelines in Azure Data Factory:
   - Process the raw data and convert it to Parquet.
   - Store the Parquet files in the `landing-zone-2` folder.
3. Databricks processes Parquet files:
   - Bronze: Reads Parquet and stores as Delta tables.
   - Silver: Refines Delta tables with transformations.
   - Gold: Combines refined data into a single analytics-ready table.
4. Automated triggers ensure seamless execution whenever new files are uploaded.

---

## **Future Improvements**
- Fully automate raw data upload using Azure Logic Apps or Python scripts.
- Add monitoring and logging for ADF pipelines and Databricks jobs.
- Implement data quality checks in the Silver layer.

---
