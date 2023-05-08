# Apache Iceberg Catalog Migration

There is now a catalog migration tool for migrating tables from one catalog to another.

[FIND DOCUMENTATION HERE](https://github.com/projectnessie/iceberg-catalog-migrator)

To download the tool in your current folder using `wget` just do the following:

```
wget https://github.com/projectnessie/iceberg-catalog-migrator/releases/download/catalog-migrator-0.2.0/iceberg-catalog-migrator-cli-0.2.0.jar
```

Then you can just run the file with the following command assuming you have Java JDK 11+ installed:

```
java -jar iceberg-catalog-migrator-cli-0.2.0.jar
```

You have two commands: 

- `migrate`: add tables to new catalog, remove the from old catalog
- `register`: move tables to new catalog, don't remove from old catalog (avoid using tables from multiple catalogs for consistency)

You can run this jar with the command and series of flags providing details on:

- `source`: The catalog that currently has the tables, what kind of catalog, auth credentials, etc.
- `target`: The catalog that you registering the tables with

For example: 

```
java -jar iceberg-catalog-migrator-cli-0.2.0.jar migrate \
--source-catalog-type HADOOP \
--source-catalog-properties warehouse=/tmp/warehouse,type=hadoop \
--target-catalog-type NESSIE  \
--target-catalog-properties uri=http://localhost:19120/api/v1,ref=main,warehouse=/tmp/warehouse
```

## Flags

### CLI OPTIONS
- `--output-dir`: Path for writing CLI logs
- `--dry-run`: to run a simulation, but move no tables, just output what would've happened
- `--disable-safety-prompts`: Disable prompts that seek user input
- `--stacktrace`: captures stack trace in log files
- `-h, --help`: show help message
- `-v, --version`: show version number
