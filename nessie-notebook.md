## Creating a Nessie/Spark Notebook Environment

This assumes you have docker installed.

#### Step 1 - Creating the DOCKERFILE

- Open your editor (VSCODE, etc.) to an empty directory.

- Create a file called `docker-compose.yml`

- Put the following in the file

```yml
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
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.0,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.54.0')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.nessie', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.nessie.uri', URI)
        .set('spark.sql.catalog.nessie.ref', 'main')
        .set('spark.sql.catalog.nessie.authentication.type', 'NONE')
        .set('spark.sql.catalog.nessie.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.nessie.warehouse', WAREHOUSE)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run Queries
spark.sql("CREATE TABLE nessie.names (name STRING) USING iceberg").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("CREATE BRANCH IF NOT EXISTS my_branch IN nessie").show()
    
spark.sql("USE REFERENcE my_branch IN nessie").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("USE REFERENcE main IN nessie").show() 
    
spark.sql("SELECT * FROM nessie.names").show()
```

If you wanted to try this out but have it write to S3, here is how the settings would differ...

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY") ## AWS CREDENTIALS
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_KEY") ## AWS CREDENTIALS
WAREHOUSE = os.environ.get("WAREHOUSE") ## S3 Address to Write to
URI = "http://nessie:19120/api/v1"


conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.3_2.12:1.2.0,org.projectnessie:nessie-spark-extensions-3.3_2.12:0.54.0')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.nessie', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.nessie.uri', URI)
        .set('spark.sql.catalog.nessie.ref', 'main')
        .set('spark.sql.catalog.nessie.authentication.type', 'NONE')
        .set('spark.sql.catalog.nessie.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set('spark.sql.catalog.nessie.warehouse', WAREHOUSE)
        .set('spark.sql.catalog.arctic.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
  		#AWS CREDENTIALS
        .set('spark.hadoop.fs.s3a.access.key', AWS_ACCESS_KEY)
        .set('spark.hadoop.fs.s3a.secret.key', AWS_SECRET_KEY)
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## Run Queries
spark.sql("CREATE TABLE nessie.names (name STRING) USING iceberg").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("CREATE BRANCH IF NOT EXISTS my_branch IN nessie").show()
    
spark.sql("USE REFERENcE my_branch IN nessie").show()
    
spark.sql("INSERT INTO nessie.names VALUES ('Alex Merced'), ('Dipankar Mazumdar'), ('Jason Huges')").show()
    
spark.sql("SELECT * FROM nessie.names").show()
    
spark.sql("USE REFERENcE main IN nessie").show() 
    
spark.sql("SELECT * FROM nessie.names").show()
```

With the above keep in mind you'll want your AWS credentials defined in your docker-compose.yml

```yml
#### Nessie + Notebook Playground Environment
services:
 spark-notebook:
   image: alexmerced/spark33-notebook
   ports:
     - "8888:8888"
   environment:
      - AWS_ACCESS_KEY=xxxxxxxxxxxxxxxx
      - AWS_SECRET_KEY=xxxxxxxxxxxxxxxx
      - WAREHOUSE=s3a://someS3Bucket/
 nessie:
   image: projectnessie/nessie
   ports:
     - "19120:19120"
```

For more ways to pass environmental variables in Docker Compose [read this](https://release.com/blog/how-to-set-docker-compose-environment-variables)
