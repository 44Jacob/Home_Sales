# Home_Sales
Home Sales Data Analysis with PySpark

Overview
This project involves analyzing a home sales dataset using PySpark. The analysis includes reading data from an AWS S3 bucket, performing various SQL queries, and comparing runtime performance with and without caching. The project demonstrates the use of PySpark for big data processing and analysis, including creating temporary views, caching tables, and partitioning data.

Requirements
Python 3.x
PySpark
Java 11
AWS S3 access

Setup
Install Spark and Java


Update and install Java:

sh

!apt-get update
!apt-get install openjdk-11-jdk-headless -qq > /dev/null


Download and extract Spark:

sh

!wget -q http://www.apache.org/dist/spark/{SPARK_VERSION}/{SPARK_VERSION}-bin-hadoop3.tgz
!tar xf {SPARK_VERSION}-bin-hadoop3.tgz
!pip install -q findspark


Set Environment Variables

import os
spark_version = 'spark-3.4.3'
os.environ['SPARK_VERSION'] = spark_version
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-11-openjdk-amd64"
os.environ["SPARK_HOME"] = f"/content/{spark_version}-bin-hadoop3"
Initialize PySpark
python
Copy code
import findspark
findspark.init()
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("SparkSQL").getOrCreate()


Data Loading

Read Data from AWS S3 Bucket

from pyspark import SparkFiles
url = "https://2u-data-curriculum-team.s3.amazonaws.com/dataviz-classroom/v1.2/22-big-data/home_sales_revised.csv"
spark.sparkContext.addFile(url)
home_sales_df = spark.read.csv(SparkFiles.get('home_sales_revised.csv'), header=True)
home_sales_df.show()

Create a Temporary View
home_sales_df.createOrReplaceTempView('home_sales')


SQL Queries

Average Price for a Four Bedroom House Sold Per Year
spark.sql('''
  SELECT
    YEAR(date) AS year,
    ROUND(AVG(price),2) AS avg_price
  FROM home_sales
  WHERE bedrooms = 4
  GROUP BY year
  ORDER BY year DESC
''').show()


Average Price of a Home by Year Built (3 Bedrooms, 3 Bathrooms)
spark.sql('''
  SELECT
    date_built AS year,
    ROUND(AVG(price),2) AS avg_price
  FROM home_sales
  WHERE bedrooms = 3 AND bathrooms = 3
  GROUP BY year
  ORDER BY year DESC
''').show()

Average Price of a Home by Year Built (3 Bedrooms, 3 Bathrooms, 2 Floors, ≥2000 sqft)
spark.sql('''
  SELECT
    date_built AS year,
    ROUND(AVG(price),2) AS avg_price,
    MIN(bedrooms) AS bedrooms,
    MIN(bathrooms) AS bathrooms,
    ROUND(AVG(sqft_living),0) AS sqft
  FROM home_sales
  WHERE bedrooms = 3 AND bathrooms = 3 AND floors = 2 AND sqft_living >= 2000
  GROUP BY year
  ORDER BY year DESC
''').show()

Average Price of a Home Per "View" Rating (≥ $350,000)
import time
start_time = time.time()

spark.sql('''
  SELECT
    view,
    ROUND(AVG(price),2) AS avg_price
  FROM home_sales
  GROUP BY view
  HAVING avg_price >= 350000
  ORDER BY view DESC
''').show()

print("--- %s seconds ---" % (time.time() - start_time))
Caching and Performance Comparison

Cache the Temporary Table
spark.sql("cache table home_sales")

Check if the Table is Cached
spark.catalog.isCached('home_sales')


Run the Last Query Again with Cached Data
start_time = time.time()
spark.sql('''
  SELECT
    view,
    ROUND(AVG(price),2) AS avg_price
  FROM home_sales
  GROUP BY view
  HAVING avg_price >= 350000
  ORDER BY view DESC
''').show()
print("--- %s seconds ---" % (time.time() - start_time))


Parquet File Operations

Partition Data by "date_built" and Write to Parquet
home_sales_df.write.partitionBy('date_built').parquet('p_home_sales', mode='overwrite')


Read the Parquet Data

p_home_sales_df = spark.read.parquet('p_home_sales')
p_home_sales_df.show(5)
Create a Temporary View for the Parquet Data
python
Copy code
p_home_sales_df.createOrReplaceTempView('p_home_sales')


Run the Last Query on Parquet Data

start_time = time.time()
spark.sql('''
  SELECT
    view,
    ROUND(AVG(price),2) AS avg_price
  FROM p_home_sales
  GROUP BY view
  HAVING avg_price >= 350000
  ORDER BY view DESC
''').show()
print("--- %s seconds ---" % (time.time() - start_time))


Clean Up

Uncache the Temporary Table
spark.sql('uncache table home_sales')

Check if the Table is No Longer Cached
spark.catalog.isCached('home_sales')

Summary
This project demonstrates how to use PySpark for big data processing and analysis. It covers data loading, SQL queries, caching, and performance comparison. Additionally, it shows how to partition data and read/write Parquet files. The analysis provides insights into home sales data, such as average prices for different configurations and the impact of view ratings on prices.

By following the steps outlined above, you can replicate the analysis and adapt it to similar datasets for further exploration and insights.
