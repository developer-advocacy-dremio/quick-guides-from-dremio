## Creating a Dremio / Iceberg / Minio / Spark test environemnt

### Step 1 - Docker Compose Setup

_Must have docker installed_

- Create a file called `docker-compose.yml` with the follow:

```yml
version: "3.9"
services:

  dremio:
    platform: linux/x86_64
    image: dremio/dremio-oss:latest
    ports: 
    - 9047:9047
    - 31010:31010
    - 32010:32010
    container_name: dremio

  minioserver:
    image: minio/minio
    ports:
    - 9000:9000
    - 9001:9001
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    container_name: minio
    command: server /data --console-address ":9001"

  spark_notebook:
    image: alexmerced/spark33-notebook
    ports: 
      - 8888:8888
    env_file: .env
    container_name: notebook

networks:
    default:
      name: iceberg_env
      driver: bridge
```

In the same directory create a `.env` file with the following:

```env
######################################
## Fill in Details
######################################

#### AWS_REGION is used by Spark
AWS_REGION=us-east-1
#### This must match if using minio
MINIO_REGION=us-east-1
#### Used by pyIceberg
AWS_DEFAULT_REGION=xxxx
#### AWS Credentials
AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXX
AWS_SECRET_ACCESS_KEY=xxxxxxx
#### If using Minio this should API address of Minio Server
AWS_S3_ENDPOINT=http://0.0.0.0:9000
#### Dremio Personal Access Token
TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
#### Location where files will be written when creating new tables
WAREHOUSE=s3a://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/
#### URI of Dremio Arctic Catalog (or Standalone Nessie Catalog)
ARCTIC_URI=https://nessie.dremio.cloud/v1/repositories/xxxxxxxxxxxxx
```


Create three terminals (easier to views logs if each service has their own terminal). Run the following commands in each one (the terminal must be in the same directory as the docker-compose.yml file).

- `docker-compose up dremio`
- `docker-compose up minioserver`

The minio terminal will show the API endpoint, you'll want to put this in your .env file as the `AWS_S3_ENDPOINT` before running the next command (include http:// for .env file).

- `docker-compose up spark_notebook`

That will spin up all necessary containers, note in the terminal that is running the minioserver the api address of the minio server, it will be needed soon.

So the following will be accessible in the browser:
- `localhost:9001`: Minio Dashboard (login with `minioadmin/minioadmin`)
- `localhost:9047`: Dremio, will create user on first visit
- `localhost:8888`: Jupyter Notebook with pyspark (Spark 3.3 running on container)

## Setup Minio

Head over to Minios dashboard login and do the following:

- create a few buckets, upload some sample files.
- create an access key, keep it available

## Setup Dremio Source

Get to Dremio dashboard then:

- Create a New Source
- Select S3
- Put in the access and secret key from minio, uncheck the encrypted checkbox
- head over to the advanced section and check off compatibility mode
- in the connection settings in the advanced section add the follow properties
  - `fs.s3a.path.style.access` set to `true`
  - `fs.s3a.endpoint` set to the API address of your Minio server example `0.0.0.0:9000` (without http://)

Now your connected to Minio from Dremio which can read and write data to the store.

## Connect in Notebook

You can use the settings specified in the [pySpark settings section of this repo](./icebergpyspark), just make sure to add the following setting so it points to your minio instance.

```
.set('spark.sql.catalog.arctic.s3.endpoint', "http://0.0.0.0:9000")
```

So for example if using a Dremio Arctic Catalog:

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
ARCTIC_URI = os.environ.get("ARCTIC_URI") ## Nessie Server URI
TOKEN = os.environ.get("TOKEN") ## Authentication Token
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS
MINIO_ENDPOINT = os.environ.get("AWS_S3_ENDPOINT") ## POINT TO MINIO

conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.0.0,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.44.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
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
        .set('spark.sql.catalog.arctic.s3.endpoint', MINIO_ENDPOINT)
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

If using just an HDFS (file system) catalog

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS
MINIO_ENDPOINT = os.environ.get("AWS_S3_ENDPOINT") ## POINT TO MINIO


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
        .setMaster(SPARK_MASTER)
  		#packages
        .set('spark.jars.packages', 'org.apache.hadoop:hadoop-aws:3.3.4,org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.0.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.hdfs_catalog', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.hdfs_catalog.type', 'hadoop')
        .set('spark.sql.catalog.hdfs_catalog.warehouse', 's3a://my-bucket/path/')
        .set('spark.sql.catalog.hdfs_catalog.s3.endpoint', MINIO_ENDPOINT)
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
