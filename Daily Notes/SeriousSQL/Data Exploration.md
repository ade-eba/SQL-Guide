

# :triangular_flag_on_post:Data Exploration
  
  #### Select & Sort Data
  
  
  #### Record Counts & Distinct Values
  
  
  #### Identifying Duplicate Records
````sql
SELECT *
FROM health.user_logs
LIMIT 10;
````
````sql
SELECT COUNT(*)
FROM health.user_logs;
````
````sql
SELECT COUNT(DISTINCT id)
FROM health.user_logs;
````
````sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY frequency DESC;
````
````sql
SELECT
  id,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY id
ORDER BY frequency DESC
LIMIT 10;
````
````sql
SELECT
  measure_value,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
````
````sql
SELECT
  systolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
````
````sql
SELECT
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
````
````sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE measure_value = 0
GROUP BY 1;
````
````sql
SELECT *
FROM health.user_logs
WHERE measure_value = 0
AND measure = 'blood_pressure'
LIMIT 10;
````
````sql
SELECT *
FROM health.user_logs
WHERE measure_value != 0
AND measure = 'blood_pressure'
LIMIT 10;
````
````sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE systolic IS NULL
GROUP BY 1;
````
````sql
SELECT
  measure,
  COUNT(*)
FROM health.user_logs
WHERE diastolic IS NULL
GROUP BY 1;
````

Count of each log
````sql

SELECT
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
ORDER BY frequency DESC
````
Selecting duplicate logs only
 Don't forget to clean up any existing temp tables!
````sql
DROP TABLE IF EXISTS unique_duplicate_records;

CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;
````
 Finally let's inspect the top 10 rows of our temp table
````sql
SELECT *
FROM unique_duplicate_records
LIMIT 10;
````

Selecting & counting duplicate logs only
````sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT *
FROM groupby_counts
WHERE frequency > 1
ORDER BY frequency DESC
LIMIT 10;
````
Which id value has the most number of duplicate records in the health.user_logs table?
````sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT id,SUM(frequency) AS Total_duplicate_rows
FROM groupby_counts
WHERE frequency > 1
GROUP BY id
ORDER BY Total_duplicate_rows desc
LIMIT 10;

````
Which log_date value had the most duplicate records after removing the max duplicate id value from question above?
````sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  WHERE id!='054250c692e07a9fa9e62e345231df4b54ff435d'
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)

SELECT log_date,SUM(frequency) AS Total_duplicate_rows
FROM groupby_counts
WHERE frequency > 1 
GROUP BY log_date
ORDER BY Total_duplicate_rows desc
LIMIT 10;
````
Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?
````sql
SELECT measure_value,count(measure_value) as frequency
FROM health.user_logs
WHERE measure='weight'
GROUP BY measure_value
ORDER BY frequency desc
LIMIT 10
````
How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table?

````sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  WHERE measure = 'blood_pressure'
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT
  COUNT(*) as single_duplicate_rows,
  SUM(frequency) as total_duplicate_records
FROM groupby_counts
WHERE frequency > 1;

````
What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition?
- With window function
````sql
WITH all_measure_values AS (
  SELECT
    measure_value,
    COUNT(*) AS total_records,
    SUM(COUNT(*)) OVER () AS overall_total
  FROM health.user_logs
  WHERE measure = 'blood_pressure'
  GROUP BY 1
)
SELECT
  measure_value,
  total_records,
  overall_total,
  ROUND(100 * total_records::NUMERIC / overall_total, 2) AS percentage
FROM all_measure_values
WHERE measure_value = 0;
````
- Without window function

What percentage of records are duplicates in the health.user_logs table?

````sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT
  ROUND(
  100* SUM(CASE
           WHEN frequency>1 THEN frequency-1
           ELSE 0 END
  )::NUMERIC/SUM(frequency),2)
 AS Duplicate_percentage
FROM groupby_counts;
````
with subqueries
````sql
WITH deduped_logs AS (
  SELECT DISTINCT *
  FROM health.user_logs
)
SELECT
  ROUND(
    100 * (
      (SELECT COUNT(*) FROM health.user_logs) -
      (SELECT COUNT(*) FROM deduped_logs)
    )::NUMERIC /
    (SELECT COUNT(*) FROM health.user_logs),
    2
  ) AS duplicate_percentage;
````  
  
  #### Summary Statistics
  
  ````sql
SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value,
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    -- this function actually returns a float which is incompatible with ROUND!
    -- we use this cast function to convert the output type to NUMERIC
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';
````

What is the average, median and mode values of blood glucose values to 2 decimal places?
````sql
SELECT 
  ROUND(AVG(measure_value), 2) AS average_value,
  ROUND(CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC), 2) AS median_value,
  ROUND(MODE() WITHIN GROUP(ORDER BY measure_value) ,2) AS mode_value
FROM health.user_logs
WHERE measure='blood_glucose';
````
What is the most frequently occuring measure_value value for all blood glucose measurements?
````sql
SELECT
ROUND(MODE() WITHIN GROUP(ORDER BY measure_value),2) AS frequent_value
FROM health.user_logs
WHERE measure='blood_glucose';
````
OR
````sql
SELECT 
  measure_value,
  COUNT(*) AS frequency
FROM health.user_logs
WHERE measure='blood_glucose'
GROUP BY measure_value
ORDER BY frequency DESC
LIMIT 10;
````
Pearson Coefficient of Skewness
````sql
WITH skew_check AS ( 
SELECT
 AVG(measure_value) AS mean_value,
 PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY measure_value) AS median_value,
 MODE() WITHIN GROUP(ORDER BY measure_value) AS mode_value,
 STDDEV(measure_value) AS standard_deviation
FROM health.user_logs
WHERE measure='blood_glucose'
)

SELECT
 (mean_value-mode_value)/standard_deviation AS mode_skewness,
 3*(mean_value-median_value)/standard_deviation AS median_skewness
FROM skew_check;
````
  #### Distribution Functions
  
````sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;

````
##### Window Functions for sorting values
- What is the difference between ROW_NUMBER, RANK and DENSE_RANK window functions?
- Finding the large outliers
````sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  measure_value,
  -- these are examples of window functions below
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
  RANK() OVER (ORDER BY measure_value DESC) as rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;
````
##### Finding the small outliers
````sql
WITH percentile_values AS (
  SELECT measure_value,
    NTILE(100) OVER(
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure='weight'
)

SELECT measure_value,
       ROW_NUMBER() OVER(ORDER BY measure_value) AS row_number_order,
       RANK() OVER (ORDER BY measure_value) AS rank_order,
       DENSE_RANK() OVER (ORDER BY measure_value) AS dense_rank_order
       FROM percentile_values
       WHERE percentile=1
       ORDER BY measure_value;
   ````    
##### Removing the outliers
1.Create a temporary weight_logs table with our outliers removed
````sql
DROP TABLE IF EXISTS weight_logs;
CREATE TEMP TABLE weight_logs AS (
  SELECT *
  FROM health.user_logs
  WHERE measure = 'weight'
    AND measure_value > 0
    AND measure_value < 201
);
````
2.Calculate summary statistics on this new temp table

````sql
SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value,
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    -- this function actually returns a float which is incompatible with ROUND!
    -- we use this cast function to convert the output type to NUMERIC
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM weight_logs;
````
3.Show the new cumulative distribution function with treated data
````sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM weight_logs
)
SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;

````
##### Line plot
![cumudist](https://user-images.githubusercontent.com/17575943/136075295-d9ee4199-3df0-4c1d-86ce-5441bec2caf1.PNG)


##### Histograms for frequency distribution

1.We need to find the minimum and maximum values from the weight values to use as the ranges for ``WIDTH_BUCKET`` function.
````sql
SELECT
  MIN(measure_value) AS minimum_value,
  MAX(measure_value) AS maximum_value
FROM weight_logs;
````
2.- let’s take 0 as the minimum and 200 as the maximum and use 50 buckets for our visualization, we will also need the AVG from each bucket to use as our X-value location for the histogram:
````sql
SELECT
  WIDTH_BUCKET(measure_value, 0, 200, 50) AS bucket,
  AVG(measure_value) AS average_value,
  COUNT(*) AS frequency
FROM weight_logs
GROUP BY bucket
ORDER BY bucket;
````
##### Histogram plotted
![histogram](https://user-images.githubusercontent.com/17575943/136075001-9f53108e-1664-4a8d-9463-1579ff5e0295.PNG)


  #### Health Analytics Mini Case Study
  
  Click here to view this case study on my Projects & Case Studies Repository.
  
  

