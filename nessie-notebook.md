## Creating a Nessie/Spark Notebook Environment

This assumes you have docker installed.

#### Step 1 - Creating the DOCKERFILE

- Open your editor (VSCODE, etc.) to an empty directory.

- Create a file called `docker-compose.yml`

- Put the following in the file

```docker
#### Nessie + Notebook Playground Environment
services:
 spark-notebook:
   image: alexmerced/spark33-notebook
   ports:
     - "8888:8888"
 nessie:
   image: projectnessie/nessie
   ports:
     - "19120:19120"
```

- Then run the command `docker-compose up`

- copy the url to Jupyter Notebook from terminal output, should be a url like `http://127.0.0.1:8888/?token=c27b1fd57027a7f83820810e14c59e524217ba504f3c236a`

- create a new notebook with the following python code:

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
WAREHOUSE = "nessie"
URI = "http://nessie:19120/api/v1"


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.13:1.2.0,org.projectnessie.nessie-integrations:nessie-spark-extensions-3.3_2.13:0.54.0')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.nessie', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.nessie.uri', URI)
        .set('spark.sql.catalog.nessie.ref', 'main')
        .set('spark.sql.catalog.nessie.authentication.type', 'NONE')
        .set('spark.sql.catalog.nessie.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.nessie.warehouse', WAREHOUSE)
        .set('spark.sql.catalog.nessie.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run Queries
spark.sql("CREATE TABLE nessie.names (name STRING) USING iceberg").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("CREATE BRANCH IF NOT EXISTS my_branch IN nessie;").show()
    
spark.sql("USE REFERENcE my_branch IN nessie").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("USE REFERENcE main IN nessie").show() 
    
spark.sql("SELECT * FROM nessie.names").show()
```
