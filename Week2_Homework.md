# Week 2 Homework

Assignment

The goal will be to construct an ETL pipeline that loads the data, performs some transformations, and writes the data to a database (and Google Cloud!).

Create a new pipeline, call it green_taxi_etl

![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/e63c7ebb-781e-4d80-a161-ab5f6970b66f)

**Add a data loader block and use Pandas to read data for the final quarter of 2020 (months 10, 11, 12).**

You can use the same datatypes and date parsing methods shown in the course.

BONUS: load the final three months using a for loop and pd.concat

'''import pandas as pd

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

    url1 = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-10.csv.gz'
    
    url2 = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-11.csv.gz'
    
    url3 = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-12.csv.gz'

    url_list = [url1, url2, url3]

    taxi_dtypes = {
                'VendorID': pd.Int64Dtype(),
                'passenger_count': pd.Int64Dtype(),
                'trip_distance': float,
                'RatecodeID':pd.Int64Dtype(),
                'store_and_fwd_flag':str,
                'PULocationID':pd.Int64Dtype(),
                'DOLocationID':pd.Int64Dtype(),
                'payment_type': pd.Int64Dtype(),
                'fare_amount': float,
                'extra':float,
                'mta_tax':float,
                'tip_amount':float,
                'tolls_amount':float,
                'improvement_surcharge':float,
                'total_amount':float,
                'congestion_surcharge':float
            }
    parse_dates_green_taxi = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']

#Initialize an empty list to store DataFrames

    dfs = []

    for url in url_list:
    
        df = pd.read_csv(url, sep=',', compression='gzip', dtype=taxi_dtypes, parse_dates=parse_dates_green_taxi)
        
        dfs.append(df)

    #Concatenate the DataFrames
    
    print(f'Once the dataset is loaded, the shape of the data is : {pd.concat(dfs, axis = 0).shape}')
   
    return pd.concat(dfs, axis = 0)

@test

def test_output(output, *args) -> None:

    """
    Template code for testing the output of the block.
    """
    
    assert output is not None, 'The output is undefined'''
`

=============================================================================================================================================================

**Add a transformer block and perform the following:**

Remove rows where the passenger count is equal to 0 or the trip distance is equal to zero.

Create a new column lpep_pickup_date by converting lpep_pickup_datetime to a date.

Rename columns in Camel Case to Snake Case, e.g. VendorID to vendor_id.

Add three assertions:

vendor_id is one of the existing values in the column (currently)

passenger_count is greater than 0

trip_distance is greater than 0

`import re

if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@transformer
def transform(data, *args, **kwargs):
    """
    Template code for a transformer block.

    Add more parameters to this function if this block has multiple parent blocks.
    There should be one parameter for each output variable from each parent block.

    Args:
        data: The output from the upstream parent block
        args: The output from any additional upstream blocks (if applicable)

    Returns:
        Anything (e.g. data frame, dictionary, array, int, str, etc.)
    """
    # Specify your transformation logic here

    # Remove rows where the passenger count is equal to 0 or the trip distance is equal to 0

    data = data[(data['passenger_count'] != 0) & (data['trip_distance'] != 0)]

    print(f'''No. of rows left upon filtering the dataset where the passenger count
            is greater than 0 and the trip distance is greater than zero: {len(data)}''')

    # Create a new column 'lpep_pickup_date' with only the date
    data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date

    # Function to convert Camel Case to Snake Case

    def camel_to_snake(column_name):
        result = re.sub('([a-z0-9])([A-Z])', r'\1_\2', column_name)
        return result.lower()

    # Get the list of columns that need to be renamed
    columns_to_rename = [col for col in data.columns if camel_to_snake(col) != col]

    # Number of columns that need to be renamed
    num_columns_to_rename = len(columns_to_rename)

    print(f"Number of columns to be renamed to snake case: {num_columns_to_rename}")
    
    # Rename columns using the custom function
    data.rename(columns={col: camel_to_snake(col) for col in data.columns}, inplace=True)

    print(f"The existing values of vendor_id in the dataset:\n{data['vendor_id'].value_counts()}")

    return data


