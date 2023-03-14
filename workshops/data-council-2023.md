# Data As Code Workshop

In this exercise we will connect to a Dremio Arctic catalog using Apache Spark locally.

#### Dremio Arctic Resources

- [Gnarly Data Waves Episode on Data As Code (Includes Demo)]()

#### Pre-Requisites

- An AWS Account
- A Dremio Cloud Account [Sign Up or Login Here](https://app.dremio.cloud/)
- A Dremio Arctic Catalog 

#### Information you'll need

- AWS Region, Access Key and Secret Key 
- Dremio Cloud Personal Access Token (get these from account settings, bottom left corner)
- S3 Address of where you want to write to (this can be any location your AWS Account is able to write to)
- Arctic Catalog URI (in the project settings for the particular Arctic catalog you want to connect to)

## Step 1 - Startup a Docker Container with Spark & Jupyter Notebook

```
docker run -p 8888:8888 --env AWS_REGION=us-east-1 --env AWS_ACCESS_KEY_ID=xxxxxx --env AWS_SECRET_ACCESS_KEY=xxxxx --env TOKEN=xxxxxxx --env ARCTIC_URI=https://nessie.dremio.cloud/v1/repositories/xxxxxxxxxx --env WAREHOUSE=s3a://someS3Bucket/ --name spark-notebook alexmerced/spark33-notebook
```
_*Make sure to replace all the enviornment variables with ones that apply to you_

When you run this command, the URL with Jupyter sever token will be in the output, make sure to use it to open up notebook in the browser.

```
[I 20:37:04.165 NotebookApp] http://487790f29fb0:8888/?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
[I 20:37:04.165 NotebookApp]  or http://127.0.0.1:8888/?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
[I 20:37:04.165 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

Once this is opened in the browser create a new notebook and add the following:

```py
# Install a pip package in the current Jupyter kernel
import sys
!{sys.executable} -m pip install pandas matplotlib

import pyspark
from pyspark.sql import SparkSession
import os
import matplotlib.pyplot as plt
import pandas as pd

## DEFINE SENSITIVE VARIABLES
ARCTIC_URI = os.environ.get("ARCTIC_URI") ## Nessie Server URI
TOKEN = os.environ.get("TOKEN") ## Authentication Token
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS
WAREHOUSE = os.environ.get("WAREHOUSE") ## S3 Address to Write to


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
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
        .set('spark.sql.catalog.arctic.warehouse', WAREHOUSE)
        .set('spark.sql.catalog.arctic.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query
spark.sql("CREATE TABLE arctic.notebook.example (id INT, x INT, y INT)").show()
```

## Adding data to the table we created

```py
spark.sql("INSERT INTO arctic.datacouncil.example VALUES (1, 1, 2), (2, 2, 3), (3, 3, 2), (4, 4, 4), (5, 5, 6)").show()
spark.sql("SELECT * FROM arctic.datacouncil.example").show()
```

## Creating a Branch to Isolate Experimentation

#### Plot the Data Before Experimentation

```py
df = spark.sql("SELECT * FROM arctic.datacouncil.example")
data = df.select('x','y').sample(False, 0.8).toPandas()
plt.plot(data.x,data.y)
plt.xlabel('x')
plt.ylabel('y')
plt.title('Chart of Data on Production/Main Branch')
plt.show()
```

### Create Branch and Run Experiements

```py
spark.sql("CREATE BRANCH IF NOT EXISTS datacouncilbranch IN arctic")
spark.sql("USE REFERENCE datacouncilbranch IN arctic")
spark.sql("INSERT INTO arctic.datacouncil.example VALUES (6, 1, 1), (7, 7, 9), (8, 2, 3), (9, 3, 8), (10, 5, 2)").show()
df = spark.sql("SELECT * FROM arctic.datacouncil.example")
data = df.select('x','y').sample(False, 0.8).toPandas()
plt.plot(data.x,data.y)
plt.xlabel('x')
plt.ylabel('y')
plt.title('Chart of Data on Example Branch')
plt.show()
```

### Plot data to show main branch is unaffected

```py
# This... doesn't look write, lucky, this insert doesn't effect our production branch
spark.sql("USE REFERENCE main IN arctic")
df = spark.sql("SELECT * FROM arctic.datacouncil.example")
data = df.select('x','y').sample(False, 0.8).toPandas()
plt.plot(data.x,data.y)
plt.xlabel('x')
plt.ylabel('y')
plt.title('Chart of Data on Production/Main Branch')
plt.show()
```

## Bonus

#### Additional SparkSQL Arctic/Nessie Syntax
- [SparkSQL Nessie/Arctic Syntax](https://projectnessie.org/tools/sql/)

#### Dremio Sonar Syntax
This is syntax to be used in the Dremio Sonar engine, the beauty is, it's the same catalog regardless which engine you use. So effects of commands in Dremio will carry over to Spark and vice-versa.

- [Assigning a Branch to a prior commit](https://docs.dremio.com/cloud/sql/commands/alter-branch/)
- [OPTIMIZING TABLE (Compaction)](https://docs.dremio.com/cloud/sql/commands/optimize-table/)

#### OTHER
- [Nessie/Arctic with Flink](https://projectnessie.org/tools/iceberg/flink/)
- [Nessie/Arctic with Hive](https://projectnessie.org/tools/iceberg/hive/)
- [Nessie/Arctic with Presto](https://prestodb.io/docs/current/connector/iceberg.html)
