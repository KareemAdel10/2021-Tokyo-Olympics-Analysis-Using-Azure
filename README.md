# 2021-Tokyo-Olympics-Analysis-Using-Azure

## Project Overview
  This project involves analyzing the "2021 Olympics in Tokyo" dataset by leveraging various Azure services to streamline the data ingestion, transformation, and visualization processes. The workflow begins with ingesting the dataset directly from Kaggle using the Kaggle API within a Databricks notebook. The data is downloaded, unzipped, converted from XLSM to CSV format, and securely stored in Azure Data Lake Storage (ADLS). Subsequently, a data pipeline is created in Azure Synapse, where multiple transformations are applied to cleanse and prepare the data for analysis. The transformed data is then loaded back into ADLS for persistent storage. Azure Synapse’s Serverless SQL pool is utilized to generate insights through SQL queries, and these insights are further visualized in an interactive Power BI dashboard. This project demonstrates the integration of data engineering and analytics tools within Azure to deliver a comprehensive data analysis solution.

## Architecture
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
### 1. **Create An ADLS :**
   - In this step, an Azure Data Lake Storage Gen2 is created to store both  raw data and transformed data.
     
### 2. **Data Ingestion:**
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
    
### 3. **Data Transformation:**
  - In this step, a data flow is created in Azure Synapse that performs multiple transformations including Joins, Type casts, Select, Derived Columns, and file renaming.
    ![image](https://github.com/user-attachments/assets/338145d8-fc9a-4769-9733-9099f5b398db)
    
### **4. Load Transformed Data:**
  - In this step, a data pipeline is created including the data flow previously created to load the transformed data into the ADLS account
    ![image](https://github.com/user-attachments/assets/d9accf6e-d4c5-4eab-824b-cf2bac6bf66b)
    
### **5. create a Serverless SQL pool:**
  - In this step, Serverless SQL pool is created by creating five tables each one is pointing to one of the transformed csv files
    ![image](https://github.com/user-attachments/assets/0ea041d9-75a4-4703-a00b-b1a92b4cf778)
    
### 6. **Creating a SQL Script over the SQL Pool:**
  - In this step, generate some insights by creating a SQL Script over the SQL Pool.
    ```
    --1. Distribution of Male and Female Athletes Across Disciplines
    SELECT Discipline, 
           SUM(Female) AS Female_Athletes, 
           SUM(Male) AS Male_Athletes, 
           SUM(Total) AS Total_Athletes 
    FROM EntriesGender
    GROUP BY Discipline
    ORDER BY Total_Athletes DESC;
    
    
    ----
    --2. Top 5 Countries that earned Total Medal Count
    SELECT
        TOP 5
        [Team/NOC],
        Gold + Silver + Bronze as Total_medals
    FROM 
        Medals
    
    ----
    --3. Coaches by Country and Discipline
    SELECT 
        NOC, 
        Discipline,
        count(*) as Total_coaches
    FROM 
        Coaches
    GROUP BY    
        NOC, 
        Discipline
    
    ----
    --4. Countries with the Highest Number of Disciplines Coached
    SELECT
        NOC,
        count(DISTINCT Discipline) as Total_Coached_Disciplines
    FROM
        Coaches
    GROUP BY
        NOC
    ORDER by
        Total_Coached_Disciplines DESC
    
    ----
    --5. Relationship Between Athletes and Coaches by Discipline
    SELECT a.Discipline, 
           COUNT(DISTINCT a.Name) AS Number_of_Athletes, 
           COUNT(DISTINCT c.Name) AS Number_of_Coaches, 
           COUNT(DISTINCT a.Name)/COUNT(DISTINCT c.Name) AS Athlete_to_Coach_Ratio
    FROM Atheletes a
    JOIN Coaches c ON a.Discipline = c.Discipline AND a.NOC = c.NOC
    GROUP BY a.Discipline
    ORDER BY Athlete_to_Coach_Ratio DESC;
    
    ----
    --6. Countries with Equal Representation in Male and Female Entries
    SELECT Discipline
    From EntriesGender
    Where Female = Male
    
    ----
    --7. Total members in each Team
    SELECT NOC,
          count(distinct name) + count(distinct coach_name) as Team_capacity
    FROM Atheletes
    GROUP BY NOC
    ORDER BY Team_capacity desc
    ```
### **7. Create A PowerBI visualization:**
  - In this step, a dashboard is created using PowerBI
  ![image](https://github.com/user-attachments/assets/bb1a585c-451c-4718-a18a-a8bd672cfb8a)

## Contribution
Contributions to this project are welcome. If you have any ideas for improvement, feel free to fork the repository, make your changes, and submit a pull request. Please ensure your changes align with the overall project structure and objectives.

## Acknowledgements
I would like to thank the following:
- arjunprasadsarkhel on Kaggle for providing the "2021 Olympics in Tokyo" dataset.
- Azure for offering powerful tools and services that enabled the seamless data engineering and analytics processes.

## Contact
For any questions, feedback, or further information, feel free to reach out:
- **Email:** kareema9001@gmail.com
- **LinkedIn:** https://www.linkedin.com/in/kareem-adel-b76ab9201/

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