@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'

    assert output['vendor_id'].isin([1, 2]).all(), "Assertion Error: vendor_id is not one of the existing values."

    assert (output['passenger_count'] > 0).all(), "Assertion Error: passenger_count is not greater than 0."

    assert (output['trip_distance'] > 0).all(), "Assertion Error: trip_distance is not greater than 0."`


  ==================================================================================================================================

 **Using a Postgres data exporter (SQL or Python), write the dataset to a table called green_taxi in a schema mage. Replace the table if it already exists.**

`from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.postgres import Postgres
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter

@data_exporter
def export_data_to_postgres(df: DataFrame, **kwargs) -> None:

    schema_name = 'mage'  # Specify the name of the schema to export data to
    table_name = 'green_taxi'  # Specify the name of the table to export data to
    config_path = path.join(get_repo_path(), 'io_config.yaml')
    config_profile = 'dev'

    with Postgres.with_config(ConfigFileLoader(config_path, config_profile)) as loader:
        loader.export(
            df,
            schema_name,
            table_name,
            index=False,  # Specifies whether to include index in exported table
            if_exists='replace',  # Specify resolution policy if table name already exists
        )`

![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/83b06ec7-e5b4-483f-8cd5-8bc1fc64af3b)


==================================================================================================================================
**Write your data as Parquet files to a bucket in GCP, partioned by lpep_pickup_date. Use the pyarrow library!**

`from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.google_cloud_storage import GoogleCloudStorage
from pandas import DataFrame
import os
import pyarrow as pa
import pyarrow.parquet as pq 

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/home/src/terraform-demo-412315-6ac61949496c.json"
bucket_name = 'mage-zoomcamp-mpathak'
project_id = 'terraform-demo-412315'
object_key = 'green_taxi.parquet'
table_name = 'green_taxi'
root_path = f'{bucket_name}/{table_name}'


@data_exporter
def export_data(data, *args, **kwargs) -> None:
    table = pa.Table.from_pandas(data)
    gcs = pa.fs.GcsFileSystem()
    pq.write_to_dataset(
        table,
        root_path = root_path,
        partition_cols = ['lpep_pickup_date'],
        filesystem = gcs
    )`


![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/6ef2ad53-fa74-415b-b5cc-8c9c739a16cf)

![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/62094ac6-7fd3-4fbb-8705-7361a7187666)


======================================================================================================================

**Schedule your pipeline to run daily at 5AM UTC.**

![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/91c40184-96e3-44f8-b5ed-43fffb060b86)

**Question 1. Data Loading
Once the dataset is loaded, what's the shape of the data?**

**266,855 rows x 20 columns [Correct_Option]**

544,898 rows x 18 columns

544,898 rows x 20 columns

133,744 rows x 20 columns

**Question 2. Data Transformation
Upon filtering the dataset where the passenger count is greater than 0 and the trip distance is greater than zero, how many rows are left?**

544,897 rows

266,855 rows

**139,370 rows [Correct_Option]**

266,856 rows

**Question 3. Data Transformation
Which of the following creates a new column lpep_pickup_date by converting lpep_pickup_datetime to a date?**

data = data['lpep_pickup_datetime'].date

data('lpep_pickup_date') = data['lpep_pickup_datetime'].date

**data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date  [Correct_Option]**

data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt().date()

**Question 4. Data Transformation
What are the existing values of VendorID in the dataset?**

1, 2, or 3

**1 or 2 [Correct_Option]**

1, 2, 3, 4

1

**Question 5. Data Transformation
How many columns need to be renamed to snake case?**

3

6

2

**4 [Correct_Option]**

**Question 6. Data Exporting
Once exported, how many partitions (folders) are present in Google Cloud?**

**96 [Correct_Option]**

56

67

108

![image](https://github.com/learnermp/ZoomcampDE2024/assets/64764963/c6753422-50a4-4945-91d2-47077127046b)

===========================================================================================================





 


