# Simple Migration with Query Federation

This workshop demonstrates how to perform a seamless migration from a traditional relational database to an Iceberg lakehouse, while keeping the experience completely transparent to downstream consumers. 

You will walk through two different paths—one using open-source Trino and another using Dremio Cloud.

## Table of Contents
- [What you will prove](#what-you-will-prove)
- [Why this is useful](#why-this-is-useful)
- [The Sample Model](#the-sample-model)
- [Part 1: Trino Workshop](#part-1-trino-workshop)
  - [Prerequisites](#prerequisites)
  - [Start the Stack](#start-the-stack)
  - [Seed PostgreSQL](#seed-postgresql)
  - [Create Stable Views](#create-stable-views)
  - [Copy the Tables](#copy-the-tables)
  - [Cut Over the Views](#cut-over-the-views)
  - [Verify](#verify)
- [Part 2: Dremio Cloud Workshop](#part-2-dremio-cloud-workshop)
  - [Prerequisites](#prerequisites-1)
  - [Create the PostgreSQL Source in Neon](#create-the-postgresql-source-in-neon)
  - [Connect Neon to Dremio Cloud](#connect-neon-to-dremio-cloud)
  - [Create Semantic Layer Views](#create-semantic-layer-views)
  - [Replicate into Open Catalog](#replicate-into-open-catalog)
  - [Cut Over the Views](#cut-over-the-views-1)
  - [Add Semantics](#add-semantics)
- [What to Compare](#what-to-compare)

## What you will prove
You will create stable views first, then move the physical tables from PostgreSQL to Iceberg without changing the consumer-facing object names.

## Why this is useful
Consumers can keep querying the same view path while the storage layer changes underneath. This decouples the query logic from the physical storage, enabling zero-downtime migrations and a robust semantic layer.

## The Sample Model
We will use three small tables to simulate a simple retail backend. 

Run the following SQL in your PostgreSQL instance to create and populate the sample tables:

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date DATE NOT NULL,
    status VARCHAR(50)
);

CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_name VARCHAR(255) NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

INSERT INTO customers (name, email) VALUES
('Alice Smith', 'alice@example.com'),
('Bob Jones', 'bob@example.com');

INSERT INTO orders (customer_id, order_date, status) VALUES
(1, '2026-04-01', 'shipped'),
(2, '2026-04-02', 'pending');

INSERT INTO order_items (order_id, product_name, quantity, price) VALUES
(1, 'Laptop', 1, 1200.00),
(1, 'Mouse', 1, 25.00),
(2, 'Keyboard', 1, 150.00);
```

---

## Part 1: Trino Workshop

In this section, we will use open-source Trino to federate queries and perform the migration. 

### Prerequisites
- **Docker Compose** installed locally.
- A basic understanding of containers.

The stack will include:
- **Trino**: The federated query engine.
- **Nessie**: The Nessie-backed Iceberg REST catalog for managing the destination lakehouse and views.
- **MinIO-compatible object store**: We will use AIStor Free (or equivalent S3-compatible store) to store the physical Iceberg tables.
- **PostgreSQL**: Our source system.

### Start the Stack
Create a directory for your lab and set up the necessary Trino catalog configuration files.

Trino uses two catalogs for this workshop:

1. **`postgres` catalog**:
```properties
# etc/catalog/postgres.properties
connector.name=postgresql
connection-url=jdbc:postgresql://postgres:5432/workshop
connection-user=postgres
connection-password=postgres
```

2. **`lakehouse` catalog** (configured as a Nessie Iceberg REST catalog):
```properties
# etc/catalog/lakehouse.properties
connector.name=iceberg
fs.native-s3.enabled=true
iceberg.catalog.type=rest
iceberg.rest-catalog.uri=http://nessie:19120/iceberg/
iceberg.rest-catalog.prefix=main
iceberg.rest-catalog.vended-credentials-enabled=true
s3.endpoint=http://minio:9000
s3.path-style-access=true
s3.region=us-east-1
```

*(Note: The Iceberg REST catalog is necessary in Trino to support view management using the Iceberg View specification. Nessie's REST support is currently experimental but fully functional for this lab.)*

Create a `docker-compose.yml` file to run all four services together:

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: workshop
    ports:
      - "5432:5432"

  minio:
    image: minio/minio
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: password
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"

  minio-init:
    image: minio/mc
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
      sleep 5;
      /usr/bin/mc config host add myminio http://minio:9000 admin password;
      /usr/bin/mc mb myminio/workshop;
      exit 0;
      "

  nessie:
    image: projectnessie/nessie:0.107.5
    environment:
      NESSIE_CATALOG_DEFAULT_WAREHOUSE: workshop
      NESSIE_CATALOG_WAREHOUSES_WORKSHOP_LOCATION: s3://workshop/
      NESSIE_CATALOG_WAREHOUSES_WORKSHOP_CONFIG_S3_ENDPOINT: http://minio:9000
      NESSIE_CATALOG_WAREHOUSES_WORKSHOP_CONFIG_S3_ACCESS_KEY: admin
      NESSIE_CATALOG_WAREHOUSES_WORKSHOP_CONFIG_S3_SECRET_KEY: password
      NESSIE_CATALOG_WAREHOUSES_WORKSHOP_CONFIG_S3_PATH_STYLE_ACCESS: "true"
    ports:
      - "19120:19120"
    depends_on:
      - minio-init

  trino:
    image: trinodb/trino:480
    volumes:
      - ./etc/catalog:/etc/trino/catalog
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - minio
      - nessie
```

Once your `docker-compose.yml` and catalog files (`./etc/catalog/postgres.properties` and `./etc/catalog/lakehouse.properties`) are in place, start the stack by running:

```bash
docker compose up -d
```

Verify that all services are running and accessible. Wait a few moments for Trino to fully initialize.

### Seed PostgreSQL
Connect to the `postgres` container and run the sample SQL provided in "The Sample Model" section to create the `workshop` database and tables.

### Create Stable Views
Connect to Trino via the Trino CLI or your preferred SQL tool. Create views in the `lakehouse` catalog that initially read directly from PostgreSQL. This establishes the stable consumer contract.

```sql
-- create stable abstraction first
CREATE SCHEMA IF NOT EXISTS lakehouse.workshop;

CREATE OR REPLACE VIEW lakehouse.workshop.customers_v AS
SELECT * FROM postgres.public.customers;

CREATE OR REPLACE VIEW lakehouse.workshop.orders_v AS
SELECT * FROM postgres.public.orders;

CREATE OR REPLACE VIEW lakehouse.workshop.order_items_v AS
SELECT * FROM postgres.public.order_items;
```

### Copy the Tables
Use Trino to migrate the physical data from PostgreSQL into Iceberg tables in the lakehouse.

```sql
-- replicate physical tables into Iceberg
CREATE TABLE lakehouse.workshop.customers AS
SELECT * FROM postgres.public.customers;

CREATE TABLE lakehouse.workshop.orders AS
SELECT * FROM postgres.public.orders;

CREATE TABLE lakehouse.workshop.order_items AS
SELECT * FROM postgres.public.order_items;
```

### Cut Over the Views
Now, redefine the views so that the logical names point to the newly created Iceberg tables instead of PostgreSQL. 

```sql
-- switch logical objects to the lakehouse
CREATE OR REPLACE VIEW lakehouse.workshop.customers_v AS
SELECT * FROM lakehouse.workshop.customers;

CREATE OR REPLACE VIEW lakehouse.workshop.orders_v AS
SELECT * FROM lakehouse.workshop.orders;

CREATE OR REPLACE VIEW lakehouse.workshop.order_items_v AS
SELECT * FROM lakehouse.workshop.order_items;
```

### Verify
Run the same query before and after the cutover:
```sql
SELECT * FROM lakehouse.workshop.orders_v;
```
Notice that the consumer-facing object path (`lakehouse.workshop.orders_v`) did not change, achieving a transparent migration.

---

## Part 2: Dremio Cloud Workshop

In this section, we will achieve the same outcome using Dremio Cloud as the federated query engine and semantic layer, with Neon providing the hosted PostgreSQL source.

### Prerequisites
- **Dremio Cloud trial** account (30 days with $400 in free credits).
- **Neon free account** (includes 100 projects and hosted PostgreSQL).

### Create the PostgreSQL Source in Neon
1. Log in to Neon and create a new project.
2. Open the Neon SQL Editor.
3. Run the sample SQL provided in "The Sample Model" section to create and populate the tables.

### Connect Neon to Dremio Cloud
1. Log in to Dremio Cloud.
2. Add a new Source, selecting **PostgreSQL**.
3. Name the source (e.g., `neon`).
4. Copy the connection details from the Neon dashboard (host, port, user, password, and database).
5. Ensure you enable **Encrypt connection** (SSL/TLS is required by Neon).

### Create Semantic Layer Views
Use the Dremio SQL Runner to create folders and stable views in Dremio. These views initially read from the live Neon source.

```sql
CREATE FOLDER IF NOT EXISTS workshop;

-- stable semantic layer on top of the live Neon source
CREATE OR REPLACE VIEW workshop.customers_v AS
SELECT * FROM <source_name>.public.customers;

CREATE OR REPLACE VIEW workshop.orders_v AS
SELECT * FROM <source_name>.public.orders;

CREATE OR REPLACE VIEW workshop.order_items_v AS
SELECT * FROM <source_name>.public.order_items;
```
*Note: Replace `<source_name>` with the name you gave your PostgreSQL source in Dremio (e.g., `neon`).*

### Replicate into Open Catalog
Use `CREATE TABLE AS SELECT` to pull data from Neon and write it as Iceberg tables directly into Dremio's built-in Open Catalog (powered by Apache Polaris).

```sql
-- replicate into Iceberg tables in the built-in Open Catalog
CREATE TABLE workshop.customers AS
SELECT * FROM <source_name>.public.customers;

CREATE TABLE workshop.orders AS
SELECT * FROM <source_name>.public.orders;

CREATE TABLE workshop.order_items AS
SELECT * FROM <source_name>.public.order_items;
```

### Cut Over the Views
Redefine the views to redirect the same paths to the Iceberg tables in the Open Catalog.

```sql
-- switch the semantic layer to the lakehouse copies
CREATE OR REPLACE VIEW workshop.customers_v AS
SELECT * FROM workshop.customers;

CREATE OR REPLACE VIEW workshop.orders_v AS
SELECT * FROM workshop.orders;

CREATE OR REPLACE VIEW workshop.order_items_v AS
SELECT * FROM workshop.order_items;
```

### Add Semantics
Dremio excels at maintaining an AI-ready Semantic Layer. To enrich the consumer experience:
1. Create an aggregated "business" view:
```sql
CREATE OR REPLACE VIEW workshop.order_facts_v AS
SELECT 
  o.order_id, 
  c.name AS customer_name, 
  o.order_date, 
  o.status, 
  SUM(oi.quantity * oi.price) AS total_amount
FROM workshop.orders_v o
JOIN workshop.customers_v c ON o.customer_id = c.customer_id
JOIN workshop.order_items_v oi ON o.order_id = oi.order_id
GROUP BY 1, 2, 3, 4;
```
2. Navigate to `workshop.order_facts_v` in the Dremio UI.
3. Open the **Wiki** panel and add markdown describing the logic (e.g., "Contains order total amounts calculated from item quantities and prices").
4. Add **Labels** (like `sales` or `certified`) so users can easily discover this curated dataset.

---

## What to Compare

Once you have completed both paths, reflect on the differences:
- **Setup friction**: Docker Compose vs. SaaS signups (Dremio/Neon).
- **How views are managed**: Trino requiring Iceberg REST catalogs for view specifications vs. Dremio's native logical abstractions.
- **Where the semantic layer lives**: Trino's catalog-level view definitions vs. Dremio's rich semantic layer with Wikis and Labels.
- **What the cutover looks like**: In both paths, it's a seamless `CREATE OR REPLACE VIEW`.
- **How much stays the same for query consumers**: Both paths succeed in insulating the user completely from the underlying storage migration!
