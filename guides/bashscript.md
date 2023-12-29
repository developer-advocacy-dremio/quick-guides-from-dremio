## Spark Configuration Bash Script

Run this script to start your Spark 3.3 server with Iceberg already configured as a catalog named "Iceberg".

Feel free to adjust the script to your needs.

```bash
echo "What catalog are you using?"
echo "[1] HDFS | [2] NESSIE | [3] GLUE | [4] ARCTIC"

read selection

echo "Storing Locally or on S3?"
echo "[1] local | [2] S3"

read selection2

######################################################

if [ $selection2 -eq 1 ]
then
echo "What is the warehouse name? (folder to save data in)"
echo "example: datawarehouse"

read WAREHOUSE

IO_IMPL= "--conf spark.sql.catalog.iceberg.type=hadoop"

fi


if [ $selection2 -eq 2 ]
then
echo "What is the warehouse path? (S3 PATH TO SAVE DATA INTO)"
echo "example: s3a://my_bucket/subfolder"

read WAREHOUSE

echo "What is your AWS_ACCESS_KEY_ID"

read AWS_ACCESS_KEY_ID

export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID

echo "What is your AWS_ACCESS_KEY_ID"

read AWS_SECRET_ACCESS_KEY

export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

echo "What is your S3 Region? (ex. us-east-1)"

read AWS_REGION

export AWS_REGION=$AWS_REGION

export AWS_DEFAULT_REGION=$AWS_REGION

IO_IMPL="--conf spark.sql.catalog.iceberg.io-impl=org.apache.iceberg.aws.s3.S3FileIO"

ICEBERG_PACKAGES=",software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178"

fi

#####################################################

if [ $selection -eq 1 ]
then
echo "1 1"

COMMAND="spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.1$ICEBERG_PACKAGES --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.warehouse=$WAREHOUSE $IO_IMPL"

fi

if [ $selection -eq 2 ]
then

echo "What is your Nessie Server URL"

read AWS_SECRET_ACCESS_KEY

COMMAND="spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.1,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.44.0$ICEBERG_PACKAGES --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg.catalog-impl=org.apache.iceberg.nessie.NessieCatalog --conf spark.sql.catalog.iceberg.uri=$NESSIE_URI --conf spark.sql.catalog.iceberg.ref=main --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.warehouse=$WAREHOUSE/iceberg-warehouse $IO_IMPL"
fi

if [ $selection -eq 3 ]
then
COMMAND="spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.1$ICEBERG_PACKAGES --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg.catalog-impl=org.apache.iceberg.aws.glue.GlueCatalog --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.warehouse=$WAREHOUSE/iceberg-warehouse $IO_IMPL"
fi

if [ $selection -eq 4 ]
then
echo "What is your Nessie Server URL"

read NESSIE_URI

echo "What is your Nessie Auth Token"

read TOKEN

COMMAND="spark-sql --packages org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.1,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.44.0$ICEBERG_PACKAGES --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg.catalog-impl=org.apache.iceberg.nessie.NessieCatalog --conf spark.sql.catalog.iceberg.uri=$NESSIE_URI --conf spark.sql.catalog.iceberg.ref=main --conf spark.sql.catalog.iceberg.authentication.type=BEARER --conf spark.sql.catalogiceberg.authentication.token=$TOKEN --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.warehouse=$WAREHOUSE/iceberg-warehouse $IO_IMPL"
fi

echo $COMMAND

eval $COMMAND
```
