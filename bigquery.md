Mage DATA LOADER

```python
import pandas as pd
import pyarrow.parquet as pq
import requests
from io import BytesIO

if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test

@data_loader
def load_data(*args, **kwargs):
    """
    Template code for loading data from any source.

    Returns:
        Anything (e.g. data frame, dictionary, array, int, str, etc.)
    """
    # Specify your data loading logic here

    url1 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-01.parquet'
    url2 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-02.parquet'
    url3 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-03.parquet'
    url4 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-04.parquet'
    url5 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-05.parquet'
    url6 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-06.parquet'
    url7 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-07.parquet'
    url8 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-08.parquet'
    url9 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-09.parquet'
    url10 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-10.parquet'
    url11 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-11.parquet'
    url12 = 'https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2022-12.parquet'

    url_list = [url1, url2, url3, url4, url5, url6, url7, url8, url9, url10, url11, url12]

    # Initialize an empty list to store DataFrames
    dfs = []

    for url in url_list:
        # Download Parquet file
        response = requests.get(url)
        file_like = BytesIO(response.content)
        # Read Parquet file using Arrow
        table = pq.read_table(file_like)
        # Convert to DataFrame
        df = table.to_pandas()

        # Convert date columns to datetime format and format them
        date_columns = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']
        for col in date_columns:
            df[col] = pd.to_datetime(df[col])
            df[col] = df[col].dt.strftime('%Y-%m-%d %H:%M:%S')
            

        dfs.append(df)

    # Concatenate the DataFrames
    final_df = pd.concat(dfs, axis=0)

    print(f'Once the dataset is loaded, the shape of the data is : {final_df.shape}')
    print(f'Once the dataset is loaded, the dtypes of the data are : {final_df.info()}')

    return final_df

@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'
```
================================================================================
Mage DATA EXPORTER  to gcp
```python
from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.google_cloud_storage import GoogleCloudStorage
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter


@data_exporter
def export_data_to_google_cloud_storage(df: DataFrame, **kwargs) -> None:
    """
    Template for exporting data to a Google Cloud Storage bucket.
    Specify your configuration settings in 'io_config.yaml'.

    Docs: https://docs.mage.ai/design/data-loading#googlecloudstorage
    """
    config_path = path.join(get_repo_path(), 'io_config.yaml')
    config_profile = 'default'

    bucket_name = 'mage-zoomcamp-mpathak'
    object_key = 'green_taxi_2022_data1.parquet'

    GoogleCloudStorage.with_config(ConfigFileLoader(config_path, config_profile)).export(
        df,
        bucket_name,
        object_key,
    )
```

===================================================================================================================================

Important Note:

For this homework we will be using the 2022 Green Taxi Trip Record Parquet Files from the New York City Taxi Data found here:
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
If you are using orchestration such as Mage, Airflow or Prefect do not load the data into Big Query using the orchestrator.
Stop with loading the files into a bucket.

NOTE: You will need to use the PARQUET option files when creating an External Table

SETUP:
Create an external table using the Green Taxi Trip Records Data for 2022.
Create a table in BQ using the Green Taxi Trip Records for 2022 (do not partition or cluster this table).

```SQL

-- Creating external table referring to gcs path
CREATE OR REPLACE EXTERNAL TABLE `terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://mage-zoomcamp-mpathak/green_taxi_2022_data1.parquet']
);

```

```SQL

-- Create a non partitioned table from external table
CREATE OR REPLACE TABLE terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1_non_partitioned AS
SELECT * FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1;

```
===============================================================================================


Question 1: What is count of records for the 2022 Green Taxi Data??

65,623,481
**840,402**
1,936,423
253,647

```SQL

SELECT COUNT(*) FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1;

```


