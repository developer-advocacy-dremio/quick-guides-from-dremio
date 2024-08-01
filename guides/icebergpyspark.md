- [Configuring Spark Iceberg Catalog](#configuring-your-catalog-in-pyspark)
- [Writing to Iceberg from a File](#writing-to-an-iceberg-table-from-a-file)

## Configuring your Catalog in pySpark

Below are several examples of configuring your catalog in pySpark depending which catalog your using. The name of the catalog is arbitrary and can be changed.

[Article on Spark Configuration for Iceberg](https://dev.to/alexmercedcoder/configuring-apache-spark-for-apache-iceberg-2d41)

For all the examples below you can remove the following if using a single node ([Like a Docker Container](https://github.com/developer-advocacy-dremio/quick-guides-from-dremio/blob/main/sparknotebook.md)) vs a spark cluster:

```
.setMaster(SPARK_MASTER)
```

#### Project Nessie/Dremio Arctic
```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
ARCTIC_URI = os.environ.get("ARCTIC_URI") ## Nessie Server URI
TOKEN = os.environ.get("TOKEN") ## Authentication Token
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,org.projectnessie.nessie-integrations:nessie-spark-extensions-3.3_2.12:0.76.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.arctic', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.arctic.uri', ARCTIC_URI)
        .set('spark.sql.catalog.arctic.ref', 'main')
        .set('spark.sql.catalog.arctic.authentication.type', 'BEARER')
        .set('spark.sql.catalog.arctic.authentication.token', TOKEN)
        .set('spark.sql.catalog.arctic.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.arctic.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.arctic.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM arctic.table1;").show()
```

#### Polaris Catalog

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
POLARIS_URI = 'http://localhost:8181/api/catalog'
POLARIS_CATALOG_NAME = 'polariscatalog'
POLARIS_CREDENTIALS = 'cf0c7cfe155d8f71:ee1037c68e4c5399a7ab50407e8bd7d5'
POLARIS_SCOPE = 'PRINCIPAL_ROLE:ALL'



conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.2,org.apache.hadoop:hadoop-aws:3.4.0')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.polaris', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.polaris.warehouse', POLARIS_CATALOG_NAME)
        .set('spark.sql.catalog.polaris.header.X-Iceberg-Access-Delegation', 'true')
        .set('spark.sql.catalog.polaris.catalog-impl', 'org.apache.iceberg.rest.RESTCatalog')
        .set('spark.sql.catalog.polaris.uri', POLARIS_URI)
        .set('spark.sql.catalog.polaris.credential', POLARIS_CREDENTIALS)
        .set('spark.sql.catalog.polaris.scope', POLARIS_SCOPE)
        .set('spark.sql.catalog.polaris.token-refresh-enabled', 'true')
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("CREATE TABLE polaris.names (name STRING) USING iceberg").show()
spark.sql("INSERT INTO polaris.names VALUES ('Alex Merced'), ('Andrew Madson')").show()
spark.sql("SELECT * FROM polaris.names").show()

```

#### AWS Glue

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.glue', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.glue.catalog-impl', 'org.apache.iceberg.aws.glue.GlueCatalog')
        .set('spark.sql.catalog.glue.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.glue.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM glue.table1;").show()
```

#### HDFS

`this library is -org.apache.hadoop:hadoop-aws:3.3.4- is needed for writing to S3 with hadoop catalog`

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.hadoop:hadoop-aws:3.3.4,org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.hdfs_catalog', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.hdfs_catalog.type', 'hadoop')
        .set('spark.sql.catalog.hdfs_catalog.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.hdfs_catalog.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM hdfs_catalog.table1;").show()
```

#### HIVE

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
HIVE_URI = os.environ.get("ARCTIC_URI") ## Hive Server URI
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.hive', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.hive.type', 'hive')
        .set('spark.sql.catalog.hive.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.hive.uri', HIVE_URI)
        .set('spark.sql.catalog.hive.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM hive.table1;").show()
```

#### JDBC

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
DB_URI = os.environ.get("ARCTIC_URI") ## Database URI
DB_USER = os.environ.get("DB_USER") ## Database Username
DB_PASS = os.environ.get("DB_PASS") ## Database Password
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.dbcatalog', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.dbcatalog.catalog-impl', 'org.apache.iceberg.jdbc.JdbcCatalog')
        .set('spark.sql.catalog.dbcatalog.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.dbcatalog.jdbc.uri', HIVE_URI)
        .set('spark.sql.catalog.dbcatalog.jdbc.useSSL', "true")
        .set('spark.sql.catalog.dbcatalog.jdbc.user', DB_USER)
        .set('spark.sql.catalog.dbcatalog.jdbc.password', DB_PASSWORD)
        .set('spark.sql.catalog.dbcatalog.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM dbcatalog.table1;").show()
```

## DynamoDB

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.dynamo', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.dynamo.catalog-impl', 'org.apache.iceberg.aws.dynamodb.DynamoDbCatalog')
        .set('spark.sql.catalog.dynamo.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.dynamo.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("SELECT * FROM dynamo.table1;").show()
```

* This by default creates dynamodb table called `iceberg` for storing catalog entries. If you want to use a different table [set the table-name property](https://iceberg.apache.org/docs/latest/aws/#dynamodb-catalog).

```
.set('spark.sql.catalog.dynamo.dynamodb.table-name', 'some-other-table-name')
```

#### Iceberg and Delta Lake in the same Spark session

In this example it would be a Nessie/Arctic catalog under the namespace `arctic` and delta lake configured to use the internal spark catalog under the namespace `default`.

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
ARCTIC_URI = os.environ.get("ARCTIC_URI") ## Nessie Server URI
TOKEN = os.environ.get("TOKEN") ## Authentication Token
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'io.delta:delta-core_2.12:2.1.1,org.apache.hadoop:hadoop-aws:3.3.1,org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.4.3,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.44.0')
  		#SQL Extensions
        .set('spark.sql.extensions', 'io.delta.sql.DeltaSparkSessionExtension,org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.arctic', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.arctic.uri', ARCTIC_URI)
        .set('spark.sql.catalog.arctic.ref', 'main')
        .set('spark.sql.catalog.arctic.authentication.type', 'BEARER')
        .set('spark.sql.catalog.arctic.authentication.token', TOKEN)
        .set('spark.sql.catalog.arctic.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.arctic.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.arctic.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
        #Configuring Delta Lake
        .set('spark.sql.catalog.spark_catalog', 'org.apache.spark.sql.delta.catalog.DeltaCatalog')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query creating an Iceberg table from a Delta Lake table
spark.sql("CREATE TABLE arctic.table1 USING iceberg AS (SELECT * FROM default.table1);")
```

## Writing to an Iceberg Table from a File

General Steps
- Turn file into a dataframe
- Turn dataframe into a view
- Use view for SQL (CTAS, MERGE, etc.)

```py
## Create Dataframe from Parquet File
dataframe=spark.read.parquet("s3a://bucket/output/people.parquet")

## Turn Dataframe into a temporary view
dataframe.createOrReplaceTempView("myview")

## Create new iceberg table in my configured catalog
spark.sql("CREATE TABLE mycatalog.table1 AS (SELECT * FROM myview)")

## Run an upsert again view
spark.sql("""
MERGE INTO mycatalog.table1 t
    USING (SELECT * FROM myview) s
    ON t.id = s.id
    WHEN MATCHED AND s.op = 'delete' THEN DELETE
    WHEN MATCHED AND t.count IS NULL AND s.op = 'increment' THEN UPDATE SET t.count = 0
    WHEN MATCHED AND s.op = 'increment' THEN UPDATE SET t.count = t.count + 1
    WHEN NOT MATCHED THEN INSERT *
""")

```
