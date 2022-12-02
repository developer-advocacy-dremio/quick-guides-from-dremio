## Creating a Spark/Notebook Environment

The purpose of this guide is to be able to run Spark Locally in a docker container and be able to write notebooks that use Spark from that container. Having docker installed is required.

## Starting the Container

Run the command

```
docker run -it --name spark-notebook -p 8888:8888 alexmerced/spark33playground
```

This command does the following
- `-it` starts the container in interactive mode which you can exit with the command `exit`
- `--name` this gives the container a name so you can easily turn it on and off with `docker start spark-notebook && docker attach spark-notebook` and `docker stop spark-notebook`
- `-p 8080:8080` maps port 8080 in the container to port 8080 in the host machine
- `alexmerced/spark33playground` a docker image that has Spark 3.3 running [Dockerfile used to create image](https://github.com/AlexMercedCoder/apache-iceberg-docker-starter-image/blob/main/SPARK33ICEBERGNESSIE.DOCKERFILE)

## Get the notebook server running

Once you are in the Docker containers shell we need to install jupyter notebook.

```
pip install notebook pyspark
```

The normal command `jupyter-notebook` won't work so we'll have to use the binary directly and pass it a flag to host the server on `0.0.0.0` so it is accessible outside of the container.

```
~/.local/bin/jupyter-notebook --ip 0.0.0.0
```

Now you have a notebook environment that should work

## Example of Using Iceberg to Write to the Local Container

In the example below we'll have a notebook where we run some queries to the local container. To see other pySpark examples of configuring the iceberg catalog for different catalogs checkout [this section of this repository on Iceberg/Python](https://github.com/developer-advocacy-dremio/quick-guides-from-dremio/blob/main/icebergpyspark.md).

```py
import os
import sys

## Install pySpark
!{sys.executable} -m pip install pyspark

## import pyspark
import pyspark
from pyspark.sql import SparkSession

conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.0.0,software.amazon.awssdk:bundle:2.17.178,software.amazon.awssdk:url-connection-client:2.17.178')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.iceberg', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.iceberg.type', 'hadoop')
        .set('spark.sql.catalog.iceberg.warehouse', '$PWD/iceberg-warehouse')
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run a Query to create a table
spark.sql("CREATE TABLE iceberg.table1 (name string) USING iceberg;")

## Run a Query to insert into the table
spark.sql("INSERT INTO iceberg.table1 VALUES ('Alex'), ('Dipankar'), ('Jason')")

## Run a Query to get data
df = spark.sql("SELECT * FROM iceberg.table1")

## Display Dataframe
df.show()
```
