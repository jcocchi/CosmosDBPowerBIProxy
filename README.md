# Cosmos DB Power BI Connector Spark Proxy

This project shows how to use Spark as a proxy to access large amounts of data stored in Cosmos DB in Power BI. 

## Setup Solution

These deployment scripts have been written and tested on [Ubuntu 18 LTS](http://releases.ubuntu.com/18.04/) so please ensure you are using a compatible environment. At the end of deployment you will have an Azure Cosmos DB account populated with 10 sample records and a Databricks cluster running and hosting a table that contains those records. This Databricks table is the data source we will connect to PowerBI. 

In order for PowerBI to connect to your table the cluster needs to remain running. Initally the cluster has been set to autoterminate after 300 minutes of inactivity to avoid unneccessary cost while giving you time to connect to PowerBI.

> Note: many of these scripts were modified from the [Streaming at Scale](https://github.com/Azure-Samples/streaming-at-scale) repository and repurposed for this project. If you are interested, please check out that repository as it is a fantastic resource for learning about streaming data in Azure!

### Prerequisties

You will need the following tools to run the project
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest)
  - Install: `sudo apt install azure-cli`
- [jq](https://stedolan.github.io/jq/download/)
  - Install: `sudo apt install jq`
- [python](https://www.python.org/)
  - Install: `sudo apt install python python-pip`
- [databricks-cli](https://github.com/databricks/databricks-cli)
  - Install: `pip install --upgrade databricks-cli`

### Deploy Resources

First log in to your Azure account

    az login

and make sure you are using the correct subsription

    az account list
    az account set --subscription <subscription_name>

once you have selected the subscription you want to use execute the following command to deploy all necessary resources

    ./create-solution.sh -d <solution_name> -l <azure_location>

>Note: The `solution_name` value will be used to create a resource group that will contain all resources created by the script. It will also be used as a prefix for all resources created so, in order to help to avoid name duplicates that will break the script, you may want to generate a name using a unique prefix. **Please also use lowercase letters and numbers only**, since the `solution_name` is also used to create a storage account, which has several constraints on characters usage:
>
>[Storage Naming Conventions and Limits](https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions#storage)

The first time you run the script it will fail because there has to be a manual step to get the Databricks Personal Access Token (PAT). 

Once the script fails the first time, log in to your databricks workspace at `https://<azure_location>.azuredatabricks.net`, hit the person icon in the top right hand corner, select `User Settings`, and `Generate New Token`. This token will only display once so be sure to save it as you will need it again later for connecting to Power BI.

Take the token and update the `DATABRICKS_TOKEN` secret in the Azure Key Vault that was provisoned for you. Once you have saved the PAT token in Key Vault, re-run the `create-solution.sh` script with the same parameters you originally used.

    ./create-solution.sh -d <solution_name> -l <azure_location>

## Visualize Data with Power BI

If you haven't already, download the latest version of [PowerBI Desktop](https://powerbi.microsoft.com/en-us/desktop/). Follow the steps below to connect your new data source to Power BI or follow this [tutorial](https://docs.azuredatabricks.net/bi/power-bi.html) from the Databricks documentation for more information.

## Get the JDBC URL

In your Databricks workspace navigate to the `Clusters` tab and select the `powerbi-proxy-cluster`. 

Expand the `Advanced Options` and select `JDBC/ODBC`.
![jdbc](pictures/jdbc_url.PNG)

To construct the neccessary URL start with `https://<azure_location>.azuredatabricks.net:443/` and append the unique HTTP Path. The final URL for this example would be `https://westus2.azuredatabricks.net:443/sql/protocolv1/o/6349145078251239/1024-225105-routs122`.
![jdbc](pictures/jdbc_url_2.PNG)

## Connect your Data Source in Power BI

Open PowerBI Desktop and create a new report.

Select `Get Data`, search for and select `Spark`, then hit `Connect`.
![get data](pictures/get_data.PNG)

Enter the JDBC URL you formed in the step above in the `Server` box. Ensure `Direct Query` is selected and hit `OK`.
![spark config](pictures/spark_config.PNG)

Enter `token` as the user name to connect to spark and enter the PAT token you generated earlier as the password. Press `Connect`.
![spark creds](pictures/spark_creds.PNG)

Select the `cosmosdata` table from the drop down and press `Load`.
![cosmos table](pictures/cosmos_table.PNG)