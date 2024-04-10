```
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StringType, IntegerType

spark = SparkSession.builder.\
    appName("KafkaIntegrationExample").\
    config("spark.jars.packages", "org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1").\
    getOrCreate()

kafka_df = spark.readStream.format("kafka")\
    .option("kafka.bootstrap.servers", "localhost:9092")\
    .option("subscribe", "test")\
    .option("startingOffsets", "earliest")\
    .load()

df = kafka_df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

query = df.writeStream.outputMode("append").format("console").start()
query.awaitTermination()
```
###参考
- [Pyspark Sql Utils Analysisexception Failed to Find Data Source Kafka](https://tech.sadaalomma.com/sql/pyspark-sql-utils-analysisexception-failed-to-find-data-source-kafka/)