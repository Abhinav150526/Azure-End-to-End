
# End-to-End Azure Data Pipeline Project

## Introduction
This document details the creation and implementation of a comprehensive data pipeline on Azure, beginning from account setup to data transformation, analysis, and visualization. The pipeline involves Azure Data Factory, Azure Data Lake Storage Gen2 (ADLS Gen2), Azure Databricks, Azure Synapse Analytics, and various visualization tools.

## Prerequisites
- An active Azure account
- Familiarity with Azure services, Spark, and data pipelines

## Step 1: Azure Account Setup
1. **Create an Azure Account**:
   - Visit the [Azure Portal](https://portal.azure.com/).
   - Sign up for a free Azure account or log in if you already have one.

## Step 2: Resource Group Creation
1. **Create a Resource Group**:
   - In the Azure Portal, search for "Resource Groups."
   - Click on "Create" and provide a name and region for your resource group.
   - Click "Review + Create" and then "Create."

## Step 3: Storage Account Creation
1. **Create a Storage Account**:
   - Navigate to "Storage accounts" in the Azure Portal.
   - Click "Create" and fill in the required details:
     - Select your resource group.
     - Provide a unique storage account name.
     - Choose a region and performance level.
     - Click "Review + Create" and then "Create."

2. **Create Containers and Folders**:
   - After creating the storage account, go to it and select "Containers" under the "Data storage" section.
   - Click "Create container," name it (e.g., `raw-data`), and set the public access level to "Private."
   - Inside the container, create folders (e.g., `raw`, `transformed`) to organize the data.

## Step 4: Azure Data Factory Setup
1. **Create Azure Data Factory**:
   - Search for "Data Factory" in the Azure Portal and click "Create."
   - Select your resource group, name the Data Factory, choose a region, and click "Review + Create."
   - Once created, go to the Data Factory Studio to design your pipelines.

2. **Create a Pipeline in Azure Data Factory**:
   - In the Data Factory Studio, navigate to the "Author" tab and click on "Pipelines."
   - Add activities for data movement and transformation:
     - Use the "Copy Data" activity to move data from GitHub to ADLS Gen2.
     - Use "Data Flow" activities to transform the data.
   - Configure source and destination datasets and set up the pipeline schedule if needed.

## Step 5: Azure Databricks Setup
1. **Create Azure Databricks Workspace**:
   - In the Azure Portal, search for "Databricks" and click "Create."
   - Choose your resource group, name the workspace, select a region, and click "Review + Create."
   - After creation, launch the workspace and create a new cluster.

2. **Create App Registrations for ADLS Integration**:
   - In the Azure Portal, search for "App registrations" and create a new registration.
   - Name the app and register it. Copy the **Application (client) ID** and **Directory (tenant) ID**.
   - Under "Certificates & secrets," create a new client secret and copy the secret value.

## Step 6: Connecting Databricks to ADLS
1. **Assign Role for ADLS Access**:
   - Go to your Storage Account in the Azure Portal.
   - Navigate to "Access control (IAM)" and click "Add role assignment."
   - Assign the "Storage Blob Data Contributor" role to your Databricks app registration.

2. **Configure Databricks to Connect to ADLS**:
   - In your Databricks notebook, use the following Spark code snippet to configure the connection:

     ```python
     spark.conf.set("fs.azure.account.auth.type.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net", "OAuth")
     spark.conf.set("fs.azure.account.oauth.provider.type.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
     spark.conf.set("fs.azure.account.oauth2.client.id.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net", "<APPLICATION_CLIENT_ID>")
     spark.conf.set("fs.azure.account.oauth2.client.secret.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net", "<CLIENT_SECRET>")
     spark.conf.set("fs.azure.account.oauth2.client.endpoint.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net", "https://login.microsoftonline.com/<DIRECTORY_TENANT_ID>/oauth2/token")
     ```

3. **Mount ADLS to Databricks**:
   - Use the following code to mount the ADLS container to Databricks:

     ```python
     dbutils.fs.mount(
         source = "abfss://<CONTAINER_NAME>@<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net/",
         mount_point = "/mnt/<MOUNT_NAME>",
         extra_configs = {"fs.azure.account.key.<STORAGE_ACCOUNT_NAME>.dfs.core.windows.net":dbutils.secrets.get(scope = "<SCOPE_NAME>", key = "<SECRET_NAME>")})
     ```

## Step 7: Data Transformation with Databricks
1. **Perform Data Transformations**:
   - In your Databricks notebook, write Spark code to read data from the mounted ADLS, transform it, and then write the transformed data back to ADLS:

     ```python
     df = spark.read.format("csv").option("header", "true").load("/mnt/<MOUNT_NAME>/raw-data/*.csv")
     transformed_df = df.filter(df['column_name'] > value).groupBy("column_name").agg({"another_column": "sum"})
     transformed_df.write.format("parquet").save("/mnt/<MOUNT_NAME>/transformed-data/")
     ```

## Step 8: Data Analysis with Azure Synapse Analytics
1. **Load Transformed Data into Azure Synapse Analytics**:
   - Set up a linked service in Azure Synapse to connect to your ADLS Gen2.
   - Load the transformed data into a dedicated SQL pool or an on-demand SQL pool for analysis.

2. **Perform Data Analysis**:
   - Use SQL queries within Azure Synapse to perform complex analysis on the transformed data.
   - Visualize the results using built-in visualization tools in Azure Synapse or export the data to Power BI for more advanced visualizations.

## Step 9: Data Visualization
1. **Visualize Data Using Power BI**:
   - Connect Power BI to your Azure Synapse Analytics workspace.
   - Create interactive dashboards and reports based on the analyzed data.

2. **Alternative Visualization Tools**:
   - You can also use other tools like Tableau, Excel, or third-party visualization platforms that support Azure Synapse Analytics integration.

## Conclusion
By following these steps, you have successfully set up an end-to-end data pipeline in Azure. This pipeline facilitates data ingestion, transformation, analysis, and visualization using Azure Data Factory, ADLS Gen2, Azure Databricks, Azure Synapse Analytics, and various visualization tools. This setup provides a scalable and efficient solution for big data processing and analytics on the Azure platform.
