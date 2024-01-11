## DBT - Data Build Tool - Setup Guide

- Setup Dremio Locally on our Laptop (only needed if don't already have Dremio environment)
- Setup Python Environment
- Configure your DBT Profile

## Setup Dremio (For Setting Up an Evaluation Environment to Practice in)

- Create a `docker-compose.yml`

```yml
version: "3"

services:
  # Nessie Catalog Server Using In-Memory Store
  catalog:
    image: projectnessie/nessie:0.76.0
    container_name: catalog
    networks:
      dremio-laptop-lakehouse:
    ports:
      - 19120:19120
  # Minio Storage Server
  storage:
    image: minio/minio:RELEASE.2024-01-01T16-36-33Z
    container_name: storage
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=storage
      - MINIO_REGION_NAME=us-east-1
      - MINIO_REGION=us-east-1
    networks:
      dremio-laptop-lakehouse:
    ports:
      - 9001:9001
      - 9000:9000
    command: ["server", "/data", "--console-address", ":9001"]
  # Dremio
  dremio:
    platform: linux/x86_64
    image: dremio/dremio-oss:latest
    ports:
      - 9047:9047
      - 31010:31010
      - 32010:32010
    container_name: dremio
    networks:
      dremio-laptop-lakehouse:
networks:
  dremio-laptop-lakehouse:

```

- [Directions for Dremio Setup](https://github.com/developer-advocacy-dremio/quick-guides-from-dremio/blob/main/guides/nessie_dremio.md)

## Setup Python Environment, install dbt

- `python -m venv venv`

- `source ./venv/bin/activate`

- `pip install dbt-dremio`

## Create a dbt project

The end result will create a profile which can be found in `~/.dbt/profiles.yml`

- `dbt init <projectname>`

- select dremio

- select dremio with software username/password

- put `127.0.0.1` as host

- use `9047` as port

- put username and password (or username/PAT if using Dremio cloud or choose software with PAT)

- use the name of nessie/metastore/object storage source for "object_storage_soure" (can also be as arctic catalog for cloud)

- write a path to a subfolder in that source for "object_storage_path"

- write the name of a space for "dremio_space" (for Dremio Cloud, this should be the name of an Arctic catalog)

- write the path to a subfolder in your space for "dremio_space_path"

- select 1 thread

## dbt function

**{{ config() }}**

Configures the behavior for the following model.

Example Arguments:

- `materialized`: `view` to create a sql view or `table` to create a table

- `database`: The dremio space (view) or source (table) to create the result in

- `schema`: the path to a subfolder in the source to out the results.

**{{ ref() }}**

Reference to a source model. This ensure that the referenced model will be run before this model.

## dbt commands

- `dbt init <projectname>` - create a new project
- `dbt run` run dbt models

## Using the project.yml to configure groups of models

You can specify different locations for different models by folder or tag like below.

```yml
models:
  my_project:
    # Apply to all models
    +materialized: view

    # Configuration for models in a specific folder
    marketing:
      +schema: marketing_schema
      +database: marketing_database

    # Configuration for models with a specific naming pattern
    tag:bi:
      +materialized: table
```

You can can tag different models like so...

In a single model:

```sql
{{ config(tags=["daily", "analytics"]) }}
```

Groups of models from project.yml

```yml
models:
  my_project:
    # Apply tags to all models in a specific directory
    marketing:
      +tags: ["marketing", "weekly"]
    # Apply tags to a specific model
    my_model:
      +tags: ["core_model", "daily"]
```

You can even only run models with certain tags using:

```
dbt run --model tag:daily
```
