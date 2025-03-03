```py
import pyspark
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
NESSIE_URI = "http://nessie:19120/api/v1"
MINIO_ACCESS_KEY = "Gh2YstOvL9GffNWMTFA1"
MINIO_SECRET_KEY = "UQWoVtjF5qwiKIDhjzFW5u59DC5KsE3fal9JWF5O"



conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.3.1,org.projectnessie.nessie-integrations:nessie-spark-extensions-3.3_2.12:0.67.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.nessie', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.nessie.uri', NESSIE_URI)
        .set('spark.sql.catalog.nessie.ref', 'main')
        .set('spark.sql.catalog.nessie.authentication.type', 'NONE')
        .set('spark.sql.catalog.nessie.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.nessie.warehouse', 's3a://warehouse')
        .set('spark.sql.catalog.nessie.s3.endpoint', 'http://minio:9000')
        .set('spark.sql.catalog.nessie.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#MINIO CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', MINIO_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', MINIO_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")




spark.sql("CREATE TABLE IF NOT EXISTS nessie.test1 (name string) USING iceberg;")
spark.sql("INSERT INTO nessie.test1 VALUES ('test');").show()
spark.sql("SELECT ** FROM nessie.test1;").show()


csv_df = spark.read.format("csv").option("header", "true").load("../datasets/df_open_2023.csv")
csv_df.createOrReplaceTempView("csv_open_2023")



spark.sql("CREATE TABLE IF NOT EXISTS nessie.df_open_2023_lesson2 USING iceberg AS SELECT * FROM csv_open_2023").show()



spark.sql("SELECT * FROM nessie.df_open_2023_lesson2 limit 10").show()

```