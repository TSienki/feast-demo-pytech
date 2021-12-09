Repo contains code presented during Pytech Summit 2021. It's an example of using [feast](https://github.com/feast-dev/feast) with [azure provider](https://github.com/Azure/feast-azure).


# Walkthrough

## Creating and preparing virtual environment
```
python -m venv .venv
source .venv/bin/activate
pip install pandas feast-azure-provider azureml-core
``` 
In case of lack of odbc please follow [this tutorial.](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver15)

## Generating data
It creates artificial time series which were used during the live demo.
```
cd data_utils
python generator.py 
```

## Creating cloud environment
Cloud enviroment can be created based on template.json. It can be done: 
* [Using CLI](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-tutorial-create-first-template?tabs=azure-cli)
* [Using azure portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/quickstart-create-templates-use-the-portal) 

One of required PrincipleID which can be found in Cloud Shell using command:

```
az ad signed-in-user show --query objectId -o tsv
```
For more information please go to [feast azure provider repo.](https://github.com/Azure/feast-azure/tree/main/provider)

Creating cloud environment lasts about 20-30 minutes.

## Copying generated data to data lake
Generated data has to be copied to data laked linked to synapse resource.

## Preparing azure ml config 
Firstly it's needed to download config.json from azure machine learning resource. Next replace CONFIG_PATH variable value in *data_utils/load_data.py* and *utils/prepare_feast.py* (next time it's be done through some config file :D) with this path.


## Moving data from data lake to synapse 
It can be done by script:
```
cd data_utils
python generator.py 
``` 
Or explicitly on synapse with queries. 

Creating payments table:
```
CREATE TABLE dbo.payments (
    event_id INT,
    player_id NVARCHAR(5),
    ts DATETIME2,
    amount FLOAT,
    transactions INT
)
``` 
Creating payments table:
```
CREATE TABLE dbo.payments (
    event_id INT,
    player_id NVARCHAR(5),
    ts DATETIME2,
    amount FLOAT,
    transactions INT
)
``` 
Creating stats table:
```
CREATE TABLE dbo.stats (
    event_id INT,
    player_id NVARCHAR(5),
    ts DATETIME2,
    win_loss_ratio FLOAT,
    games_played INT,
    time_in_game FLOAT
)
``` 
Moving payments data ({URL} should be replaced by data lake URL linked to synapse):
```
COPY INTO dbo.payments
FROM '{URL}.csv'
WITH
(
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 0
    ,FIRSTROW = 2
)
``` 

Moving stats data ({URL} should be replaced by data lake URL linked to synapse):
```
COPY INTO dbo.stats
FROM '{URL}.csv'
WITH
(
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 0
    ,FIRSTROW = 2
)
``` 

## Apply features

```
python apply_feast.py
```

## Materialize store

```
python materialize_stores.py
```


## Show stores 

```
python show_stores.py
```


