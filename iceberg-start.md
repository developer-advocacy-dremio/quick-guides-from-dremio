# Apache Iceberg Quickstart

If just looking to get started just evaluating Apache Iceberg, follow the directions in the code snippet below:

```py
# RUN THIS COMMAND TO START DOCKER CONTAINER
docker run -it --name spark34 alexmerced/spark34

# Once Container is Running, this will start spark-sql with a catalog called "my_iceberg_catalog"
spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-3.4_2.13:1.3.1 \
--conf spark.sql.catalog.my_iceberg_catalog=org.apache.iceberg.spark.SparkCatalog \
--conf spark.sql.catalog.my_iceberg_catalog.type=hadoop \
--conf spark.sql.catalog.my_iceberg_catalog.warehouse=$PWD/warehouse

# Run this query
CREATE TABLE my_iceberg_catalog.names (name STRING) USING iceberg;
```

Here are some other tutorials for getting hands on building Apache Iceberg lakehouses:
- [Ingesting Data into Iceberg with Flink](https://www.dremio.com/blog/using-flink-with-apache-iceberg-and-nessie/)
- [Building an Iceberg Lakehouse on your laptop with Spark/Nessie/Flink](https://dev.to/alexmercedcoder/data-engineering-create-a-apache-iceberg-based-data-lakehouse-on-your-laptop-41a8)
