# 2021-Tokyo-Olympics-Analysis-Using-Azure

## Project Overview
  This project involves analyzing the "2021 Olympics in Tokyo" dataset by leveraging various Azure services to streamline the data ingestion, transformation, and visualization processes. The workflow begins with ingesting the dataset directly from Kaggle using the Kaggle API within a Databricks notebook. The data is downloaded, unzipped, converted from XLSM to CSV format, and securely stored in Azure Data Lake Storage (ADLS). Subsequently, a data pipeline is created in Azure Synapse, where multiple transformations are applied to cleanse and prepare the data for analysis. The transformed data is then loaded back into ADLS for persistent storage. Azure Synapse’s Serverless SQL pool is utilized to generate insights through SQL queries, and these insights are further visualized in an interactive Power BI dashboard. This project demonstrates the integration of data engineering and analytics tools within Azure to deliver a comprehensive data analysis solution.

## Atchitecture
![image](https://github.com/user-attachments/assets/391996c2-e8b3-4c8e-96a0-d7d147db42c4)

- **Data Source:**
  - XLSM Files from Kaggle
- **Data Storage:**
  - Azure Data Lake Storage Gen2 that stores both raw and transformed data
- **Data Processing**:
  - Databricks:
    - Databricks is Used to convert the files into CSV format
  - Azure Data Factory:
    - ADF is used to apply some transformations to the data
- **Data Warehouse:**
  - Transformed data is loaded into a Synapse SQL Pool
- **Data Exploration:**
  - Azure Storage Explorer is used to interact with Azure Blob Storage and ADLS Gen2.
