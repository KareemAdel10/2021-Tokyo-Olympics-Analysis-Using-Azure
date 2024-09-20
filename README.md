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
- **Data Visualization:**
  - A Dashboard is created by PowerBI

## Steps 
1. **Data Ingestion:**
   - In this step, The data is downloaded, unzipped, converted from XLSM to CSV format, and securely stored in Azure Data Lake Storage (ADLS).
   - This is how the XLSM files is downloaded and unzipped into the cluster workspace
    ```
    import os
    
    %pip install kaggle
  
    os.environ['KAGGLE_CONFIG_DIR'] = '<directory>/kaggle.json' #enter your directory
    
    !kaggle datasets download -d arjunprasadsarkhel/2021-olympics-in-tokyo
    
    !unzip 2021-olympics-in-tokyo.zip -d /tmp/2021-olympics-in-tokyo
    ```
  - This is how the unzipped files are converted from XLSM to CSV format using pandas
    ```
    import pandas as pd
    import os
    
    
    
    input_path = "/tmp/2021-olympics-in-tokyo"
    
    xlsm_files = [f for f in os.listdir(input_path)]
    
    output_path = "/tmp/2021-olympics-in-tokyo-csv"
    os.makedirs(output_path, exist_ok=True)
    
    for file_name in xlsm_files:
        xlsm_file_path = os.path.join(input_path, file_name)
        
        df = pd.read_excel(xlsm_file_path, sheet_name=None)  
    
        for sheet_name, data in df.items():
            csv_file_name = f"{os.path.splitext(file_name)[0]}_{sheet_name}.csv"
            csv_file_path = os.path.join(output_path, csv_file_name)
            
            data.to_csv(csv_file_path, index=False)
    ```
  - This is how the files are copied into an ADLS 
    ```
    #first create a secret scope (Input Your Own Variables)
    spark.conf.set(
      "fs.azure.account.key.<ADLS name>.dfs.core.windows.net",
      "<Secret key>"
    )
    #copy the data
    # Set up the ADLS path and container details
    adls_path = "abfss://<container name>@<ADLS name>.dfs.core.windows.net/<directory>"
    
    # Copy the file from DBFS to ADLS
    dbutils.fs.cp("file:/tmp/2021-olympics-in-tokyo-csv", adls_path, recurse=True)
    ```
