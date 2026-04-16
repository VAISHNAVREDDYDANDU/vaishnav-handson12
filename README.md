# AWS Glue Data Processing – Reviews Dataset

## Project Overview
This project demonstrates how to process and analyze a dataset using AWS Glue and Apache Spark. The goal is to clean raw review data stored in Amazon S3 and generate meaningful insights.

## Technologies Used
- AWS Glue (ETL)
- Apache Spark (PySpark)
- Amazon S3
- SQL (Spark SQL)

## Dataset
Input File: reviews.csv  
Location: s3://vaishnav-handson12/reviews.csv

## Steps Performed

### Step 1: Data Ingestion
The dataset was read from Amazon S3 into a Spark DataFrame.

### Step 2: Data Cleaning
- Converted rating column to integer  
- Replaced null ratings with 0  
- Converted review_date to date format  
- Filled missing review text with "No review"  
- Created a new column product_id_upper (uppercase)

### Step 3: Data Transformation
The cleaned dataset was structured and stored back into S3.

### Step 4: Data Analysis
Used Spark SQL to compute:
- Average rating per product  
- Total number of reviews per product  

## SQL Query Used

SELECT product_id_upper, AVG(rating) AS avg_rating, COUNT(*) AS total_reviews  
FROM reviews  
GROUP BY product_id_upper;

## Output

Processed Data:
s3://vaishnav-handson12-pocessed/processed-data/

Analysis Results:
s3://vaishnav-handson12-pocessed/analysis/

## Output Screenshots
(Add screenshots here)
- S3 bucket with processed-data folder  
- S3 bucket with analysis folder  
- Glue job run status (Succeeded)

## Challenges Faced
- Incorrect S3 path (used s3::// instead of s3://)  
- Bucket name typo (processed vs pocessed)  
- Glue job failed with NoSuchBucket error  
- Debugging multiple failed runs  

## Learnings
- Hands-on experience with AWS Glue  
- Working with Spark DataFrames  
- Writing Spark SQL queries  
- Debugging AWS errors  
- Understanding S3 paths and structure  

## Conclusion
This project successfully demonstrates a complete data pipeline using AWS Glue. Raw data was cleaned, transformed, and analyzed to generate useful insights.

## Future Improvements
- Add more transformations  
- Automate pipeline with triggers  
- Visualize results using dashboards  
