# Sales-deals-Azure-End2End-Project
This Project helps in understanding how various azure services &amp; tools are used to complete end-end data project.

# Dataset

The sales deals data set is located on-premise(from maven analytics), dataset has 5 csv files named accounts, products, sales teams, and sales opportunities, about a B2B sales pipeline data from a fictitious company that sells computers hardware.

# In this project we will perform the following tasks

1. Move on-premises data to the cloud using Azure Data Factory with a Self-Hosted Integration Runtime(IR).
2. Integrate data factory with Git for version control.
3. Use Azure Data Lake Storage Gen2 as a central data repository.
4. Improve reliability with Logic Apps for email alerts and Azure Monitor for detailed tracking.
5. Perform data cleaning and transformations using Azure Databricks.
6. Implement security best practices with Azure Key Vault + Databricks Secret Scope to protect sensitive secrets.
7. Analyze data with Azure Synapse Analytics using SQL.
8. Build interactive dashboards in Power BI for business insights.

   # Project detailed steps

1)	Create azure resource group
2)	Create azure storage account (ADLS Gen 2)
3)	Create azure data factory (ADF)
4)	Create containers: raw, transformed data.
5)	Link GitHub repository to azure data factory for version control.
6)	Create self-hosted integration runtime (IR).
7)	Run the downloaded IR file.
8)	Create Linked services (to connect on-prem to cloud, use filesystem) and provide the path to on-premise data source, test the connection.
9)	Create a pipeline in ADF and configure source (on-premise path), use wild file path(*.csv) and sink (raw data in storage account) for the pipeline.
10)	Create Logic apps, click on add an action
i)	add when https request is received and write the following expression in the body section: 
{
  "type": "object",
  "properties": {
    "pipelinename": {
      "type": "string"
    },
    "status": {
      "type": "string"
    },
    "pipelineid": {
      "type": "string"
    },
    "time": {
      "type": "string"
    }
  }
}

ii) click on add action, add send email, select parameters body and subject and write the below expression to get email with the following details:
     Pipeline Details:
Name: 
Status
ID:
time:

11)	Create Azure Monitor and under metrics add the failure pipeline and success pipeline (Count) monitor metrics.
12)	Now in pipeline add activity web and provide the https URL from the logic apps and configure body with the following for both success and failed web activities: 
             {
    "pipelinename": "@{pipeline().Pipeline}",
    "status":  "Failed",
    "runid": @{pipeline().RunId},
    "time": "@{utcNow()}"
}

13)	Validate the pipeline and run by manually triggering or scheduling.
14)	Now the data from the on-premise should be moved to raw container in storage.
15)	Create Azure Key vault -> under objects –> click on secret and create secret, use storage account (Adls)security –> access key –> key 1 and click on create secret.
16)	Create Databricks workspace.
17)	Create new notebook in workspace and create secret scope for connection between storage account and databricks. Duplicate the notebook link and edit the link as: https://adb-1025563089158539.19.azuredatabricks.net/?o=1025563089158539#secrets/createScope
18)	Click on properties in key vault and copy vault URI, resource id and paste in vault section DNS name, resource id.
19)	Create compute first since it takes time to start.
20)	Now mount the storage account to databricks using the following code:
  dbutils.fs.mount(
    source = "wasbs://raw-data@azureprojectspractice.blob.core.windows.net",
    mount_point = "/mnt/raw-data",
    extra_configs = {"fs.azure.sas.raw-data.azureprojectspractice.blob.core.windows.net":
                     dbutils.secrets.get(scope = "databricks scope1", key = "azuresecret1")})

21)	Verify the mount using:  dbutils.fs.ls("/mnt/raw-data")
22)	Now repeat the same to mount transformed data container from storage account, copy the code and rename raw-data to transformed-data: 
   dbutils.fs.mount(
    source = "wasbs://transformed-data@azureprojectspractice.blob.core.windows.net",
    mount_point = "/mnt/transformed-data",
    extra_configs = {"fs.azure.sas.transformed-data.azureprojectspractice.blob.core.windows.net":
                     dbutils.secrets.get(scope = "databricks scope1", key = "azuresecret1")})

23)	Check the notebook for more transformation details.
24)	Now write back the cleaned data to transformed-data store in ADLS.
25)	Terminate the cluster to avoid charges.
26)	Now create Synapse analytics workspace.
27)	Create SQL database and then create views (virtual queries, data is not stored)
28)	Create queries that we need for reporting.
29)	Open Power BI and connect it to ADLS gen 2.
30)	Provide the link to transformed-data container URL.
31)	Change the blob in url to dfs.
32)	Authenticate -> transform data -> select files and start building visualizations.

