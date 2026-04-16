from pyspark.context import SparkContext
from pyspark.sql.functions import col, to_date, when, upper
from awsglue.context import GlueContext

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session

df = spark.read.option("header", "true").csv("s3://vaishnav-handson12/reviews.csv")
output_path = "s3://vaishnav-handson12-processed/"

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
