# Hands-on 12: Spark on AWS (Glue + S3 + Lambda)

## 📌 Project Overview
This project demonstrates how to build an automated data processing pipeline using AWS services. The pipeline reads a dataset from Amazon S3, processes it using AWS Glue (PySpark), and stores the output back into S3. It also includes automation using AWS Lambda to trigger the Glue job.

---

## 🏗️ Architecture
- Amazon S3 (Input Bucket) → Stores raw dataset  
- AWS Glue (ETL Job) → Processes data using Spark  
- AWS Lambda → Triggers Glue job automatically  
- Amazon S3 (Output Bucket) → Stores processed results  

---

## 📂 Dataset
- File: `reviews.csv`  
- Location: `s3://vaishnav-handson12/reviews.csv`

---

## ⚙️ Implementation Steps

### Step 1: Created S3 Buckets
- Input Bucket: `vaishnav-handson12`
- Output Bucket: `vaishnav-handson12-pocessed`

---

### Step 2: Uploaded Dataset
- Uploaded `reviews.csv` into the input bucket

---

### Step 3: Created IAM Role
- Created IAM role for AWS Glue
- Attached policies:
  - AWSGlueServiceRole
  - AmazonS3FullAccess

---

### Step 4: Created AWS Glue Job
- Job Name: `process_reviews_job`
- Engine: Spark
- Used Script Editor
- Attached IAM role

---

### Step 5: Implemented PySpark ETL Script

```python
from pyspark.context import SparkContext
from pyspark.sql.functions import col, to_date, when, upper
from awsglue.context import GlueContext

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session

df = spark.read.option("header", "true").csv("s3://vaishnav-handson12/reviews.csv")
output_path = "s3://vaishnav-handson12-pocessed/"

df = df.withColumn("rating", col("rating").cast("int"))
df = df.withColumn("review_date", to_date(col("review_date"), "yyyy-MM-dd"))
df = df.withColumn("rating", when(col("rating").isNull(), 0).otherwise(col("rating")))
df = df.fillna({"review_text": "No review"})
df = df.withColumn("product_id_upper", upper(col("product_id")))

df.write.mode("overwrite").option("header", "true").csv(output_path + "processed-data/")

df.createOrReplaceTempView("reviews")

result = spark.sql("""
SELECT product_id_upper, AVG(rating) AS avg_rating, COUNT(*) AS total_reviews
FROM reviews
GROUP BY product_id_upper
""")

result.write.mode("overwrite").option("header", "true").csv(output_path + "analysis/")

### Step 6: Created AWS Lambda Function
import json
import boto3

glue = boto3.client('glue')

def lambda_handler(event, context):
    response = glue.start_job_run(JobName='process_reviews_job')
    return {
        'statusCode': 200,
        'body': json.dumps('Glue job started successfully')
    }

### Step 7: Configured Permissions
	•	Added inline policy to Lambda role:
	•	glue:StartJobRun
### Step 8: Configured S3 Trigger
	•	Event: Object Created (PUT)
	•	Bucket: vaishnav-handson12
### Step 9: Executed Pipeline
	•	Uploaded file to S3
	•	Lambda triggered Glue job automatically
	•	Glue processed data and stored results in S3
### SQL Query Used
SELECT product_id_upper, AVG(rating) AS avg_rating, COUNT(*) AS total_reviews
FROM reviews
GROUP BY product_id_upper;
