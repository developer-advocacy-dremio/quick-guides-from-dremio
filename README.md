# quick-guides-from-dremio
Quick Guides from Dremio on Several topics

Want to learn more about Dremio? Here is a [great list of resources](./digests/getstarted.md).

Just need to the docker command to try it out on your laptop?
```
docker run -p 9047:9047 -p 31010:31010 -p 45678:45678 -p 32010:32010 -e DREMIO_JAVA_SERVER_EXTRA_OPTS=-Dpaths.dist=file:///opt/dremio/data/dist --name try-dremio dremio/dremio-oss
```


* Use the local lakehouse guide below to add a Nessie Catalog and Minio Object Storage to Local Setup
* If you want to deploy a single node instance to an Ubuntu Based VM, [Follow These Directions](https://www.dremio.com/blog/evaluating-dremio-deploying-a-single-node-instance-on-a-vm/)

## Dremio
- [Creating a Local Dremio/Nessie/Minio Lakehouse on your Laptop for Evaluation](./guides/nessie_dremio.md)
- [Repo of Docker-Compose Examples](https://github.com/developer-advocacy-dremio/dremio-compose)
- [Dremio Cloud SQL Function Examples](./guides/dremiocloudsql.md)
- [Dremio Cloud Data Quality and Validation Examples](./guides/dremiocloudquality.md)
- [Dremio Arctic in Spark Notebook](./guides/arcticexercise.md)
- [Creating a Local Dremio/Spark/Minio Environment for Experimentation](./guides/icebergminiodremio.md)
- [Configuring Dremio with dbt](https://github.com/AlexMercedCoder/dbt-with-dremio-walkthrough-template)
- [Trying Superset with Dremio Cloud from your Laptop](./guides/superset-dremio.md)
- [Examples of Dremio for Machine Learning](./guides/dremio_ml.md)

## Iceberg
- [Iceberg 101 Course](https://www.dremio.com/subsurface/apache-iceberg-101-your-guide-to-learning-apache-iceberg-concepts-and-practices/)
- [Iceberg & Tableau](./guides/icebergtableau.md)
- [Iceberg with Spark 3.4 Quickstart](./guides/iceberg-start.md)
- [Iceberg & pySpark Example](./guides/icebergpyspark.md)
- [Iceberg & Dremio Example Queries](./guides/icebergdremio.md)
- [Creating a Local Environment with Nessie, Spark, and Notebook](./guides/nessie-notebook.md)
- [Creating a Local Spark/Notebook Environment](./guides/sparknotebook.md)
- [Apache Iceberg Catalog Migration](./guides/catalogmigration.md)
- [Spark Catalog Configuration Spark Scripts](./guides/bashscript.md)


## Apache Arrow
- [Simple Apache Arrow Flight Client Python Code](./guides/arrowclientpy.md)
- [Python ODBC and Arrow connecto to Dremio examples](./guides/pythonodbcarrow.md)

## Other
- [Connecting R Applications to Dremio via ODBC](./guides/rodbc.md)
- [Querying Dremio in Other Languages](./guides/languages.md)
