```SQL
-- Creating external table referring to gcs path
CREATE OR REPLACE EXTERNAL TABLE `terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://mage-zoomcamp-mpathak/green_taxi_2022_data1.parquet']
);

-- Viewing top 10 rows of the external table
SELECT COUNT(*) FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1;

-- Create a non partitioned table from external table
CREATE OR REPLACE TABLE terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1_non_partitoned AS
SELECT * FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1;

-- Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.

SELECT DISTINCT PULocationID FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1;  # 0 KB

SELECT DISTINCT PULocationID FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1_non_partitoned; # 6.41 MB


-- How many records have a fare_amount of 0?

SELECT COUNT(*) FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1 WHERE fare_amount = 0; 

# 1622

-- What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)
-- Creating a partition and cluster table

CREATE TABLE `terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_non_partitioned_new`
AS
SELECT
  *,
  TIMESTAMP(PARSE_DATETIME('%Y-%m-%d %H:%M:%S', lpep_pickup_datetime)) AS new_lpep_pickup_datetime,
  TIMESTAMP(PARSE_DATETIME('%Y-%m-%d %H:%M:%S', lpep_dropoff_datetime)) AS new_lpep_dropoff_datetime,
FROM
  `terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1_non_partitoned`;


-- SELECT * FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_non_partitioned_new LIMIT 10;

CREATE OR REPLACE TABLE terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_partitioned_and_clustered
PARTITION BY DATE(new_lpep_pickup_datetime) 
CLUSTER BY PUlocationID AS
SELECT * FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_non_partitioned_new;

-- Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime 06/01/2022 and 06/30/2022 (inclusive)


SELECT DISTINCT PULocationID FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_non_partitioned_new
WHERE new_lpep_pickup_datetime BETWEEN '2022-06-01' AND '2022-06-30';

# 12.82 MB


SELECT DISTINCT PULocationID FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data_partitioned_and_clustered
WHERE new_lpep_pickup_datetime BETWEEN '2022-06-01' AND '2022-06-30';

# 1.12 MB

SELECT count(*) FROM terraform-demo-412315.ny_taxi.external_green_taxi_2022_data1_non_partitoned


```
