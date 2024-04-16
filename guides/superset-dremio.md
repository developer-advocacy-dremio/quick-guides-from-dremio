## Trying out Building BI Dashboards from SuperSet from your Laptop

[Video Demonstrating how to setup Apache Superset and connect with Dremio](https://youtu.be/604i8vaukZs)

[Link to Repo with Dockerfile for dremio-superset Docker Image](https://github.com/AlexMercedCoder/dremio-superset-docker-image)

run the container

```shell
docker run -d -p 8080:8088 --name superset alexmerced/dremio-superset
```

then run
```
docker exec -it superset superset init
```

- then login at `http://localhost:8080/login/` with username `admin` and password `admin`

Use the dockerfile to rebuild this image with a custom secret key and admin account email.

This image is usuable for evaluation and and education.

The URL to connect to Dremio Cloud

```
dremio+flight://data.dremio.cloud:443/?token=<PAT>&UseEncryption=true
```

Make sure the PAT is URL encoded by opening up the browser and using `EncodeURIComponent()` or you can do so from [this website](https://www.urlencoder.org/).

## Using Dremio Software - Local Setup

docker-compose.yml

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
  # Superset for Building BI Dashboard
  dashboards:
    image: alexmerced/dremio-superset
    container_name: dashboards
    networks:
      dremio-laptop-lakehouse:
    ports:
      - 8080:8088
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
- run `docker-compose up`
- Setup dremio following the directions [here](./nessie_dremio.md)
- run `docker-compose exec dashboards superset init`
- login at `localhost:8080/login`
- use the url to esablish connection `dremio+flight://<dremio-username>:<dremio-password@dremio:32010/?UseEncryption=false` (if not using the docker-compose file above change `dremio` to the ip address to machine with Dremio running)
- If you need to look up the ip address of a docker container use `docker network ls` to see your docker networks then `docker network <name_or_id>` to see the details of the containers on that network
- click test connection
