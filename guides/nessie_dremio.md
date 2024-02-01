## Lakehouse On Your Laptop

#### With Nessie/Minio/Dremio

**Pre-Reqs:** Docker & Docker-Compose installed

### Step 1 - The docker-compose.yml

The docker-compose.yml will define all the pieces you need in your lakehouse which will include:

- Nessie: Catalog with Git-Like functionality for Apache Iceberg tables

- Minio: S3 Compliant Object Storage software to act as our data lake storage.

- Dremio: Data Lakehouse platform to provide an easy to use and fast point of access for the Apache Iceberg tables stored on Nessie/Minio and other sources we connect.

`docker-compose.yml`

```yaml
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

* If you want the files minio writes to be in your host file system add a volume entry

```yaml
image: minio/minio
    volumes:
      - /path/to/your/host/directory:/data
```

Open up a terminal in the same folder as this `docker-compose.yml` file and run the command

```shell
# latest versions of docker-desktop
docker compose up

# older versions
docker-compose up
```

This will create all the containers specified in our `docker-compose.yml` if you ever need to shut them down in another terminal in the same folder just run:

```shell
docker compose down
## or
docker-compose down
```

### Step 2 - Setting up our storage bucket

- Open up an internet browser

- Visit minio at `http://localhost:9001`

- login with the username: `admin` and the password: `password` (these were specified in the `docker-compose.yml`)

- Create a bucket, let's call it `warehouse`

### Step 3 - Connect Dremio to Data Lake

- Open up a new internet browser tab

- Visit Dremio at `http://localhost:9047`

- Fill out the form to create your account

- Then on the dashboard choose to connect a new source

- Select Nessie as your new source

There are two sections we need to fill out, the **general** and **storage** sections:

##### General (Connecting to Nessie Server)
- Set the name of the source to “nessie”
- Set the endpoint URL to “http://catalog:19120/api/v2”
Set the authentication to “none”

##### Storage Settings 
##### (So Dremio can read and write data files for Iceberg tables)

- For your access key, set “admin”
- For your secret key, set “password”
- Set root path to “/warehouse”
    Set the following connection properties:
    - `fs.s3a.path.style.access` to `true`
    - `fs.s3a.endpoint` to `minio:9000`
    - `dremio.s3.compat` to `true`
- Uncheck “encrypt connection” (since our local Nessie instance is running on http)

### Testing it Out

- Head to the SQL Runner on Dremio

- Run the following SQL statements

```
CREATE TABLE nessie.names (name varchar);
INSERT INTO nessie.names VALUES ('Gnarly the Narwhal');
SELECT * FROM nessie.names;
```

- Go explore you storage on minio, you should see all the Apache Iceberg data & metadata stored in your warehouse bucket.

### Things to Be Aware Of
- By default the Nessie images uses memory to store references, so if you shut down and restart the cluster Nessie will forget your tables. [Configure Nessie with a Data Store to Persist Nessie Metadata](https://projectnessie.org/try/configuration/#support-for-the-database-specific-implementations)
